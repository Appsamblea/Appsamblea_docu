#Sistema de despliegue continuo

El sistema de despliegue continuo tiene que subir a App Engine todo el contenido de la aplicación en Django cada vez que se haga un pull a la rama Master.

##Opción 1: Conectar un repositorio externo desde Google Developers Console

Accediendo a la Google Developers Console podemos conectar un repositorio externo, de forma que se duplica todo lo que haya en dicho repositorio.

Se prueba a conectar este repositorio para ver qué ocurre. Sin embargo no conecta nada, debido a que solo se puede tener código fuente dentro del repositorio, y este contiene más cosas.

##Opción 2: [Shippable](https://www.shippable.com/)

Se comentó en clase esta opción para despliegue continuo. Por ello tras no poder hacerlo con Google Developers Console se intenta con esta opción.

Primero pide permisos para acceder a la cuenta de Github. A continuación se le dice con qué repositorio trabajar.

Sin embargo hace falta cierta preparación del repositorio. Por ello se sigue [este tutorial](http://docs.shippable.com/en/latest/start.html).

Primero se crea un fichero shippable.yml, utilizando [este](https://github.com/shippableSamples/sample-django-cloudsql-appengine/blob/master/shippable.yml) como ejemplo.

Sin éxito se encuentran varios ejemplos y se crea un pequeño fichero que ejecutará test.py.

Como se necesita una estructura de ficheros compatible con un proyecto de Django se reestructura por completo el repositorio.

Después de dos tardes intentando hacerlo funcional, se ha podido hacer funcionar Shippable, pero para ahorrar tiempo no se ha documentado todo el proceso de acierto y error.

##Cómo funciona Shippable

Shippable proporciona una máquina virtual, la cual salta cuando se hace un push a una rama del repositorio virtual si dicha rama tiene un archivo __shippable.yml__ en la raíz. Dicha máquina virtual ejecuta las instrucciones que hay dentro del fichero.

Dentro de este fichero hay varios apartados:

- env: son las variables de enotno. Nosotros usamos el directorio de GAE (Google App Engine), el email del dueño de la aplicación y la contraseña encriptada para acceder a dicha cuenta.
- before install: son las instrucciones que se ejecutan antes de que se ejecuten las del apartado __install__. En nuestro caso se mira si está la SDK de GAE, y si no está se la descarga y la descomprime.
- install: se instalan los requerimientos necesarios para Django. En nuestro caso están en el fichero _requirements.txt_.
- before_script: después de ejecutar __install__ se ejecuta script, pero podemos preparar algunas cosas. En nuestro caso debido a que la máquina virtual y los servidores de GAE no tenían la hora sincronizada y nos daba fallo cambiamos la hora de la máquina virtual a la de Madrid. También creamos dos directorios que salen en el tutorial y que por ahora no se usan, pero que seguramente se utilizarán.
- script: aquí se ejecutan las pruebas. En este momento solo ejecuta el fichero _test.py_ que solo tiene un _echo_ dentro, pero en un futuro contendrán las pruebas unitarias. Si las pruebas se pasan se ejecuta el apartado __after_success__.
- after_success: aquí indicamos las instrucciones a ejecutar si se pasan los test. En nuestro caso, simplemente es desplegar en App Engine, tal y como viene en los tutoriales. 


##Travis CI y Snap CI

Para los 2 se ha usado para hacer pruebas el proyecto que se encuentra [aquí](https://github.com/silt99/TestDjangoAE). Este programa de ejemplo se ha realizado siguiendo [este tutorial](http://django-appengine.com/post/37778427717/create-a-simple-guestbook-application).

##Opción 3: [Travis CI](https://travis-ci.org/)
Muy similar a Shippable, este también fue comentado en clase. Trabaja con máquinas virtuales de linux, trae una serie de recursos y librerías preinstaladas para varios lenguajes y permite instalar otras si fuese necesario.

Su configuración se hace con un archivo yml idéntico al de shippable, solo que llamado .travis.yml. 

Su respuesta ha sido algo lenta en las pruebas que he hecho (parece ser que están teniendo algunos problemas con la red), pero aun así ha funcionado bien. Además, hay mucha documentación disponible y es bastante fácil de entender.

Para usar shippable con un proyecto:

1. En la página de tus repositorios, activa aquel que quieras usar.
	
2. Añadir el archivo .travis.yml al repositorio. Como es casi idéntico al de shippable, voy a omitir su explicación. Lo único distinto e que, para emplear una variable encriptada, esta ha de encriptarse con una utilidad que proporciona travis. [Aquí](http://docs.travis-ci.com/user/encryption-keys/) puede encontrar una explicación más detallada de como hacerlo.
	
3. Validar el archivo .travis.yml con [esta utilidad](http://lint.travis-ci.org/).
	
4. Hacer un commit push en el repositorio. Id a la página de settings del repositorio, seleccionar "Webhooks & Services". Bajo "Services", selecciona "Travis CI" y dadle al botón "Test service".

##Opción 4: [Snap CI](https://snap-ci.com/)

Aunque no se comentó en clase,se encontró la página y se decidió probar este sistema de integración continua.

Es muy similar a travis: También trabaja con máquinas virtuales de linux, trae una serie de recursos y librerías preinstaladas para varios lenguajes y permite instalar otras si fuese necesario.

La principal diferencia con travis es que no requiere configuración mediante un archivo yml, si no que esta se hace mediante una interfaz web que proporciona con pipelines. Estos están formados por una serie de fases que se ejecutan secuencialmente, en cada una de las cuales podemos introducir los comandos que queremos que se ejecuten; de forma que si una fase falla no se continua con la siguiente.
He comprobado, así mismo, que responde mucho más rápido que travis.

Como añadir un proyecto Django y App Engine para su integración continua:

1. Añadir repositorio: en el menú Builds, darle a +Repository. En el nuevo menú, darle a +Add del repositorio que se quiere añadir.

2. Configurar la construcción:

	* Se desplegará un menú nuevo donde fijar las opciones de construcción. 
	* En la parte de Languages and Database, seleccionar none para todas las opciones presente salvo para python, que se pondrá como 2.7.
	* Darle a +ADD STAGE.
	* Nombradla como PythonRequirements.
	* En Environment Variables añadir GAE_DIR /tmp/gae.
	* En Artifacts añadir $GAE_DIR.
	* En Commands to be executed in this stage, añadir:
      * test -e $GAE_DIR || (mkdir -p $GAE_DIR && wget https://storage.googleapis.com/appengine-sdks/featured/google_appengine_1.9.15.zip -q -O /tmp/gae.zip && unzip -q /tmp/gae.zip -d $GAE_DIR)
	   * pip install -r requirements.txt
	   * python test.py
	* Darle a +ADD STAGE.
	* Nombradla como Deployment.
	* En Environment Variables añadir GAE_DIR /tmp/gae y APP_DIR . .
	* Aunque permite añadir variables encriptadas, por algún motivo no funciona con app engine, así que los comandos para subir a app engine y la contraseña han de introducirse en un archivo seguro:
	* En Secure Files, añadir scrip_aux con permisos para ejecutarse.
	* Añadir en el archivo:
	```
	#!/bin/sh
	export GAE_OAUTH=<your_oauth_token>
	python $GAE_DIR/appcfg.py --oauth2_refresh_token=$GAE_OAUTH update $APP_DIR
  ```
  * En Commands to be executed in this stage, añadir: 
	    * test -e $GAE_DIR || (mkdir -p $GAE_DIR && wget https://storage.googleapis.com/appengine-sdks/featured/google_appengine_1.9.15.zip -q -O /tmp/gae.zip && unzip -q /tmp/gae.zip -d $GAE_DIR)
	    * /var/snap-ci/repo/scrip_aux

3. Darle a Save.

Con esto, ya habremos realizado el primer test.
	
Como hemos visto, funciona con un pipeline de pruebas, de forma que si no se pasan las 1º fases las siguientes no se ejecutan. Este hecho puede usarse, además de para separar en grupos los test y descubrir más fácilmente donde está el error, para iniciar el despliegue de la aplicación solo si pasa todos los test.
