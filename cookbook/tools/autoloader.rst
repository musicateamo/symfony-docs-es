.. index::
   pair: Autocargador; Configurando

Cómo cargar clases automáticamente
==================================

Siempre que uses una clase no definida, PHP utiliza el mecanismo de carga automática para delegar la carga de un archivo de definición de clase. Symfony2 proporciona un cargador automático "universal", que es capaz de cargar clases desde los archivos que implementan uno de los siguientes convenios:

* Los `estándares`_ de interoperabilidad técnica para los espacios de nombres y nombres de clases de PHP 5.3;

* La convención de nomenclatura de clases de `PEAR`_.

Si tus clases y las bibliotecas de terceros que utilizas en tu proyecto siguen estas normas, el cargador automático de Symfony2 es el único cargador automático que necesitarás siempre.

Utilizando
----------

Registrar la clase del cargador automático :class:`Symfony\\Component\\ClassLoader\\UniversalClassLoader` es sencillo::

    require_once '/ruta/a/src/Symfony/Component/ClassLoader/UniversalClassLoader.php';

    use Symfony\Component\ClassLoader\UniversalClassLoader;

    $loader = new UniversalClassLoader();
    $loader->register();

Para una menor ganancia en rendimiento puedes memorizar en caché las rutas de las clases usando APC, con sólo registrar la clase :class:`Symfony\\Component\\ClassLoader\\ApcUniversalClassLoader`::

    require_once '/ruta/a/src/Symfony/Component/ClassLoader/UniversalClassLoader.php';
    require_once '/ruta/a/src/Symfony/Component/ClassLoader/ApcUniversalClassLoader.php';

    use Symfony\Component\ClassLoader\ApcUniversalClassLoader;

    $loader = new ApcUniversalClassLoader('apc.prefix.');
    $loader->register();

El cargador automático es útil sólo si agregas algunas bibliotecas al cargador automático.

.. note::

    El autocargador se registra automáticamente en una aplicación Symfony2 (consulta el ``app/autoload.php``).

Si las clases a cargar automáticamente utilizan espacios de nombres, utiliza cualquiera de los métodos :method:`Symfony\\Component\\ClassLoader\\UniversalClassLoader::registerNamespace` o :method:`Symfony\\Component\\ClassLoader\\UniversalClassLoader::registerNamespaces`::

    $loader->registerNamespace('Symfony', __DIR__.'/vendor/symfony/src');

    $loader->registerNamespaces(array(
        'Symfony' => __DIR__.'/../vendor/symfony/src',
        'Monolog' => __DIR__.'/../vendor/monolog/src',
    ));

Para las clases que siguen la convención de nomenclatura de PEAR, utiliza cualquiera de los métodos :method:`Symfony\\Component\\ClassLoader\\UniversalClassLoader::registerPrefix` o :method:`Symfony\\Component\\ClassLoader\\UniversalClassLoader::registerPrefixes`::

    $loader->registerPrefix('Twig_', __DIR__.'/vendor/twig/lib');

    $loader->registerPrefixes(array(
        'Swift_' => __DIR__.'/vendor/swiftmailer/lib/classes',
        'Twig_'  => __DIR__.'/vendor/twig/lib',
    ));

.. note::

    Algunas bibliotecas también requieren que su ruta de acceso raíz esté registrada en la ruta de include PHP (``set_include_path()``).

Las clases de un subespacio de nombres o una subjerarquía de clases PEAR se puede buscar en una lista de ubicaciones para facilitar el "vendoring" de un subconjunto de clases para los grandes proyectos::

    $loader->registerNamespaces(array(
        'Doctrine\\Common'           => __DIR__.'/vendor/doctrine-common/lib',
        'Doctrine\\DBAL\\Migrations' => __DIR__.'/vendor/doctrine-migrations/lib',
        'Doctrine\\DBAL'             => __DIR__.'/vendor/doctrine-dbal/lib',
        'Doctrine'                   => __DIR__.'/vendor/doctrine/lib',
    ));

En este ejemplo, si intentas utilizar una clase en el espacio de nombres ``Doctrine\Common`` o uno de sus hijos, el cargador automático buscará primero la clase en el directorio ``doctrine\common``, y entonces vuelve a la reserva al directorio predeterminado ``doctrine`` (el último configurado) si no se encuentra, antes de darse por vencido.
El orden de registro es importante en este caso.

.. _estándares: http://groups.google.com/group/php-standards/web/psr-0-final-proposal
.. _PEAR:      http://pear.php.net/manual/en/standards.php
