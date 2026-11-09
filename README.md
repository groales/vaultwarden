# Vaultwarden con Docker Compose

Vaultwarden es una implementación comunitaria compatible con la API de Bitwarden, escrita en Rust. Este proyecto la ejecuta con la imagen mantenida por el propio proyecto, `vaultwarden/server`, y conserva la base de datos y el resto de los datos en `./data`.

## Enlaces oficiales

- Proyecto y guía principal: <https://github.com/dani-garcia/vaultwarden>
- Wiki de configuración: <https://github.com/dani-garcia/vaultwarden/wiki>
- Uso con Docker Compose: <https://github.com/dani-garcia/vaultwarden/wiki/Using-Docker-Compose>
- Imagen Docker Hub: <https://hub.docker.com/r/vaultwarden/server>
- Guía de HTTPS: <https://github.com/dani-garcia/vaultwarden/wiki/Enabling-HTTPS>
- Ejemplos de proxy inverso: <https://github.com/dani-garcia/vaultwarden/wiki/Proxy-examples>

Vaultwarden no es el servidor oficial de Bitwarden ni está afiliado a Bitwarden, Inc.

## Requisitos y puertos

- Docker Engine o Docker Desktop con Docker Compose.
- El servicio escucha en el puerto `80` dentro del contenedor.
- El proceso usa UID:GID definidos por `PUID` y `PGID` (por defecto, `1000:1000`).
- El host publica `127.0.0.1:8000` (fijo en `compose.yaml`); abre <http://localhost:8000> en este equipo. Ese puerto debe estar libre.
- Para uso desde otros equipos o clientes, configura un dominio HTTPS y un proxy inverso. No expongas directamente el puerto HTTP a Internet: 


> El Web Vault requiere HTTPS para funcionar de forma segura.

## Despliegue clonando con Git

Después de crear un repositorio Git con estos archivos y asignarle un remoto, clónalo en el servidor:

```sh
git clone <URL_DEL_REPOSITORIO> vaultwarden
cd vaultwarden
cp .env.example .env
```

Edita `.env` con el dominio HTTPS, `PUID`, `PGID` y los valores apropiados para ese servidor. No compartas `.env` ni lo incluyas en Git. Si publicas el repositorio, hazlo solo después de configurar tú mismo el remoto; este proyecto no incluye un remoto.

## Despliegue manual sin Git

1. Copia `compose.yaml` a una carpeta del equipo de destino.
2. Crea allí `.env` copiando `.env.example` y ajusta los valores.
3. Desde esa carpeta, valida la configuración e inicia el servicio:

```sh
docker compose config --quiet
docker compose up -d --pull always
```

La primera puesta en marcha permite registrar usuarios (`SIGNUPS_ALLOWED=true`) para crear la cuenta inicial. Después, establece `SIGNUPS_ALLOWED=false` en `.env` y aplica el cambio con `docker compose up -d`.

## Variables de entorno

Compose carga `.env` automáticamente para interpolar las variables usadas en `compose.yaml`. También se pueden definir explícitamente con `docker run -e`, `docker run --env-file`, `environment:` o `env_file:` en Compose. No guardes contraseñas, tokens ni otros secretos en Git, en el archivo de Compose ni en una imagen.

| Variable | Valor inicial | Función |
| --- | --- | --- |
| `PUID` | `1000` | UID numérico con el que se ejecuta el proceso dentro del contenedor. |
| `PGID` | `1000` | GID numérico con el que se ejecuta el proceso dentro del contenedor. |
| `DOMAIN` | `http://localhost:8000` | URL canónica del servicio; para acceso externo debe ser la URL pública HTTPS. |
| `SIGNUPS_ALLOWED` | `true` | Permite registros iniciales; desactívalo tras crear las cuentas necesarias. |

## Persistencia y operación

- `./data:/data` conserva la base de datos SQLite, adjuntos y demás estado del servicio fuera del contenedor. La carpeta `data/` se excluye de Git.
- El contenedor usa UID:GID `${PUID}:${PGID}`; ajusta `PUID` y `PGID` en `.env` según corresponda y asegúrate de que ese usuario pueda escribir en `data/` en el sistema anfitrión antes de iniciar Vaultwarden.
- Estado: `docker compose ps`
- Registros: `docker compose logs -f vaultwarden`
- Aplicar cambios de configuración: `docker compose up -d`
- Actualizar la imagen: `docker compose up -d --pull always`
- Parar el servicio: `docker compose down`
- `docker compose down` conserva `data/`; no borres esa carpeta si quieres mantener la bóveda.

Haz copias de seguridad periódicas de `data/` y comprueba que se pueden restaurar. Para copias coherentes, detén Vaultwarden durante la copia o sigue la guía oficial de respaldo.

## Seguridad

- Este despliegue queda limitado al propio equipo mediante `127.0.0.1:8000`; configura un proxy inverso con TLS antes de habilitar acceso por red.
- Configura `DOMAIN` con la URL HTTPS real detrás del proxy, siguiendo sus ejemplos oficiales.
- Deshabilita `SIGNUPS_ALLOWED` después del alta inicial y mantén la imagen actualizada.
- Protege las copias de seguridad y restringe permisos de acceso a `data/`; esa carpeta contiene información sensible.
- No habilites el panel de administración ni configures tokens hasta que realmente los necesites; cualquier secreto debe mantenerse fuera del repositorio.
