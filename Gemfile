# frozen_string_literal: true

source 'https://rubygems.org'

# gem 'retlab', path: '../retlab'

group :jekyll_plugins do
  gem 'jekyll', '~> 4.0'
  gem 'jekyll-avatar'
  gem 'jekyll-feed'
  gem 'jekyll-github-metadata'
  gem 'jekyll-mentions'
  gem 'jekyll-redirect-from'
  gem 'jekyll-relative-links'
  gem 'jekyll-sass-converter', '~> 3.0'
  gem 'jekyll-seo-tag'
  gem 'jekyll-sitemap'
  gem 'jemoji'
  gem 'retlab', github: 'benbalter/retlab'
end

group :test, :development do
  gem 'classifier-reborn', github: 'jekyll/classifier-reborn'
  gem 'github-pages-health-check'
  #gem 'gsl', github: 'SciRuby/rb-gsl' - Not compatible with Ruby 3.0
  gem 'wdm', '>= 0.1.0' if Gem.win_platform?
  gem 'numo-narray' if !Gem.win_platform?
  gem 'numo-linalg' if !Gem.win_platform?
  gem 'html-proofer'
  gem 'nokogiri'
  gem 'pry'
  gem 'rake'
  gem 'rspec'
  gem 'rubocop'
  gem 'rubocop-github'
  gem 'rubocop-performance'
  gem 'rubocop-rspec'
  gem 'sass'
  gem 'webrick'
end
