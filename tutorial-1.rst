1. Despliegue del sistema y generación de aplicaciones con *symfonite*
======================================================================

En este primer tutorial nos centraremos en el despliegue del framework y en la generación del esqueleto [1]_ de las aplicaciones. 
Los primeros pasos a seguir son los siguientes:

1. Obtener el fichero *symfonite.r108.tgz* que contiene el *framework*:

   http://ntic.educacion.es/desarrollo/symfonite/downloads/symfonite-1.2.tgz

   También puedes obtener el código a través de github:

   https://github.com/juanda/symfonite-project

   Mira las instrucciones de despliegue que aparecen en el README de github.
   Si lo haces de esta manera, puedes pasar directamente al punto 3.

2. Descomprimir dicho fichero en algún directorio accesible por el web server
   (*/var/www* suele ser el *DocumentRoot* en muchas distribuciones *linux*) y entrar
   en el directorio recien creado (*symfonite*).

   .. code-block:: sh

      tar xvzf symfonite-1.2.tgz
      cd sft

3. Instalar el *framework*

   .. code-block:: sh

      php symfony project:installSft

   En este momento ya tenemos disponible todas las herramientas para desarrollar aplicaciones
   *symfonite* y los permisos de ficheros correctamente definidos. En el directorio *lib/symfony* se
   encuentra el nucleo original de *symfony* y en el directorio *plugins* se encuentran las siguientes
   extensiones que componen el *framework* *symfonite*:

   • *symfonitePlugin*, es un *plugin* básico que se necesita para el proceso de construcción de
     aplicaciones *symfonite* desde la *CLI*.
   • *themesPlugin*, se encarga de la gestión de los temas. Un tema es un conjunto de plantillas
     (HTML, XML, *json*, …) y un conjunto de activos (*css's*, *javascripts*, *imágenes*) que le dan un
     aspecto determinado a las aplicaciones desarrolladas con este *framework*. En un principio, el
     *plugin* define un tema por defecto (llamado *default*), con todos los elementos necesarios para
     dar una aspecto homogéneo a las aplicaciones. Pero se pueden añadir nuevos temas que
     redefinan las partes que deseemos cambiar del tema por defecto. Cada aplicación define en
     su configuración qué tema va a utilizar.
   • *sfBreadNav2Plugin*, se utiliza para construir los menús de las aplicaciones.
   • *sfJqueryReloadedPlugin*, incluye el *framework* *javascript* *jQuery* y algunos *helpers* (funciones
     PHP) que hacen uso de esta librería.
   • *sfGuardPlugin* y *sftGuardPlugin* se encargan del proceso de inicio de sesión de las
     aplicaciones que se desarrollen con este *framework*. También incluye las librerías con los
     objetos de acceso a datos (ORM), los formularios y los filtros.
   • *sftGestionPlugin*, implementa la aplicación de gestión del sistema *symfonite* (usuarios,
     unidades organizativas, perfiles, ámbitos, periodos, credenciales y aplicaciones, y otra serie
     de datos auxiliares)
   • *sftSAMLPlugin*, Identidad federada *SAML*. Incluye la librería *simpleSAMLphp* [2]_ (UNINETT) que se
     usa para desplegar un *Identity Provider* y como librería para que las aplicaciones construidas
     con el *framework* puedan actuar como *Services Providers* que usan el protocolo *SAML*. Estas
     aplicaciones internas pueden utilizar para realizar la autenticación este *IdP* sin necesidad de
     configurar nada. Por otro lado pueden autenticarse contra *IdP's* externos si se lleva a cabo el
     correspondiente intercambio de metadatos. De la misma manera el *IdP* interno puede ser
     utilizado para autenticar a los usuarios del sistema a petición de *SP's* externos conocidos.
   • *sftPAPIPlugin*, identidad federada *PAPI*. Implementa un *Authentication Server PAPI* y utiliza la
     librería *phpPoA* [3]_ (*RedIRIS*) para que las aplicaciones construidas con el *framework* puedan
     actuar como *Points of Access* (*PoA's*) que usan el protocolo *PAPI*. Estas aplicaciones internas
     pueden utilizar para realizar la autenticación este *AS* sin necesidad de configurar nada. Por
     otro lado pueden autenticarse contra *AS's* externos si se lleva a cabo el correspondiente
     intercambio de metadatos. De la misma manera el *AS* interno puede ser utilizado para
     autenticar a los usuarios del sistema a petición de *PoA's* externos conocidos.


4. Ahora vamos a desplegar la base de datos básica del sistema y la aplicación que la
   administra. En esta base de datos se almacenará la información sobre los usuarios, sus
   perfiles (organizados según el organigrama de la institución), las aplicaciones integradas en el
   sistema y las credenciales que conectan a los usuarios con las aplicaciones.
   En primer lugar, si aún no disponemos de la base de datos, la creamos [4]_:

   .. code-block:: sh

      mysqladmin -uroot -p create {nombre_base_datos}

   por ejemplo:

   .. code-block:: sh

      mysqladmin -uroot -p create sft

   Ahora definimos los parámetros de conexión a la base de datos. Para lo que ajustamos los
   parámetros *dsn*, *username* y *password* del fichero *config/databases.yml* del proyecto
   *symfonite* que acabas de desplegar. La parte sombreada en el siguiente fragmento del
   archivo *config/databases.yml* es la que debes tocar para ajustar los parámetros de conexión.
   
   .. code-block:: yaml

      …
      all:
         sft:
            class: sfPropelDatabase
            param:
               classname: PropelPDO
               dsn: 'mysql:dbname=symfonite;host=localhost'
               username: root
               password: root
               encoding: utf8
               persistent: true
               pooling: true

   .. note::
  
      Si tenemos una base de datos que se llame *sft*, en el servidor *localhost*, con un
      usuario *root* con password *root*, no es necesario hacer este paso pues estos son los parámetros de
      conexión que vienen por defecto en archivo *config/databases.yml*. En un entorno de desarrollo este
      hecho puede ser muy útil pues evita tener que tocar dicho fichero cada vez que se despliega el
      *framework*.

5. Crear la aplicación de administración y las tablas de la base de datos [5]_, para lo cual 
   ejecutamos desde el directorio raíz del proyecto la siguiente tarea de *symfony*:

   .. code-block:: sh

      ./symfony generate:appITE --titulo='Administración de la plataforma' --es_admin=true --url='http://localhost/sft/web' backend 

   El último parámetro (*backend*) es el nombre que *symfony* le asignará a la aplicación.

   .. note::

      Estamos suponiendo que la instalación la estamos realizando en la máquina local y que estamos
      desplegando el framework en un directorio que cuelga directamente del *DocumentRoot* y al
      que hemos llamado ``sft``.

   .. note::

      Si falla esta instrucción asegurate de que en la configuración del *PHP* (fichero *php.ini*, ubicado
      en */etc/php.ini* o */etc/php5/cli/php.ini* en muchas de las distribuciones *linux*) tienes asignada suficiente memoria (directiva
      *memory_limit*). Además debes borrar la aplicacion *backend* fallida (*rm -r ./apps/backend* desde el directorio *sft*).

   Durante la ejecución de la tarea aparece un mensaje de advertencia que indica que los datos
   almacenados en la base de datos serán sobreescritos. Como en nuestro caso estamos
   creándola por primera vez, esto no supone ningún problema. Así que contestamos afirmativamente.

   En este momento ya tenemos disponible una aplicación completamente funcional para la
   administración del sistema *symfonite*. Vamos a probarla un poco.

5. Abrimos el navegador y realizamos una petición al siguiente recurso [6]_:

      http://localhost/sft/web/index.php

   Y debe aparecer la pantalla de login de la aplicación que acabamos de crear. Para comenzar a
   utilizar la aplicación, se ha creado automáticamente un usuario con username “admin” y
   password “admin”. Úsalo para entrar en la aplicación. Ahora puedes dar una vuelta por los
   distintos menús para curiosear un poco.

   .. image:: imagenes/tutorial-sft-0.png

   Todas las aplicaciones *symfonite* presentan por defecto los siguientes elementos (puedes verlo en la
   aplicación que acabas de crear):

    • Un aspecto gráfico común
    • Un menú general donde se indica

      - el username del usuario,
      - su perfil
      - cambio de perfil,
      - cambio de la configuración personal,
      - lanzador de aplicaciones,
      - consulta de la ayuda
      - logout.

    • El menú de la aplicación, el cual se puede crear y modificar desde la aplicación de
      administración que acabamos de crear.
    • Un proceso común para realizar el inicio de sesión.

   El objetivo primordial de *symfonite* es servir como una plataforma de desarrollo rápido de
   aplicaciones. Para lo cual proporciona una herramienta con la que se generan esqueletos que
   incorporan las funcionalidades comunes que hemos mencionado más arriba. Para que dichas
   aplicaciones utilicen la base de datos con información acerca de usuarios-perfiles-credenciales,
   han de ser registradas en el sistema. Esto se hace a través de la aplicación de gestión que
   acabamos de desplegar. En el resto de este apartado veremos como registrar una nueva aplicación
   en el sistema, como generarla y como asociarla a un perfil para darle acceso.

6. Abre el menú "Aplicaciones → gestión" de aplicaciones, verás que existe una aplicación
   denominada “Gestión *symfonite*”. Se trata de la aplicación de administración que acabas de
   crear y estás utilizando en estos momentos.
7. Crea una nueva aplicación (en sentido estricto no la creamos, la registramos), para ello pica
   en el botón “Nuevo” y rellena el formulario. Utiliza los siguientes datos (registraremos una
   hipotética aplicación para la catalogación de recursos educativos):

   • *Codigo*: “catalogacion”
   • *Nombre*: “Catalogación de recursos educativos”
   • *Descripción*: “Aplicación para la catalogación de los recursos educativos de La Madraza.”
   • *Texto intro*: Puedes poner una introducción que aparecerá en la pantalla de login de la
     aplicación.
   • *es_syfonite*: marca la opción. En *symfonite* podemos registrar cualquier aplicación web,
     aunque no este hecha con el *framework*. Incluso aunque no esté construida con *PHP*. En ese
     caso el sistema la tiene en cuenta para que los usuarios puedan utilizarla si disponen de su
     credencial de acceso. Como la aplicación que estamos dando de alta será desarrollada con
     *symfonite* debes marcar la opción.
   • *Tipo login*: Aquí se define el tipo de login que la aplicación utilizará para identificar a los
     usuarios. Puede seleccionar uno de los siguientes:

      - *Normal*. Se trata de un inicio de sesión nativo. No dispone de *SSO* (*Single Sign On*)
      - *Identidad Federada SAML*. Para realizar el inicio de sesión sobre el Proveedor de Identidad
        *SAML* integrado o con otro externo. En el próximo tutorial se trata la identidad federada
        con más detalle.
      - *Identidad Federada PAPI*. Para realizar el inicio de sesión sobre el Servidor de
        Autenticación *PAPI* integrado o con otro externo.
        Por lo pronto elige login “Normal”.

   • *Logotipo*: sube la imagen que quieras (si quieres, no es obligatorio)
   • *Url*: La *URL* de la aplicación: http://localhost/sft/web/catalogacion.php
   • *Url svn*: si tienes la aplicación bajo control de versiones puedes utilizar este campo (en
     realidad, y por lo pronto no sirve de nada)
   • *Clave*: pulsa en el botón “genera clave” y se generará una clave automáticamente. Esta clave
     es necesaria para que la aplicación pueda identificarse en el sistema.
   • *Created at y Updated at*: déjalos en blanco y se definirán automáticamente..

   Pulsa el botón guardar y ya tenemos la aplicación registrada. Pulsa el botón “Atrás” para
   volver al listado de aplicaciones y comprueba que ha sido dada de alta en el sistema.

8. Ahora vamos a generar el esqueleto funcional de la aplicación, a partir del cual
   programaremos las funcionalidades específicas de la aplicación. Para ello volvemos a la *CLI* y
   lanzamos el siguiente comando [7]_:

   .. code-block:: sh

      ./symfony generate:appITE --titulo="Catalogación de recursos" --clave=73b1ec9a760173a catalogacion

   El último parámetro de la tarea anterior (catalogación) es el nombre que *symfony* le asignará
   a la aplicación.

   Debes sustituir el valor del parámetro clave por la clave que le has asociado a la aplicación
   cuando la registraste. Ahora ya tenemos un “esqueleto” de la aplicación que implementa las
   funcionalidades comunes de todas las aplicaciones *symfonite*. Podemos probarlo
   introduciendo en el navegador la *URL* que hemos definido en el registro de la aplicación o
   picando directamente en el enlace que aparece en la fila correspondiente a la aplicación en
   cuestión en la pantalla que lista las aplicaciones. Si intentas entrar con el único usuario que
   tenemos (por lo pronto) la aplicación lo rechazará, ya que en ningún momento le hemos dado
   permiso para que pueda utilizar la nueva aplicación.

9. Ahora vamos a conectar al usuario admin con la aplicación de catalogación que acabamos de
   crear. Esto se hace a través de los perfiles. Si te fijas este usuario sólo tiene un perfil (que por
   otro lado es el único que por lo pronto existe; el SuperAdministrador). Puedes ver esto desde
   la pantalla “Usuarios → gestión de personas”. Lo que haremos es asociar al perfil
   “SuperAdministrador” la credencial que da acceso a la aplicación que acabamos de crear.

   • Accedemos a “UOS [8]_ → gestión de perfiles”, y picamos en “Asoc. Creds” del perfil
     “SuperAdministrador”. Se abrirá una ventana modal [9]_ que permite asociar y desasociar
     credenciales a los perfiles. Cada aplicación posee, al menos, una credencial de acceso (más
     adelante se pueden añadir más credenciales a la aplicación para realizar un control más fino
     sobre el acceso a las funcionalidades de la misma). Asociamos la credencial
     “catalogacion_ACCESO” picando en el botón “Poner”. Y salimos de la ventana modal picando
     fuera de ella. Acabamos de dar permiso al perfil SuperAdministrador para que acceda a la
     aplicación catalogación. Vamos a probarlo.


10. Introducimos en el navegador la *URL* de la aplicación catalogación.

      http://localhost/sft/web/catalogacion.php

   Introducimos el username y el password (*admin*, *admin*) y ya estamos dentro. La primera vez
   que entras tienes que seleccionar el perfil por defecto con el que accederás en lo sucesivo a
   la aplicación. Sin embargo esta forma de entrar, aunque perfectamente legítima, no es la más
   cómoda. Así que vamos a volver a entrar en la aplicación. Vuelve a la aplicación de gestión
   (*backend*) y pica en el enlace “aplicaciones” que está en el menú general (arriba a la
   derecha). Te deben aparecer todas las aplicaciones a las que el usuario *admin* tiene acceso.
   En nuestro caso a la aplicación de gestión y a la de catalogación. Desde aquí puedes acceder
   directamente. Además, si has elegido como tipo de login alguno de los que incluyen identidad
   federada, podrás entrar en las aplicaciones sin volver a hacer login (*Single Sign On*).
   De todo esto se deduce que se puede saltar de una aplicación a otra a través del menu
   general “aplicaciones”, ya que este menú es ofrecido por todas las aplicaciones generadas
   por el *framework* [10]_.

   En esta primera parte del tutorial hemos descrito como utilizar el *framework* para construir
   aplicaciones que comparten los mismos usuarios y ofrecen una serie de funcionalidades comunes
   (marco) como son el inicio de sesión, el diseño gráfico, el menú general y la posibilidad de definir un
   menú particular para cada aplicación. El sistema de inicio de sesión permite que el usuario sólo
   tenga que autenticarse una vez en el sistema (*SSO*). Además, el usuario puede saltar entre las
   aplicaciones que tiene asociadas a través de sus perfiles.

   Ahora se trata de desarrollar las funcionalidades específicas de la aplicación. El desarrollo se realiza
   como cualquier otra aplicación *symfony*, por lo que es importante conocer este *framework*. También
   es importante conocer los datos que la aplicación tiene accesible a través de la sesión para acoplar
   adecuadamente la aplicación que se desarrolla al sistema de usuarios-perfiles-credenciales. Pero ese
   será el tema de otro tutorial [11]_: “Ejemplo de desarrollo de una aplicación con *symfonite*.

2. Uso del *framework* de Identificación federada
=================================================

*Symfonite* proporciona, desde el momento en que se despliega, dos sistemas distintos de identidad
federada:

• *SAML*
• *PAPI*

En esta segunda parte del tutorial comprobaremos el enriquecimiento que supone utilizar alguno (o
ambos) de estos sistemas en lo referente al acceso de usuarios a las aplicaciones.

2.1. *Single Sign On* (*SSO*) y Autenticación con *SAML*
--------------------------------------------------------

Los sistemas de *SSO* permiten al usuario acceder a todas sus aplicaciones identificándose una sola
vez. Los dos sistemas de identidad federada que integra *symfonite* ofrecen esta funcionalidad.
Vamos a configurar el *SSO* usando el componente *SAML* de *symfonite*. Para ello:

1. Entra en el módulo de gestión de aplicaciones (menú “Aplicaciones”) de la aplicación de
   gestión y cambia el tipo de login de las dos aplicaciones que tenemos disponibles a “*SAML*”
   (puedes hacer esto desde el propio listado de aplicaciones, marcándolas y ejecutando la
   operación en lote *Login tipo SAML*).

   .. image:: imagenes/tutorial-sft-1.png

2. Sal de la aplicación (botón “salir” arriba a la derecha). Verás que se te redirige a una pantalla
   en la que puedes elegir un proveedor de identidad. Selecciona el *IdP symfonite*. Aparecerá
   la pantalla de login del *IdP* (diferente a la del login normal). Identifícate y entrarás en la
   aplicación.

3. Ahora cambia de aplicación. Pica sobre el enlace “aplicaciones” (arriba a la derecha) y
   aparecerá el lanzador de aplicaciones. Si picas en el botón de la aplicación de “catalogación”
   verás como entras en ella sin necesidad de autentificarte de nuevo (*SSO*).

   .. image:: imagenes/tutorial-sft-2.png

2.2. *SSO* y Autenticación con *PAPI*
-------------------------------------

Ahora hacemos lo mismo que en el subapartado anterior pero usando *PAPI*. Basta con repetir los
pasos anteriores cambiando el tipo de login de las aplicaciones a “*PAPI*”.

.. note::

   Para la identidad federada *PAPI* se ha integrado la librería *phpPoA* (*RedIRIS*). En el momento
   en que se escribe esto, dicha librería no dispone de un servicio de logout. Por ello para salir
   completamente de la aplicación debes borrar manualmente las cookies de tu navegador.

2.3. Más allá del *SSO*
-----------------------

En los ejemplos anteriores hemos visto que las aplicaciones *symfonite* incorporan dos sistemas
distintos para realizar *Single Sign On*, La clave está en el uso de los protocolos *SAML* y/o *PAPI*. Pero la
integración de estos protocolos en el *framework* va más alla del *SSO*. De hecho la parte más
interesante consiste en la posibilidad de crear y formar parte de lo que se conoce como federaciones
de aplicaciones o sistemas de identidad federada.

Lo que se pretende con estos sistemas es que la identificación del usuario se lleve a cabo fuera de la
aplicación. Técnicamente a la aplicación se le denomina proveedor de servicio, y al servicio que lleva
a cabo la identificación se le llama proveedor de identidad. Esta separación permite:

• Que una misma aplicación dé servicio a usuarios que pertenecen a distintas instituciones,
  ampliando el número de usuarios que pueden usarla y liberandose, a la vez, de la gestión
  detallada de los mismos.
• Que un mismo proveedor de servicio pueda “ofrecer” a sus usuarios muchas aplicaciones que
  pueden pertenecer a distintas instituciones, ampliandose así el nº de aplicaciones que una
  misma institución puede ofrecer a sus usuarios.

Para ello se deben establecer unas relaciones de confianza entre los proveedores de servicio y los de
identidad. Los protocolos *SAML* y *PAPI* gestionan dichas relaciones, La combinación de estos dos
puntos enriquece enórmemente las posiblididades de conectividad entre usuarios y aplicaciones de
distintas instituciones.

Las aplicaciones construidas con *symfonite* se integran fácilmente en una federación como
proveedoras de servicio. Y además, los proveedores de identidad *SAML* y *PAPI* incluidos en *symfonite*
pueden ser utilizados como servicio de identidad por aplicaciones externas al sistema.

2.3.1. Auntenticación sobre un *IdP* *SAML* externo
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Ahora veremos como realizar la autenticación sobre un *IdP* externo. Utilizaremos un *IdP* de
pruebas [12]_. Para que esta parte del tutorial funcione, es imprescindible que hayas desplegado el
*framework* de manera que la *URL* sea:

   http://localhost/sft/web/

Sal de nuevo de la aplicación y selecciona ahora el *IdP* “openidp feide”. Se produce una redirección
a la pantalla de login del *IdP* de pruebas de *Feide*. Introduce como datos de identificación:

- username=”dilbert”
- password=”dilbert”. 

Una vez devuelto el control a la aplicación esta mostrará
los atributos que el *IdP* externo ha enviado. Ahora se trata de hacer algo con ellos. Es decir, de
programar la aplicación.

La autenticación se ha llevado a cabo gracias a la confianza que existe entre el sistema *symfonite* y
el proveedor de identificación. Esta confianza se establece mediante el intercambio de los
denominados metadatos entre ambos servicios. Los metadatos de todos los Proveedores de
Identificación *SAML* que vayan a ser utilizados por las aplicaciones *symfonite* deben ser añadidos al
archivo:

.. code-block:: sh

   /ruta/a/sft/pluginss/sftSAMLplugins/lib/vendor/simpleSAMLphp-1.8.0-rc1/metadata/SAML20-idp-remote.php

Échale un vistazo. Verás los metadatos del *IdP* interno y los del *openidp.feide.no.* Por otra parte, los
metadatos del Proveedor de servicio, es decir, de las aplicaciones *symfonite*, deben ser facilitados al
Proveedor de Identificación con el que se vaya a establecer la confianza. Para consultar estos
metadatos pica en el menú “*Identidad Federada-> Gestión SAML*”, se abrirá la aplicación de
instalación de la librería *simpleSAMLphp* y en la pestaña “Federación” puedes ver tanto los
metadatos del *SP* como los del *IdP*.

2.3.2. Auntenticación de una aplicación externa con los *IdP 's* de *symfonite*
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Para que una aplicación externa pueda usar el servicio de identificación de *symfonite* debe saber
“hablar” *SAML* y/o *PAPI*.

En el caso de utilizar *SAML* el procedimiento para realizar la federación es el siguiente:

1. La damos de alta en el sistema a través del módulo de gestión de aplicaciones de la
   aplicación de gestión. En este caso se deja sin marcar la opción “es *symfonite*”.
2. Elegimos como tipo de login “SSO con sistema de identidad federada *SAML*”.
3. Pedimos los metadatos del IdP al que deseamos conectar y lo añadimos al fichero *symfonite-1.2.tgz*

.. code-block:: sh

   /ruta/a/sft/pluginss/sftSAMLplugins/lib/vendor/simpleSAMLphp-1.8.0-rc1/metadata/SAML20-idp-remote.php

4. Obtenemos los metadato *SAML* de *symfonite*. Para ello entramos en la aplicación de
   instalación de *simpleSAMLphp* (menú “Identidad Federada->Gestión *SAML*” ). La pestaña
   “Federación” da acceso a los metadatos de todos los *IdP's* y *SP's* instalados.
5. Proporcionar estos metadatos al administrador del *IdP* para que los incluya en su sistema
6. Y ya está.

En el caso de utilizar *PAPI* el procedimiento es como sigue:

1. La damos de alta en el sistema a través del módulo de gestión de aplicaciones de la
   plicación de gestión. En este caso se deja sin marcar la opción “es *symfonite*”.
2. Elegimos como tipo de login “SSO con sistema de identidad federada *PAPI*”.
3. Pedimos la clave pública y la *URL* del *AS* o *GpoA* al que deseamos conectar e insertamos
   estos datos en el formulario al que se accede desde el menú: “*Identidad Federada → Gestión
   PAPI*”.
4. Proporcionamos al administrador del *AS* en cuestión los datos que nos pida.
5. Y ya está.

3. Desarrollo de un caso práctico. Sistema de aplicaciones web para el centro de estudios “La Madraza”
======================================================================================================

En esta tercera parte del tutorial desarrollaremos un sistema de aplicaciones web para un organismo
ficticio dedicado a la formación a distancia, al que denominaremos “La Madraza. Centro de Estudios
Filosóficos“. Este centro de estudios se organiza en tres departamentos, cada uno de los cuales
define sus perfiles según el siguiente esquema:



+--------------------------------+--------------------------------+--------------------------------+
| Departamento                   | Perfil                         | Ámbito                         |
+================================+================================+================================+
| Telemática                     | Sistemas (tipo de ámbito:      | Servidores web                 |
|                                | áreas)                         +--------------------------------+
|                                |                                | Servidores de correo           |
|                                |                                +--------------------------------+
|                                |                                | Electrónica de red             |
|                                +--------------------------------+--------------------------------+
|                                | Desarrollo (tipo de ámbito:    | Plataforma educativa           |
|                                | proyectos)                     +--------------------------------+
|                                |                                | Programa de contabilidad       |
|                                |                                +--------------------------------+
|                                |                                | Administración de la plataforma|
|                                |                                +--------------------------------+
|                                |                                | Mensajería interna             |
|                                +--------------------------------+--------------------------------+
|                                | Super Administrador            |              ---               |
+--------------------------------+--------------------------------+--------------------------------+
| Formación                      | Jefe de Estudios               |              ---               |
|                                +--------------------------------+--------------------------------+
|                                | Coordinador (tipo de ámbito:   | Los presocráticos (2009 y 2010)|
|                                | curso)                         +--------------------------------+
|                                |                                | La moral en Kant (2009 y 2010) |
|                                |                                +--------------------------------+
|                                |                                | Iniciación al Capital de Karl  |
|                                |                                | Marx (2010)                    |
|                                |                                +--------------------------------+
|                                |                                | Bioética (2010)                |
|                                +--------------------------------+--------------------------------+
|                                |Profesor (tipo de ámbito: curso)| Los mismos que en la fila de   |
|                                |                                | arriba                         |
|                                +--------------------------------+--------------------------------+
|                                |Alumno (tipo de ámbito: curso)  | Los mismos que en la fila de   |
|                                |                                | arriba                         |
+--------------------------------+--------------------------------+--------------------------------+
| Secretaría                     | Secretario                     |              ---               |
|                                +--------------------------------+--------------------------------+
|                                | Contable                       |              ---               |
+--------------------------------+--------------------------------+--------------------------------+


En la columna de los perfiles se ha indicado entre paréntesis el tipo del ámbito donde el perfil actúa.
Por ejemplo: podemos tener desarrolladores que trabajan en los proyectos “Plataforma educativa”,
“Programa de contabilidad”, Administración de la plataforma” y “Mensajería interna”.

Supondremos que nuestro centro de estudios lleva funcionando desde el 2009, y que las tareas que
realizan en la secretaría están organizadas temporalmente en ejercicios contables, es decir en
periodos. En la columna de ámbitos, entre paréntesis se ha indicado los periodos en los que existen.

La siguiente lista muestra los nombres de los usuarios de nuestro centro, así como las funciones que
tienen asignadas, es decir, los perfiles:

+----------------------+----------------------+----------------------+------------------------------------------------------+
| Nombre               | Apellidos            | Nombre de usuario    | Perfiles                                             |
+======================+======================+======================+======================================================+
| Alberto              | Einstein             | alberto              | Super Administrador de Telemática                    |
|                      |                      |                      +------------------------------------------------------+
|                      |                      |                      | Sistemas, área de servidores web                     |
+----------------------+----------------------+----------------------+------------------------------------------------------+
| Max                  | Planck               | max                  | Sistemas, área de servidores de correo               |
|                      |                      |                      +------------------------------------------------------+
|                      |                      |                      | Desarrollador de “Mensajería interna”                |
|                      |                      |                      +------------------------------------------------------+
|                      |                      |                      | Desarrollador de la “plataforma educativa”           |
|                      |                      |                      +------------------------------------------------------+
|                      |                      |                      | Contable                                             |
+----------------------+----------------------+----------------------+------------------------------------------------------+
| Isaac                | Newton               | Isaac                | Secretario                                           |
|                      |                      |                      +------------------------------------------------------+
|                      |                      |                      | Contable                                             |
|                      |                      |                      +------------------------------------------------------+
|                      |                      |                      | Profesor del curso “Los presocráticos” (2009 y 2010) |
|                      |                      |                      +------------------------------------------------------+
|                      |                      |                      | Profesor del curso “Bioética” (2010)                 |                              
+----------------------+----------------------+----------------------+------------------------------------------------------+
| Alejandro            | Volta                | alejandro            | Jefe de Estudios                                     |
|                      |                      |                      +------------------------------------------------------+
|                      |                      |                      | Secretario                                           |
|                      |                      |                      +------------------------------------------------------+
|                      |                      |                      | Profesor del curso “La moral en Kant” (2010)         |
+----------------------+----------------------+----------------------+------------------------------------------------------+
| Miguel               | Faraday              | miguel               | Alumno del curso “Los presocráticos” (2010)          |
|                      |                      |                      +------------------------------------------------------+
|                      |                      |                      | Alumno del curso “Bioética” (2010)                   |
+----------------------+----------------------+----------------------+------------------------------------------------------+
| Ernesto              | Rutherford           | ernesto              | Alumno del curso “Los presocráticos” (2010)          |
|                      |                      |                      +------------------------------------------------------+
|                      |                      |                      | Alumno del curso “Bioética” (2010)                   |
+----------------------+----------------------+----------------------+------------------------------------------------------+

El centro educativo cuenta con las siguientes aplicaciones desplegadas en su sistema telemático:


+---------------------------------+-------------------------------------------------------------------+
| Nombre de la aplicación         | Descripción                                                       |
+=================================+===================================================================+
| Plataforma educativa            | Es la plataforma de e-learning que utilizan los profesores para   |
|                                 | realizar el seguimiento de los alumnos y los alumnos para seguir  |
|                                 | los cursos.                                                       |
+---------------------------------+-------------------------------------------------------------------+
| Programa de contabilidad        | Pues eso, un programa para llevar la contabilidad de la empresa   |
+---------------------------------+-------------------------------------------------------------------+
| Administración de la plataforma | Es la aplicación con la que se administra la estructura           |
|                                 | administrativa.                                                   |
+---------------------------------+-------------------------------------------------------------------+
| Mensajería interna              | Es una aplicación con la que los usuarios del centro de estudios  |
|                                 | pueden comunicarse entre sí.                                      |               
+---------------------------------+-------------------------------------------------------------------+

Como es lógico, se corresponden con los proyectos de desarrollo del departamento de telemática.

A lo largo de este tutorial mostraremos como utilizar *symfonite* para desarrollar el sistema de
aplicaciones que proporcionará el software de apoyo del centro de estudios ficticio “La Madraza”.

Por simplificar, como estrategia de despliegue, hemos decidido desarrollar todas las aplicaciones
dentro de un mismo proyecto de *symfony*

3.1. Creación de la base de datos que implementa la estructura organizativa y de la aplicación de administración de la misma
----------------------------------------------------------------------------------------------------------------------------

Lo primero que haremos será crear el proyecto de *symfony* que alojará a la aplicación de
administración de la estructura organizativa, es decir, lo que hemos llamado más arriba
“Administración de la plataforma”. Se trata de seguir los 3 primeros puntos de la primera parte de
este tutoria (obviamente si has realizado el tutorial desde el principio no tienes que volver a repetir
dichos pasos). En este momento ya podemos entrar en la aplicación a través de la *URL*:

.. code-block:: sh

   http://localhost/sft/web/index.php

Para entrar utilizamos el usuario identificado como:

   - Usuario: admin
   - Password: admin

El cual fue creado automáticamente por la tarea *appITE* para poder entrar la primera vez en la
aplicación.

3.2. Alta de usuarios
---------------------

Lo que haremos ahora es editar este usuario para cambiarle su nombre, su nombre de usuario y su
password. Para ello vamos al menú *USUARIOS → GESTIÓN DE PERSONAS* y seleccionamos la acción EDITAR
del usuario admin. Entonces le cambiamos el nombre por “Alberto” y el apellido 1 por “Einstein”.
Presionamos GUARDAR y volvemos al listado de personas. Ahora le cambiamos el nombre de usuario y
password a través de la acción PASSWORD del usuario “Alberto Einstein” . Definimos como nombre de
usuario “alberto” y como password “pruebas”. También tienes que rellenar el campo e-mail pues
es obligatorio. Invéntatelo.

Ahora es el momento de dar de alta al resto de las personas. Para ello debes utilizar el enlace NUEVO
de la pantalla del listado de personas. Para dar de alta el próximo usuario de manera más ágil,
puedes utilizar el botón GUARDAR Y CREAR OTRO. Observa que cuando se crea una persona,
automáticamente se genera un nombre de usuario y una contraseña. El nombre de usuario es una
combinación del nombre y los apellidos, y el password sigue el siguiente patrón:
pass{nombre_usuario}. El usuario debería cambiar este password la primera vez que entra en su
cuenta utilizando el enlace CONFIGURACIÓN PERSONAL. Sin embargo, para facilitar el seguimiento de este
tutorial, cámbia tú mismo el nombre de usuario y contraseña a los usuarios que hayas creado desde
la cuenta de Super Administrador. Usa como nombre de usuario el que se indica en la tabla de
usuarios de nuestro centro de estudio y como contraseña para todos ellos 'pruebas'.

3.3. Alta del organigrama
-------------------------

Una vez que tenemos registradas las personas del centro de estudios, hay que asignarles perfiles.
Por lo que previamente hay que crear el organigrama del centro. Comenzamos por crear los tres
departamentos (unidades organizativas). Esto lo hacemos a través del menú *UOS → GESTION DE
UOS*. Utilizando el link EDITAR, cambia el nombre a la UO de Administración por “Telemática” (Puedes
introducir también la traducción al inglés), y añade las demás utilizando el link NUEVO del listado de
*UOS*. Debes rellenar el campo código pues es obligatorio. Invéntatelo.
Fíjate que en el listado de UOS, aparecen las que acabamos de crear con un aviso en rojo indicando
que aún hay que asociarles perfiles.

Ahora vamos a definir los periodos en las unidades organizativas de “Formación” y “Secretaría”. En
la de “Telemática” no es necesario pues no se organiza por periodos y le basta el periodo que se
crea automáticamente al crear la unidad organizativa.

Recordemos que la Secretaría debía tener en cuenta periodos contables, y el departamento de
Formación ejercicios académicos. Como el centro de estudios comenzó su actividad en el 2009 y
estamos en el 2010, crearemos 2 ejercicios para la Secretaría y otros dos para el departamento de
Formación. En ambos casos serán:

- Periodo 1: desde el 1-1-2009 hasta el 31-12-2009
- Periodo 2: desde el 1-1-2010 hasta el 31-12-2010

Siendo el estado del primero INACTIVO y el segundo ACTIVO. Los periodos requieren el campo
código. Invéntatelos.

Tendrás que editar el periodo correspondiente a la UO Telemática, ya que este periodo se creó
automáticamente cuando creamos la aplicación de administración (recuerda que hemos cambiado el
nombre de esta *UO*). Puedes acceder directamente a los periodos de cada *UO* utilizando el link
PERIODOS del listado de *UOS*. O a través del enlace de menú *UOS → PERIODOS*. El único período del
departamento de Telemática debe tener como fecha de inicio el 1-1-2009, y la fecha final se dejará
sin rellenar. Asegúrate de que esté activo.

Ten en cuenta que hay que darle una descripción a los periodos. Pon lo que creas conveniente.

Y ahora vamos a crear los perfiles de cada departamento. Pero antes es conveniente definir los tipos
de ámbitos que podrán asociarse a los perfiles y los ámbitos que podrán asociarse a los usuarios con
un perfil determinado. Esto lo hacemos a través del menú *UOS → GESTIÓN DE ÁMBITOS*.
Una vez creado los tipos de ámbitos cursos, proyectos y áreas, añade los ámbitos concretos en los
periodos correspondientes. Por ejemplo, una vez creado el tipo de ámbito cursos, desde la pantalla
que muestra el listado de tipos de ámbitos, pulsa en el enlace ÁMBITOS para añadir los cursos en el
periodo del departamento (UO) de Formación que les corresponda:

• Los presocráticos, en el 2009 y 2010.
• La moral en Kant, en el 2009 y 2010.
• Iniciación al Capital de Karl Marx, en el 2010.
• Bioética en el 2010.

Haz lo mismo con los tipos de ámbitos “proyectos” y “áreas” y con sus ámbitos asociados.
Ya puedes dar de alta los perfiles a través del menú *UOS → GESTIÓN DE PERFILES*, pulsando el link
NUEVO del listado de perfiles. También puedes hacerlo a través de la acción PERFILES en el listado de
UOS. No olvides asociar a cada perfil el tipo de ámbito que le corresponde (en caso de que tenga
asociado ámbitos).

3.4. Registro de aplicaciones y credenciales
--------------------------------------------

Ahora vas a registrar las aplicaciones que forman el sistema de aplicaciones del centro de estudios
“La Madraza”. Pulsa en el menú *APLICACIONES → GESTIÓN DE APLICACIONES*, y en el link NUEVO. Inserta
los datos correspondiente a cada aplicación.

• El código de la aplicación debe ser único y no debe contener espacios ni tildes. Este dato es
  utilizado por el sistema para establecer el nombre de la sesión de usuario en el servidor y 
  como nombre de la aplicación *symfony*. (*sfapp*). Es obligatorio.
• El nombre puede ser cualquier frase de no más de 255 caracteres y es obligatorio.
• La descripción puede ser cualquier frase de no más de 255 caracteres y no es obligatoria.
• Pon los textos de introducción que quieras.
• Todas las aplicaciones que vamos a registrar en este tutorial son *symfonite*. Así que marca
  esta opción.
• Elige como tipo de login “Normal”
• Le puedes asociar un logotipo. No es obligatorio.
• La URL es la url que tendrá asociada la aplicación en el entorno de producción. Es obligatoria.
  En tu caso será algo así como:

  .. code-block:: sh

   http://localhost/sft/web/nombre_aplicacion.php, pero depende

  de cómo hayas desplegado el *framework*.
• La *URL SVN* es la url del proyecto en subversion. No es obligatoria.
• La clave es un conjunto de caracteres alfanuméricos que debe ser único para cada aplicación.
  Lo mejor es pulsar en el link GENERAR CLAVE para obtenerla.
• No es necesario introducir las fechas. Lo hace automáticamente la aplicación.

.. note::

   Por lo pronto no le cambies ni el código ni la clave a la aplicación de administración de la
   plataforma. Más adelante veremos como hacerlo. El problema es que, como después veremos, si
   cambiamos el código y/o la clave hay que hacer algún cambio en el código de la aplicación.

Usa la siguiente tabla para definir los nombres, códigos y las *URL's* de las aplicaciones:

+--------------------------------+--------------------------------+---------------------------------------+
| Nombre Aplicación              | Código                         | URL                                   |
+================================+================================+=======================================+
| Administración de la plataforma| backend                        | http://localhost/sft/web/index.php    |
+--------------------------------+--------------------------------+---------------------------------------+
| Plataforma educativa           | platedu                        | http://localhost/sft/web/platedu.php  |
+--------------------------------+--------------------------------+---------------------------------------+
| Contabilidad                   | conta                          | http://localhost/sft/web/conta.php    |
+--------------------------------+--------------------------------+---------------------------------------+
| Mensajería interna             | mensajeria                     |http://localhost/sft/web/mensajeria.php|
+--------------------------------+--------------------------------+---------------------------------------+

Cuando se registra una nueva aplicación, automáticamente se crea una credencial especial, que
denominamos credencial de acceso a la aplicación, y que es utilizada por el sistema *symfonite*
aplicación para dejar que accedan a ella únicamente los usuarios que posean esta
credencial. El nombre de la credencial de acceso sigue el patrón: {nombre_aplicacion}_ACCESO.

Ahora vas a definir las credenciales particulares de cada aplicación, las cuales permitirán mostrar
distintas funcionalidades a cada perfil que tenga acceso a la aplicación. Utiliza la información de la
siguiente tabla para crear dichas credenciales. Accede a través del menú APLICACIONES → GESTIÓN DE
APLICACIONES para realizar las operaciones. Cuando listes las credenciales observa como se han
creado automáticamente las credenciales de acceso a cada aplicación que has registrado.

+--------------------------------+-----------------------------------+-------------------------------------------+
| Aplicación                     | Credencial                        | Descripción                               |
+================================+===================================+===========================================+
| Administración de la           | SFTGESTION*plugins_administracion | Administración completa de la             |
| plataforma                     |                                   | aplicación (ya está dada de alta)         |
|                                +-----------------------------------+-------------------------------------------+
|                                |SFTGESTION*plugins_administracion_ | Administración restringida. Permite a     |
|                                |uo                                 | quien la posee, asociar perfiles de su    |
|                                |                                   | propia Unidad Organizativa a los          |
|                                |                                   | usuario que están dados de alta. (ya      |
|                                |                                   | está dada de alta)                        |
|                                +-----------------------------------+-------------------------------------------+
|                                | admin_ACCESO                      | Credencial de acceso a la aplicación      |
|                                |                                   | de administración                         |
+--------------------------------+-----------------------------------+-------------------------------------------+
| Plataforma educativa           | platedu_Docente                   | Permite usar la funcionalidad de          |
|                                |                                   | docente de la plataforma educativa        |
|                                +-----------------------------------+-------------------------------------------+
|                                | platedu_Alumno                    | Permite usar la funcionalidad de          |
|                                |                                   | alumno de la plataforma educativa         |
|                                +-----------------------------------+-------------------------------------------+
|                                | platedu_Admon                     | Permite usar la funcionalidad de          |
|                                |                                   | administración de la plataforma educativa |
|                                +-----------------------------------+-------------------------------------------+
|                                | platedu_Coord                     | Permite usar la funcionalidad de          |
|                                |                                   | coordinación de la plataforma educativa   |
|                                +-----------------------------------+-------------------------------------------+
|                                | platedu_ACCESO                    | Credencial de acceso a la plataforma      |
|                                |                                   | educativa                                 |
+--------------------------------+-----------------------------------+-------------------------------------------+
| Contabilidad                   | conta_ACCESO                      | Credencial de acceso a la                 |
|                                |                                   | contabilidad                              |
+--------------------------------+-----------------------------------+-------------------------------------------+
| Mensajería interna             | mensajeria_ACCESO                 | Credencial de acceso a la mensajería      |
|                                |                                   | interna                                   |
+--------------------------------+-----------------------------------+-------------------------------------------+

Observa que las credenciales de la aplicación de Administración de la plataforma ya están dadas de
alta. Esto se hizo cuando se generó dicha aplicación.

3.5. Asociación de perfiles a usuario y de credenciales a perfiles
------------------------------------------------------------------

Ahora vas a asociar perfiles y ámbitos (en su caso) a los usuarios del sistema. Esto se hace desde la
gestión de personas (o de organismos, según el caso). Entra en ella a través del menú *USUARIOS →
GESTIÓN DE PERSONAS*. Una vez en el listado pulsa en el link PERFILES del usuario al quieras asociar
perfiles y ámbitos, y desde la pantalla que se abre (“gestión de perfiles del usuario”), usa el link
ASOCIAR/DESASOCIAR PERFILES para asociar los perfiles a los usuarios siguiendo este esquema. Los
ámbitos se añaden a cada perfil que lo requiera usando el botón AÑADIR ÁMBITO que aparece en la
columna de acciones del listado de perfiles:

+----------------------+------------------------------------------------------+
| Nombre de usuario    | Perfiles                                             |
+======================+======================================================+
| alberto              | Super Administrador de Telemática                    |
|                      +------------------------------------------------------+
|                      | Sistemas, área de servidores web                     |
+----------------------+------------------------------------------------------+
| max                  | Sistemas, área de servidores de correo               |
|                      +------------------------------------------------------+
|                      | Desarrollador de “Mensajería interna”                |
|                      +------------------------------------------------------+
|                      | Desarrollador de la “plataforma educativa”           |
|                      +------------------------------------------------------+
|                      | Contable                                             |
+----------------------+------------------------------------------------------+
| Isaac                | Secretario                                           |
|                      +------------------------------------------------------+
|                      | Contable                                             |
|                      +------------------------------------------------------+
|                      | Profesor del curso “Los presocráticos” (2009 y 2010) |
|                      +------------------------------------------------------+
|                      | Profesor del curso “Bioética” (2010)                 |                              
+----------------------+------------------------------------------------------+
| alejandro            | Jefe de Estudios                                     |
|                      +------------------------------------------------------+
|                      | Secretario                                           |
|                      +------------------------------------------------------+
|                      | Profesor del curso “La moral en Kant” (2010)         |
+----------------------+------------------------------------------------------+
| miguel               | Alumno del curso “Los presocráticos” (2010)          |
|                      +------------------------------------------------------+
|                      | Alumno del curso “Bioética” (2010)                   |
+----------------------+------------------------------------------------------+
| ernesto              | Alumno del curso “Los presocráticos” (2010)          |
|                      +------------------------------------------------------+
|                      | Alumno del curso “Bioética” (2010)                   |
+----------------------+------------------------------------------------------+

Ahora asocia a los perfiles que acabas de crear las credenciales adecuadas. Esto se hace usando la
operación ASOC. CREDS. del menú *UOS->GESTIÓN DE PERFILES*. Usa la siguiente tabla:

+--------------------------+----------------------------------------+
| Perfil                   | Credenciales asociadas                 |
+==========================+========================================+
| SuperAdministrador       | Mensajeria_ACCESO                      |
|                          +----------------------------------------+
|                          | GestionEDAE3_ACCESO                    |
|                          +----------------------------------------+
|                          | EDAGESTION*plugins_administracion      |
|                          +----------------------------------------+
|                          | EDAGESTION*plugins_administracion_uo   |
+--------------------------+----------------------------------------+
| Sistemas                 | Mensajeria_ACCESO                      |
+--------------------------+----------------------------------------+
| Desarrollador            | Mensajeria_ACCESO                      |
+--------------------------+----------------------------------------+
| Profesor                 | Mensajeria_ACCESO                      |
|                          +----------------------------------------+
|                          | PlatEdu_ACCESO                         |
|                          +----------------------------------------+
|                          | PlatEdu_Docente                        |
+--------------------------+----------------------------------------+
| Alumno                   | Mensajeria_ACCESO                      |
|                          +----------------------------------------+
|                          | PlatEdu_ACCESO                         |
|                          +----------------------------------------+
|                          | PlatEdu_Alumno                         |
+--------------------------+----------------------------------------+
| Coordinador              | Mensajeria_ACCESO                      |
|                          +----------------------------------------+
|                          | PlatEdu_ACCESO                         |
|                          +----------------------------------------+
|                          | PlatEdu_Coord                          |
+--------------------------+----------------------------------------+
| Jefe de Estudios         | Mensajeria_ACCESO                      |
|                          +----------------------------------------+
|                          | PlatEdu_ACCESO                         |
|                          +----------------------------------------+
|                          | PlatEdu_Admon                          |        
+--------------------------+----------------------------------------+
| Secretario               | Mensajeria_ACCESO                      |
+--------------------------+----------------------------------------+
| Contable                 | Mensajeria_ACCESO                      |
|                          +----------------------------------------+
|                          | Contabilidad_ACCESO                    |
+--------------------------+----------------------------------------+

Y ya tenemos toda la estructura organizativa construida.

Prueba a pulsar el botón APLICACIONES que esta arriba a la derecha. Obtendrás accesos directos a
todas las aplicaciones del usuario “Anselmo Lorenzo”, que es con el que te encuentras registrado.
Verás que, además de la aplicación “Administración de la plataforma”, también aparece la
aplicación “Mensajería Interna”, tal y como se deduce de la política de credenciales.

El problema es que aunque hayamos registrado las aplicaciones, el código de estas, como tal, no
existe (a excepción de la de administración de la plataforma que es la que estás utilizando en estos
momentos). Por ello el próximo paso será desarrollar estas aplicaciones.

3.6. Despliegue y construcción de las aplicaciones
--------------------------------------------------

Ahora hay que construir el código de cada una de las aplicaciones. Es decir, construir el esqueleto de
cada aplicación y programarla. Esta operación se realiza con la tarea generate:appITE del *plugins
*symfonite* plugins.

Vas a comenzar construyendo la aplicación “Plataforma Educativa”. Como ya hemos dicho
anteriormente de desplegará como una aplicación *symfony* del mismo proyecto que alberga a la
aplicación de administración (cuyo controlador frontal hemos denominado backend).

.. note::

   Si la aplicación necesita nuevas tablas en la base de datos (que es lo normal) , deberíamos definir en el
   directorio config del proyecto, el schema que las modela. Como vamos a tener varias aplicaciones en el mismo
   proyecto, también tendremos varios schemas en este directorio, así que podemos llamarlo
   *platedu.schema.yml*. Además, si las tablas de esta base de datos van a pertenecer a otra base de datos
   distinta a la del núcleo, debemos añadir al fichero *config/databases.yml* la conexión a dicha base de datos. De
   todas formas esto es algo que podemos hacer más tarde. Aunque si se conocen qué tablas precisa la
   aplicación este sería un buen momento para definirlas en el schema.

Observarás en el listado de aplicaciones que aparece en cada una de las aplicaciones que acabas de
registrar una advertencia que dice que hay que generar el código. En la propia advertencia te indica
la instrucción que debes lanzar en el directorio raíz del proyecto a través de una *CLI*:

.. code-block:: sh

   symfony generate:appITE platedu --titulo='Plataforma Educativa' --clave=5caeb7411bd2d1f

.. note::

   La clave que aparece en la línea anterior se corresponde con la clave que asignó el sistema en que se
   desarrolló este tutorial en el momento del registro de la aplicación, y no coincidirá con la tuya. Debes mirar
   qué clave tiene asignada esa aplicación en tu sistema y colocarla en su lugar.

Esta operación

1. Genera una aplicación *symfony* con el nombre que se haya facilitado en el argumento
   (platedu).
2. Reconstruye el modelo, los formularios y los filtros de todo el proyecto (*plugin* y *aplicación*).
3. Modifica los templates de la aplicación (*layout.php*, *ventanaNueva.php* e *inicio.php*) para
   insertar el título de la aplicación (Plataforma Educativa).
4. Modifica el fichero *security.yml* para insertar la credencial de acceso de la aplicación.
5. Modifica el fichero *routing.yml* para insertar el módulo y la acción que se ejecutará cuando el
   usuario se ha registrado correctamente.
6. Modifica el fichero *app.yml* para insertar la clave (*5caeb7411bd2d1f*) que tenga asociada la
   aplicación.

Puedes comprobar todo esto mirando el código generado.
Las aplicaciones construidas con la tarea *generate:appITE* cuenta con un proceso de inicio de sesión
y con las siguientes operaciones (aparecen como links en la esquina superior derecha de cada
aplicación):

• *CAMBIAR DE PERFIL*, para cambiar el perfil y ámbito (si lo hubiese) con el que vas a utilizar la
  aplicación. A la derecha de cada perfil/ámbito, aparece un botón-radio que sirve para que el
  usuario seleccione (si lo desea) el perfil/ámbito por defecto con el que desea entrar en lo
  sucesivo.
• *CONFIGURACIÓN PERSONAL*, ofrece formularios para que el usuario pueda cambiar su nombre de
  usuario, su password y su idioma por defecto. También puede seleccionar el perfil/ámbito por
  defecto con el que desea entrar en lo sucesivo.
• *APLICACIONES*, presenta al usuario un listado con todas las aplicaciones a las que tiene acceso.
  Desde aquí el usuario puede lanzar sus aplicaciones.
• *y SALIR*. Para cerrar la sesión.

Puedes probar todo esto ingresando en la aplicación que acabas de crear, lo cual puedes hacer:

• Colocando la *URL* de la aplicación (http://localhost/LaMadraza/web/platedu_dev.php o algo
  así) en el navegador web, o
• Accediendo desde el listado de aplicaciones de la aplicación “Administración de la
  Plataforma”.

Debes introducir el login y password de un usuario que tenga perfiles en la aplicación para poder
ingresar en ella (por ejemplo, “Max Plack”). Una vez dentro puedes lanzar cualquier aplicación que el
usuario tenga accesible a través de sus perfiles usando el link APLICACIONES.

Ahora construye las aplicaciones “Contabilidad” y “Mensajería Interna” de la misma forma que has
construido la aplicación “Plataforma Educativa”. Una vez que tengas todo el sistema de aplicaciones
construido, prueba a “saltar” de una a otra aplicación usando el link APLICACIONES. Prueba también a
usar la funcionalidad SSO mediante *SAML* y *PAPI*.

Ya solo queda desarrollar las funcionalidades propias de cada aplicación, lo cual es la única (¡y ya es
bastante!) tarea del programador. Pero eso no entra dentro del alcance de este tutorial.

3.7. Los Menús de las aplicaciones
----------------------------------

Los menús de las aplicacines se construyen con la herramienta “Menús” que se encuentra en el
listado de aplicaciones.

Veamos como definir siguientes menús:

Alumno:
   • Curso
      - Actividades
      - Recursos
   • Seguimiento
      - Calificaciones
      - Notas del Profesor
Profesor:
   • Alumnos
      - Seguimiento
      - Consulta
   • Valoracion
      - del curso
      - del profesor
   • Recursos
      - FAQ
      - Enlaces externos

Para todos los menús hay que definir el item padre. Podemos llamarle “Inicio” y mapearlo a la acción
“inicio/index” que traen por defecto todas las aplicaciones *symfonite*.

+--------------------------+----------------------------------------+
| Campo                    | Valor                                  |
+==========================+========================================+
| Page                     | Inicio (puede llamarse como sea)       |
+--------------------------+----------------------------------------+
| Module                   | inicio (es el nombre del módulo que    |
|                          | dispara este menú)                     |
+--------------------------+----------------------------------------+
| Action                   | index (es el nombre de la acción que   |
|                          | dispara este menú)                     |
+--------------------------+----------------------------------------+
| Credential               | Lo dejaremos vacío, así todos los      |
|                          | perfiles lo verán                      |
+--------------------------+----------------------------------------+

Ahora vamos creando nuevos ítems seleccionando adecuadamente el menú padre y definiendo sus
atributos *page*, *module* y *action*. Por último debemos asignar una credencial (por ejemplo
*platedu_Alumno* que ya existe y está asociada al alumno) a los ítems del menú del alumno, y otra
distinta (por ejemplo *platedu_Docente* que ya existe y está asociada al profesor) a los ítems del
menú del profesor. Si no existieran las credenciales habría que crearlas y asociarlas a los perfiles.
Cuando vuelvas a entrar en la aplicación, aparecerá el nuevo menú.

3.8. Cambio de clave en la aplicación de administración
-------------------------------------------------------

Terminamos este tutorial indicando como hay que proceder para cambiar la clave de la aplicación de
administración. Por cuestiones de seguridad es importante cambiarla.

Editamos desde la aplicación núcleo el campo clave de la propia aplicación núcleo (utiliza el link
GENERAR CLAVE) Y después edita el fichero *apps/backend/config/app.yml* y cambiamos la línea 6: ::

   clave: cambiar

por ::

   clave: 73e7381956fad40 // La que te haya salido a tí, se comprende

Y ya está.


Créditos y versiones
====================

CREDITOS:
---------

- *Versión:* 1.0.0
- *Fecha:* 16/08/10
- *Autor/es:* Juan David Rodríguez García

MODIFICACIONES:
---------------

- *Versión:* 1.0.1
- *Fecha:* 15/10/10
- *Modificación:* Juan David Rodríguez

NOTAS:
------

.. [1] Código básico con funcionalidades comunes a todas las aplicaciones *symfonite*.
.. [2] http://simpleSAMLphp.org/
.. [3] https://forja.rediris.es/projects/phppoa/
.. [4] Ver la nota que encabeza este documento
.. [5] No hay que preocuparse si aparecen mensajes de alerta en rojo que dicen:
       Could not perform XLST transformation. Make sure PHP has been compiled/configured to support XSLT.
.. [6] Obviamente la URL dependerá de cómo y dónde se haya desplegado la aplicación. La cuestión es que hay
       que solicitar el recurso backend_dev.php que se encuentra en el directorio web del proyecto *symfony* que
       estamos desarrollando. Una aclaración más. En *symfony* se crean por defecto, para cada aplicación, dos
       controladores frontales, es decir dos puntos de entrada a la aplicación. Uno se utiliza para el entono de
       producción y el otro para el entorno de desarrollo. La diferencia fundamental es que si utilizamos el
       controlador de desarrollo para ejecutar la aplicación, se nos muestra en el navegador una barra de
       depuración con información muy útil para el desarrollo (y muy comprometida para un entorno de producción,
       hay que tener cuidado con esto). Utilizaremos en el tutorial el controlador de desarrollo (que termina en
       _dev.php)
.. [7] En un sistema linux, si no ejecutas esta tarea como usuario root puede que te aparezacan muchos warnings.
       No te preocupes no afectan al funcionamiento del sistema. Si te molestan ejecuta la acción con sudo.
.. [8] UOS significa Unidades Organizativas, y sirven para organizar los perfiles.
.. [9] Para que la ventana modal funcione hay que tener activado javascript en el navegador.
.. [10] Si intentas saltar desde la aplicación “catalogación” a la de “administración” el navegador mostrará el
        siguiente error: “Cannot redirect to an empty URL”. La razón es que la aplicación de administración fue 
        la primera que se instaló, y al desconocer el proceso de instalación laURL desde la que se accede a    
        ella, deja el campo URL vacío. Para que todo funcione correctamente basta que edites la aplicación  
        backend y le coloques la URL adecuada (http://localhost/sft/web/index.php en nuestro caso).
.. [11] Este tutorial aún no existe.
.. [12] https://openidp.feide.no/
