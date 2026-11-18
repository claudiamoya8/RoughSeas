# Briefing del proyecto: "Mar Revuelto" (nombre provisional)

Vas a ayudarnos a desarrollar un videojuego en Unity. Este documento explica desde cero el contexto, la idea y todas las reglas que ya hemos decidido. Léelo entero antes de proponer nada.

---

## 1. Contexto

- **Quiénes somos:** dos estudiantes de IMAT (Universidad Pontificia Comillas, ICAI). Es el proyecto final de la asignatura *Paradigmas de Programación*.
- **Experiencia:** sabemos C#, pero tenemos poca experiencia con Unity. Explícanos los pasos de Unity con claridad.
- **Motor:** Unity 6 (el enunciado pide la versión 6000.6.8f1).
- **Qué se evalúa:** pesa mucho más la calidad del código que el arte: programación orientada a objetos, patrones de diseño y un diagrama UML que justifique la arquitectura. El arte y el sonido suman, pero no son lo principal.
- **Requisitos obligatorios del enunciado:**
  - Menú principal.
  - Bucle de juego cerrado: jugar → victoria o derrota → volver a jugar, reiniciándose correctamente cada vez.
  - Sistema de puntuación.
  - Al menos un oponente con IA que tome decisiones para que el jugador pierda.
  - Que se pueda compilar y jugar en PC o Mac sin fallos.
- **Deseable según el enunciado:** objetos que se crean y se destruyen en cada partida, y personajes animados.
- **Bonus:** una IA entrenada por nosotros con el paquete ML-Agents de Unity. **Nosotros la vamos a hacer sí o sí.** Se valora que la información del entorno que recibe la IA sea la adecuada para su tarea.
- **Entregas:**
  1. **3/12/2026:** descripción del proyecto (objetivo, funcionamiento, esquema general) y diseño UML de la arquitectura, detallando los patrones de diseño.
  2. **10/01/2027:** juego compilado más el proyecto de Unity (sin las carpetas Library, Temp, UserSettings, Logs ni Obj), el archivo `.yaml` de configuración del entrenamiento de ML-Agents y el proyecto de entrenamiento si es distinto.
- **Repositorio:** usamos un repositorio de GitHub para la documentación y el código.

**Muy importante:** queremos que el juego sea **sencillo**. No añadas mecánicas ni sistemas nuevos sin preguntarnos antes. Si algo te parece mejorable, propónlo, pero respeta las decisiones de este documento.

---

## 2. La idea en una frase

Guías el tráfico de barcos entre islas dibujando sus rutas con el ratón, para que lleguen a su destino sin chocar entre ellos ni contra las rocas, mientras un barco pirata controlado por una IA entrenada con ML-Agents intenta hundirlos a cañonazos.

Inspiración: juegos de dibujar rutas como *Harbor Master* o *Flight Control*. Lo propio de nuestro juego es el pirata entrenado y el mapa con estrechos.

---

## 3. Vista y escenario

- **Vista:** 3D en estilo low-poly, con una cámara fija en perspectiva que mira el mar en diagonal desde arriba. El jugador no mueve la cámara; como mucho, se balancea un poco como efecto visual. Toda la jugabilidad ocurre sobre el plano del agua, así que la lógica es prácticamente 2D.
- **Mapa:** uno solo, diseñado a mano. Es un rectángulo de mar con:
  - **3 islas:** roja, azul y amarilla.
  - **Dos puertos en cada isla:**
    - **Puerto de salida:** es donde aparecen los barcos. Siempre mira hacia mar abierto, de forma que un barco que sale en línea recta tarde al menos 5 segundos en chocar con algo.
    - **Puerto de entrada:** es el destino de los barcos con la bandera del color de esa isla.
  - **Rocas** sueltas repartidas por el mar.
  - **Un arrecife en todo el borde del mapa.** Si un barco llega al borde, se hunde.
  - **2 o 3 estrechos** entre islas y rocas por los que solo cabe un barco a la vez. Son atajos, pero también donde más fácil es chocar.
- **Arte:** packs gratuitos low-poly, por ejemplo el *Pirate Kit* de Kenney (licencia CC0).

---

## 4. Barcos mercantes

| Tipo | Velocidad (valor inicial) | Aspecto |
|---|---|---|
| Lento | 1 | Barco de carga grande |
| Normal | 1,5 | Velero |
| Rápido | 2,2 | Lancha |

- Cada barco lleva una **bandera del color de su isla de destino** y aparece en el puerto de salida de **otra** isla.
- **Si no tiene ruta**, avanza en línea recta hasta que el jugador se la dibuje o hasta chocar. No da la vuelta en ningún caso.
- **Llegada:** si entra en el puerto de entrada de su color, desaparece y cuenta como entregado. Si pasa por un puerto de otro color, no atraca y sigue su camino.
- **Se hunde si choca con:**
  - otro barco mercante (se hunden los dos);
  - una roca, la costa de una isla o el arrecife del borde;
  - el barco pirata;
  - una bala del pirata.
- **Aviso:** cuando dos barcos están a punto de chocar, se marcan en rojo.

---

## 5. Cómo se dibujan las rutas (la mecánica principal)

- El jugador hace clic izquierdo sobre un barco y, sin soltar, arrastra el ratón. Se dibuja una línea sobre el agua, del color del barco.
- Al soltar el botón, el barco sigue la línea suavizando las curvas, y la línea se va borrando según avanza.
- Si la línea acaba dentro del puerto de entrada correcto, el barco atraca al llegar. Si acaba en otro sitio, al terminar la línea el barco sigue recto desde ahí.
- Se puede volver a dibujar la ruta de un barco en cualquier momento, y la nueva sustituye a la anterior.

**Dónde está el reto:** con muchos barcos, las rutas tienen que cruzarse por fuerza, sobre todo en los estrechos. No basta con dibujar bien: hay que calcular **cuándo** pasa cada barco por cada punto, y por eso importan las velocidades distintas. Para hacer esperar a un barco, el jugador le dibuja un rodeo o una vuelta. **No hay botón de parar.**

---

## 6. El barco pirata (el oponente)

- Es un barco negro que entra en el mapa a los 30 segundos de partida.
- Se mueve libremente: no sigue rutas, el jugador no lo controla y no se le puede hundir.
- **Cañón:** alcance de 6 unidades y 3 segundos de recarga.
- **Balas:** van en línea recta y **tardan en llegar**. Si el barco objetivo cambia de rumbo, la bala puede fallar.
- **Cómo se defiende el jugador:** solo redibujando rutas para alejar los barcos del pirata o cambiarles el rumbo en el último momento.

### 6.1 IA del pirata con ML-Agents

| Aspecto | Definición |
|---|---|
| **Observaciones** | Su propia posición y velocidad; la posición y velocidad **relativas** de los 3 barcos mercantes más cercanos; si el cañón está cargado; sensores de rayos 3D que detectan islas, rocas, arrecife y barcos |
| **Acciones (discretas, 3 ramas)** | Girar: izquierda, recto o derecha. Velocidad: frenar, mantener o acelerar. Disparar: sí o no |
| **Recompensas** | +1 por hundir un barco · −0,1 por fallar un disparo · −0,5 por chocar con una roca, una isla o el arrecife · una pequeña penalización por paso para que no se quede quieto |
| **Entrenamiento** | En una escena aparte, con barcos mercantes que siguen rutas aleatorias y **a veces cambian de rumbo** (si siempre fueran rectos, el pirata aprendería algo que no sirve contra un jugador). Entre 8 y 16 copias del escenario a la vez, con el tiempo acelerado. Algoritmo PPO (el de ML-Agents por defecto). Hay que guardar el `.yaml` para la entrega |
| **Dificultad** | Fácil: un modelo guardado al principio del entrenamiento. Difícil: el modelo final |
| **Plan B** | Un pirata con máquina de estados: patrulla, persigue al barco más cercano y dispara cuando lo tiene a tiro. Sirve también de modo fácil si el entrenamiento falla |

**Comportamiento que esperamos que aprenda:** apostarse cerca de los estrechos, porque es por donde los barcos tienen que pasar.

---

## 7. La partida

- **Menú principal:** Jugar (eligiendo dificultad Fácil o Difícil), Récords y Salir.
- **Controles:** clic izquierdo y arrastrar para dibujar rutas; Esc para pausar.
- **En pantalla:** barcos entregados (x/20), barcos hundidos (x/3), tiempo y botón de pausa.
- **Ritmo de aparición:**
  - Al principio sale un barco cada 8 segundos.
  - Cada 30 segundos el intervalo baja 1 segundo, hasta un mínimo de 3.
  - El puerto de salida y el tipo de barco se eligen al azar.
- **Victoria:** entregar 20 barcos.
- **Derrota:** que se hundan 3 barcos.
- **Puntuación:** 100 puntos por barco entregado, más un bonus por terminar rápido. Se guardan los 5 mejores récords.
- **Fin de partida:** sale la pantalla de victoria o derrota con la puntuación, y botones para volver a jugar o ir al menú.
- **Reinicio:** al volver a jugar se eliminan o reciclan todos los barcos, balas y el pirata, y todo vuelve al estado inicial.

Todos los números (velocidades, tiempos, alcance) son **valores iniciales** que ajustaremos probando.

---

## 8. Arquitectura prevista (base del UML)

| Clase | Responsabilidad | Patrón |
|---|---|---|
| `GameManager` | Estados de la partida: menú, jugando, pausa, victoria y derrota | State, Singleton |
| `GeneradorBarcos` | Crea barcos en los puertos de salida y reutiliza los hundidos | Factory, Object Pool |
| `Barco` (abstracta) | Movimiento y choques comunes a todos los barcos | Herencia |
| `BarcoMercante` | Sigue su ruta y atraca. Los datos de cada tipo (velocidad, modelo) van en ScriptableObjects | — |
| `BarcoPirata` | Recibe órdenes de un controlador intercambiable: agente de ML-Agents o máquina de estados | Strategy |
| `DibujadorRutas` | Convierte el arrastre del ratón en una lista de puntos sobre el agua y la asigna al barco | Command (asignar ruta) |
| `Ruta` | Lista de puntos que sigue un barco mercante | — |
| `Puerto` / `Isla` | Puerto de salida y de entrada con su color | — |
| `Bala` | Proyectil del pirata | Object Pool |
| `Marcador`, `HUD`, `Records` | Escuchan los eventos "barco entregado" y "barco hundido", y actualizan puntuación, interfaz y récords | Observer |

Pista técnica para dibujar rutas: lanzar un rayo desde la cámara en la posición del ratón (`Camera.ScreenPointToRay`), cruzarlo con el plano del agua, añadir un punto cada cierta distancia y pintar la línea con un `LineRenderer`.

---

## 9. Solo si sobra tiempo (no hacer sin consultarnos)

- Viento que acelera o frena los barcos según su dirección. Solo cambia la velocidad: nunca desvía a los barcos de su ruta.
- Más de un pirata.
- Más mapas.

---

## 10. Cómo queremos que nos ayudes

- Mantén el juego sencillo y respeta las reglas de este documento.
- Antes de escribir código grande, propón la estructura de clases y espera nuestro visto bueno.
- Explica los pasos dentro del editor de Unity (escenas, prefabs, componentes), porque tenemos poca experiencia con él.
- Prioriza un código limpio y orientado a objetos, con los patrones de la tabla, porque es lo que más se evalúa.
- Trabajaremos por fases, y cada fase tiene que quedar jugable:
  1. Mapa, barcos, dibujar rutas, entregas, choques y puntuación (sin pirata).
  2. Pirata con máquina de estados.
  3. Pirata entrenado con ML-Agents.
  4. Menús, récords, pulido y extras si sobra tiempo.
