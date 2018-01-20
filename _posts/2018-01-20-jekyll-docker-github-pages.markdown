---
layout: post
title: Jekyll + Docker + Github Pages
date: 2018/01/18
image: /images/posts/2018-01-20-jekyll-docker-github-pages/jekyll-docker.jpg
image-share: /images/posts/2018-01-20-jekyll-docker-github-pages/jekyll-docker.jpg
categories:
tags: [jekyll, docker, github, pages, blog]
---
[Jekyll][jekyll_homepage] es un generador de sitios estáticos, es tan sencillo de usar que en unos pocos pasos y de manera muy sencilla podremos generar nuestro sitio web / blog.

[Jekyll][jekyll_homepage] está escrito en [ruby][ruby_lang], pero como es algo tan sencillo de usar tampoco es necesario tener conocimientos del lenguaje, además veremos que si hay que tocar algo es muy intuitivo y sencillo.

Una de las mayores ventajas de [jekyll][jekyll_homepage] es que no se necesita una base de datos, se basa en documentos escritos en [markdown][markdown_wikipedia], es algo así como un intérprete, nosotros creamos los documentos y simplemente colocandolos en una carpeta está nuestro post publicado.

[GitHub Pages][githubpages_home] son páginas web alojadas y publicadas directamente a través de tu repositorio de [GitHub][github_home].

[Jekyll][jekyll_homepage] y [GitHub Pages][githubpages_home] casan tan bien porque precisamente [jekyll][jekyll_homepage] es el motor tras [GitHub Pages][githubpages_home] por lo tanto tendremos *casi* la total certeza de que lo que vemos en nuestro local cuando creamos una página / entrada es lo mismo que veremos cuando lo pongamos en producción, y cuando digo ponerlo en producción no me refiero a dificultosos procesos de deploy, sino simplemente subir nuestra rama actualizada al repositorio.

## Docker

Para quien aún no sepa que es [Docker][docker_home] puede leer este muy recomendable post introductorio de Javier Garzás: [¿Qué es Docker? ¿Para qué se utiliza? Explicado de forma sencilla](http://www.javiergarzas.com/2015/07/que-es-docker-sencillo.html), ahorramos verborrea y Javier lo explica genial.

Además en la [página web oficial][docker_home] tenemos una muy completa y de gran calidad documentación.

En este caso no va a haber deploy en producción con [Docker][docker_home], entonces ¿por qué utilizarlo? Para mi la razón con más peso es que no hay que instalar nada en tu equipo local, creo un contenedor, lo utilizo y si no me sirve lo borro y listo, mi máquina ni se entera.

## Repositorio en local y remoto

### Clonar un tema en local

Podríamos crear nuestro propio tema desde cero, pero no es el caso, queremos tener nuestro blog online lo más rápidamente posible, entonces, ¿que mejor manera que empezar con un tema ya hecho?

Podemos encontrar temas a lo largo y ancho de internet, pero estas dos páginas tienen muchos y además muy buenos, los hay gratuitos y de pago:

​	[https://jekyllthemes.io/][jekyllthemes_io]

​	[http://jekyllthemes.org/][jekyllthemes_org]

Una vez que hemos elegido el candidato ideal para nuestro blog, clonamos el repo en nuestra maquina local (doy por supuesto que sabemos utilizar o medio utilizar [Git][git_home], si no es así [aquí tienes una guía imprescindible][git_la_guia_sencilla])

`mkdir my-blog && cd my-blog`

`git clone git@github.com:joshgerdes/jekyll-uno.git .`

Borramos el repo original como remoto, no nos hace falta tenerlo enlazado, usaremos nuestro propio repositorio remoto:

`git remote remove origin`

Ahora que tenemos nuestro repo en local, el siguiente paso será crear nuestro repo remoto, al que subiremos nuestros cambios y el que estará en GitHub Pages para que todo el mundo pueda leer nuestros posts.

### Repositorio Remoto

Debemos crear en nuestra cuenta de [GitHub][github_home], desde la propia web, dentro de nuestra cuenta, un nuevo repositorio:

![boton crear repositorio en github](/images/posts/2018-01-20-jekyll-docker-github-pages/github-create-repo.jpg)

No vale ponerle cualquier nombre, el nombre del repositorio debe ser: `<nombre_de_usuario>.githug.io` en mi caso el nombre del repo es [PabloLaVegui.github.io](https://github.com/PabloLaVegui/PabloLaVegui.github.io).

Si la cuenta que tenemos en [GitHub][github_home] es una cuenta personal (es decir no es de empresa, de pago o similar) lo que veremos publicado en [GitHub Pages][githubpages_home] será la rama **master** ([más info aquí](https://help.github.com/articles/configuring-a-publishing-source-for-github-pages/)).

Una vez creado lo añadimos a nuestro repositorio local como remoto, desde la carpeta del proyecto ejecutamos:

`git remote add origin git@github.com:<nombre_de_usuario>/<nombre_de_usuario>.github.io.git`

Ahora si tenemos nuestro repo local enlazado con nuestro remoto y además [GitHub][github_home] lo reconocerá como nuestro repositorio para [GitHub Pages][githubpages_home].

## Trabajo en Local

### Configuración de Jekyll

La configuración del proyecto está centralizada en el archivo *_config.yml*, aquí es donde hay que ajustar los parámetros del blog.

Estos son algunos de los parámetros que se pueden configurar:

`vim _config.yml`

```yaml
# Site settings
title: 'My Blog'
description: 'Este es mi blog'
url: 'http://PabloLaVegui.github.io
baseurl: ''
locale: 'es_ES'

profile_image: '/images/profile.jpg'

author:
  name: 'Pablo LaVegui'
  twitter_username: pabloLaVegui
  github_username:  pablolavegui

defaults:
  -
    scope:
      path: ''
      type: 'posts'
    values:
        layout: 'post'

# Build settings
destination: _site
paginate: 10
permalink: /:year/:title/
markdown: kramdown
highlighter: rouge

kramdown:
  # use Github Flavored Markdown
  input: GFM
  # do not replace newlines by <br>s
  hard_wrap: false

gems: ['jekyll-paginate']
exclude: ['README.md', 'Gemfile', 'Gemfile.lock', 'Dockerfile', 'miblog.yml']
```

Se puede consultar la lista completa de parámetros configurables, así como para que sirven en la [sección correspondiente de la documentación oficial][jekyll_config_doc].

Cualquiera de estas variables se pueden luego utilizar en el blog, por ejemplo, para usar una variable de configuración dentro de una plantilla:

```html
<h1>{{ "{{ site.title " }}}}</h1>  
```

### Docker setup

Lo primero es crear un [Dockerfile][dockerfile_reference], que será el archivo que nos servirá como plantilla para construir el contenedor que contendrá nuestro Jekyll. Hay algunos temas (los menos) que lo incluyen, porque ya el autor del tema ha pensado en su uso en local con Docker.

Si el tema no contiene [**Dockerfile**][dockerfile_reference], lo creamos:

`vim Dockerfile`

<div class="language-dockerfile highlighter-rouge"><pre class="highlight"><code><span class="k">FROM</span> <span class="p">jekyll/jekyll:pages</span>

<span class="k">COPY</span> <span class="p">Gemfile* /srv/jekyll/</span>

<span class="k">WORKDIR</span> <span class="p">/srv/jekyll</span>

<span class="k">RUN</span> <span class="p">bundle config build.nokogiri --use-system-libraries && bundle install</span>

<span class="k">EXPOSE</span> <span class="p">4000</span>

<span class="k">CMD</span> <span class="p">jekyll serve --watch --drafts</span></code></pre></div>

Con este fichero indicamos a [Docker][docker_home] los pasos que tiene que seguir para construir la imagen:

- Partimos de la [imagen oficial de Jekyll][jekyll_dockerhub] pero en la versión ya optimizada para utilizar con [GitHub Pages][githubpages_home].
- Copiamos los archivos con las dependencias a la carpeta */srv/jekyll/* de dentro del contenedor.
- Indicamos que efectivamente la ruta */srv/jekyll/* va a ser la carpeta de trabajo dentro del contenedor.
- Instalamos las gemas dentro del contenedor. Le indicamos explicitamente la configuración para la gema *nokogiri* ya que es común que falle la instalación utilizando el contenedor oficial de Jekyll.
- Indicamos que el contenedor va a estar escuchando el puerto 4000
- Y por último decimos cual va a ser el comando que debe ejecutar el contenedor al arrancar: Arrancar el servidor, pero además que *escuche* los cambios en los ficheros (para no tener que reiniciar el contenedor cada vez que se hace un cambio) y además que muestre también los posts en borradores (explicado más adelante).

Con la plantilla lista, construimos la imagen:

`docker build -t miblog .`  (*Construir la imagen con nombre 'miblog' con el Dockerfile que esta en está misma carpeta*)

Una vez a terminado de construirse tenemos la imagen del contenedor lista para correr Jekyll con nuestro tema.

### Arrancar el contenedor

Para arrancar el contenedor:

`docker run --name myBlog -v "$PWD":/srv/jekyll -p "4000:4000" miblog`

- *--name myBlog* -> Nombre que va a tener el contenedor, en este caso *myBlog*.


- *-v "$PWD":/srv/jekyll* -> Mapeamos nuestra ruta actual (donde estamos colocados) con la carpeta */srv/jekyll* del contendor, que es la que dijimos que iba a ser la carpeta de trabajo al construir la imagen.
- *-p "4000:4000"* -> Mapeamos el puerto 4000 de nuestra máquina con el puerto 4000 del contenedor, que es el puerto que dijimos que expondría el contenedor.
- *miblog* -> el nombre de la imagen que usaremos para arrancar el contenedor.

Otra manera de arrancar el contenedor de manera más sencilla es creando un archivo de plantilla para **Docker Compose**. [Docker Compose][dockercompose_reference] es una herramienta para gestionar multiples containers mediante plantillas, por lo tanto mediante una plantilla podemos arrancar el contenedor y no tendremos que recordar todos los parámetros del comando *docker run*:

`vim miblog.yml`

```yaml
myBlog:
  container_name: myBlog
  image: jekylluno
  ports:
    - "4000:4000"
  volumes:
    - .:/srv/jekyll
```

Ahora ya no hay que recordar un enorme comando, simplemente con ejecutar el siguiente comando, el contendor estará en marcha y el servidor de [Jekyll][jekyll_homepage] en su interior corriendo:

`docker-compose -f miblog.yml up`

Para parar el contenedor:

`docker stop myBlog`

Después de la primear ejecución, en caso de que no se hagan modificaciones en el contenedor o en la imagen del mismo, para volver a arrancarlo simplemente con ejecutar el siguiente comando, el contenedor arrancará con toda la configuración seteada (el parámetro *-a* es para ver el log del servidor de jekyll, si no se pone, el contenedor arranca en segundo plano) :

`docker start -a myBlog`

### Ver el blog en local

Una vez que tenemos en marcha el contenedor con el servidor de [Jekyll][jekyll_homepage] corriendo en su interior, simplemente hay que abrir un navegador y navegar hacia http://localhost:4000

Veremos nuestro blog con la plantilla que hemos elegido y seguramente traerá algún post de ejemplo para hacernos una idea de como quedará.

### Borradores

Al construir el contenedor, le hemos dicho que cuando arranque el servidor de Jekyll sea con los parámetros *--watch --drafts*, ya se vió que la opción *--watch* era para que el servidor esté escuchando y los cambios que hagamos se vean automáticamente sin tener que reiniciar el servidor.

El caso que nos ocupa es la opción ***--drafts***. Con esta opción lo que le estamos diciendo al servidor es que muestre los posts que creemos dentro de la carpeta ***_drafts***, or lo tanto podemos meter en esta carpeta nuestros nuevos posts y trabajar sobre ellos y al mismo tiempo ir viendo en local como quedan hasta que los demos por terminados. Una vez que estén listos para publicar, los movemos a la carpeta ***_posts*** para que la publicación sea efectiva.

Es interesante mantener esta carpeta fuera del repositorio y añadir los posts al mismo sólo cuando las publicaciones estén listas, es decir cuando los movamos a la carpeta _posts. Para ello añadimos la siguiente linea al archivo .gitignore:

```tex
_drafts
```

Cuando estamos trabajando con un borrador el nombre que pongamos al archivo no es importante, el nombre es indiferente, pero cuando esté listo si que tiene que tener el formato de nombre de archivo de los posts: ***YYY-MM-DD-este-es-el-titulo.markdown***:

`mv _drafts/mi-primer-post.markdown _posts/2018-01-20-el-primer-post-guay.markdown`

También es importante tener en cuenta que los archivos *markdown* de los posts deben comenzar seteando algunas variables:

```markdown
---
title:  "Welcome to Jekyll!"
date:   2016/01/08
categories: [jekyll]
tags: [jekyll]
---
```

Estas variables como la configuración general de Jekyll se pueden utilizar dentro de las páginas y los templates:

```html
<h1>{{ "{{ post.title " }}}}</h1>  
```

##  Publicación

Una vez que en local lo tenemos todo a nuestro gusto, lo único que faltaría sería subir todo el código y los posts para verlo online.

Simplemente con hacer push de la rama master local a nuestro repo remoto tendremos todos los cambios online.

Las siguientes lineas espero que sean mejoradas (me refiero a que hay que ir haciendo pequeños commits y además con buenos nombres):

`git add . `

`git commit -m "Set config and publish first post" `

`git push origin master `

Para ver el resultado, en el navegador:

<u>https://nombre-de-usuario.github.io</u>

### Conclusión

No hay escusa para que no tener un blog online ya mismo, con herramientas muy potentes y sencillas de usar (al menos en principio y para el caso que nos ocupa) es facilísimo trabajar en local con el blog y luego una vez listo ponerlo online sin complicados procesos de deploy. 



[jekyll_homepage]: https://jekyllrb.com/	"Jekyll"
[ruby_lang]: https://www.ruby-lang.org/es/	"Ruby"
[markdown_wikipedia]: https://es.wikipedia.org/wiki/Markdown	"Markdown Wikipedia"
[githubpages_home]: https://pages.github.com/	"GitHub Pages"
[github_home]: https://github.com/	"GitHub"
[docker_home]: https://www.docker.com/	"Docker"
[git_home]: https://git-scm.com/	"Git"
[git_la_guia_sencilla]: http://rogerdudler.github.io/git-guide/index.es.html	"Git - La guia definitiva"
[jekyllthemes_io]: https://jekyllthemes.io/	"Jekyllthemes.io"
[jekyllthemes_org]: http://jekyllthemes.org/	"Jekyllthemes.org"
[dockerfile_reference]: https://docs.docker.com/engine/reference/builder/	"Dockerfile Reference"
[jekyll_dockerhub]: https://hub.docker.com/r/jekyll/jekyll/	"Imagen oficial de Jekyll en Docker Hub"
[dockercompose_reference]: https://docs.docker.com/compose/	"Docker Compose referene"
[jekyll_config_doc]: https://jekyllrb.com/docs/configuration/	"Jekyll config doc"
