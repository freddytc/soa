# 🚀 Guía de Despliegue en Render

Esta guía te ayudará a desplegar el sistema SOA Ticketing completo en Render.com de forma **gratuita**.

## 📋 Pre-requisitos

1. **Cuenta en GitHub** - Para subir tu código
2. **Cuenta en Render** - Regístrate en https://render.com (gratis)
3. **Git configurado** en tu máquina

---

## 🔧 Paso 1: Preparar el Repositorio

### 1.1 Sube tu código a GitHub

```powershell
# Si aún no tienes un repositorio Git
cd 'D:\Tareas de programacion\SOA'
git init
git add .
git commit -m "feat: Sistema SOA Ticketing completo con Docker"

# Crea un repositorio en GitHub y conecta
git remote add origin https://github.com/TU_USUARIO/soa-ticketing.git
git branch -M main
git push -u origin main
```

### 1.2 Asegúrate de que todos los Dockerfiles existan

Verifica que estos archivos existan:
- `gateway/Dockerfile`
- `user-service/Dockerfile`
- `event-service/Dockerfile`
- `camunda-service/Dockerfile`
- `payment-service/Dockerfile`
- `notification-service/Dockerfile`
- `ticket-service/Dockerfile`
- `image-service/Dockerfile`
- `Frontend/Dockerfile`

---

## 🌐 Paso 2: Desplegar en Render

### Opción A: Blueprint (Automático - Recomendado)

1. **Ve a Render Dashboard**: https://dashboard.render.com

2. **Click en "New" → "Blueprint"**

3. **Conecta tu repositorio de GitHub**
   - Autoriza Render a acceder a tu GitHub
   - Selecciona el repositorio `soa-ticketing`

4. **Render detectará automáticamente el archivo `render.yaml`**
   - Click en **"Apply"**
   - Render creará todos los servicios automáticamente

5. **Espera 10-15 minutos** para que todos los servicios se desplieguen

6. **Tu aplicación estará disponible en:**
   ```
   https://soa-frontend.onrender.com
   ```

### Opción B: Manual (Servicio por Servicio)

Si prefieres control total, crea cada servicio manualmente:

#### 2.1 Crear Base de Datos PostgreSQL

1. Click en **"New" → "PostgreSQL"**
2. **Name**: `soa-database`
3. **Database**: `ticketing`
4. **User**: `soa_user`
5. **Region**: Oregon (US West)
6. **Plan**: Free
7. Click **"Create Database"**

Guarda la **Internal Database URL** que aparece (la necesitarás).

#### 2.2 Desplegar User Service

1. Click en **"New" → "Web Service"**
2. **Connect Repository**: Selecciona tu repo de GitHub
3. **Name**: `soa-user-service`
4. **Region**: Oregon
5. **Branch**: `main`
6. **Root Directory**: `user-service`
7. **Environment**: Docker
8. **Plan**: Free
9. **Environment Variables**:
   ```
   SPRING_DATASOURCE_URL = [Internal Database URL de PostgreSQL]
   SPRING_DATASOURCE_USERNAME = soa_user
   SPRING_DATASOURCE_PASSWORD = [Password de PostgreSQL]
   JWT_SECRET = mysecretkeymysecretkeymysecretkeymysecretkey
   GATEWAY_SECRET = soa-gateway-secret-key-2024
   SERVER_PORT = 8081
   ```
10. Click **"Create Web Service"**

#### 2.3 Repetir para cada servicio

Repite el paso 2.2 para:
- `soa-event-service` (puerto 8082)
- `soa-payment-service` (puerto 8084)
- `soa-notification-service` (puerto 8085)
- `soa-ticket-service` (puerto 8086)
- `soa-image-service` (puerto 8087)
- `soa-camunda-service` (puerto 8083) - **Importante**: Agrega las URLs de los otros servicios:
  ```
  SERVICES_USER_SERVICE_URL = https://soa-user-service.onrender.com
  SERVICES_EVENT_SERVICE_URL = https://soa-event-service.onrender.com
  SERVICES_PAYMENT_SERVICE_URL = https://soa-payment-service.onrender.com
  SERVICES_NOTIFICATION_SERVICE_URL = https://soa-notification-service.onrender.com
  SERVICES_TICKET_SERVICE_URL = https://soa-ticket-service.onrender.com
  ```
- `soa-gateway` (puerto 8080)

#### 2.4 Desplegar Frontend

1. Click en **"New" → "Static Site"**
2. **Name**: `soa-frontend`
3. **Build Command**: `cd Frontend && npm install && npm run build`
4. **Publish Directory**: `Frontend/dist`
5. **Environment Variables**:
   ```
   VITE_API_URL = https://soa-gateway.onrender.com
   ```
6. Click **"Create Static Site"**

---

## ⚙️ Paso 3: Configuración Post-Despliegue

### 3.1 Configurar el Frontend para usar el Gateway público

El frontend necesita conectarse al Gateway. Render usará la variable `VITE_API_URL`.

### 3.2 Verificar que todos los servicios estén "Healthy"

En el Dashboard de Render, verifica que todos los servicios tengan estado **"Live"** (verde).

### 3.3 Probar la Aplicación

1. Abre tu frontend: `https://soa-frontend.onrender.com`
2. Intenta registrar un usuario
3. Intenta crear un evento
4. Intenta comprar un ticket

---

## 🐛 Troubleshooting

### Los servicios se duermen tras 15 minutos

**Problema**: Render plan gratuito duerme los servicios inactivos.

**Solución**: Al primer acceso, espera ~30 segundos para que despierten. Luego funcionarán normal.

### Error de conexión entre servicios

**Problema**: Los servicios no se encuentran entre sí.

**Solución**: 
1. Verifica que las URLs en `camunda-service` sean correctas
2. Asegúrate de usar las URLs internas de Render: `https://NOMBRE-SERVICIO.onrender.com`

### Base de datos no se conecta

**Problema**: Error de conexión a PostgreSQL.

**Solución**:
1. Copia la **Internal Database URL** exacta de Render
2. Pégala en `SPRING_DATASOURCE_URL` de cada servicio
3. Cambia `jdbc:mysql://...` por `jdbc:postgresql://...` en tus application.properties si es necesario

### Frontend muestra error 404 en las rutas

**Problema**: React Router no funciona.

**Solución**: Asegúrate de que en la configuración del Static Site esté:
- **Rewrite Rules**: `/* → /index.html`

---

## 💰 Límites del Plan Gratuito

- ✅ **750 horas/mes por servicio** (suficiente para 1 servicio 24/7)
- ✅ **Ancho de banda**: 100 GB/mes
- ✅ **Base de datos**: 1 GB de almacenamiento
- ⚠️ **Servicios se duermen** tras 15 min de inactividad
- ⚠️ **Build time**: Máx 500 minutos/mes

**Para proyecto académico**: Más que suficiente.

---

## 📊 URLs de tus Servicios

Después del despliegue, tendrás:

| Servicio | URL |
|----------|-----|
| Frontend | https://soa-frontend.onrender.com |
| Gateway | https://soa-gateway.onrender.com |
| User Service | https://soa-user-service.onrender.com |
| Event Service | https://soa-event-service.onrender.com |
| Camunda | https://soa-camunda-service.onrender.com |
| Payment | https://soa-payment-service.onrender.com |
| Notification | https://soa-notification-service.onrender.com |
| Ticket | https://soa-ticket-service.onrender.com |
| Image | https://soa-image-service.onrender.com |

---

## 🎯 Resumen de Pasos

1. ✅ Sube código a GitHub
2. ✅ Crea cuenta en Render
3. ✅ Usa Blueprint (render.yaml) o crea servicios manualmente
4. ✅ Espera 10-15 minutos
5. ✅ Accede a `https://soa-frontend.onrender.com`

---

## 📚 Recursos

- [Documentación de Render](https://render.com/docs)
- [Render Blueprint Spec](https://render.com/docs/blueprint-spec)
- [Deploy con Docker](https://render.com/docs/docker)

---

✅ **Con Render, tu aplicación estará disponible públicamente de forma gratuita.**

**Última actualización**: Diciembre 2025
