#Test

Para la realización de los test, consulté [este tutorial](http://toastdriven.com/blog/2011/apr/10/guide-to-testing-in-django/) sobre test con Django, si bien el código que se proporciona no funcionará, ya que usamos app engine para la base de datos, por lo que tenemos que acudir a estos dos tutoriales para el testeo de aplicaciones de app engine: [unit test](https://cloud.google.com/appengine/docs/python/tools/localunittesting#Python_Introducing_the_Python_testing_utilities) y [handler testing](https://cloud.google.com/appengine/docs/python/tools/handlertesting). Cabe mencionar que hay un bug en el sdk de Gogle App Engine cuando se usa con python < 2.7.8: Cuando se importa testbed, que es el paquete que se necesita para los test, se da un error. Para solucionarlo, ir a la carpeta google_appengine/lib/fancy_urllib/fancy_urllib y copiar el contenido en google_appengine/lib/fancy_urllib sobreescribiendo. Este bug es complicado de reparar, no por el proceso (simple, como se puede ver), sino porque a penas hay documentación de él.

En resumen: 
* Para realizar todos los test de las aplicaciones instaladas en Django, hay que emplear el siguiente comando: python manage.py test.
* Si quieres que solo se ejecuten los test de una aplicación, emplea: python manage.py test <app_name>.
* Para añadir test propios a una aplicación que hayas creado, añade en su carpeta el archivo tests.py, donde has de poner los test.
* Los test han de situarse en una clase que herede de unittest.TestCase:
	* Toda actividad previa a los test ha de situarse en el método setUp(self). En el caso una aplicación app engine, hay que poner al menos:
		```python
			# Primero, crear una instancia de la clase Testbed.
			self.testbed = testbed.Testbed()
			# Activar testbed, que prepara los stub de los servicios para su uso.
			self.testbed.activate()
			# Luego, declara que stubs de servicios quieres usar.
			...
		```
	* Toda actividad posterior a los test ha de ir en un método tearDown(self). En el caso una aplicación app engine, hay que poner al menos:
		```python
			self.testbed.deactivate()
		```		
	* Los test que queramos que se ejecute, deben de incluirse en métodos de esta clase que empiecen por test.

