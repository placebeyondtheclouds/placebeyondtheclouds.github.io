# My personal webpage

Static site generated with Jekyll and hosted on GitHub Pages. Markdown language for the content markup. Continuous deployment [CD] with GitHub Actions. The site is built with the [Chirpy theme][chirpy].

## Steps to reproduce (on Linux)

- install the dependencies
```bash
sudo apt install ruby-full build-essential zlib1g-dev git
```
- modify path for RubyGems packages to be installed in the home directory of the user
```bash
echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```
- install Jekyll and Bundler
```bash
gem install jekyll bundler
```

- create repo from template https://github.com/cotes2020/chirpy-starter , name the repo `username.github.io`, set the pages settings to use GitHub actions
- clone the repo
- edit `_config.yml` to change the title, description, author, baseurl, url, and social media links
- run `bundle` to install the dependencies
- run `bundle exec jekyll serve` to start the local server
- add content to `_tabs/` and `_posts/`, put images to `assets/images/`
- copy favicon files to `assets/img/favicons/
- push the changes to the repo

## Refetences

- https://technotim.live/posts/jekyll-docs-site/
- https://github.com/cotes2020/jekyll-theme-chirpy/wiki

## License

This work is published under [MIT][mit] License.

[gem]: https://rubygems.org/gems/jekyll-theme-chirpy
[chirpy]: https://github.com/cotes2020/jekyll-theme-chirpy/
[CD]: https://en.wikipedia.org/wiki/Continuous_deployment
[mit]: https://github.com/cotes2020/chirpy-starter/blob/master/LICENSE
