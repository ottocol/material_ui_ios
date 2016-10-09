# Sesión 2: Vistas

<!--TOC-->
## Creación de la interfaz por código

Hasta ahora hemos visto como crear la interfaz visualmente con Xcode, mediante *storyboars* o NIBs,  pero todo lo que se puede hacer con dicha herramienta se puede hacer también de forma programática, ya que lo único que hace el entorno es crear objetos de la API de Cocoa Touch (o definidos por nosotros) y establecer visualmente sus propiedades.

### Ventanas

Las aplicaciones iOS en principio tienen una única ventana, que es la propiedad `window` del *Application Delegate*. Esta ventana la instancia y gestiona de forma automática el *delegate*, así que no tenemos que hacer nada al respecto. Solo en caso de no usar *storyboards* tendríamos que instanciar el *controller* a mostrar y asignárselo a la propiedad `rootView` del objeto `window`.

> El *application delegate* es el objeto que gestiona la ventana principal de la aplicación. Además de esto, también gestiona el ciclo de vida de la aplicación. Recibe los eventos correspondientes a cuando la aplicación arranca, cuando se sale de ella, cuando pasa a segundo plano, etc. Es la clase `AppDelegate` que crea automáticamente la plantilla de Xcode para cualquier aplicación nueva.

En aplicaciones que usen una pantalla externa (*external display*) sí necesitaremos crear una ventana de las dimensiones apropiadas

```swift
//Contamos cuántas pantallas hay (principal + externas)
let screens = UIScreen.screens
if screens.count > 1 {
  //cogemos "a piñón" la primera externa
  let extScreen = screens[1]
  //creamos una ventana con el mismo tamaño que la pantalla externa
  let extWindow = UIWindow(frame: extScreen.bounds)
  //asignamos la ventana a la pantalla
  extWindow.screen = extScreen
  //desde iOS9 todas las ventanas deben tener un controller
  //aquí suponemos definida en algún lugar su clase
  let extController = ViewControllerSecundario()
  //hacemos que la ventana tenga como controller principal el creado
  extWindow.rootViewController = extController
  //mostramos la ventana
  extWindow.isHidden = false
}
```

### Vistas

En el ejemplo anterior hemos supuesto que teníamos una clase `ViewControllerSecundario` que usábamos para mostrar la vista externa. Vamos a ver paso a paso cómo construiríamos esa clase si no quisiéramos emplear un NIB o un *storyboard*.

Podemos construir una vista creando un objeto de tipo `UIView`. En el inicializador deberemos proporcionar el marco que ocupará dicha vista. El marco se define mediante el tipo `CGRect` (se trata de una estructura, no de un objeto). Por ejemplo, podríamos inicializar una vista de la siguiente forma:

```swift
var vista = UIView(frame: CGRect(x:0,y:0,width:100,height:100))
```

En lugar de poner un tamaño fijo deberíamos usar el de la pantalla en la que queramos que aparezca. Se puede obtener el `CGRect` con los límites de la pantalla en la propiedad `bounds` del objeto `UIScreen` que nos interese.

> En los siguientes apartados explicaremos el sistema de coordenadas usado en `CGRect`

Las vistas (`UIView`) también nos proporcionan una serie de métodos para consultar y modificar la jerarquía. El método básico que necesitaremos es `addSubview`, que nos permitirá añadir una subvista a una vista determinada (o a una ventana):

```swift
var label = UILabel()
label.text = "Soy la pantalla secundaria"
vista.addSubView(label)
```
 
En el controlador debemos sobreescribir el método `loadView` del controlador, que es el método encargado por defecto de crear las vistas (usualmente desde un NIB o *storyboard*, aquí por código), y en él crear las vistas necesarias, asignando la vista raíz a `self.view`.

```swift
//método del controlador de la pantalla secundaria
override func loadView() {
    let vista = UIView(frame: self.screen.bounds)
    vista.backgroundColor = UIColor.green
    let label = UILabel()
    label.text = "soy la pantalla secundaria"
    label.frame = CGRect(x: 0, y: 0, width: 200, height: 100)
    vista.addSubView(label)
    self.view = vista
}
```

Podemos eliminar una vista enviándole el mensaje `removeFromSuperview` (se le envía a la vista hija que queremos eliminar). Podemos también consultar la jerarquía con los siguientes métodos:

- `superview`: Nos da la vista padre de la vista destinataria del mensaje.
- `subviews`: Nos da la lista de subvistas de una vista dada.
- `isDescendantOfView:` Comprueba si una vista es descendiente de otra.

Como vemos, una vista tiene una lista de vistas hijas. Cada vista hija tiene un índice, que determinará el orden en el que se dibujan. El índice 0 es el más cercano al observador, y por lo tanto tapará a los índices superiores. Podemos insertar una vista en un índice determinado de la lista de subvistas con `insertSubview:atIndex:`.

Puede que tengamos una jerarquía compleja y necesitemos acceder desde el código a una determinada vista por ejemplo para inicializar su valor. Una opción es hacer un *outlet* para cada vista que queramos modificar, pero esto podría sobrecargar nuestro objeto de *outlets*. También puede ser complejo y poco fiable el buscar la vista en la jerarquía. En estos casos, lo más sencillo es darle a las vistas que buscamos una etiqueta (*tag*) mediante la propiedad `Tag` del inspector de atributos (debe ser un valor entero), o asignando la propiedad `tag` de forma programática. Podremos localizar en nuestro código una vista a partir de su etiqueta mediante `viewWithTag`. Llamando a este método sobre una vista, buscará entre todas las subvistas aquella con la etiqueta indicada:

```swift
 texto = self.window.viewWithTag(tag:1)
```

> La jerarquía de vistas de una pantalla determinada de nuestra aplicación puede llegar a ser muy compleja. Es por eso que en Xcode 6 se ha añadido una opción que nos permite mostrar un “despiece” visual en 3D de las vistas que componen la pantalla actual. Dicha opción está disponible en `Debug > View Debugging`.En modo texto podemos usar la propiedad `recursiveDescription` para [imprimir la descripción textual](https://developer.apple.com/library/ios/technotes/tn2239/_index.html#//apple_ref/doc/uid/DTS40010638-CH1-SUBSECTION34) de las vistas que contiene una vista dada.

## Propiedades de una vista

A continuación vamos a repasar las propiedades básicas de las vistas, que podremos modificar tanto desde Xcode como de forma programática.

### Disposición

Entre las propiedades más importantes en las vistas encontramos aquellas referentes a su disposición en pantalla. Hemos visto que tanto cuando creamos la vista con Xcode como cuando la inicializamos de forma programática hay que especificar el **marco** que ocupará la vista en la pantalla. 

Cuando se crea de forma visual, el marco se puede definir pulsando con el ratón sobre los márgenes de la vista y arrastrando para así mover sus límites. En el código estos límites se especifican mediante el tipo `CGRect`, en el que se especifica posición `(x,y)` de inicio, y el ancho y el alto que ocupa la vista. Estos datos se especifican en el sistema de coordenadas de la supervista.

El sistema de coordenadas tiene su origen en la esquina superior izquierda. Las coordenadas no se dan en *pixels*, sino en **puntos**, una medida que nos permite independizarnos de la resolución en pixels de la pantalla. Las coordenadas en puntos son reales, no enteras.

En los modelos de iPhone/iPod Touch de 3.5’’ la resolución de pantalla en puntos es de 320x480 (aun en los de *retina display*, que tiene un número de pixels mucho mayor). Los dispositivos de 4’’ usan una resolución de 320x568 puntos. El iPhone 6 devuelve 375x667 y el 6 plus 736x414. Podéis consultar una [tabla muy completa](http://www.iosres.com) con muchos más datos 

> Otros frameworks de iOS definen sistemas de coordenadas distintos. Los de gráficos (Core Graphics y OpenGL ES) ponen el origen en la esquina inferior izquierda con el eje Y apuntando hacia arriba.

Algunos ejemplos de cómo obtener la posición y dimensiones de una vista:

```swift
// Limites en coordenadas locales
// Su origen siempre es (0,0)
CGRect areaLocal = vista.bounds
// Posición del centro de la vista en coordenadas de su supervista
CGPoint centro = vista.center
// Marco en coordenadas de la supervista
CGRect marco = vista.frame

> Nótese que a partir de `bounds` y `center` podemos calcular `frame`, aunque nos lo da directamente el sistema

Aquí estamos usando tamaños fijos para las coordenadas de `CGRect`. Sin embargo, en la mayoría de ocasiones nos interesa que el tamaño de las vistas no sea fijo sino que se adapte al área disponible. De esta forma nuestra interfaz podría adaptarse de forma sencilla a distintas orientaciones del dispositivo (horizontal o vertical) o a distintas resoluciones de la pantalla. Esto lo podemos conseguir mediante el uso del **autolayout**, que *calcula de manera automática el *frame* de cada vista basándose en un conjunto de restricciones*. Veremos esta tecnología en sesiones posteriores.

### Transformaciones

Podemos también aplicar una transformación a las vistas, mediante su propiedad `transform`. Por defecto las vistas tienen aplicada la transformación identidad `CGAffineTransform.identity`.

La transformación se define mediante una matriz de transformación 2D de dimensión 3x3. Podemos crear transformaciones sencillas (rotaciones, traslaciones o escalados “puros” con las funciones `CGAffineTransform(rotationAngle:)`, `CGAffineTransform(translationX:,y:)` y  `CGAffineTransform(scaleX:,y:)`.

> Si nuestra vista tiene aplicada una transformación diferente a la identidad, su propiedad `frame` no será significativa. En este caso sólo deberemos utilizar `center` y `bounds`.

### Otras propiedades
En las vistas encontramos otras propiedades que nos permiten determinar su color o su opacidad. En primer lugar tenemos `backgroundColor`, con la que podemos fijar el color de fondo de una vista. En el inspector de atributos (sección `View`) podemos verlo como propiedad `Background`. El color de fondo puede ser transparente, o puede utilizarse como fondo un determinado patrón basado en una imagen.

De forma programática, el color se especifica mediante un objeto de clase `UIColor`. En esta clase podemos crear un color personalizado a partir de sus componentes (rojo, verde, azul, alpha), como en `UIColor(red:,green:,blue:,alpha:)` o con una constante predefinida (por ejemplo, `UIColor.green`)

Por otro lado, también podemos hacer que una vista tenga un cierto grado de transparencia, o esté oculta. A diferencia de `backgroundColor`, que sólo afecta al fondo de la vista, con la propiedad `alpha`, de tipo `CGFloat`, podemos controlar el nivel de transparencia de la vista completa con todo su contenido y sus subvistas. Si una vista no tiene transparencia, podemos poner su propiedad `opaque` a `true` para así optimizar la forma de dibujarla. Esta propiedad sólo debe establecerse a `true` si la vista llena todo su contendo y no deja ver nada del fondo. De no ser así, el resultado es impredecible. Debemos llevar cuidado con esto, ya que por defecto dicha propiedad es `true`.

Por último, también podemos ocultar una vista con la propiedad `isHidden`. Cuando hagamos que una vista se oculte, aunque seguirá ocupando su correspondiente espacio en pantalla, no será visible ni recibirá eventos.

## Algunos controles de interfaz de usuario

A lo largo de los ejemplos que hemos ido haciendo en las sesiones anteriores ya hemos probado bastantes de los controles básicos de interfaz de usuario que nos proporciona iOS: botones, etiquetas, imágenes, campos de texto,… Vamos a ver aquí algunas de las características de los controles, aunque solo vamos a dar unas pinceladas, ya que una descripción exhaustiva de cada propiedad sería imposible y tediosa. Se remite al lector a la documentación de Apple, en concreto el [UIKit User Interface Catalog](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/UIKitUICatalog/UIKitUICatalog.pdf), excelente y bastante exhaustivo.

> Aunque aquí hablemos de controles indistintamente para referirnos a las etiquetas, botones, … en realidad este término tiene un significado más preciso en iOS. La clase `UIControl` es de la que heredan los controles más “interactivos” como los botones, mientras que las etiquetas lo hacen de `UIView` (no obstante todos los`UIControl` son también vistas ya que a su vez esta clase hereda de `UIView`).


### Campos de texto

Un campo de texto nos proporciona un espacio donde el usuario puede introducir y editar texto. Se define en la clase `UITextField`, y pertenece a un grupo de vistas denominados controles, junto a otros componentes como por ejemplo los botones. Esto es así porque permiten al usuario interactuar con la aplicación. No heredan directamente de `UIView`, sino de su subclase `UIControl`, que incorpora los métodos para tratar eventos de la interfaz mediante el patrón *target-action* como hemos visto anteriormente.

Sus propiedades se pueden encontrar en la sección `Text Field` del inspector de atributos. Podremos especificar un texto por defecto (`Text`), o bien un texto a mostrar sombreado en caso de que el usuario no haya introducido nada (`Placeholder Text`). Esto será útil por ejemplo para dar una pista al usuario sobre lo que debe introducir en dicho campo.

Si nos fijamos en el inspector de conexiones del campos de texto, veremos la lista de eventos que podemos conectar a nuestra acciones. Esta lista de eventos es común para cualquier control. En el caso de un campo de texto por ejemplo nos puede interesar el evento `Value Changed`.

#### Teclado en pantalla

Cuando un campo de texto adquiere el foco porque el usuario hace *tap* sobre él, automáticamente aparece el teclado software *on screen*. El problema es que por defecto no desaparece salvo que escribamos algo de código.

Si queremos que desaparezca cuando pulsamos sobre la tecla de “Aceptar” tenemos que escribir un *action* que responda al evento `Did end on exit` del campo de texto. 

![](Captura%20de%20pantalla%202016-10-08%20a%20las%2020.25.05.png)

> Cuando crees el *action* indica en el menú *popup* que el `type` debe ser `UITextField`. Esta opción es el parámetro que se le pasará al *action* y representa el objeto que ha disparado el evento, en nuestro caso el campo de texto. Si lo pasamos como `AnyObject` no tendremos acceso a los métodos del API de `UITextField` (salvo que hiciéramos una conversión con `as`)

En teoría dentro de este *action* debemos hacer que el campo deje de ser el objeto que hace de *first responder* para que el teclado deje de mostrarse

> El *first responder* es el objeto que recibe en primer lugar ciertas clases de eventos, por ejemplo los de teclado o movimiento. Cuando hacemos *tap* sobre un campo de texto, iOS hace que este pase a ser el *first responder*

```swift
 @IBAction func introPulsado(_ sender: UITextField) {
        sender.resignFirstResponder()
 }
```

> Aunque en muchas fuentes (libros, tutoriales en web, …) aparece el método del `resignFirstResponder`, curiosamente *parece bastar con tener un action que responda al `Did end on exit`, aunque no haga nada* para ocultar el teclado.*

Por desgracia, no todos los tipos de teclado en pantalla tienen un botón de “intro” y por tanto no disparan el evento `Did end on exit`, por ejemplo el teclado numérico no lo hace. En ese caso hay que usar una solución algo más rebuscada. Por ejemplo, es muy típico que al pulsar sobre cualquier parte de la pantalla que no sea el campo el teclado se oculte. Esto podemos conseguirlo detectando el evento de toque sobre la vista. Si un evento no se procesa en los componentes de la vista va “subiendo” en la jerarquía hasta llegar al *view controller*, por lo que en él podríamos hacer esto (no es necesario crear un *action*, ya que el evento no va a ir directamente al *controller*, solo escribir el código):

```swift
override func touchesEnded(_ touches: Set<UITouch>, with: UIEvent?) {
   print("¡touch en la pantalla!")
   self.view.viewWithTag(100)?.resignFirstResponder()
}
```



### Botones

Al igual que los campos de texto, los botones son otro tipo de control (heredan de `UIControl`). Se definen en la clase `UIButton`, que puede ser inicializada de la misma forma que el resto de vistas.

Si nos fijamos en el inspector de atributos de un botón (en la sección `Button`), vemos que podemos elegir el tipo de botón (atributo `Type`). Podemos seleccionar una serie de estilos prefedinidos para los botones, o bien darle un estilo propio (`Custom`).

El texto que aparece en el botón se especifica en la propiedad `Title`, y podemos configurar también su color, sombreado, o añadir una imagen como icono.

En el inspector de conexiones, el evento que utilizaremos más comúnmente en los botones es `Touch Up Inside`, que se producirá cuando levantemos el dedo tras pulsar dentro del botón. Este será el momento en el que se realizará la acción asociada al botón.

### Alertas y *action sheets*

Son cuadros de diálogo modales que se usan para informar al usuario de eventos importantes. No se crean gráficamente en Xcode sino por código Pueden simplemente informar de algo o además pedir al usuario que elija uno entre varios cursos de acción.

Vamos a ver primero cómo crear una *alerta*. En realidad el API para *alertas* y *action sheets* es el mismo, solo se diferencian en una constante que se le pasa al inicializador. Luego veremos las diferencias en cuanto a su significado de cara al usuario.

Por ejemplo veamos cómo se mostraría una alerta con dos opciones

![](Captura%20de%20pantalla%202016-10-08%20a%20las%2023.20.24.png)

```swift
//El alert en sí. Vemos que el preferredStyle es .alert
let alert = UIAlertController(title: "¡Elige!", 
            message: "Tienes que elegir uno de estos dos", 
            preferredStyle: .alert)
//cada opción es un UIAlertAction
let susto = UIAlertAction(title: "Susto", style: .cancel) {
    action in
      print("BU!!! haber elegido muerte!")
}
let muerte = UIAlertAction(title: "Muerte", style: .default) {
    action in
      print("Aquí se acaba todo")
}
//Añadimos las opciones al cuadro de diálogo
alert.addAction(susto)
alert.addAction(muerte)
//Mostramos el alert con present, como se hace con cualquier controller
self.present(alert, animated: true) {
    print("Ha desaparecido el alert")
}
```
 
> Podemos tener el número de botones que queramos. No obstante, las “iOS Human Interface Guidelines” [recomiendan como máximo dos](https://developer.apple.com/ios/human-interface-guidelines/ui-views/alerts/), y en caso que necesitemos más usar un *“Action Sheet”*.

Por otro lado un *action sheet* es un cuadro de diálogo que se usa para dar alternativas al usuario cuando ha realizado una acción. Por ejemplo supongamos un gestor de email en el que el usuario ha hecho *tap* sobre un mensaje. Podría aparecer la siguiente *action sheet*

![](Captura%20de%20pantalla%202016-10-08%20a%20las%2023.44.18.png)

El API es exactamente el mismo, solo que en el inicializador de `UIAlertController` se pasa como parámetro `preferredStyle` el valor `actionSheet`.

```swift
let actionSheet = UIAlertController(title: "Opciones", 
    message: "Seleccione la opción", 
    preferredStyle: .actionSheet)
let archivar = UIAlertAction(title: "Archivar", style: .default){
            action in
            print("Aquí se archivaría el mensaje")
}
let eliminar = UIAlertAction(title: "Eliminar", style: .destructive) {
            action in
            print("Aquí se eliminaría el mensaje")
        }
let cancelar = UIAlertAction(title: "Cancelar", style: .cancel) {
            action in
            print("Aquí no se haría nada")
}
actionSheet.addAction(archivar)
actionSheet.addAction(eliminar)
actionSheet.addAction(cancelar)
self.present(actionSheet, animated: true) {
   print("Ha desaparecido el action sheet")
}
```
## Ejercicios de vistas

### Creación de ventana/controlador/vista de modo manual

Ya hemos visto que por defecto las aplicaciones de Xcode usan el *storyboard*. Este *storyboard* lo carga automáticamente el Application Delegate, haciendo además las siguientes tareas:

- Crear una ventana con el tamaño de la pantalla
- Cargar el controlador asociado a la pantalla inicial, y asignarle a su propiedad `view` las subvistas de esta pantalla
- Hacer visible la ventana, que ahora contiene al controlador

Vamos a hacer lo mismo pero de modo manual, para comprender un poco mejor todo el proceso 

- Cread un nuevo proyecto en la carpeta `sesion2` de las plantillas llamado `VistaManual`
- Lo primero es desactivar el *storyboard*. Para ello en el navegador de Xcode hacemos clic sobre el icono del proyecto. Aparecerán los distintos apartados de configuración. Elegimos `info` y eliminamos una propiedad que pone “Main Storyboard name” pulsando sobre el botón de ‘-‘. Ahora el proyecto ya no usa el storyboard
- En el `applicationDidFinishLaunching:withOptions` del AppDelegate tenemos que hacer lo siguiente

	 self.window = [[UIWindow alloc]
	                      initWithFrame: [[UIScreen mainScreen] bounds]];
	 self.window.rootViewController = [[ViewController alloc] init];
	 UIButton *boton = [[UIButton alloc] init];
	 [self.window makeKeyAndVisible];

- No obstante, en pantalla no se verá nada, ya que la vista del `ViewController` está vacía. Probad a hacer que aparezca algo para comprobar que todo funciona:
	- Tened en cuenta que la vista es `self.window.rootViewController.view`
	- cambiad el color, propiedad `backgroundColor` de la vista. Para especificar color lo más sencillo es usar una serie de métodos de clase (estático) de `UIColor`: `[UIcolor greenColor]`, `[UIcolor redColor]`,…
	- cread un botón (instanciad un objeto de la clase UIButton\*), especificando el título (método `setLabel:forState`) y unas dimensiones

	[boton setTitle:@"Soy un botón manual" forState:UIControlStateNormal];	
	[boton setFrame:CGRectMake(100,100,100,40)];


## Uso de controles de interfaz de usuario

En un nuevo proyecto, llamado `Controles`, probad el uso de algunos de los controles de interfaz de usuario de iOS que todavía no hemos visto. Para cualquier duda, consultad la documentación de Xcode o la [referencia de controles de UIKit](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/UIKitUICatalog/UIControl.html)

- Cread un *slider* cuya escala vaya desde 0 hasta 100, y un *label* al lado. Al mover el slider, el *label* debería cambiar para reflejar su valor actual. 
	- Cread un *outlet* para poder manipular la etiqueta
	- Crear un “action” con `ctrl+arrastrar` desde el *slider* al código de `ViewController.m`. En este *action* podéis obtener el valor actual del *slider* (propiedad `value` del parámetro `sender` y fijarlo como texto de la etiqueta. Tened en cuenta que el valor del slider es un número y el texto es un `NSString*`, podéis usar `[NSString stringWithFormat]` para convertir como ya hemos hecho varias veces.

- cread un botón que cuando se pulse aparezca un `UIActionSheet` dando varias opciones con texto inventado por vosotros. Detectad qué opción se ha mostrado e imprimid un mensaje con `NSLog` indicándolo. Tendréis que consultar la [documentación de `UIActionSheet`](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/UIKitUICatalog/UIActionSheet.html#//apple_ref/doc/uid/TP40012857-UIActionSheet-SW1). Mirad el apartado “Behavior of action sheets”. Básicamente la idea es que al crear el *action sheet* se especifica qué objeto es su “delegate” (lo más sencillo es que sea el controller) y este tiene que
	- Especificar que implementa el protocolo `UIActionSheetDelegate`
	- Implementar el método `actionSheet:clickedButtonAtIndex:` que nos dirá el número de opción elegida, comenzando en 0.