# 🎨 Cambio de Logo en Cluster HA (MediaWiki)

> **Objetivo:** Cambiar el logo por defecto y asegurar que se vea en todas las Apps usando el almacenamiento compartido (NFS), solucionando los bloqueos de permisos.

## 1\. Requisitos de la Imagen

Antes de subir nada, asegurate que tu archivo cumpla con esto para no tener problemas de visualización:

  * **Formato:** **PNG** (con fondo transparente idealmente) o **SVG**.
  * **Dimensiones:** **135 x 135 píxeles** (estándar de la skin Vector).
  * **Peso:** Menos de **50 KB** (para que cargue rápido).
  * **Nombre:** Corto y simple, sin espacios ni caracteres raros (ej: `Escudo.png`).

-----

## 2\. Transferencia del Archivo (Desde Windows a la VM) 🚚

Usamos **SCP** desde la terminal de tu máquina física (PowerShell/CMD).

**Comando clave:**
Si el archivo está en una ruta profunda de Windows, usá comillas para evitar errores. Mandalo a la carpeta temporal `/tmp/` de la **App 1 (`.13`)**.

```powershell
scp "C:\Users\TuUsuario\Ruta\Completa\Escudo.png" usuario@192.168.0.13:/tmp/
```

*(Te va a pedir la contraseña de tu usuario de Linux).*

-----

## 3\. Mover al Almacenamiento Compartido (NFS) 📦

Una vez que el archivo está en la App 1, lo movemos a la carpeta `images`.
**¿Por qué ahí?** Porque esa carpeta está montada en el servidor NFS. Al ponerlo ahí, automáticamente aparece en la App 2.

**En la App 1 (`.13`):**

```bash
# Movemos el archivo a la carpeta de imágenes de la wiki
sudo mv /tmp/Escudo.png /var/www/html/wiki/images/
```

-----

## 4\. Solución de Permisos NFS (Si te sale un Error 'chown') 🔐

**El Problema:** Al intentar cambiar el dueño (`chown`) desde la App 1 Es probable que te salga: *"Operation not permitted"*.
**La Causa:** **Root Squashing**. El servidor NFS no confía en el usuario "root" de la App 1 y lo trata como un usuario anónimo, impidiéndole cambiar dueños.

Tenés **dos caminos** para solucionar esto (anotá los dos):

### Opción A: La Solución "Quirúrgica" (Recomendada para salir del paso) 💉

Ir directamente al dueño de casa (el Servidor NFS) y cambiar los permisos ahí.

1.  Logueate en la **VM NFS (`.15`)**.
2.  Ejecutá el cambio de dueño localmente:
    ```bash
    sudo chown www-data:www-data /var/nfs/wikipics/Escudo.png
    ```

### Opción B: La Solución "Zona de Confianza" (Para que confíe en las Apps) 🤝

Configurar el NFS para que confíe plenamente en el root de las Apps (útil en laboratorios, cuidado en producción real).

1.  En la **VM NFS (`.15`)**, editá las exportaciones:
    ```bash
    sudo nano /etc/exports
    ```
2.  Agregá `no_root_squash` a las líneas de configuración. Quedaría así:
    ```text
    /var/nfs/wikipics 192.168.0.13(rw,sync,no_subtree_check,no_root_squash)
    /var/nfs/wikipics 192.168.0.14(rw,sync,no_subtree_check,no_root_squash)
    ```
3.  Aplicá los cambios:
    ```bash
    sudo exportfs -a
    sudo systemctl restart nfs-kernel-server
    ```
    *Ahora sí podrías hacer `chown` desde la App 1 sin que te reboten.*

-----

## 5\. Configuración de MediaWiki (El Código Final) 📝

Finalmente, le decimos a la Wiki que use nuestro escudo. Esto se hace en **AMBAS Apps (`.13` y `.14`)**.

1.  Editar configuración:

    ```bash
    sudo nano /var/www/html/wiki/LocalSettings.php
    ```

2.  Buscar el bloque `$wgLogos` existente, **borrarlo** y reemplazarlo por esto:

    ```php
    ## CONFIGURACIÓN DE LOGO E ÍCONO
    $wgLogos = [
        # El logo principal (Barra lateral)
        '1x' => "$wgScriptPath/images/Escudo.png",

        # El ícono (Pestañas móviles / Notificaciones)
        'icon' => "$wgScriptPath/images/Escudo.png",
    ];

    # Compatibilidad con skins antiguas (Fallback)
    $wgLogo = "$wgScriptPath/images/Escudo.png";
    ```

3.  Guardar y Salir.

4.  **Refrescar navegador:** `Ctrl + F5` para ver el cambio.

-----