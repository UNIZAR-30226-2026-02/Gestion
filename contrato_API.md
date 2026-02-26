# Contrato de API - Código Secreto (Secret Panda S.L.)

En este archivo se concretan las URLs (endpoints) a utilizar para la comunicación entre el backend y el frontend.

---

## 1. API REST (Peticiones HTTP - Gestión fuera de partida)

**URL Base:** `https://api.codigosecreto.com/api`
**Autenticación:** Cabecera `Authorization: Bearer <token_google>` obligatoria (salvo en `/auth/login`).

### 1.1. Gestión de Usuarios y Perfil

#### `POST /auth/login`
* **Funcionalidad:** Punto de entrada al sistema tras la validación de Google (RF-1, RF-2).
* **Body Frontend:** `{"id_google": "token_proporcionado_por_google"}`
* **Comportamiento Backend:** 1. Verifica el token con la API de Google Auth 2.0 (RNF-7).
  2. Busca en la tabla `JUGADOR`. Si no existe, hace un `INSERT` con estadísticas a 0 y balas iniciales.
  3. Ejecuta el método `iniciarSesionGoogle()`.
* **Respuesta (200 OK):** Objeto `JUGADOR` completo.

#### `GET /jugadores/{id_google}`
* **Funcionalidad:** Obtener el perfil y las estadísticas globales (RF-3).
* **Comportamiento Backend:** Hace un `SELECT` en `JUGADOR` para traer `victorias`, `partidas_jugadas`, `numAciertos`, `numFallos` y `balas`. Ejecuta `mostrarEstadisticas()`.
* **Respuesta (200 OK):** Objeto `JUGADOR`.

#### `PUT /jugadores/{id_google}`
* **Funcionalidad:** Modificación del perfil del usuario (RF-6).
* **Body Frontend:** `{"tag": "NuevoPanda", "fotoPerfil": "uri_imagen_perfil.png"}`
* **Comportamiento Backend:** Hace un `UPDATE` en la tabla `JUGADOR` validando que el `tag` (que es Unique Key) no esté ya cogido por otro jugador. Ejecuta `listalmgPerfil()` para asegurar que la foto es una de las 5 válidas.
* **Respuesta (200 OK):** Objeto `JUGADOR` actualizado. Error `409 Conflict` si el tag ya existe.

#### `GET /jugadores/{id_google}/historial`
* **Funcionalidad:** Consultar partidas pasadas (RF-4).
* **Comportamiento Backend:** Ejecuta `listarHistorial()`. Hace un `JOIN` entre `JUGADOR_PARTIDA` y `PARTIDA` donde el jugador participó. **CRÍTICO:** Se aplica un `LIMIT 30` ordenado por `fechaFin DESC`.
* **Respuesta (200 OK):** Array de objetos combinando datos de `PARTIDA` y `JUGADOR_PARTIDA`.

---

### 1.2. Logros y Leaderboards

#### `GET /jugadores/{id_google}/logros`
* **Funcionalidad:** Obtener el progreso de la colección de medallas (RF-5).
* **Comportamiento Backend:** Ejecuta `listarLogros()`. Cruza las tablas `LOGRO` y `JUGADOR_LOGRO`.
* **Respuesta (200 OK):** Array de objetos mezclados: `[{"id_logro": 1, "nombre": "Novato", "progreso_actual": 10, "completado": true}]`


#### `GET /leaderboard/global`
* **Funcionalidad:** Clasificación mundial de jugadores (RF-25).
* **Comportamiento Backend:** Ejecuta `listarRankingGlobal()`. Hace un `SELECT` en `JUGADOR` ordenado por `victorias DESC`. Aplica un límite (ej. top 100).
* **Respuesta (200 OK):** Array de objetos `JUGADOR` **completos**.
  *Nota: El frontend debe filtrar y mostrar solo los atributos públicos (tag, fotoPerfil, victorias) en la UI, pero recibe el objeto completo para coherencia con el resto de endpoints.*


#### `GET /leaderboard/amigos/{id_google}`
* **Funcionalidad:** Clasificación restringida a los amigos (RF-25).
* **Comportamiento Backend:** Ejecuta `listarRankingAmigos()`. Obtiene los amigos de `AMISTAD` (estado "aceptada") y hace el ranking incluyendo al propio jugador.
* **Respuesta (200 OK):** Array de objetos `JUGADOR` **completos** ordenados por victorias.
  *Nota: El frontend debe filtrar y mostrar solo los atributos públicos en la UI, pero recibe el objeto completo para coherencia.*

---

### 1.3. Sistema de Amigos

#### `POST /amigos/{id_google}/solicitudes`
* **Funcionalidad:** Enviar una petición de amistad (RF-24).
* **Body Frontend:** `{"id_receptor": "tag_o_id_destino"}`
* **Comportamiento Backend:** Ejecuta `enviarSolicitudAmistad()`. Inserta una fila en `AMISTAD` con estado "pendiente" y `fecha_solicitud` actual.
* **Respuesta (201 Created):** Objeto `AMISTAD` creado.


#### `PUT /amigos/solicitudes/{id_solicitante}`
* **Funcionalidad:** Aceptar o rechazar una solicitud entrante.
* **Body Frontend:** `{"estado": "aceptada"}` (o "rechazada")
* **Comportamiento Backend:** Hace un `UPDATE` (o `DELETE` si es rechazada) en la tabla `AMISTAD`.
* **Respuesta (200 OK):** Objeto `AMISTAD` actualizado (o vacío si fue eliminada por rechazo).
  *Nota: Se devuelve el objeto actualizado para mantener coherencia con el resto de endpoints.*

---

### 1.4. Tienda y Personalización

#### `GET /tienda/temas`
* **Funcionalidad:** Obtener el catálogo de la tienda (RF-8).
* **Comportamiento Backend:** Ejecuta `listarTematicas()`. Devuelve filas de la tabla `TEMA` y `PERSONALIZACION`.
* **Respuesta (200 OK):** Array de objetos `TEMA`.


#### `POST /tienda/comprar/{id_google}`
* **Funcionalidad:** Transacción de compra de un aspecto o tema (RF-8).
* **Body Frontend:** `{"id_tema": 3}`
* **Comportamiento Backend:** 1. Ejecuta `verificarBalasDisponibles()` comprobando la tabla `JUGADOR`. Si no hay balas, lanza HTTP `400`.
  2. Ejecuta `restarBalas()` haciendo `UPDATE` en `JUGADOR`.
  3. Hace un `INSERT` en `INVENTARIO_PERSONALIZACION`.
* **Respuesta (200 OK):**
  - `{"balas_restantes": 100, "mensaje": "Compra exitosa"}`
  - **Opcional:** También puede devolverse el objeto `JUGADOR` actualizado para que el frontend tenga el estado completo tras la compra.
  *Nota: Se devuelve el objeto JUGADOR actualizado para coherencia y facilitar la actualización del estado en frontend.*

---

### 1.5. Gestión de Partidas (Lobby)

#### `POST /partidas`
* **Funcionalidad:** Crear una sala nueva (RF-12).
* **Body Frontend:** `{"id_creador": "...", "id_tema": 3, "tiempoEspera": 60, "maxJugadores": 8, "esPublica": true}`
* **Comportamiento Backend:** Inserta en `PARTIDA`, autogenera el `codigo_partida` alfanumérico. Añade al creador a la tabla `JUGADOR_PARTIDA`.
* **Respuesta (201 Created):** Objeto `PARTIDA` completo.

#### `GET /partidas/publicas`
* **Funcionalidad:** Listado para el "Matchmaking" (RF-19).
* **Comportamiento Backend:** Ejecuta `listar Partidas Disponibles()`. Busca partidas con `esPublica = true` y `estado = "ESPERANDO"`. **Validación:** Filtra para devolver solo aquellas cuyo `id_tema` el jugador ya posea en su `INVENTARIO_PERSONALIZACION`.
* **Respuesta (200 OK):** Array de `PARTIDA`.

#### `POST /partidas/{id_partida}/unirse`
* **Funcionalidad:** Entrar al lobby de una partida.
* **Body Frontend:** `{"id_jugador": "...", "codigo_partida": "XJ92K"}` (el código solo si es privada).
* **Comportamiento Backend:** 1. `verificar Codigolnvitacion()` (si aplica).
  2. `maxJugadoresAlcanzado()`: Si la sala está llena, devuelve error `403`.
  3. Inserta fila en `JUGADOR_PARTIDA`.
* **Respuesta (200 OK):** Objeto `JUGADOR_PARTIDA`.

#### `DELETE /partidas/{id_partida}/jugadores/{id_jugador_partida}`
* **Funcionalidad:** Abandonar la partida (RF-11).
* **Comportamiento Backend:** Ejecuta `detectarAbandono Reiterado()`. Elimina o marca como inactivo en `JUGADOR_PARTIDA`. Aplica penalización de -10 balas haciendo `UPDATE` en `JUGADOR`.
* **Respuesta (200 OK):** Vacía.

---
---

## 2. API WebSockets (Tiempo Real - En Partida)

**Protocolo:** STOMP sobre WebSockets (WSS). Latencia estricta < 500ms.

### 2.1. Sincronización del Tablero (El Backend difunde)
* **Canal de Suscripción Frontend:** `/topic/partidas/{id_partida}/estado`
* **Comportamiento Backend:** Cuando la partida arranca (o un jugador se reconecta RNF-1), el backend ejecuta `listarCartas Tablero()` y envía el tablero. 
* **REGLA DE SEGURIDAD (ANTI-TRAMPAS):** Antes de enviar el JSON, el backend comprueba el `rol` de los destinatarios. 
  * A los **Agentes** les sobreescribe el atributo `tipo` de la tabla `TABLERO_CARTA` enviando el valor `"oculta"`. 
  * A los **Jefes**, les envía el valor real `"rojo"`, `"azul"`, `"asesino"`, `"civil"`.
* **Payload difundido:**
```json
{
  "evento": "ACTUALIZACION_TABLERO",
  "partida": { "estado": "EN_CURSO", "turno_actual": "ROJO", "fase": "VOTACION" },
  "tablero": [ { "id_carta_tablero": 1, "id_palabra": "uri_imagen.png", "estado": "OCULTA", "tipo": "oculta" } ]
}
```

## 2.2. Acción: Dar Pista (Jefe de Espionaje)

**Frontend envía a:**  
`/app/partidas/{id_partida}/pista`

**Comportamiento Backend:**  
Inserta un nuevo registro en la tabla `TURNO` con `palabraPista` y `pistaNumero`.  
Activa el temporizador de votación (RF-26).  
Difunde el evento al canal `/topic/partidas/{id_partida}/estado` para que los agentes empiecen a jugar.

**Payload Frontend:**
```json
{
  "id_jugador_partida": 45,
  "palabraPista": "Arma",
  "pistaNumero": 2
}
```

---

## 2.3. Acción: Votar Carta (Agentes)

**Frontend envía a:**  
`/app/partidas/{id_partida}/votar`

**Comportamiento Backend:**

1. Inserta fila en `VOTO_CARTA`.  
2. Ejecuta `gestionarVotos()` calculando la mayoría simple en tiempo real.  
3. Si hay mayoría:
   - Ejecuta `comprobarResultadoVotacion()`.  
   - Cambia el estado en `TABLERO_CARTA` a `"REVELADA"`.  
   - Difunde a todos la carta descubierta.  
4. Comprueba condiciones de victoria (RF-22):
   - ¿Se destapó el asesino?  
   - ¿Se destaparon todas las del equipo?

**Payload Frontend:**
```json
{
  "id_jugador_partida": 48,
  "id_carta_tablero": 12,
  "id_turno": 5
}
```

---

## 2.4. Chat Exclusivo por Equipos

**Frontend envía a:**  
`/app/partidas/{id_partida}/chat`

**Frontend ROJO escucha en:**  
`/topic/partidas/{id_partida}/chat/rojo`

**Frontend AZUL escucha en:**  
`/topic/partidas/{id_partida}/chat/azul`

**Comportamiento Backend:**

1. El backend recibe el mensaje y pasa el filtro de palabras prohibidas (RF-28).  
2. Inserta el mensaje en la tabla `CHAT`.  
3. Difunde el mensaje únicamente al topic del equipo correspondiente (el Jefe de Espías de ese equipo está suscrito, pero su frontend no muestra input de escritura).

**Payload Frontend:**
```json
{
  "id_jugador_partida": 48,
  "mensaje": "Es esa seguro"
}
```