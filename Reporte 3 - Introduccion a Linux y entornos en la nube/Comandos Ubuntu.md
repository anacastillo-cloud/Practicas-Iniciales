# Comandos de Ubuntu y para qué sirven

| **Comando** | **Para qué sirve** |
|---|---|
| `pwd` | Muestra la ruta completa (ubicación) donde te encuentras actualmente en la terminal |
| `ls` | Lista los archivos y carpetas del directorio actual |
| `ls -la` | Lista todo el contenido, incluyendo archivos ocultos y detalles (permisos, dueño, tamaño, fecha) |
| `cd [directorio]` | Cambia de directorio, permite moverte entre carpetas del sistema |
| `mkdir [nombre]` | Crea un nuevo directorio (carpeta) |
| `cp [origen] [destino]` | Copia un archivo o carpeta a otra ubicación, sin eliminar el original |
| `mv [origen] [destino]` | Mueve un archivo a otra ubicación, o lo renombra si se usa en la misma carpeta |
| `rm [archivo]` | Elimina un archivo de forma permanente |
| `sudo [comando]` | Ejecuta un solo comando con privilegios de superusuario, sin cambiar de sesión — la forma más segura de administrar el sistema |
| `su` | Cambia por completo la sesión al usuario root (superusuario), dando acceso total a todo el sistema |
| `exit` | Regresa de la sesión root a tu sesión de usuario normal |
| `chmod [permisos] [archivo]` | Modifica los permisos de lectura, escritura y ejecución de un archivo |
| `chown [usuario:grupo] [archivo]` | Cambia el propietario y/o grupo de un archivo o carpeta |
| `nano [archivo]` | Abre un editor de texto simple, con atajos visibles en pantalla (`Ctrl+O` guarda, `Ctrl+X` sale) |
| `vim [archivo]` | Abre un editor de texto avanzado con modo comando (`i` para insertar texto, `:wq` para guardar y salir) |
| `sudo apt update` | Sincroniza el índice de paquetes disponibles con los repositorios (no instala nada, solo actualiza la lista) |
| `sudo apt upgrade` | Actualiza todos los paquetes ya instalados a su última versión disponible |
| `sudo apt install [paquete]` | Instala un paquete nuevo desde los repositorios (ej. `apache2`) |
| `sudo apt remove` / `apt purge [paquete]` | Desinstala un paquete; `remove` deja su configuración, `purge` la borra también |
| `sudo systemctl status apache2` | Muestra si el servicio de Apache2 está activo y corriendo correctamente |
