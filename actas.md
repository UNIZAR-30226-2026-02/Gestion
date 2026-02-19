# Actas 

## 18 de febrero de 2026
**Asistentes**: grupo 02

**Tema de la reunión**: Se acuerdan las inconsistencias / dudas / ambigüedades de los requisitos. 

**Duración**: 3h
Se acuerda que:

**Prototipos de pantalla**

Durante la partida, permitir ver qué jugadores están en la partida.
En móvil, dar la opción de ampliar la imagen.

**Funcionalidad del juego**

- Hay un líder de partida que establece la temática de las cartas. Los usuarios pueden personalizar su propio tablero.
- Hay 3 roles en total: jefe de espías, agente de campo e infiltrado. 
- Un usuario puede crear una partida pública, una privada o unirse (a partidas públicas disponibles o a privadas con un código).
- Al unirse a una partida pública, el usuario solo puede unirse a aquellas partidas con alguna temática que él ya tenga adquirida. En el caso de que la partida sea privada esto no ocurre.
- El líder (el creador de la partida) puede elegir todos los parámetros de la temática de cartas al crear la partida → configuración inicial.
- En las partidas privadas, en el lobby el líder puede reconfigurar todos los parámetros de la partida (duración máxima del turno…) y temática de carta.
- En las partidas públicas, en el lobby no se puede reconfigurar nada (solo se espera).
- Hay ranking, falta por concretar los detalles de cómo se realizará la clasificación
- Las temáticas de cartas se compran con balas en la tienda.
- En el lobby se puede hacer la personalización individual de cada jugador (color de fondo, tablero…)

Adicionalmente, se comienzan a definir las issues e hitos del proyecto, con el objetivo de elabrorar el diagrama de Gantt


## 19 de febrero de 2026
**Asistentes**: grupo 02

**Tema de la reunión**: Se redactan los requisitos para evitar las ambigüedades. 

**Duración**: 3h

**Pendientes fruto de la discusión** 
- Posible diccionario de datos: qué se muestra en el historial (fecha de la partida, victoria o derrota, aciertos o fallos), qué se entiende por número de aciertos, qué es un logro y tipos (días seguidos de conexión), medalla (correspondiente a num de partidas ganadas), imagen de perfil : 5 icons a elegir, que entendemos por personalizar (marco de carta y fondo de tablero), tema visual, paquete de cartas, pista, palabra (definir límte) , número, si han votado carta más de la mitad se selecciona carta, el orden de seleccion importa si un error el resto de tirnos invalido, solo puedes entrar a partidas con tu temática, listado de partidsa públicas, vista de jefe (ve el chat pero no participa), seleccionar intervalo, penalizaciones, abandonos reiterados ( 3 partidas seguidas), longitud de una palabra, estándares proteccion de datos, diseño actractivo e intuitivo????, leaderbord

- Se acuerda concretar y compartir el contrato de API's y nombres por parte del equipo de backend, una vez se concrete la base de datos, con los equipos de frontend 

**Comentarios de la entrega** 
- Demo semana del 11 en el resumen ejecutivo NO entrega el 27
- Resumen ejecutivo, versiones a las que nos comprometemos
- 15 de marzo contrato de requisitos INCONTESTABLE
- Tiempo de despliegue: si no, vídeos
- Precio y coste en el resumen ejecutivo e hitos
- Demasiados párrafos en objetivos, da imagen apresurada
- Requisitos y GUI or encima de lo esperado, demasiado detallistas
- Mapear horas con actividades en el presupuesto (issues genéricas maybe)
- Definir hitos compuestos
- Ha dicho que no afecta quitar el matchmaking
- Quitamos la música de fondo 
- Añadir jugador a partida privada
- Se delega toda la autorización e la API