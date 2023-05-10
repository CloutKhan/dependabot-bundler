# dependabot-bundler
Demonstrate the `Bundler::GemfileNotFound` thrown during `fetch_path_gemspec_paths`. See [initial discovery](https://github.com/Skenvy/dependabot-linguist/issues/6).

In [`::Dependabot::Bundler::FileFetcher::fetch_path_gemspec_paths`](https://github.com/dependabot/dependabot-core/blob/v0.217.0/bundler/lib/dependabot/bundler/file_fetcher.rb#L172-L191), if a `Gemfile.lock` or `gems.locked` exists, an instance of [`::Bundler::LockfileParser`](https://github.com/rubygems/rubygems/blob/bundler-v2.4.12/bundler/lib/bundler/lockfile_parser.rb#L59) will be initialised.

This cascades down to a [`raise GemfileNotFound`](https://github.com/rubygems/rubygems/blob/bundler-v2.4.12/bundler/lib/bundler.rb#L305-L313). There is no [`rescue ::Bundler::GemfileNotFound`](https://github.com/dependabot/dependabot-core/blob/v0.217.0/bundler/lib/dependabot/bundler/file_fetcher.rb#L185-L187) in [`::Dependabot::Bundler::FileFetcher::fetch_path_gemspec_paths`](https://github.com/dependabot/dependabot-core/blob/v0.217.0/bundler/lib/dependabot/bundler/file_fetcher.rb#L172-L191).

Per the [example workflow](https://github.com/CloutKhan/dependabot-bundler/blob/main/.github/workflows/GemfileNotFound.yml), `::Bundler::GemfileNotFound` is thrown during [::Bundler::LockfileParser.new(...)](https://github.com/dependabot/dependabot-core/blob/v0.217.0/bundler/lib/dependabot/bundler/file_fetcher.rb#L174-L175) for directories that do contain a `Gemfile`, that dependabot's file fetcher is able to find, independent of `::Bundler`.

So running the bundler file fetcher on a directory with a lockfile will end up with dependabot throwing a `::Bundler::GemfileNotFound`. _Unless a `Gemfile` exists in the directory that the script is being run from_. Or unless a `BUNDLE_GEMFILE` environment variable is set to any garbage value.

The inverse call stack to the point that the `ENV["BUNDLE_GEMFILE"]` is read is
1. [`::Bundler::SharedHelpers::find_gemfile`](https://github.com/rubygems/rubygems/blob/bundler-v2.4.12/bundler/lib/bundler/shared_helpers.rb#L214-L218)
1. [`::Bundler::SharedHelpers::root`](https://github.com/rubygems/rubygems/blob/bundler-v2.4.12/bundler/lib/bundler/shared_helpers.rb#L13-L17)
1. [`::Bundler::root`](https://github.com/rubygems/rubygems/blob/bundler-v2.4.12/bundler/lib/bundler.rb#L305C12-L313)
1. [`::Bundler::app_cache`](https://github.com/rubygems/rubygems/blob/bundler-v2.4.12/bundler/lib/bundler.rb#L329-L332)
    1. With a default input `custom_path = nil` that if set would bypass the call to root; `path = custom_path || root`.
1. [`::Bundler::Source::Rubygems::cache_path`](https://github.com/rubygems/rubygems/blob/bundler-v2.4.12/bundler/lib/bundler/source/rubygems.rb#L460-L462)
1. [`::Bundler::Source::Rubygems::initialize`](https://github.com/rubygems/rubygems/blob/bundler-v2.4.12/bundler/lib/bundler/source/rubygems.rb#L15C11-L25)
1. [`::Bundler::Source::Rubygems::from_lock`](https://github.com/rubygems/rubygems/blob/bundler-v2.4.12/bundler/lib/bundler/source/rubygems.rb#L87C23-L89)
1. [`::Bundler::LockfileParser::parse_source`](https://github.com/rubygems/rubygems/blob/bundler-v2.4.12/bundler/lib/bundler/lockfile_parser.rb#L109C17-L147)
1. [`::Bundler::LockfileParser::initialize`](https://github.com/rubygems/rubygems/blob/bundler-v2.4.12/bundler/lib/bundler/lockfile_parser.rb#L59C16-L94)
1. [`::Dependabot::Bundler::FileFetcher::fetch_path_gemspec_paths`](https://github.com/dependabot/dependabot-core/blob/v0.217.0/bundler/lib/dependabot/bundler/file_fetcher.rb#L172-L191)

A [search of bundler's use of `caches`](https://github.com/search?q=repo%3Arubygems%2Frubygems+caches+path%3A%2F%5Ebundler%5C%2F%2F+NOT+path%3A%2F%5Ebundler%5C%2Fspec%5C%2F%2F&type=code) and of [its use of `cache_path`](https://github.com/search?q=repo%3Arubygems%2Frubygems+cache_path+path%3A%2F%5Ebundler%5C%2F%2F+NOT+path%3A%2F%5Ebundler%5C%2Fspec%5C%2F%2F&type=code) don't make it immediately obvious why there needs to be an error if the path can't be found, despite the benefit of the doubt that it's used somewhere else to know where to look for and open the `Gemfile`.
