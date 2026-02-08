# Media-server

<p align="center">
  <img src="https://img.shields.io/badge/Docker-Compose-blue?logo=docker" />
  <img src="https://img.shields.io/badge/Media-Server-purple" />
</p>


Este repositorio contiene la configuración de **Docker Compose** para desplegar un servidor multimedia automatizado. El sistema utiliza **Real-Debrid** (a través de `decypharr`) para descargas y streaming, junto con la suite *Arr para la gestión de bibliotecas.

---

## Servicios y puertos

> Stack multimedia automatizado basado en Docker + Real-Debrid + *Arr Suite.

| Servicio | Puerto | Descripción |
| :--- | :---: | :--- |
| **Decypharr** | `8282` | Motor de descargas y montaje de unidad virtual Real-Debrid. |
| **Jellyfin** | `8096` | Servidor de streaming multimedia (tu Netflix personal). |
| **Jellyseerr** | `5055` | Interfaz web para solicitar películas y series. |
| **Prowlarr** | `9696` | Gestor centralizado de indexadores (Torrentio, etc.). |
| **Sonarr** | `8989` | Gestor automatizado de series de TV. |
| **Radarr** | `7878` | Gestor automatizado de películas. |
| **Bazarr** | `6767` | Gestor y descargador automático de subtítulos. |
| **Profilarr** | `6868` | Sincronización de perfiles de calidad y *Custom Formats*. |

---

## Requisitos Previos

1. Docker y Docker Compose instalados.
2. Cuenta activa **Real-Debrid Premium**.
3. Sistema operativo Linux (Ubuntu/Debian recomendado).
4. Usuario con `PUID=1000` y `PGID=1000` (ajustable en `docker-compose.yml`).

---

## Instalación y Despliegue

> Se recomienda clonar este repositorio en `/opt/freeflix`.

### Crear estructura de directorios

El stack utiliza `/opt/freeflix` como base. Ejecuta:

```bash
sudo mkdir -p /opt/freeflix/{decypharr,jellyfin/config,jellyfin/cache,radarr/config,sonarr/config,prowlarr/config,jellyseerr/config,profilarr,bazarr,library}
sudo mkdir -p /mnt
```

---

### Configurar permisos

Asegúrate de que tu usuario tenga permisos sobre estas carpetas:
 > Si tu UID o GID no es 1000, edita el comando para el ID correspondiente

```bash
sudo chown -R 1000:1000 /opt/freeflix
sudo chmod -R 775 /opt/freeflix
```

---

### Despliegue

Crea el archivo `docker-compose.yml` en tu carpeta de proyecto y levanta los servicios:

```bash
docker-compose up -d
```

---

## Configuración Inicial

> Sigue estos pasos **en orden** para evitar problemas de permisos o rutas.

### 🔹 1. Decypharr (El motor)

**Función:**
Conecta con Real-Debrid y monta los archivos en la carpeta `/mnt`.

**Configuración:**

- https://sirrobot01.github.io/decypharr/usage/


---

### 🔹 2. Prowlarr (Indexadores)

Acceso: `http://TU-IP:9696`

1. Añade cualquier indexador que desees, se recomiendan Torrentio y Comet.
2. Ve a **Settings → Apps**.
3. Añade **Sonarr** y **Radarr**.

> Prowlarr sincronizará automáticamente los indexadores con ambos servicios.

---

### 🔹 3. Sonarr y Radarr

Puertos:
- Sonarr: `8989`
- Radarr: `7878`

**Media Management:**
- Activa *Rename Episodes* / *Rename Movies*.

**Root Folders:**

- `/library/downloads/sonarr` para sonarr, donde sonar organizará los enlaces simbólicos
- `/library/downloads/radarr` para sonarr, donde sonar organizará los enlaces simbólicos

**Download Client:**

- Añade un cliente **qBittorrent** (según configuración de Decypharr).

---

### 🔹 4. Jellyfin

Acceso: `http://TU-IP:8096`

1. Añade bibliotecas de **Películas** y **Series**.
2. Usa las rutas de las librerias de sonarr y radarr para las bibliotecas de series y películas respectivamente.
3. Activa **Aceleración por Hardware** si tu equipo la soporta.

---

### 🔹 5. Jellyseerr

Acceso: `http://TU-IP:5055`

1. Conecta tu servidor **Jellyfin**.
2. Conecta **Radarr** y **Sonarr**.

---

## Comandos de Mantenimiento

> Útiles para administración diaria del stack.

### Ver logs de un contenedor

```bash
docker logs -f decypharr
```

### Reiniciar un servicio específico

```bash
docker-compose restart sonarr
```

###  Actualizar todos los contenedores

```bash
docker-compose pull
docker-compose up -d
```

---

## Notas Técnicas

- **Volumen `/mnt`:**
  - Decypharr requiere privilegios `SYS_ADMIN` para montar el sistema de archivos remoto de Real-Debrid.
  - El resto de contenedores acceden a `/mnt` en modo lectura según sea necesario.

- **Profilarr:**
  - Disponible en el puerto `6868`.
  - Permite importar y sincronizar *Custom Formats* avanzados para Sonarr y Radarr (idioma, resolución, audio, etc.).

---

---

## Recomendaciones

- Usa **Jellyseerr** como único punto de entrada para peticiones de series o peliculas.
- Mantén **Prowlarr** como gestor central de indexadores.
- Haz backups periódicos de `/opt/freeflix`.
- Si tu servidor no soporta la acelaración por hardware, no uses la interfaz web de Jellyfin directamente. Usa el cliente específico para el dispositivo en el que vayas a visualizar el contenido. Si tu servidor no tiene aceleración por hardware, esa transcodificación se hace por CPU, lo que dispara el consumo de recursos, provoca buffering, cortes y puede dejar el servidor al límite incluso con un solo stream. En cambio, los clientes nativos de Jellyfin (Android TV, móvil, PC, etc.) están pensados para reproducir el archivo tal cual, evitando la transcodificación casi por completo y ofreciendo una reproducción mucho más fluida incluso en servidores pequeño (como es mi caso, una Raspberry 5).

---

## Nota

Este proyecto **no aloja ni distribuye contenido**. Es únicamente una herramienta de automatización y gestión.

El usuario es **responsable del uso** que haga de los servicios configurados (Real-Debrid, indexadores, etc.) y del cumplimiento de las leyes de su país.

---

## TODO

- Usar Traefik
- Añadir mas documentación
- Perfiles para priorizar el español

---

