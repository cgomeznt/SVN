Configurar un servidor SUBVERSION BASICO
=========================================

Buscamos:: 

	# yum search subversion

instalamos::

	# apt-get install subversion

Creamos tantos repositorio como uno quiera. Por defecto, subversion es en /srv/svn, pero podemos cambiarlo.::

	# mkdir -p /opt/svn/repo

	# svnadmin create /opt/svn/repo/sopapli
	# svnadmin create /opt/svn/repo/sopunix
	# svnadmin create /opt/svn/repo/sopwin

Sguridad de los repositorio, es decir, que usuarios podran acceder. Editamos  /opt/svn/repo/sopapli/conf/svnserve.conf
Si necesitas un nivel de seguridad mayor ver manual de SVN.::

	[general]
	anon-access = none
	auth-access = write
	password-db = passwd
	realm = My First Repository

Con esa modificación le decimos al servidor svn que los usuarios anónimos no tienen acceso, los usuarios autenticados pueden leer/escribir y que la base de datos empleada para las contraseñas es el archivo passwd.

Configuración de los usuarios y las contraseñas. Editamos el archivo /srv/svn/test/conf/passwd::

	[users]
	carlos = clave_carlos
	pedro = clave_pedro

Ya iniciamos el servicio de subversion::
	
	# svnserve -d -r /opt/svn/repo

El argumento "-r", esto se usa para esconder la ruta real de nuestro repositorio en la url pública del repositorio. sin la -r los usuarios para hacer una tarea común como el 'checkout' deben hacer svn checkout svn://hostname/opt/svn/repo/sopapli, pero si incluímos la -r solo se coloca el repositorio svn checkout svn://hostname/sopapli

Debemos verlo que abriera el puerto 3690::

	# netstat -natp

Y tambien en memoria:

	# ps -ef | grep svn
	root	1789	1	10:47	?	00:00:00	svnserve -d -r /opt/svn/repo

Creamos la carpeta de nuestro proyecto (con la estructura trunk, branch y tags)::

	# mkdir -p /opt/svn/repo/sopapli/trunk
	# mkdir -p /opt/svn/repo/sopapli/branch
	# mkdir -p /opt/svn/repo/sopapli/tags


Vamos a crear unos archivos de prueba y para pasarlos al repositorio svn con import , para ir poblando de data.::

	$ mkdir /tmp/prueba
	$ cd /tmp/prueba
	$ echo "Hola Mundo" > test
	$ echo "Otro Texto" > test2

Importando
-----------

Con el cliente svn ejecutamos la siguiente instrucción::

	$ svn import /tmp/prueba/* svn://hostname/test -m "Initial Commit"
	Adding /tmp/prueba/test1
	Adding /tmp/prueba/test2

	Commited revision 1.

Comprobando
------------

Ahora pasate a otro directorio y veamos como hacer el checkout::

	$ cd /home/usuario
	$ svn co svn://hostname/sopapli

Editar y aplicar
----------------

Modificamos un archivo y subimos el cambio con el commit::

	$ cd sopapl
	$ vi test1
	$ svn commit -m "Modificar un archivo"

Crea otro directorio en otra ruta y vuelve hacer el checkout y commit de un archivo y retorna nuevamente a este directorio para hacer pruebas de actualizacion.

Para verificar el estado de tus archivos de trabajo::

	$ svn status -u

Para actualizar tu "copia" del repositorio ejecuta::

	$ svn update

Adicionando elementos
-----------------------------------
::
	$ vi test3
	$ svn add test3
	$ svn commit -m "Agregar un archivo"

Revisión de la bitácora
------------------------
::
	$ svn log svn://hostname/sopapli

Nos mostrara la información detallada de la revisión 5, incluyendo los archivos afectados::

	$ svn log -v -r5 svn://hostname/sopapli

Revisar la información de un archivo específico.
------------------------------------------------
::
	$ svn info test3
	$ svn log test3

Borrando elementos
-----------------------------------
::
	$ svn delete test3
	$ svn commit -m "Borrado de un archivo"

Revertir
---------------

Revertir cambios de un directorio::

	$ svn log svn://hostname/sopapl/mydir
	$ svn co -r 3 svn://hostname/sopapl/mydir
	$ svn commit -m "Revirtiendo una version"

Revertir cambios: Hemos realizado cambios locales a los cuales no le hemos realizado commit alguno, entonces deseamos revertir dichos cambios::

 $ svn revert /ruta/sistema/archivo.php

Para revertir todos los cambios en el proyecto recursivamente::

	$ svn revert -R /ruta/sistema

Revertir a una revisión anterior Supongamos que estamos en la revisión 105 y queremos devolvernos a la versión 104, típicamente alguien actualizo en producción un sistema y algo no funciono bien.

ingresamos al proyecto versionado y indicamos revisión deseada en este caso 104.

	$ svn merge -rHEAD:104 test3
	$ svn commit -m "Revirtiendo una version"

Realizar un export: 
-------------------
Se realiza un export cuando necesitamos el proyecto que tenemos versionado sin la información del control de versiones, esta acción limpia los directorios ocultos .svn que se encuentran en cada directorio del proyecto. Esto es muy util a la hora de distribuir el proyecto o para colgarlo en producción::

	$ svn export sistema-versionado sistemalimpio
 
::

	$svn export svn://dominiosvn.com/sistema-versionado/trunk sistemalimpio




