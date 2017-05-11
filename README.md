Cows and Code
==================
Pablo LaVegui Blog
------------------

From Jekyll-uno theme: https://github.com/joshgerdes/jekyll-uno

Build container
---------------

```
$ docker build -t jekyll-uno .
```

Run in local with Docker
------------------------

```
$ docker run -v "$PWD":/srv/jekyll -p "4000:4000" jekyll-uno jekyll serve --watch --drafts
```
or
```
$ docker-compose -f docker-compose.yml up jekyllblog
```

### Install and Test

1. Download or clone repo `git clone git@github.com:joshgerdes/jekyll-uno.git`
2. Enter the folder: `cd jekyll-uno/`
3. If you don't have bundler installed: `gem install bundler`
3. Install Ruby gems: `bundle install`
4. Start Jekyll server: `bundle exec jekyll serve --watch`

Access via: [http://localhost:4000/jekyll-uno/](http://localhost:4000/jekyll-uno/)

### Copyright and license

It is under [the MIT license](/LICENSE).
