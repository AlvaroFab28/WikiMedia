# 🎨 Anexo: Personalización Avanzada de Páginas y Gestión de Contenido

**Objetivo:** Crear páginas de asignaturas con diseño visual atractivo (CSS/HTML), eliminar elementos automáticos no deseados y comprender el flujo de publicación de archivos dentro de las mismas.

-----

## 1\. Gestión de la Tabla de Contenidos (Sumario) 🚫

Por defecto, MediaWiki genera automáticamente un cuadro de "Sumario" o Índice cuando una página tiene **3 o más títulos**. En portadas o páginas de diseño, esto suele romper la estética.

**Solución:**
Para desactivarlo, debe insertar la siguiente "palabra mágica" en la primera línea del código de la página:

```wikitext
__NOTOC__
```

*(Significa: No Table Of Contents).*

-----

## 2\. Creación de una Página de Asignatura Estilizada 🖌️

MediaWiki soporta HTML y CSS incrustado. A continuación, un código plantilla para crear una página de materia (Ej: "Apuntes de INFRA") que incluye:

  * Encabezado con color.
  * Ficha de datos flotante.
  * Tabla de descargas.
  * Categorización automática.

**Código Plantilla (Copiar y Pegar):**

```wikitext
__NOTOC__

<div style="background-color: #2a4b8d; color: white; padding: 20px; border-radius: 10px; text-align: center; margin-bottom: 20px; box-shadow: 2px 2px 5px #888;">
  <h1 style="border:none; margin:0; color: white; font-family: sans-serif;">🏗️ Infraestructura de Redes y Servicios</h1>
  <span style="font-size: 1.2em;">Repositorio Oficial - Semestre 1/2025</span>
</div>

{| style="float: right; width: 280px; margin-left: 20px; background: #f8f9fa; border: 1px solid #ddd; padding: 10px; border-radius: 5px;"
! style="background: #eee; padding: 5px; border-bottom: 2px solid #2a4b8d;" | 📋 Información
|-
| 
* **Semestre:** 1ro
* **Créditos:** 6
* **Estado:** Activa
* **Admin:** [[Usuario:Admin|Admin]]
|}

Bienvenido al espacio de **Infraestructura**. Aquí encontrará los laboratorios de Docker, Linux y diagramas de red del cluster.

== 📥 Material Descargable ==
{| class="wikitable" style="width: 100%;"
! 📄 Archivo !! 📝 Descripción !! 📅 Fecha
|-
| [[Archivo:Guia_Linux.pdf|120px|center]] || **Guía de Comandos**<br>Manual básico de terminal. || 24-Nov
|-
| [[Archivo:Diagrama_Red.png|120px|center]] || **Topología del Proyecto**<br>Esquema de IPs. || 25-Nov
|}

<div style="background-color: #ffe6e6; border-left: 5px solid #ff0000; padding: 10px; margin: 20px 0;">
'''⚠️ Atención:''' Los laboratorios deben subirse antes del viernes.
</div>

[[Categoría:Primer Semestre]]
```

-----

## 3\. Flujo de Trabajo: Cómo publicar contenido en una página 📤

A diferencia de sistemas como Google Drive, en una Wiki **no se sube el archivo "adentro" de la página**. El proceso consta de dos pasos: **Subir** al servidor y **Linkear** en la página.

### Paso 1: Subir el Archivo (Upload)

1.  Ir a **Herramientas \> Subir archivo**.
2.  Seleccionar el documento (Ej: `Laboratorio1.pdf`).
3.  Subirlo al servidor.

### Paso 2: Linkear en la Página (Embed)

1.  Ir a la página de la materia (Ej: "Apuntes de INFRA") y hacer clic en **Editar**.

2.  Ubicar el cursor donde se desea mostrar el archivo (por ejemplo, dentro de una tabla).

3.  Escribir el código de enlace:

      * **Para crear un link de descarga:**
        `[[Media:Laboratorio1.pdf | Descargar Lab 1]]`

      * **Para mostrar una miniatura (imágenes/PDFs):**
        `[[Archivo:Laboratorio1.pdf | miniatura | Texto descriptivo]]`

4.  **Guardar cambios.**

-----