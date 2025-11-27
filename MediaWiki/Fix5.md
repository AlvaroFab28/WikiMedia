# 📘 Guía de Implementación: Repositorio Académico Dinámico en MediaWiki

**Objetivo:** Transformar la página principal de MediaWiki en un navegador de archivos jerárquico (tipo explorador de carpetas) utilizando categorías dinámicas.
**Prerrequisitos:** Tener instalado MediaWiki y acceso de administrador.

-----

## FASE 1: Configuración del Backend (SysAdmin) ⚙️

Para que el sistema pueda mostrar árboles de carpetas, necesitamos activar la extensión nativa `CategoryTree`.

**Importante:** Al estar en un clúster de Alta Disponibilidad, este cambio debe realizarse en **todos los nodos de aplicación** (App 1 y App 2).

1.  **Editar la configuración:**
    Acceder a la terminal de la VM y editar el archivo `LocalSettings.php`:

    ```bash
    sudo nano /var/www/html/wiki/LocalSettings.php
    ```

2.  **Agregar el código de activación:**
    Ir al final del archivo y pegar las siguientes líneas:

    ```php
    # --- Extensión CategoryTree (Árbol de Directorios) ---
    wfLoadExtension( 'CategoryTree' );

    # Configuración base
    $wgCategoryTreeForceHeaders = true; 
    # (Opcional) Define una categoría raíz para la barra lateral
    $wgCategoryTreeSidebarRoot = "Categoría:Repositorio Académico"; 
    ```

3.  **Guardar y Replicar:**
    Guardar cambios (`Ctrl+O`, `Enter`, `Ctrl+X`) y repetir el proceso en la segunda VM (App 2) para mantener la consistencia del clúster.

-----

## FASE 2: Diseño de la Interfaz (Frontend) 🎨

Configuración de la Página Principal para mostrar el explorador de archivos.

1.  Iniciar sesión en la Wiki como Administrador.
2.  Ir a la **Página principal** y seleccionar la pestaña **Editar**.
3.  Reemplazar el contenido existente con el siguiente código *Wikitext*:

<!-- end list -->

```wikitext
{| class="wikitable" style="width: 100%;"
! colspan="2" | 📂 Repositorio Académico Universitario
|-
| style="width: 40%; vertical-align: top;" |
=== 📚 Navegación por Semestre ===
Explore el material académico desplegando las carpetas a continuación.

<categorytree mode="pages" depth="0">Repositorio Académico</categorytree>

| style="width: 60%; vertical-align: top;" |
=== 🚀 Panel de Control ===
Bienvenido al banco de proyectos y apuntes.

==== Accesos Rápidos ====
* [[Ayuda:Contenidos|¿Cómo subir un archivo?]]
* [[Especial:SubirArchivo|Subir un nuevo documento]]

==== 📢 Avisos ====
Recuerde categorizar correctamente sus subidas para que aparezcan en el árbol de la izquierda.
|}
```

4.  **Guardar** la página. (Se verá un error de "Categoría no encontrada", esto es normal hasta completar la Fase 3).

-----

## FASE 3: Creación de la Estructura de Directorios 🗂️

MediaWiki no usa carpetas reales, usa **Categorías** anidadas. Debemos crear la jerarquía Padre \> Hijo.

### 1\. Crear el Directorio Raíz

1.  En el buscador de la Wiki, ingresar: `Categoría:Repositorio Académico`.
2.  Hacer clic en el enlace rojo para crearla.
3.  Escribir una breve descripción (ej: *"Raíz del sistema de archivos"*).
4.  **Guardar página**.

### 2\. Crear Sub-Directorios (Ej. "Primer Semestre")

1.  En el buscador, ingresar el nombre de la subcarpeta: `Categoría:Primer Semestre`.
2.  Crear la página.
3.  **Paso Crítico:** Para vincularla a la raíz, escribir el siguiente código al final del texto:
    ```wikitext
    [[Categoría:Repositorio Académico]]
    ```
4.  **Guardar página**.
5.  *Repetir este paso para "Segundo Semestre", "Tercer Semestre", etc.*

-----

## FASE 4: Manual de Usuario (Cómo poblar el repositorio) 👥

Instrucciones para que los usuarios suban contenido y este aparezca automáticamente en el lugar correcto.

### Caso A: Subir Archivos (PDFs, Word, Imágenes)

1.  Ir a **Herramientas \> Subir archivo**.
2.  Seleccionar el archivo desde la PC.
3.  En el cuadro **Resumen**, es obligatorio ingresar la etiqueta de la categoría destino.
      * *Ejemplo:* Para enviarlo a la carpeta de 1er Semestre:
    <!-- end list -->
    ```wikitext
    [[Categoría:Primer Semestre]]
    ```
4.  Hacer clic en **Subir un archivo**.

### Caso B: Crear Páginas o Artículos

1.  Crear una página nueva con el nombre del tema (ej: "Apuntes de Álgebra").
2.  Redactar el contenido.
3.  Al final del documento, agregar la etiqueta de categoría:
    ```wikitext
    [[Categoría:Primer Semestre]]
    ```
4.  **Guardar página**.

-----