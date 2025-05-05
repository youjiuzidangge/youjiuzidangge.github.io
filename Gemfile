source 'https://rubygems.org'

git_source(:github) do |repo_name|
  repo_name = "#{repo_name}/#{repo_name}" unless repo_name.include?("/")
  "https://github.com/#{repo_name}.git"
end

gem 'jekyll'
gem 'jekyll-paginate'
gem 'rouge'
gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]

gem 'eventmachine', git: 'https://github.com/eventmachine/eventmachine.git', tag: 'v1.2.7'

# gem 'wdm', '>= 0.1.0' if Gem.win_platform?

group :development do
  gem 'wdm', '>= 0.1.0', platforms: [:mswin, :mingw, :x64_mingw]
end


