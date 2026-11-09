# Instalación y cambio de puerto SSHD en CentOS Stream 10

**Integrantes:** Francisco Morillo y Esteban Nievencela

**Objetivo:** Configurar el servidor SSH para escuchar en el puerto TCP 2237 y acceder desde Windows.

> **Precaución:** Si ya estás conectado por SSH, conserva la sesión actual abierta hasta comprobar que funciona el nuevo puerto.

### Paso 1. Instalar SSH y conocer la IP de CentOS
```bash
sudo dnf install -y openssh-server
ip -4 a
```
Identificar la IP de la interfaz que se comunica con Windows.

### Paso 2. Activar el servicio y guardar una copia de seguridad
```bash
sudo systemctl enable --now sshd
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
```

### Paso 3. Editar la configuración del servidor
```bash
sudo nano /etc/ssh/sshd_config
```
Buscar `#Port 22` (si existe) y establecer una línea activa:
```text
Port 2237
```
El archivo correcto es **`sshd_config`** (servidor), no `ssh_config` (cliente). Comprobar que no existan otras directivas `Port` conflictivas, incluso en archivos incluidos. Guardar con `Ctrl+O`, `Enter`, `Ctrl+X`.

### Paso 4. Autorizar el puerto con SELinux
```bash
sudo dnf install -y policycoreutils-python-utils
getenforce
sudo semanage port -l | grep ssh_port_t
```
Si SELinux está en modo `Enforcing` o `Permissive` y el puerto 2237 no aparece ya como puerto SSH:
```bash
sudo semanage port -a -t ssh_port_t -p tcp 2237
```
Si el puerto ya pertenece a otro tipo SELinux, investigar el conflicto antes de cambiarlo. No desactivar SELinux para resolverlo.

### Paso 5. Permitir el puerto en el firewall
```bash
sudo firewall-cmd --permanent --add-port=2237/tcp
sudo firewall-cmd --reload
sudo firewall-cmd --list-ports
```
Verificar la zona activa de la interfaz (`sudo firewall-cmd --get-active-zones`) y aplicar la regla a esa zona si no coincide con la predeterminada.

### Paso 6. Validar y reiniciar SSHD
```bash
sudo sshd -t
sudo systemctl restart sshd
sudo systemctl status sshd
```
`sudo sshd -t` debe terminar sin errores antes de reiniciar. Verificar que el servicio figure como `active (running)`.

### Paso 7. Verificar que escucha en el puerto nuevo
```bash
sudo ss -lntp | grep 2237
```

### Paso 8. Conectarse desde Windows
Desde PowerShell:
```powershell
ssh -p 2237 USUARIO_CENTOS@IP_DE_CENTOS
```
O desde MobaXterm: seleccionar sesión SSH, introducir la IP, el usuario de CentOS y el puerto `2237`.

### Paso 9. Comprobación y seguridad
Confirmar que la conexión por 2237 funciona antes de considerar cerrar el puerto 22. Si se necesita bloquear el acceso anterior, verificar primero que ningún otro usuario o servicio dependa de él; revisar reglas del firewall y puertos activos antes de modificar nada. Cambiar de puerto no sustituye el uso de contraseñas fuertes o claves SSH.

![Evidencia original de SSHD](https://github.com/user-attachments/assets/f046d63e-8ce6-44fd-98a0-d9077cdd3d23)

**Evidencias sugeridas:** IP, archivo `sshd_config`, autorización SELinux, firewall, estado de `sshd`, puerto escuchando y conexión desde Windows.
