.. index::
   single: Contenedor de servicios
   single: Contenedor de inyección de dependencias

Contenedor de servicios
=======================

Una moderna aplicación PHP está llena de objetos. Un objeto te puede facilitar la entrega de mensajes de correo electrónico, mientras que otro te puede permitir mantener información en una base de datos. En tu aplicación, puedes crear un objeto que gestiona tu inventario de productos, u otro objeto que procesa los datos de una API de terceros. El punto es que una aplicación moderna hace muchas cosas y está organizada en muchos objetos que se encargan de cada tarea.

En este capítulo, vamos a hablar de un objeto PHP especial en Symfony2 que te ayuda a crear una instancia, organizar y recuperar muchos objetos de tu aplicación.
Este objeto, llamado contenedor de servicios, te permitirá estandarizar y centralizar la forma en que se construyen los objetos en tu aplicación. El contenedor te facilita la vida, es superveloz, y enfatiza una arquitectura que promueve el código reutilizable y disociado. Y como todas las clases Symfony2 básicas usan el contenedor, aprenderás cómo ampliar, configurar y utilizar cualquier objeto en Symfony2. En gran parte, el contenedor de servicios es el mayor contribuyente a la velocidad y extensibilidad de Symfony2.

Por último, configurar y usar el contenedor de servicios es fácil. Al final de este capítulo, te sentirás cómodo creando tus propios objetos y personalizando objetos de cualquier paquete de terceros a través del contenedor. Empezarás a escribir código más reutilizable, comprobable y disociado, simplemente porque el contenedor de servicios facilita la escritura de buen código.

.. index::
   single: Contenedor de servicios; ¿Qué es un servicio?

¿Qué es un servicio?
--------------------

En pocas palabras, un :term:`Servicio` es cualquier objeto PHP que realiza algún tipo de tarea "global". Es un nombre genérico que se utiliza a propósito en informática para describir un objeto creado para un propósito específico (por ejemplo, la entrega de mensajes de correo electrónico). Cada servicio se utiliza en toda tu aplicación cada vez que necesites la funcionalidad específica que proporciona. No tienes que hacer nada especial para hacer un servicio: simplemente escribe una clase PHP con algo de código que realiza una tarea específica. ¡Felicidades, acabas de crear un servicio!

.. note::

    Por regla general, un objeto PHP es un servicio si se utiliza a nivel global en tu aplicación. Un solo servicio ``Mailer`` se utiliza a nivel global para enviar mensajes de correo electrónico mientras que muchos objetos ``Mensaje`` que entrega no son *servicios*. Del mismo modo, un objeto ``producto`` no es un servicio, sino un objeto ``producto`` que persiste objetos a una base de datos *es* un servicio.

Entonces, ¿cuál es la ventaja? La ventaja de pensar en "servicios" es que comienzas a pensar en la separación de cada parte de la funcionalidad de tu aplicación como una serie de servicios. Puesto que cada servicio se limita a un trabajo, puedes acceder fácilmente a cada servicio y usar su funcionalidad siempre que la necesites. Cada servicio también se puede probar y configurar más fácilmente, ya que está separado de la otra funcionalidad de tu aplicación. Esta idea se llama `arquitectura orientada a servicios`_ y no es única de Symfony2 e incluso de PHP. Estructurando tu aplicación en torno a un conjunto de clases *Servicio* independientes es una bien conocida y confiable práctica mejor orientada a objetos. Estas habilidades son clave para ser un buen desarrollador en casi cualquier lenguaje.

.. index::
   single: Contenedor de servicios; ¿Qué es?

¿Qué es un contenedor de servicios?
-----------------------------------

Un  :term:`Contenedor de servicios` (o *contenedor de inyección de dependencias*) simplemente es un objeto PHP que gestiona la creación de instancias de servicios (es decir, objetos).
Por ejemplo, supongamos que tenemos una clase PHP simple que envía mensajes de correo electrónico.
Sin un contenedor de servicios, debemos crear manualmente el objeto cada vez que lo necesitemos:

.. code-block:: php

    use Acme\HolaBundle\Mailer;

    $cartero = new Mailer('sendmail');
    $cartero->send('ryan@foobar.net', ... );

Esto es bastante fácil. La clase imaginaria ``Mailer`` nos permite configurar el método utilizado para entregar los mensajes de correo electrónico (por ejemplo, ``sendmail``, ``smtp``, etc.)
¿Pero qué si queremos utilizar el servicio cliente de correo en algún otro lugar? Desde luego, no queremos repetir la configuración del gestor de correo *cada* vez que tenemos que utilizar el objeto ``Mailer``. ¿Qué pasa si necesitamos cambiar el ``transporte`` de ``sendmail`` a ``smtp`` en todas partes en la aplicación? Necesitaríamos cazar todos los lugares que crean un servicio ``Mailer`` y modificarlo.

.. index::
   single: Contenedor de servicios; Configurando servicios

Creando/configurando servicios en el contenedor
-----------------------------------------------

Una mejor respuesta es dejar que el contenedor de servicios cree el objeto ``Mailer`` para ti. Para que esto funcione, debemos *enseñar* al contenedor cómo crear el servicio ``Mailer``. Esto se hace a través de configuración, la cual se puede especificar en YAML, XML o PHP:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        services:
            mi_cartero:
                class:        Acme\HolaBundle\Mailer
                arguments:    [sendmail]

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <services>
            <service id="mi_cartero" class="Acme\HolaBundle\Mailer">
                <argument>sendmail</argument>
            </service>
        </services>

    .. code-block:: php

        // app/config/config.php
        use Symfony\Component\DependencyInjection\Definition;

        $contenedor->setDefinition('mi_cartero', new Definition(
            'Acme\HolaBundle\Mailer',
            array('sendmail')
        ));

.. note::

    Cuando se inicia, por omisión Symfony2 construye el contenedor de servicios usando la configuración de (``app/config/config.yml``). El archivo exacto que se carga es dictado por el método ``AppKernel::registerContainerConfiguration()``, el cual carga un archivo de configuración específico al entorno (por ejemplo, ``config_dev.yml`` para el entorno ``dev`` o ``config_prod.yml`` para ``prod``).

Una instancia del objeto ``Acme\HolaBundle\Mailer`` ahora está disponible a través del contenedor de servicios. El contenedor está disponible en cualquier controlador tradicional de Symfony2, desde donde puedes acceder al servicio del contenedor a través del método ``get()``::

    class HolaController extends Controller
    {
        // ...

        public function sendEmailAction()
        {
            // ...
            $cartero = $this->get('mi_cartero');
            $cartero->send('ryan@foobar.net', ... );
        }
    }

Cuando pedimos el servicio ``mi_cartero`` desde el contenedor, el contenedor construye el objeto y lo devuelve. Esta es otra de las principales ventajas de utilizar el contenedor de servicios. Es decir, un servicio *nunca* es construido hasta que es necesario. Si defines un servicio y no lo utilizas en una petición, el servicio no se crea. Esto ahorra memoria y aumenta la velocidad de tu aplicación.
Esto también significa que la sanción en rendimiento por definir muchos servicios es muy poca o ninguna. Los servicios que nunca se usan nunca se construyen.

Como bono adicional, el servicio ``Mailer`` se crea sólo una vez y se vuelve a utilizar la misma instancia cada vez que solicites el servicio. Este casi siempre es el comportamiento que tendrá (el cual es más flexible y potente), pero vamos a aprender más adelante cómo puedes configurar un servicio que tiene varias instancias.

.. _book-service-container-parameters:

Parámetros del servicio
-----------------------

La creación de nuevos servicios (es decir, objetos) a través del contenedor es bastante sencilla. Los parámetros provocan que al definir los servicios estén más organizados y sean más flexibles:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        parameters:
            mi_cartero.class:      Acme\HolaBundle\Mailer
            mi_cartero.transport:  sendmail

        services:
            mi_cartero:
                class:        %mi_cartero.class%
                arguments:    [%mi_cartero.transport%]

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <parameters>
            <parameter key="mi_cartero.class">Acme\HolaBundle\Mailer</parameter>
            <parameter key="mi_cartero.transport">sendmail</parameter>
        </parameters>

        <services>
            <service id="mi_cartero" class="%mi_cartero.class%">
                <argument>%mi_cartero.transport%</argument>
            </service>
        </services>

    .. code-block:: php

        // app/config/config.php
        use Symfony\Component\DependencyInjection\Definition;

        $contenedor->setParameter('mi_cartero.class', 'Acme\HolaBundle\Mailer');
        $contenedor->setParameter('mi_cartero.transport', 'sendmail');

        $contenedor->setDefinition('mi_cartero', new Definition(
            '%mi_cartero.class%',
            array('%mi_cartero.transport%')
        ));

El resultado final es exactamente igual que antes - la diferencia es sólo en *cómo* definimos el servicio. Al rodear las cadenas ``mi_cartero.class`` y ``mi_cartero.transport`` entre signos de porcentaje (``%``), el contenedor sabe que tiene que buscar los parámetros con esos nombres. Cuando se construye el contenedor, este busca el valor de cada parámetro y lo utiliza en la definición del servicio.

El propósito de los parámetros es alimentar información a los servicios. Por supuesto no había nada malo en la definición del servicio sin utilizar ningún parámetro.
Los parámetros, sin embargo, tienen varias ventajas:

* Separan y organizan todo el servicio en "opciones" bajo una sola clave ``parameters``;

* Los valores del parámetro se pueden utilizar en la definición de múltiples servicios;

* Cuando creas un servicio en un paquete (vamos a mostrar esto en breve), utilizar parámetros permite que el servicio sea fácil de personalizar en tu aplicación.

La opción de usar o no parámetros depende de ti. Los paquetes de alta calidad de terceros *siempre* usan parámetros, ya que producen servicios almacenados en el contenedor más configurables. Para los servicios de tu aplicación, sin embargo, posiblemente no necesites la flexibilidad de los parámetros.

Importando la configuración de recursos desde otros contenedores
----------------------------------------------------------------

.. tip::

    En esta sección, nos referiremos a los archivos de configuración de servicios como *recursos*.
    Se trata de resaltar el hecho de que, si bien la mayoría de la configuración de recursos debe estar en archivos (por ejemplo, YAML, XML, PHP), Symfony2 es tan flexible que la configuración se puede cargar desde cualquier lugar (por ejemplo, una base de datos e incluso a través de un servicio web externo).

El contenedor de servicios se construye usando un recurso de configuración simple (``app/config/config.yml`` por omisión). Toda la configuración de otros servicios (incluido el núcleo de Symfony2 y la configuración de paquetes de terceros) se debe importar desde el interior de este archivo en una u otra forma. Esto proporciona absoluta flexibilidad sobre los servicios en tu aplicación.

La configuración externa de servicios se puede importar de dos maneras diferentes. En primer lugar, vamos a hablar sobre el método que utilizarás con más frecuencia en tu aplicación: la Directiva ``imports``. En la siguiente sección, vamos a introducir el segundo método, que es el método flexible y preferido para importar la configuración del servicio desde paquetes de terceros.

.. index::
   single: Contenedor de servicios; imports

.. _service-container-imports-directive:

Importando configuración con ``imports``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Hasta ahora, hemos puesto nuestra definición del contenedor del servicio ``mi_cartero`` directamente en el archivo de configuración de la aplicación (por ejemplo, ``app/config/config.yml``). Por supuesto, debido a que la clase ``Mailer`` vive dentro de ``AcmeHolaBundle``, tiene más sentido poner la definición del contenedor de ``mi_cartero`` en el paquete también.

En primer lugar, mueve la definición del contenedor de ``mi_cartero`` a un nuevo archivo contenedor de recursos dentro ``AcmeHolaBundle``. Si los directorios ``Resourses`` y ``Resourses/config`` no existen, créalos.

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/HolaBundle/Resources/config/services.yml
        parameters:
            mi_cartero.class:      Acme\HolaBundle\Mailer
            mi_cartero.transport:  sendmail

        services:
            mi_cartero:
                class:        %mi_cartero.class%
                arguments:    [%mi_cartero.transport%]

    .. code-block:: xml

        <!-- src/Acme/HolaBundle/Resources/config/services.xml -->
        <parameters>
            <parameter key="mi_cartero.class">Acme\HolaBundle\Mailer</parameter>
            <parameter key="mi_cartero.transport">sendmail</parameter>
        </parameters>

        <services>
            <service id="mi_cartero" class="%mi_cartero.class%">
                <argument>%mi_cartero.transport%</argument>
            </service>
        </services>

    .. code-block:: php

        // src/Acme/HolaBundle/Resources/config/services.php
        use Symfony\Component\DependencyInjection\Definition;

        $contenedor->setParameter('mi_cartero.class', 'Acme\HolaBundle\Mailer');
        $contenedor->setParameter('mi_cartero.transport', 'sendmail');

        $contenedor->setDefinition('mi_cartero', new Definition(
            '%mi_cartero.class%',
            array('%mi_cartero.transport%')
        ));

La propia definición no ha cambiado, sólo su ubicación. Por supuesto, el contenedor de servicios no sabe sobre el nuevo archivo de recursos. Afortunadamente, es fácil importar el archivo de recursos utilizando la clave ``imports`` en la configuración de la aplicación.

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        imports:
            hola_bundle:
                resource: @AcmeHolaBundle/Resources/config/services.yml

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <imports>
            <import resource="@AcmeHolaBundle/Resources/config/services.xml"/>
        </imports>

    .. code-block:: php

        // app/config/config.php
        $this->import('@AcmeHolaBundle/Resources/config/services.php');

La directiva ``imports`` permite a tu aplicación incluir recursos de configuración del contenedor de servicios de cualquier otro lugar (comúnmente desde paquetes).
La ubicación de ``resourses``, para archivos, es la ruta absoluta al archivo de recursos. La sintaxis especial ``@AcmeHola`` resuelve la ruta al directorio del paquete ``AcmeHolaBundle``. Esto te ayuda a especificar la ruta a los recursos sin tener que preocuparte más adelante de si se mueve el ``AcmeHolaBundle`` a un directorio diferente.

.. index::
   single: Contenedor de servicios; Configurando extensiones

.. _service-container-extension-configuration:

Importando configuración vía extensiones del contenedor
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Cuando desarrollas en Symfony2, comúnmente debes usar la directiva ``imports`` para importar la configuración del contenedor desde los paquetes que haz creado específicamente para tu aplicación. La configuración del paquete contenedor de terceros, incluyendo los servicios básicos de Symfony2, normalmente se cargan con cualquier otro método que sea más flexible y fácil de configurar en tu aplicación.

Así es como funciona. Internamente, cada paquete define sus servicios muy parecido a lo que hemos visto hasta ahora. Es decir, un paquete utiliza uno o más archivos de configuración de recursos (por lo general XML) para especificar los parámetros y servicios para ese paquete. Sin embargo, en lugar de importar cada uno de estos recursos directamente desde la configuración de tu aplicación utilizando la directiva ``imports``, sólo tienes que invocar una *extensión contenedora de servicios* dentro del paquete, la cual hace el trabajo por ti. Una extensión del contenedor de servicios es una clase PHP creada por el autor del paquete para lograr dos cosas:

* Importar todos los recursos del contenedor de servicios necesarios para configurar los servicios del paquete;

* permite una configuración semántica y directa para poder configurar el paquete sin interactuar con los parámetros de configuración planos del paquete contenedor del servicio.

En otras palabras, una extensión del contenedor de servicios configura los servicios para un paquete en tu nombre. Y como veremos en un momento, la extensión proporciona una interfaz sensible y de alto nivel para configurar el paquete.

Tomemos el ``FrameworkBundle`` - el núcleo de la plataforma Symfony2 - como ejemplo. La presencia del siguiente código en la configuración de tu aplicación invoca a la extensión en el interior del contenedor de servicios ``FrameworkBundle``:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        framework:
            secret:          xxxxxxxxxx
            charset:         UTF-8
            form:            true
            csrf_protection: true
            router:        { resource: "%kernel.root_dir%/config/routing.yml" }
            # ...

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <framework:config charset="UTF-8" secret="xxxxxxxxxx">
            <framework:form />
            <framework:csrf-protection />
            <framework:router resource="%kernel.root_dir%/config/routing.xml" />
            <!-- ... -->
        </framework>

    .. code-block:: php

        // app/config/config.php
        $contenedor->loadFromExtension('framework', array(
            'secret'          => 'xxxxxxxxxx',
            'charset'         => 'UTF-8',
            'form'            => array(),
            'csrf-protection' => array(),
            'router'          => array('resource' => '%kernel.root_dir%/config/routing.php'),
            // ...
        ));

Cuando se analiza la configuración, el contenedor busca una extensión que pueda manejar la directiva de configuración ``framework``. La extensión en cuestión, que vive en el ``FrameworkBundle``, es invocada y cargada la configuración del servicio para el ``FrameworkBundle``. Si quitas la clave ``framework`` del archivo de configuración de tu aplicación por completo, no se cargarán los servicios básicos de Symfony2. El punto es que tú tienes el control: la plataforma Symfony2 no contiene ningún tipo de magia o realiza cualquier acción en que tú no tengas el control.

Por supuesto que puedes hacer mucho más que simplemente "activar" la extensión del contenedor de servicios del ``FrameworkBundle``. Cada extensión te permite personalizar fácilmente el paquete, sin tener que preocuparte acerca de cómo se definen los servicios internos.

En este caso, la extensión te permite personalizar el juego de caracteres - ``charset``, gestor de errores - ``error_handler``, protección CSRF - ``csrf_protection``, configuración del ruteador - ``router`` - y mucho más. Internamente, el ``FrameworkBundle`` utiliza las opciones especificadas aquí para definir y configurar los servicios específicos del mismo. El paquete se encarga de crear todos los ``parámetros`` y ``servicios`` necesarios para el contenedor de servicios, mientras permite que la mayor parte de la configuración se pueda personalizar fácilmente. Como bono adicional, la mayoría de las extensiones del contenedor de servicios también son lo suficientemente inteligentes como para realizar la validación - notificándote opciones omitidas o datos de tipo incorrecto.

Al instalar o configurar un paquete, consulta la documentación del paquete de cómo se deben instalar y configurar los servicios para el paquete. Las opciones disponibles para los paquetes básicos se pueden encontrar dentro de la :doc:`Guía de Referencia </reference/index>`.

.. note::

   De forma nativa, el contenedor de servicios sólo reconoce las directivas ``parameters``, ``services`` e ``imports``. Cualquier otra directiva es manejada por una extensión del contenedor de servicios.

.. index::
   single: Contenedor de servicios; Refiriendo servicios

Refiriendo (inyectando) servicios
---------------------------------

Hasta el momento, nuestro servicio original ``mi_cartero`` es simple: sólo toma un argumento en su constructor, el cual es fácilmente configurable. Como verás, el poder real del contenedor se realiza cuando es necesario crear un servicio que depende de uno o varios otros servicios en el contenedor.

Comencemos con un ejemplo. Supongamos que tenemos un nuevo servicio, ``BoletinGestor``, que ayuda a gestionar la preparación y entrega de un mensaje de correo electrónico a una colección de direcciones. Por supuesto el servicio ``mi_cartero`` ya es realmente bueno en la entrega de mensajes de correo electrónico, así que lo usaremos dentro de ``BoletinGestor`` para manejar la entrega real de los mensajes. Se pretende que esta clase pudiera ser algo como esto::

    namespace Acme\HolaBundle\Boletin;

    use Acme\HolaBundle\Mailer;

    class BoletinGestor
    {
        protected $cartero;

        public function __construct(Mailer $cartero)
        {
            $this->cartero = $cartero;
        }

        // ...
    }

Sin utilizar el contenedor de servicios, podemos crear un nuevo ``BoletinGestor`` muy fácilmente desde el interior de un controlador::

    public function sendNewsletterAction()
    {
        $cartero = $this->get('mi_cartero');
        $boletin = new Acme\HolaBundle\Boletin\BoletinGestor($cartero);
        // ...
    }

Este enfoque está bien, pero, ¿si más adelante decidimos que la clase ``BoletinGestor`` necesita un segundo o tercer argumento constructor? ¿Y si nos decidimos a reconstruir nuestro código y cambiar el nombre de la clase? En ambos casos, habría que encontrar todos los lugares donde se crea una instancia de ``BoletinGestor`` y modificarla. Por supuesto, el contenedor de servicios nos da una opción mucho más atractiva:

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/HolaBundle/Resources/config/services.yml
        parameters:
            # ...
            boletin_gestor.class: Acme\HolaBundle\Boletin\BoletinGestor

        services:
            mi_cartero:
                # ...
            boletin_gestor:
                class:     %boletin_gestor.class%
                arguments: [@mi_cartero]

    .. code-block:: xml

        <!-- src/Acme/HolaBundle/Resources/config/services.xml -->
        <parameters>
            <!-- ... -->
            <parameter key="boletin_gestor.class">Acme\HolaBundle\Boletin\BoletinGestor</parameter>
        </parameters>

        <services>
            <service id="mi_cartero" ... >
              <!-- ... -->
            </service>
            <service id="boletin_gestor" class="%boletin_gestor.class%">
                <argument type="service" id="mi_cartero"/>
            </service>
        </services>

    .. code-block:: php

        // src/Acme/HolaBundle/Resources/config/services.php
        use Symfony\Component\DependencyInjection\Definition;
        use Symfony\Component\DependencyInjection\Reference;

        // ...
        $contenedor->setParameter('boletin_gestor.class', 'Acme\HolaBundle\Boletin\BoletinGestor');

        $contenedor->setDefinition('mi_cartero', ... );
        $contenedor->setDefinition('boletin_gestor', new Definition(
            '%boletin_gestor.class%',
            array(new Reference('mi_cartero'))
        ));

En YAML, la sintaxis especial ``@mi_cartero`` le dice al contenedor que busque un servicio llamado ``mi_cartero`` y pase ese objeto al constructor de ``BoletinGestor``. En este caso, sin embargo, el servicio especificado ``mi_cartero`` debe existir. Si no es así, lanzará una excepción. Puedes marcar tus dependencias como opcionales - explicaremos esto en la siguiente sección.

La utilización de referencias es una herramienta muy poderosa que te permite crear clases de servicios independientes con dependencias bien definidas. En este ejemplo, el servicio ``boletin_gestor`` necesita del servicio ``mi_cartero`` para poder funcionar. Al definir esta dependencia en el contenedor de servicios, el contenedor se encarga de todo el trabajo de crear instancias de objetos.

Dependencias opcionales: Inyección de definidores
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Inyectar dependencias en el constructor de esta manera es una excelente manera de asegurarte que la dependencia está disponible para usarla. Si tienes dependencias opcionales para una clase, entonces, la "inyección de definidor" puede ser una mejor opción. Esto significa inyectar la dependencia usando una llamada a método en lugar de a través del constructor. La clase se vería así::

    namespace Acme\HolaBundle\Boletin;

    use Acme\HolaBundle\Mailer;

    class BoletinGestor
    {
        protected $cartero;

        public function setCartero(Mailer $cartero)
        {
            $this->cartero = $cartero;
        }

        // ...
    }

La inyección de la dependencia por medio del método definidor sólo necesita un cambio de sintaxis:

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/HolaBundle/Resources/config/services.yml
        parameters:
            # ...
            boletin_gestor.class: Acme\HolaBundle\Boletin\BoletinGestor

        services:
            mi_cartero:
                # ...
            boletin_gestor:
                class:     %boletin_gestor.class%
                calls:
                    - [ setCartero, [ @mi_cartero ] ]

    .. code-block:: xml

        <!-- src/Acme/HolaBundle/Resources/config/services.xml -->
        <parameters>
            <!-- ... -->
            <parameter key="boletin_gestor.class">Acme\HolaBundle\Boletin\BoletinGestor</parameter>
        </parameters>

        <services>
            <service id="mi_cartero" ... >
              <!-- ... -->
            </service>
            <service id="boletin_gestor" class="%boletin_gestor.class%">
                <call method="setCartero">
                     <argument type="service" id="mi_cartero" />
                </call>
            </service>
        </services>

    .. code-block:: php

        // src/Acme/HolaBundle/Resources/config/services.php
        use Symfony\Component\DependencyInjection\Definition;
        use Symfony\Component\DependencyInjection\Reference;

        // ...
        $contenedor->setParameter('boletin_gestor.class', 'Acme\HolaBundle\Boletin\BoletinGestor');

        $contenedor->setDefinition('mi_cartero', ... );
        $contenedor->setDefinition('boletin_gestor', new Definition(
            '%boletin_gestor.class%'
        ))->addMethodCall('setCartero', array(
            new Reference('mi_cartero')
        ));

.. note::

    Los enfoques presentados en esta sección se llaman "inyección de constructor" e "inyección de definidor". El contenedor de servicios de Symfony2 también es compatible con la "inyección de propiedad".

Haciendo referencias opcionales
-------------------------------

A veces, uno de tus servicios puede tener una dependencia opcional, lo cual significa que la dependencia no es necesaria para que el servicio funcione correctamente. En el ejemplo anterior, el servicio ``mi_cartero`` *debe* existir, si no, será lanzada una excepción. Al modificar la definición del servicio ``boletin_gestor``, puedes hacer opcional esta referencia. Entonces, el contenedor será inyectado si es que existe y no hace nada si no:

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/HolaBundle/Resources/config/services.yml
        parameters:
            # ...

        services:
            boletin_gestor:
                class:     %boletin_gestor.class%
                arguments: [@?mi_cartero]

    .. code-block:: xml

        <!-- src/Acme/HolaBundle/Resources/config/services.xml -->

        <services>
            <service id="mi_cartero" ... >
              <!-- ... -->
            </service>
            <service id="boletin_gestor" class="%boletin_gestor.class%">
                <argument type="service" id="mi_cartero" on-invalid="ignore" />
            </service>
        </services>

    .. code-block:: php

        // src/Acme/HolaBundle/Resources/config/services.php
        use Symfony\Component\DependencyInjection\Definition;
        use Symfony\Component\DependencyInjection\Reference;
        use Symfony\Component\DependencyInjection\ContainerInterface;

        // ...
        $contenedor->setParameter('boletin_gestor.class', 'Acme\HolaBundle\Boletin\BoletinGestor');

        $contenedor->setDefinition('mi_cartero', ... );
        $contenedor->setDefinition('boletin_gestor', new Definition(
            '%boletin_gestor.class%',
            array(new Reference('mi_cartero', ContainerInterface::IGNORE_ON_INVALID_REFERENCE))
        ));

En YAML, la sintaxis especial ``@?`` le dice al contenedor de servicios que la dependencia es opcional. Por supuesto, ``BoletinGestor`` también se debe escribir para permitir una dependencia opcional:

.. code-block:: php

        public function __construct(Mailer $cartero = null)
        {
            // ...
        }

El núcleo de Symfony y servicios en un paquete de terceros
----------------------------------------------------------

Puesto que Symfony2 y todos los paquetes de terceros configuran y recuperan sus servicios a través del contenedor, puedes acceder fácilmente a ellos e incluso utilizarlos en tus propios servicios. Para mantener las cosas simples, de manera predeterminada Symfony2 no requiere que los controladores se definan como servicios. Además Symfony2 inyecta el contenedor de servicios completo en el controlador. Por ejemplo, para manejar el almacenamiento de información sobre la sesión de un usuario, Symfony2 proporciona un servicio ``sesión``, al cual se puede acceder dentro de un controlador estándar de la siguiente manera::

    public function indexAction($bar)
    {
        $sesion = $this->get('sesion');
        $sesion->set('foo', $bar);

        // ...
    }

En Symfony2, constantemente vas a utilizar los servicios prestados por el núcleo de Symfony o paquetes de terceros para realizar tareas como la reproducción de plantillas (``templating``), el envío de mensajes de correo electrónico (``mailer``), o para acceder a información sobre la petición.

Podemos dar un paso más allá usando estos servicios dentro de los servicios que haz creado para tu aplicación. Vamos a modificar el ``BoletinGestor`` para usar el gestor de correo real de Symfony2, el servicio ``mailer`` (en vez del pretendido ``mi_cartero``).
También vamos a pasar el servicio del motor de plantillas al ``BoletinGestor`` para que puedas generar el contenido del correo electrónico a través de una plantilla::

    namespace Acme\HolaBundle\Boletin;

    use Symfony\Component\Templating\EngineInterface;

    class BoletinGestor
    {
        protected $cartero;

        protected $plantilla;

        public function __construct(\Swift_Mailer $cartero, EngineInterface $plantilla)
        {
            $this->cartero = $cartero;
            $this->plantilla = $plantilla;
        }

        // ...
    }

Configurar el contenedor de servicios es fácil:

.. configuration-block::

    .. code-block:: yaml

        services:
            boletin_gestor:
                class:     %boletin_gestor.class%
                arguments: [@mailer, @templating]

    .. code-block:: xml

        <service id="boletin_gestor" class="%boletin_gestor.class%">
            <argument type="service" id="mailer"/>
            <argument type="service" id="templating"/>
        </service>

    .. code-block:: php

        $contenedor->setDefinition('boletin_gestor', new Definition(
            '%boletin_gestor.class%',
            array(
                new Reference('mailer'),
                new Reference('templating')
            )
        ));

El servicio ``boletin_gestor`` ahora tiene acceso a los servicios del núcleo ``mailer`` y ``templating``. Esta es una forma común de crear servicios específicos para tu aplicación que aprovechan el poder de los distintos servicios en la plataforma.

.. tip::

    Asegúrate de que la entrada ``SwiftMailer`` aparece en la configuración de la aplicación. Como mencionamos en :ref:`service-container-extension-configuration`, la clave ``SwiftMailer`` invoca a la extensión de servicio desde ``SwiftmailerBundle``, la cual registra el servicio ``mailer``.

.. index::
   single: Contenedor de servicios; Configuración avanzada

Configuración avanzada del contenedor
-------------------------------------

Como hemos visto, definir servicios dentro del contenedor es fácil, generalmente implica una clave de configuración ``service`` y algunos parámetros. Sin embargo, el contenedor tiene varias otras herramientas disponibles que ayudan a *etiquetar* servicios por funcionalidad especial, crear servicios más complejos y realizar operaciones después de que el contenedor está construido.

Marcando servicios como público / privado
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Cuando definas servicios, generalmente, querrás poder acceder a estas definiciones dentro del código de tu aplicación. Estos servicios se llaman ``public``. Por ejemplo, el servicio ``doctrine`` registrado en el contenedor cuando se utiliza ``DoctrineBundle`` es un servicio público al que puedes acceder a través de::

   $doctrine = $contenedor->get('doctrine');

Sin embargo, hay casos de uso cuando no quieres que un servicio sea público. Esto es común cuando sólo se define un servicio, ya que se podría utilizar como argumento para otro servicio.

.. note::

    Si utilizas un servicio privado como argumento a más de otro servicio, esto se traducirá en dos diferentes instancias utilizadas como la creación del servicio privado realizada en línea (por ejemplo, ``new PrivateFooBar()``).

Dice simplemente: El servicio será privado cuando no deseas acceder a él directamente desde tu código.

Aquí está un ejemplo:

.. configuration-block::

    .. code-block:: yaml

        services:
           foo:
             class: Acme\HolaBundle\Foo
             public: false

    .. code-block:: xml

        <service id="foo" class="Acme\HolaBundle\Foo" public="false" />

    .. code-block:: php

        $definicion = new Definition('Acme\HolaBundle\Foo');
        $definicion->setPublic(false);
        $contenedor->setDefinition('foo', $definicion);

Ahora que el servicio es privado, *no* puedes llamar a::

    $contenedor->get('foo');

Sin embargo, si haz marcado un servicio como privado, todavía puedes asignarle un alias (ve más abajo) para acceder a este servicio (a través del alias).

.. note::

   Los servicios por omisión son públicos.

Rebautizando
~~~~~~~~~~~~

Cuando utilizas el núcleo o paquetes de terceros dentro de tu aplicación, posiblemente desees utilizar métodos abreviados para acceder a algunos servicios. Puedes hacerlo rebautizándolos y, además, puedes incluso rebautizar servicios no públicos.

.. configuration-block::

    .. code-block:: yaml

        services:
           foo:
             class: Acme\HolaBundle\Foo
           bar:
             alias: foo

    .. code-block:: xml

        <service id="foo" class="Acme\HolaBundle\Foo"/>

        <service id="bar" alias="foo" />

    .. code-block:: php

        $definicion = new Definition('Acme\HolaBundle\Foo');
        $contenedor->setDefinition('foo', $definicion);

        $containerBuilder->setAlias('bar', 'foo');

Esto significa que cuando utilizas el contenedor directamente, puedes acceder al servicio ``foo`` al pedir el servicio ``bar`` así::

    $contenedor->get('bar'); // podrías devolver el servicio foo

Incluyendo archivos
~~~~~~~~~~~~~~~~~~~

Puede haber casos de uso cuando necesites incluir otro archivo justo antes de cargar el servicio en sí. Para ello, puedes utilizar la directiva ``file``.

.. configuration-block::

    .. code-block:: yaml

        services:
           foo:
             class: Acme\HolaBundle\Foo\Bar
             file: %kernel.root_dir%/src/ruta/al/archivo/foo.php

    .. code-block:: xml

        <service id="foo" class="Acme\HolaBundle\Foo\Bar">
            <file>%kernel.root_dir%/src/rita/al/archivo/foo.php</file>
        </service>

    .. code-block:: php

        $definicion = new Definition('Acme\HolaBundle\Foo\Bar');
        $definicion->setFile('%kernel.root_dir%/src/ruta/al/archivo/foo.php');
        $contenedor->setDefinition('foo', $definicion);

Ten en cuenta que internamente Symfony llama a la función PHP ``require_once``, lo cual significa que el archivo se incluirá una sola vez por petición.

.. _book-service-container-tags:

Etiquetas (``tags``)
~~~~~~~~~~~~~~~~~~~~

De la misma manera que en la Web una entrada de blog se puede etiquetar con cosas tales como "Symfony" o "PHP", los servicios configurados en el contenedor también se pueden etiquetar. En el contenedor de servicios, una etiqueta implica que el servicio está destinado a usarse para un propósito específico. Tomemos el siguiente ejemplo:

.. configuration-block::

    .. code-block:: yaml

        services:
            foo.twig.extension:
                class: Acme\HolaBundle\Extension\FooExtension
                tags:
                    -  { name: twig.extension }

    .. code-block:: xml

        <service id="foo.twig.extension" class="Acme\HolaBundle\Extension\FooExtension">
            <tag name="twig.extension" />
        </service>

    .. code-block:: php

        $definicion = new Definition('Acme\HolaBundle\Extension\FooExtension');
        $definicion->addTag('twig.extension');
        $contenedor->setDefinition('foo.twig.extension', $definicion);

La etiqueta ``twig.extension`` es una etiqueta especial que ``TwigBundle`` usa
durante la configuración. Al dar al servicio esta etiqueta ``twig.extension``, el paquete sabe que el servicio ``foo.twig.extension`` se debe registrar como una extensión Twig con Twig. En otras palabras, Twig encuentra todos los servicios con la etiqueta ``twig.extension`` y automáticamente los registra como extensiones.

Las etiquetas, entonces, son una manera de decirle a Symfony2 u otros paquetes de terceros que tu servicio se debe registrar o utilizar de alguna forma especial por el paquete.

La siguiente es una lista de etiquetas disponibles con los paquetes del núcleo de Symfony2.
Cada una de ellas tiene un efecto diferente en tu servicio y muchas etiquetas requieren argumentos adicionales (más allá de sólo el parámetro ``nombre``).

* assetic.filter
* assetic.templating.php
* data_collector
* form.field_factory.guesser
* kernel.cache_warmer
* kernel.event_listener
* monolog.logger
* routing.loader
* security.listener.factory
* security.voter
* templating.helper
* twig.extension
* translation.loader
* validator.constraint_validator

Aprende más en el recetario
---------------------------

* :doc:`/cookbook/service_container/factories`
* :doc:`/cookbook/service_container/parentservices`
* :doc:`/cookbook/controller/service`

.. _`arquitectura orientada a servicios`: http://wikipedia.org/wiki/Service-oriented_architecture
..
