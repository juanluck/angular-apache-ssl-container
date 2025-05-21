# 🚀 Despliegue de Aplicación Angular con Apache + SSL en Docker

Este repositorio proporciona un ejemplo mínimo de cómo desplegar una aplicación Angular utilizando Apache HTTP Server dentro de un contenedor Docker. La aplicación se sirve de forma segura a través de HTTPS utilizando certificados SSL previamente generados.

---

## 📁 Estructura del Proyecto

```bash
├── apache-docker
│   ├── apache
│   │   ├── apache-ssl.conf         # Configuración de Apache con SSL
│   │   └── DockerfileApache        # Dockerfile para construir el contenedor
│   ├── certificados
│   │   ├── ClavePrivada.pem        # Clave privada SSL
│   │   └── Certificado.pem         # Certificado SSL
│   ├── docker-compose.yml          # Composición de servicios Docker
│   └── web_juanlu                  # Carpeta donde se colocan los archivos de la app Angular
├── LICENSE
├── mi-app-basica
│   ├── dist
│   │   └── mi-app-basica
│   │       ├── 3rdpartylicenses.txt
│   │       ├── browser
│   │       │   ├── favicon.ico
│   │       │   ├── index.html
│   │       │   ├── main-*.js
│   │       │   ├── polyfills-*.js
│   │       │   └── styles-*.css
│   │       └── prerendered-routes.json
└── README.md
```
---

## ⚙️ Pasos para el Despliegue

- **Requisitos previos:** Esta guía asume que ya tienes una aplicación Angular compilada (`ng build --configuration production`), cuyo contenido está incluido en `dist/`.

## 1️⃣ Copiar Archivos Compilados

Copia el contenido de `mi-app-basica/dist/mi-app-basica/browser/` en la carpeta `apache-docker/web_juanlu/`.

```cp -r mi-app-basica/dist/mi-app-basica/browser/* apache-docker/web_juanlu/```

## 2️⃣ Copiar los Certificados SSL

Coloca tu certificado y clave privada SSL en la carpeta `apache-docker/certificados/` con los nombres siguientes (o ajusta apache-ssl.conf en consecuencia):

```bash
cp /ruta/a/tu/Certificado.pem apache-docker/certificados/
cp /ruta/a/tu/ClavePrivada.pem apache-docker/certificados/
```

Asegúrate de que los permisos de estos archivos permiten que Docker los lea sin comprometer la seguridad.


## 3️⃣ Configurar Apache con SSL

Edita el archivo `apache-docker/apache/apache-ssl.conf` y reemplaza los siguientes valores con los correspondientes a tu dominio y certificados:

```xml
<VirtualHost *:443>
   ServerName YOURMACHINE.YOURDOMAIN.COM

   DocumentRoot /var/www/html

   SSLEngine on
   SSLCertificateFile /etc/apache2/ssl/YOURCERTIFICATEFILE.PEM
   SSLCertificateKeyFile /etc/apache2/ssl/YOURPRIVATEKEY.pem

   # Permitir cifrados cifrados MD5, RC4 y 3DES.
   SSLCipherSuite "HIGH:MEDIUM:!MD5:!RC4:!3DES"
   SSLHonorCipherOrder on
   
   #Deshabilitar protocolos inseguros y cifrados débiles que pueden ser explotados por ataques.
   #Quitar todos (-all) y permitir protocolo TLS v1, TLS v1.1 y TLS v1.2 
   SSLProtocol -all +TLSv1 +TLSv1.1 +TLSv1.2
   
   SSLProxyEngine On
   SSLProxyProtocol -all +TLSv1.2
   SSLProxyCheckPeerCN off
   SSLProxyCheckPeerName off

   <Directory /var/www/html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/mi-app-error.log
    CustomLog ${APACHE_LOG_DIR}/mi-app-access.log combined
</VirtualHost>
```

## 4️⃣ Añadir .htaccess para Soporte de Angular SPA

Crea un archivo `.htaccess` dentro de `apache-docker/web_juanlu/` con el siguiente contenido:
```xml
<IfModule mod_rewrite.c>
  RewriteEngine On
  RewriteBase /
  RewriteRule ^index\.html$ - [L]
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteRule . /index.html [L]
</IfModule>
```
Esto asegura que todas las rutas de la aplicación Angular se manejen correctamente desde `index.html`.

## 5️⃣ Levantar los Contenedores

Desde el directorio `apache-docker`, ejecuta:

```docker compose up -d```
