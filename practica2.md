# Ejercicio 1:
  a) Existen N personas que deben ser chequeadas por un detector de metales
  antes de poder ingresar al avión. Implemente una solución que modele solo el
  acceso de la persona al detector (es decir si el detector esta libre la
  persona lo puede utilizar caso contrario debe esperar).

sem e = 1  
### process P [i= 1 to n]
  P(e)
  -- Ingreso al detector
  V(e)
  -- Ingreso al avion

  b) Modifique su solución para que funcione en el caso que el detector pueda controlar a tres
  personas a la vez.

sem e = 3
### process P [i= 1 to n]
  P(e)
  -- Ingreso al detector
  V(e)
  -- Ingreso al avion

# Ejercicio 2:
  Un sistema operativo mantiene 5 instancias de un recurso almacenadas en una
  cola, cuando un proceso necesita usar una instancia del recurso la saca de la
  cola, la usa y cuando termina de usarla la vuelve a depositar.

Cola buffer
sem e = 1
sem recurso = 5

### process proceso [i = 1 to n]

  P(recurso)

  P(e)
  push(buffer, i)
  V(e)
  
  --Uso recurso
  
  P(e)
  pop(buffer, i)
  V(e)

  V(recurso)


# Ejercicio 3:
  Se tiene un curso con 40 alumnos, la maestra entrega una tarea distinta a
  cada alumno, luego cada alumno realiza su tarea y se la entrega a la maestra
  para que la corrija, esta revisa la tarea y si está bien le avisa al alumno
  que puede irse, si la tarea está mal le indica los errores, el alumno
  corregirá esos errores y volverá a entregarle la tarea a la maestra para que
  realice la corrección nuevamente, esto se repite hasta que la tarea no tenga
  errores.

alumnoActual, tareas[40], notas[40]= 0,
sem tarea_asignada[40] = 0
sem e = 1
sem profe = 0
sem esperando_nota = 0
###process alumno [i = 1 to 40]

> Espero hasta que tenga tarea asignada
  P( tarea_asignada[i] )
  
  while (no_aprobe) {
    tareas[i]= hacerTarea()
    P( e )
    alumnoActual = i
    V( profe )
    P(esperando_nota)
  }

#process maestra

> Asigno tarea a todos los alumnos
  for i = 0 to 39
    V( tarea[i] )
  while( True ) {
    P( profe )
    nota[ alumnoActual ] = corregir( tareas[ alumnoActual ] )
    V( e )
    V( esperando_nota )
  }

# Ejercicio 4:
    Suponga que se tiene un curso con 50 alumnos. Cada alumno elije una de las
    10 tareas para realizar entre todos. Una vez que todos los alumnos
    eligieron su tarea comienzan a realizarla. Cada vez que un alumno termina
    su tarea le avisa al profesor y si todos los alumnos que tenían la misma
    tarea terminaron el profesor les otorga un puntaje que representa el orden
    en que se terminó esa tarea.  
    _Nota_: Para elegir la tarea suponga que existe una función elegir que le
    asigna una tarea a un alumno (esta función asignará 10 tareas diferentes
    entre 50 alumnos, es decir, que 5 alumnos tendrán la tarea 1, otros 5 la
    tarea 2 y así sucesivamente para las 10 tareas). El tiempo en un alumno
    tarda en realizar la tarea es random.

    cont= 0
    tareas_terminadas[10]= 0
    sem e= 0
    todos= 0
    tarea_en_cuestion= 0
    notas[10]= 0
### process alumno[ i= 1 to 10 ]
    tarea_asignada= elegir()
    > Espero que todos tengan su tarea asignada
    P( e )
    cont++
    if ( cont < 50 )
      V( e )
      P( todos )
    else 
      V( e )
      for x= 1 to 49
        V( todos )
    
    hacerTarea( tarea_asignada )
    P( e )
    tarea_en_cuestion= tarea_asignada
    V( profedor )

### process profesor
    nota_actual= 10
    termino= false
    while ( !termino ) {
      P( profesor )
      tareas_terminadas[ tarea_en_cuestion ]++
      if ( tareas_terminadas[ tarea_en_cuestion ] == 5 )
        notas[ tarea_en_cuestion ]= nota_actual
        nota_actual--
      if ( nota_actual == 0 ) 
        termine= true        
    }
    
# Ejercicio 5: 
    Existen N alumnos que desean consultar a alguno de los A ayudantes en una
    práctica. Para esto cada alumno se agrega en una cola para consultas. Si
    una vez que el alumno se agregó en la cola pasaron más de 15 minutos el
    mismo se retira; por el contrario si fue atendido por un ayudante, entonces
    el alumno le entrega su ejercicio resuelto y espera la opinión del
    ayudante. Si el ejercicio estaba correcto el alumno se retira, en caso que
    tenga que modificarlo, realiza las modificaciones y vuelve a agregarse a la
    cola (Ya no debe tenerse en cuenta lo de los 15 minutos).  
    Nota: Suponga que existe una función que devuelve si un alumno hizo bien o
    mal el ejercicio. El alumno no decide a cual ayudante le consulta. Los 15
    minutos solo deben tenerse en cuenta la primera vez, es decir, si el alumno
    fue atendido ya no se toma más en cuenta el tiempo.

sem s_cola= 1
sem s_estado= 1
sem s_timer[N]= 0
sem esperando[N]= 0  'espera por las dos condiciones'
sem tareas[N]= 0     'tareas para corregir'
estado[ N ]= ""
Cola cola

### process alumno [ id= 1 to N ]
  P( s_cola )
  estado[ id ]= "en cola"
  push( cola, id )
  V( s_cola )
> aviso al timer que llegue
  V( s_timer[ id ] )
> Espero x las condiciones
  P( esperando[id] )
  P( s_estado )                                ###__???__
> si estroy corrigiendo (sino me fui por timer, saltea while)
  while (estado[ id ]= "en correccion") {
>   habilito el ayudante para que me corrija
    V( s_estado )
    V( tareas[ id ] ) 
    P( esperando[ id ] )
    P( s_estad )
  }        
  V( s_estado )

### process timer [ id= 1 to N ]
  P( s_timer[ id ] )
  DELAY( 15 )
  P( s_estado )
  if estado[ id ] == "en cola"
    estado[ id ]= "me fui"
    V( esperando[ id ] )
  V( s_estado )
  
### process ayudante [ id= 1 to A ]
  
  while( true ) { 
    P( s_cola )
    if ( cola.not_empty )
      pop(cola, id_alu)
      V( s_cola )
      P( s_estado[ id_alu ] )
      if ( estado[ id_alu ] == "en cola" )
        estado[ id_alu ] = "en correcion"
        V( s_estado[ id_alu ] )
        V( esperando[id_alu] ) 
        P( tareas[ id_alu ] ) 
        corregirTarea(id_alu)
        V( esperando[id_alu] ) 
      else
        V( s_estado[ id_alu ] )
    else
      V( s_cola )
  }

# Ejercicio 6: 
  A una empresa llegan E empleados y por día hay T tareas para hacer (T>E), una
  vez que todos los empleados llegaron empezaran a trabajar. Mientras haya
  tareas para hacer los empleados tomaran una y la realizarán. Cada empleado
  puede tardar distinto tiempo en realizar cada tarea. Al finalizar el día se
  le da un premio al empleado que más tareas realizó.


sem s_todos= 1
sem s_tareas= 1
sem todos= 0
cont= 0
tareas= 0
tareas_x_empleado[E]= 0
### process empleados [id= 1 to E]
  P( s_todos )
  cont++
  if cont < E
    V( s_todos )
    P( todos )
  else
    V( s_todos )
    for i= 1 to E-1
      V( todos )
> Aca siguen todos juntos.
  while ( tareas < T ){
    tarea= tomarTarea(  ) 
    P( s_tareas )
    tareas++
    V( s_tareas )
    hacerTarea( tarea )
    tareas_x_empleado[ id ]++
  }

# Ejercicio 7:
  Suponga que hay N tareas que se realizan en forma diaria por los operarios de
  una fábrica. Suponga que existen M operarios (M = Nx5). Cada tarea se realiza
  de a grupos de 5 operarios, ni bien llegan a la fábrica se juntan de a 5 en
  el orden en que llegaron y cuando se ha formado el grupo se le da la tarea
  correspondiente empezando de la tarea uno hasta la enésima.  Una vez que los
  operarios del grupo tienen la tarea asignada producen elementos hasta que
  hayan realizado exactamente X entre los operarios del grupo. Una vez que
  terminaron de producir los X elementos, se juntan los 5 operarios del grupo y
  se retiran.  Tenga en cuenta que pueden salir varios grupos de operarios al
  mismo tiempo y que no debe interferir con la entrada.  
  
  Nota: cada operario puede hacer 0, 1 o más elementos de una tarea. El tiempo
  que cada operario tarda en hacer cada elemento es diferente y random.
  Maximice la concurrencia.

sem s_cinco= 1
sem cinco= 0
sem tarea_grupo[ N ]
cont= 0
tarea_actual= 1
### process operario[ id= 1 to N ]
  P( s_cinco )
  if ( cont < 5 )
    cont++
    V( s_cinco )
    P( cinco )
  else
    cont= 0
    V( s_cinco )
    for i= 1 to 4
      V( cinco )

> Aca se formo un grupo de cinco
  
  tarea= obtenerTarea(  )

sem s_cola= 1
sem tarea[ N ]= 0
Cola cola
### process operario[ id= 1 to N ]
  P( s_cola )
  push( cola, id )
  V( s_cola )
  P( tarea[ id ] )

### process fabrica
  while true { 
    P( s_cola )
    pop( cola, id )
    V( s_cola )
    
  }
