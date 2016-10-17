#Proyecto de la asignatura de interfaces de usuario, parte de iOS básico

Este "miniproyecto" trata de hacer una pequeña aplicación basándose en [el API](developer.marvel.com) de la editorial Marvel, que nos permite consultar información sobre los personajes, los *comics*, las *series*,...

##Pasos iniciales (0.5 puntos)

###Registro en la API de Marvel

Lo primero que necesitas es [registrarte](https://secure.marvel.com/user/register?referer=https%3A%2F%2Fdeveloper.marvel.com%2Faccount) en Marvel para poder obtener un par de claves. Este paso es necesario porque la API tiene un número máximo de llamadas diario, lo que significa que no podemos compartir todos las mismas claves.

###Acceso a la API 

La API de Marvel es REST, por lo que acepta peticiones HTTP. No obstante hacerlas directamente con los APIs de iOS sería un poco engorroso, por lo que vamos a usar un par de librerías que nos faciliten no solo hacer la petición en sí sino sobre todo *parsear* el JSON, que en Swift es algo tedioso. Usaremos un par de librerías adicionales:

- Marvelous: que encapsula las llamadas al API en una serie de clases de modo que no tenemos que hacer 



####Instalación de las librerías auxiliares

Por desgracia Xcode no integra ningún sistema de gestión de dependencias de librerías de terceros, así que acudiremos a una herramienta que no es de Apple pero que se ha convertido en un estándar *de facto* en el mundo iOS: [*CocoaPods*](http://cocoapods.org).

Cocoapods es a la vez un repositorio de librerías y un gestor de dependencias para instalar automáticamente estas librerías en nuestros proyectos. Hay muchas librerías de terceros disponibles con este sistema, puedes buscarlas desde la página de CocoaPods.

Para instalar `cocoapods`, desde la terminal hacer

```bash
#Con --pre decimos que queremos instalar la última versión, aunque sea RC
sudo gem install cocoapods  --pre
```

> Esto instala la herramienta desde un repositorio de Internet, así que  necesitarás conectividad...y paciencia, según vaya la red.

Si todo va bien se instalará un comando llamado `pod`. Ejecútalo desde la terminal para comprobar al menos que existe. Ahora debes seguir estos pasos:

1. Crear un proyecto Xcode para la aplicación. Llámalo por ejemplo `Marvel`
2. Con un editor de textos cualquiera, crear un fichero llamado `Podfile` en el directorio del proyecto (el que contiene el fichero `.xcodeproj`). Este archivo debe contener la configuración y las dependencias (o *pods*) del proyecto

```bash
platform :ios, '10.0'
use_frameworks!
#SUSTITUYE 'Marvel' por el nombre de tu proyecto, si no se llama así!!!
target 'Marvel' do
   pod 'Alamofire','~> 4.0'
   pod 'Marvelous'
end
```

5. Abre una terminal. Muévete hasta el directorio donde está el `Podfile` y desde él ejecuta el comando `pod install`. Las dependencias de nuestro proyecto se bajarán automáticamente y se creará en el directorio actual un `Marvel.xcworkspace`
6. **A partir de ahora para trabajar en el proyecto siempre abriremos el fichero Marvel.xcworkspace**, que es un *workspace* de Xcode (un conjunto de proyectos), no el proyecto Marvel directamente (**NO ABRAS DIRECTAMENTE el `Marvel.xcodeproj`**).
7. Veremos que en Xcode se muestra nuestro proyecto y además un proyecto adicional llamado `Pods`, que contiene las dependencias. Este último no lo tocaremos, pero es necesario que esté ahí para que funcione el nuestro.

> ANTES de empezar a escribir código asegúrate de hacer en Xcode un `Product > Clean`, y `Product > Build` para asegurarse de que las dependencias están compiladas y accesibles en nuestro código.

###Ejemplo de uso de `Marvelous`

> Para poder hacer llamadas al API de Marvel necesitas un par de claves. Las puedes ver, una vez dado de alta y autentificado en Marvel, en `https://developer.marvel.com/account`

Para probar de manera sencilla la librería `Marvelous` puedes poner este `import` en el `ViewController`

```swift
import Marvelous
```

Y ahora colocar el siguiente código en el `viewDidLoad()` y comprobar que se muestran en la consola todos los personajes cuyo nombre comienza por "spider".

```swift
let marvelAPI = RCMarvelAPI()
//CAMBIA ESTO PARA PONER TUS CLAVES!!!!!
marvelAPI.publicKey = "a6927e7e15930110aade56ef90244f6d"
marvelAPI.privateKey = "487b621fc3c0d6f128b468ba86c99c508f24d357"
let filtro = RCCharacterFilter()
filtro.nameStartsWith = "spider"
marvelAPI.characters(by: filtro) {
    resultados, info, error in
    if let personajes = resultados as! [RCCharacterObject]? {
        for personaje in personajes {
            print(personaje.name)
        }
        print("Hay \(personajes.count) personajes")
    }
}
```

##Estructura general de la interfaz

Para que te hagas una idea de la estructura, se muestra el *storyboard* de la aplicación ya terminada

![](storyboard.png)

En la aplicación se podrá buscar, listar y mostrar los detalles de alguno de los recursos que ofrece el API. Elige tú lo que prefieras: personajes, comics, creadores... 

## Pantalla inicial

Puedes comenzar creando un *tab bar* a partir de la pantalla inicial: selecciónala y en el menú `Editor` elige `Embed In > Tab Bar Controller`.

##Vista maestro (2 puntos)

Esta debe ser una pantalla con una barra de búsqueda y una vista de tabla en la que se puede buscar y listar el recurso elegido (por ejemplo personajes). El resultado final será algo como:

![](img/maestro.png)


Ve a la primera (y por el momento única) pantalla de contenido del *tab bar*, selecciona la barra inferior y en las propiedades `bar item` ponle un título apropiado (por ejemplo "Personajes") y un icono relevante.


> Recuerda que tienes unas cuantas webs de donde puedes coger iconos "planos", por ejemplo [https://www.iconfinder.com/iconsets/ios-7-icons](https://www.iconfinder.com/iconsets/ios-7-icons) o [http://www.flaticon.com/packs/line-icon-set](http://www.flaticon.com/packs/line-icon-set)



###Crear los componentes de la interfaz

La tabla:

- Arrastra una *table view* a la pantalla de "personajes" (o "comics", o lo que hayas elegido)
- Selecciona la tabla y en las propiedades crea un prototipo de celda incrementando el `Prototype cells` a 1.
- Selecciona el prototipo de celda recién creado (el `table view cell`) y en las propiedades elige como `style` el valor `Basic`, para poder usar uno de los tipos predefinidos de celda y no tener que hacerlo tú.

La barra de búsqueda:

- Arrastra un componente de tipo *search bar* a la pantalla y déjalo justo arriba de la tabla.

Nos falta fijar el *autolayout*:

- Selecciona barra y tabla manteniendo pulsada la tecla `Cmd` e inserta ambas en un *stack view* utilizando el primero de los botones de *autolayout* (parte inferior derecha del *storyboard)
- Haz que este *stack view* ocupe toda la pantalla. Lo más sencillo es ponerle 4 restricciones de distancia 0 a los bordes.

Una vez hecho esto puedes poner en marcha la *app* para ver si la interfaz tiene buen aspecto. La tabla aparecerá vacía, por supuesto.

###Gestionar la barra de búsqueda

La barra de búsqueda utiliza el patrón *delegación* para gestionar los eventos de escribir en ella, pulsar "buscar" en el teclado, etc.

El objeto que actúe como *delegate* de la barra de búsqueda debe implementar el protocolo `UISearchBarDelegate`. Puede ser cualquiera, aunque siguiendo la filosofía MVC (Monster View Controller) haremos que sea el *view controller* de la pantalla actual :).

Vamos a crear este *view controller*: 

- Crea una nueva clase de Cocoa Touch llamada `ListaController` y haz que sea una subclase de `UIViewController`
- Asígnale ese *controller* a la pantalla de lista que has creado antes. Recuerda que esto se hace a través del `identity inspector`.

Ahora vamos a conectar el *view controller* con la barra de búsqueda:


- Selecciona la *search bar* (te será más fácil si lo haces en el árbol de componentes) y en el `connections inspector` conecta el *outlet* `delegate` con el *controller* de esta pantalla.
- Pon en la cabecera `ListaController` que esta clase implementa el protocolo `UISearchBarDelegate`
- En el `ListaController` implementa el método `searchBarSearchButtonClicked(_)`, que se llamará cuando se escriba algo en la barra y se pulse el botón del teclado "buscar". Para probar que funciona de momento basta que imprimas en la consola el texto escrito en la barra y quites el teclado *on screen*.

```swift
func searchBarSearchButtonClicked(_ searchBar: UISearchBar) {
    print("Buscado: \(searchBar.text!)")
    searchBar.resignFirstResponder()
}
```

Ejecuta la *app* y comprueba que efectivamente funciona.

Ahora **tendrás que poner código propio** para que cuando se pulse en el botón de buscar se haga la llamada a la API de Marvel. Por el momento lo más simple es mostrar los resultados con `print`. Puedes asignárselos también a un array que sea una propiedad del *controller*, para que luego sean sencillos de mostrar en la tabla.


###Mostrar los resultados en la tabla

Una vez conseguido esto, tendrás que hacer que los resultados aparezcan en la tabla. Recuerda que necesitas un *datasource* para ella, y que para simplificar puedes hacer que sea el `ListaController`. A grandes rasgos esto implica:

- Que, como decíamos antes, el *controller* debe tener acceso a los datos, por ejemplo guardándolos en una propiedad que sea un array.
- Que tienes que conectar en el `connections inspector` el *outlet* `datasource` con el *controller*.
- Que el `ListaController` debe implementar el protocolo `UITableViewDataSource`. Hay que poner en la cabecera que esto es así, y además implementar los correspondientes métodos en el código del *controller*, uno para devolver el número de filas en la tabla y otro para devolver una celda dado su `indexPath`.

Cada vez que se haga una búsqueda y se añadan datos al array tendrás que decirle a iOS que vuelva a redibujar la tabla llamando a su método `reloadData`. Cuidado, porque al ser una actualización de la interfaz debes asegurarte de que esto lo estás haciendo desde el *thread* principal, algo como:

```swift
//sustituye self.vistaTabla por el outlet que hayas definido
//para acceder a la tabla desde el controlador
OperationQueue.main.addOperation() {
        self.vistaTabla.reloadData();
}
```

> Obtener la cola principal de operaciones explícitamente es necesario si, como es lo más lógico, colocas el código anterior en el bloque que obtiene la respuesta del API de Marvel. Recuerda que para evitar bloqueos, Marvelous funciona de modo asíncrono en un *thread* distinto al principal

Una vez terminado todo esto, esta pantalla debería ser casi totalmente funcional. Solo quedaría ver los detalles de cada *item* al hacer *tap* sobre él.

Opcionalmente **(0,25 puntos extra)**, puedes hacer que aparezca un *spinner* mientras se está procesando la petición, para que el usuario sepa que el sistema está esperando datos. Los *spinners* en iOS se denominan *Activity Indicators*. Puedes consultar la referencia de la [clase `UIActivityIndicator`](https://developer.apple.com/library/prerelease/ios/documentation/UIKit/Reference/UIActivityIndicatorView_Class/index.html) para ver la forma de uso, aunque es muy sencillo, básicamente debes hacer `startAnimating` para que empiece a girar y `stopAnimating` para pararlo. 

> Es posible que el *activity indicator* se te quede detrás de los otros elementos de interfaz y no se vea. Puedes usar el método `bringSubviewToFront` que se llama desde la vista "madre" para pasar a una vista "hija" a "primer plano". Si estamos en el *controller* sería algo como `self.view bringSubview(toFront:self.miActivityIndicator)`, ya que recuerda que `self.view`en un *controller* es la vista principal. Tendrás que sustituir la referencia `self.miActivityIndicator` por la apropiada en tu caso. Además de esto lleva cuidado también con el color, otro problema típico es usar uno del mismo color que el fondo.

## Vista de detalle (0,75 puntos)

La nueva pantalla mostrará los datos de un *item* al hacer *tap*. Como una vez en los detalles queremos poder volver atrás al listado de forma sencilla, lo mejor es usar un *navigation controller*:


### El *navigation controller*  

En este apartado todavía no estamos implementando la vista de detalle, solo creando la jerarquía de navegación.

- Selecciona la pantalla de lista y elige la opción de menú `Editor > Embed In > Navigation Controller`. Aparecerá un nuevo *navigation controller* que tiene como primer nivel de navegación a la pantalla de lista de items.
- Fijate que en la parte superior de la pantalla de items habrá aparecido un espacio reservado para el título y los botones de navegación. Esto puede haber "trastocado" en modo diseño el resto de los elementos de la interfaz (aunque cuando se ejecute la *app* deberían seguir funcionando bien). Recuerda que puedes volverlos "a su sitio" en modo diseño con el `Update Frames` del *autolayout*.
- Selecciona la nueva barra de título de la vista de lista y en las propiedades pon como `title` el recurso que estás mostrando (personajes, autores, comics,...).

### La pantalla de detalles del *item*

Esta sería la que muestra todos los datos de un personaje, comic, creador,... junto a su imagen si es que la tiene.

> Si quieres usar una tabla estática para diseñar esta pantalla debes usar como *controller* una clase que herede de `UITableViewController`

### Creación de la interfaz y conexión con la pantalla anterior

- Arrastra un "view controller" al *storyboard*
- En la pantalla anterior, haz `ctrl+arrastrar` entre la celda prototipo y la pantalla actual. Elige el tipo adecuado de *segue*.
- Implementa una clase `DetalleViewController` que herede de `UIViewController` y asóciala a esta pantalla.

### Implementación de la funcionalidad

- Define en el `DetalleViewController` una propiedad del tipo de recurso que estés mostrando (`RCCharacterObject`, `RCComicsObject`, `RCCreatorObject`,...)
- En el `prepareForSegue` de la pantalla de lista instancia esta propiedad para que contenga el objeto a mostrar. Para saber qué fila de la tabla se ha seleccionado puedes seleccionar la propiedad del objeto tabla llamada `indexPathForSelectedRow`, que almacena el `indexPath` de la fila seleccionada. La propiedad `row` de este *index path* es el número de fila.
- Usa los componentes que consideres necesarios para mostrar los datos del objeto (no es necesario que sean todos, solo los que quieras, para probar que funciona). Tendrás que crear un *outlet* por cada campo, y rellenar los campos en el `viewDidLoad` del *controller*. Puedes consultar la referencia de clases en la [documentación *online* de Marvelous](http://cocoadocs.org/docsets/Marvelous/0.7.3/) para saber qué propiedades tiene cada objeto. La referencia está en Objective-C pero no es difícil deducir qué es cada propiedad
- Entre otras cosas, en esta pantalla deberías mostrar la imagen del personaje, comic, creador o lo que sea que hayas elegido, a un tamaño relativamente pequeño. La carga de la imagen deberías hacerla en un hilo secundario, para no paralizar la interfaz de usuario si la imagen tarda en cargarse:

```swift
let colaBackground = OperationQueue()
colaBackground.addOperation {
    if let thumb = personajes[0].thumbnail {
      let url = "\(thumb.basePath!)/portrait_uncanny.\(thumb.extension!)"
      let urlHttps = url.replacingOccurrences(of: "http", with: "https")
        if let urlFinal = URL(string:urlHttps) {
            do {
               let datos = try Data(contentsOf:urlFinal)
                if let img = UIImage(data: datos) {
                    OperationQueue.main.addOperation {
                        self.miImagen.image = img
                    }
                }
            }
            catch {             
            }
        }
    }
}
```

Puedes consultar [esta página](http://developer.marvel.com/documentation/images) para ver el formato de las URL de las imágenes. Básicamente se construyen con una trayectoria base seguidas de un "modificador" de aspecto y tamaño (`portrait_small`, `landscape_medium`, ...) y la extensión del archivo.

> En el código anterior se obtiene la URL de la imagen y luego se cambia el `http:` por `https:`. Como ya hemos visto en otras sesiones, en principio en iOS>=9 una *app* no puede hacer una petición a una URL web si no es con `https:`. Esto debería cambiarse en la propia librería `Marvelous`, pero podemos salir del paso con este pequeño *parche*.

##Imagen a tamaño completo (0,5 puntos)

Implementa una nueva pantalla en la que se pueda ver solo la imagen a mayor tamaño. Haz que la transición se realice con un *segue* modal pulsando sobre algún botón "ver imagen ampliada" (si tienes activadas las *size classes* el tipo equivalente es `present modally`).

En el API de Marvel, la URL de la imagen a tamaño completo se consigue simplemente concatenando las propiedades `basePath` y `extension`, y añadiendo un punto entre ellas, de modo que sería prácticamente igual al código para cargar la imagen en la pantalla anterior pero sin poner el "modificador" (en el ejemplo era `portrait-uncanny`).
            
## Vista "Acerca de" (0,25 puntos) 

Esta es la segunda de las pantallas del *tab bar*. Simplemente debe consistir en una imagen estática y un *text view* con información sobre la aplicación.