---
layout: post
title:  "Website update"
lang: en
tags: [en]
---

Reworked the website from React/Vite to Jekyll with [Chirpy theme](https://github.com/cotes2020/jekyll-theme-chirpy/), mainly following this tutorial [https://technotim.live/posts/jekyll-docs-site/](https://technotim.live/posts/jekyll-docs-site/).
The old website was a quick fix with React/Vite from the time when I was coding a frontend with the same Vite template.  The new website is built with Jekyll and still hosted on GitHub Pages and use CD with GitHub Actions. 
Content in Jekyll is easier to maintain because it uses Markdown language for the content markup.


#### Steps to reproduce (on Linux)

- install the dependencies
```bash
sudo apt install ruby-full build-essential zlib1g-dev git
```
- modify the variables for RubyGems packages to be installed in the home directory of the user
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

- create repo from template [https://github.com/cotes2020/chirpy-starter](https://github.com/cotes2020/chirpy-starter) , name the repo `username.github.io`, set the pages settings to use GitHub actions
- clone the repo
- edit `_config.yml` to change the title, description, author, baseurl, url, and social media links
- run `bundle` to install the dependencies
- run `bundle exec jekyll serve` to start the local server
- add content to `_tabs/` and `_posts/`, put images to `assets/images/`
- copy favicon files to `assets/img/favicons/`
- push the changes to the repo

#### Local deployment

Run 
```bash
JEKYLL_ENV=production bundle exec jekyll b
```
to build the site in `_site` folder. And now it is possible to serve it with NGINX or any other webserver.