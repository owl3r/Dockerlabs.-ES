# Índice
- Nmap
- Puerto 80
- Fuzzing
- Port Knocking
- Hydra
- Escalada de privilegios



______
# Reconocimiento

### Nmap
________

> Procedemos a realizar el clásico escaneo básico con nmap.

````python
nmap -p- -sS --open --min-rate 5000 -vvv -n -Pn 172.17.0.2 -oN Allports
````


`-p-`Escanear todos los puertos

`-sS` Omitir conexion antes de finalizarla

`--open` Output solo de los puertos abiertos

`--min--rate` 5000 Envio de paquetes no mas lentos que 5000

`-vvv` Verbosidad

`-n` Evitar la resolucion DNS

`-Pn` No realizar ping

`-oN` Allports Output tal cual copiado al archivo Allports

<p>
&nbsp;
</p>

![Allports](https://github.com/owl3r/Dockerlabs.es/assets/169026357/08f642d1-69c6-4772-b19b-158f9a29157b)
<p>
&nbsp;
</p>
Realizamos un escaneo mas exhaustivo del puerto 80/HTTP

````python
nmap -sC -sV -p80 172.17.0.2 -oN Scanports
````
`-sC` Scripts basicos de reconocimiento de nmap

`-sV` Output de la version del servicio que corre

`-p80` Indicacion del puerto/s a escanear

`-oN` Scanports Output tal cual copiado al archivo Scanports
<p>
&nbsp;
</p>

![Scanports](https://github.com/owl3r/Dockerlabs.es/assets/169026357/f9b20bd1-7ef6-4d0a-9a20-dea7c2d8a296)

>Realizado ya el escaneo del puerto podemos ubicar la siguiente información de valor:
- El servicio que corre es Apache
- La versión es la 2.4.58
- Por el http-title podemos deducir que es la clásica pagina por defecto de apache


### Puerto 80/HTTP
_______
Vamos a conectarnos a la ip de la maquina victima con nuestro navegador para ver que tenemos en ese servicio web y como habíamos deducido tenemos la pagina por defecto de apache
Una buena costumbre cuando abrimos paginas web y analizar el código con `Ctrl+U` ya que a veces podemos encontrar información de valor. En este caso no aparece nada interesante.

![WEB1](https://github.com/owl3r/Dockerlabs.es/assets/169026357/504291b6-9f54-434f-ae59-2c3354d22bd1)

### Fuzzing
_____

Visto que hasta el momento toda la información que tenemos es nula ya que no nos sirve para nada, procedemos a realizar un fuzzeo de rutas con `gobuster` a ver si sacamos información de valor.

````python
gobuster dir -u 172.17.0.2 -w /usr/share/SecList/Discovery/Web-content/directory-list-2.3-medium.txt -t 200 -x html,php,txt 
````

`dir` Analisis de tipo direccion

`-u` Indicamos la URL

`-w` Indicamos la biblioteca que vamos a utilizar

`-t` Hilos

`-x` Indicar el tipo de extension

![Gobuster](https://github.com/owl3r/Dockerlabs.es/assets/169026357/40a39e07-7446-4f3f-8f46-3a5b5d6abf9b)

Realizado el análisis encontramos una ruta de interés `qdefense.txt` al acceder a ella nos encontramos con el siguiente mensaje
````
Recuerda llama antes de entrar, no seas como toctoc el maleducado
7000 8000 9000
busca y llama +54 2933574639
````

> Como datos de interés en este archivo podemos encontrar:
> - toctoc: Parece un nombre de usuario
> - 7000 8000 9000: Podrían ser puertos de la maquina victima

Indagando un poco por internet encontramos un tipo de técnica llamada `Portknocking` que concuerda bastante con la pista de lo que parecían 3 puertos indicados en el archivo anterior.
<p>
&nbsp;
</p>

# Explotacion

### Portknocking
_____
El `Port Knocking` (golpeo de puertos) **es un mecanismo que permite abrir puertos** a través de una **serie predefinida de intentos de conexión** a puertos que se encuentran cerrados. 
Es decir procedemos a un intento de conexión o llamada a los puertos de forma ordenada, de esta forma se "desbloqueara" el puerto en cuestión que este definido en la secuencia.
Para el Port Knocking utilizamos una herramienta llamada `knockd` , la sintaxis quedaría de la siguiente manera:

````python
knock -v 172.17.0.2 7000 8000 9000
````
`-v Vervosidad`

![knock](https://github.com/owl3r/Dockerlabs.es/assets/169026357/938f005d-557c-456d-a762-049962b2cc36)


Una vez utilizada la herramienta volveremos a pasar el escaneo con Nmap para descubrir que puerto a quedado abierto.

### Hydra
__________
En este caso el puerto descubierto es el `22/SSH` .
Como siguiente paso ya que tenemos un posible usuario `toctoc` vamos a tirar un ataque por fuerza bruta con `hydra` para descubir la contraseña, quedaria de la siguiente manera:

````python
hydra -l toctoc -P /usr/share/wordlist/rockyou.txt ssh://172.17.0.2 -t 64
````

`-l` Para indicar un usuario ya conocido

`-P` Diccionario a utilizar por desconocimiento de contraseña

`-t` Hilos

![contraseña](https://github.com/owl3r/Dockerlabs.es/assets/169026357/0f7ba8c4-d641-43fe-a5b9-e11d85ecfd04)

Conectamos a la maquina via SSH
````python
ssh toctoc@172.17.0.2
````
<p>
&nbsp;
</p>

# Escalada de privilegios

Una vez dentro de la maquina victima procedemos a ejecutar `sudo -l` para listar permisos como root.

![sudo-l](https://github.com/owl3r/Dockerlabs.es/assets/169026357/6ba21a79-6f93-4561-ae90-5dc578e9a3c0)

Como algo común podemos ejecutar el comando `sudo /opt/bash -p` para ejecutar una shell como root, pero nos dice que en el directorio /opt no contiene ningun archivo.

En busca de algo que nos sirva o sea de interes por el sistema encontramos el siguiente archivo `.bashrc` que contiene lo siguiente.
Realizaremos el mismo proceso que anteriormente con `knockd` y activamos de nuevo un puerto de entrada.

![puertasnuevas](https://github.com/owl3r/Dockerlabs.es/assets/169026357/ae6ed145-23ba-46fa-b57d-d184ec4b8d61)

Una vez repetida la operacion solo tenemos que ejecutar de nuevo el comando `sudo /opt/bash -p`  y de esta forma ya hemos escalado privilegios. COMPLETE

![root](https://github.com/owl3r/Dockerlabs.es/assets/169026357/21a3aa0c-1f13-4fc1-8c2d-f94216293570)

