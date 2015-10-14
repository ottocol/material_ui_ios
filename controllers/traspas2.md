

##Volver atrás en un *segue* (II)

2. `Ctrl+Arrastrar` desde el elemento de interfaz que queremos que desencadene el *unwind*, hasta el icono de `Exit` 

![](img/unwind_segue.png)

---

## 4. NIBs

---

##NIBs

- Un NIB (o `.xib`) contiene la jerarquía de vistas de un *view controller* (una pantalla)
    -  No se crea de manera manual, sino visualmente con Xcode (el nombre viene de “NeXT Interface Builder”)
    -  Es responsabilidad del desarrollador cambiar de un controlador a otro y cargar el NIB correspondiente

---

##Crear NIBs con Xcode

- Opciones en las plantillas de Xcode:
    + Crear un controller y automáticamente un NIB asociado
    + Crear directamente el NIB y luego asociarle manualmente un *controller*    

---


##Cargar un NIB (opción 1)

Si el NIB tiene un *controller* asociado, se cargará automáticamente al instanciar el *controller*.

```objectivec
MiViewController *mvc = [[MiViewController alloc] init];
//pasamos a este controller
self.window.rootViewController = mvc;
```

---

##Cargar un NIB (opción 2)

```objectivec
EjemploViewController *evc =   [[EjemploViewController alloc] 
                                 initWithNibName:@"ejemplo" bundle:nil];
self.window.rootViewController = evc;
```

---

##El File's Owner

- Es el objeto que ha cargado en memoria el NIB (normalmente el *controller* asociado). Tiene su propio icono en el editor visual

- Se pueden crear *outlets* y actions arrastrando entre el icono del File’s Owner y el elemento de interfaz (primero se escribe manualmente el código y luego se hace la conexión).



