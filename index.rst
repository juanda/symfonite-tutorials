.. Tutoriales Symfonite documentation master file, created by
   sphinx-quickstart on Mon Feb 20 14:04:40 2012.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Tutoriales Symfonite
====================

Con el fin de ilustrar la mayor parte de las funcionalidades del framework
symfonite de forma progresiva, hemos divido el tutorial en tres partes. En la
primera nos centramos en el despliegue del *framework* y en la generación del
esqueleto de las aplicaciones. En la segunda se muestra el funcionamiento del
*Identity Provider SAML* y el Authentication Server *PAPI* que vienen incorporado en el framework. También mostramos en esta parte como las aplicaciones symfonite pueden actuar como *Service Provider SAML* y/o *Point Of Access PAPI*, tanto del *IdP/AS* interno como de otros externos al sistema. Por último, en la
tercera parte se realiza un estudio más detallado del framework mediante la
construcción de una plataforma de aplicaciones web para una hipotética institución educativa.

 .. note::

   Para poder desplegar el framework es necesario contar con un sistema gestor
   de bases de datos MySql.

   Aunque symfony viene equipado con una capa de acceso a datos que admite
   distintos tipos de bases de datos (MySql, Postgresql, Oracle, sqlite), el
   framework que presentamos en este tutorial únicamente ha sido probado con
   MySql.

   Por otro lado, si este MySql está instalado en localhost y existe una base
   de datos denominada symonite a la que se puede acceder con permisos de
   SELECT, INSERT, UPDATE, DELETE, CREATE, ALTER y INDEX desde un usuario
   denomindo root con password root, entonces la instalación resulta más directa
   puesto que estos son los valores de los parámetros de conexión a la base de
   datos que trae el framework por defecto. Obviamente se pueden cambiar, pero
   para probarlo por primera vez y realizar desarrollos rápidos, puede ser muy
   conveniente.

Contenido:

.. toctree::
   :maxdepth: 2
   
   tutorial-1
   tutorial-2
   tutorial-3

Indices and tables
==================

* :ref:`genindex`
* :ref:`search`

