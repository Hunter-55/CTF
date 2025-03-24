Primero Obtenemos la IP de nuestra maquina atacante:
```
ifconfig
```

Utilizando " netdiscover " procedemos a encontrar los equipos conectados en la red:
```
sudo netdiscover -r 192.168.0.0/24
```
![[/imagenes/ip_victima.png]]

Una ves encontrada la IP de la maquina victima comenzamos a realizar un escaneo de puertos, utilizaremos " nmap ":
```
sudo nmap -sV -A -T5 192.168.0.101
```
![[escaneo_nmap.png.png]]

Encontramos 2 puertos abiertos:
- El puerto 22 que corresponde al ssh con una versiuon "OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)" .
- El puerto 80 que corresponde a un servidor web con una version "Apache httpd 2.4.38 ((Debian))".

Revisamos la paguina web, ya que nos marca que tiene activo un servicio web y no nos deja entrar pero nos manda un nombre de dominio que debemos agregar a nuestro archivo hosts para acceder.
- http://192.168.0.101 -> http://deathnote.vuln/wordpress/
```
	sudo nano /etc/hosts/
```
![[hosts.png]]

Volvemos a revisar la paguina y nos muestra un fondo del tema del famoso anime llamado "death note".
Revisamos el codigo de la paguina y nada inusual, pero al entrar al botón de "HINT" nos muestra un mensaje que nos da una pista:
![[mensaje_web.png]]

Buscando algun comentario vi una linea que llamo mi atención en la paguina que dice "my fav line is iamjustic3". 
![[linea_fav.png]]

Si conoces la serie o si análisas bien la paguina te daras cuenta que kira es el pricipal por lo que si realizas una enumeración de directorios sia dirb, dirsearch o wpscan, podras ver los directorios del servidor.
```
	dirsearch -u http://deathnote.vuln/wordpress/
```

Como sabemos el servidor web utiliza wordpress por lo que entrando a "http://deathnote.vuln/wordpress/wp-admin/" nos pediras credenciales de acceso, esto lo podemos tener mediante el nombre del personaje principal y el cual se ve mucho en la misma paguina "kira"
y la contraseña se encuentra en la nota que encontramos en la paguina:
- USERNAME: kira
- PASSWORK: iamjustic3
![[credenciales_login.png]]

Nos mostrara el panel de wordpress:
![[wordpress_admin.png]]

Revisando en el apartado de media encontramos la nota que nos mencionaban al principio "note.txt".
![[note.png]]

Vemos el contenido de "note.txt".
![[note_conte.png]]

Revisando la url "http://deathnote.vuln/wordpress/wp-content/uploads/2021/07/notes.txt"
podemos revisar los archivos contenidos en directorio "/07" y encontramos "http://deathnote.vuln/wordpress/wp-content/uploads/2021/07/user.txt" este archivo "user.txt".
![[user_conte.png]]

Revisamos si encontramos algo en "http://deathnote.vuln/robots.txt" y encontramos una nota.
![[robots_txt.png]]

En la nota menciona un archivo "important.jpg" en cual al realizarle un curl nos muestra lo siguiente.
```
curl http://deathnote.vuln/important.jpg
```
![[curl_result.png]]

Eso nos indica que para obtener acceso por ssh debemos encontrar el usuario y contraseña.
podemos utilizar hydra para poder conectarnos. 
Descargamos los archivos txt de note.txt y user.txt.
```
wget http://deathnote.vuln/wordpress/wp-content/uploads/2021/07/notes.txt
```

```
wget http://deathnote.vuln/wordpress/wp-content/uploads/2021/07/user.txt 
```

```
hydra -L user.txt -P notes.txt ssh://192.168.0.101
```
![[coneccion_ssh.png]]

La contraseña del ssh son.
- login: l   
- password: death4me
![[ssh_login.png]]

Buscamos en la siguiente ruta "/opt/L/fake-notebook-rule" y encontramos dos archivos:
- case.wav
- hint
```
use cyberchef
```
![[ssh_1.png]]
![[cyberchef.png]]

Utilizaremos la herramienta de cyberchef y decodificamos: 63 47 46 7a 63 33 64 6b 49 44 6f 67 61 32 6c 79 59 57 6c 7a 5a 58 5a 70 62 43 41 3d.
![[hex_string.png]]

El resultado es: passwd : kiraisevil 

Revisando las demás carpetas podemos observar que la carpeta kira y L.
Al ya conocer la credencial de kira podemos iniciar sesión al ssh con kira
```
ssh kira@192.168.0.101
```

Al abrir el archivi kira.txt vemos:
```
cat kira.txt
```
cGxlYXNlIHByb3RlY3Qgb25lIG9mIHRoZSBmb2xsb3dpbmcgCjEuIEwgKC9vcHQpCjIuIE1pc2EgKC92YXIp
![[kita_txt.png]]

Nos encontrmos con este resultado:
- please protect one of the following 
	- 1. L (/opt)
	- 2. Misa (/var)

En la ruta del home encontramos un archivo user.txt con un mensaje codifiucado en Brainfuck.
```
++++++++++[>+>+++>+++++++>++++++++++<<<<-]>>>>+++++.<<++.>>+++++++++++.------------.+.+++++.---.<<.>>++++++++++.<<.>>--------------.++++++++.+++++.<<.>>.------------.---.<<.>>++++++++++++++.-----------.---.+++++++..<<.++++++++++++.------------.>>----------.+++++++++++++++++++.-.<<.>>+++++.----------.++++++.<<.>>++.--------.-.++++++.<<.>>------------------.+++.<<.>>----.+.++++++++++.-------.<<.>>+++++++++++++++.-----.<<.>>----.--.+++..<<.>>+.--------.<<.+++++++++++++.>>++++++.--.+++++++++.-----------------.
```

Utilizando https://www.dcode.fr/brainfuck-language decodificamos el mensaje que dice:
"i think u got the shell , but you wont be able to kill me -kira".

Al logearse como root buscamos la carpeta root y encontramos la flag.
![[root_.flag.png]]
