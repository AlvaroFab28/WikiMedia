# 🏆 Resumen Ejecutivo: Logros del Proyecto

Se ha implementado una **Arquitectura Empresarial de Alta Disponibilidad**.

### 1. Alta Disponibilidad (High Availability - HA) 🔄
* **Logro:** Se configuró redundancia en el punto de entrada. Si el Proxy 1 falla, el Proxy 2 asume la dirección IP virtual (VIP) `.10` de forma automática.
* **Impacto:** Se garantiza la continuidad del servicio ("Business Continuity"), evitando interrupciones.

### 2. Escalabilidad Horizontal 📈
* **Logro:** La arquitectura cuenta con dos servidores de aplicación (`.13` y `.14`).
* **Impacto:** Permite un crecimiento flexible. Ante un aumento en la demanda (ej. 10,000 usuarios concurrentes), es posible clonar un nuevo servidor de aplicación (ej. `.18`) e integrarlo al bloque `upstream` de Nginx en minutos, sin interrumpir el servicio.

### 3. Persistencia de Datos y Estado (Stateful) 💾
* **Logro:**
    * **Archivos:** Se centralizaron los archivos en un servidor NFS (`.15`), asegurando que cualquier archivo subido esté disponible globalmente.
    * **Sesiones:** Las sesiones de usuario se gestionan de forma centralizada con Redis (`.16`), permitiendo que un usuario mantenga su sesión activa sin necesidad de volver a autenticarse al ser redirigido entre servidores.
* **Impacto:** Se proporciona una experiencia de usuario fluida y consistente, independientemente del servidor que atienda la solicitud.

### 4. Rendimiento (Performance) 🚀
* **Logro:** Se implementó un sistema de caché en memoria utilizando Redis.
* **Impacto:** Se reduce la carga sobre la base de datos (`.17`), ya que las consultas frecuentes se sirven directamente desde la memoria RAM, mejorando significativamente la velocidad del sitio.

### 5. Seguridad en Capas (Defense in Depth) 🛡️
* **Logro:**
    * **Perímetro:** Se restringió el acceso público únicamente a los puertos 80/443.
    * **Encriptación:** Se implementó SSL/TLS con redirección forzada a HTTPS.
    * **Segmentación:** Los servidores de aplicaciones y la base de datos están aislados tras un firewall (UFW), permitiendo comunicación solo entre ellos.
    * **Gestión:** Se modificó el puerto de acceso SSH al 2222 y se deshabilitó el inicio de sesión como root.

### 6. Observabilidad 👁️
* **Logro:** Se integró una pila de monitoreo con Prometheus y Grafana.
* **Impacto:** Proporciona visibilidad completa sobre el estado de la infraestructura, permitiendo monitorear el uso de recursos (ej. CPU) y predecir posibles fallos de forma proactiva.

---

### 📝 Veredicto Final

La infraestructura resultante alcanza un nivel de madurez alto, comparable a un estándar profesional. Las áreas de mejora incluirían el uso de certificados SSL/TLS emitidos por una autoridad certificadora y la implementación de VLANs en un entorno físico.

**La arquitectura construida es:**
* **Resiliente:** Diseñada para tolerar fallos de componentes.
* **Segura:** Fortalecida mediante múltiples capas de defensa.
* **Rápida:** Optimizada a través de mecanismos de caché.
* **Profesional:** Completa con un dominio propio y un sistema de monitoreo.
