##Ejercicios de *view controllers*

Vamos a hacer una aplicación que vamos a llamar "Pioneras", y que nos mostrará información sobre mujeres pioneras de la informática. La aplicación tendrá una pantalla principal en la que aparecerán sus imágenes, y haciendo *tap* sobre cada una podremos ir a las pantallas secundarias donde se nos dará más información.

###Realizar la estructura básica de la aplicación

1. En el moodle tenemos las imágenes de las tres pioneras: Ada Lovelace, Grace Hopper y Barbara Liskov, que como siempre **arrastraremos al `assets.xcassets`**. También tenemos los textos sobre ellas que se mostrarán en las pantallas secundarias.
2. Crea tres botones en la pantalla principal, y para cada uno de ellos en lugar de texto vamos a usar como imagen de fondo la de cada mujer. Al final cada botón debería ocupar todo el ancho de la pantalla y un tercio del alto.
3. Arrastra un nuevo “view controller” al storyboard (una “pantalla” nueva), que será el que aparezca cuando se pulse en el primero de los botones (el de Ada Lovelace). Inserta un campo de texto de varias líneas (*text view*) y copia en él el contenido de `textos/lovelace.txt`
4. Ahora **establece el *segue* entre las dos pantallas**: haz `Ctrl+Arrastrar` desde el primero de los botones con la imagen de Ada Lovelace hasta la segunda pantalla. En el menú contextual elige el tipo `modal`, **ya que los otros no funcionarán**.
    - Si haces *clic* en el *segue* y vas al *Attribute inspector* puedes cambiar las propiedades, pero tal como está hecha la aplicación solo va a tener efecto la `transition`. Pon el valor que quieras, salvo `partial curl` que puede dar problemas a la hora de volver atrás en el *segue*.
    - Ejecuta el proyecto para comprobar que funciona lo que has hecho, aunque *todavía no puede volver atrás desde la pantalla secundaria*
5. Implementa la opción de **volver atrás** de la secundaria a la principal
    - Crea un botón “atrás” en la secundaria y colócalo en la parte de arriba (para que no lo tape el teclado *on-screen* si aparece)
    - En el *controller* destino crea un método para que funcione el *unwinding* (no hace falta que haga nada, solo que exista)

'' - (IBAction)retornoDeSecundaria:(UIStoryboardSegue*)segue {
'' }

    - Con `Ctrl+Arrastrar` conecta el botón “atrás” con el icono de “Exit” de la parte superior del *controller*
    - Ejecuta el proyecto y comprueba que puedes volver atrás desde la pantalla secundaria
6. Repite lo que has hecho en el caso de Ada Lovelace para las otras dos mujeres, creando las pantallas secundarias y la navegación adelante y atrás.

##Comunicar un *controller* con otro

Es un poco redundante tener tantas pantallas secundarias cuando en realidad lo único que cambia es el texto a mostrar. Valdría con una sola secundaria en la que cambiáramos dinámicamente dicho texto. Vamos a implementarlo así.

###Guardar la versión actual del proyecto para que no se pierda

Lo primero es guardar una copia del estado actual del proyecto. Hay dos posibilidades:
- con `git` podemos crear una etiqueta o *tag* que nos marque la versión actual del proyecto para poder recuperarla luego. No obstante Xcode no nos permite gestionar *tags*, por lo que, **tras asegurarnos de que hemos hecho commit y push de todos los cambios actuales**, haríamos desde la terminal (estando dentro del repositorio):

'' git tag v1.0   (etiquetamos el commit actual)
'' git push origin v1.0 (subimos la etiqueta al repositorio remoto)

- Otra posibilidad es crear una copia manual de los ficheros del proyecto y guardarlo en una carpeta `v1.0` de las plantillas

Ahora podéis eliminar los segues y las pantallas secundarias, es mejor crearlos de nuevo.

###Crear la nueva interfaz

- Crea de nuevo una pantalla secundaria con un campo de texto de varias líneas
- Con `Ctrl+arrastrar` podemos crear un *segue* desde cada uno de los botones hasta la pantalla. Habrán tres *segues* que lleguen a la misma, no debería ser problema.
- Añádele a la pantalla el botón de “atrás” y conéctalo con el icono de “exit”. El código necesario para el *unwinding* (método `retornoDeSecundaria`) ya debería estar en el `ViewController.m`
- Comprueba que la navegación funciona correctamente yendo adelante y atrás

###Crear un controlador personalizado para la pantalla secundaria

Si en la parte derecha de la pantalla miras el *identity inspector* verás que el controlador de la pantalla secundaria es un tipo propio de Cocoa, el `UIViewController`. Vamos a cambiarlo por uno propio

1. Crea una nueva clase de Cocoa Touch, (File> New > File…, plantilla “cocoa touch class”). En la segunda pantalla del asistente dale a la clase el nombre `SecundarioViewController` y haz que sea una subclase de `UIViewController`. Deja sin marcar la opción de crear el .XIB
2. En el *storyboard*, selecciona el *controller* de la pantalla secundaria (es mejor que lo hagas pulsando en el primero de los iconos que aparecen en  la parte superior) 
(img)
3. Una vez seleccionado, ve al *identity inspector* en el área de `Utilities` y en el apartado de `Custom class` selecciona como clase la que has creado, `SecundarioViewController`

###Añadirle un *outlet* al controlador secundario

Tienes que añadir un *outlet* al campo de texto para que su contenido se pueda cambiar desde el controlador secundario. Hazlo como habitualmente, con ctrl+arrastrar entre el campo y el `SecundarioViewController.h`, en el `assistant editor`.

###Hacer que el texto cambie según el botón pulsado

- Lo primero es añadir físicamente los ficheros `*.txt` con los textos al proyecto para que se puedan cargar dinámicamente por código. Pulsa con el botón derecho sobre el proyecto y selecciona `Add files to Pioneras`. Selecciona los tres `.txt`, que se añadirán al proyecto
- Para que le podamos decir al controlador secundario qué fichero tiene que abrir, vamos a crear una `@property` en el `SecundarioViewController` llamada `nomFich` de tipo `NSString*`

'' //En el SecundarioViewController.h
'' @property NSString *nomFich;

- Para establecer una asociación sencilla entre cada segue y los datos a mostrar puedes usar el identificador del *segue*. Haz clic sobre él y en el `Attributes inspector` cambia su `identifier`, respectivamente por `lovelace`, `hopper` y `liskov`
- ahora en la clase `ViewController`, que es el controlador de la pantalla principal, puedes implementar el `prepareForSegue:sender`

''//En el ViewController.m
''-(void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender {
''     SecundarioViewController *svc = segue.destinationViewController;
''     //El fichero a cargar se corresponde con el identificador del segue
''     svc.nomFich = segue.identifier;
'' }

- Finalmente, en el `viewDidLoad` del `SecundarioViewController` puedes acceder a la propiedad `self.nomFich`, cargar el texto del fichero y mostrarlo en el campo de texto.

''//Sacamos el path completo el fichero 
''NSString *filePath = [[NSBundle mainBundle] pathForResource:self.nomFich
''                                                          ofType:@"txt"];
'' NSError *error;
'' NSString *textoFich;
'' if (filePath) {
''     //Leemos el fichero y guardamos su contenido en un NSString*
''     textoFich = [NSString stringWithContentsOfFile:filePath
''                           encoding:NSUTF8StringEncoding
''                           error:&error];
'' //HAY QUE CAMBIAR self.campoTexto POR COMO HAYAIS LLAMADO AL OUTLET!!!!!
'' if (!error) {
''     //Si todo es OK, mostramos la cadena en el campo de texto
''     self.campoTexto.text = textoFich;
'' }
''  
