.. index::
   single: Formularios; Campos; money

Tipo de campo ``money``
=======================

Reproduce un campo de entrada de texto especializado en el manejo de la presentación de datos tipo "moneda".

Este tipo de campo te permite especificar una moneda, cuyo símbolo se representa al lado del campo de texto. También hay otras opciones para personalizar la forma de la entrada y salida de los datos manipulados.

+-------------+---------------------------------------------------------------------+
| Reproducido | campo ``input`` ``text``                                            |
| como        |                                                                     |
+-------------+---------------------------------------------------------------------+
| Opciones    | - `currency`_                                                       |
|             | - `divisor`_                                                        |
|             | - `precision`_                                                      |
|             | - `grouping`_                                                       |
+-------------+---------------------------------------------------------------------+
| Opciones    | - `required`_                                                       |
| heredadas   | - `label`_                                                          |
|             | - `read_only`_                                                      |
|             | - `error_bubbling`_                                                 |
+-------------+---------------------------------------------------------------------+
| Tipo del    | :doc:`field</reference/forms/types/field>`                          |
| padre       |                                                                     |
+-------------+---------------------------------------------------------------------+
| Clase       | :class:`Symfony\\Component\\Form\\Extension\\Core\\Type\\MoneyType` |
+-------------+---------------------------------------------------------------------+

Opciones del campo
~~~~~~~~~~~~~~~~~~

``currency``
~~~~~~~~~~~~

**tipo**: ``string`` **predeterminado**: ``EUR``

Especifica la moneda en la cual se especifica el dinero. Esta determina el símbolo de moneda que se debe mostrar en el cuadro de texto. Dependiendo de la moneda - el símbolo de moneda se puede mostrar antes o después del campo de entrada de texto.
    
También lo puedes establecer a ``false`` para ocultar el símbolo de moneda.

``divisor``
~~~~~~~~~~~

**tipo**: ``integer`` **predeterminado**: ``1``

Si, por alguna razón, tienes que dividir tu valor inicial por un número antes de reproducirlo para el usuario, puedes utilizar la opción ``divisor``.
Por ejemplo::

    $builder->add('precio', 'money', array(
        'divisor' => 100,
    ));

En este caso, si el campo ``precio`` está establecido en ``9900``, entonces en realidad al usuario se le presentará el valor ``99``. Cuando el usuario envía el valor ``99``, este se multiplicará por ``100`` y finalmente se devolverá ``9900`` a tu objeto.

``precision``
~~~~~~~~~~~~~~

**tipo**: ``integer`` **predeterminado**: ``2``

Por alguna razón, si necesitas alguna precisión que no sean dos decimales, puedes modificar este valor. Probablemente no necesitarás hacer esto a menos que, por ejemplo, desees redondear al dólar más cercano (ajustando la precisión a ``0``).

.. include:: /reference/forms/types/options/grouping.rst.inc

Opciones heredadas
------------------

Estas opciones las hereda del tipo :doc:`field </reference/forms/types/field>`:

.. include:: /reference/forms/types/options/required.rst.inc

.. include:: /reference/forms/types/options/label.rst.inc

.. include:: /reference/forms/types/options/read_only.rst.inc

.. include:: /reference/forms/types/options/error_bubbling.rst.inc
