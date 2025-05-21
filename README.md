# Angular + Apache + SSL en Contenedor

Este repositorio contiene un ejemplo mínimo de cómo desplegar una aplicación Angular utilizando Apache HTTP Server en un contenedor Docker, sirviendo a través de HTTPS con certificados SSL ya generados.

## 📦 Estructura del Proyecto

## Modifica el fichero apache-ssl.conf

Modifica la lineas siguientes en el fichero con los valores que correspondan a tu servidor y los certificados que correspondan:

   - ServerName YOURMACHINE.YOURDOMAIN.COM
   - SSLCertificateFile /etc/apache2/ssl/YOURCERTIFICATEFILE.PEM
   - SSLCertificateKeyFile /etc/apache2/ssl/YOURPRIVATEKEY.pem

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
## Añade el fichero `.htaccess` en la carpeta `web_juanlu`
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
