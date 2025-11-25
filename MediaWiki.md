# 🚀 MediaWiki Universitario: Infraestructura High Availability

> **Status:** En construcción 🚧  
> **Objetivo:** Despliegue de MediaWiki escalable, resiliente y a prueba de balas.

---

## 1. La Configuración de Red (Netplan) 🌐

Para que esto ande **joya**, cada VM necesita su IP estática fija según el diagrama de arquitectura. Asumiendo que estás corriendo **Ubuntu Server 24.04**, el archivo de configuración suele encontrarse en `/etc/netplan/50-cloud-init.yaml` (o a veces `00-installer-config.yaml`).

> [!WARNING]
> **¡Ojo al piojo! 🧐**
> Asegurate que el router físico (el que da internet a las 3 PCs) tenga la puerta de enlace en `192.168.0.1` y la máscara `/24` (255.255.255.0).
>
> Si tu router real tiene otra IP (tipo `192.168.1.1`), vas a tener que cambiar las IPs del diseño para que coincidan con ese rango, o las VMs **no van a tener internet**.

### Plantilla Universal (Netplan)
Aquí tenés la plantilla para todas tus VMs. Solo cambiá lo que está en mayúsculas:

```yaml
network:
  version: 2
  ethernets:
    enp0s3:  # ⚠️ Chequeá el nombre de tu interfaz con 'ip a', puede ser enp0s3, eth0, etc.
      dhcp4: no
      addresses:
        - IP_DE_LA_VM/24  # Ej: 192.168.0.11/24 para Proxy 1
      routes:
        - to: default
          via: 192.168.0.1 # Tu Gateway físico (Router de internet)
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```

Para aplicar los cambios, mandale mecha con:

```bash
sudo netplan apply
```

### 📋 Tabla de IPs (El Mapa del Tesoro)

| Ubicación Física | Rol | Hostname Sugerido | IP (Netplan) | Notas |
| :--- | :--- | :--- | :--- | :--- |
| **PC 1** | Proxy Master | `ha1-proxy` | `192.168.0.11` | Nginx + Keepalived |
| **PC 1** | Proxy Backup | `ha2-proxy` | `192.168.0.12` | Nginx + Keepalived |
| **PC 1** | **VIP (Virtual)** | N/A | `192.168.0.10` | **No poner en Netplan**. La maneja Keepalived. |
| **PC 2** | App Wiki 1 | `app1-wiki` | `192.168.0.13` | Apache/Nginx + PHP |
| **PC 2** | App Wiki 2 | `app2-wiki` | `192.168.0.14` | Apache/Nginx + PHP |
| **PC 2** | NFS Server | `srv-nfs` | `192.168.0.15` | Almacenamiento compartido |
| **PC 3** | Redis | `srv-redis` | `192.168.0.16` | Caché de objetos/sesiones |
| **PC 3** | MariaDB | `srv-db` | `192.168.0.17` | Base de datos principal |

---

## 2. Roadmap Táctico y Roles de las VMs 🗺️

No se pongan a instalar todo de golpe porque se les va a armar un **quilombo** bárbaro. Vamos por fases, paso a paso, como construyendo un edificio.

### Fase 1: Los Cimientos (Base de Datos y Archivos) 🏗️
*El equipo de PC 2 y PC 3 arranca primero.*

#### **VM MariaDB (`192.168.0.17`)**
* **Rol:** El corazón de los datos.
* **Acción:**
    1.  Instalar MariaDB Server.
    2.  Configurar el `bind-address` en `0.0.0.0` o la IP local para permitir conexiones remotas.
    3.  Crear la DB `wikidb` y un usuario `wikiuser` con permisos desde la red `192.168.0.%`.

#### **VM NFS (`192.168.0.15`)**
* **Rol:** Almacenamiento compartido. Clave para MediaWiki.
* **Por qué:** Si subo una foto en la App1, la App2 tiene que verla sí o sí.
* **Acción:** Instalar servidor NFS y exportar una carpeta (ej. `/var/nfs/images`) para que las Apps la monten. Sin esto, su wiki va a tener imágenes rotas la mitad de las veces.

---

### Fase 2: El Cuerpo (Aplicaciones) ⚙️
*El equipo de PC 2 entra en acción.*

#### **VMs App Wiki 1 y 2 (`.13` y `.14`)**
* **Rol:** Los trabajadores. Procesan PHP y sirven el contenido.
* **Acción:**
    1.  Instalar pila **LEMP/LAMP** (Nginx/Apache + PHP-FPM).
    2.  Montar la carpeta NFS en la ruta de imágenes de MediaWiki (`/var/www/html/wiki/images`).
    3.  Instalar MediaWiki.
    4.  **Importante:** Al configurar, apunten a la IP de la MariaDB (`.17`), **no** a localhost.
* **Tip Pro:** Configuren primero una App, copien el `LocalSettings.php` a la segunda y listo el pollo.

---

### Fase 3: La Entrada (Proxy y Alta Disponibilidad) 🚪
*El equipo de PC 1 se luce.*

#### **VMs Proxy 1 y 2 (`.11` y `.12`)**
* **Rol:** Porteros y Balanceadores de carga.
* **Acción:**
    1.  **Nginx:** Configurar `upstream` apuntando a las IPs de App1 y App2 (`.13` y `.14`).
    2.  **Keepalived:** Configurar la VIP `192.168.0.10`. Uno como **MASTER** y otro como **BACKUP**.
* **Prueba de fuego:** Si apagás (o desenchufás) el Proxy 1, la VIP `.10` debe saltar automáticamente al Proxy 2 y la wiki seguir andando sin drama.

---

### Fase 4: El Nitro (Optimización) 🚀
*Volvemos a PC 3.*

#### **VM Redis (`192.168.0.16`)**
* **Rol:** Memoria a corto plazo para que el sitio vuele.
* **Acción:** Instalar Redis Server y configurar para escuchar en su IP.
* **Integración:** Editar el `LocalSettings.php` en las Apps para decirle a MediaWiki: *"Che, guardá las sesiones y el caché de objetos en la IP .16"*. Esto le saca una mochila de encima a la base de datos.

---

### Fase 5: El Blindaje (Seguridad y Monitoreo) 🛡️
*Esto lo dejan para el final, cuando todo ande.*

1.  **Hardening (UFW):** Bloquear todo el tráfico y solo permitir los puertos necesarios entre sí (Lab 5.1).
2.  **Monitoreo:** Levantar Prometheus/Grafana apuntando a todos los nodos para ver métricas en tiempo real.

## 🗺️ Roadmap de hoy: Los Cimientos (La Base de Datos)

En un edificio no arrancás por el techo, arrancás por los cimientos. En este proyecto, los cimientos son la **Base de Datos (MariaDB)** y el **Almacenamiento (NFS)**. Sin esto, cuando quieras instalar MediaWiki en las otras VMs, te va a tirar error porque no tiene dónde guardar la info.

Vamos a configurar la VM **"MariaDB"**.

> **Objetivo:** Tener un servidor de base de datos listo, con IP fija, seguro y aceptando conexiones externas (porque las Apps le van a pegar desde otra IP).

---

### Paso 1: Configuración de Red (Netplan) 🔌

Arrancá tu VM de Ubuntu Server (que será la de MariaDB). Logueate. Vamos a fijarle la IP `192.168.0.17`.

1.  **Identificar el archivo:** En Ubuntu 24.04, suele estar en `/etc/netplan/`. Tirá este comando para ver el nombre exacto:
    ```bash
    ls /etc/netplan/
    ```
    *(Seguramente veas `50-cloud-init.yaml` o `00-installer-config.yaml`).*

2.  **Editar:** Vamos a editarlo con nano.
    ```bash
    sudo nano /etc/netplan/50-cloud-init.yaml
    ```
    *(Cambiá el nombre del archivo si te salió otro en el paso anterior).*

3.  **El Código (Copiá y pegá con cuidado):**
    Borrá lo que hay (o comentalo con `#`) y dejalo así.

    > [!CAUTION]
    > **¡OJO con la indentación!** ⚠️
    > En YAML los espacios son sagrados. **No uses tabulaciones**, usá espacios. Si le pifias a un espacio, Netplan explota.

    ```yaml
    network:
      version: 2
      ethernets:
        enp0s3:              # ⚠️ Verificá con 'ip a' si tu interfaz se llama así
          dhcp4: no          # Apagamos DHCP para que la IP no cambie nunca
          addresses:
            - 192.168.0.17/24  # La IP del diagrama
          routes:
            - to: default
              via: 192.168.0.1 # ⚠️ Tu Gateway real (la IP de tu router de casa)
          nameservers:
            addresses:
              - 8.8.8.8      # DNS de Google para tener internet
              - 1.1.1.1
    ```

    *¿Qué hicimos? Le dijimos a la máquina: "Che, tu nombre es 192.168.0.17, no le pidas nombre a nadie (DHCP no), y si querés salir a internet, andá a la puerta 192.168.0.1".*

4.  **Aplicar cambios:**
    ```bash
    sudo netplan apply
    ```
    *(Si no te tira error, vamos bien. Si te tira error, revisá los espacios en el archivo).*

5.  **Verificar:**
    ```bash
    ip a
    ping -c 4 8.8.8.8
    ```
    *Si el ping responde, ¡tenemos red y salida a internet!*

---

### Paso 2: Instalación de MariaDB y Hardening 🛡️

Ahora instalamos el motor de la base de datos. Como vimos en el Laboratorio 4.1, hay que instalar y asegurar.

1.  **Instalar:**
    ```bash
    sudo apt update && sudo apt install mariadb-server -y
    ```

2.  **Hardening (Hacerla segura):**
    Ejecutá el script de seguridad.
    ```bash
    sudo mysql_secure_installation
    ```

    **Respondé así (basado en Lab 4.1):**
    * `Enter current password for root`: **Enter** (porque no hay).
    * `Switch to unix_socket authentication`: **No (n)**.
    * `Change the root password?`: **Yes (y)**. (Poné una contraseña segura y acordatela).
    * `Remove anonymous users?`: **Yes (y)**. (Borramos usuarios fantasmas).
    * `Disallow root login remotely?`: **Yes (y)**. (El root solo entra local, por seguridad).
    * `Remove test database?`: **Yes (y)**.
    * `Reload privilege tables now?`: **Yes (y)**.

---

### Paso 3: Configurar Acceso Remoto (Clave para tu Cluster) 🔓

Por defecto, MariaDB es "tímida" y solo escucha "hacia adentro" (`localhost` o `127.0.0.1`). Si las VMs de la Wiki intentan conectarse, la DB las va a ignorar. Tenemos que decirle que escuche a la red.

1.  **Editar configuración:**
    ```bash
    sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
    ```

2.  **Buscar y cambiar:**
    Buscá la línea que dice `bind-address = 127.0.0.1`. Cambiala por:
    ```ini
    bind-address = 0.0.0.0
    ```
    > **Explicación:** `127.0.0.1` es "solo yo". `0.0.0.0` es "escucho a cualquiera que llegue a mi interfaz de red".

3.  **Reiniciar el servicio:**
    Para que tome el cambio:
    ```bash
    sudo systemctl restart mariadb
    ```

---

### Paso 4: Crear la Base de Datos y el Usuario para la Wiki 🗄️

Ahora vamos a hablar en **SQL** para crear el "cubo" donde la Wiki guardará sus datos.

1.  **Entrar a la consola de MariaDB:**
    ```bash
    sudo mysql -u root -p
    ```
    *(Poné la contraseña que creaste en el paso 2).*

2.  **Comandos SQL:**
    Escribí esto línea por línea (y dale Enter al final de cada una).

    ```sql
    -- Creamos la base de datos
    CREATE DATABASE wikidb;

    -- Creamos el usuario 'wikiuser'
    -- IMPORTANTE: El '%' significa que este usuario puede conectarse desde CUALQUIER IP.
    -- Esto es necesario porque tus Wikis están en otras máquinas (.13 y .14).
    CREATE USER 'wikiuser'@'%' IDENTIFIED BY 'tu_password_seguro';

    -- Le damos permisos totales sobre esa base de datos
    GRANT ALL PRIVILEGES ON wikidb.* TO 'wikiuser'@'%';

    -- Aplicamos los permisos
    FLUSH PRIVILEGES;

    -- Salimos
    EXIT;
    ```
    *¿Qué hicimos? Creamos un espacio vacío (`wikidb`) y le dimos las llaves a un usuario (`wikiuser`) para que pueda entrar desde cualquier máquina de la red (`%`).*

---

### 🔎 Validación Final (No te saltes esto)

Para confirmar que esta VM ya está lista y pasar a la siguiente, hacé lo siguiente:

1.  Desde **TU PC FÍSICA** (Windows/Linux), abrí una terminal (CMD o PowerShell).
2.  Intentá hacerle ping a la VM:

    ```powershell
    ping 192.168.0.17
    ```

**Si responde, ¡golazo! ⚽**
Ya tenés el servidor de base de datos vivo en la red y listo para recibir conexiones.

> **¿Cómo la ves?** ¿Te funcionó el ping y la configuración de red? Si esto está listo, el siguiente paso lógico es levantar la **VM NFS (Archivos)** antes de meternos con las Wikis.

## Ahora, no nos dormimos en los laureles. Vamos por el segundo pilar de los cimientos: El Servidor NFS (Archivos Compartidos).

### 📂 ¿Por qué necesitamos esto? (La lógica detrás del comando)
Imaginate que tus dos Wikis (App 1 y App 2) son dos personas editando el mismo cuaderno.

- **Base de Datos:** Es el texto que escriben. (Ya lo resolvimos con MariaDB).
- **NFS:** Son las fotos y archivos que pegan en el cuaderno.

Si subís una foto a la App 1 y se guarda en su disco duro local, cuando el usuario entre por la App 2, va a ver un agujero roto ❌ porque la App 2 no tiene esa foto. Solución: Las dos apps van a guardar y leer las fotos en un disco compartido externo: Tu servidor NFS.

### Preparación de la VM "NFS"
Creá (o cloná) una VM nueva en VirtualBox. Esta va a ser tu Servidor NFS.
- **Rol:** Almacén de imágenes.
- **IP Objetivo:** `192.168.0.15` (Según tu diseño).

### Paso 1: Configuración de Red (Netplan) 🌐
Ya te la sabés de memoria, pero la repetimos para fijar el conocimiento. Entrá a tu nueva VM NFS y editá el Netplan.

**Editar archivo:**
```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

**El Código (Ajustado para la .15):**
```yaml
network:
  version: 2
  ethernets:
    enp0s3:              # Chequeá con 'ip a'
      dhcp4: no
      addresses:
        - 192.168.0.15/24  # <--- CAMBIO IMPORTANTE AQUÍ
      routes:
        - to: default
          via: 192.168.0.1 # Tu Gateway real
      nameservers:
        addresses:
          - 8.8.8.8
```

**Aplicar:**
```bash
sudo netplan apply
```
Verificá con un ping a Google o a tu PC física.

### Paso 2: Instalar el Servidor NFS 🛠️
Ahora transformamos este Ubuntu común en un servidor de archivos.

**Instalar el paquete del kernel:**
```bash
sudo apt update && sudo apt install nfs-kernel-server -y
```

### Paso 3: Crear la Carpeta Compartida 📁
Necesitamos crear la carpeta "física" donde van a vivir las imágenes de la Wiki.

**Crear directorio:** Vamos a crear una carpeta llamada `wikipics` dentro de `/var/nfs`.
```bash
sudo mkdir -p /var/nfs/wikipics
```
*(La bandera `-p` crea las carpetas padre si no existen).*

**Asignar dueños (Permisos):** Para no complicarnos la vida con usuarios específicos ahora (que suele dar dolores de cabeza al principio), vamos a decirle a Linux que esta carpeta es de "nadie" (`nobody`), lo que permite que el servicio NFS escriba sin bloquearse por permisos de usuario local.
```bash
sudo chown nobody:nogroup /var/nfs/wikipics
```

### Paso 4: Configurar las Exportaciones (`/etc/exports`) 📝
Acá está la magia. Tenemos que decirle al servidor: *"Che, compartí esta carpeta, pero OJO, solo dejá entrar a la App 1 (`.13`) y a la App 2 (`.14`). A nadie más"*. Esto es seguridad básica.

**Editar el archivo de exportaciones:**
```bash
sudo nano /etc/exports
```

**Agregar la configuración al final:** Copiá y pegá estas líneas al final del archivo. Fijate que ponemos las IPs de las futuras Apps.
```plaintext
/var/nfs/wikipics 192.168.0.13(rw,sync,no_subtree_check)
/var/nfs/wikipics 192.168.0.14(rw,sync,no_subtree_check)
```

**Explicación técnica (para que aprendas):**
- `rw`: Read/Write. Las apps pueden leer y escribir (subir fotos).
- `sync`: Confirma que el archivo se escribió en el disco físico antes de decirle "OK" a la App. Es más seguro para no perder datos si se corta la luz.
- `no_subtree_check`: Optimiza el rendimiento evitando que el sistema verifique todo el árbol de carpetas cada vez.

**Guardar y Salir:** (Ctrl+O, Enter, Ctrl+X).

### Paso 5: Activar y Validar ✅
**Exportar los cambios:** Este comando lee el archivo que acabamos de editar y pone las reglas en marcha sin reiniciar todo el servidor.
```bash
sudo exportfs -a
```

**Reiniciar el servicio (por las dudas):**
```bash
sudo systemctl restart nfs-kernel-server
```

**Verificar qué estamos compartiendo:** Si tirás este comando, te tiene que listar las reglas que pusimos:
```bash
sudo exportfs -v
```
Deberías ver `/var/nfs/wikipics` con permisos para la `.13` y la `.14`.

### ⏸️ Pausa Táctica: Estado de Situación
¡Listo el pollo con los cimientos! 🏗️ Tenemos:
- ✅ Base de Datos (`.17`) esperando conexiones.
- ✅ Archivos Compartidos (`.15`) esperando que alguien monte la carpeta.

Ahora sí viene lo divertido: **Fase 2 - El Cuerpo**. Tenemos que levantar las Apps de MediaWiki. Esto va a ser más largo porque hay que instalar Webserver + PHP + Configurar el montaje del NFS.

## Fase 2: El Cuerpo (Servidor de Aplicación)

Ahora entramos en la **Fase 2: El Cuerpo**. Vamos a levantar el **Servidor de Aplicación 1 (App 1)**.

Acá es donde va a vivir "la inteligencia" de tu Wiki. Este servidor va a agarrar el código PHP de MediaWiki, lo va a procesar, va a pedirle los textos a la Base de Datos (`.17`) y las fotos al NFS (`.15`), y se lo va a entregar todo armadito al usuario.

### 🏗️ Preparación de la VM "App Wiki 1"
Creá una VM nueva.
- **Rol:** Servidor Web + PHP (MediaWiki).
- **IP Objetivo:** `192.168.0.13`.

### Paso 1: Configuración de Red (Netplan)
Vamos de nuevo con la rutina, pero ahora para la `.13`.

**Editar:**
```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

**El Código:**
```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 192.168.0.13/24  # <--- IP DE APP 1
      routes:
        - to: default
          via: 192.168.0.1
      nameservers:
        addresses:
          - 8.8.8.8
```

**Aplicar:**
```bash
sudo netplan apply
```

**Validar:** Tirá un `ping 192.168.0.17` (a tu base de datos). Si responde, ¡ya se ven entre ellas!

### Paso 2: Instalar el Stack Web (Nginx + PHP) 🛠️
MediaWiki está hecho en PHP, así que necesitamos un servidor web (Nginx) y el procesador de PHP (PHP-FPM). También necesitamos varias librerías para que MediaWiki pueda procesar textos, imágenes y conectarse a la base de datos.

**Actualizar repositorios:**
```bash
sudo apt update
```

**Instalar todo el paquete de una:** Copiá y pegá este choricera de comando. Incluye Nginx, PHP y todas las extensiones que MediaWiki pide a gritos.
```bash
sudo apt install nginx php-fpm php-mysql php-xml php-mbstring php-intl php-gd php-curl texlive -y
```

**Explicación:**
- `php-fpm`: El motor que procesa PHP rápido (FastCGI Process Manager).
- `php-mysql`: Para hablar con tu MariaDB `.17`.
- `php-intl`, `php-xml`, `php-mbstring`: Para manejar idiomas, caracteres raros y formatos de texto de la Wiki.

### Paso 3: Descargar MediaWiki 📦
Vamos a bajar el código fuente de la Wiki y ponerlo en su lugar.

**Ir a la carpeta temporal:**
```bash
cd /tmp
```

**Descargar (Versión estable actual 1.42):**
```bash
wget https://releases.wikimedia.org/mediawiki/1.42/mediawiki-1.42.1.tar.gz
```

**Descomprimir:**
```bash
tar -xvzf mediawiki-1.42.1.tar.gz
```

**Mover a la carpeta web:** Vamos a moverla a `/var/www/html/wiki` para que quede ordenado.
```bash
sudo mv mediawiki-1.42.1 /var/www/html/wiki
```

### Paso 4: Conectar el Almacenamiento Compartido (Cliente NFS) 🔌
¡Atención acá! Este es el paso crítico de tu diseño. Vamos a decirle a esta VM que la carpeta de imágenes de la Wiki NO es local, sino que está en el servidor NFS (`.15`).

**Instalar el cliente NFS:** Sin esto, la VM no sabe hablar el protocolo de archivos compartidos.
```bash
sudo apt install nfs-common -y
```

**Preparar la carpeta de imágenes:** MediaWiki guarda las subidas en la carpeta `images`. Vamos a borrar la que viene por defecto (está vacía) y crear el punto de montaje.
```bash
# Entramos a la carpeta de la wiki
cd /var/www/html/wiki

# Borramos la carpeta images local (si existe) para evitar conflictos
sudo rm -rf images

# Creamos la carpeta vacía de nuevo (será nuestro "enchufe")
sudo mkdir images
```

**Montar manualmente (Prueba de fuego):** Vamos a conectar el cable.
```bash
# Sintaxis: sudo mount IP_DEL_NFS:CARPETA_REMOTA CARPETA_LOCAL
sudo mount 192.168.0.15:/var/nfs/wikipics /var/www/html/wiki/images
```
Si no tira error, es buena señal.

**Hacerlo permanente (fstab):** Si reiniciás la VM ahora, el montaje se pierde. Vamos a grabarlo en piedra en el archivo `/etc/fstab`.

**Editar:**
```bash
sudo nano /etc/fstab
```

**Agregar al final (todo en una línea):**
```plaintext
192.168.0.15:/var/nfs/wikipics /var/www/html/wiki/images nfs defaults 0 0
```
Guardar y salir.

### Paso 5: Configurar Nginx para la Wiki ⚙️
Nginx por defecto no sabe qué hacer con los archivos `.php`, te los descarga en vez de ejecutarlos. Hay que enseñarle.

**Crear archivo de configuración del sitio:**
```bash
sudo nano /etc/nginx/sites-available/wiki
```

**El Código (Plantilla optimizada para MediaWiki):** Copiá esto. Fijate que `server_name` lo dejamos genérico (`_`) o poné la IP, porque luego el Proxy se encargará de los nombres.
```nginx
server {
    listen 80;
    listen [::]:80;

    root /var/www/html/wiki;
    index index.php index.html index.htm;

    server_name _; # Acepta cualquier nombre que le mande el proxy

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    # Configuración para procesar PHP
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock; # OJO: Verificá tu versión de PHP
    }

    # Bloquear acceso a archivos sensibles de la Wiki
    location ^~ /maintenance/ {
        return 403;
    }

    # Caché de imágenes estáticas (Optimización básica T4)
    location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
        expires max;
        log_not_found off;
    }
}
```
> **Nota:** Si instalaste ubuntu 24.04, es probable que sea `php8.3-fpm.sock`. Si usaste otra versión, verificá la carpeta `/var/run/php/` para ver el nombre exacto del socket.

**Activar el sitio:** Hacemos un enlace simbólico (como un acceso directo) de `sites-available` a `sites-enabled`.
```bash
sudo ln -s /etc/nginx/sites-available/wiki /etc/nginx/sites-enabled/
```

**Desactivar el sitio por defecto:** Para que no moleste.
```bash
sudo rm /etc/nginx/sites-enabled/default
```

**Probar configuración y reiniciar:**
```bash
sudo nginx -t
```
*(Debe decir "syntax is ok").*
```bash
sudo systemctl restart nginx
```

### 🏁 Verificación Final de la App 1
Ya tenemos todo listo. Para probar si funciona:
1. Abrí el navegador de tu PC FÍSICA.
2. Escribí la IP de esta VM: `http://192.168.0.13`

Deberías ver la pantalla de *"MediaWiki 1.42.1 LocalSettings.php not found"* o *"Please set up the wiki first"*. ¡Eso es éxito puro! Significa que Nginx y PHP están andando.

> [!WARNING]
> **¡No la configures todavía desde el navegador!** 🛑
> ¿Por qué? Porque si la configurás ahora, vas a generar el archivo `LocalSettings.php` solo en esta VM.

## Opción A: La Clonación (Golden Image)
Esto es lo que hacen los pros: arman una imagen base "dorada" (Golden Image) y después despliegan copias.

### Paso 1: Clonación en VirtualBox 🐑🐑
Necesitamos crear el gemelo **App 2 (`.14`)** a partir de la **App 1 (`.13`)**.

**Apagar la VM:** Primero apagá la `App Wiki 1` limpiamente.
```bash
sudo poweroff
```

**Clonar:**
1. En la lista de VirtualBox, click derecho sobre `App Wiki 1` -> **Clonar**.
2. **Nombre:** Ponéle `App Wiki 2`.
3. **MAC Address Policy (¡CRÍTICO!):** ⚠️ Seleccioná **"Generate new MAC addresses for all network adapters"**.
   > ¿Por qué? Si dejás la misma MAC, tu router se va a volver loco porque va a ver dos máquinas distintas con la misma "huella digital" física y les va a dar la misma IP. Caos total.
4. **Tipo de Clonación:** Elegí **"Clonación Completa"** (Full Clone). Tarda un cachito más pero es más seguro porque independiza el disco duro.

### Paso 2: Ajuste de Identidad de la App 2 🆔
Ahora tenés dos VMs idénticas. Si las prendés juntas, van a pelear por la IP `.13`. Vamos a operar a la App 2.

1.  **Prender SOLO la App 2:** Iniciá la VM nueva.
2.  Logueate.
3.  **Cambiar el Hostname:** Para no confundirnos de terminal.
    ```bash
    sudo hostnamectl set-hostname app2-wiki
    ```
4.  **Cambiar la IP (Netplan):** Acá le asignamos su lugar en el mapa: la `.14`.
    ```bash
    sudo nano /etc/netplan/50-cloud-init.yaml
    ```
    Cambiá `192.168.0.13` por `192.168.0.14`. El resto dejalo igual.
5.  **Aplicar:**
    ```bash
    sudo netplan apply
    ```
    *(Si se te desconecta el SSH es normal, ahora tiene otra IP).*
6.  **Reiniciar:** Mandale un `sudo reboot` para que levante todo limpio con el nuevo nombre y la nueva IP.

### Paso 3: La Prueba de Fuego (Verificar el NFS) 🔥
Como clonaste la máquina, la configuración de `/etc/fstab` (el montaje automático del disco compartido) ya está ahí. Y como en la fase anterior ya configuramos el servidor NFS para aceptar a la `.14`, esto debería andar solo.

**En App 2 (`.14`), verificá el montaje:**
```bash
df -h
```
Buscá una línea al final que diga algo como: `192.168.0.15:/var/nfs/wikipics ... /var/www/html/wiki/images`

**Prueba de escritura cruzada:**
- **En App 2,** creá un archivo en la carpeta compartida:
  ```bash
  sudo touch /var/www/html/wiki/images/test_desde_app2.txt
  ```
- Ahora andá a la **App 1** (prendela si estaba apagada) o al servidor **NFS**, y fijate si el archivo apareció ahí. Si lo ves, ¡magia! ✨ Tienen el cerebro compartido.

### Paso 4: Instalación de MediaWiki (La Federación) 🏛️
Ahora tenemos dos servidores web listos, pero MediaWiki no está configurado. Vamos a configurarlo en uno y "copiarle la tarea" al otro.

1.  **Prender AMBAS Apps (`.13` y `.14`) y la Base de Datos (`.17`).**
2.  **Abrir navegador en tu PC:** Entrá a `http://192.168.0.13` (App 1).
3.  **Setup Wizard:** Hacé click en "set up the wiki".
4.  **Idioma:** Español (o el que quieras).
5.  **Comprobaciones:** Debería decir "Environment checked. You can install MediaWiki". (Si sale algo en rojo, avisame).
6.  **Conexión a Base de Datos (¡OJO ACÁ!):**
    - **Database host:** `192.168.0.17` (Tu VM DB).
    - **Database name:** `wikidb`
    - **Database table prefix:** (Dejalo vacío).
    - **Database username:** `wikiuser`
    - **Database password:** (La que pusiste en el `CREATE USER`).
7.  **Configuración de DB:** Dejá los valores por defecto (InnoDB, UTF-8).
8.  **Nombre:** "Wiki Universitaria" (o lo que quieras).
9.  **Usuario Admin:** Creá tu usuario administrador.
10. **Opciones:** Cuando te pregunte, habilitá "Enable file uploads". Verificá que la carpeta de imágenes sea correcta (debería detectarla sola). Importante: En la sección de caché, si te pregunta, seleccioná "No caching" por ahora (el Redis lo metemos después, paso a paso).
11. **Finalizar:** Dale a "Install". Si todo sale bien, te va a decir "Installation was successful!" 🎉 y te va a descargar automáticamente un archivo llamado `LocalSettings.php`.

### Paso 5: Desplegar la Configuración (Sincronizar los clones) 🔄
Este archivo `LocalSettings.php` es la llave maestra. Contiene las contraseñas de la DB y la config del sitio. Necesitamos que ambas Apps tengan exactamente el mismo archivo.

En tu PC física, tenés el archivo `LocalSettings.php` en "Descargas". Para subirlo a las VMs, es más fácil copiar y pegar el contenido.

1.  Abrí el `LocalSettings.php` en tu PC con el Bloc de Notas y copiá todo el texto.
2.  **En App 1 (`.13`):**
    ```bash
    sudo nano /var/www/html/wiki/LocalSettings.php
    ```
    Pegá el contenido. Guardá y salí.
3.  **En App 2 (`.14`):**
    ```bash
    sudo nano /var/www/html/wiki/LocalSettings.php
    ```
    Pegá el mismo contenido. Guardá y salí.

### 🏁 Gran Final de la Fase 2
Probá entrar a:
- `http://192.168.0.13` -> Deberías ver la Portada de la Wiki.
- `http://192.168.0.14` -> Deberías ver la MISMA Portada.

**Prueba de Fuego:** Logueate en la `.13`, creá una página que diga "Hola Mundo". Guardala. Andá a la `.14` y refrescá. ¿Aparece la página? Si aparece, ¡felicitaciones! 🥂 Acabás de crear un Cluster de Aplicaciones Stateful.

> ¿Te funcionó la clonación y la sincronización de la Wiki? Si es así, ya estamos listos para la **Fase 3: El Proxy y Balanceador (Keepalived + Nginx)**. ¡Esto se pone picante! 🌶️

## Fase 3: El Proxy y Balanceador (HA)

Si al probar notaste un "saltito" al loguearte es totalmente normal ahora mismo. Seguro en tu `LocalSettings.php` quedó grabada la IP de la `.13` como la dirección principal del sitio (`$wgServer`), entonces la wiki intenta ser "prolija" y te manda a donde cree que es su casa.

¡Justamente esta Fase 3 viene a solucionar eso! Vamos a poner un "Jefe de Tráfico" (el Proxy) adelante. Vos ya no vas a entrar ni a la `.13` ni a la `.14`, vas a entrar a la **VIP (IP Virtual)**, y el Proxy va a decidir a quién le pasa la pelota sin que te des cuenta.

Vamos a armar la **Alta Disponibilidad (HA)** con Nginx (como balanceador) y Keepalived (para la IP flotante).

### 🚧 Preparación de la VM "Proxy Master" (ha1-proxy)
Creá una VM nueva (o cloná una base limpia de Ubuntu, ¡no clones la de la Wiki que ya tiene basura!).
- **Rol:** Balanceador de Carga y Dueño de la VIP.
- **IP Objetivo:** `192.168.0.11`.

### Paso 1: Configuración de Red (Netplan) 🌐
Ya sos experto en esto. Configurá la IP fija para el Proxy 1.

**Editar:**
```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

**El Código:**
```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 192.168.0.11/24  # <--- IP PROXY 1
      routes:
        - to: default
          via: 192.168.0.1
      nameservers:
        addresses:
          - 8.8.8.8
```
**Aplicar:**
```bash
sudo netplan apply
```

### Paso 2: Instalar Nginx (El Balanceador) 🚦
Este Nginx no va a tener páginas web, solo va a redirigir tráfico.

**Instalar:**
```bash
sudo apt update && sudo apt install nginx -y
```

**Configurar el Balanceo:** Vamos a editar el sitio por defecto para convertirlo en un pasamanos inteligente.
```bash
sudo nano /etc/nginx/sites-available/default
```

**El Código (Borrá todo y pegá esto):** Este bloque define el grupo de servidores (`upstream`) y cómo se reparte el juego.
```nginx
# Grupo de servidores de backend (Tus wikis)
upstream backend_wiki {
    server 192.168.0.13:80; # App 1
    server 192.168.0.14:80; # App 2
}

server {
    listen 80;
    server_name _;

    location / {
        # Le pasamos la pelota al grupo 'backend_wiki'
        proxy_pass http://backend_wiki;

        # Cabeceras importantes para que la Wiki no se maree
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

**Validar y Reiniciar:**
```bash
sudo nginx -t
sudo systemctl restart nginx
```

### Paso 3: Clonar el Proxy (Crear el Respaldo) 🐑
Antes de meter Keepalived, vamos a crear el **Proxy 2 (ha2-proxy)** para tener la pareja lista.

1.  **Apagar Proxy 1:** `sudo poweroff`
2.  **Clonar en VirtualBox:**
    - **Nombre:** `Proxy Backup` (o `ha2-proxy`).
    - **MAC Address:** Generar nuevas (¡Muy importante!).
    - **Tipo:** Clonación completa.
3.  **Configurar Proxy 2:**
    - Prender solo Proxy 2.
    - Cambiar Hostname: `sudo hostnamectl set-hostname ha2-proxy`.
    - Cambiar IP en Netplan a `192.168.0.12`.
    - Aplicar Netplan y reiniciar.

### Paso 4: Instalar y Configurar Keepalived (La VIP) 🎩
Acá ocurre la magia. Vamos a configurar la IP `192.168.0.10`. Esta IP no es de nadie y es de los dos a la vez.

#### En Proxy 1 (MASTER - .11)
**Instalar:**
```bash
sudo apt install keepalived -y
```

**Configurar:**
```bash
sudo nano /etc/keepalived/keepalived.conf
```

**Pegar Configuración MASTER:** Asegurate que `interface` sea la correcta (ej. `enp0s3`).
```
vrrp_instance VI_1 {
    state MASTER           # Soy el Jefe
    interface enp0s3       # Mi tarjeta de red
    virtual_router_id 51   # ID del equipo (debe ser igual en el otro)
    priority 101           # 101 gana a 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass secreto123
    }
    virtual_ipaddress {
        192.168.0.10       # <--- LA VIP QUE USAREMOS PARA ENTRAR
    }
}
```
**Arrancar servicio:**
```bash
sudo systemctl restart keepalived
```

#### En Proxy 2 (BACKUP - .12)
**Instalar:**
```bash
sudo apt install keepalived -y
```
**Configurar:**
```bash
sudo nano /etc/keepalived/keepalived.conf
```
**Pegar Configuración BACKUP:** Fijate que cambia `state` y `priority`.
```
vrrp_instance VI_1 {
    state BACKUP           # Soy el Suplente
    interface enp0s3
    virtual_router_id 51   # Mismo ID
    priority 100           # Menor prioridad
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass secreto123
    }
    virtual_ipaddress {
        192.168.0.10       # La misma VIP
    }
}
```
**Arrancar servicio:**
```bash
sudo systemctl restart keepalived
```

### Paso 5: Ajuste Final en la Wiki (El toque maestro) 🔧
Para arreglar el tema de la redirección, tenemos que decirle a la Wiki: *"Che, tu dirección oficial es la VIP (`192.168.0.10`), no la IP de la VM"*.

**Editar `LocalSettings.php` (En App 1 y App 2):**
```bash
sudo nano /var/www/html/wiki/LocalSettings.php
```

**Buscar y Modificar `$wgServer`:** Buscá la línea que dice `$wgServer`. Seguramente dice `http://192.168.0.13`. Cambiala por:
```php
$wgServer = "http://192.168.0.10";
```
Esto fuerza a que todos los enlaces de la wiki apunten a la IP flotante.

### 🏁 Prueba de Fuego Final
1.  **Navegador:** Entrá a `http://192.168.0.10`. ¡Debería cargar tu Wiki! 🎉
2.  **Prueba de Failover (Lo divertido):**
    - Dejá un `ping 192.168.0.10 -t` corriendo en tu PC.
    - Apagá "a lo bruto" el Proxy 1 (`.11`).
    - El ping debería perder 1 o 2 paquetes y seguir respondiendo. Eso significa que el Proxy 2 agarró la VIP.
    - Recargá la página de la Wiki. ¡Debe seguir online!

> ¿Cómo te fue? ¿La VIP .10 te responde y lograste ver la wiki? Si esto anda, ya tenés una infraestructura de Alta Disponibilidad real. Lo que sigue es ponerle el "turbo" con Redis (Fase 4).

### 💡 ¿Por qué `$wgServer` debe ser la VIP (`.10`)?
Si ponés la IP propia en cada servidor (ej. `.13` en App 1 y `.14` en App 2), pasa esto:
1. Entrás por el Proxy (`.10`). Todo bien.
2. Hacés clic en "Editar" o "Iniciar sesión".
3. MediaWiki genera el link usando `$wgServer`. Si dice `.13`, tu navegador se va a redirigir a `http://192.168.0.13/...`.
4. ¡Boom! 💥 Te saliste del Clúster. Ahora estás hablando directo con la App 1.

**Problemas:**
- **Problema 1:** Si la App 1 se cae, te quedás sin servicio (aunque la App 2 esté viva), porque tu navegador está "pegado" a la IP `.13`.
- **Problema 2:** El balanceador de carga queda de adorno.
- **Problema 3:** Las cookies de sesión pueden fallar si el dominio cambia de `.10` a `.13`.

**Conclusión:** Es obligatorio poner la `.10` para que el usuario nunca se entere de qué servidor le responde y para que el sistema de Alta Disponibilidad funcione de verdad.

## 🚀 Fase 4: El Nitro (Caché con Redis)
Ahora sí, vamos a darle velocidad a esto. Como decían en tu justificación: *"Optimización de Plataformas (T4) para manejar el tráfico concurrente"*.

Cuando entrás a la Wiki, MariaDB tiene que trabajar un montón para armar la página. Con Redis, la primera vez se hace el trabajo duro, se guarda en la memoria RAM de Redis, y la segunda vez se entrega al instante. ¡Vuela! 🏎️💨

### 🏗️ Preparación de la VM "Redis"
Creá la VM en la PC 3 (según tu diseño).
- **Rol:** Servidor de Caché de Objetos y Sesiones.
- **IP Objetivo:** `192.168.0.16`.

### Paso 1: Configuración de Red (Netplan)
La rutina de siempre para la `.16`.

**Editar:**
```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

**El Código:**
```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 192.168.0.16/24  # <--- IP REDIS
      routes:
        - to: default
          via: 192.168.0.1
      nameservers:
        addresses:
          - 8.8.8.8
```
**Aplicar:**
```bash
sudo netplan apply
```

### Paso 2: Instalar y Configurar Redis ⚡
**Instalar:**
```bash
sudo apt update && sudo apt install redis-server -y
```

**Configurar Acceso Remoto:** Igual que con MariaDB, Redis por defecto es egoísta y solo escucha en local. Hay que abrirlo.
```bash
sudo nano /etc/redis/redis.conf
```

**Modificar `bind`:** Buscá la línea que dice `bind 127.0.0.1 ::1`. Cambiala por:
```ini
bind 0.0.0.0
```
*(También podés comentar la línea `bind` por completo, pero poner `0.0.0.0` es más explícito).*

**Protección (Modo Protegido):** Buscá la línea `protected-mode yes`. Cambiala a:
```ini
protected-mode no
```
> **OJO:** En producción real esto es peligroso si no tenés firewall. Como acá estamos en una red interna controlada y vamos a poner reglas UFW después, lo desactivamos para que las Apps puedan conectarse sin renegar con contraseñas por ahora.

**Reiniciar:**
```bash
sudo systemctl restart redis-server
```

**Validar:**
```bash
ss -an | grep 6379
```
Deberías ver que escucha en `*:6379` o `0.0.0.0:6379`.

### Paso 3: Conectar las Apps al Cerebro de Redis 🔌
Ahora tenemos que decirle a las Wikis (`.13` y `.14`) que dejen de guardar cosas en su disco y usen la memoria de Redis.

**¡Hacé esto en AMBAS Apps (App 1 y App 2)!**

1.  **Instalar el driver de PHP para Redis:** Sin esto, PHP no sabe hablar el idioma de Redis.
    ```bash
    sudo apt install php-redis -y
    ```
2.  **Reiniciar PHP-FPM:** Para que tome el módulo nuevo.
    ```bash
    # Verificá tu versión de PHP (ej. 8.3)
    sudo systemctl restart php8.3-fpm 
    # O si no estás seguro de la versión: sudo systemctl restart php*-fpm
    ```
3.  **Editar `LocalSettings.php`:**
    ```bash
    sudo nano /var/www/html/wiki/LocalSettings.php
    ```
4.  **Agregar la Configuración de Caché:** Andá al final del archivo y pegá este bloque mágico. Esto le dice a MediaWiki: *"Guardá las sesiones y el caché principal en la máquina `.16`"*.
    ```php
    # --- Configuración de Redis ---

    # Servidores Redis disponibles
    $wgObjectCaches['redis'] = [
        'class' => 'RedisBagOStuff',
        'servers' => [ '192.168.0.16:6379' ], # Tu IP de Redis
    ];

    # Usar Redis para el caché principal (acelera carga de páginas)
    $wgMainCacheType = 'redis';

    # Usar Redis para las sesiones de usuario
    # Esto es CRÍTICO: permite que te loguees en App 1 y sigas logueado en App 2
    $wgSessionCacheType = 'redis';

    # Opcional: Caché de parser (para que no procese el wikitexto cada vez)
    $wgParserCacheType = 'redis';
    $wgMessageCacheType = 'redis';
    ```
    Guardar y Salir.

### 🏁 Prueba de Fuego de la Fase 4
Vamos a ver si es verdad que esto anda.

1.  **En la VM Redis (`.16`):** Abrí la consola de monitoreo de Redis para ver pasar los datos en vivo.
    ```bash
    redis-cli monitor
    ```
    Debería decir "OK". Quedate mirando esa pantalla.
2.  **En tu Navegador:** Entrá a tu Wiki (`http://192.168.0.10`) y navegá un poco. Entrá a una página, editá algo, logueate/deslogueate.
3.  **Mirá la consola de Redis:** ¿Ves que empiezan a caer líneas de texto a lo loco? (comandos como `GET`, `SET`, `EXISTS`). ¡Eso son tus Apps (`.13` y `.14`) guardando y leyendo datos en Redis! 🕵️‍♂️

**Beneficio Extra:** Ahora probá esto: Logueate. Apagá la App 1. Refrescá la página (te atiende App 2). ¿Seguís logueado?
- **Antes:** Probablemente te pateaba porque la sesión estaba en el archivo de la App 1.
- **Ahora:** ¡Seguís adentro! Porque la sesión está guardada segura en Redis (`.16`).

> Si ves el monitor escupiendo datos, ¡Fase 4 completada! 🥂 Tu infraestructura ya es de alto rendimiento.
> ¿Viste las líneas en el monitor? Si está todo OK, nos queda la **Fase 5: El Blindaje (Seguridad y Monitoreo Final)** para cerrar el proyecto con moño.