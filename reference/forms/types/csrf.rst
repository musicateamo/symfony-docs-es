.. index::
   single: Formularios; Campos; csrf

Tipo de campo ``csrf``
======================

El tipo ``csrf`` es un campo de entrada oculto que contiene un fragmento CSRF.

+-------------+--------------------------------------------------------------------+
| Reproducido | campo ``input`` ``hidden``                                         |
| como        |                                                                    |
+-------------+--------------------------------------------------------------------+
| Opciones    | - ``csrf_provider``                                                |
|             | - ``page_id``                                                      |
|             | - ``property_path``                                                |
+-------------+--------------------------------------------------------------------+
| Tipo del    | ``hidden``                                                         |
| padre       |                                                                    |
+-------------+--------------------------------------------------------------------+
| Clase       | :class:`Symfony\\Component\\Form\\Extension\\Csrf\\Type\\CsrfType` |
+-------------+--------------------------------------------------------------------+

Opciones del campo
~~~~~~~~~~~~~~~~~~

``csrf_provider``
~~~~~~~~~~~~~~~~~

**tipo**: ``Symfony\Component\Form\CsrfProvider\CsrfProviderInterface``

El objeto ``CsrfProviderInterface`` que debe generar la ficha CSRF.
Si no se establece, el valor predeterminado es el proveedor predeterminado.

intención
~~~~~~~~~

**tipo**: ``string``

Un opcional identificador único que se utiliza para generar la ficha CSRF.

.. include:: /reference/forms/types/options/property_path.rst.inc