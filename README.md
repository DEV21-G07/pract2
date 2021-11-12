# Practica 2

!! This is a new repository as I had to delete the old one (see issue #1 for explanation and commit history: https://github.com/DEV21-G07/practica2/issues/1)

**New link after reading feedback of practica1 and adding more comments, restructuring files, ...: 
https://drive.google.com/file/d/1YXodqeNlWkXdCCFiZrz7NS9Z5DNaFtGo/view?usp=sharing**

(Old version: https://drive.google.com/file/d/159zH-t63HN4xmSJBOt80BzSgH6qOlySz/view?usp=drivesdk)

----

# El proyecto

El juego existe de un mundo virtual, lo cual debería verse como una fabrica de robots. En ese mundo, el jugador tiene que colectar algunos chips que contienen información sobre la fabrica, y llegar al fin para extraer esa información.

Para llegar al fin, el jugador tiene que recoger tarjetas que le dan entrada a diferentes partes de la fabrica, donde tiene que buscar los chips. Pero mientras está navegándose por la fabrica, el jugador va a encontrarse con robots que le quieren parar y matar.  Los robots perseguirán al jugador si lo pueden ver, si no siguen su camino a puntos fijos y a veces aleatorios (depende del tipo de enemigo - en mi juego hay dos tipos, lo que se explica en https://github.com/DEV21-G07/practica2/wiki/Las-pruebas en la parte sobre "Los enemigos").

El jugador puede disparar al robot para defenderse (pero lo mejor es evitarlos) y puede destruir los spawners de los robots para qué dejan de aparecer en el mundo. Disparando va perdiendo balas, por eso hay paquetes de munición que se puede recoger para tener más balas. Los robots también pueden dañar al jugador, quitándose las vidas. Después de un tiempo esas vidas se recuperan.

----

# Proceso

## Preproducción (Oct 14)

#### Estética 
Primero había que decidir para usar el sci-fi pack  (https://www.unrealengine.com/marketplace/en-US/product/modular-scifi-season-2-starter-bundle) o crear el mundo desde cero. Utilizar el mundo ya hecho podría resultar más difícil, ya que hay muchos detalles y extender ese mundo con muchos detalles podría tomar más tiempo. Pero si pensamos en la estética, seguramente será mejor con el mundo sci-fi, lo que hará el mundo parecer a la realidad más que si yo hubiera hecho el escenario. 

#### Planificación
Habrá que averiguar como hacer los AIs. Por eso también hay que investigar como funcionan los Nav Meshes.

Después de eso, iré haciendo todos los objetos que se necesita para la jugabilidad del juego, antes de preocuparme con como se va a ver el mapa. 

Cuando todo funcione bien, voy a empezar con añadir más partes de la fabrica. Quiero que el mapa sea un pasillo grande, a veces pasando una sala más grande. Así el jugador va abriendo las puertas una por una hasta que llega a la ultima puerta.

A veces hay que haber trampas como objetos un poco escondidos, o pequeñas salas escondidas para que el jugador que no tenga cuidada abra la puerta equivocada.

## Produccion

#### Los AI robots (Oct 15)
En este paso, hice el "EnemyController" y el "EnemyCharacter", seguí este tutorial: https://www.vikram.codes/blog/ai/01-basic-navigation. En este momento puedo poner un EnemyCharacter en el mundo, poner unos TargetPoints y el enemigo ira al azar a los TargetPoints.

Para hacer que el robot sigue el jugador, seguí más partes del tutorial. Pero, en vez de usar la manera de detectar un enemigo del tutorial, usé el "AI Perception", que es menos complicado, ya que es una herramienta incorporada en UE4. También no tenía que implementar el "it behaviour" del tutorial.

Esto también se podría implementar con un "behavior tree", que puede resultar más limpio, pero a mí no me pareció necesario (aunque sería bien ya haberlo hecho para el futuro cuando hay situaciones más complicadas).

#### Perder vidas y balas (Oct 16)
Cuando el robot toca el jugador, pierde una vida, después sigue dañando al jugador cada x segundos. Lo hice con un CollisionBox que es un poco más grande que lo del enemigo, para hacerlo un poco más difícil para el jugador. También puse un delay en el jugador para que cada x segundos el player recupera una vida si no tiene todas sus vidas.

Para mostrar que el jugador está perdiendo vida, hice que su visión tiene que volverse más oscuro, como se va a perder conciencia. Eso lo hice con la cámara del jugador, cada vez que las vidas del jugador cambian, hay una función en la cámara que cambia la visión dependiendo de las vidas del jugador. La cámara pone su ColorTint más oscuro, hasta que sea negro si se muere el jugador.

Para las balas es muy simple: deja de lanzar balas si el jugador ya no tiene munición.

#### Recoger objetos (Oct 16)
Munición: Para esto hice un BluePrint que se ve como munición, en tocarlo, la munición del jugador sube.

Chips: Recoger un chip tiene que incrementar la cantidad de chips que tiene el player. Al fin del juego tiene que tener todos los chips que hay.

Tarjetas de entrada: Esto también tiene que incrementar la cantidad de las tarjetas.

Había algunos problemas con recoger los objetos, así que tenía que añadir collisionBoxes que son más grandes que los objetos para que seguramente el jugador puede recogerlo. 

#### Destruir objetos (Oct 16)
Esto se puede separar en dos fases: destruir los robots y destruir los spawners.

Para destruir el robot, el tiene que reaccionar cuando le toca una bala del jugador, los robots normales tienen 3 vidas. Cuando ya no tiene vidas el robot se destruye.
Para destruir el spawner, funciona igual: el spawner tiene que tomar daño cuando le toca una bala del jugador. 

#### Tarjetas de entradas: Puertas y spawners (Oct 17)
Tuve que implementar puertas que solo se abren si el jugador tiene (por lo menos) una tarjeta de entrada. Esto es necesario para hacer que el jugador solo puede avanzar cuando tiene una tarjeta de entrada. Cuando el jugador se acerca a la puerta, diminuye la cantidad de tarjetas que tiene el jugador.

También el spawner tiene que reaccionar en el momento que el jugador recoge una tarjeta. Cuando el jugador recoge una tarjeta, llama una función del carácter que al fin llama un "Event Dispatcher". Hice un event binding en el spawner, para que cuando el jugador recoge una tarjeta, el delay entro el poner de los robots en el mundo será menos.

Primero puse el "event dispatcher" en el game mode, pero al fin me di cuenta que eso no es necesario, y puede estar en el código del carácter.

#### Comunicación entre los robots (Oct 17)
Esto al principio era un poco más complicado, porque no sabía cuál era la mejor manera de hacerlo. Al fin, esta lógica decidí de ponerla en el spawner, todos los robots del mismo spawner se pueden avisar entre ellos. El spawner tiene una lista de los robots, el robot tiene una referencia del spawner, cuando el robot ve el jugador, le avisa al spawner, quien le avisa a los demás robots.

Con el código principal, había errores cuando un spawner o un robot del spawner está destruido. Porque en ambos casos intentan de llamar a funciones de un objeto que ya no existe. Cuando un robot está destruido, tiene que borrarse de la lista de robots del spawner. 

Cuando un spawner está destruido, robots ya no pueden llamar otros robots. Esto es un "side effect" de la manera que usé yo para comunicarse entre los robots. Pero al fin, me parece bien y realista.

#### Agregar más salas y el fin (Oct 19-20)
A acabar todo el código necesario para hacer el proyecto, pude empezar con añadir más salas y partes del mapa. Esto no era mucho programar, si no utilizar el interface de ue4 para copiar y pegar partes del mapa para hacer más salas.

Al final del juego también implementé una puerta que solo se abre cuando el jugador ha recogido todos los chips.

Las salas y sus pruebas realizadas se encuentra en https://github.com/DEV21-G07/practica2/wiki/Las-pruebas en "El juego".

## Postproducción (Oct 20)
Empece a jugar el juego algunas veces para saber si hay algo demasiado fácil o difícil. Además había unos errores pequeños que tuve que resolver cuando estaba haciendo el testing.

----

# Diseño del juego 
## Jugabilidad

### Mecánica
**Salud**
- El jugador tiene 10 vidas 
- Al tocar un robot, el jugador pierde una vida. Después sigue perdiendo vidas cada 1 segundo.
- El jugador recupera vida cada 5 segundos.

**Combate**
- El jugador pierde balas mientras está disparando.
- Se puede matar a los robots disparándolos 3 veces.
- Se puede destruir los spawners disparándolos 3/4/5 veces (depende del nivel).

**Coleccionar**
- El jugador puede recoger munición caminando sobre el objeto o tocándolo.
- El jugador puede recoger tarjetas de entrada caminando sobre el objeto o tocándolo.
- El jugador puede recoger chips caminando sobre el objeto o tocándolo.
- Se puede abrir puertas si tiene una tarjeta de entrada.
- Se puede abrir la puerta final si tiene todos los chips.

### Dinámica
- El jugador necesita evitar o matar a los robots para que no se muera. Necesita acumular munición para seguir disparando.
- Se necesita coleccionar tarjetas de entradas, para superar a la parte actual del mapa y seguir a una otra parte.
- El jugador se va juntando todos los chips del nivel. Así podrá abrir la ultima puerta y acabar con el juego.

### Estética
- El mundo debería verse como una fabrica real, para que el jugador se siente realmente adentro de una fabrica de robots.
- El jugador necesita tener miedo de no saber de donde vienen los robots, no debería ser obvio donde está el spawner. Al descubrirlo se sentirá aliviado.
- Dará susto descubrir un robot fuerte detrás de una esquina, la apariencia de esos robots darán más miedo que los robots normales.
- En alguna parte el jugador puede abrir la puerta equivocada si va avanzando demasiado rápido, tendrá que volver lo que le da frustración pero también curiosidad por no haber descubierto algo antes.
- En alguna parte también hay que mirar bien que no se ha perdido un chip que está un poco escondido.  Aquí también tendrá que regresar, para buscarlo. Si ha acabado con todos los spawners se sentirá tranquilo, sin mucho apuro, si no será más miedoso y difícil.
- El acercarse de los robots al jugador tiene que darle miedo, cuando está perdiendo vidas también habrá un efecto de la vista que se da cuenta de que se va a perder su vida.



## Contenido
Todas las siguientes partes se encuentra en el wiki:  https://github.com/DEV21-G07/practica2/wiki/Las-pruebas
### Los enemigos
### Los objetos especiales 
### Los niveles
