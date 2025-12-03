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
* [📖 **Índice del material de temas avanzados**](Temas/Indice_de_Temas.md)

---

### 🛠️ 2. Proyecto Final SIS313: Plataforma Wiki Universitaria en Alta Disponibilidad (HA)
Despliegue de un Cluster Web Escalable, Resiliente y Monitoreado.

---
*Asignatura:* SIS313: Infraestructura, Plataformas Tecnológicas y Redes  
*Semestre:* 2/2025  
*Docente:* Ing. Marcelo Quispe Ortega 


## 👥 Miembros del Grupo G-07
| Rol | Miembro | GitHub |
| :--- | :--- | :--- |
| **Proxy-1, Proxy-2, DNS, Monitoreo Grafana** | Villena Mamani Alvaro Fabian | @AlvaroFab28 |
| **App-1, App-2, NFS** | Castro Siñanis Jose Luis | @josezx |
| **Redis, Base de Datos** | Villca Araca Jhesica | @cero0202 |

---
### 2.1 Objetivo del Proyecto 
*Objetivo:*  
Diseñar y configurar una *plataforma MediaWiki Universitaria en Alta Disponibilidad* que:

- Utilice *dos servidores de aplicación* detrás de *dos Proxies en HA* con IP Virtual (VIP).
- Centralice datos en una *base de datos única (MariaDB)*.
- Comparta archivos subidos mediante un *servidor NFS*.
- Gestione sesiones y caché de objetos con *Redis*.
- Resuelva nombres con un *DNS local (Dnsmasq)* y monitorice toda la infraestructura con *Prometheus + Grafana*.
- Implemente *hardening de red y servicios* (SSH, TLS, UFW).

De forma que, ante la caída de un *Proxy* o una *App*, la wiki siga disponible para los usuarios, manteniendo coherencia de datos, archivos y sesiones.

---

### 2.2 Justificación e Importancia
*Justificación:*  
En una universidad, una *Wiki Académica* es un servicio crítico para:

- Publicar material de clase, laboratorios y proyectos.
- Centralizar conocimiento entre diferentes materias y carreras.
- Permitir colaboración entre docentes y estudiantes.

Un único servidor MediaWiki introduce un *Single Point of Failure*: si esa máquina falla, toda la comunidad pierde acceso. Además, sin diseño cuidadoso:

- Archivos subidos pueden quedar desincronizados entre servidores.
- Sesiones de usuario se pierden al cambiar de nodo.
- La base de datos y los servicios internos quedan expuestos si no hay hardening.

Este proyecto:

- Atiende la *Continuidad Operacional (T1)* mediante:
  - Proxies redundantes con VIP.
  - Dos servidores de aplicación con datos y archivos compartidos.
- Implementa *Alta Disponibilidad (T2)* y *Optimización (T4)*:
  - Balanceo de carga con Nginx.
  - Redis para caché/sesiones.
- Refuerza la *Seguridad (T5)*:
  - SSH en puerto no estándar, sin login de root.
  - Cifrado de tráfico con HTTPS (TLS).
  - Firewall UFW con reglas por rol e IP.

---

Los pasos desarrollados en el proyecto.

| Tipo | Recurso | Descripción |
| :--- | :--- | :--- |
| 🗺️ | [**Diseño Topológico**](MediaWiki/Diseño_Topologico.png) | El mapa del tesoro. Miralo bien. |
| 📝 | [**Guía Paso a Paso**](MediaWiki/MediaWiki.md) | La biblia de la instalación. |
| ⚙️ | [**Utilidades Básicas**](MediaWiki/Utilidades/Utl_1.md) | Cambio de Hostname, Password, Usuario. |
| 🏁 | [**Resultados**](MediaWiki/Conclusiones.md) | Resumen de logros y futuras mejoras. |

#### ⚠️ **IMPORTANTE: MÁQUINAS VIRTUALES**
> VMs utilizadas en el proyecto.
>
> 👉 [**CLICK ACÁ PARA IR AL DRIVE DE VMs (v.29/11/2025)**](https://drive.google.com/drive/folders/1c1CrpNQM8bl0YEJp2GHndt4T4YmYpKk7?usp=drive_link) 👈

---

### 🐛 3. Fixes & Mejoras 
¿Se rompió algo? ¿La página carga más lento ? aqui hay soluciones.

* [**Fix 1:** Error de carga de archivos](MediaWiki/Fixes/Fix1.md)
* [**Fix 2:** Nuevos formatos de archivos](MediaWiki/Fixes/Fix2.md)
* [**Fix 3:** Tamaño máximo de archivos](MediaWiki/Fixes/Fix3.md)
* [**Fix 4:** Cambiar el logotipo](MediaWiki/Fixes/Fix4.md) _(Ponete creativo)_
* [**Fix 5:** Personalizar Página principal](MediaWiki/Fixes/Fix5.md)
    * ↳ [**Fix 5.1:** Subir página personalizada por subcategorías (1er Semestre)](MediaWiki/Fixes/Fix5_1.md)
* [**Fix 6:** ⚡ Mejora de Red (Anti-Lag)](MediaWiki/Fixes/Fix6.md)

---

### 🧪 4. Zona de Testing 
1.  [📊 **Test 1:** Monitoreo de BD](MediaWiki/Test1.md)
2.  [🌐 **Test 2:** IP del Server Wiki](MediaWiki/Tests/Test2.md)
3.  [⚖️ **Test 3:** Balanceo de carga Redis](MediaWiki/Test3.md) _(Una joyita)_
4.  [🛡️ **Test 4:** Failover en Proxy Principal](MediaWiki/Test4.md)
5.  [🔄 **Test 5:** Failover Wiki](MediaWiki/Test5.md)

---

### 3.1. Tecnologías Clave
- *MediaWiki 1.42.x*  
  Función específica: Plataforma wiki colaborativa universitaria (portal de materiales, tareas, proyectos).

- *Nginx (Proxies y/o Apps)*  
  Función específica: Proxy inverso y balanceador de carga entre App1 y App2; terminación TLS en Proxies.

- *Apache/Nginx + PHP-FPM (App1 y App2)*  
  Función específica: Servidores de aplicación que procesan el código PHP de MediaWiki y entregan páginas dinámicas.

- *MariaDB (srv-db – 192.168.0.17)*  
  Función específica: Base de datos central wikidb para toda la información estructurada de la wiki.

- *NFS (srv-nfs – 192.168.0.15)*  
  Función específica: Almacenamiento compartido para /images de MediaWiki; asegura que los archivos subidos estén disponibles en ambas Apps.

- *Redis (srv-redis – 192.168.0.16)*  
  Función específica: Caché de objetos y *almacenamiento de sesiones*; permite que el login persista aunque cambie el backend.

- *Keepalived + VRRP (ha1-proxy, ha2-proxy)*  
  Función específica: Gestión de IP Virtual 192.168.0.10 (VIP) con roles MASTER/BACKUP para failover automático de entrada a la wiki.

- *Dnsmasq (srv-monitor – 192.168.0.20)*  
  Función específica: DNS local que resuelve wiki.usfx.bo (o wiki.usfx) hacia la VIP 192.168.0.10.

- *Prometheus + Node Exporter*  
  Función específica: Recolección de métricas de CPU, RAM, disco y red de las 8 VMs.

- *Grafana*  
  Función específica: Visualización y dashboards en tiempo real del estado de la infraestructura HA.

- *UFW (Uncomplicated Firewall)*  
  Función específica: Firewall por host con política deny incoming y reglas específicas para cada rol.

- *OpenSSL*  
  Función específica: Generación de certificados autofirmados TLS para wiki.usfx.bo.

- *Netplan*  
  Función específica: Configuración de IPs estáticas, gateway y DNS en todas las VMs.

---

### 3.2. Conceptos de la Asignatura Puestos en Práctica (T1 - T6)

- *Alta Disponibilidad (T2) y Tolerancia a Fallos:* ✅  
  - Dos Proxies (MASTER/BACKUP) con VIP 192.168.0.10 usando Keepalived/VRRP.  
  - Dos servidores de aplicación sirviendo la misma wiki (DB, archivos y sesiones compartidas).

- *Seguridad y Hardening (T5):* ✅  
  - SSH en puerto 2222, sin acceso root por SSH.  
  - HTTPS obligatorio con redirección HTTP→HTTPS y cabeceras de seguridad (X-Frame-Options, etc.).  
  - UFW con reglas por rol:  
    - Proxies: solo 80/443/2222 + VRRP.  
    - Apps: puerto 80 solo desde Proxies, 2222, 9100 desde monitor.  
    - DB/Redis/NFS: accesibles solo desde IPs autorizadas.

- *Automatización y Gestión (T6):* ✅  
  - Configuraciones reproducibles de Netplan, Keepalived, Nginx, Dnsmasq, etc.  
  - Montaje automático de NFS vía /etc/fstab.  
  - Despliegue organizado por fases y roles claros, evitando configuraciones ad-hoc.

- *Balanceo de Carga/Proxy (T3/T4):* ✅  
  - Nginx como reverse proxy y load balancer (upstream backend_wiki con App1/App2).  
  - VIP como punto único de acceso lógico hacia el clúster.

- *Monitoreo (T4/T1):* ✅  
  - Node Exporter en todas las VMs (puerto 9100).  
  - Prometheus centralizando métricas y Grafana mostrando panel unificado.

- *Networking Avanzado (T3):* ✅  
  - Diseño de topología con VIP, VRRP y segmentación lógica por rol.  
  - DNS local con Dnsmasq para resolver dominio → VIP en toda la red.

---


### 4.2. Estrategia Adoptada

- *Estrategia de Alta Disponibilidad basada en VIP:*  
  La IP 192.168.0.10 no pertenece físicamente a ninguna VM; es gestionada por Keepalived.  
  Si el Proxy MASTER cae, el BACKUP asume la VIP y el servicio continúa operativo.

- *Estrategia “Stateful compartido”:*  
  - Datos en una única DB central (wikidb en .17).  
  - Archivos subidos compartidos vía NFS (/var/nfs/wikipics montado en wiki/images).  
  - Sesiones y caché en Redis (.16:6379).  
  Esto garantiza que, aunque el usuario cambie de App, vea exactamente el mismo estado.

- *Estrategia de Seguridad en Capas:*  
  - Dominio y TLS terminan en Proxies; backend interno solo HTTP.  
  - Servicios internos (DB/Redis/NFS) aislados del exterior mediante UFW e IPs permitidas.  
  - DNS propio (srv-monitor) para independencia de Internet y menor latencia interna.

---
### 5 Conclusiones 


Se implementó una Wiki Universitaria en Alta Disponibilidad con proxies en HA, dos servidores de aplicación, DB centralizada, NFS, Redis y monitoreo, logrando que el servicio siga funcionando aunque falle uno de los nodos. El proyecto integró en la práctica los temas T1–T6 (continuidad, HA, balanceo, optimización, seguridad y gestión) y demostró que no basta con “tener dos servidores”, sino diseñar bien cómo comparten datos, sesiones y acceso seguro. Como trabajo futuro, se puede añadir un plan de backups/DRP y automatizar todo el despliegue con Ansible.
---
<div align="center">

_Desarrollado con ❤️ y mucho café para Infraestructura 2/2025_

</div>
