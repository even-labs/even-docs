ruby "3.2.0"
source 'https://rubygems.org'

# Middleman
# Pinned to the 4.4.x line: 4.5/4.6 require rack >= 3, which forces the
# Sprockets 3 -> 4 migration and breaks the .css.scss asset pipeline.
gem 'middleman', '~> 4.4.0'
gem 'middleman-syntax', '~> 3.2'
gem 'middleman-autoprefixer', '~> 3.0'
gem 'middleman-sprockets', '~> 4.1'
gem 'rouge', '~> 3.21'
gem 'redcarpet', '~> 3.6.0'
# Relaxed from ~> 1.13.3 so we can reach the security-patched 1.18.9+ line.
gem 'nokogiri', '~> 1.18'
gem 'sass'
gem 'webrick'

# Security floor for a transitive dependency. Middleman 4.4 caps activesupport
# at < 7.1, so 6.1.7.x is the highest reachable line; it clears the activesupport
# advisories patched within the 6.1 series.
gem 'activesupport', '~> 6.1.7'
# concurrent-ruby >= 1.3.5 dropped its implicit `require 'logger'`, which
# activesupport 6.1.x relies on (raises "uninitialized constant Logger" at boot).
gem 'concurrent-ruby', '< 1.3.5'
