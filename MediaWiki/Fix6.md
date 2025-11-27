# 📉 Análisis de Cuellos de Botella y Latencia en Infraestructura Distribuida

**Contexto:** Despliegue de clúster de Alta Disponibilidad (HA) distribuido en 3 nodos físicos interconectados mediante red inalámbrica (WiFi).
**Problema Detectado:** Alta latencia en la carga de la aplicación y lentitud general del sistema.

## 1\. Diagnóstico: Limitaciones del Medio Físico (WiFi vs Ethernet)

La causa raíz de la lentitud no es el software, sino la naturaleza del protocolo inalámbrico en un entorno de servidores.

### A. Half-Duplex (WiFi) vs. Full-Duplex (Cable)

  * **WiFi (Half-Duplex):** Funciona como un "Walkie-Talkie". Solo un dispositivo puede transmitir a la vez en el canal de frecuencia. Si la **PC 1** envía datos, la **PC 2** y la **PC 3** deben esperar y "escuchar".
  * **Ethernet (Full-Duplex):** Funciona como una autopista de doble mano. Los servidores pueden enviar y recibir datos simultáneamente sin colisiones.

### B. El Efecto "Hairpinning" (Ida y Vuelta)

En tu arquitectura actual, un solo *request* de un usuario genera una tormenta de tráfico que satura el aire:

1.  **Petición:** Usuario ➡️ Router ➡️ Proxy (PC1).
2.  **Procesamiento:** Proxy (PC1) ➡️ Router ➡️ App (PC2).
3.  **Consulta de Datos:** App (PC2) ➡️ Router ➡️ DB/Redis (PC3).
4.  **Respuesta de Datos:** DB (PC3) ➡️ Router ➡️ App (PC2).
5.  **Entrega Final:** App (PC2) ➡️ Router ➡️ Proxy (PC1) ➡️ Usuario.

> **Impacto:** Cada flecha (➡️) representa una transmisión que ocupa el canal WiFi. Al sumarse la latencia de cada salto (aprox 5ms a 50ms en WiFi), una página que requiere 20 consultas a la base de datos puede tardar varios segundos en cargar.

-----

## 2\. Solución de Hardware: Migración a Ethernet 🔌

La solución definitiva es conectar los 3 nodos físicos mediante cable (Cat5e o superior) a un switch o router Gigabit.

**Mejoras inmediatas:**

  * **Switching:** El switch crea canales dedicados entre puertos. El tráfico entre la App y la DB viaja directo (PC2 \<-\> PC3) sin afectar al tráfico del Proxy.
  * **Latencia \< 1ms:** El tiempo de respuesta baja de \~20ms a \<1ms.
  * **Estabilidad NFS:** El protocolo NFS (Sistema de Archivos de Red) requiere una conexión estable. El cable elimina los micro-cortes que congelan la carga de imágenes.

-----

## 3\. Optimización de Software (Tuning) 🚀

Independientemente de la red, se deben aplicar estas configuraciones para reducir la cantidad de viajes necesarios entre servidores.

### A. Caché de Opcode (PHP OpCache)

Evita que el servidor tenga que leer y compilar los scripts PHP en cada visita. Mantiene el código pre-compilado en la memoria RAM.

  * **Archivo:** `/etc/php/8.3/fpm/php.ini`
  * **Configuración recomendada:**
    ```ini
    opcache.enable=1
    opcache.memory_consumption=128
    opcache.interned_strings_buffer=8
    opcache.max_accelerated_files=10000
    opcache.validate_timestamps=0
    ```
    *(Nota: Con `validate_timestamps=0`, si modificas código PHP, debes reiniciar el servicio php-fpm para ver los cambios).*

### B. Optimización del Montaje NFS

Reduce la "charla" constante entre el cliente (App) y el servidor NFS, evitando validaciones innecesarias de atributos de archivo.

  * **Archivo:** `/etc/fstab` (En los nodos App)
  * **Parámetros clave:**
    ```bash
    192.168.0.15:/var/nfs/wikipics /var/www/html/wiki/images nfs defaults,noatime,nodiratime,actimeo=60 0 0
    ```
      * `noatime`: No actualiza la fecha de último acceso al leer un archivo (ahorra escrituras).
      * `actimeo=60`: Cachea los atributos del archivo por 60 segundos (reduce drásticamente las peticiones a la red).

### C. Persistencia en Redis

Asegurar que las sesiones y el caché de objetos (`ObjectCache`) estén delegados a Redis para minimizar las consultas SQL a la base de datos principal (MariaDB).
