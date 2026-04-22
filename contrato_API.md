# Documentación de la API de Código Secreto

Esta documentación detalla los endpoints disponibles en el backend del proyecto Código Secreto, incluyendo descripciones de su uso y los atributos intercambiados en cada petición y respuesta.

## 1. Autenticación (`/api/auth`)

### POST `/api/auth/login`
- **Descripción:** Recibe el idToken de Google para autenticar al usuario.
- **Recibe (Body - `LoginRequestDTO`):**
    - `id_google` (String): El idToken proporcionado por Google.
- **Envía (Response - `AuthResponseDTO`):**
    - `es_nuevo` (boolean): `true` si el jugador no existe y debe registrarse.
    - `token` (String): JWT de sesión (solo si `es_nuevo` es `false`).
    - `jugador` (`JugadorDTO`): Perfil del jugador (solo si `es_nuevo` es `false`).
    - `partida_activa_id` (Integer): ID de la partida en curso si el jugador tiene una pendiente.

### POST `/api/auth/registro`
- **Descripción:** Crea un nuevo jugador tras la verificación de Google.
- **Recibe (Body - `RegistroRequestDTO`):**
    - `id_google` (String): Identificador único de Google (sub).
    - `tag` (String): Nombre de usuario elegido.
- **Envía (Response - `AuthResponseDTO`):**
    - `es_nuevo` (boolean): `false`.
    - `token` (String): JWT de sesión generado.
    - `jugador` (`JugadorDTO`): Perfil del jugador recién creado.
    - `partida_activa_id` (Integer): `null` normalmente.

### POST `/api/auth/logout`
- **Descripción:** Invalida la sesión del usuario eliminando la cookie HttpOnly.
- **Recibe:** Nada.
- **Envía:** 204 No Content.

### PUT `/api/auth/desactivar`
- **Descripción:** Realiza un borrado lógico de la cuenta del jugador e invalida la sesión.
- **Recibe:** Nada (identifica al usuario por la sesión).
- **Envía:** 204 No Content.

---

## 2. Jugadores (`/api/jugadores`)

### GET `/api/jugadores`
- **Descripción:** Obtiene el perfil completo del jugador autenticado.
- **Recibe:** Nada.
- **Envía (Response - `JugadorDTO`):**
    - `id_google`, `tag`, `foto_perfil`, `balas`, `activo` (boolean).
    - `partidas_jugadas`, `victorias`, `derrotas`, `num_aciertos`, `num_fallos`, `porcentaje_victorias` (double).
    - `marco_carta_equipado`, `fondo_tablero_equipado` (String - URLs).

### PUT `/api/jugadores`
- **Descripción:** Actualiza el tag y/o la foto de perfil del jugador.
- **Recibe (Body - `ActualizarPerfilDTO`):**
    - `tag` (String, opcional).
    - `foto_perfil` (String, opcional - URL).
- **Envía (Response - `JugadorDTO`):** Perfil actualizado.

### GET `/api/jugadores/temas`
- **Descripción:** Obtiene la lista de packs de palabras (temas) adquiridos por el jugador.
- **Recibe:** Nada.
- **Envía (Response - List<`TemaInventarioDTO`>):**
    - `id_tema`, `nombre`, `descripcion`.

### GET `/api/jugadores/historial`
- **Descripción:** Obtiene las últimas 30 partidas jugadas por el usuario.
- **Recibe:** Nada.
- **Envía (Response - List<`PartidaResumenDTO`>):**
    - `id_partida`, `codigo_partida`, `fecha_fin`, `estado`, `rojo_gana` (Boolean).
    - `equipo` ("rojo"/"azul"), `rol` ("lider"/"agente"), `abandono` (boolean).
    - `num_aciertos`, `num_fallos`, `tag_creador`.

### GET `/api/jugadores/logros`
- **Descripción:** Obtiene la colección de logros y medallas con su progreso.
- **Recibe:** Nada.
- **Envía (Response - List<`LogroDTO`>):**
    - `id_logro`, `es_logro` (boolean), `nombre`, `descripcion`.
    - `progreso_actual`, `progreso_max`, `completado` (boolean), `balas_recompensa`.

### GET `/api/jugadores/personalizaciones`
- **Descripción:** Obtiene los cosméticos (marcos, fondos) adquiridos.
- **Recibe:** Nada.
- **Envía (Response - List<`PersonalizacionInventarioDTO`>):**
    - `id_personalizacion`, `nombre`, `tipo` ("carta"/"tablero"), `valor_visual` (URL), `equipado` (boolean).

### PUT `/api/jugadores/equipar`
- **Descripción:** Equipa o desequipa un cosmético.
- **Recibe (Body - `EquiparItemRequest`):**
    - `id_personalizacion` (Integer).
    - `equipado` (boolean).
- **Envía:** 200 OK.

---

## 3. Partidas y Lobby (`/api/partidas`)

### POST `/api/partidas/`
- **Descripción:** Crea una nueva partida (privada o pública).
- **Recibe (Body - `CrearPartidaDTO`):**
    - `id_tema` (Integer), `tiempo_espera` (int: 30-120), `max_jugadores` (int: 4-16), `es_publica` (boolean).
- **Envía (Response - `LobbyStatusDTO`):** Estado inicial del lobby de la partida creada.

### POST `/api/partidas/{codigo_partida}/unirse/privada`
- **Descripción:** Se une a una partida privada usando su código alfanumérico.
- **Recibe:** `codigo_partida` en la URL.
- **Envía (Response - Integer):** El ID interno de la partida para redirigir al jugador.

### POST `/api/partidas/{id_partida}/unirse/publica`
- **Descripción:** Se une a una partida pública disponible.
- **Recibe:** `id_partida` en la URL.
- **Envía:** 200 OK.

### DELETE `/api/partidas/{id_partida}/participantes`
- **Descripción:** Abandona una partida en curso.
- **Recibe:** `id_partida` en la URL.
- **Envía:** 200 OK.

### GET `/api/partidas/{id_partida}/participantes/rol`
- **Descripción:** Obtiene el rol y equipo asignado al jugador en una partida.
- **Recibe:** `id_partida` en la URL.
- **Envía (Response - `RolPartidaDTO`):**
    - `rol` ("lider"/"agente"), `equipo` ("rojo"/"azul"), `equipo_inicial`.

### GET `/api/partidas/{id_partida}/lobby`
- **Descripción:** Obtiene el estado completo del lobby (jugadores, configuración).
- **Recibe:** `id_partida` en la URL.
- **Envía (Response - `LobbyStatusDTO`):**
    - `id_partida`, `codigo_partida`, `estado` ("esperando", "en_curso", "finalizada").
    - `max_jugadores`, `es_publica`, `id_tema`, `nombre_tema`, `tiempo_espera`.
    - `tag_creador`, `hay_minimo` (boolean).
    - `jugadores` (List de `JugadorLobbyDTO` con `tag`, `foto_perfil`, `equipo`).

### GET `/api/partidas/publicas`
- **Descripción:** Lista todas las partidas públicas en espera.
- **Recibe:** Nada.
- **Envía (Response - List<`PartidaPublicaDTO`>):**
    - `id_partida`, `tag` (creador), `nombre` (tema), `tiempo_espera`, `max_jugadores`, `jugadores_actuales`.

### PUT `/api/partida/{id_partida}/iniciar`
- **Descripción:** El creador inicia la partida (cambia estado a 'en_curso' y asigna roles).
- **Recibe:** `id_partida` en la URL.
- **Envía:** 200 OK.

---

## 4. Clasificación y Social (`/api`)

### GET `/api/leaderboard/global`
- **Descripción:** Ranking global de jugadores por número de victorias.
- **Recibe:** Nada.
- **Envía (Response - List<`RankingDTO`>):**
    - `tag`, `foto_perfil`, `victorias`.

### GET `/api/leaderboard/amigos`
- **Descripción:** Ranking entre los amigos del jugador.
- **Recibe:** Nada.
- **Envía (Response - List<`RankingDTO`>):** Igual que el global.

### GET `/api/amigos`
- **Descripción:** Obtiene la lista de amigos aceptados.
- **Recibe:** Nada.
- **Envía (Response - List<`RankingDTO`>):** Lista de amigos con su perfil básico.

### GET `/api/amigos/solicitudes`
- **Descripción:** Obtiene las solicitudes de amistad pendientes de recibir.
- **Recibe:** Nada.
- **Envía (Response - List<`AmistadDTO`>):**
    - `id_solicitante`, `tag_solicitante`, `foto_perfil_solicitante`, `fecha_solicitud`, `estado`.

### POST `/api/amigos/solicitudes`
- **Descripción:** Envía una solicitud de amistad.
- **Recibe (Body):** `{ "tag_receptor": "Nombre#1234" }`
- **Envía:** 200 OK.

### PUT `/api/amigos/solicitudes`
- **Descripción:** Acepta o rechaza una solicitud.
- **Recibe (Body):** `{ "id_solicitante": "...", "estado": "aceptada" / "rechazada" }`
- **Envía:** 200 OK.

### GET `/api/jugadores/buscar`
- **Descripción:** Busca jugadores por tag para enviar solicitudes.
- **Recibe:** Query param `tag`.
- **Envía (Response - List<`RankingDTO`>):** Jugadores encontrados.

---

## 5. Tienda (`/api`)

### GET `/api/temas/activos`
- **Descripción:** Lista los temas disponibles en la tienda.
- **Recibe:** Nada.
- **Envía (Response - List<`TemaDTO`>):**
    - `id_tema`, `nombre`, `descripcion`, `precio_balas`, `comprado` (boolean).

### GET `/api/personalizaciones/activas`
- **Descripción:** Lista los cosméticos disponibles en la tienda.
- **Recibe:** Nada.
- **Envía (Response - List<`PersonalizacionDTO`>):**
    - `id_personalizacion`, `nombre`, `descripcion`, `precio_bala`, `tipo`, `valor_visual`, `comprado` (boolean).

### POST `/api/tienda/comprar/tema`
- **Descripción:** Realiza la compra de un paquete de cartas (tema).
- **Recibe (Body):** `{ "id_tema": Integer }`
- **Envía (Response):** `{ "balas": Integer }` (saldo restante).

### POST `/api/tienda/comprar/personalizacion`
- **Descripción:** Realiza la compra de un cosmético (personalización).
- **Recibe (Body):** `{ "id_personalizacion": Integer }`
- **Envía (Response):** `{ "balas": Integer }` (saldo restante).

---

## 6. Juego (Gameplay)

### GET `/api/partidas/{id_partida}/estado`
- **Descripción:** Carga el estado inicial del tablero y el turno al entrar a jugar.
- **Recibe:** `id_partida` en la URL.
- **Envía (Response - `GameStateDTO`):**
    - `id_partida`, `estado`, `equipo_turno_actual`, `fase_turno` ("esperando_pista"/"votando"), `segundos_restantes`.
    - `num_jugadores_rojo`, `num_jugadores_azul` (int).
    - `cartas_rojas_restantes`, `cartas_azules_restantes`, `rojo_gana`.
    - `pista_actual` (`PistaDTO`): `palabra_pista`, `pista_numero`, `equipo_lider`, `aciertos_turno`.
    - `tablero` (`TableroDTO`): List de `CartaDTO` (`id_carta_tablero`, `palabra`, `fila`, `columna`, `estado`, `tipo` - *el tipo es null si es oculta para el agente*).
    - `votos_turno_actual` (List de `VotoDTO`: `id_carta_tablero`, `tag`, `equipo`).

### GET `/api/partidas/{id_partida}/resultado`
- **Descripción:** Obtiene el estado final de una partida terminada para la pantalla de resultados.
- **Recibe:** `id_partida` en la URL.
- **Envía (Response - `GameStateDTO`):** Incluye todo el tablero revelado y `segundos_restantes` a 0.

### GET `/api/partida/{id_partida}/fin`
- **Descripción:** Obtiene un resumen estadístico del cierre de la partida.
- **Recibe:** `id_partida` en la URL.
- **Envía (Response - `PartidaFinDTO`):**
    - `equipo_ganador` (String: "rojo" o "azul").
    - `aciertos_rojo` (int).
    - `aciertos_azul` (int).

---

## 7. Utilidades y Test

### GET `/api/hello`
- **Descripción:** Endpoint de prueba para verificar que el backend está operativo.
- **Envía:** String de saludo.

---

## 8. WebSockets (STOMP)

### Suscripción a Partidas Públicas
- **Topic:** `/topic/partidas/publicas`
- **Tipo:** `SubscribeMapping` (Envía datos inmediatamente al suscribirse).
- **Envía (List<`PartidaPublicaDTO`>):** Lista de partidas en espera.

### Enviar Mensaje de Chat
- **Topic:** `/app/partidas/{id_partida}/chat`
- **Recibe (Payload - `EnviarMensajeDTO`):**
    - `mensaje` (String).
- **Broadcast (Hacia el equipo):** `/topic/partidas/{id}/chat/{equipo}`
- **Atributos Enviados (`ChatMessageDTO`):** `id_mensaje`, `id_partida`, `id_jugador`, `tag`, `equipo`, `mensaje`, `fecha`, `es_valido`.

### Dar Pista (Solo Líder)
- **Topic:** `/app/partidas/{id_partida}/pista`
- **Recibe (Payload):** `{ "palabra_pista": "string", "pista_numero": int }`
- **Acción:** Emite `PistaDTO` a `/topic/partidas/{id}/pista` y actualiza el estado.

### Votar Carta (Solo Agente)
- **Topic:** `/app/partidas/{id_partida}/votar`
- **Recibe (Payload):** `{ "id_carta_tablero": int, "id_turno": int (opcional) }`
- **Acción:** Registra el voto y comprueba si se resuelve la votación.

### Cambiar Equipo (Lobby)
- **Topic:** `/app/partida/{id_partida}/participantes/equipo`
- **Recibe (Payload):** `{ "equipo": "rojo" / "azul" }`

### Cambiar Tema/Tiempo (Lobby - Creador)
- **Topics:** `/app/partida/{id}/tema`, `/app/partida/{id}/tiempoTurno`
- **Recibe:** ID del tema o segundos de espera respectivamente.

### Abandonar Lobby
- **Topic:** `/app/partida/{id_partida}/abandonarLobby`
- **Descripción:** Notifica que un jugador sale del lobby antes de empezar.

### Temporizador (Servidor -> Cliente)
- **Topic:** `/topic/partidas/{id}/temporizador`
- **Envía (`TemporizadorDTO`):** `id_partida`, `segundos_restantes`.
