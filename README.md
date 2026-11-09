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
- Por defecto, el host publica `127.0.0.1:8081`; abre <http://localhost:8081> en este equipo.
- Para uso desde otros equipos o clientes, configura un dominio HTTPS y un proxy inverso. No expongas directamente el puerto HTTP a Internet: el Web Vault requiere HTTPS para funcionar de forma segura.

## Despliegue clonando con Git

Después de crear un repositorio Git con estos archivos y asignarle un remoto, clónalo en el servidor:

```sh
git clone <URL_DEL_REPOSITORIO> vaultwarden
cd vaultwarden
cp .env.example .env
```

Edita `.env` con el dominio HTTPS y los valores apropiados para ese servidor. No compartas `.env` ni lo incluyas en Git. Si publicas el repositorio, hazlo solo después de configurar tú mismo el remoto; este proyecto no incluye un remoto.

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
| `DOMAIN` | `http://localhost:8081` | URL canónica del servicio; para acceso externo debe ser la URL pública HTTPS. |
| `VAULTWARDEN_PORT` | `8081` | Puerto del host, enlazado solo a `127.0.0.1`. |
| `SIGNUPS_ALLOWED` | `true` | Permite registros iniciales; desactívalo tras crear las cuentas necesarias. |

## Persistencia y operación

- `./data:/data` conserva la base de datos SQLite, adjuntos y demás estado del servicio fuera del contenedor. La carpeta `data/` se excluye de Git.
- Estado: `docker compose ps`
- Registros: `docker compose logs -f vaultwarden`
- Aplicar cambios de configuración: `docker compose up -d`
- Actualizar la imagen: `docker compose up -d --pull always`
- Parar el servicio: `docker compose down`
- `docker compose down` conserva `data/`; no borres esa carpeta si quieres mantener la bóveda.

Haz copias de seguridad periódicas de `data/` y comprueba que se pueden restaurar. Para copias coherentes, detén Vaultwarden durante la copia o sigue la guía oficial de respaldo.

## Seguridad

- Este despliegue queda limitado al propio equipo mediante `127.0.0.1`; configura un proxy inverso con TLS antes de habilitar acceso por red.
- Configura `DOMAIN` con la URL HTTPS real detrás del proxy, siguiendo sus ejemplos oficiales.
- Deshabilita `SIGNUPS_ALLOWED` después del alta inicial y mantén la imagen actualizada.
- Protege las copias de seguridad y restringe permisos de acceso a `data/`; esa carpeta contiene información sensible.
- No habilites el panel de administración ni configures tokens hasta que realmente los necesites; cualquier secreto debe mantenerse fuera del repositorio.
