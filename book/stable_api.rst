API estable de Symfony2
=======================

La API estable de Symfony2 es un subconjunto de todos los métodos públicos de Symfony2 (componentes y paquetes básicos) que comparten las siguientes propiedades:

* El nombre del espacio de nombres y clase no va a cambiar;
* El nombre del método no va a cambiar;
* La firma del método (argumentos y tipo del valor de retorno) no va a cambiar;
* La semántica del método no va a cambiar.

Sin embargo, la imprementación en sí misma puede cambiar. El único caso válido para un cambio en la API estable es con el fin de corregir un problema de seguridad.

La API estable se basa en una lista blanca, marcada con ``@api``. Por lo tanto, todo lo no etiquetado explícitamente no es parte de la API estable.

.. tip::

    Cualquier paquete de terceros también deberá publicar su propia API estable.

A partir de Symfony 2.0, los siguientes componentes tienen una API etiquetada pública:

* BrowserKit
* ClassLoader
* Console
* CssSelector
* DependencyInjection
* DomCrawler
* EventDispatcher
* Finder
* HttpFoundation
* HttpKernel
* Locale
* Process
* Routing
* Templating
* Translation
* Validator
* Yaml
