# dependabot-bundler
Demonstrate the `Bundler::GemfileNotFound` thrown during `fetch_path_gemspec_paths`. See [initial discovery](https://github.com/Skenvy/dependabot-linguist/issues/6).

In [`::Dependabot::Bundler::FileFetcher::fetch_path_gemspec_paths`](https://github.com/dependabot/dependabot-core/blob/v0.217.0/bundler/lib/dependabot/bundler/file_fetcher.rb#L172-L191), if a `Gemfile.lock` or `gems.locked` exists, an instance of [`::Bundler::LockfileParser`](https://github.com/rubygems/rubygems/blob/bundler-v2.4.12/bundler/lib/bundler/lockfile_parser.rb#L59) will be initialised.

This cascades down to a [`raise GemfileNotFound`](https://github.com/rubygems/rubygems/blob/bundler-v2.4.12/bundler/lib/bundler.rb#L305-L313).
