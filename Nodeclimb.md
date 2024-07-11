## Indice
- Nmap

- Puerto 21

- John

- Puerto 22

- Escalada de privilegios





`Recuerda que necesitas ejecutar los comandos con permisos SUDO`
# Reconocimiento

### Nmap

Comenzamos con un escaneo básico con Nmap para revelar los puertos abiertos y seguidamente escaneamos los puertos abiertos.
````python
nmap -p- -sS --open --min-rate 5000 -vvv -n -Pn 172.17.0.2 -oN ScanPorts
````
`-p-`Escanear todos los puertos

`-sS` Omitir conexion antes de finalizarla

`--open` Output solo de los puertos abiertos

`--min--rate` 5000 Envio de paquetes no mas lentos que 5000

`-vvv` Verbosidad

`-n` Evitar la resolución DNS

`-Pn` No realizar ping

`-oN` Allports Output tal cual copiado al archivo Allports

<p>
&nbsp;
</p>

````python
nmap -p21,22 -sC -sV 172.17.0.2 -oN ScanPorts
````

`-sC` Scripts básicos de reconocimiento de nmap

`-sV` Output de la versión del servicio que corre

`-p21,22` Indicación del puerto/s a escanear

`-oN` Scanports Output tal cual copiado al archivo Scanports

![scanports](https://github.com/owl3r/Dockerlabs.-ES/assets/169026357/3993a20e-dcdf-470b-b982-eb1d6de34e5d)

> En el escaneo de puertos individual podemos observar que el puerto 21 nos permite la conexión a través del usuario Anonymous y que además contiene un archivo comprimido `.zip`
-------

# Explotación

> Procedemos a conectarnos a la maquina victima a través del puerto 21 vía FTP.
````python
ftp 172.17.0.2
> Usuario: anonymous
> Password: No
````
<p>
&nbsp;
</p>

### John
-------
>Estando dentro de la maquina ejecutamos el comando `ls -la`, observamos que efectivamente contiene un archivo llamado `secretitopicaron.zip`
>Procedemos a descargarlo en nuestra maquina a través del comando `get` 

![GET](https://github.com/owl3r/Dockerlabs.-ES/assets/169026357/d56e70dc-9127-4595-a206-80d8b3e77e37)

>Una vez descargado el archivo y estando en nuestra maquina he intentado descomprimirlo con `unzip` pero me pide una contraseña para su descompresión.
>En este caso vamos a hashear el archivo con `zip2john` y siguiente paso será realizar un ataque de fuerza bruta con `johntheripper` para ya poder descomprimir el archivo.
>Los pasos a seguir serán los siguientes:

````python
zip2john secretitopicaron.zip > secret.hash
````
`>` Indicar nombre del hash
`secret.hash` Nombre del archivo

<p>
&nbsp;
</p>

````python
john --wordlist=/usr/share/wordlist/rockyou.txt secret.hash
````
`--wordlist=` Ruta del diccionario para el ataque

<p>
&nbsp;
</p>

![contraseña2](https://github.com/owl3r/Dockerlabs.-ES/assets/169026357/bf7d7594-23a9-42fe-ae90-4dea02a527e5)

````python
# Descomprimimos el archivo
> 7z x secretitopicaron.zip
````


- Contenido del archivo descomprimido

![contraseña](https://github.com/owl3r/Dockerlabs.-ES/assets/169026357/e4b62bb4-d28f-404b-8021-6224d3159f29)

--------
# Escalada de privilegios


> Una vez descomprimido el archivo observamos un posible usuario y una contraseña, nos conectaremos a la maquina victima a través del puerto 22 vía SSH.
> Dentro de la maquina ejecuto el comando `ls -la ` para visualizar que archivos contiene, en este caso contiene un archivo `script.js` pero esta vacío.
> Seguido ejecuto el comando `sudo -l` para listar si tengo algún tipo de privilegio SUDO
> En este caso recibo este output:

````bash
(ALL) NOPASSWD: /usr/bin/node /home/mario/script.js
````

<p>
&nbsp;
</p>

>Acudo a nuestra pagina de confianza GTFObins a ver si hay algo referente a node y encuentro lo siguiente

![gtfobins](https://github.com/owl3r/Dockerlabs.-ES/assets/169026357/c3c727ed-50ac-459a-acdd-5386a1d9aaf7)

>Según nos dice `sudo -l` necesitamos ejecutar el script también, con lo cual eliminamos la parte de `node -e` ya que es solo para NODE e introducimos el comando dentro del script con NANO.
>Ejecutamos
````python
sudo /usr/bin/node /home/mario/script.js
````

![escalada](https://github.com/owl3r/Dockerlabs.-ES/assets/169026357/86de7171-40a2-46ac-a331-0e2e942ecd8c)

- Ya somos usuario root.
