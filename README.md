# 🐴 Lotería Mexicana

¡Bienvenido a **Lotería Ranchera**! Una versión digital y temática del clásico juego de la Lotería Mexicana. El objetivo es ser el primero en completar una figura en tu tabla (carta) cantando "¡Lotería!".

Este proyecto es una aplicación web completamente funcional que se ejecuta en el navegador. No requiere instalación.

## 🎮 ¿Qué hace?

La aplicación permite jugar a la Lotería Mexicana de dos maneras principales:

1.  **Modo Vs CPU**: Juega en solitario contra la computadora.
2.  **Modo Multijugador**: Crea o únete a una sala para jugar con amigos. Cada jugador necesita su propio dispositivo (celular, computadora) conectado a internet para acceder a la misma sala.

El juego gestiona toda la lógica: la selección de cartas, el mazo de 54 figuras, el turno de las cartas, el marcado de las figuras en tu tabla y la verificación automática de las diferentes formas de ganar.

## ✨ Características Principales

*   **Modos de Juego**: Contra la CPU y Multijugador en línea.
*   **Selección de Carta**: Cada jugador elige su tabla (carta) de un conjunto de 54 tablas únicas y generadas proceduralmente.
*   **Juego Clásico**: El "cantador" (la computadora en este caso) va mostrando las cartas del mazo principal de forma aleatoria.
*   **Marcado Automático**: Los jugadores marcan las figuras de su tabla que ya han salido. La CPU lo hace automáticamente.
*   **Múltiples Victorias**: El juego reconoce 5 formas de ganar:
    *   Línea Horizontal
    *   Línea Vertical
    *   Diagonal
    *   Cuatro Esquinas
*   **Salas Multijugador**: Crea una sala con un token único de 7 dígitos y un PIN opcional para que tus amigos se unan.
*   **Sincronización en Tiempo Real**: En el modo multijugador, el estado del juego (qué cartas han salido y qué han marcado los demás) se sincroniza entre todos los jugadores mediante una API externa.
*   **Interacción Táctil**: Diseñado para funcionar tanto en computadoras como en dispositivos móviles. Las tablas son fáciles de leer y las celdas se pueden tocar para marcar.

## 🛠️ ¿Cómo lo hace? (Tecnología y Arquitectura)

La aplicación es una **Single Page Application (SPA)** construida con tecnologías web estándar.

### Frontend (La interfaz que ves)

*   **HTML5**: Proporciona la estructura de la página y los diferentes contenedores (popups, tableros, botones) para cada pantalla del juego.
*   **CSS3**: Se encarga de todo el estilo visual.
    *   Tema "ranchero" con colores café, beige y rojo, sombras para dar un efecto 3D y bordes decorativos.
    *   Diseño responsivo que se adapta a diferentes tamaños de pantalla.
    *   Animaciones para darle vida a la interacción, como la aparición de una nueva carta o un botón al presionarlo.
*   **JavaScript (Vanilla JS)**: Es el cerebro de la operación. Todo el código está en un solo archivo dentro de la etiqueta `<script>`. No utiliza frameworks ni librerías externas (aparte de Font Awesome para íconos y Google Fonts para la tipografía).

### Lógica del Juego (El Cerebro)

1.  **El Mazo (`BARAJA`)**: Un array con 54 objetos, cada uno representando una carta de la lotería. Cada objeto tiene un `id` único, un nombre (`n`) y la URL de su imagen (`img`). Las imágenes están alojadas externamente en `catbox.moe`.

2.  **Las Tablas de los Jugadores (`G.cartas`)**:
    *   No hay dos tablas iguales. Cuando la página carga, la función `buildCartas()` genera 54 tablas únicas de 4x4 (16 figuras).
    *   Para cada tabla, se toma una copia del mazo completo y se baraja usando una función de "shuffle determinista" (`semShuffle`). Esto significa que, usando una "semilla" (un número), el orden de las cartas en la tabla es siempre el mismo. Esto es clave para el multijugador, para que todos los jugadores vean las mismas tablas.

3.  **Gestión del Estado (`G`)**: El objeto `G` guarda toda la información del juego en un momento dado:
    *   Modo de juego (CPU o multi).
    *   Índice de la carta seleccionada por el jugador.
    *   Las 54 tablas generadas.
    *   Información de los jugadores (nombre, ID, su tabla y qué celdas tienen marcadas).
    *   El mazo principal (`baraja`), las cartas que han salido (`hist`) y la carta actual.
    *   Estado de las salas multijugador (token, PIN, jugadores listos, etc.).

### Modo Multijugador (El Reto)

Dado que no hay un servidor en tiempo real (como WebSockets), la sincronización se logra mediante un método de **"polling"** (consulta periódica) a una **API REST externa**.

*   **La API (`API`)**: El código se conecta a un servicio externo (en este caso, un mockapi.io) que actúa como una base de datos. Cada sala de juego es un "recurso" en esta API.
*   **Crear Sala**: Cuando un jugador crea una sala, su navegador hace una petición `POST` a la API para crear un nuevo registro con la información de la sala (token, PIN, jugadores, etc.).
*   **Unirse a Sala**: Al unirse, el navegador del segundo jugador busca (`GET`) la sala por su token y luego actualiza (`PUT`) el registro de la sala para añadirse a la lista de jugadores.
*   **Sincronización del Juego**:
    1.  **El "Host" (quien creó la sala)** es el encargado de "cantar" las cartas. Su navegador tiene el mazo principal y, cada 3 segundos, toma una carta y la añade al historial.
    2.  **Compartir el Progreso**: Cada vez que el "host" saca una carta, actualiza el registro de la sala en la API, guardando el `historialIds` (los IDs de las cartas que han salido).
    3.  **Los "Clientes" (los otros jugadores)** tienen un `setInterval` en su navegador que cada 2 segundos le pregunta a la API (`GET`) por el estado de la sala.
    4.  Si el historial en la API es más largo que el historial local del cliente, el cliente avanza su propio mazo (usando la misma semilla que el host) hasta igualar el historial del host. De esta manera, todos ven las mismas cartas salir al mismo ritmo (con un pequeño retraso de red).
    5.  **Marcado de Celdas**: Cuando un jugador marca una celda en su tabla, su navegador envía esa información a la API (`PUT`), actualizando su lista de celdas marcadas (`marcadosPorJugador`). Los demás jugadores, en su siguiente consulta, verán esta actualización y el tablero del otro jugador se marcará automáticamente en sus pantallas.

En resumen, la aplicación usa un enfoque de "base de datos compartida" para simular un juego multijugador, donde el "host" actúa como la fuente de verdad para el mazo y cada jugador es responsable de reportar sus propias jugadas.

---

## 🚀 Cómo usarlo

1.  **Abre el archivo `index.html`** en cualquier navegador web moderno (Chrome, Firefox, Safari, Edge).
2.  **Selecciona un modo de juego**.
3.  **Elige tu carta** de la galería. Las cartas que ya han sido seleccionadas por otros (o por la CPU) aparecerán bloqueadas.
4.  **Sigue las instrucciones en pantalla**:
    *   **Vs CPU**: Ponle un nombre a tu jugador y presiona "¡Jugar!".
    *   **Multijugador**: Decide si crear una sala o unirte a una existente con el token y PIN proporcionados.
5.  **En la sala de espera**, todos los jugadores deben presionar "¡Estoy Listo!". El anfitrión podrá iniciar la partida cuando todos lo estén.
6.  **Durante el juego**:
    *   En la parte superior verás la carta que ha "cantado" el sistema.
    *   En tu tablero (el que tiene borde dorado y dice "⭐ TU CARTA"), haz clic o toca las figuras que coincidan con la carta actual para marcarlas.
    *   El sistema verificará automáticamente si alguien ha ganado después de cada marcada.

¡Disfruta la partida!
