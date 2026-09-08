# CRM PYMES

Backend inicial de un CRM para pequeñas y medianas empresas. La aplicación está construida con Node.js, Express, Sequelize y PostgreSQL alojado en Neon.

## Requisitos

- Node.js 18 o superior.
- npm.
- Acceso a la base de datos PostgreSQL en Neon.
- Git.

Puedes comprobar las versiones instaladas con:

```powershell
node --version
npm --version
git --version
```

## Estructura del proyecto

```text
CRM-PYMES/
├── backend/
│   ├── config/                 # Configuración de Sequelize
│   ├── migrations/             # Cambios de estructura de la base de datos
│   ├── models/                 # Modelos Usuario y Rol
│   ├── seeders/                # Datos iniciales
│   ├── src/                    # Respuestas y servicios de negocio
│   ├── test/                   # Pruebas automatizadas con Jest
│   ├── index.js                # Servidor Express y endpoints
│   ├── package.json            # Dependencias y scripts del backend
│   ├── .env.example            # Plantilla de variables de entorno
│   └── .sequelizerc            # Ubicación de la configuración de Sequelize CLI
├── .gitignore
└── README.md
```

## Instalación

Clona el repositorio y entra en la carpeta del backend:

```powershell
git clone https://github.com/DevDemianM/CRM-PYMES.git
cd CRM-PYMES
```

Si necesitas trabajar en la rama de desarrollo:

```powershell
git checkout DAMIAN
```

Instala las dependencias:

```powershell
cd backend
npm install
```

## Variables de entorno

Crea el archivo `backend/.env` a partir de `backend/.env.example`:

```powershell
Copy-Item .env.example .env
```

Edita `backend/.env` y completa los valores:

```env
DATABASE_URL=postgresql://usuario:contraseña@host/neondb?sslmode=require
JWT_SECRET=un_secreto_largo_y_aleatorio
```

- `DATABASE_URL` es la cadena de conexión de Neon.
- `JWT_SECRET` se utiliza para firmar los tokens de autenticación.
- El archivo `.env` contiene información privada y no debe subirse a Git.
- Cada integrante debe crear su propio `.env` local.

Puedes generar un secreto JWT en PowerShell con:

```powershell
[guid]::NewGuid().ToString('N') + [guid]::NewGuid().ToString('N')
```

## Base de datos

La aplicación utiliza PostgreSQL en Neon. No es necesario instalar MySQL.

Desde `backend`, consulta el estado de las migraciones:

```powershell
npx sequelize-cli db:migrate:status
```

Aplica las migraciones pendientes:

```powershell
npx sequelize-cli db:migrate
```

Las migraciones crean las tablas:

- `roles`
- `usuarios`

Para cargar los roles y el usuario administrador inicial, ejecuta:

```powershell
npx sequelize-cli db:seed:all
```

El seeder crea estos roles:

- administrador
- gerente
- vendedor

También crea el usuario inicial:

```text
Correo: admin@crm.local
Contraseña: Admin123
```

Ejecuta el seeder una sola vez por base de datos. Si ya fue ejecutado, no lo repitas sin revisar antes los datos existentes.

## Ejecutar el backend

Desde `C:\CRM-PYMES\backend`:

```powershell
npm run dev
```

El servidor estará disponible en:

```text
http://localhost:3000
```

Si la terminal está ubicada en la raíz del proyecto, también puedes iniciar el backend con:

```powershell
npm --prefix C:\CRM-PYMES\backend run dev
```

La terminal debe mostrar:

```text
Servidor escuchando en http://localhost:3000
```

Para detener el servidor, presiona `Ctrl + C`.

## Pruebas automatizadas

Desde `backend`, ejecuta:

```powershell
npm test
```

Las pruebas verifican que el servicio rechace correos inexistentes y genere un
token cuando la contraseña sea correcta. Deben terminar mostrando `PASS`.

## Endpoints disponibles

### Comprobar que el servidor está activo

```http
GET http://localhost:3000/ping
```

Respuesta:

```text
pong
```

## Endpoint de login

### Solicitud

```http
POST http://localhost:3000/auth/login
Content-Type: application/json
```

Cuerpo de ejemplo:

```json
{
  "correo": "admin@crm.local",
  "password": "Admin123"
}
```

En PowerShell:

```powershell
Invoke-RestMethod `
  -Uri http://localhost:3000/auth/login `
  -Method Post `
  -ContentType 'application/json' `
  -Body '{"correo":"admin@crm.local","password":"Admin123"}'
```

Respuesta exitosa:

```json
{
  "success": true,
  "data": {
    "token": "...",
    "usuario": {
      "id": 1,
      "nombre": "Administrador Inicial",
      "rol": "administrador"
    }
  }
}
```

La contraseña se verifica con `bcrypt` y el servidor devuelve un token JWT con una duración de 8 horas.

Las respuestas de error utilizan este formato:

```json
{
  "success": false,
  "error": {
    "code": "CREDENCIALES_INVALIDAS",
    "message": "Correo o contraseña incorrectos"
  }
}
```

Después de cinco intentos incorrectos, la cuenta se bloquea durante cinco minutos.

## Respuestas comunes del login

| Código | Significado |
|---|---|
| `200` | Login correcto |
| `400` | Faltan correo o contraseña |
| `401` | Correo o contraseña incorrectos |
| `423` | Cuenta temporalmente bloqueada |
| `500` | Error interno del servidor |

## Flujo de trabajo con Git

Cada integrante debe trabajar en su propia rama:

```powershell
cd C:\CRM-PYMES
git checkout -b nombre-de-la-rama
```

Antes de comenzar a trabajar:

```powershell
git pull origin DAMIAN
```

Después de realizar cambios:

```powershell
git status
git add .
git commit -m "Describe brevemente el cambio"
git push -u origin nombre-de-la-rama
```

No agregues manualmente `backend/.env`. Debe permanecer ignorado por Git.

Para actualizar tu rama con los cambios de `DAMIAN`:

```powershell
git fetch origin
git merge origin/DAMIAN
```

## Solución de problemas

### `Cannot find module 'C:\\CRM-PYMES\\index.js'`

El comando se ejecutó desde la raíz, pero `index.js` está dentro de `backend`. Ejecuta:

```powershell
cd C:\CRM-PYMES\backend
npm run dev
```

### `Falta JWT_SECRET en el archivo .env`

Agrega `JWT_SECRET` a `backend/.env` y reinicia el servidor.

### `ECONNREFUSED 127.0.0.1:3306`

La aplicación no utiliza MySQL. Revisa que `DATABASE_URL` apunte a Neon PostgreSQL y que el archivo `backend/.env` exista.

### El puerto 3000 está ocupado

Cierra la instancia anterior del servidor o cambia el puerto en `backend/index.js`.

## Convenciones

- No subir secretos, contraseñas ni archivos `.env`.
- Mantener las migraciones en `backend/migrations/`.
- Mantener los datos iniciales en `backend/seeders/`.
- Usar nombres de commits claros y breves.
- Probar el login después de modificar modelos, migraciones o autenticación.
