<!---
$"metadata"$
{
  "md": true,
  "title": "Crear bibliotecas no es un juego",
  "draft": true,
  "slug": "crear-bibliotecas-no-es-juego",
  "tags": [
    "desarrollo",
    "python-es",
    "bibliotecas"
  ]
}
$"metadata"$
-->

Al igual que muchas otras personas, estoy obsesionado con las [IPA](http://es.wikipedia.org/wiki/Interfaz_de_programaci%C3%B3n_de_aplicaciones) (API de ahora en adelante). Me encanta la consistencia, simplicidad y elegancia que puede ser aplicada a cada una de ellas. Lamentablemente, no siempre es así.

Durante las últimas tres semanas, he dedicado parte de mi tiempo al desarrollo de la biblioteca Python de [Marconi](https://wiki.openstack.org/wiki/Marconi) y pensé compartir parte de esta obsesión con ustedes. A lo largo de este post, explicaré algunas de las características principales de una buena API. Algunas de estás características tienen años de maduréz, otras nacieron como estándar común y algunas otras de mi propia opinión.

Consistencia
============

Consistencia, es decir, sin contradicciones ni ambigüedades. Este es uno de los puntos críticos y comúnmente omitidos en el desarrollo de APIs. Cada una de las funciones, métodos, clases y módulos expuestos a través de la API pública de la biblioteca, deben exponer una entrada - signatura - y una salida (siempre que sea posible) consistente. De igual manera, los nombres de los elementos de dicha API deben ser consistentes. Por ejemplo:

    def queue_create(transport, request, name, callback=None):
        ...

    def queue_set_metadata(transport, request, name, metadata, callback=None):
        ...

Ambas funciones exponen una signatura consistente y solo una pequeña porción de la misma - `metadata` - varía. Los nombres de ambas funciones son, a su vez, consistentes en su forma.

La consistencia de una API también puede ser evaluada en base a los tipos de objetos aceptados por los elementos expuestos. Por ejemplo:

    def create_file(archivo):
        """Crea archivo.

        :params archivo: Ruta del archivo
        :type archivo: unicode
        """

    def update_file(archivo):
        """Actualiza archivo.

        :params archivo: Instancia del nuevo archivo
        :type archivo: `file`
        """

En el primer caso de este ejemplo, la función (`create_file`) acepta un parámetro `archivo` de tipo `unicode` que representa la ruta del archivo que debe ser subido al servidor. En el segundo caso, la función (`update_file`) acepta un parámetro `archivo` pero de tipo `file`. En este ejemplo, la inconsistencia es bastante clara, ambas funciones tienen nombres y signaturas consistentes pero el tipo de objeto aceptado difiere. Este tipo de inconsistencias es el más común y uno de los que causa mayor frustración en los usuarios.

Se puede decir que es más fácil memorizar los nombres de las funciones que adivinar los tipos de objetos aceptados por estas y tener que 'cubrir' todos los casos posibles.

Mantener la consistencia a lo largo de la biblioteca ayuda a reducir la frustración de los usuarios y facilita el uso, aprendizaje y mantenimiento de la misma. Mantener la consistencia de la API ayuda a mejorar la simplicidad de esta.

Facilidad de Uso
================

`Simple es mejor que complejo. - El Zen de Python`

Tal cual. La API expuesta a través de la biblioteca debe ser simple y elegante. La complejidad de la API, dificultará la adopción de la biblioteca y cualquier implementación dependiente de esta.

El usuario de la biblioteca debe poder lograr ejecutar las acciones que desea en el menor número de pasos posibles. Sin embargo, esto no significa que la biblioteca debe tomar todas las decisiones por el usuario. La API debe exponer un nivel de abstracción que permita al usuario de la biblioteca decidir cuando 'tomar el control' y cuando dejarlo en manos de la misma.

Tomando como ejemplo la biblioteca de Marconi para Python, se puede notar que esta expone 2 niveles de abstracción.

    transport = http.HttpTransport(conf)
    request = request.prepare_request(conf, endpoint='http://localhost:8888')
    core.queue_create(transport, request, 'my_queue, callback=my_callback)

El primer nivel ([core](https://git.openstack.org/cgit/openstack/python-marconiclient/tree/marconiclient/queues/v1/core.py)) permite al usuario decidir que protocolo (transport) utilizar para dicha operación. De igual manera, permite al usuario construir su propio objeto `request` que a su vez requiere un objeto API que el mismo usuario tendrá que instanciar. Este primer nivel de abstracción, da al usuario el control **total** sobre los parámetros necesarios para el suceso de dicha acción.

El segundo nivel de abstracción, en cambio, expone una API mas simple e intuitiva con la cual el usuario podrá interactuar más fácilmente.

    cli = client.Client(URL)
    queue = cli.queue(queue_name)
    queue.post(messages)
    queue.delete()

En este ejemplo, se puede notar que cada uno de los objetos expresa algo y mantiene un contexto semántico a través del cual el usuario de la biblioteca puede intuir las acciones disponibles y los argumentos requeridos por cada una de ellas.


Retrocompatibilidad
===================

Fácil de decir, difícil de respetar. Probablemente uno de los puntos más críticos del diseño de APIs. La [retrocompatibilidad](http://es.wikipedia.org/wiki/Retrocompatibilidad) se refiere a la capacidad de una aplicación de utilizar datos generados y / o utilizados por versiones anteriores de esta. En pocas palabras, cuando un usuario pasa de la versión 0.1 a la versión 0.2 de tu biblioteca, esta debe seguir funcionando normalmente.

Nuevas versiones de tu biblioteca, muy probablemente, introducirán cambios en la API y en algunas de las funcionalidades. Ninguno de estos cambios debe, por ninguna razón, modificar el comportamiento de la biblioteca sin antes haber marcado el comportamiento actual como `obsoleto`. Es importante dar al usuario la opurtunidad de migrar su código a la nueva versión de la API una vez que esta esté estable.

Todos los cambios deben permitir al usuario decidir cuando migrar a la nueva version, sin romper la `ortogonalidad` de la aplicación de dicho usuario. Normalmente, cambios en el funcionamiento de la biblioteca y su API requieren una release de una versión 'mayor'.

En general, es más fácil agregar nuevas funcionalidades a una API que quitarlas, por ende, es recomendable no exponer nada a menos de que realmente no sea necesario. Una buena manera de hacer cumplir esto es siguiendo los lineamientos del principio [YAGNI](http://en.wikipedia.org/wiki/You_aren't_gonna_need_it). Implementa y/o publica las funcionalidades cuando realmente sea necesario.

Soporte para extensiones
========================

El soporte para extensiones no es exactamente un requisito. Dependiendo de la biblioteca y de lo que esta debe hacer, puede ser necesario y útil añadir soporte para extensiones. Las extensiones, en este caso, son pequeños componentes externos que serán cargados dinámicamente por dicha biblioteca con el fin de añadir funcionalidades específicas del usuario que no hacen parte del objetivo principal de la biblioteca.

Por ejemplo, Marconi tiene soporte para `transports`, es decir, es posible escribir un 'plugin' que añade suporte para un protocolo específico al servidor. Para poder usar dicho protocolo, es necesario que este sea soportado por la biblioteca y para ello, es necesario que la biblioteca permita a los usuarios cargar dinámicamente el componente que permitirá el uso de dicho protocolo.

Un caso más simple se puede ver en la biblioteca `python-request`. Esta permite a los usuarios implementar sus 'handlers' para el protocolo HTTP. Este tipo de soporte no requiere cargar dinámicamente un componente externo, en cambio, permite al usuario registrar su 'handler' si así lo desea.

Es importante saber diferenciar cuales funcionalidades hacen parte del objetivo principal de la biblioteca y cuales deben ser dejadas por fuera. El soporte para extensiones, como dicho anteriormente, no es un requisito obligatorio, sin embargo, es importante que la API no limite al usuario.

Una vez más, seguir los lineamentos del principio YAGNI puede ayudar a entender cuando el soporte para extensiones es necesario.

Pruebas unitarias
=================

No me detendré a explicar lo que son las [pruebas unitarias](http://es.wikipedia.org/wiki/Prueba_unitaria), pero si repetiré que toda aplicación debería tenerlas. ;)

Hace no mucho escribí un post acerca de las pruebas de código en general [[link](http://blog.flaper87.com/post/522b9e560f06d32542ede77f/)] en el cual explico algunas de las características de estas que, en mi opinión, están entre las más importantes. En este mismo post, menciono el hecho de que las pruebas unitarias no debe ser parte del paquete final. Esto quiere decir que una vez creado el paquete de la biblioteca, las pruebas no deben estar incluidas. Sin embargo, las clases abstractas para dichas pruebas, pueden hacer parte del paquete. Los invito a leer el post.