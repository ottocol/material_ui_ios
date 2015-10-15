#Interfaz de usuario en dispositivos móviles
##iOS, sesión 1: View controllers

---

Recordemos que los *controllers* son **el “pegamento” que relaciona la vista con el modelo**. Contienen el código que:

- Reacciona a los eventos del usuario 
- Modifica la vista.
- Le pide al modelo que "haga cosas"

---

**Aclaración**

Apple usa el confuso término "View Controller" para referirse al controlador de una vista. **El *view controller* es la C**, no la V del MVC


---

##Puntos a tratar

- Introducción. Tipos de *controllers*
- Ciclo de vida
- *Storyboards* y *segues*
- NIBs 

---

##Relación entre *view* y *view controller*

![](img/controller_vs_view.png) 

---

##Tipos básicos de *controllers*

- Los que muestran directamente contenido (*content controllers*)
- Los que contienen otros controladores (*container controllers*).

---

![](img/content_vs_container.png) 



---

##2. Estructura básica del código de un *controller*


---

## Show me the code!

Un *controller* no es más que una clase que hereda de `UIViewController`. Podemos escribirla nosotros o ser propia de Cocoa

```objectivec
//Archivo ViewController.h
@interface ViewController : UIViewController
```

---

## Métodos del ciclo de vida de un *controller*

```objectivec
- (void)loadView
- (void)viewDidLoad
- (void)viewDidUnload
- (void)viewDidAppear
- (void)viewWillAppear
- (void)didReceiveMemoryWarning
```

---

## `viewDidLoad`

- Se llama una única vez tras cargar el controlador en memoria
- Se suele aprovechar para inicializar propiedades, rellenar la vista con contenido si son datos dinámicos, etc.

---

##`viewDidAppear`

- Se llama cada vez que se muestra la vista asociada al controlador
- Típicamente usada para iniciar animaciones

---

##`loadView`

- Solo se usa si queremos crear los componentes de la interfaz por código

```objectivec
- (void)loadView {
    UIView *vista = [[UIView alloc]
                     initWithFrame: [[UIScreen mainScreen] applicationFrame]];
    vista.backgroundColor = [UIColor greenColor];
    self.view = vista;
 }
```

---


##Posibilidades para crear la interfaz de usuario

-  **Storyboards**, (por defecto). representan las “pantallas” de nuestra aplicación y el flujo de navegación entre ellas. 
- **NIBs**: representan una “pantalla” concreta de nuestra aplicación,  asociada a un *view controller*. Pasar de una pantalla a otra (de un *view controller* a otro) es responsabilidad del desarrollador. 
- **Vistas programáticas**: crear por código las vistas y subvistas que componen la interfaz.

---


##3. Storyboards y *segues*


---


##Añadir una "pantalla" a nuestra aplicación

- Área de `Utilities` (derecha), `Object Library`: todos los objetos que contienen en su nombre "View controller"

![](img/controllers_library.png) <!-- .element class="stretch" -->

---


##Cambiar la clase del controller

- Por defecto, las clases de las nuevas pantallas serán propias de Cocoa: `UIViewController`, `UITableViewController`,... 
- En general las cambiaremos por las nuestras, para poder escribir nuestro propio código.

![](img/controller_class.png) 


---

- **Controlador inicial:** la "pantalla" que se va a mostrar al inicio de la aplicación (sin contar la de *splash*)

![](img/controlador_inicial.png) 

---

##Segues

Transiciones entre controladores. Se pueden crear con `Ctrl+arrastrar` entre el elemento que debe disparar la transición y el controlador destino

![](img/segue.png) 


---

##Pasar de un controlador a otro

- Al seguir un *segue*, el cambio de controlador es automático
- Cuando se va a cambiar de un controlador a otro iOS llama a `prepareForSegue:sender`. 

```objectivec
- (void) prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender {
    NSLog(@"Vamos a cambiar de controller!!!")
}
```

---



##Pasar datos entre controllers

1. Definimos una propiedad en el controller destino
2. En el `prepareForSegue:sender` modificamos el valor de la propiedad desde el *controller* origen
3. Cuando llegamos al destino, en la propiedad tenemos la información deseada


```objc
- (void) prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender {
    //Supongamos que el controller destino es de la clase "ViewController2"
    ViewController2 *destino = [segue destinationViewController];
    //Supongamos que la clase "ViewController2"
    //tiene una @property NSString *texto
    destino.texto = @"Hola, bienvenidos a esta pantalla";
}
```

---

##Volver atrás en un *segue*

**Paso 1**. Implementar en el *controller* al que se vuelve un método que devuelva un `IBAction` y tenga como único parámetro un `UIStoryboardSegue *`. No es necesario que el método haga nada en especial. Simplemente sabemos que se ejecutará cuando se vuelva atrás.

```objectivec
- (IBAction)miUnwind:(UIStoryboardSegue *)sender {
    NSLog(@"Vuelta atrás!!");
}
```
