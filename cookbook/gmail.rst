.. index::
   single: Correo electrónico; Gmail

Cómo utilizar Gmail para enviar mensajes de correo electrónico
==============================================================

Durante el desarrollo, en lugar de utilizar un servidor SMTP regular para enviar mensajes de correo electrónico, verás que es más fácil y más práctico utilizar Gmail. El paquete SwiftMailer hace que sea muy fácil.

.. tip::

    En lugar de utilizar tu cuenta normal de Gmail, por supuesto, recomendamos crear una cuenta especial.

En el archivo de configuración de desarrollo, cambia el ajuste ``transporte`` a ``gmail`` y establece el ``nombre de usuario`` y ``contraseña``  a las credenciales de Google:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config_dev.yml
        swiftmailer:
            transport: gmail
            username:  your_gmail_username
            password:  your_gmail_password

    .. code-block:: xml

        <!-- app/config/config_dev.xml -->

        <!--
        xmlns:swiftmailer="http://symfony.com/schema/dic/swiftmailer"
        http://symfony.com/schema/dic/swiftmailer http://symfony.com/schema/dic/swiftmailer/swiftmailer-1.0.xsd
        -->

        <swiftmailer:config
            transport="gmail"
            username="your_gmail_username"
            password="your_gmail_password" />

    .. code-block:: php

        // app/config/config_dev.php
        $contenedor->loadFromExtension('swiftmailer', array(
            'transport' => "gmail",
            'username'  => "your_gmail_username",
            'password'  => "your_gmail_password",
        ));

¡Ya está!

.. note::

    El transporte ``gmail`` simplemente es un acceso directo que utiliza el transporte ``smtp`` y establece ``encryption``, ``auth_mode`` y ``host``  para trabajar con Gmail.
