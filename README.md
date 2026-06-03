# Frigate (Docker Compose)

Repositorio mínimo para ejecutar Frigate NVR usando Docker Compose.

## Contenido

- `docker-compose.yml` — configuración del servicio Frigate.
- `config.yml` — ejemplo de configuración de cámaras y grabación.
- `storage/` — volumen para grabaciones y medios (crear localmente).
- `logs/` — volumen para logs de Frigate (crear localmente).

## Descripción

Este proyecto contiene una configuración lista para ejecutar Frigate en Docker. Usa `ghcr.io/blakeblackshear/frigate:stable` y está preparado para aceleración por hardware (VAAPI) y reenvío de streams RTSP.

## Requisitos

- Docker y Docker Compose (`docker compose`).
- Host Linux con acceso a `/dev/dri` para aceleración VAAPI (recomendado). En Windows, usar WSL2 o una VM Linux para soporte completo.
- Cámaras accesibles vía RTSP/ONVIF en la misma red.

## Archivos principales

- [docker-compose.yml](docker-compose.yml): define el contenedor `frigate`, volúmenes, puertos y variables.
- [config.yml](config.yml): configuración de `ffmpeg`, `go2rtc`, cámaras, detección y retención de grabaciones.

## Puertos expuestos

- `8971` — Interfaz web (UI) autenticada.
- `8554` — RTSP (re-stream desde go2rtc dentro del container).
- `8555` — WebRTC (TCP y UDP).
- `5000` — UI sin autenticación (está comentado en el `docker-compose.yml`).

## Configuración rápida

1. Edita `config.yml` y sustituye los placeholders por los datos de tus cámaras:

```yaml
go2rtc:
	streams:
		camera:
			- rtsp://<usuario_cam>:<contraseña>@<ip_camara>:554/stream_main
		camera_sub:
			- rtsp://<usuario_cam>:<contraseña>@<ip_camara>:554/stream_sub

cameras:
	camera:
		onvif:
			host: <ip_camara>
			port: <puerto_onvif>
			user: <usuario_cam>
			password: <contraseña>
```

2. Crea los directorios locales que monta Docker (si no existen):

Shell (Linux/macOS):

```bash
mkdir -p storage logs
```

PowerShell (Windows):

```powershell
New-Item -ItemType Directory -Path storage, logs -Force
```

3. Levanta el servicio:

```bash
docker compose up -d
```

4. Ver logs en tiempo real:

```bash
docker compose logs -f frigate
```

5. Actualizar imagen y reiniciar:

```bash
docker compose pull
docker compose up -d
```

## Notas importantes

- La configuración activa usa `ffmpeg.hwaccel_args: preset-vaapi` en `config.yml`. Para que VAAPI funcione el host necesita exponer `/dev/dri/renderD128` (ya mapeado en `docker-compose.yml`). Si no tienes VAAPI, Frigate funcionará con CPU pero consumirá más recursos.
- El `docker-compose.yml` monta `./config.yml` en `/config/config.yml`. Mantén los permisos y rutas correctos.
- El `tmpfs` configurado mejora rendimiento de caché temporal; `shm_size` se ajustó a `128mb`.
- Si vas a ejecutar en Windows, ten en cuenta que el mapeo de dispositivos y la opción `privileged: true` no siempre son compatibles con Docker Desktop sin WSL2; se recomienda ejecutar en un host Linux para producción.

## Solución de problemas (rápido)

- No hay stream RTSP: revisa credenciales, puerto y ruta de la cámara.
- ONVIF no conecta: verifica puerto ONVIF (suelen usar 80 o 8899) y credenciales.
- Hardware acceleration no funciona: revisa permisos y existencia de `/dev/dri` en el host.

## Créditos y referencias

- Proyecto Frigate: https://github.com/blakeblackshear/frigate