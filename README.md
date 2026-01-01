# Tuning / Tuning

ES — Español

## Descripción
Tuning es una aplicación web (Flask) de recomendación y reproducción musical que integra:
- Autenticación y datos en Supabase (Postgres + Storage).
- Integración con la API de Spotify para reproducción, búsquedas y recomendaciones.
- Reconocimiento de voz y verificación de hablante (Vosk) para comandos de voz seguros.
- Síntesis de voz para retroalimentación (gTTS) y procesamiento de audio (pydub).
- Paneles para usuarios, artistas y administradores (templates Jinja2).

## Funcionalidades principales
- Registro e inicio de sesión de usuarios.
- Recomendaciones basadas en el género musical y el historial.
- Control de reproducción vía la API de Spotify (play, pause, next, previous, volumen).
- Comandos por voz con identificación de hablante y protección por rol (usuario, artista, admin).
- Gestión de playlists, historial de escucha y métricas administrativas.
- Subida de canciones por artistas (almacenadas en Supabase Storage).

## Stack / Tecnologías
- Backend: Python 3.x, Flask
- Base de datos y storage: Supabase
- Reconocimiento de voz / identificación de hablante: Vosk
- Síntesis de voz: gTTS, pygame (para reproducción local)
- Audio processing: pydub, wave
- Integraciones: Spotify Web API (requests / spotipy)
- Dependencias JS (solo para estáticos/linters): Tailwind, ESLint (package.json incluido)

## Estructura principal del repositorio
- app.py — Aplicación Flask principal (rutas, lógica, integración con Supabase y Spotify)
- requirements.txt — Dependencias Python
- package.json / package-lock.json — Dependencias y scripts para áreas de frontend (lint etc.)
- templates/ — Plantillas Jinja2 usadas por la aplicación
- static/ — Archivos estáticos (CSS / JS)
- fallback_songs.json — Canciones de respaldo por género
- tests/ — Tests y sesiones (ejemplo de pruebas)

## Requisitos previos
- Python 3.9+ recomendado
- pip
- Node.js / npm (solo si vas a trabajar en los assets estáticos o ejecutar lint)
- Modelos Vosk descargados en la carpeta `models/` (ver más abajo)
- Cuenta y proyecto en Supabase con las tablas y buckets que se indican
- Credenciales de Spotify (Client ID / Client Secret) y configuración del Redirect URI

## Variables de entorno (ejemplos)
Define un fichero .env con al menos las siguientes variables:

- SUPABASE_URL
- SUPABASE_KEY
- FLASK_SECRET_KEY
- SECRET_KEY
- SPOTIFY_CLIENT_ID
- SPOTIFY_CLIENT_SECRET
- SPOTIFY_REDIRECT_URI
- SPOTIFY_AUTH_URL (por ejemplo: https://accounts.spotify.com/authorize)
- SPOTIFY_TOKEN_URL (por ejemplo: https://accounts.spotify.com/api/token)
- SPOTIFY_API_BASE_URL (por ejemplo: https://api.spotify.com/v1/)

Ajusta las URLs si usas otro endpoint.

## Preparación de modelos Vosk
La aplicación espera modelos de reconocimiento de voz y de hablante en `models/`:
- models/vosk-model-small-en-us-0.15 (u otro modelo compatible)
- models/vosk-model-small-es-0.42
- models/vosk-model-small-fr-0.22
- models/vosk-model-small-ja-0.22
- models/vosk-model-small-ko-0.22
- models/vosk-model-spk-0.4 (modelo para identificación de hablante)

Descarga los modelos desde la página oficial de Vosk y colócalos en la carpeta `models/`.

## Configuración de Supabase (tablas y buckets sugeridos)
Crea al menos las siguientes tablas con los campos que aparecen por su uso en app.py (los nombres clave se muestran):
- users: user_id (uuid/text), email, name, password, music_genre, language, role (1=admin,2=user,3=artist), speaker_vector (array/JSON), created_at, bio
- playlists: id, user_id, name, songs (JSON array), created_at
- listening_history: id, user_id, song_uri, song_title, song_artist, song_genre, listened_at
- artist_songs: id, user_id, title, file_url, created_at
- comentarios: id, user_id, comment, created_at
- player_state: id, user_id, track_queue (JSON), current_index, is_playing, position_ms, volume, shuffle_mode, repeat_mode, context, updated_at

Storage:
- bucket/tabla `artist_songs` (o nombre equivalente) para almacenar archivos subidos por artistas

Ajusta tipos y reglas de permisos según tus necesidades.

## Instalación (local)
1) Clona el repositorio:
   git clone https://github.com/JohanSE17/Tuning.git

2) Crear y activar entorno virtual (recomendado):
   python -m venv venv
   source venv/bin/activate  # macOS / Linux
   venv\\Scripts\\activate   # Windows

3) Instalar dependencias Python:
   pip install -r requirements.txt

4) (Opcional) Instalar dependencias JS para lint / build de estáticos:
   npm install

5) Crear .env con las variables listadas arriba.

6) Descargar y colocar modelos Vosk en `models/`.

## Ejecutar
- En desarrollo:
  python app.py
  (La app corre por defecto en http://0.0.0.0:5000 con debug=True en app.py)

- En producción: configura un WSGI server (gunicorn/uvicorn) y proxies reversos; revisa las credenciales y desactiva debug.

## Uso rápido
- Regístrate como usuario y sube una muestra de voz (se usa para identificación de hablante en comandos por voz).
- Inicia sesión y conecta tu cuenta Spotify a través de "Login with Spotify".
- Usa el panel del dashboard para ver recomendaciones, historial y crear playlists.
- Usa la ruta /voice-command para enviar archivos de audio y ejecutar comandos por voz (la app procesa, verifica hablante y ejecuta acciones vía la API de Spotify).

## Seguridad y privacidad
- La aplicación implementa medidas básicas: hashing de contraseñas (werkzeug), CSRF (Flask-WTF), sanitización de comentarios (bleach) y verificación de hablante. Aún así, revisa y endurece políticas de CORS, sesiones, y reglas de Supabase antes de exponer en producción.
- No incluyas claves sensibles en el repositorio.

## Tests y lint
- Tests: hay un archivo de prueba en tests/test_tuning.py. Ejecuta las pruebas con tu runner preferido (pytest) si adaptas/añades pruebas.
- Lint JS: npm run lint (usa eslint para archivos en static/js/).

## Contribuir
1) Abre un issue describiendo el cambio / bug.
2) Crea una rama con un nombre descriptivo.
3) Envía un Pull Request con una descripción clara y tests cuando apliquen.

## Problemas comunes / Troubleshooting
- Errores con los modelos Vosk: asegúrate de que los modelos estén en la carpeta `models/` y que las rutas en app.py coincidan.
- Errores con Spotify API: revisa que las credenciales y redirect URI estén correctamente configurados; el token expira y la app tiene lógica para refrescarlo, pero revisa logs.
- Errores con Supabase: valida que las tablas y permisos existan; revisa las consultas en app.py para ver campos necesarios.

## Próximos pasos recomendados
- Añadir un archivo LICENSE (MIT o similar) si quieres permitir contribuciones.
- Añadir integración CI (tests automatizados) y despliegue con Docker.
- Desacoplar partes (servicio de voz, servicio de recomendaciones) para escalabilidad.

EN — English

## Description
Tuning is a Flask-based web application for music recommendations and playback that integrates:
- Authentication and data via Supabase (Postgres + Storage).
- Spotify API integration for playback, search and recommendations.
- Voice commands and speaker verification using Vosk for secure voice actions.
- Voice feedback with gTTS and audio processing with pydub.
- User, Artist and Admin dashboards (Jinja2 templates).

## Main features
- User registration and login.
- Recommendations based on music genre and listening history.
- Playback control via Spotify API (play, pause, next, previous, volume).
- Voice commands with speaker identification and role-based permissions (user, artist, admin).
- Playlist management, listening history and admin metrics.
- Artist song uploads stored in Supabase Storage.

## Stack
- Backend: Python 3.x, Flask
- Database and storage: Supabase
- Speech recognition / speaker ID: Vosk
- TTS: gTTS, pygame (for local playback)
- Audio processing: pydub, wave
- Integrations: Spotify Web API (requests / spotipy)
- JS tools: Tailwind, ESLint (package.json included)

## Repo layout
- app.py — Main Flask app (routes, Supabase & Spotify integration)
- requirements.txt — Python deps
- package.json / package-lock.json — JS deps & scripts
- templates/ — Jinja2 templates
- static/ — Static files (CSS / JS)
- fallback_songs.json — Fallback songs by genre
- tests/ — Tests and session examples

## Prerequisites
- Python 3.9+
- pip
- Node.js / npm (if working on static assets or linting)
- Vosk models in `models/` (see Spanish section above)
- Supabase project and credentials
- Spotify credentials (Client ID / Secret) and Redirect URI

## Environment variables
Set a .env file with at least the following variables:
- SUPABASE_URL
- SUPABASE_KEY
- FLASK_SECRET_KEY
- SECRET_KEY
- SPOTIFY_CLIENT_ID
- SPOTIFY_CLIENT_SECRET
- SPOTIFY_REDIRECT_URI
- SPOTIFY_AUTH_URL
- SPOTIFY_TOKEN_URL
- SPOTIFY_API_BASE_URL

## Vosk models
Place Vosk models inside `models/` (see Spanish section above).

## Supabase setup
Create the tables and storage bucket referenced in the Spanish section.

## Installation (local)
1) Clone the repo:
   git clone https://github.com/JohanSE17/Tuning.git

2) Create & activate virtual env:
   python -m venv venv
   source venv/bin/activate

3) Install Python deps:
   pip install -r requirements.txt

4) (Optional) Install JS deps:
   npm install

5) Create .env and download Vosk models.

## Run
- Development:
  python app.py

- Production: use a WSGI server and ensure debug is off and secrets are secure.

## Quick usage
- Register and upload a voice sample (used for speaker verification).
- Login and connect Spotify via Login with Spotify.
- Use the dashboard to get recommendations, create playlists and see history.
- POST audio to /voice-command to issue voice-controlled Spotify actions.

## Security & privacy
- Passwords are hashed (werkzeug). CSRF enabled via Flask-WTF. Inputs are sanitized where shown (bleach).
- Review CORS, session and Supabase policies before production.

## Tests & lint
- Tests: see tests/test_tuning.py. Run with pytest after adding/adjusting tests.
- Lint: npm run lint (eslint configured for static/js/).

## Contributing
1) Open an issue describing the change or bug.
2) Create a branch.
3) Send a Pull Request with clear description and tests when applicable.

## Troubleshooting
See Spanish section above.

## License
No LICENSE file detected in the repository. Consider adding one (MIT, Apache-2.0, etc.) to clarify usage and contributions.
