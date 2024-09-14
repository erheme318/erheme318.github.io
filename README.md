# Personal blog

[https://erkhem.org/](https://erkhem.org/)

## Deploying to the DO App Platform ##

Click this button to deploy the app to the DigitalOcean App Platform.

 [![Deploy to DO](https://mp-assets1.sfo2.digitaloceanspaces.com/deploy-to-do/do-btn-blue.svg)](https://cloud.digitalocean.com/apps/new?repo=https://github.com/erheme318/blog/tree/master)

## Troubleshooting

Installing Jekyll on MacbookPro with M1 chip

```
brew install chruby
brew install ruby-install
ruby-install ruby-2.7.2 -- --with-openssl-dir="$(brew --prefix openssl)"
echo "chruby ruby-2.7.2" >> ~/.zshrc
source /opt/homebrew/opt/chruby/share/chruby/chruby.sh
chruby ruby-2.7.2
gem update --system
gem update bundler
sysctl -n hw.ncpu //n
bundle config --global jobs 7 // n-1 output of the above command
gem install jekyll
````

ref: https://github.com/monfresh/laptop/blob/master/mac
