.. index::
   single: Seguridad, Votantes

Cómo implementar tu propio votante para agregar direcciones IP a la lista negra
===============================================================================

El componente ``Security`` de Symfony2 ofrece varias capas para autenticar a los usuarios.
Una de las capas se llama ``voter`` ("votante" en adelante). Un votante o es una clase dedicada que comprueba si el usuario tiene el derecho a conectarse con la aplicación. Por ejemplo, Symfony2 proporciona una capa que comprueba si el usuario está plenamente autenticado o si se espera que tenga algún rol.

A veces es útil crear un votante personalizado para tratar un caso específico que la plataforma no maneja. En esta sección, aprenderás cómo crear un votante que te permitirá añadir usuarios a la lista negra por su IP.

La interfaz Votante
-------------------

Un votante personalizado debe implementar la clase :class:`Symfony\\Component\\Security\\Core\\Authorization\\Voter\\VoterInterface`, la cual requiere los tres siguientes métodos:

.. code-block:: php

    interface VoterInterface
    {
        function supportsAttribute($attribute);
        function supportsClass($class);
        function vote(TokenInterface $muestra, $object, array $attributes);
    }


El método ``supportsAttribute()`` se utiliza para comprobar si el votante admite el atributo usuario dado (es decir: un rol, una :abbr:`acl ("access control list", en adelante: lista de control de acceso)`, etc.).

El método ``supportsClass()`` se utiliza para comprobar si el votante apoya a la clase simbólica usuario actual.

El método ``vote()`` debe implementar la lógica del negocio que verifica cuando o no se concede acceso al usuario. Este método debe devolver uno de los siguientes valores:

* ``VoterInterface::ACCESS_GRANTED``: El usuario puede acceder a la aplicación
* ``VoterInterface::ACCESS_ABSTAIN``: El votante no puede decidir si se concede acceso al usuario o no
* ``VoterInterface::ACCESS_DENIED``: El usuario no está autorizado a acceder a la aplicación

En este ejemplo, vamos a comprobar si la dirección IP del usuario coincide con una lista de direcciones en la lista negra. Si la IP del usuario está en la lista negra, devolveremos ``VoterInterface::ACCESS_DENIED``, de lo contrario devolveremos ``VoterInterface::ACCESS_ABSTAIN`` porque la finalidad del votante sólo es para negar el acceso, no para permitir el acceso.

Creando un votante personalizado
--------------------------------

Para poner a un usuario en la lista negra basándonos en su IP, podemos utilizar el servicio ``Petición`` y comparar la dirección IP contra un conjunto de direcciones IP en la lista negra:

.. code-block:: php

    namespace Acme\DemoBundle\Security\Authorization\Voter;

    use Symfony\Component\HttpFoundation\Request;
    use Symfony\Component\Security\Core\Authorization\Voter\VoterInterface;
    use Symfony\Component\Security\Core\Authentication\Token\TokenInterface;

    class ClientIpVoter implements VoterInterface
    {
        public function __construct(Request $peticion, array $blacklistedIp = array())
        {
            $this->request       = $peticion;
            $this->blacklistedIp = $blacklistedIp;
        }

        public function supportsAttribute($attribute)
        {
            // no vamos a verificar contra un atributo de usuario, por lo
            // tanto devuelve true
            return true;
        }

        public function supportsClass($class)
        {
            // nuestro votante apoya todo tipo de clases, por lo que
            // devuelve true
            return true;
        }

        function vote(TokenInterface $muestra, $object, array $attributes)
        {
            if (in_array($this->request->getClientIp(), $this->blacklistedIp)) {
                return VoterInterface::ACCESS_DENIED;
            }

            return VoterInterface::ACCESS_ABSTAIN;
        }
    }

¡Eso es todo! El votante está listo. El siguiente paso es inyectar el votante en el nivel de seguridad. Esto se puede hacer fácilmente a través del contenedor de servicios.

Declarando el votante como servicio
-----------------------------------

Para inyectar al votante en la capa de seguridad, se debe declarar como servicio, y la etiqueta como "security.voter":

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/AcmeBundle/Resources/config/services.yml

        services:
            security.access.blacklist_voter:
                class:      Acme\DemoBundle\Security\Authorization\Voter\ClientIpVoter
                arguments:  [@request, [123.123.123.123, 171.171.171.171]]
                public:     false
                tags:
                    -       { name: security.voter }

    .. code-block:: xml

        <!-- src/Acme/AcmeBundle/Resources/config/services.xml -->

        <service id="security.access.blacklist_voter"
                 class="Acme\DemoBundle\Security\Authorization\Voter\ClientIpVoter" public="false">
            <argument type="service" id="request" strict="false" />
            <argument type="collection">
                <argument>123.123.123.123</argument>
                <argument>171.171.171.171</argument>
            </argument>
            <tag name="security.voter" />
        </service>

    .. code-block:: php

        // src/Acme/AcmeBundle/Resources/config/services.php

        use Symfony\Component\DependencyInjection\Definition;
        use Symfony\Component\DependencyInjection\Reference;

        $definicion = new Definition(
            'Acme\DemoBundle\Security\Authorization\Voter\ClientIpVoter',
            array(
                new Reference('request'),
                array('123.123.123.123', '171.171.171.171'),
            ),
        );
        $definicion->addTag('security.voter');
        $definicion->setPublic(false);

        $contenedor->setDefinition('security.access.blacklist_voter', $definicion);

.. tip::

   Asegúrate de importar este archivo de configuración desde el archivo de configuración principal de tu aplicación (por ejemplo, ``app/config/config.yml``). Para más información, consulta :ref:`service-container-imports-directive`. Para leer más acerca de definir los servicios en general, consulta el capítulo :doc:`/book/service_container`.

Cambiando la estrategia de decisión de acceso
---------------------------------------------

A fin de que los cambios del nuevo votante tengan efecto, tenemos que cambiar la estrategia de decisión de acceso predeterminada, que, por omisión, concede el acceso si *cualquier* votante permite el acceso.

En nuestro caso, vamos a elegir la estrategia ``unánime``. A diferencia de la estrategia ``afirmativa`` (predeterminada), con la estrategia ``unánime``, aunque un votante sólo niega el acceso (por ejemplo, el ``ClientIpVoter``), no otorga acceso al usuario final.

Para ello, sustituye la sección ``access_decision_manager`` predeterminada del archivo de configuración de tu aplicación con el siguiente código.

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            access_decision_manager:
                # Strategy puede ser: affirmative, unanimous o consensus
                strategy: unanimous

¡Eso es todo! Ahora, a la hora de decidir si un usuario debe tener acceso o no, el nuevo votante deniega el acceso a cualquier usuario en la lista negra de direcciones IP.