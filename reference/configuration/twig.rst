.. index::
   pair: Twig; Referencia de configuración

Referencia de configuración de TwigBundle
=========================================

.. configuration-block::

    .. code-block:: yaml

        twig:
            form:
                resources:

                    # predeterminado:
                    - div_base.html.twig

                    # Ejemplo:
                    - MyBundle::form.html.twig
            globals:

                # Ejemplos:
                foo:                 "@bar"
                pi:                  3.14

                # Prototipo
                key:
                    id:                   ~
                    type:                 ~
                    value:                ~
            autoescape:           ~
            base_template_class:  ~ # Ejemplo: Twig_Template
            cache:                %kernel.cache_dir%/twig
            charset:              %kernel.charset%
            debug:                %kernel.debug%
            strict_variables:     ~
            auto_reload:          ~
            exception_controller:  Symfony\Bundle\TwigBundle\Controller\ExceptionController::showAction

    .. code-block:: xml

        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:twig="http://symfony.com/schema/dic/twig"
            xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd
                                http://symfony.com/schema/dic/twig http://symfony.com/schema/dic/doctrine/twig-1.0.xsd">

            <twig:config auto-reload="%kernel.debug%" autoescape="true" base-template-class="Twig_Template" cache="%kernel.cache_dir%/twig" charset="%kernel.charset%" debug="%kernel.debug%" strict-variables="false">
                <twig:form>
                    <twig:resource>MyBundle::form.html.twig</twig:resource>
                </twig:form>
                <twig:global key="foo" id="bar" type="service" />
                <twig:global key="pi">3.14</twig:global>
            </twig:config>
        </container>

    .. code-block:: php

        $contenedor->loadFromExtension('twig', array(
            'form' => array(
                'resources' => array(
                    'MyBundle::form.html.twig',
                )
             ),
             'globals' => array(
                 'foo' => '@bar',
                 'pi'  => 3.14,
             ),
             'auto_reload'         => '%kernel.debug%',
             'autoescape'          => true,
             'base_template_class' => 'Twig_Template',
             'cache'               => '%kernel.cache_dir%/twig',
             'charset'             => '%kernel.charset%',
             'debug'               => '%kernel.debug%',
             'strict_variables'    => false,
        ));

Configurando
------------

exception_controller
....................

**tipo**: ``string`` **predeterminado**: ``Symfony\\Bundle\\TwigBundle\\Controller\\ExceptionController::showAction``

Este es el controlador que se activa después de una excepción en cualquier lugar de tu aplicación. El controlador predeterminado (:class:`Symfony\\Bundle\\TwigBundle\\Controller\\ExceptionController`) es el responsable de reproducir plantillas específicas en diferentes condiciones de error (consulta :doc:`/cookbook/controller/error_pages`). La modificación de esta opción es un tema avanzado. Si necesitas personalizar una página de error debes utilizar el enlace anterior. Si necesitas realizar algún comportamiento en una excepción, debes añadir un escucha para el evento ``kernel.exception`` (consulta :ref:`dic-tags-kernel-event-listener`).
