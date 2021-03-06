.. index::
   single: Formularios; Campos; file

Tipo de campo ``file``
======================

El tipo ``file`` representa una entrada de archivo en tu formulario.

+-------------+---------------------------------------------------------------------+
| Reproducido | campo ``input`` ``file``                                            |
| como        |                                                                     |
+-------------+---------------------------------------------------------------------+
| Opciones    | - `required`_                                                       |
| heredadas   | - `label`_                                                          |
|             | - `read_only`_                                                      |
|             | - `error_bubbling`_                                                 |
+-------------+---------------------------------------------------------------------+
| Typo del    | :doc:`form</reference/forms/types/field>`                           |
| padre       |                                                                     |
+-------------+---------------------------------------------------------------------+
| Clase       | :class:`Symfony\\Component\\Form\\Extension\\Core\\Type\\FileType`  |
+-------------+---------------------------------------------------------------------+

Uso básico
----------

Digamos que tienes esta definición de formulario:

.. code-block:: php

    $builder->add('attachment', 'file');

.. caution::

    No olvides añadir el atributo ``enctype`` en la etiqueta del formulario: ``<form action="#" method="post" {{ form_enctype(form) }}>``.

Cuando se envía el formulario, el campo de datos adjuntos (``attchment``) será una instancia de :class:`Symfony\\Component\\HttpFoundation\\File\\UploadedFile`. Este lo puedes utilizar para mover el archivo ``adjunto`` a una ubicación permanente:

.. code-block:: php

    use Symfony\Component\HttpFoundation\File\UploadedFile;

    public function uploadAction()
    {
        // ...

        if ($formulario->isValid()) {
            $algúnNuevoNombreDeArchivo = ...

            $formulario['attachment']->move($dir, $algúnNuevoNombreDeArchivo);

            // ...
        }

        // ...
    }

El método ``move()`` toma un directorio y un nombre de archivo como argumentos.
Puedes calcular el nombre de archivo en una de las siguientes formas::

    // usa el nombre de archivo original
    $file->move($dir, $file->getClientOriginalName());

    // calcula un nombre aleatorio e intenta adivinar la extensión (más seguro)
    $extension = $file->guessExtension();
    if (!$extension) {
        // no puede adivinar la extensión
        $extension = 'bin';
    }
    $file->move($dir, rand(1, 99999).'.'.$extension);

Usar el nombre original a través de ``getClientOriginalName()`` no es seguro, ya que el usuario final lo podría haber manipulado. Además, puede contener caracteres que no están permitidos en nombres de archivo. Antes de usar el nombre directamente, lo debes desinfectar.

Lee en el recetario el :doc:`ejemplo </cookbook/doctrine/file_uploads>` de cómo manejar la carga de archivos adjuntos con una entidad Doctrine.

Opciones heredadas
------------------

Estas opciones las hereda del tipo :doc:`field </reference/forms/types/field>`:

.. include:: /reference/forms/types/options/required.rst.inc

.. include:: /reference/forms/types/options/label.rst.inc

.. include:: /reference/forms/types/options/read_only.rst.inc

.. include:: /reference/forms/types/options/error_bubbling.rst.inc