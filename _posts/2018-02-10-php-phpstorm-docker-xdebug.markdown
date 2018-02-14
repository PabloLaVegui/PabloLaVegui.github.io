---
layout: post
title: Entorno completo para trabajar con PHP, PHP + Xdebug + PHPUnit + PHPStorm + Docker
description: Como montar un entorno para desarrollar en local con PHP, XDebug, testear con PHPUnit, usar Docker para desarrollar en local y PHPStorm como IDE. Configuración de PHPStorm para que funcione todo.
date: '2018-02-10'
image: /images/posts/php-docker-xdebug-rename/docker_php.png
image-share: /images/posts/php-docker-xdebug-rename/docker_php.png
categories:
tags: [php, phpstorm, xdebug, docker, phpunit]
---
El objetivo es montar un entorno de desarrollo en local con PHP, PHPUnit, Xdebug y Docker.
Ya hemos hablado en [el posts anterior](http://cowsandcode.com/2018/jekyll-docker-github-pages/) de las ventajas de trabajar con Docker, entre otras la de que no hay que instalar nada en nuestra máquina, salvo Docker claro.

## Estructura básica del proyecto

La estructura de carpetas de nuestro proyecto es la siguiente:

```
.
├── public
├── src
└── test
```

La estructura es muy sencilla:

*/public* será el punto de entrada de la aplicación, es donde estará el archivo *index.php*

*/src* contendrá todo la lógica de la aplicación.

Y evidentemente */test* contendrá los tests.

## Construir la imagen de PHP

Lo primero que vamos a hacer es construir la imagen a partir de la cual haremos el contenedor con el que trabajará la aplicación.
Un Dockerfile es algo así como una receta para generar una nueva imagen, lo bueno es que podemos usar una imagen existente y añadir lo que nos interese para que se adecúe a nuestras necesidades.
En nuestro caso es la [imagen original de **PHP** con **Apache**](https://hub.docker.com/_/php/) y le añadimos la extensión [**Xdebug.**](https://xdebug.org/)

*Dockerfile*
<div class="language-dockerfile highlighter-rouge"><pre class="highlight"><code><span class="k">FROM</span> <span class="p">FROM php:7.2.1-apache</span>

<span class="k">RUN</span> <span class="p">apt-get update && \</span>
<span class="p">    apt-get install git unzip -y</span>

<span class="k">RUN</span> <span class="p">curl -fsSL 'https://xdebug.org/files/xdebug-2.6.0.tgz' -o xdebug.tar.gz \</span>
<span class="p">    && mkdir -p xdebug \</span>
<span class="p">    && tar -xf xdebug.tar.gz -C xdebug --strip-components=1 \</span>
<span class="p">    && rm xdebug.tar.gz \</span>
<span class="p">    && ( \</span>
<span class="p">    cd xdebug \</span>
<span class="p">    && phpize \</span>
<span class="p">    && ./configure --enable-xdebug \</span>
<span class="p">    && make -j$(nproc) \</span>
<span class="p">    && make install \</span>
<span class="p">    ) \</span>
<span class="p">    && rm -r xdebug \</span>
<span class="p">    && docker-php-ext-enable xdebug</span>

<span class="k">RUN</span> <span class="p">echo 'xdebug.remote_host=192.168.1.131' >> /usr/local/etc/php/php.ini</span>
<span class="k">RUN</span> <span class="p">echo 'xdebug.remote_enable=1' >> /usr/local/etc/php/php.ini</span>
<span class="k">RUN</span> <span class="p">echo 'xdebug.remote_port=9001' >> /usr/local/etc/php/php.ini</span>

<span class="k">RUN </span><span class="p">sed -i -e 's/DocumentRoot \/var\/www\/html/DocumentRoot \/var\/www\/html\/public/g' /etc/apache2/sites-available/000-default.conf</span>

<span class="k">RUN</span> <span class="p">php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" && \</span>
<span class="p">    php composer-setup.php && \</span>
<span class="p">    php -r "unlink('composer-setup.php');" && \</span>
<span class="p">    mv composer.phar /usr/local/bin/composer</span>

<span class="k">VOLUME</span> <span class="p">["/var/www/html"]</span>
<span class="k">WORKDIR</span> <span class="p">/var/www/html</span>
</code></pre></div>

Que hace este Dockerfile:

- Añade la extensión Xdebug.
- Configura Xdebug.
- Configura el *virtual host* en Apache para que la carpeta de acceso a la aplicación sea */public*.

Xdebug lo configuramos en el puerto 9001 para evitar posibles conflictos con nuestra máquina (es un puerto que es fácil que esté siendo utilizado por otra aplicación).

Además hay que decirle cual es la ip del **host**, es decir nuestra máquina para que el contenedor sepa como acceder a ella, eso es porque PHPStorm está en nuestra máquina y Xdebug desde dentro del contenedor se va a querer conectar a ella (es una definición simple pero nos da la idea de lo que está pasando).

Para saber cual es la ip:

`$ ifconfig`

![resultado de ifconfig](/images/posts/php-docker-xdebug-rename/ifconfig.png)

Construir el contenedor:

`$ docker build -t php-7.2.1-xdebug .`

*-t nombre_del_contenedor*

## Composer

Añadimos el archivo *composer.json* en la raiz del proyecto con los namespaces de la apliación y la dependencia de PHPUnit:

```json
{
  "require-dev": {
    "phpunit/phpunit": "^7.0"
  },
  "autoload": {
    "psr-4": {
      "MyApp\\": "src/",
    }
  },
  "autoload-dev": {
    "psr-4": {
      "MyApp\\Test\\": "test/"
    }
  }
}
```

Ejecutamos composer:

`$ docker run -v "$PWD":/var/www/html php-7.2.1-xdebug composer install`

## El Código

Lo primero vamos a configurar PHPStorm para que añada los namespaces de las clases directamente al crear la clase en función de la carpeta en la que esté el archivo, asociamos los namespaces igual que están en *composer.json*, es decir, la carpeta */src* es el punto de partida del namespace *Myapp* y la carpeta */test* será la de *MyApp\Test*.

Vamos a *Prefences > Directories*

![directories](/images/posts/php-docker-xdebug-rename/directories.png)

Añadimos una clase y un test de prueba:

*src/Serenity.php*
```php
<?php
namespace MyApp;

/**
 * Class Serenity
 */
class Serenity
{
    /** @var array */
    private $crew = [];

    /**
     * @param $crewMember
     */
    public function addCrew($crewMember)
    {
        $this->crew[] = $crewMember;
    }

    /**
     * @return array
     */
    public function getCrew()
    {
        return $this->crew;
    }
}
```

*test/SerenityTest.php*
```php
<?php
namespace MyApp\Test;

use MyApp\Serenity;
use PHPUnit\Framework\TestCase;

/**
 * Class SerenityTest
 */
class SerenityTest extends TestCase
{
    /** @test */
    public function should_add_crew_members()
    {
        $river = 'River Tam';

        $serenity = new Serenity();

        $serenity->addCrew($river);

        $this->assertTrue(in_array($river, $serenity->getCrew()));
    }
}
```

Añadimos la configuración de PHPUnit:

*phpunit.xml*
```xml
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         bootstrap="./vendor/autoload.php">
    <testsuites>
        <testsuite name="Unit">
            <directory>./test</directory>
        </testsuite>
    </testsuites>
</phpunit>
```


## Configuraración del intérprete de PHP

*Preferences > Languages & Frameworks > PHP > CLI Interpreter ... > + > From Docker, Vagrant...*
![configure remote php interpreter](/images/posts/php-docker-xdebug-rename/configure_remote_php_interpreter.png)

Ahora la configuración CLI Interpretes queda así:
![cli interpreters](/images/posts/php-docker-xdebug-rename/cli_interpreters.png)

Ahora ya casi está toda la configuración del intérprete, sólo falta poner la ruta correcta en *Path mappings* y *Docker container*, para ello pinchamos en ... de *Docker container*
La configuración de las rutas debe ser **Container path**: */var/wwww/html* (la del contendor), **Host path**: */la/ruta/de/nuestra/aplicación*
![edit docker container settings](/images/posts/php-docker-xdebug-rename/edit_docker_container_settings.png)

Con lo cual la configuración final del interpréte habrá quedado así:
![interpreter config](/images/posts/php-docker-xdebug-rename/interpreter_config.png)

## Configuración de PHPUnit

*Preferences > PHP > Test Frameworks*
Añadir el intérprete de nuestro contendor:

*+ > PHPUnit by Remote Interpreter*
![phpunit by remote interpreter](/images/posts/php-docker-xdebug-rename/phpunit_by_remote_interpreter.png)

La configuración final de PHPUnit queda así:
![php test frameworks](/images/posts/php-docker-xdebug-rename/php_testframeworks.png)

Para **ejecutar los tests** hacer click derecho sobre el archivo *phpunit.xml* y elegir *Run 'phpunit.xml'*

El debugger también debería funcionar, para asegurarnos ponemos un punto de ruptura en el test y la ejecución debería pararse, eso si hay que tener en cuenta que hay que ejecutar los tests (o el test en cuestión) en modo debug:

*click derecho > Debug 'phpunit.xml'*

![test debug](/images/posts/php-docker-xdebug-rename/test_debug.png)

## Puesta en marcha de la aplicación. Crear un servidor de debug

Para que funcione el debug durante la ejecución normal de la aplicación hay que crear un servidor intermedio, que comunique el servidor remoto con la máquina local, en nuestro caso el contendor de Docker y el host, osea nuestra máquina local.

*Preferences > Languages & Frameworks > PHP > Servers*
![php servers](/images/posts/php-docker-xdebug-rename/php_servers.png)

## Dándole a la manivela. Arrancar el contenedor

Es hora de arrancar el contenedor y ver que todo funciona:

`$ docker run --name my-app -v "$PWD":/var/www/html -p "8080:80" -e XDEBUG_CONFIG="remote_host=192.168.1.131" -e PHP_IDE_CONFIG="serverName=PHPServer" php-7.2.1-xdebug`

- *--name my-app* -> Nombre que va a tener el contenedor.
- *-v "$PWD":/var/www/html* -> Mapeamos la carpeta de nuestro proyecto con la carpeta que sirve el contenedor.
- *-p "8080:80" -> Mapeamos el puerto 80 del contenedor con el 8080 de nuestra máquina local.
- *-e XDEBUG_CONFIG="remote_host=192.168.1.131"* -> Seteamos en el contenedor esta variable de entorno, la ip es la de nuestra máquina local desde el container.
- *-e PHP_IDE_CONFIG="serverName=PHPServer"* -> Seteamos como variable de entorno el nombre del servidor de debug que hemos creado antes.
- *php-7.2.1-xdebug* -> Nombre de la imagen que vamos a usar para crear el contenedor.

Una vez que lo hemos arrancado por primera vez con todos esos parámetros, si después lo paramos, para volver a arrancarlo bastará con ejecutar:

`$ docker start my-app`

El comando es un poco loco, así que mejor tener un archivo para ejecutarlo con [docker-compose](https://docs.docker.com/compose/)

*docker-compose.yml*
```yaml
myapp:
  container_name: my-app
  image: php-7.2.1-xdebug
  ports:
    - "8080:80"
  environment:
    XDEBUG_CONFIG: "remote_host=192.168.1.131"
    PHP_IDE_CONFIG: "serverName=PHPServer"
  volumes:
    - .:/var/www/html
```

Ahora todo es más sencillo:

`$ docker-compose -f docker-compose.yml up`

También lo podemos configurar directamente desde PHPStorm.

Abrimos la ventana de Docker, y a la izquierda veremos una lista con los containers y las imagenes que tenemos en nuestra máquina.
Pinchamos con el botón derecho sobre la imagen que hemos creado y elegimos la opción *Create container > Create...*
![create docker configuration](/images/posts/php-docker-xdebug-rename/create_docker_configuration.png)

En cuanto le demos al botón *Run* nuestro contenedor se pondrá en marcha.

Creamos el fichero *index.php* dentro de la carpeta */public*. *OJO:* No haga esto en su casa, es un muy mal código y sólo tiene intenciones pedagógicas:
```php
<?php

include_once __DIR__ . '/../vendor/autoload.php';

use MyApp\Serenity;

$serenity = new Serenity();
$serenity->addCrew('River Tam');
$serenity->addCrew('Simon Tam');
$serenity->addCrew('Derrial Book');
?>

<h1>This is Serenity</h1>
<h2>These are the new crew members:</h2>

<?php foreach ($serenity->getCrew() as $newCrewMember) { ?>
    <p><?php echo $newCrewMember; ?></p>
<?php } ?>
```

Una vez que el contenedor está en marcha, si apuntamos el navegador a [http://localhost:8080] tendremos que ver la página *index.php* de la carpeta */public*.

![serenity web page](/images/posts/php-docker-xdebug-rename/serenity_window.png)

Si ponemos un punto de interrupción y ponemos a escuchar a PHPStorm

![start debugger](/images/posts/php-docker-xdebug-rename/start_debugger.png)

veremos que al recargar la página la ejecución se para en el punto.

![debug screenshot](/images/posts/php-docker-xdebug-rename/debug_screenshot.png)

## Conclusión

Conseguir un entorno para desarrollar con PHP basado en Docker no es tan difícil, y nos aporta todas las ventajas que nos da Docker a la hora de desarrollar, entornos portables, misma configuración en todas las máquinas, etc, todo ventajas oiga.
Adelante, prueba y disfruta.

Sí, es cierto, nos falta una parte muy importante para desarrollar, la **base de datos**, pero no se pongan nerviosos, lo abordaremos en una próxima entrega.
