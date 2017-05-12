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

Access via: [http://localhost:4000/jekyll-uno/](http://localhost:4000/jekyll-uno/)

### Copyright and license

It is under [the MIT license](/LICENSE).
