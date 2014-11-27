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

Como se necesita una estructura de ficheros compatible con un proyecto de Django se restructura por completo el repositorio.

Después de dos tardes intentando hacerlo funcional, se ha podido hacer funcionar Shippable, pero para ahorrar tiempo no se ha documentado todo el proceso de acierto y error.

##Cómo funciona Shippable

Shippable proporciona una máquina virtual, la cual salta cuando se hace un push a una rama del repositorio virtual si dicha rama tiene un archivo __shippable.yml__ en la raiz. Dicha máquina virtual ejecuta las instrucciones que hay dentro del fichero.

Dentro de este fichero hay varios apartados:

- env: son las variables de enotno. Nosotros usamos el directorio de GAE (Google App Engine), el email del dueño de la aplicación y la contraseña encriptada para acceder a dicha cuenta.
- before install: son las instrucciones que se ejecutan antes de que se ejecuten las del apartado __install__. En nuestro caso se mira si está la SDK de GAE, y si no está se la descarga y la descomprime.
- install: se instalan los requerimientos necesarios para Django. En nuestro caso están en el fichero _requirements.txt_.
- before_script: después de ejecutar __install__ se ejecuta script, pero podemos preparar algunas cosas. En nuestro caso debido a que la máquina virtual y los servidores de GAE no tenían la hora sincronizada y nos daba fallo cambiamos la hora de la máquina virtual a la de Madrid. También creamos dos directorios que salen en el tutorial y que por ahora no se usan, pero que seguramnete se utilizarán.
- script: aquí se ejecutan las pruebas. En este momento solo ejecuta el fichero _test.py_ que solo tiene un _echo_ dentro, pero en un futuro contendrán las pruebas unitarias. Si las pruebas se pasan se ejecuta el apartado __after_success__.
- after_success: aquí indicamos las instrucciones a ejecutar si se pasan los test. En nuestro caso, simplemente es desplegar en App Engine, tal y como viene en los tutoriales. 





