.. index::
   pair: Assetic; Referencia de configuración

Referencia de configuración de ``AsseticBundle``
================================================

Configuración predeterminada completa
-------------------------------------

.. configuration-block::

    .. code-block:: yaml

        assetic:
            debug:                true
            use_controller:       true
            read_from:            %kernel.root_dir%/../web
            write_to:             %assetic.read_from%
            java:                 /usr/bin/java
            node:                 /usr/bin/node
            sass:                 /usr/bin/sass
            bundles:

                # Predeterminados (todos los paquetes actualmente registrados):
                - FrameworkBundle
                - SecurityBundle
                - TwigBundle
                - MonologBundle
                - SwiftmailerBundle
                - DoctrineBundle
                - AsseticBundle
                - ...

            assets:

                # Prototipo
                name:
                    inputs:               []
                    filters:              []
                    options:

                        # Prototipo
                        name:                 []
            filters:

                # Prototipo
                name:                 []
            twig:
                functions:

                    # Prototipo
                    name:                 []
