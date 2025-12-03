<div align="center">

# 🚀 INFRAESTRUCTURA & PLATAFORMAS TECNOLÓGICAS

![Status](https://img.shields.io/badge/Estado-Activo-success?style=for-the-badge&logo=git)
![Version](https://img.shields.io/badge/Versión-2025-blue?style=for-the-badge&logo=ubuntu)
![Power](https://img.shields.io/badge/Power-Unlimited-orange?style=for-the-badge)

<p align="center">
  <b>"Si no escala, no sirve. Bienvenido al futuro de la virtualización."</b>
</p>

---
</div>

## 👋 ¡Hola, Bienvenido!
En este repo vas a encontrar todo el material necesario...

---

### 📚 1. Material Teórico (Para los que leen)
* [📖 **Índice del material de temas avanzados**](Temas/Indice_de_Temas.md) _(Arrancá por acá si estás perdido)_

---

### 🛠️ 2. Proyecto MediaWiki
## 👥 El Dream Team
| Rol | Miembro | GitHub |
| :--- | :--- | :--- |
| **Proxy1, Proxy2, DNS, Monitoreo Grafana** | Villena Mamani Alvaro Fabian | @AlvaroFab28 |
| **App1, App2, NFS** | Castro Siñanis Jose Luis | @tu_usuario |
| **Redis, Base de Datos** | Villca Araca Jhesica | @cero0202 |


Acá está la todo lo que necesitás para montar la infraestructura.

| Tipo | Recurso | Descripción |
| :--- | :--- | :--- |
| 🗺️ | [**Diseño Topológico**](MediaWiki/Diseño_Topologico.png) | El mapa del tesoro. Miralo bien. |
| 📝 | [**Guía Paso a Paso**](MediaWiki/MediaWiki.md) | La biblia de la instalación. |
| ⚙️ | [**Utilidades Básicas**](MediaWiki/Utilidades/Utl_1.md) | Cambio de Hostname, Password, Usuario. |
| 🏁 | [**Resultados**](MediaWiki/Conclusiones.md) | Resumen de logros y futuras mejoras. |

#### ⚠️ **IMPORTANTE: MÁQUINAS VIRTUALES**
> Bajate las VMs actualizadas.
>
> 👉 [**CLICK ACÁ PARA IR AL DRIVE DE VMs (v.29/11/2025)**](https://drive.google.com/drive/folders/1c1CrpNQM8bl0YEJp2GHndt4T4YmYpKk7?usp=drive_link) 👈

---

### 🐛 3. Fixes & Mejoras 
¿Se rompió algo? ¿La página carga más lento que tortuga con reuma? Acá tenés la solución.

* [**Fix 1:** Error de carga de archivos](MediaWiki/Fixes/Fix1.md)
* [**Fix 2:** Nuevos formatos de archivos](MediaWiki/Fixes/Fix2.md)
* [**Fix 3:** Tamaño máximo de archivos](MediaWiki/Fixes/Fix3.md)
* [**Fix 4:** Cambiar el logotipo](MediaWiki/Fixes/Fix4.md) _(Ponete creativo)_
* [**Fix 5:** Personalizar Página principal](MediaWiki/Fixes/Fix5.md)
    * ↳ [**Fix 5.1:** Subir página personalizada por subcategorías (1er Semestre)](MediaWiki/Fixes/Fix5_1.md)
* [**Fix 6:** ⚡ Mejora de Red (Anti-Lag)](MediaWiki/Fixes/Fix6.md)

---

### 🧪 4. Zona de Testing (Probá que no explote)
Si no testeás, no es producción. Corta.

1.  [📊 **Test 1:** Monitoreo de BD](MediaWiki/Test1.md)
2.  [🌐 **Test 2:** IP del Server Wiki](MediaWiki/Tests/Test2.md)
3.  [⚖️ **Test 3:** Balanceo de carga Redis](MediaWiki/Test3.md) _(Una joyita)_
4.  [🛡️ **Test 4:** Failover en Proxy Principal](MediaWiki/Test4.md)
5.  [🔄 **Test 5:** Failover Wiki](MediaWiki/Test5.md)


# 🚀 Proyecto Final SIS313: Plataforma Wiki Universitaria en Alta Disponibilidad (HA)

*Asignatura:* SIS313: Infraestructura, Plataformas Tecnológicas y Redes  
*Semestre:* 2/2025  
*Docente:* Ing. Marcelo Quispe Ortega  

---

## 👥 Miembros del Equipo G-XX

> (Completar con los datos reales del grupo)

| Nombre Completo              | Rol en el Proyecto                                 | Contacto (GitHub/Email) |
|-----------------------------|----------------------------------------------------|--------------------------|
| Jose Luis Castro Siñanis    | Coordinador Infraestructura / Apps / Documentación |                          |
| Integrante 2 (PC 1)         | Proxy / Keepalived / VIP                           |                          |
| Integrante 3 (PC 2)         | Apps / NFS / MediaWiki                             |                          |
| Integrante 4 (PC 3)         | DB / Redis / Monitor + DNS                         |                          |

---

## 🎯 I. Objetivo del Proyecto

Diseñar e implementar una *plataforma MediaWiki Universitaria en Alta Disponibilidad*, basada en múltiples máquinas virtuales, con:

- *Balanceo de carga* entre dos servidores de aplicación.
- *IP flotante (VIP)* con failover automático (Keepalived + VRRP).
- *Almacenamiento compartido* para archivos (NFS).
- *Base de datos centralizada* (MariaDB).
- *Caché distribuido* (Redis) para sesiones y objetos.
- *DNS local + Monitorización centralizada* (Dnsmasq, Prometheus, Grafana).
- *Hardening* de red y servicios (SSH, TLS, UFW).

De manera que la wiki *siga operativa* frente a fallos individuales de nodos, mantenga coherencia en archivos e inicios de sesión y permita su administración y diagnóstico de forma centralizada.

---

## 💡 II. Justificación e Importancia

En un entorno universitario con múltiples docentes y estudiantes, una *plataforma wiki centralizada* es crítica para:

- Publicar material de clase, laboratorios, tareas y proyectos.
- Compartir recursos entre materias y carreras.
- Construir un repositorio colaborativo de conocimiento institucional.

Problemas que se atacan directamente:

- *Continuidad Operacional (T1):*  
  - Un solo servidor MediaWiki es un *Single Point of Failure*.  
  - Si cae la máquina de la wiki, toda la comunidad pierde acceso.  
  - El proyecto elimina este punto único mediante *dos App Servers, Proxies en HA y VIP flotante*.

- *Seguridad (T5):*  
  - Se endurece el acceso con SSH en puerto no estándar, deshabilitando root.  
  - Se cifra el tráfico web con HTTPS (TLS) y certificados (self-signed).  
  - Se limita el acceso a DB, Redis y NFS únicamente desde nodos autorizados (UFW).

- *Optimización de Plataformas (T4) y Automatización (T6):*  
  - Redis reduce la carga sobre la base de datos al cachear sesiones y objetos.  
  - NFS evita inconsistencias en archivos subidos.  
  - Prometheus + Grafana permiten monitoreo proactivo y diagnóstico rápido.

Esta plataforma *simula una infraestructura real de producción*, aplicable tanto a universidades como a pequeñas/medianas empresas que deseen exponer servicios web críticos con alta disponibilidad y seguridad razonable.

---

## 🛠 III. Tecnologías y Conceptos Implementados

### 3.1. Tecnologías Clave

- *Ubuntu Server 24.04:*  
  Sistema operativo base en todas las VMs.

- *MediaWiki 1.42.x:*  
  Plataforma de gestión de contenido colaborativo (wiki).

- *Nginx (Proxies y/o Apps):*  
  - Como *balanceador de carga* en los Proxies.  
  - Como servidor web en Apps (según stack elegido).

- *Apache/PHP-FPM (Apps Wiki 1 y 2):*  
  Procesamiento de contenido PHP y entrega de páginas de MediaWiki.

- *MariaDB (srv-db – 192.168.0.17):*  
  Base de datos relacional centralizada (wikidb, usuario wikiuser).

- *NFS (srv-nfs – 192.168.0.15):*  
  Servidor de archivos compartidos. Exporta /var/nfs/wikipics para que ambas Apps monten /var/www/html/wiki/images.

- *Redis (srv-redis – 192.168.0.16):*  
  Caché de objetos y *almacenamiento de sesiones* compartidas entre App1 y App2.

- *Keepalived + VRRP (ha1-proxy, ha2-proxy):*  
  Gestión de la *IP Virtual (VIP)* 192.168.0.10, con rol MASTER/BACKUP y failover automático.

- *Dnsmasq (srv-monitor – 192.168.0.20):*  
  Servidor DNS local para resolver el dominio wiki.usfx / wiki.usfx.bo → VIP 192.168.0.10.

- *Prometheus + Node Exporter:*  
  Recolección de métricas de CPU, RAM, disco y red de todas las VMs.

- *Grafana:*  
  Dashboard web para visualizar el estado de toda la infraestructura HA.

- *OpenSSL:*  
  Generación de certificados autofirmados para HTTPS.

- *UFW (Uncomplicated Firewall):*  
  Firewall en todas las VMs con política *deny incoming* y reglas específicas por rol.

- *Netplan:*  
  Configuración de IP estática y gateway en todas las VMs.

### 3.2. Conceptos de la Asignatura Puestos en Práctica (T1 – T6)

- *Continuidad Operacional (T1):* ✅  
  - Diseño HA que evita caída total de servicio ante falla de un Proxy o App.  

- *Alta Disponibilidad y Tolerancia a Fallos (T2):* ✅  
  - 2 Proxies con VIP flotante (Keepalived + VRRP).  
  - 2 App Servers sirviendo la misma wiki.  
  - Sesiones y archivos compartidos entre nodos.

- *Networking Avanzado / Proxy / Balanceo (T3/T4):* ✅  
  - Nginx como *reverse proxy + load balancer* hacia App1 y App2.  
  - Uso de VIP para abstracción de backend.  

- *Optimización y Performance (T4):* ✅  
  - Redis como caché de objetos y sesiones.  
  - Caché de contenidos estáticos en Nginx (imágenes, CSS, JS).  

- *Seguridad y Hardening (T5):* ✅  
  - SSH en puerto 2222, sin login de root.  
  - HTTPS obligatorio y redirección de HTTP→HTTPS.  
  - UFW con listas blancas de IPs por servicio.  

- *Automatización y Gestión (T6):* ✅  
  - Uso de Netplan, configuraciones reproducibles.  
  - Diseño por fases y roles de VMs con scripts y configuraciones reutilizables.  

---

## 🌐 IV. Diseño de la Infraestructura y Topología

### 4.1. Diseño Esquemático

> (Incluir diagrama de red con 3 PCs físicas, 8 VMs, VIP y flujo de tráfico)

*Tabla de VMs e IPs*

| Ubicación Física | Rol                         | Hostname sugerido | IP (Netplan)  | Notas                                       |
|------------------|-----------------------------|-------------------|--------------|---------------------------------------------|
| PC 1             | Proxy Master                | ha1-proxy         | 192.168.0.11 | Nginx + Keepalived (MASTER)                 |
| PC 1             | Proxy Backup                | ha2-proxy         | 192.168.0.12 | Nginx + Keepalived (BACKUP)                 |
| PC 1             | VIP (Virtual)               | wiki.usfx / VIP   | 192.168.0.10 | No va en Netplan. La gestiona Keepalived    |
| PC 2             | App Wiki 1                  | app1-wiki         | 192.168.0.13 | Nginx/Apache + PHP + MediaWiki              |
| PC 2             | App Wiki 2                  | app2-wiki         | 192.168.0.14 | Clon de App1, misma Wiki                    |
| PC 2             | NFS Server                  | srv-nfs           | 192.168.0.15 | /var/nfs/wikipics compartido              |
| PC 3             | Redis                       | srv-redis         | 192.168.0.16 | Caché sesiones + objetos                    |
| PC 3             | MariaDB                     | srv-db            | 192.168.0.17 | Base de datos wikidb                      |
| PC 1/3           | Monitor + DNS               | srv-monitor       | 192.168.0.20 | Prometheus + Grafana + Dnsmasq (DNS local)  |

*Flujo de tráfico principal*

Usuario → DNS (wiki.usfx.bo → 192.168.0.10) → VIP (Proxies)  
→ Nginx Balanceador → App1/App2 → (DB/Redis/NFS) → respuesta al usuario.

### 4.2. Estrategia Adoptada

- *Estrategia “Activos Redundantes” (HA):*  
  - Dos Proxies, dos Apps y servicios centrales separados (DB, Redis, NFS, DNS/Monitor).

- *Estrategia “Stateless en Frontend”:*  
  - App1 y App2 comparten:
    - Base de datos central (wikidb).
    - Carpeta de imágenes por NFS.
    - Sesiones centralizadas en Redis.  
  Al caerse una App, la otra puede seguir atendiendo usuarios con el mismo estado.

- *VIP como Punto Único Lógico de Acceso:*  
  - Se obliga a MediaWiki a usar $wgServer = "https://wiki.usfx.bo" (VIP).  
  - El usuario nunca ve las IPs internas (.13/.14).  

- *Monitorización y Observabilidad:*  
  - Todas las VMs exponen métricas en :9100 (node-exporter).  
  - Prometheus centraliza y Grafana visualiza en 192.168.0.20:3000.  

- *Seguridad en Capas:*  
  - Solo el *Monitor* ve todo.  
  - Solo *Proxies* llegan a Apps.  
  - Solo *Apps* llegan a DB, Redis y NFS.  
  - UFW filtra por IP y puerto según rol.

---

## 📋 V. Guía de Implementación y Puesta en Marcha

### 5.1. Pre-requisitos

- 3 PCs físicas con VirtualBox.  
- 8 Máquinas Virtuales con *Ubuntu Server 24.04*.  
- Router físico con red 192.168.0.0/24 y gateway 192.168.0.1.  
- Acceso sudo en todas las VMs.  
- Conectividad a Internet para instalación inicial de paquetes.

### 5.2. Despliegue por Fases

*Fase 1 – Cimientos: Base de Datos (srv-db) y Almacenamiento NFS (srv-nfs)*  
- Configurar IP estática con Netplan (.17 y .15).  
- Instalar y asegurar MariaDB (mysql_secure_installation).  
- Configurar bind-address = 0.0.0.0.  
- Crear BD wikidb y usuario wikiuser@'%'.  
- Instalar nfs-kernel-server.  
- Crear /var/nfs/wikipics, asignar nobody:nogroup.  
- Exportar carpeta a .13 y .14 vía /etc/exports.

*Fase 2 – Cuerpo: Servidores de Aplicación (App1 y App2)*  
- Configurar IP con Netplan (.13 y .14).  
- Instalar stack web: nginx + php-fpm + extensiones.  
- Descargar MediaWiki en /var/www/html/wiki.  
- Instalar cliente NFS (nfs-common) y montar 192.168.0.15:/var/nfs/wikipics en wiki/images (vía /etc/fstab).  
- Configurar Nginx virtual host para la Wiki.  
- Instalar MediaWiki vía navegador contra DB .17 (creación de LocalSettings.php).  
- Copiar LocalSettings.php a ambas Apps.

*Fase 3 – Entrada: Proxies y Balanceo (ha1-proxy, ha2-proxy)*  
- Configurar IPs .11 y .12 con Netplan.  
- Instalar Nginx como *reverse proxy* con upstream backend_wiki { .13, .14 }.  
- Instalar Keepalived, configurar virtual_ipaddress = 192.168.0.10 con prioridades distintas.  
- Ajustar $wgServer en LocalSettings.php para usar http://192.168.0.10 (y luego https://wiki.usfx.bo).  

*Fase 4 – Nitro: Redis (srv-redis)*  
- Configurar IP .16.  
- Instalar Redis, configurar bind 0.0.0.0 y protected-mode no.  
- Instalar php-redis en Apps.  
- Configurar en LocalSettings.php: $wgMainCacheType, $wgSessionCacheType, $wgParserCacheType, $wgMessageCacheType → redis.

*Fase 5 – Cerebro: Monitorización + DNS (srv-monitor)*  
- Configurar IP .20 y Netplan.  
- Instalar y configurar *Dnsmasq* para resolver:
  - wiki.usfx / wiki.usfx.bo → 192.168.0.10.  
- Configurar router para usar .20 como DNS.  
- Instalar Prometheus y editar prometheus.yml con targets :9100 de todas las VMs.  
- Instalar prometheus-node-exporter en todas las VMs.  
- Instalar Grafana y configurar datasource Prometheus, importar dashboard (Node Exporter Full).

*Fase 6 – Blindaje Final: SSH, SSL/TLS y UFW*  
- Cambiar SSH a puerto 2222 en todas las VMs, deshabilitar root login.  
- Generar certificados autofirmados en Proxies (wiki.usfx.bo).  
- Configurar Nginx para:
  - Redirigir 80 → 443.  
  - Servir HTTPS con headers de seguridad.  
- Ajustar UFW:
  - Proxies: permitir 80/443/2222 + VRRP + 9100 desde monitor.  
  - Monitor: 2222, 53, 3000, 9090.  
  - Apps: 2222, 80 solo desde Proxies, 9100 desde monitor.  
  - DB/Redis/NFS: 2222, puertos de servicio solo desde Apps, 9100 desde monitor.

### 5.3. Ficheros de Configuración Clave

(Rutas y archivos más importantes del proyecto)

- *Netplan (Todas las VMs)*  
  - /etc/netplan/50-cloud-init.yaml  
    - IP, gateway, DNS (local 127.0.0.1 para srv-monitor).

- *MediaWiki (Apps)*  
  - /var/www/html/wiki/LocalSettings.php  
    - Configuración de DB (host .17).  
    - $wgServer = "https://wiki.usfx.bo";  
    - Configuración de Redis ($wgObjectCaches['redis']).

- *Nginx (Apps y Proxies)*  
  - Apps: /etc/nginx/sites-available/wiki  
  - Proxies: /etc/nginx/sites-available/default  
    - upstream backend_wiki { 192.168.0.13:80; 192.168.0.14:80; }  
    - Bloque HTTPS con certificados y headers de seguridad.

- *Keepalived (Proxies)*  
  - /etc/keepalived/keepalived.conf  
    - state MASTER/BACKUP, priority, virtual_router_id, virtual_ipaddress.

- *NFS (srv-nfs)*  
  - /etc/exports → Export de /var/nfs/wikipics hacia .13 y .14.

- *Redis (srv-redis)*  
  - /etc/redis/redis.conf → bind 0.0.0.0, protected-mode no.

- *Dnsmasq (srv-monitor)*  
  - /etc/dnsmasq.conf → address=/wiki.usfx.bo/192.168.0.10.

- *Prometheus (srv-monitor)*  
  - /etc/prometheus/prometheus.yml → scrape_configs con todas las VMs.

- *Firewall (Todas las VMs)*  
  - Configuración vía ufw (política y reglas por rol).

---

## ✅ VI. Pruebas y Validación

| Prueba Realizada                                                | Resultado Esperado                                                                 | Resultado Obtenido (observado)      |
|-----------------------------------------------------------------|------------------------------------------------------------------------------------|-------------------------------------|
| Acceso a http://wiki.usfx.bo                                   | Redirección automática a https://wiki.usfx.bo y carga de la wiki.               | [ ] Por documentar / completar      |
| Failover de Proxy (apagado de ha1-proxy)                        | La VIP 192.168.0.10 migra a ha2-proxy, la wiki sigue funcionando.               | [ ]                                  |
| Balanceo de carga entre App1 y App2                             | Peticiones alternadas a .13 y .14 (via Nginx), mismas páginas en ambos nodos. | [ ]                                  |
| Subida de archivos en App1 y lectura desde App2 (NFS)           | Imagen subida en App1 visible inmediatamente en App2.                              | [ ]                                  |
| Persistencia de sesión con Redis (apagar App1 estando logueado) | Usuario sigue logueado atendido por App2.                                          | [ ]                                  |
| Acceso directo a DB (192.168.0.17:3306 desde PC física)       | Bloqueado por UFW (timeout / connection refused).                                  | [ ]                                  |
| Resolución DNS local (ping wiki.usfx.bo desde varias PCs)     | Resuelve siempre a 192.168.0.10.                                                 | [ ]                                  |
| Monitoreo (http://192.168.0.20:3000)                          | Dashboard de Grafana con métricas de las 8 VMs.                                    | [ ]                                  |

> (Marcar como ÉXITO/ERROR y complementar con capturas y logs en el informe extendido.)

---

## 📚 VII. Conclusiones y Lecciones Aprendidas

- Se logró diseñar e implementar una *infraestructura Wiki en Alta Disponibilidad*, con:
  - Proxies redundantes y VIP flotante.
  - Dos servidores de aplicación compartiendo base de datos, archivos y sesiones.
  - Servicios de soporte dedicados (DB, NFS, Redis, DNS/Monitor).

- El proyecto permite *mantener el servicio disponible* frente a:
  - Fallos de un Proxy (Keepalived asume la VIP en el otro nodo).  
  - Caída de una App (la otra sigue sirviendo la Wiki).  

- Se comprobó que no basta con “que la web no se caiga”:  
  - Es igual de importante garantizar *coherencia de datos*, sesiones y archivos al trabajar con múltiples nodos.  
  - NFS y Redis fueron claves para que el sistema sea realmente *stateful y consistente* en un entorno distribuido.

- A nivel de asignatura, se integraron en un solo proyecto los conceptos de:
  - *T1:* Continuidad Operacional.  
  - *T2:* Alta Disponibilidad y Tolerancia a Fallos.  
  - *T3/T4:* Balanceo, proxy y optimización del rendimiento.  
  - *T5:* Seguridad en varias capas (SSH, TLS, UFW, segmentación).  
  - *T6:* Gestión y despliegue ordenado por fases, con configuraciones reproducibles.

Como trabajo futuro, se podrían integrar:

- Backups automáticos de DB y archivos (combinando con un diseño tipo DRP).  
- Alta disponibilidad para la propia DB (réplicas) y NFS.  
- Automatización de despliegue con Ansible o Terraform para que toda la infraestructura pueda recrearse desde cero como *Infraestructura como Código (IaC)*.

---
<div align="center">

_Desarrollado con ❤️ y mucho café para Infraestructura 2/2025_

</div>
