# Instalación y configuración de VSFTPD en CentOS Stream 10

**Integrantes:** Francisco Morillo y Esteban Nievencela

**Objetivo:** Instalar el servidor FTP, crear una cuenta de práctica y comprobar una conexión desde FileZilla en una red de laboratorio.

> **Seguridad:** FTP tradicional transmite credenciales y datos sin cifrado. Para uso real, se recomienda SFTP mediante SSH o FTPS correctamente configurado. No publicar contraseñas en GitHub.

### Paso 1. Instalar VSFTPD
```bash
sudo dnf install -y vsftpd
```

### Paso 2. Guardar una copia de seguridad
```bash
sudo cp /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf.backup
```

### Paso 3. Crear una cuenta de prueba
```bash
sudo useradd -m efnievecela_ftp
sudo passwd efnievecela_ftp
```
Si el usuario ya existe, omitir `useradd` y utilizar solamente `passwd`. Introducir una contraseña segura cuando se solicite; no escribirla en el documento. Se utiliza una cuenta con shell válido para evitar problemas de autenticación por PAM que pueden aparecer con `/sbin/nologin`.

### Paso 4. Configurar VSFTPD
```bash
sudo nano /etc/vsftpd/vsftpd.conf
```
Comprobar que estén presentes, sin duplicados contradictorios, las siguientes directivas (editar las existentes cuando corresponda):
```ini
anonymous_enable=NO
local_enable=YES
write_enable=YES
local_umask=022
chroot_local_user=YES
pasv_enable=YES
pasv_min_port=30000
pasv_max_port=30010
```
Con `chroot_local_user=YES`, la raíz del usuario no debe ser escribible por él. Para una cuenta nueva de laboratorio, preparar un directorio raíz protegido y un subdirectorio de carga:
```bash
sudo chown root:root /home/efnievecela_ftp
sudo chmod 755 /home/efnievecela_ftp
sudo mkdir -p /home/efnievecela_ftp/archivos
sudo chown efnievecela_ftp:efnievecela_ftp /home/efnievecela_ftp/archivos
```
El usuario podrá cargar archivos en `archivos`, no en la raíz de su sesión FTP. Si el directorio personal ya contiene datos, revisar propietarios y permisos antes de aplicar estos comandos.

### Paso 5. Ajustar SELinux si impide el acceso
```bash
getenforce
sudo ausearch -m AVC -ts recent
```
No desactivar SELinux. Si hay denegaciones al acceder a `/home/efnievecela_ftp`, revisar sus etiquetas y aplicar una política adecuada para el directorio FTP. En un laboratorio, una opción de alcance amplio es `sudo setsebool -P ftpd_full_access on`, pero **no debe activarse sin autorización**, porque amplía los permisos del servicio. Preferir políticas y etiquetas específicas según el entorno.

### Paso 6. Habilitar e iniciar VSFTPD
```bash
sudo systemctl enable --now vsftpd
sudo systemctl restart vsftpd
sudo systemctl status vsftpd
```
El servicio debe aparecer `active (running)`.

### Paso 7. Permitir FTP y puertos pasivos en el firewall
```bash
sudo firewall-cmd --permanent --add-service=ftp
sudo firewall-cmd --permanent --add-port=30000-30010/tcp
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
```
Comprobar que se está modificando la zona de firewall utilizada por la interfaz de red de CentOS.

### Paso 8. Consultar la IP
```bash
ip -4 a
```
La IP utilizada anteriormente en la práctica puede cambiar; verificarla antes de conectar. Si se usa VirtualBox con NAT, configurar el acceso de red o las redirecciones necesarias, incluidos los puertos pasivos. En redes con NAT, el servidor puede necesitar anunciar su dirección accesible mediante `pasv_address`.

### Paso 9. Probar desde FileZilla en Windows
Instalar y abrir **FileZilla Client**. Configurar una conexión FTP solo dentro de una red de laboratorio de confianza:

- **Servidor:** IP actual de CentOS.
- **Protocolo:** FTP (para esta prueba de laboratorio; preferir SFTP o FTPS fuera de ella).
- **Puerto:** 21.
- **Modo de acceso:** Normal.
- **Usuario:** `efnievecela_ftp`.
- **Contraseña:** la establecida en el paso 3 (no documentarla).
- **Modo de transferencia:** Pasivo.

Conectarse y probar la carga de un archivo de texto en la carpeta `archivos`. No exponer el servicio FTP sin cifrado a Internet.

### Paso 10. Verificación final
```bash
sudo systemctl status vsftpd
sudo ss -lntp | grep ':21'
sudo journalctl -u vsftpd -n 30 --no-pager
```
Si falla el inicio de sesión, comprobar contraseña, permisos del directorio, reglas de firewall, registros del servicio y denegaciones SELinux.

**Evidencias sugeridas:** instalación, archivo de configuración, usuario creado (sin mostrar contraseña), estado del servicio, firewall, FileZilla conectado y archivo de prueba transferido.
