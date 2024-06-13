# Writeup - Tomcat Takeover from CyberDefenders
 
Herramienta: Wiresahark
Autor: Miquel Navarro
Dificultad: Facil
Categoria: Network Forensics

### Q1. Given the suspicious activity detected on the web server, the pcap analysis shows a series of requests across various ports, suggesting a potential scanning behavior. Can you identify the source IP address responsible for initiating these requests on our server?

Para ubicar que posible IP es la que ha hecho el escaneo de puertos vamos a "Statistics > Endpoints".
Desde aquí, iremos a "IPv4" donde vemos que las IPs 10.0.0.112 y la 14.0.0.120 han tenido mucha interacción.

![img 1](https://github.com/miqueln4/CTF/edit/main/Tomcat%20Takeover/_images/cap_1.png)

Seguramente es la 14.0.0.120 la IP del atacante así que filtramos con "ip.src == 14.0.0.120" para verificar que se ha realizado el esxaneo de puertos puesto que se puede ver que el tiempo entre peticiones es muy breve.

![img 2](https://github.com/miqueln4/CTF/edit/main/Tomcat%20Takeover/_images/cap_2.png)

### Q2. Based on the identified IP address associated with the attacker, can you ascertain the city from which the attacker's activities originated?

Para ver la ciudad donde se encuentra el atacante, utilizamos cualquier geolocalizador de IPs online.

![img 3](https://github.com/miqueln4/CTF/edit/main/Tomcat%20Takeover/_images/cap_3.png)

### Q3. From the pcap analysis, multiple open ports were detected as a result of the attacker's activitie scan. Which of these ports provides access to the web server admin panel?

Para saber que puerto nos proporciona acceso al admin panel, filtramos por "ip.dst == 14.0.0.120" de esta forma sabemos que le esta ofreciendo respuesta a sus peticiones.
Desde una de las respuesta con OK por parte del protocolo HTTP hacemos click derecho sobre el y entramos a "Follow > TCP Stream".

![img 4](https://github.com/miqueln4/CTF/edit/main/Tomcat%20Takeover/_images/cap_4.png)

Aquí encontraremos que la conexión se realiza por el puerto 8080. 

![img 5](https://github.com/miqueln4/CTF/edit/main/Tomcat%20Takeover/_images/cap_5.png)

### Q4. Following the discovery of open ports on our server, it appears that the attacker attempted to enumerate and uncover directories and files on our web server. Which tools can you identify from the analysis that assisted the attacker in this enumeration process?

Si seguimos navegando por el canal de la pregunta anterior, se encuentra cual es la herramienta que se ha utilizado para listar los directorios.

![img 6](https://github.com/miqueln4/CTF/edit/main/Tomcat%20Takeover/_images/cap_6.png)

### Q5. Subsequent to their efforts to enumerate directories on our web server, the attacker made numerous requests trying to identify administrative interfaces. Which specific directory associated with the admin panel was the attacker able to uncover?

Siguiendo el stream, vemos cual es el directorio al que atacan. También se puede ver filtrando por "http.authorization" para ver donde estan realizando los ataques de fuerza bruta.

![img 7](https://github.com/miqueln4/CTF/edit/main/Tomcat%20Takeover/_images/cap_7.png)

![img 8](https://github.com/miqueln4/CTF/edit/main/Tomcat%20Takeover/_images/cap_8.png)

### Q6. Upon accessing the admin panel, the attacker made attempts to brute-force the login credentials. From the data, can you identify the correct username and password combination that the attacker successfully used for authorization?

Aprovechando el filtrado anterior, se puede observar  en que frames se intenta realizar el ataque de fuerza bruta, así que realizando un filtrado nuevo entre los frames en los que vemos el ataque ("frame.number > 205333 and frame.number < 20625 and http"), podemos ver cuando consiguen acceso.
Analizando el paquete encontramos las credenciales utilizadas.

![img 9](https://github.com/miqueln4/CTF/edit/main/Tomcat%20Takeover/_images/cap_9.png)

### Q7. Once inside the admin panel, the attacker attempted to upload a file with the intent of establishing a reverse shell. Can you identify the name of this malicious file from the captured data?

Si seguimos el paquete tcp, podemos ver que se ha subido un archivo.

![img 10](https://github.com/miqueln4/CTF/edit/main/Tomcat%20Takeover/_images/cap_10.png)

### Q8. Upon successfully establishing a reverse shell on our server, the attacker aimed to ensure persistence on the compromised machine. From the analysis, can you determine the specific command they are scheduled to run to maintain their presence?

Una vez subido el archivo, si seguimos el "TCP Stream" y justo el siguiente frame tenemos el comando que han programado para obtener acceso y tener persistencia en el equipo.

![img 11](https://github.com/miqueln4/CTF/edit/main/Tomcat%20Takeover/_images/cap_11.png)
