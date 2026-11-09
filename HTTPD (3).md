# Instalación y configuración de HTTPD en CentOS Stream 10

**Integrantes:** Francisco Morillo y Esteban Nievencela

**Objetivo:** Publicar una página HTML desde CentOS y acceder a ella desde Windows.

### Paso 1. Instalar Apache HTTP Server
```bash
sudo dnf install -y httpd
```

### Paso 2. Verificar la versión
```bash
httpd -v
```

### Paso 3. Consultar el estado inicial
```bash
sudo systemctl status httpd
```
Si aparece `inactive (dead)`, el servicio está detenido; no significa necesariamente que esté deshabilitado para el arranque.

### Paso 4. Iniciar el servidor y habilitarlo al arrancar
```bash
sudo systemctl enable --now httpd
sudo systemctl status httpd
```
Debe aparecer `active (running)`.

![Evidencia del servicio HTTPD](https://github.com/user-attachments/assets/9f0b03e8-c271-407e-b239-7a15e7283548)

### Paso 5. Probar desde CentOS
Abrir `http://127.0.0.1` en el navegador de CentOS. También se puede probar desde la terminal:
```bash
curl -I http://127.0.0.1
```

### Paso 6. Revisar el firewall
```bash
sudo firewall-cmd --state
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --list-all
```
Si se utiliza una zona distinta de la predeterminada, ajustar los comandos siguientes a la zona asignada a la interfaz de red.

### Paso 7. Permitir HTTP
```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
```
Comprobar que aparezca `http` entre los servicios permitidos.

![Evidencia del firewall](https://github.com/user-attachments/assets/ceff4c03-b57e-48ea-856f-d7858db1f425)

### Paso 8. Crear o editar la página principal
La ubicación habitual del contenido público es `/var/www/html/`, no la página de bienvenida de `/usr/share/httpd/noindex/`.
```bash
sudo nano /var/www/html/index.html
```
Ejemplo de contenido:
```html
<!DOCTYPE html>
<html lang="es">
<head><meta charset="UTF-8"><title>Mi servidor HTTPD</title></head>
<body><h1>Bienvenidos a mi servidor CentOS</h1><p>Práctica de Francisco y Esteban.</p></body>
</html>
```
Guardar en nano con `Ctrl+O`, `Enter`, `Ctrl+X`. Si ya existe una página creada en Visual Studio Code, copiar su `index.html` y sus recursos al directorio web, respetando las rutas.

### Paso 9. Consultar la IP de CentOS y probar desde Windows
```bash
ip -4 a
```
Desde Chrome en Windows, abrir `http://IP_DE_CENTOS` (sustituir por la IP actual). La máquina virtual debe tener conectividad con Windows: por ejemplo, red puente o red solo-anfitrión; con NAT puede requerirse redirección del puerto 80 en VirtualBox. No confundir `127.0.0.1` de CentOS con el de Windows.

### Paso 10. Verificación final
```bash
sudo apachectl configtest
sudo systemctl status httpd
```
**Evidencias sugeridas:** instalación, servicio activo, regla de firewall, archivo `index.html`, navegador en CentOS y navegador en Windows mostrando la página.
