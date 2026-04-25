# Vaultwarden

Servidor de gestión de contraseñas compatible con Bitwarden, escrito en Rust. Implementación ligera y eficiente que permite autoalojar tu propio gestor de contraseñas con todas las características de Bitwarden.

## Características

- 🔐 **Compatible con Bitwarden**: Funciona con todas las aplicaciones oficiales de Bitwarden
- 🚀 **Ligero y rápido**: Escrito en Rust, consume menos recursos que Bitwarden oficial
- 📱 **Multiplataforma**: Clientes para móvil, escritorio, navegador y CLI
- 🔄 **Sincronización en tiempo real**: WebSocket para actualizaciones instantáneas
- 🔒 **Cifrado de extremo a extremo**: Tus contraseñas están cifradas localmente
- 👥 **Organizaciones**: Comparte contraseñas de forma segura con equipos
- 🌐 **Autoalojado**: Control total sobre tus datos

## Archivos de este Repositorio

Este repositorio contiene:

- `compose.yaml` - Stack de Vaultwarden
- `.env.example` - Plantilla de variables de entorno
- `README.md` - Esta documentación
- `data/` - Persistencia local (se crea al arrancar)

## Requisitos Previos

- Docker Engine instalado
- **Dominio configurado**: Vaultwarden requiere HTTPS para funcionar correctamente
- **ADMIN_TOKEN generado**: Token seguro para acceder al panel de administración
- Red Docker externa `proxy` creada

⚠️ **IMPORTANTE - Seguridad**: Vaultwarden requiere HTTPS en producción. Los clientes de Bitwarden no funcionarán correctamente con HTTP.

## Generar ADMIN_TOKEN

**Antes de cualquier despliegue**, es **crítico** generar un token en formato Argon2 seguro:

```bash
# 1. Generar un token aleatorio
openssl rand -base64 48

# 2. Convertirlo a formato Argon2 seguro
docker run --rm -it vaultwarden/server:latest /vaultwarden hash
# Pega el token generado arriba cuando lo solicite
```

Usa el **hash Argon2** resultante (empieza con `$argon2id$`) como valor de `ADMIN_TOKEN`.

> ⚠️ **Importante**: 
> - No uses el token en texto plano, siempre usa el hash Argon2
> - **Usa comillas simples** en el archivo `.env` para evitar que `$` se interprete como variable
> - Ejemplo: `ADMIN_TOKEN='$argon2id$v=19$m=65540,t=3,p=4$...'`

---

## Despliegue con Docker Compose

### 1. Crear Directorio y Archivos

```bash
# Crear directorio
mkdir vaultwarden
cd vaultwarden
```

### 2. Preparar archivo .env

El archivo `compose.yaml` ya está configurado con bind mount local `./data`.

Crea el archivo `.env`:

```env
ADMIN_TOKEN='$argon2id$v=19$m=65540,t=3,p=4$...'  # Tu hash Argon2
```

### 3. Crear red proxy

```bash
docker network create proxy
```

### 4. Desplegar

```bash
# Crear red proxy si no existe
docker network create proxy

# Iniciar servicios
docker compose up -d

# Ver logs
docker compose logs -f vaultwarden
```

---

## Método Alternativo: Clonar desde Git

Si prefieres usar Git para mantener la configuración actualizada:

```bash
# Clonar repositorio
git clone https://git.ictiberia.com/groales/vaultwarden.git
cd vaultwarden

# Copiar y editar variables
cp .env.example .env
nano .env

# Crear red proxy si no existe
docker network create proxy

# Desplegar
docker compose up -d
```

---

## Configuración Inicial

## Configuración Inicial
### 1. Acceder al Panel de Administración

Visita `https://vaultwarden.tudominio.com/admin` e introduce tu `ADMIN_TOKEN`.

### 2. Configuración Recomendada

En el panel de administración (`General Settings`):

- `Domain URL`: `https://vaultwarden.tudominio.com`
- `Require Email Verification`: Activar si tienes SMTP
- `Show password hints`: Desactivar por seguridad
- `Allow new signups`: `false` (salvo que lo necesites)
- `Invitation Organization Name`: Tu organización

### 3. Configurar SMTP (Opcional pero Recomendado)

Para recuperación de contraseñas y verificación de email:

```env
SMTP_HOST=smtp.tudominio.com
SMTP_PORT=587
SMTP_SECURITY=starttls
SMTP_USERNAME=vaultwarden@tudominio.com
SMTP_PASSWORD=tu_password_smtp
SMTP_FROM=vaultwarden@tudominio.com
```

### 4. Crear tu Primera Cuenta

1. Ve a `https://vaultwarden.tudominio.com`
2. Haz clic en **Create Account**
3. Introduce tu email y una contraseña maestra **fuerte**
4. Verifica tu email (si configuraste SMTP)

⚠️ **La contraseña maestra NO se puede recuperar**. Guárdala de forma segura.

## Clientes de Bitwarden

Vaultwarden es compatible con todos los clientes oficiales de Bitwarden:

- **Navegador**: Extensiones para Chrome, Firefox, Edge, Safari
- **Escritorio**: Windows, macOS, Linux
- **Móvil**: iOS, Android
- **CLI**: `bw` (para scripts y automatización)

### Configurar Cliente

Al crear cuenta o iniciar sesión:

1. Haz clic en el ⚙️ en la pantalla de login
2. En **Server URL** introduce: `https://vaultwarden.tudominio.com`
3. Inicia sesión con tu email y contraseña maestra



- ✅ Redirección automática HTTP → HTTPS
- ✅ Certificados SSL con Let's Encrypt
- ✅ Soporte completo para WebSocket (`/notifications/hub`)
- ✅ Headers de seguridad

Requiere:
- Red Docker `proxy` existente
- Variables: `DOMAIN` (con protocolo) y `DOMAIN_HOST` (solo dominio)




1. **Proxy Hosts** → **Add Proxy Host**
2. **Details**:
   - Domain Names: `vaultwarden.tudominio.com`
   - Scheme: `http`
   - Forward Hostname / IP: `vaultwarden`
   - Forward Port: `80`
   - ✅ **Websockets Support**: **ACTIVAR**
   - ✅ Block Common Exploits
3. **SSL**:
   - ✅ Force SSL
   - SSL Certificate: Request a new SSL Certificate (Let's Encrypt)

## Backup y Restauración

### Backup Manual

```bash
# Backup simple del directorio local
tar czf vaultwarden-backup-$(date +%Y%m%d).tar.gz ./data/
```

### Restauración

```bash
# Restaurar backup
tar xzf vaultwarden-backup-YYYYMMDD.tar.gz -C ./
docker compose restart vaultwarden
```

### Backup Automático

Considera usar una solución de backup automático como:
- [docker-volume-backup](https://github.com/offen/docker-volume-backup)
- Cron job con el script de backup manual
- Duplicati configurado para el directorio de datos

⚠️ **CRÍTICO**: Un gestor de contraseñas requiere backups regulares y probados.

## Actualización


1. Ve a tu stack `vaultwarden`
2. Haz clic en **Pull and redeploy**

### Desde CLI

```bash
docker compose pull
docker compose up -d
```

Docker Compose recreará automáticamente el contenedor con la nueva imagen manteniendo tus datos intactos.

## Solución de Problemas

### El cliente no puede conectar

**Síntomas**: Error de conexión en la app móvil o extensión del navegador

**Soluciones**:
1. Verifica que usas HTTPS (los clientes requieren conexión segura)
2. Comprueba que el certificado SSL es válido
3. Revisa que la variable `DOMAIN` incluye el protocolo completo
4. Verifica acceso desde navegador: `https://vaultwarden.tudominio.com`

### WebSocket no funciona

**Síntomas**: Las contraseñas no se sincronizan en tiempo real

**Soluciones**:

```bash
# Verificar que existen los routers de WebSocket
docker compose logs vaultwarden | grep websocket
```

1. Edita el Proxy Host
2. Pestaña **Advanced**
3. ✅ Activa **Websockets Support**

**Para Standalone**:
- Verifica que el puerto 3012 está publicado y accesible

### Error de ADMIN_TOKEN

**Síntomas**: No puedes acceder a `/admin`

**Soluciones**:
```bash
# Verificar que la variable está configurada
docker compose exec vaultwarden env | grep ADMIN_TOKEN

# Regenerar token en formato Argon2
docker run --rm -it vaultwarden/server:latest /vaultwarden hash

```

### Problemas de rendimiento

**Síntomas**: Aplicación lenta

**Soluciones**:
1. Vaultwarden usa SQLite por defecto, que es eficiente para < 1000 usuarios
2. Para instalaciones grandes, considera migrar a PostgreSQL
3. Revisa los logs: `docker compose logs vaultwarden`

### Ver logs detallados

```bash
# Logs en tiempo real
docker compose logs -f vaultwarden

# Logs dentro del contenedor
docker compose exec vaultwarden cat /data/vaultwarden.log
```

### Reiniciar completamente

```bash
# Cuidado: esto eliminará TODOS tus datos
docker compose down -v
docker compose up -d
```

## Recursos Adicionales

- [Documentación Oficial de Vaultwarden](https://github.com/dani-garcia/vaultwarden/wiki)
- [Clientes de Bitwarden](https://bitwarden.com/download/)
- [Wiki de este repositorio](https://git.ictiberia.com/groales/vaultwarden/wiki)
- [Repositorio en Gitea](https://git.ictiberia.com/groales/vaultwarden)

## Seguridad

- ⚠️ **Nunca** compartas tu ADMIN_TOKEN
- ⚠️ Usa una contraseña maestra fuerte y única
- ⚠️ Habilita 2FA en tu cuenta de Vaultwarden
- ⚠️ Realiza backups regulares y pruébalos
- ⚠️ Mantén actualizado el contenedor
- ⚠️ Usa HTTPS en producción (obligatorio)

## Licencia

Este repositorio de configuración está bajo licencia MIT. Vaultwarden es software libre bajo licencia GPL-3.0.
