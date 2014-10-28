#Proyecto de la asignatura de interfaces de usuario, parte de iOS básico

Este "miniproyecto" trata de hacer una pequeña aplicación basándose en [el API](developer.marvel.com) de la editorial Marvel, que nos permite consultar información sobre los personajes, los *comics*, las *series*,...

##Pasos iniciales

###Registro en la API de Marvel

Lo primero que necesitas es [registrarte](https://secure.marvel.com/user/register?referer=https%3A%2F%2Fdeveloper.marvel.com%2Faccount) en Marvel (si no tienes cuenta ya) para poder obtener un par de claves. Este paso es necesario porque la API tiene un número máximo de llamadas diario, lo que significa que no podemos compartir todos las mismas claves.

###La librería iOS para acceder a la API

La API de Marvel es REST, por lo que acepta peticiones HTTP. No obstante hacerlas directamente sería demasiado engorroso, por lo que usaremos una librería en Obj-C de terceros llamada [Marvelous](https://github.com/rock-n-code/Marvelous) para hacer las peticiones más simples.

####Instalación de la librería

La forma más sencilla de instalar la librería es usar un conjunto de herramientas denominado [*CocoaPods*](http://cocoapods.org). Se trata de un sistema para especificar y resolver automáticamente dependencias de librerías de terceros. Hay muchas librerías disponibles con este sistema.

Para instalar `cocoapods`, desde la terminal hacer

```bash
sudo gem install cocoapods
```

Esto instalará un comando llamado `pod`. Ahora seguir estos pasos:

1. Crear un proyecto Xcode para la aplicación. Llamarlo por ejemplo `Marvel`
2. En la terminal moverse al directorio del proyecto (el que contiene el fichero `.xcodeproj`)
3. Ejecutar el comando `pod init`. Esto creará un fichero llamado `Podfile`
4. Poner las dependencias en el `Podfile`. En nuestro caso hay que añadir al final una línea

```bash
pod 'Marvelous'
```
5. Ejecutar el comando `pod install`. Las dependencias de nuestro proyecto se bajarán automáticamente.
6. Ahora para trabajar siempre **abriremos el fichero Marvel.xcworkspace**, que representa un *workspace* de Xcode (un conjunto de proyectos), no el proyecto Marvel directamente (NO el `Marvel.xcodeproj`).
7. Veremos que en Xcode se muestra nuestro proyecto y además una especie de proyecto adicional llamado *pods* que contiene las dependencias.

###Ejemplo de uso de la librería

Para probar de manera sencilla la librería puedes poner este `import` en el
 `ViewController.m`

```objectivec
#import <Marvelous/Marvelous.h>
```

Y ahora colocar el siguiente código en el `viewDidLoad` y comprobar que se muestran todos los personajes cuyo nombre comienza por "spider".

```objectivec
RCMarvelAPI *marvelAPI = [RCMarvelAPI api];
marvelAPI.publicKey = @"tu-clave-publica-del-api";
marvelAPI.privateKey = @"tu-clave-privada-del-api";
RCCharacterFilter *filtro = [[RCCharacterFilter alloc] init];
filtro.nameStartsWith = @"spider";
[marvelAPI charactersByFilter:filtro
             andCallbackBlock:^(NSArray *resultados, RCQueryInfoObject *info, NSError *error) {
                 for (RCCharacterObject *personaje in resultados) {
                     NSLog(@"%@", personaje.name);
                 }
             }
 ];
```



##Estructura general de la aplicación


El *storyboard* de la aplicación a crear será como el de la siguiente figura

![](storyboard.png)

Puedes comenzar arrastrando al *storyboard* vacío un `tab bar view controller`, que creará el controlador en sí con dos pantallas de contenido. 

##Vista maestro/detalle (sin edición)

En esta parte de la aplicación se debe mostrar (solo ver, no editar) un determinado recurso de los que ofrece el API. Elige tú lo que prefieras: personajes, comics, creadores... 

###Vista maestro

Esta es una pantalla con una vista de tabla en la que se muestra un listado del recurso elegido (por ejemplo un listado de personajes)

**Pasos para crear la pantalla de listado:**

- Tenemos que convertir la primera pantalla de contenido que nos ha creado el `tab bar view controller` en un `navigation controller` y una pantalla de listado. Para ello selecciona la primera pantalla de contenido (lo más sencillo es pulsar en el icono del `controller` de la parte superior) y en el menú `Editor` selecciona `Embed in > Navigation Controller`
- Crea una clase propia para el controller: `ListaViewController` que herede de `UITableViewController` y haz que la pantalla de listado la use (con el `identity inspector`).
- Arrastra en la pantalla un campo de texto y un botón para poder buscar (alternativa: puedes emplear una `UISearchBar`, pero tendrás que consultar la documentación para ver cómo se usa).
- Arrastra en la pantalla una vista de tabla 
- Crea un *outlet* desde la vista de tabla al `.h` del *controller*. Llámalo por ejempo `vistaTabla`
- Conecta la propiedad `datasource` de la vista de tabla con la clase `ListaViewController` (usando el `connections inspector` arrastra desde la propiedad `datasource` hasta el icono del controlador).
- Implementar el controlador de la pantalla (`ListaViewController`)
    + En el `.h` define una propiedad `NSMutableArray *datos`. inicialízalo a un array vacío con `alloc` e `init`.
    + En el `.h` especifica que esta clase implementa el protocolo `UITableViewDataSource`
    + Recuerda que este protocolo indica que al menos debes implementar 
        + `-(NSInteger) tableView:(UITableView*) tabla numberOfRowsInSection:(NSInteger)section`
        + `-(UITableViewCell*) tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath`

**Hacer búsquedas y cargar los datos**

Haz que cuando se teclee algo en el campo de texto y se pulse el botón `buscar` se le pidan los datos al API y se muestren en la tabla. Tendrás que:

- tener un *outlet* asociado al campo de texto para saber qué se ha tecleado
- tener un *action* asociado al botón para saber que se ha pulsado
- pedirle los datos al API
- mostrar los datos llamando al método `reloadData` de la vista de tabla. Como es código que actualiza el interfaz de usuario debes asegurarte que se ejecuta en el *thread* principal para que funcione correctamente:

```objectivec
//sustituye self.vistaTabla por el outlet que hayas definido
//para acceder a la tabla desde el controlador
[[NSOperationQueue mainQueue] addOperationWithBlock:^{
                    [self.vistaTabla reloadData];
                }];
```

> En lugar de un campo de texto y un botón para buscar **sería mejor usar un [`UISearchBar`](https://developer.apple.com/library/ios/documentation/userexperience/conceptual/UIKitUICatalog/UISearchBar.html#//apple_ref/doc/uid/TP40012857-UISearchBar-SW14)**, pero tu controller (o quien sea) tendría que implementar el protocolo [`UISearchBarDelegate`](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UISearchBarDelegate_Protocol/index.html#//apple_ref/occ/intf/UISearchBarDelegate). Y todavía mejor emplear un *search display controller*, que es una barra de búsqueda vinculada automáticamente con una tabla de resultados. Este último *controller* lo verás en la parte avanzada de la asignatura. Si tienes curiosidad puedes consultar por ejemplo [este tutorial](http://www.appcoda.com/search-bar-tutorial-ios7/)

###Vista detalle

En esta pantalla deben aparecer los detalles del recurso en el que has hecho *tap*.

> Si quieres usar una tabla estática para diseñar esta pantalla debes usar como *controller* una clase que herede de `UITableViewController`

**Infraestructura y conexión con la pantalla anterior**

- Arrastra un "view controller" al *storyboard*
- En la pantalla anterior, selecciona la vista de tabla y en el *attribute inspector* haz que tenga 1 *prototype cells*, Asegúrate de que el "reuse identifier" del prototipo se corresponde con el que estás usando en el código de `cellForRowAtIndexPath` de la tabla con el listado.
- Haz `ctrl+arrastrar` entre la celda prototipo y la pantalla actual. Elige el tipo *push* para el *segue* (dentro de "selection segue").
- Implementa una clase `DetalleViewController` que herede de `UIViewController` y asóciala a esta pantalla.

**Implementación de la funcionalidad**

- Define en el `.h` del *controller* una `@property` del tipo de recurso que estés mostrando (`RCCharacterObject`, `RCComicsObject`, `RCCreatorObject`,...)
- En el `prepareForSegue` instancia esta `@property` para que contenga el objeto a mostrar.
- Diseña una pantalla en la que se muestren los datos del objeto (no es necesario que sean todos, solo los que quieras, para probar que funciona). Tendrás que crear un *outlet* por cada campo, y rellenar los campos en el `viewDidLoad` del *controller*. Consulta el .h de la clase elegida, o la documentación del API para saber qué propiedades tiene cada objeto.
- Entre otras cosas, en esta pantalla deberías mostrar la imagen del personaje, comic, creador o lo que sea que hayas elegido, a un tamaño relativamente pequeño. La carga de la imagen deberías hacerla en un hilo secundario, para no paralizar la interfaz de usuario si la imagen tarda en cargarse:

```objc
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
[queue addOperationWithBlock:^{
    //sustituye self.personaje por el objeto que quieres mostrar
    RCImageObject *thumb = self.personaje.thumbnail;
    NSString *url = [NSString stringWithFormat:@"%@/portrait_uncanny.%@", 
                     thumb.basePath,  thumb.extension];
    NSData * data = [[NSData alloc] 
                     initWithContentsOfURL: [NSURL URLWithString: url]];
    if (data) {
        UIImage *img = [[UIImage alloc] initWithData:data];
        [[NSOperationQueue mainQueue] addOperationWithBlock:^{
            //sustituye self.miImagen por el outlet para acceder a la imagen
            self.miImagen.image = img;
        }];
    }
}];

```

Puedes consultar [esta página](http://developer.marvel.com/documentation/images) para ver el formato de las URL de las imágenes. Básicamente se construyen con una trayectoria base seguidas de un "modificador" de aspecto y tamaño (`portrait_small`, `landscape_medium`, ...) y la extensión del archivo.

##Controlador modal

Implementa una nueva pantalla en la que se pueda ver solo la imagen a mayor tamaño. Haz que la transición se realice con un *segue* modal pulsando sobre algún botón "ver imagen ampliada". 

Para volver atrás en este tipo de *segue* necesitas hacer un *unwind*, pero [parece haber un *bug* en iOS 8.0](http://stackoverflow.com/questions/25654941/unwind-segue-not-working-in-ios-8) que hace que no funcione en situaciones como la que se pide. Una alternativa es usar un *action* de un botón "volver atrás" para ejecutar

```objectivec
[self dismissViewControllerAnimated:YES completion:nil];
```
que debería dejar de mostrar el controlador modal actual.

> Aclaración: en una aplicación real posiblemente usaríamos un *segue* de tipo "push", pero dado que se trata de un ejercicio el objetivo es probar todos los tipos de *segues* que conocemos.
