# 2\. Desbloquear Formatos de Archivo (PDF, Word, Zip) 🔓

**Síntoma:** MediaWiki dice *"Este tipo de archivo no está permitido"* al subir algo que no sea imagen.
**Causa:** Por seguridad, MediaWiki bloquea casi todo por defecto.

**Pasos en las VMs de Aplicación (`.13` y `.14`):**
*Debes hacerlo en ambas para mantener la sincronización.*

1.  Editar la configuración de la Wiki:

    ```bash
    sudo nano /var/www/html/wiki/LocalSettings.php
    ```

2.  Ir al final del archivo y pegar el siguiente bloque:

    ```php
    # --- CONFIGURACIÓN DE TIPOS DE ARCHIVO ---
    # 1. Habilitar extensiones adicionales
    $wgFileExtensions = array_merge(
        $wgFileExtensions,
        [
            'pdf', 'ppt', 'pptx', 'doc', 'docx', 'xls', 'xlsx',
            'zip', 'rar', '7z',
            'mp4', 'mp3', 'svg', 'txt', 'csv'
        ]
    );

    # 2. Desactivar validación estricta (útil para archivos de Office)
    $wgStrictFileExtensions = false;

    # 3. Mantener chequeo de seguridad básico
    $wgCheckFileExtensions = true;
    ```

-----
