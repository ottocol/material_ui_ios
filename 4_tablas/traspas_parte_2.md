# 4. Editar datos en tablas dinámicas: el *delegate*

---

##El *delegate*

- Debe implementar el protocolo `UITableViewDelegate`

```objectivec
@interface ViewController : UIViewController <UITableViewDelegate>
@end
```

- Los métodos para seleccionar y editar se deben implementar en él

---


##Seleccionar celdas

 - El designado como *delegate* recibirá una llamada a `tableView:didSelectRowAtIndexPath:`


```objectivec
- (void) tableView:(UITableView *)tableView 
             didSelectRowAtIndexPath:(NSIndexPath *)indexPath {
     UITableViewCell *celda = [tableView cellForRowAtIndexPath:indexPath];
     //Colocamos un "checkmark" en la celda o lo quitamos si ya estaba
     if (celda.accessoryType==UITableViewCellAccessoryNone)
         celda.accessoryType = UITableViewCellAccessoryCheckmark;
     else
         celda.accessoryType = UITableViewCellAccessoryNone;
     //Hacemos que la celda se deseleccione visualmente
     [tableView deselectRowAtIndexPath:indexPath animated:YES];
 }
 ```

---

##Poner/quitar modo edición


```objectivec
UITableView *miTableView;
...
[miTableView setEditing:YES animated:YES];
```

---

##Estilo de edición

- El estilo "delete" muestra una señal de "prohibido" en la izquierda, indicando que si la pulsamos podemos borrar la celda
- El "insert" muestra un "+"

```objectivec
- (UITableViewCellEditingStyle) tableView:(UITableView *)tableView 
      editingStyleForRowAtIndexPath:(NSIndexPath *)indexPath {
    if (indexPath.row==[self.datos count]-1)
        return UITableViewCellEditingStyleInsert;
    else
        return UITableViewCellEditingStyleDelete;
}
```



---

##Editar filas (insertar/borrar)

```objectivec
- (void) tableView:(UITableView *)tableView 
           commitEditingStyle:(UITableViewCellEditingStyle)editingStyle 
           forRowAtIndexPath:(NSIndexPath *)indexPath {
    if (editingStyle == UITableViewCellEditingStyleDelete) {
        //Eliminamos el objeto del modelo
        //IMPORTANTE: siempre antes del modelo que visualmente de la tabla
        [self.datos removeObjectAtIndex:indexPath.row];
        //lo borramos visualmente de la tabla
        [tableView deleteRowsAtIndexPaths:[NSArray arrayWithObject:indexPath]
                    withRowAnimation:UITableViewRowAnimationFade];
    }
    else {
        [self.datos insertObject:@"nuevo" atIndex:indexPath.row];
        [tableView insertRowsAtIndexPaths:[NSArray arrayWithObject:indexPath]
                         withRowAnimation:UITableViewRowAnimationAutomatic];
    }
}
```

---

