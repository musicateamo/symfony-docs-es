.. index::
   single: Buscador

Cómo localizar archivos
=======================

El componente :namespace:`Symfony\\Component\\Finder` te ayuda a buscar archivos y directorios rápida y fácilmente.

Utilizando
----------

La clase :class:`Symfony\\Component\\Finder\\Finder` busca archivos y/o
directorios::

    use Symfony\Component\Finder\Finder;

    $finder = new Finder();
    $finder->files()->in(__DIR__);

    foreach ($finder as $file) {
        print $file->getRealpath()."\n";
    }

``$file`` es una instancia de :phpclass:`SplFileInfo`.

El código anterior imprime recursivamente los nombres de todos los archivos en el directorio actual. La clase Finder utiliza una interfaz fluida, por lo que todos los métodos devuelven la instancia del Finder.

.. tip::

    Una instancia Finder es un `iterador`_ PHP. Por tanto, en lugar de iterar sobre el Finder con ``foreach``, también lo puedes convertir en una matriz con el método :phpfunction:`iterator_to_array` u obtener el número de elementos con :phpfunction:`iterator_count`.

Criterios
---------

Ubicación
~~~~~~~~~

La ubicación es el único criterio obligatorio. Este dice al buscador cual directorio utilizar para la búsqueda::

    $finder->in(__DIR__);

Busca en varios lugares encadenando llamadas al método :method:`Symfony\\Component\\Finder\\Finder::in`::

    $finder->files()->in(__DIR__)->in('/elsewhere');

Excluye directorios coincidentes con el método :method:`Symfony\\Component\\Finder\\Finder::exclude`::

    $finder->in(__DIR__)->exclude('ruby');

Debido a que Finder utiliza iteradores PHP, puedes pasar cualquier URL compatible con `protocolo`_::

    $finder->in('ftp://ejemplo.com/pub/');

Y también trabaja con flujos definidos por el usuario::

    use Symfony\Component\Finder\Finder;

    $s3 = new \Zend_Service_Amazon_S3($key, $secret);
    $s3->registerStreamWrapper("s3");

    $finder = new Finder();
    $finder->name('photos*')->size('< 100K')->date('since 1 hour ago');
    foreach ($finder->in('s3://bucket-name') as $file) {
        // hace algo

        print $file->getFilename()."\n";
    }

.. note::

    Lee la documentación de `Flujos`_ para aprender a crear tus propios flujos.

Archivos o directorios
~~~~~~~~~~~~~~~~~~~~~~

Por omisión, Finder devuelve archivos y directorios, pero lo controlan los métodos :method:`Symfony\\Component\\Finder\\Finder::files` y :method:`Symfony\\Component\\Finder\\Finder::directories`::

    $finder->files();

    $finder->directories();

Si quieres seguir los enlaces, utiliza el método ``followLinks()``::

    $finder->files()->followLinks();

De forma predeterminada, el iterador ignora archivos VCS populares. Esto se puede cambiar con el método ``ignoreVCS()``::

    $finder->ignoreVCS(false);

Ordenación
~~~~~~~~~~

Ordena el resultado por nombre o por tipo (primero directorios, luego archivos)::

    $finder->sortByName();

    $finder->sortByType();

.. note::

    Ten en cuenta que los métodos ``sort*`` necesitan obtener todos los elementos para hacer su trabajo. Para iteradores grandes, es lento.

También puedes definir tu propio algoritmo de ordenación con el método ``sort()``::

    $sort = function (\SplFileInfo $a, \SplFileInfo $b)
    {
        return strcmp($a->getRealpath(), $b->getRealpath());
    };

    $finder->sort($sort);

Nombre de archivo
~~~~~~~~~~~~~~~~~

Filtra archivos por nombre con el método
:method:`Symfony\\Component\\Finder\\Finder::name`::

    $finder->files()->name('*.php');

El método ``name()`` acepta globos, cadenas o expresiones regulares::

    $finder->files()->name('/\.php$/');

El método ``notNames()`` excluye archivos coincidentes con un patrón::

    $finder->files()->notName('*.rb');

Tamaño de archivo
~~~~~~~~~~~~~~~~~

Filtra archivos por tamaño con el método
:method:`Symfony\\Component\\Finder\\Finder::size`::

    $finder->files()->size('< 1.5K');

Filtra por rangos de tamaño encadenando llamadas a::

    $finder->files()->size('>= 1K')->size('<= 2K');

El operador de comparación puede ser cualquiera de los siguientes: ``>``, ``>=``, ``<``, '<=',
'=='.

El valor destino puede utilizar magnitudes de kilobytes (``k``, ``ki``), megabytes (``m``, ``mi``), o gigabytes (``g``, ``gi``). Los sufijos con una ``i`` usan la versión ``2**n`` adecuada de acuerdo al `estándar IEC`_.

Fecha de archivo
~~~~~~~~~~~~~~~~

Filtra archivos por fecha de última modificación con el método
:method:`Symfony\\Component\\Finder\\Finder::date`::

    $finder->date('since yesterday');

El operador de comparación puede ser cualquiera de los siguientes: ``>``, ``>=``, ``<``, '<=',
'=='. También puedes utilizar ``since`` o ``after`` como alias para ``>``, y ``until`` o ``before`` como alias para ``<``.

El valor destino puede ser cualquier fecha compatible con la función `strtotime`_.

Profundidad de directorio
~~~~~~~~~~~~~~~~~~~~~~~~~

De manera predeterminada, Finder recorre directorios recursivamente. Filtra la profundidad del recorrido con el método
:method:`Symfony\\Component\\Finder\\Finder::depth`::

    $finder->depth('== 0');
    $finder->depth('< 3');

Filtrado personalizado
~~~~~~~~~~~~~~~~~~~~~~

Para restringir que el archivo coincida con su propia estrategia, utiliza el método : :method:`Symfony\\Component\\Finder\\Finder::filter`::

    $filter = function (\SplFileInfo $file)
    {
        if (strlen($file) > 10) {
            return false;
        }
    };

    $finder->files()->filter($filter);

El método ``filter()`` toma un cierre como argumento. Para cada archivo coincidente, este es llamado con el archivo como una instancia de :phpclass:`SplFileInfo`. El archivo se excluye desde el conjunto de resultados si el cierre devuelve ``false``.

.. _strtotime:   http://www.php.net/manual/es/datetime.formats.php
.. _Iterador:     http://www.php.net/manual/es/spl.iterators.php
.. _protocolo:     http://www.php.net/manual/en/wrappers.php
.. _Flujos:      http://mx.php.net/streams
.. _estándar IEC: http://physics.nist.gov/cuu/Units/binary.html
