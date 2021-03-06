Cómo trabajar con varios administradores de entidad
===================================================

En una aplicación Symfony2 puedes utilizar múltiples administradores de entidad. Esto es necesario si estás utilizando diferentes bases de datos e incluso vendedores con conjuntos de entidades totalmente diferentes. En otras palabras, un administrador de entidad que se conecta a una base de datos deberá gestionar algunas entidades, mientras que otro administrador de entidad conectado a otra base de datos puede manejar el resto.

.. note::

    Usar varios gestores de entidad es bastante fácil, pero más avanzado y generalmente no se requiere. Asegúrate de que realmente necesitas varios gestores de entidad antes de añadir complejidad a ese nivel.

El siguiente código de configuración muestra cómo puedes configurar dos gestores de entidad:

.. configuration-block::

    .. code-block:: yaml

        doctrine:
            orm:
                default_entity_manager:   default
                entity_managers:
                    default:
                        connection:       default
                        mappings:
                            AcmeDemoBundle: ~
                            AcmeGuardaBundle: ~
                    customer:
                        connection:       customer
                        mappings:
                            AcmeCustomerBundle: ~

En este caso, hemos definido dos gestores de entidad y los llamamos ``default`` y ``customer``. El gestor de entidad ``default`` administra cualquier entidad en los paquetes ``AcmeDemoBundle`` y ``AcmeGuardaBundle``, mientras que el gestor de entidad ``customer`` gestiona cualquiera en el paquete ``AcmeCustomerBundle``.

Cuando trabajas con tus administradores de entidad, entonces debes ser explícito acerca de cual gestor de entidad deseas. Si *no* omites el nombre del gestor de entidad al consultar por él, se devuelve el gestor de entidad predeterminado (es decir, ``default``)::

    class UserController extends Controller
    {
        public function indexAction()
        {
            // ambas devuelven el em "default"
            $em = $this->get('doctrine')->getEntityManager();
            $em = $this->get('doctrine')->getEntityManager('default');

            $customerEm =  $this->get('doctrine')->getEntityManager('customer');
        }
    }

Ahora puedes utilizar Doctrine tal como lo hiciste antes - con el gestor de entidad ``default`` para persistir y recuperar las entidades que gestiona y el gestor de entidad ``customer`` para persistir y recuperar sus entidades.