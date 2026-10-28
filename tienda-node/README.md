# Tienda DAW

Aplicación de ejemplo (Node.js + Express + MariaDB) para practicar el despliegue de una aplicación
detrás de Nginx en el módulo **Despliegue de Aplicaciones Web** (2º DAW).

## Qué hace

| Ruta | Respuesta |
|------|-----------|
| `/` | Página con los productos y un contador de visitas (lee y escribe en la base de datos) |
| `/api/productos` | Los productos en JSON |
| `/salud` | `{"estado":"ok"}` si llega a la base de datos; **503** si no |

## Despliegue (resumen)

Los pasos completos, explicados, están en la guía de la práctica. Aquí, solo el orden:

```bash
# 1. El código: de este repositorio a /opt/tienda-app
sudo mkdir -p /opt/tienda-app && sudo chown $USER: /opt/tienda-app
git clone https://github.com/TU_USUARIO/tienda-node.git /opt/tienda-app

# 2. La base de datos: crear la base «tienda», su usuario con permisos justos y las tablas
sudo mariadb <<'SQL'
CREATE DATABASE IF NOT EXISTS tienda;
CREATE USER IF NOT EXISTS 'tienda'@'localhost' IDENTIFIED BY 'cambia_esta_clave';
GRANT SELECT, INSERT, UPDATE ON tienda.* TO 'tienda'@'localhost';
SQL
sudo mariadb tienda < /opt/tienda-app/bd/esquema.sql

# 3. Las librerías y la configuración
cd /opt/tienda-app
npm install                  # descarga express, mariadb y dotenv en node_modules
cp .env.example .env         # y pon en DB_PASSWORD la misma clave del paso 2

# 4. La aplicación como servicio
sudo cp despliegue/tienda-app.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now tienda-app
```

5. Nginx delante: `despliegue/app-proxy.nginx` recibe las visitas por HTTPS en `app.test` y las
reenvía a `127.0.0.1:3000`.

## Actualizar

```bash
git pull && npm install && sudo systemctl restart tienda-app
```

Lo que **no** está en el repositorio: `.env` (contraseñas) y `node_modules/` (se instala con `npm install`).
