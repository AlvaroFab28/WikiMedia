# 🚀 FIX 6: La Guía Definitiva de Optimización (Hardware + Software)

> **Estado:** Crítico 🛑

> **Objetivo:** Transformar una carreta lenta en un Fórmula 1, funcionando perfecto **con o sin internet**.

> **Contexto:** Clúster de Alta Disponibilidad con **11 Nodos** (3 PCs Físicas + 8 VMs) sobre un solo router.

-----

## 1\. El Diagnóstico: ¿Por qué se arrastra el sistema? 🐢

Tenemos dos villanos principales atacando tu infraestructura al mismo tiempo. No es culpa del software, es culpa del entorno físico y la configuración de red.

### Villano A: La Saturación del Aire (WiFi vs. Cable) 📻

El WiFi es **Half-Duplex** (como un Walkie-Talkie). Solo uno habla a la vez.

  * **El Problema:** Tenés 11 máquinas (3 físicas + 8 virtuales) queriendo gritar al mismo tiempo. Se genera una colisión masiva de paquetes.
  * **El Efecto "Hairpinning" (Tormenta de Tráfico):** Una sola visita a tu página genera esto:
    1.  Usuario ➡️ Proxy
    2.  Proxy ➡️ App
    3.  App ➡️ DB
    4.  App ➡️ NFS (fotos)
    5.  App ➡️ Redis (sesión)
    <!-- end list -->
      * **Resultado:** Multiplicá eso por 11. El aire se satura, la latencia sube de 5ms a 500ms y todo se siente "pegajoso".

### Villano B: La Trampa del DNS (El Asesino Silencioso) 🕵️‍♂️

Este es el que descubrimos recién.

  * **El Problema:** Tus VMs tienen configurado `8.8.8.8` (Google) en el Netplan.
  * **El Síntoma:** Cuando estás en una red sin internet (o inestable), la VM intenta preguntar algo afuera, no llega, y **se queda esperando 30 segundos** (Timeout) antes de fallar.
  * **Resultado:** La página no carga lento por falta de potencia, carga lento porque está **esperando** una respuesta que nunca llega.

-----

## 2\. Solución de Hardware: La Autopista Gigabit 🔌

Acá no hay vuelta que darle. El aire es para los pájaros, los servidores van por cable.

### Paso A: Usar un Router (Bueno)

  * **Requisito:** Tiene que ser **Gigabit (1000 Mbps)**. Si es Fast Ethernet (100 Mbps), es un embudo.

> **¿Cómo saber si es Gigabit? 👀**
> Mirá la luz del puerto del router donde enchufás el cable:
>
>   * 🟢 **Verde:** Gigabit (Joya, esto buscamos).
>   * 🟠 **Naranja/Ámbar:** 10/100 Mbps (Es basura para un clúster, te va a frenar).

### Paso B: Cablear Todo (Adiós WiFi)

1.  Conectá las 3 PCs Físicas al Router/Switch Gigabit con cables **Cat5e** o **Cat6**.
2.  **CRÍTICO:** Desactivá el WiFi en las 3 PCs anfitrionas. Asegurate que el tráfico viaje sí o sí por el cobre.
3.  **Beneficio:** Pasás de un "Walkie-Talkie" a una "Autopista de doble mano" (Full-Duplex). La latencia baja a **menos de 1ms**.

-----

## 3\. Solución de Red: "Modo Avión" (Eliminar Timeouts) ✈️

Para que esto ande **OFFLINE** y vuele, tenemos que sacarle la adicción a Google. Vamos a configurar las VMs para que no busquen afuera.

### Acción en TODAS las VMs (Proxy, Apps, DB, Redis, NFS)

Tenés que editar el Netplan de cada una de las 8 máquinas virtuales.

**1. Editar:**

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

**2. Modificar (Comentar los DNS externos):**
Dejalo así. La clave es borrar o comentar (`#`) las líneas que apuntan a Google.

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 192.168.0.XX/24  # La IP que corresponda a esa VM
      routes:
        - to: default
          via: 192.168.0.1
      # nameservers:      <--- ¡COMENTAR ESTO!
      #   addresses:      <--- ¡COMENTAR ESTO!
      #     - 8.8.8.8     <--- ¡CHAU GOOGLE!
      #     - 1.1.1.1
```

**3. Aplicar:**

```bash
sudo netplan apply
```

> **¿Por qué esto acelera todo?**
> Porque ahora, si la VM necesita algo y no lo encuentra en la red local, falla **al instante** (fail-fast) en vez de quedarse 30 segundos pensando en la nada.

-----

## 4\. Solución de Software: Tuning Fino (El Nitro) 🏎️

Ahora que la carretera está lisa (Cable) y sin peajes (Sin DNS), le ponemos nitro al motor.

### A. PHP OpCache (Acelerar el Procesamiento)

Evita que el servidor tenga que leer y compilar los archivos PHP cada vez. Los deja listos en la RAM.

  * **Archivo:** `/etc/php/8.3/fpm/php.ini` (o la versión que tengas).
  * **Configuración:**
    ```ini
    opcache.enable=1
    opcache.memory_consumption=128
    opcache.interned_strings_buffer=8
    opcache.max_accelerated_files=10000
    opcache.validate_timestamps=0
    ```
    *(Acordate: Con esto en 0, si cambiás código, tenés que reiniciar php-fpm).*

### B. Optimización NFS (Acelerar las Imágenes)

Esto reduce la "charla" innecesaria entre la App y el servidor de archivos.

  * **Archivo:** `/etc/fstab` (Solo en las VMs **App Wiki 1** y **App Wiki 2**).
  * **La línea mágica:**
    ```bash
    192.168.0.15:/var/nfs/wikipics /var/www/html/wiki/images nfs defaults,noatime,nodiratime,actimeo=60 0 0
    ```
      * `noatime`: No pierde tiempo anotando "cuándo fue la última vez que leí este archivo".
      * `actimeo=60`: Se acuerda de los atributos del archivo por 60 segundos. Reduce el tráfico de red brutalmente.

### C. Redis (Memoria a Corto Plazo)

Ya lo tenés configurado, pero asegurate que en el `LocalSettings.php` las sesiones (`$wgSessionCacheType`) apunten a Redis. Esto evita que la Base de Datos (`.17`) trabaje al pedo.

-----

## 🏁 Resumen para el Éxito

1.  **Capa Física:** Usá un Router (Gigabit) + Cables Cat5e/6. **Chau WiFi.** 🚫📶
2.  **Capa Red:** Sacale los DNS (`8.8.8.8`) a todas las VMs para que no haya Timeouts. ✂️🌐
3.  **Capa App:** Activá OpCache y los parámetros de montaje NFS. ⚙️🔥

Con esto la Wiki va a volar, tenga internet o no tenga internet.