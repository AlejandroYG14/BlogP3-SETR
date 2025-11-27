# BlogP3-SETR

## Introducción
El objetivo de esta práctica ha sido implementar un controlador basado en Arduino UNO para una máquina expendedora.

En mi sistema he intentado que todo funcione de manera fluida y sin bloqueos, sobre todo teniendo en cuenta que estoy leyendo sensores, refrescando la pantalla, gestionando un menú bastante grande y trabajando con un joystick y dos botones. Para lograrlo he tenido que apoyarme en varios recursos del propio microcontrolador.

## Recursos utilizados

### Interrupciones

Las interrupciones las utilizo en los dos botones: el botón principal y el del joystick.
El motivo es sencillo: no quiero perder pulsaciones ni depender de estar preguntando en cada loop si el usuario ha presionado algo. Las interrupciones me garantizan que, en el instante en que se pulsa el botón, el programa lo detecta aunque esté ejecutando otra cosa.
Además, dentro de las interrupciones hago un pequeño “debounce” usando `millis()`, para evitar lecturas falsas.

Con eso, cuando se pulsa un botón, simplemente marco flags (`buttonPressed`, `buttonReleased`, `joystickButtonPressed`) y luego el loop principal es el que decide qué hacer.

### Watchdog

También activo el watchdog a 2 segundos: `wdt_enable(WDTO_2S)`. Lo uso como medida de seguridad: si por cualquier motivo el programa se queda colgado, por ejemplo, una lectura del DHT que se bloquee, o un error lógico inesperado, el micro se reiniciará solo.
Durante el loop voy llamando a `wdt_reset()` para mantenerlo vivo. Si en algún punto el programa deja de funcionar correctamente y no llega a esa llamada, el watchdog hará un reset automático y todo vuelve al estado inicial.

### Thread para sensores

Para no estar leyendo el DHT y el sensor de ultrasonidos todo el tiempo (que ralentizaría mucho el sistema), utilizo una pequeña librería de threads. No son hilos reales a nivel hardware, pero me permiten ejecutar una función de forma periódica sin tener que pensar en ello en el loop.
Creo un thread y le asigno `updateSensors()` y lo configuro para que se ejecute cada 2 segundos. Con eso me aseguro de que las lecturas se actualizan con la frecuencia suficiente, sin bloquear otras partes del programa.

Esto hace que la interfaz (pantalla, joystick, LEDs, etc.) funcione mucho más suave, porque no estoy metiendo lecturas pesadas dentro del loop principal.

### Máquina de estados

La parte principal del programa es una máquina de estados bastante grande. Cada pantalla o modo de funcionamiento tiene su propia función: menú de servicio, preparación del café, menú de administración, edición de precios, etc.
Esto lo hago para no tener que recurrir a delays que bloqueen la ejecución. Todo se gestiona por tiempo con `millis()` y cada estado es totalmente independiente.
Gracias a eso, mientras se prepara un café, por ejemplo, la pantalla se sigue refrescando, el watchdog sigue vivo y todo el sistema permanece activo.

### Lectura no bloqueante del joystick

La lectura del joystick la he controlado con una especie de "filtro" manual usando tiempos y memorias de dirección. No uso interrupciones ahí porque es analógico, pero sí aplico una zona muerta y un retardo de 150 ms para evitar cambios bruscos o ruido.

### Control del LCD sin bloquear

Intento actualizar el LCD sólo cuando cambia realmente algo o cuando ha pasado cierto tiempo. Esto ayuda mucho a que el programa no se ralentice, porque imprimir en el LCD es relativamente lento.

### Gestión del precio con confirmación

Para la edición de precios, almaceno el precio original cuando el usuario entra en modo edición.
Solo cuando confirma con el botón guardo los cambios.
Si se va hacia atrás con el joystick restauro el precio original.

## Circuito
Este es el esquemático de mi circuito, hecho en Fritzing:

<img width="2778" height="1686" alt="circuito" src="https://github.com/user-attachments/assets/658fb9b2-0753-42f7-a8dc-88f44340b6b7" />


## Vídeo
Este es el enlace al vídeo de mi práctica, donde explico las funcionalidades del sistema:

https://youtu.be/VgD4HMpCtr4
