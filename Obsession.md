# Indice
- Nmap
- Puerto 80/HTTP
- Hydra
- Puerto 22/SSH


# Reconocimiento
Comenzamos la maquina con un escaneo basico con `Nmap` para listar primeramente los puertos y despues analizar en detalle cada puerto descubierto


`Recuerda que algunos comandos requieren de el comando SUDO al inicio`
<p>
&nbsp;
</p>

- ### Nmap
<p>

```python
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2 -oG Allports
```
![Puertos](https://github.com/owl3r/Dockerlabs.es/assets/169026357/b0adef1f-9995-47af-bce7-db18046df3b7)

</p>

<p>
&nbsp;
</p>


````python
nmap -p21,22,80 -sCV 172.17.0.2 -oN Scanports
````
![image](https://github.com/owl3r/Dockerlabs.es/assets/169026357/13b19b07-9041-4fb1-a32f-a95c422d9148)
<p></p>

- ### Reconocimiento manual
### Puerto 80

Nos dirijimos al navegador para indagar un poco que hay alojado en el puerto `80/http`,
Encontramos una pagina web de fitness que como datos interesantes contiene un formulario de asesoria y un email de contacto un poco mas abajo

![image](https://github.com/owl3r/Dockerlabs.es/assets/169026357/72adfc12-8670-42dd-87b5-43e11fb7cf3b)

<p>
&nbsp;
</p>

Realizamos un `Ctrl+U` para inspeccionar el codigo de la pagina en el cual encontramos varios datos de interes:

	> Un enlace a una pagina web
	> Una pista sobre el usuario de acceso a la pagina
	> El mail anteriormente mencionado

![image](https://github.com/owl3r/Dockerlabs.es/assets/169026357/dda56e43-4c1c-4205-a5b7-8c505d6a7967)
![iamge](https://github.com/owl3r/Dockerlabs.es/assets/169026357/f93d265a-4eec-45e9-9470-8dbf6394676d)
![iamge](https://github.com/owl3r/Dockerlabs.es/assets/169026357/8b727376-a999-4724-a6b2-73f9dcf9a65d)

# Explotacion
- ### Hydra
### Puerto 22
Como anteriormente hemos detectado que el puerto 22 estaba abierto utilizaremos `hydra` con el nombre de usuario descubierto para intentar acceder a la maquina victima

````python
hydra -l russoski -P /usr/share/wordlist/rockyou.txt -I -f ssh://172.17.0.2 -t 4
````
Detalles
- `-l` indicar usuario
- `-P` indicar biblioteca de contraseñas
- `-I` En caso de interrupcion volver al lugar del proceso donde se encontraba anteriormente
- `-f` Finalizar proceso en cuanto encuentre la contraseña valida
- `-t` Añadir hilos en paralelo al proceso para agilizar el ataque

<p></p>

![image](https://github.com/owl3r/Dockerlabs.es/assets/169026357/b0fe70cc-4605-4bea-94c7-5d0062df77ce)

Ahora ya tenemos usuario y contraseña

  >Usuario: Russoski <p></p>
  >Contraseña: iloveme

<p>
&nbsp;
</p>
Una vez descubierta la contraseña procedemos a conectarnos a traves de SSH
Estando dentro de la maquina procedemos a listar todos los directorios y nos encontramos con que .bash_history contiene datos, echamos un vistazo a ver que nos encontramos
<p></p>

![datosbashhistoy](https://github.com/owl3r/Dockerlabs.es/assets/169026357/9244a385-41ce-4f68-b67a-353a6f9c7198)
![directoriocontraseña](https://github.com/owl3r/Dockerlabs.es/assets/169026357/04ef30e9-31c1-4082-8507-9f9ddd1aac75)

<p></p>

En el recorrido que ha hecho russoski por la terminal vemos que a accedido al directorio `/var/www/html/important` en el cual se encuentra un archivo de texto que parece de importancia.
<p></p>

![contraseña](https://github.com/owl3r/Dockerlabs.es/assets/169026357/f7d61c14-3518-4ddd-89ca-81d2d0d4b66b)

Por lo visto es un borrador de un mensaje que ha enviado a el usuario Anuar en el cual indica cual es la contraseña de usuario root. En este caso esta hasheada en MD5.
La pasamos por un dehasher y la contraseña que nos arroja es : fucker

![root](https://github.com/owl3r/Dockerlabs.es/assets/169026357/d121e157-73ef-4f87-b3dc-75f007b40699)

Probamos a acceder a root con esa contraseña y nos da positivo ¡ESTAMOS DENTRO!
