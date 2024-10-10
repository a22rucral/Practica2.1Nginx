# Práctica 2.1 - Instalación y configuración de servidor Nginx. 
## Instalación servidor web Nginx.
El primer paso a seguir en esta práctica, es la instalación de nginx en nuestra máquina Debian. Para ello primero deberemos actualizar los repositorios con el comando:  

![Comando1](imagens/1comandoSudoAptUpdate.png)

Y luego instalaremos el paquete de Nginx. 
```
sudo apt install nginx
```

Si queremos comprobar que se ha instalado correctamente, ejecutaremos el siguiente comando:
![Comando2](imagens/2comandoStatusNginx.png)




## Creación de la carpeta del sitio web. 
Todos los archivos que formarán parte de un sitio web se organizarán en carpetas. Estas suelen estar en **/var/www**.
Por ello crearemos la carpeta con el nombre de nuestro sitio web; 

![Comando3](imagens/3comandoCreacionCarpetaWeb.png)

Una vez creada la carpeta, deberemos clonar este repositorio de github dentro de la misma ; 
https://github.com/cloudacademy/static-website-example

Es necesario que si no teneis instalado git en vuestra máquina que lo instaleis con el siguiente comando: 
![Comando4](imagens/4comandoInstalarGit.png)

Y para la clonación usarás el siguiente: 
![Comando5](imagens/5comandoClonacionGit.png)

Ahora, pondremos a **www-data** como propietario de esta carpeta y de todo lo que haya dentro. 
![Comando6](imagens/6comandoHacerPropietario.png)

Le daremos los permisos necesarios para evitar errores de acceso al entrar en el sitio web: 
![Comando7](imagens/7comandoObtenciónPermisos.png)

Para comprobar que todo funciona, podemos acceder desde nuestro cliente al servidor. Primero obtendremos la ip con el siguiente comando; 
![Comando9](imagens/9comandoObtencionIpMAquina.png)

Solo queda introducirlo en el navegador y si todo va bien tendremos el siguiente resultado. 
![Comando8](imagens/8comandoComprobacionServidor.png)


## Configuración de servidor web NGINX.
En Nginx hay dos rutas importantes. Una de ellas es **sites-available**, esta carpeta contiene los archivos de configuración de cada uno de los sitios web que alberga el servidor, la otra es **sites-enabled**, contiene los archivos de configuración de los sitios habilitados, los que funcionan en ese momento. 

EN **sites-available** hay un archivo default que es la página que muestra si entramos al servidor sin ningún sitio web. Para que nginx muestre el contenido de nuestra web, necesitaremos crear un bloque de servidor con las directivas correctas. Para ello crearemos un archivo nuevo en /etc/nginx/sites-available/nuestro-dominio: 

![Comando11](imagens/11comandoEntrarSitestAvaliable.png)
Y lo modificaremos de la siguiente forma: 
![Comando10](imagens/10comandoConfSitesAvaliable.png)

Ruta del index ; 
![Comando12](imagens/12comandoCarpetaGitDesc.png)

Entre el index y los archivos, crearemos un archivo simbólico de los sitios que están habilitados, para que se de de alta automáticamente.
![Comando13](imagens/13comandoArchivoSitesEnable.png)

Reiniciaremos el servidor para aplicar todos los cambios hechos: 
![Comando14](imagens/14comandoRestartNginx.png)

## Comprobaciones. 
Al no poseer servidor DNS que traduzca los nombres a IPs, debemos hacerlo manualmente. Editaremos el archivo /etc/hosts de nuestra máquina anfitriona para que asocie la IP de la máquina virtual a nuestro nombre del servidor. 
Este archivo en Windows se encuentra en el siguiente directorio : 
C:\Windows\System32\drivers\etc\hosts : 

![Comando15](imagens/15comandoAsignarIpaDominio.png)

Si queremos comprobar las peticiones, podemos hacerlo gracias a un archivo situado en /var/log/nginx/acces.log. Este como el ejemplo nos muestra, registra todas las peticiones. 

![Comando16](imagens/16comandoComprobacionPeticiones.png)

## Configurar servidor SFTP en Debian.
Para transferir archivos de nuestra máquina local a nuestra máquina virtual, aunque hay métodos mejores y más modernos, en este caso utilizaremos ftp/sftp. Es un protocolo de transferencia de archivos entre sistemas conectados a una red TCP. La diferencia entre estos, esque debido a la inseguridad de FTP, se le añadió una capa SSH para hacer SFTP y darle más seguridad. 
En primer lugar instalaremos desde los repositorios: 
![Comando17](imagens/17comandoUpdate.png)
![Comando18](imagens/18comandoInstallVsftpd.png)

Crearemos una carpeta en nuestro home en debian. 
![Comando19](imagens/19comandoCreacionCarpta.png)

Iremos a la configuración vsftpd, le indicaremos que este será el directorio al cual vsftpd se cambie después de conectarse el usuario. 
Ahora crearemos los certificados de seguridad necesarios para aportar la capa de cifrado a nuestra conexión. 
![Comando20](imagens/20comandoCertificadosSeguridad.png)

Una vez terminado, cambiamos la configuración de vsftpd con el editor de textos. 
![Comando21](imagens/21comandoConfVsftpd.png)

Bucaremos las siguientes líneas; 
```
rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
ssl_enable=NO
```
Y las sustituiremos por estas. 
![Comando22](imagens/22comandoModificacionArchivoConf.png)

Tras guardar la configuración, reiniciaremos el servicio. 
![Comando23](imagens/23ComandoReiniciar.png)

### Cliente SFTP 
Link descarga : https://filezilla-project.org/download.php?type=client

Utilizaremos SFTP por su seguridad, 
Indicaremos la ip del servidor, el nombre de usuario, contraseña y el puerto 22, no el 21 en el caso de la captura. 

![Comando24](imagens/24comandoParametroFilezilla.png)

Aceptamos el certificado desconocido, ya que al ser nuestro servidor no hay peligro. 

![Comando25](imagens/25comandoAceptaCertificadoDesconocido.png)

Tras esto comprobaremos que la conexión se ha realizado con éxito. 
![Comando26](imagens/26comandoConexionServidor.png)
Probaremos a subir un archivo desde nuestra máquina local a la virtual. Comprobando que ha sido satisfactoria. 
![Comando27](imagens/27comandoConfirmaciónSubida.png)
![Comando28](imagens/28comandoTransaccionSatisfactoria.png)

## HTTPS 
En este apartado, añadiremos a nuestro servidor una capa de seguridad. Haremos que todos nuestros sitios web alojados hagan uso de los certificados SSL y se acceda a ellos por medio de HTTPS.

Para ello, necesitaremos generar unos certificados autofirados, y cambiar los parametros necesarios en el fichero de configuración de nuestros host virtuales. 

Como primer paso, instalaremos OpenSSL mediante los siguientes comandos; 
![Comando29](imagens/29comandoUpdateInstallSSL.png)

Crearemos el certificado SSL autofirmado. Explicación de comando; 
    
    - x509 : Indica que se creará un certificado autofirmado 
    - nodes : No se cifrará la clave privada con una contraseña
    - days 365 : Será válido por 365 días 
    - newkey rsa:2048 Creará una clave RSA de 2048 bits
    - keyout servidor.key : Especifica el archivo donde se guardará la clave privada. 
    - out servidor.crt :  Especifica el archivo donde se guardará el certificado.
![Comando30](imagens/30comandoCreaciónYConfi.png)

Editaremos el archivo del sitio web en Nginx: 
```
$sudo nano /etc/nginx/sites-available/tu_servidor.com
```
Configuraremos el bloque de servidor para HTTPS:
Añadiremos un bloque de servidor para el puerto 443 (HTTPS). También añadiremos un bloque para redireccionar HTTP a HTTPS para que redirigir las solicitudes HTTP al puerto 443. 
![Comando31](imagens/31comandoEdiciónArchivoConf.png)

En caso de que no lo hayas hecho, debemos habilitar el sitio web con el siguiente comando:
```
sudo ln -s /etc/nginx/sites-available/tu_servidor.com /etc/nginx/sites-enabled/
```

Y por ultimo, para comprobar la configuración de Nginx ejecuta el siguiente comando: 
![Comando32](imagens/32comandoComprobaciónConf.png)


## Resultado 
Si todo ha salido bien, al buscar tu dominio en el navegador debe darte esta salida: 
![Comando33](imagens/33comandoVisitaHTTPS.png)


### Cuestiones finales 
1. ¿Que pasa si no hago el link simbólico entre sites-available y sites-enabled de mi sitio web?
   
   Si no hacemos el enlace simbólico, nginx no activará la configuración del sitio web. **Sites-avaliable** solamente guarda los archivos, nginx, usa las configuraciones que están en **sites-enabled**. 
2. ¿Qué pasa si no le doy los permisos adecuados a /var/www/nombre_web?
   
   Que no se podrá acceder desde ningún servidor a los archivos del sitio.