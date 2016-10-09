## Ejercicios de *view controllers*

Vamos a hacer una aplicación que vamos a llamar “Pioneras”, y que nos dará datos de algunas mujeres pioneras de la informática. La aplicación tendrá una pantalla principal en la que aparecerán sus imágenes, y haciendo *tap* sobre cada una podremos ir a las pantallas secundarias donde se nos dará más información.

> Al crear el proyecto aseguráos de estar usando git (Sea desde Xcode o manualmente) ya que habrá que guardar y marcar el estado con un commit especial en un momento intermedio

### Realizar la estructura básica de la aplicación (0,75 puntos)

1. En [este archivo comprimido](recursos.zip) tenemos las imágenes de las tres pioneras: Ada Lovelace, Grace Hopper y Barbara Liskov, que como siempre **arrastraremos al `Assets.xcassets`**. También tenemos los textos sobre ellas que se mostrarán en las pantallas secundarias.
2. Crea tres botones en la pantalla principal, y para cada uno de ellos en lugar de texto vamos a usar como imagen de fondo la de cada mujer. Al final cada botón debería ocupar todo el ancho de la pantalla y más o menos un tercio del alto.
> Importante: no es necesario que la interfaz sea perfecta (todos los botones exactamente del mismo alto, etc). De hecho si la pruebas en un dispositivo de tamaño de pantalla distinto al que estás usando ahora mismo en Xcode verás que se ve “fatal”. Para hacer que todos los botones tengan el mismo alto  y que se adapten bien a la pantalla usaremos un mecanismo que todavía no hemos visto denominado *autolayout*. Por el momento  vamos a ignorar este tema
3. Arrastra un nuevo “view controller” al storyboard (una “pantalla” nueva), que será el que aparezca cuando se pulse en el primero de los botones (el de Ada Lovelace). Inserta un campo de texto de varias líneas (*text view*) y copia en él el contenido de `lovelace.txt`
4. Ahora **establece el *segue* entre las dos pantallas**: haz `Ctrl+Arrastrar` desde el primero de los botones con la imagen de Ada Lovelace hasta la segunda pantalla. 
	- Si haces *clic* en el *segue* y vas al *Attribute inspector* puedes cambiar las propiedades, pero tal como está hecha la aplicación solo va a tener efecto la `transition`. Pon el valor que quieras.
	- Ejecuta el proyecto para comprobar que funciona lo que has hecho, aunque *todavía no puede volver atrás desde la pantalla secundaria*
5. Implementa la opción de **volver atrás** de la secundaria a la principal
	- Crea un botón “atrás” en la pantalla secundaria y colócalo en la parte de arriba (para que no lo tape el teclado *on-screen* si aparece)
	- En el *controller* destino crea un método para que funcione el *unwinding* (no hace falta que haga nada, solo que exista)

```swift
@IBAction func retornoDeSecundaria(segue: UIStoryboardSegue) {
    
}
```

- Con `Ctrl+Arrastrar` conecta el botón “atrás” con el icono de “Exit” de la parte superior del *controller*
	- Ejecuta el proyecto y comprueba que puedes volver atrás desde la pantalla secundaria
6. Repite lo que has hecho en el caso de Ada Lovelace para las otras dos mujeres, creando las pantallas secundarias y la navegación adelante y atrás.

**Aseguráos de guardar el estado actual del proyecto** con un commit cuyo comentario sea “version 1”.

## Comunicar un *controller* con otro (1,25 puntos)

Es un poco redundante tener tantas pantallas secundarias cuando en realidad lo único que cambia es el texto a mostrar. Valdría con una sola secundaria en la que cambiáramos dinámicamente dicho texto. Vamos a implementarlo así.

Ahora podéis eliminar los segues y las pantallas secundarias, es mejor crearlos de nuevo.

### Crear la nueva interfaz

- Crea de nuevo una pantalla secundaria con un campo de texto de varias líneas
- Con `Ctrl+arrastrar` podemos crear un *segue* desde cada uno de los botones hasta la pantalla. Habrán tres *segues* que lleguen a la misma, no debería ser problema.
- Añádele a la pantalla el botón de “atrás” y conéctalo con el icono de “exit”. El código necesario para el *unwinding* (método `retornoDeSecundaria`) ya debería estar en el `ViewController`
- Comprueba que la navegación funciona correctamente yendo adelante y atrás

### Crear un controlador personalizado para la pantalla secundaria

Si en la parte derecha de la pantalla miras el *identity inspector* verás que el controlador de la pantalla secundaria es un tipo propio de Cocoa, el `UIViewController`. Vamos a cambiarlo por uno propio

1. Crea una nueva clase de Cocoa Touch, (File\> New \> File…, plantilla “cocoa touch class”). En la segunda pantalla del asistente dale a la clase el nombre `SecundarioViewController` y haz que sea una subclase de `UIViewController`. Deja sin marcar la opción de crear el .XIB
2. En el *storyboard*, selecciona el *controller* de la pantalla secundaria (es mejor que lo hagas pulsando en el primero de los iconos que aparecen en  la parte superior) 
![](iconos_arriba_storyboard.png)
3. Una vez seleccionado, ve al *identity inspector* en el área de `Utilities` y en el apartado de `Custom class` selecciona como clase la que has creado, `SecundarioViewController`

### Añadirle un *outlet* al controlador secundario

Tienes que añadir un *outlet* al campo de texto para que su contenido se pueda cambiar desde el controlador secundario. Hazlo como habitualmente, con ctrl+arrastrar entre el campo y el `SecundarioViewController`, en el `assistant editor`.

### Hacer que el texto cambie según el botón pulsado

- Lo primero es añadir físicamente los ficheros `*.txt` con los textos al proyecto para que se puedan cargar dinámicamente por código. Pulsa con el botón derecho sobre el proyecto y selecciona `Add files to Pioneras`. Selecciona los tres `.txt`, que se añadirán al proyecto
- Para que le podamos decir al controlador secundario qué fichero tiene que abrir, debes crear una propiedad en el `SecundarioViewController` llamada `nomFich` de tipo `String`

- Para establecer una asociación sencilla entre cada segue y los datos a mostrar puedes usar el identificador del *segue*. Haz clic sobre él y en el `Attributes inspector` cambia su `identifier`, respectivamente por `lovelace`, `hopper` y `liskov`
- ahora en la clase `ViewController`, que es el controlador de la pantalla principal, puedes implementar el `prepare(for:,sender:)`

```swift
override func prepare(for segue: UIStoryboardSegue, sender: Any?) { 
  //obtenemos el controller destino y forzamos la conversión al tipo adecuado
  let controller = segue.destination as! SecundarioViewController
  //fijamosla propiedad "nomFich" al identificador del segue
  controller.nomFich = segue.identifier
}
```

- Finalmente, en el `viewDidLoad()` del `SecundarioViewController` puedes acceder a la propiedad `self.nomFich`, cargar el texto del fichero y mostrarlo en el campo de texto. *Tendrás que escribir el código tú mismo*. Haz uso de los métodos:
	- `Bundle.main.path(forResource:, ofType:)`, que devuelve la trayectoria completa para acceder a un recurso incluido en el proyecto sabiendo su nombre y su tipo (en el tipo pon solo “txt”, sin el punto).
	- Una vez obtenida la trayectoria, puedes leer el contenido del archivo como una cadena con el constructor de `String(contentsOfFile:encoding)`. Donde el primer parámetro es la trayectoria y el segundo el juego de caracteres (en nuestro caso el  valor enumerado `String.Encoding.utf8`). CUIDADO, este método está marcado con `throws`, así que tendrás que actuar en consecuencia. (usar do..catch o cualquier otra alternativa que veas razonable)

