## Ejercicio 1:
  a) Existen N personas que deben ser chequeadas por un detector de metales
  antes de poder ingresar al avión. Implemente una solución que modele solo el
  acceso de la persona al detector (es decir si el detector esta libre la
  persona lo puede utilizar caso contrario debe esperar).

    sem e = 1  
    process P [i= 1 to n]
    P(e)
    -- Ingreso al detector
    V(e)
    -- Ingreso al avion

  b) Modifique su solución para que funcione en el caso que el detector pueda
  controlar a tres
  personas a la vez.

    sem e = 3
    process P [i= 1 to n]
    P(e)
    -- Ingreso al detector
    V(e)
    -- Ingreso al avion

## Ejercicio 2:
  Un sistema operativo mantiene 5 instancias de un recurso almacenadas en una
  cola, cuando un proceso necesita usar una instancia del recurso la saca de la
  cola, la usa y cuando termina de usarla la vuelve a depositar.

    Cola buffer
    sem e = 1
    sem recurso = 5

    process proceso [i = 1 to n]

    P(recurso)

    P(e)
    push(buffer, i)
    V(e)
    
    --Uso recurso
    
    P(e)
    pop(buffer, i)
    V(e)

    V(recurso)


## Ejercicio 3:
  Se tiene un curso con 40 alumnos, la maestra entrega una tarea distinta a
  cada alumno, luego cada alumno realiza su tarea y se la entrega a la maestra
  para que la corrija, esta revisa la tarea y si está bien le avisa al alumno
  que puede irse, si la tarea está mal le indica los errores, el alumno
  corregirá esos errores y volverá a entregarle la tarea a la maestra para que
  realice la corrección nuevamente, esto se repite hasta que la tarea no tenga
  errores.

    alumnoActual, tareas[40], notas[40]= 0,
    sem tarea_asignada[40] = 0
    sem esperando_nota[40] = 0
    sem e = 1
    sem profe = 0

    process alumno [i = 1 to 40]

    > Espero hasta que tenga tarea asignada
      P( tarea_asignada[i] )
      
      while (no_aprobe) {
        tareas[i]= hacerTarea()
        P( e )
        alumnoActual = i
        V( profe )
        P(esperando_nota[i])
      }

    process maestra

    > Asigno tarea a todos los alumnos
      for i = 0 to 39
        V( tarea_asignada[i] )
      while( True ) {
        P( profe )
        nota[ alumnoActual ] = corregir( tareas[ alumnoActual ] )
        V( e )
        V( esperando_nota[i] )
      }

## Ejercicio 4:
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

    process alumno[ i= 1 to 10 ]
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

    process profesor
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
## Ejercicio 5: 
  Existen N alumnos que desean consultar a alguno de los A ayudantes en una
  práctica. Para esto cada alumno se agrega en una cola para consultas. Si
  una vez que el alumno se agregó en la cola pasaron más de 15 minutos el
  mismo se retira; por el contrario si fue atendido por un ayudante, entonces
  el alumno le entrega su ejercicio resuelto y espera la opinión del
  ayudante. Si el ejercicio estaba correcto el alumno se retira, en caso que
  tenga que modificarlo, realiza las modificaciones y vuelve a agregarse a la
  cola (Ya no debe tenerse en cuenta lo de los 15 minutos).
  _Nota_: Suponga que existe una función que devuelve si un alumno hizo bien o
  mal el ejercicio. El alumno no decide a cual ayudante le consulta. Los 15
  minutos solo deben tenerse en cuenta la primera vez, es decir, si el alumno
  fue atendido ya no se toma más en cuenta el tiempo.

    sem s_cola= 1
    sem s_estado[N]= 1
    sem s_timer[N]= 0
    sem esperando[N]= 0  'espera por las dos condiciones'
    sem tareas[N]= 0     'tareas para corregir'
    sem s_hay= 0
    estado[ N ]= ""
    Cola cola

    process alumno [ id= 1 to N ]
    P( s_cola )
    estado[ id ]= "en cola"
    push( cola, id )
    V( s_hay )     
    V( s_cola )
    > aviso al timer que llegue
    V( s_timer[ id ] )
    > Espero x las condiciones
    P( esperando[id] )
    P( s_estado[id] )
    > si estroy corrigiendo (sino me fui por timer, saltea while)
    while (estado[ id ]= "en correccion") {
      > habilito el ayudante para que me corrija
      V( s_estado[id] )
      V( tareas[ id ] ) 
      P( esperando[ id ] )
      P( s_estado[id] )
    }        
    V( s_estado[id] )

     process timer [ id= 1 to N ]
      P( s_timer[ id ] )
      DELAY( 15 )
      P( s_estado[id] )
      if estado[ id ] == "en cola"
        estado[ id ]= "me fui"
        V( esperando[ id ] )
      V( s_estado[id] )
      
     process ayudante [ id= 1 to A ]

     P( s_hay )     
        P( s_cola )
        if ( cola.not_empty )
          pop(cola, id_alu)
          V( s_cola )
          P( s_estado[ id_alu ] )
          if ( estado[ id_alu ] == "en cola" )
            estado[ id_alu ] = "en correcion"
            V( s_estado[ id_alu ] )
            V( esperando[id_alu] ) 
            P( tareas[ id_alu ] ) /// falta agregar una variable compartida para que el alumno pase la tarea
            corregirTarea(id_alu)
            V( esperando[id_alu] ) 
          else
            V( s_estado[ id_alu ] )
        else
          V( s_cola )
      }

## Ejercicio 6: 
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
     process empleados [id= 1 to E]
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
        tareas_x_empleado[ id ]++  ///////hacer otra barrera
      }

## Ejercicio 7:
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
  _Nota_: cada operario puede hacer 0, 1 o más elementos de una tarea. El tiempo
  que cada operario tarda en hacer cada elemento es diferente y random.
  Maximice la concurrencia.

    sem s_operario[ M ]= 0   
    
    tarea_asignada[ M ]= 0
    sem s_tarea_asignada[M]= 1
    
    tarea_operarios[ N, op[1..5] ] = [0, 0]

    elementos_tarea[ N ]= 0
    sem s_elementos_tarea[N]= 1

    sem s_barrera[N]=0
    contador_barrera[N]=0
    sem s_e_contador[N]=1

    sem s_cola= 1
    Cola cola

     process operario[ id= 1 to M ]
      var mi_tarea

      P( s_cola )
      push( cola, id )
      V( s_cola )
      P( s_operario[id] )
      > me asignaron tarea en tarea_asignada[id]
      > producir elementos
      P( s_tarea_asignada[id] )
      mi_tarea= s_tarea_asignada[id] 
      V( s_tarea_asignada[id] )
      P(s_elemento_tarea[mi_tarea])
      > mientras no se termino la tarea produzco elementos
      while ( elementos_tarea[mi_tarea] < X ){
        elemento_tarea[mi_tarea]++
        V(s_elemento_tarea[mi_tarea])
        producirElemento( mi_tarea )
      }
      > espero que terminen de producir el elemento 
      P(s_e_contador[mi_tarea])
      contador_barrera[mi_tarea]++
      if (contador_barrera[mi_tarea] < 5)
        V(s_e_contador[mi_tarea])
        P(s_barrera[mi_tarea])
      else
        for i=1 to 4
          V(s_barrera[mi_tarea])
      > se retira el grupo

      process fabrica
      cont_empleados= 0
      cont_cinco= 0
      tarea = 1

      while ( cont_empleados < M ) { 
        P( s_cola )
        if ( cola.not_empty )
          pop( cola, id )
          V( s_cola )
          cont_empleados++
          cont_cinco++
          if ( cont_cinco <= 5)
            P(s_tarea_asignada[ id ])
            tarea_asignada[ id ]= tarea  
            V( s_tarea_asignada[ id ] )
            push( cola_grupo, id )
          if ( cont_cinco == 5 )
            for g = 1 to 5  
              pop( cola_grupo, id ) 
              V( s_operario[ id ] )  
            tarea++
            cont_cinco = 0
        }

## Ejercicio 8:

Resolver el siguiente problema. Existen N camiones que deben descargar
cereales en una acopiadora. Los camiones se descargan de a uno por vez, y de
acuerdo al orden de llegada. Una vez que el camión llegó, espera a lo sumo 2
hs. a que le llegue su turno para comenzar a descargar su cereal, sino se
retira sin realizar la descarga.
Cuando un camión termina de realizar la descarga de su contenido, se debe
encargar de avisarle al siguiente camión (en caso de que haya alguno
esperando) que le llegó su turno para descargar.

    Cola cola
    sem s_cola= 1

    sem s_turno[ N ]= 0
    sem s_timer[ N ]= 0
    
    estados[ N ] = "esperando"
    sem s_esados[ N ]= 1

    process camion [ id= 1 to N ] 

      >si la cola esta libre 
      >   digo que es mi turno
      >
      > activo el timer
      > me encolo
      > P( mi turno )

      P( s_cola )
      if ( cola.empty )
        V( s_cola )
        V( s_turno[ id ] )
        p( s_cola )
      V( s_cola )

      P( s_cola )
      push( cola, id )
      V( s_cola )
      V( s_timer[ id ] )

      P( s_turno[ id  ] )
      
      P( s_estados[ id ] )
      if ( estados[ id ] <> "me fui" )
        estados[ id ]= "descargando" 
        V( s_estados[ id ] )
        descargarCamion( id )
        P( s_estados[ id ] )
      V( s_estados[ id ] )
        
      descargarCamion( id )

      if ( cola.not_empty )
        P( s_cola )
        pop( cola,next_id )
        if ( estado[ next_id ]= "esperando" )
          V( s_turno[ next_id ] )
        V( s_cola )

     process timer [ t= 1 to N ] 
      P( s_timer[ t ] )
      DELAY( 2hs )
      P( s_estados[ t ] )
      if ( estado[ t ] == "esperando" )
        estado[ t ]= "me fui"
        V( s_turno[ t ] )
      V( s_estados[ t ] )

