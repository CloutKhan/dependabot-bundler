# dependabot-bundler
Demonstrate the `Bundler::GemfileNotFound` thrown during `fetch_path_gemspec_paths`. See [initial discovery](https://github.com/Skenvy/dependabot-linguist/issues/6).

In [`::Dependabot::Bundler::FileFetcher::fetch_path_gemspec_paths`](https://github.com/dependabot/dependabot-core/blob/v0.217.0/bundler/lib/dependabot/bundler/file_fetcher.rb#L172-L191), if a `Gemfile.lock` or `gems.locked` exists, an instance of [`::Bundler::LockfileParser`](https://github.com/rubygems/rubygems/blob/bundler-v2.4.12/bundler/lib/bundler/lockfile_parser.rb#L59) will be initialised.

This cascades down to a [`raise GemfileNotFound`](https://github.com/rubygems/rubygems/blob/bundler-v2.4.12/bundler/lib/bundler.rb#L305-L313). There is no [`rescue ::Bundler::GemfileNotFound`](https://github.com/dependabot/dependabot-core/blob/v0.217.0/bundler/lib/dependabot/bundler/file_fetcher.rb#L185-L187) in [`::Dependabot::Bundler::FileFetcher::fetch_path_gemspec_paths`](https://github.com/dependabot/dependabot-core/blob/v0.217.0/bundler/lib/dependabot/bundler/file_fetcher.rb#L172-L191).

Per the [example workflow](https://github.com/CloutKhan/dependabot-bundler/blob/main/.github/workflows/GemfileNotFound.yml), `::Bundler::GemfileNotFound` is thrown during [::Bundler::LockfileParser.new(...)](https://github.com/dependabot/dependabot-core/blob/v0.217.0/bundler/lib/dependabot/bundler/file_fetcher.rb#L174-L175) for directories that do contain a `Gemfile`, that dependabot's file fetcher is able to find, independent of `::Bundler`.

So running the bundler file fetcher on a directory with a lockfile will end up with dependabot throwing a `::Bundler::GemfileNotFound`. _Unless a `Gemfile` exists in the directory that the script is being run from_.
