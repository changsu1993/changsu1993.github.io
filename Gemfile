source "https://rubygems.org"

gem "jekyll", "~> 4.3"
gem "jekyll-theme-chirpy", "~> 7.0", ">= 7.0.1"

group :test do
  gem "html-proofer", "~> 5.0"
end

# Windows와 JRuby는 zoneinfo 파일을 포함하지 않으므로
# tzinfo-data gem과 관련 라이브러리를 번들링합니다
install_if -> { RUBY_PLATFORM =~ %r!mingw|mswin|java! } do
  gem "tzinfo", "~> 1.2"
  gem "tzinfo-data"
end

# Windows에서 디렉토리 감시를 위한 성능 부스터
gem "wdm", "~> 0.1.1", :install_if => Gem.win_platform?

# JRuby 빌드에서 `http_parser.rb` gem을 `v0.6.x`로 고정
# 최신 버전에는 Java 대응이 없기 때문입니다
gem "http_parser.rb", "~> 0.6.0", :platforms => [:jruby]

# 플러그인
group :jekyll_plugins do
  gem "jekyll-feed", "~> 0.12"
  gem "jekyll-sitemap"
  gem "jekyll-seo-tag"
  gem "jekyll-archives"
  gem "jekyll-paginate"
end
