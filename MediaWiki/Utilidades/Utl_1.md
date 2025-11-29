# TUTORIAL CREAR NUEVO USUARIO EN UNA VM
### 1\. Cambiar el nombre de la máquina (Hostname)

1.  **Cambia el nombre en el sistema:**
    Sustituye `nuevo-servidor` por el nombre que quieras.

    ```bash
    sudo hostnamectl set-hostname nuevo-servidor
    ```

2.  **Reinicia:**
    Debes reiniciar para que el cambio de nombre surga efecto

    ```bash
    sudo reboot
    ```
-----

### 2\. Cambiar la contraseña

Si solo quieres cambiar la contraseña de tu usuario actual:

```bash
passwd
```

*(Te pedirá la actual, luego la nueva y confirmar la nueva. Recuerda que en Linux no se ven los asteriscos al escribir).*

-----

### 3\. Crear un nuevo Usuario

**Paso A: Crear el usuario nuevo**

```bash
# 1. Crear usuario (te pedirá contraseña y datos, puedes dejar los datos en blanco)
sudo adduser nuevo_usuario

# 2. Darle permisos de administrador (sudo)
sudo usermod -aG sudo nuevo_usuario
```

**Paso B: Probar y borrar el viejo**

1.  Escribe `exit` para salir de la sesión.
2.  Entra con tu **nuevo\_usuario**.
3.  Si todo funciona, borra el usuario antiguo (y su carpeta home para limpiar espacio):
    ```bash
    sudo deluser --remove-home usuario_viejo
    ```

-----

### Resumen rápido de comandos

| Acción | Comando |
| :--- | :--- |
| **Ver nombre actual** | `hostname` |
| **Cambiar Hostname** | `sudo hostnamectl set-hostname <nombre>` |
| **Cambiar Pass** | `passwd` |
| **Nuevo Admin** | `sudo adduser <nombre>` + `sudo usermod -aG sudo <nombre>` |
