.. index::
   single: Formularios; Campos; password

Tipo de campo password
======================

El tipo de campo ``password`` reproduce un campo de texto para entrada de contraseñas.

+-------------+------------------------------------------------------------------------+
| Reproducido | campo ``input`` ``password``                                           |
| como        |                                                                        |
+-------------+------------------------------------------------------------------------+
| Opciones    | - `always_empty`_                                                      |
+-------------+------------------------------------------------------------------------+
| Opciones    | - `max_length`_                                                        |
| heredadas   | - `required`_                                                          |
|             | - `label`_                                                             |
|             | - `trim`_                                                              |
|             | - `read_only`_                                                         |
|             | - `error_bubbling`_                                                    |
+-------------+------------------------------------------------------------------------+
| Tipo del    | :doc:`text</reference/forms/types/text>`                               |
| padre       |                                                                        |
+-------------+------------------------------------------------------------------------+
| Clase       | :class:`Symfony\\Component\\Form\\Extension\\Core\\Type\\PasswordType` |
+-------------+------------------------------------------------------------------------+

Opciones del campo
~~~~~~~~~~~~~~~~~~

``always_empty``
~~~~~~~~~~~~~~~~

**tipo**: ``Boolean`` **predeterminado**: ``true``

Si es ``true``, el campo *siempre* se reproduce en blanco, incluso si el campo correspondiente tiene un valor. Cuando se establece en ``false``, el campo de la contraseña se reproduce con el atributo ``value`` fijado a su valor real.

En pocas palabras, si por alguna razón deseas reproducir tu campo de contraseña *con* el valor de contraseña ingresado anteriormente en el cuadro, ponlo a ``false``.

Opciones heredadas
------------------

Estas opciones las hereda del tipo :doc:`field </reference/forms/types/field>`:

.. include:: /reference/forms/types/options/max_length.rst.inc

.. include:: /reference/forms/types/options/required.rst.inc

.. include:: /reference/forms/types/options/label.rst.inc

.. include:: /reference/forms/types/options/trim.rst.inc

.. include:: /reference/forms/types/options/read_only.rst.inc

.. include:: /reference/forms/types/options/error_bubbling.rst.inc