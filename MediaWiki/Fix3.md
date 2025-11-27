# 3\. Aumentar el Límite de Subida a 10MB (Error 413) 🏋️‍♂️

**Síntoma:** Error *"413 Request Entity Too Large"* (Nginx) o límite visible de 2MB en la web.
**Causa:** Hay "3 barreras" que limitan el tamaño: Nginx (Proxy y App), PHP y MediaWiki.

### Paso A: Configurar Nginx (La Portería)

**Atención:** Esto se debe hacer en **4 MÁQUINAS** (Proxy 1, Proxy 2, App 1, App 2).

1.  Editar el archivo principal:
    ```bash
    sudo nano /etc/nginx/nginx.conf
    ```
2.  Dentro del bloque `http { ... }`, agregar:
    ```nginx
    client_max_body_size 10M;
    ```
3.  Reiniciar Nginx:
    ```bash
    sudo systemctl restart nginx
    ```

### Paso B: Configurar PHP (El Motor)

**Dónde:** Solo en las **Apps (`.13` y `.14`)**.

1.  Editar el `php.ini` (la ruta puede variar según versión, ej: 8.1, 8.3):
    ```bash
    sudo nano /etc/php/*/fpm/php.ini
    ```
2.  Buscar y modificar estas dos variables:
    ```ini
    upload_max_filesize = 10M
    post_max_size = 12M
    ```
3.  Reiniciar PHP-FPM:
    ```bash
    sudo systemctl restart php*-fpm
    ```

### Paso C: Configurar MediaWiki (La Regla Final)

**Dónde:** Solo en las **Apps (`.13` y `.14`)**.

1.  Editar `LocalSettings.php`:
    ```bash
    sudo nano /var/www/html/wiki/LocalSettings.php
    ```
2.  Agregar al final:
    ```php
    # Límite de subida en Bytes (10MB)
    $wgMaxUploadSize = 10485760;
    ```

-----

En caso de que los cambios no se vean reflejados en la web de las Wiki metele un: `sudo reboot`