# 🧪 Guía: Evidencia Visual de Balanceo de Carga (Cluster HA)

> **Objetivo:** Mostrar visualmente en la interfaz de la Wiki qué dirección IP (nodo) está respondiendo la solicitud HTTP en ese preciso momento.
> **Lógica:** Usamos la variable de servidor `$_SERVER['SERVER_ADDR']` de PHP para inyectar la IP local de la VM en el HTML.

## Requisito Crítico ⚠️

Este cambio se debe aplicar en **TODOS los Servidores de Aplicación** (en tu caso: App 1 `.13` y App 2 `.14`) de forma idéntica.

-----

## Opción A: La Marquesina (Alta Visibilidad) 📢

*Recomendada para demos y presentaciones. Crea una barra de aviso en el tope de la página.*

1.  **Editar configuración en App 1 (`.13`) y App 2 (`.14`):**

    ```bash
    sudo nano /var/www/html/wiki/LocalSettings.php
    ```

2.  **Agregar al final del archivo:**

    ```php
    # --- DEBUG DE CLUSTER (Barra Superior) ---
    # Muestra la IP del nodo activo bien visible arriba
    $wgSiteNotice = "<div style='background-color: #ffeba0; border: 1px solid #e6db55; color: #555; padding: 5px; text-align: center; font-weight: bold;'>
        ⚡ Balanceo de Carga Activo | Nodo Atendiendo: " . $_SERVER['SERVER_ADDR'] . "
    </div>";
    ```

3.  **Guardar y Salir.**

-----

## Opción B: El Pie de Página (Sutil y Profesional) 👣

*Recomendada si querés dejarlo permanente sin que moleste el diseño. Aparece junto a "Política de Privacidad".*

1.  **Editar configuración en App 1 (`.13`) y App 2 (`.14`):**

    ```bash
    sudo nano /var/www/html/wiki/LocalSettings.php
    ```

2.  **Agregar al final del archivo:**

    ```php
    # --- DEBUG DE CLUSTER (Enlace al Pie) ---
    # Inyecta la IP en la lista de enlaces legales del footer
    $wgHooks['SkinAddFooterLinks'][] = function ( $skin, $key, &$footerLinks ) {
        # 'places' es la fila de enlaces de privacidad y descargo
        if ( $key === 'places' ) {
            $footerLinks['server_ip'] = "🔧 Nodo: " . $_SERVER['SERVER_ADDR'];
        }
        return true;
    };
    ```

3.  **Guardar y Salir.**

-----

## 🏁 Cómo realizar la Prueba (El Show)

Para demostrar que el sistema funciona:

1.  Abrí tu navegador en la dirección del Cluster: `http://192.168.0.10`
2.  Observá el número de IP que aparece (ej: `.13`).
3.  Presioná **F5** (o botón de refrescar) varias veces seguidas.
4.  **Resultado esperado:** El número debe alternar entre `.13` y `.14` (dependiendo de la configuración de *Round Robin* de tu Proxy).


## 🧠 Análisis Técnico: Comportamiento del Balanceo y Sesiones

> **Contexto:** Al realizar la prueba de estrés con **F5** (refresco), se observa que la IP del nodo servidor a veces alterna inmediatamente (`.13` -> `.14`) y otras veces se mantiene en el mismo nodo por varias peticiones seguidas.

## 1. ¿Por qué la IP no cambia siempre "uno a uno"? (Keep-Alive) 🕸️
Aunque Nginx está configurado con el algoritmo **Round Robin** (turno rotativo), el comportamiento observado es correcto debido a la optimización de los navegadores modernos.

* **El Fenómeno "Keep-Alive":** Los navegadores intentan ser eficientes y mantienen la conexión TCP abierta con el servidor (el Proxy) durante unos segundos para no perder tiempo abriendo y cerrando "tuberías".
* **El Resultado:** Si refrescamos la página muy rápido, el Proxy puede reutilizar la conexión abierta hacia el nodo actual (ej. `.13`) en lugar de abrir una nueva hacia el otro nodo (`.14`).
* **Validación:** Esto **no es un fallo**. Para forzar el cambio inmediato, se puede usar **Modo Incógnito** (que no guarda caché ni conexiones previas) o `Ctrl + F5`. Lo importante es que, ante la caída de un nodo, el tráfico se redirige automáticamente.

## 2. El Rol de Redis: Persistencia de Sesión (Cluster Stateless) 💾
Durante la navegación, se observó en el monitor de Redis (`redis-cli monitor`) que las peticiones de lectura/escritura provenían de ambas IPs (`.13` y `.14`) alternadamente, mientras el usuario permanecía logueado sin interrupciones.

* **Arquitectura Stateless:** Las Apps (`.13` y `.14`) no guardan la sesión del usuario en su disco local, sino que la delegan a la VM Redis (`.16`).
* **La Prueba del Éxito:**
    1.  El usuario se loguea en el **Nodo A**.
    2.  El Proxy balancea la siguiente petición al **Nodo B**.
    3.  El Nodo B consulta a Redis, valida la sesión y sirve el contenido.
    4.  **Resultado:** El usuario percibe una navegación fluida y continua, sin cierres de sesión, independientemente de qué servidor físico lo esté atendiendo.

## Conclusión del Test
El sistema demuestra un funcionamiento correcto de **Alta Disponibilidad (HA)**:
1.  **Nginx Proxy:** Distribuye la carga de tráfico correctamente entre los nodos disponibles.
2.  **Redis Cache:** Centraliza las sesiones, permitiendo que la aplicación sea agnóstica al servidor (Stateless).