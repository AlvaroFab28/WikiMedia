# 1\. Solución al Error de Permisos de Carga 🚫

**Síntoma:** Error *"No se pudo abrir el archivo de bloqueo"* al intentar subir una imagen.
**Causa:** El servidor NFS tiene la carpeta con dueño "root" o "nobody", y el servidor web (www-data) no tiene permiso para escribir.

**Pasos en la VM NFS Server (`192.168.0.15`):**

1.  Asignar el dueño correcto (www-data) a la carpeta compartida:

    ```bash
    sudo chown -R www-data:www-data /var/nfs/wikipics
    ```

2.  Abrir los permisos de escritura (para evitar problemas en entornos de laboratorio):

    ```bash
    sudo chmod -R 777 /var/nfs/wikipics
    ```

3.  Reiniciar el servicio NFS para aplicar cambios:

    ```bash
    sudo systemctl restart nfs-kernel-server
    ```

-----

