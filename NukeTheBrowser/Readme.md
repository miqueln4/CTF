# Writeup - NukeTheBrowser from CyberDefenders
 
Herramientas: Wiresahark, Network Miner, Cyberchef, scdbg y Javascript Playground

Autor: Miquel Navarro

Dificultad: Difícil

Categoria: Network Forensics

### Q1. Multiple systems were targeted. Provide the IP address of the highest one.

En primer lugar, abriremos la captura .pcap con la aplicación Wireshark.
Accedemos a "Statistics > Conversations" abrimos el apartado de "IPv4" y ordenamos de manera descendente la lista "Address A".

![img 1](_images/cap_1.png)

### Q2. What protocol do you think the attack was carried over?

Para saber en que protocolo han realizado el ataque accedemos a "Statistics > Protocol Hierarchy Statistics" y podemos observar que se ha utilizado bastante el protocolo HTTP por lo que es probable que sea ese.

![img 2](_images/cap_2.png)

### Q3. What was the URL for the page used to serve malicious executables (don't include URL parameters)?

Para encontrar la URL, siguiendo los paquetes en orden en Wireshark, en el paquete 178 observamos un GET sospechoso que resulta ser la página donde se enceuntran los ejecutables maliciosos.

![img 3](_images/cap_3.png)

Si accedemos a "Follow > TCP Stream" podemos ver la página exacta de donde se descargan.

![img 4](_images/cap_4.png)

### Q4. What is the number of the packet that includes a redirect to the french version of Google and probably is an indicator for Geo-based targeting?

En este caso, filtraremos los paquetes con la opción "dns.qry.name == "www.google.fr"" ya que .fr nos indica que es perteneciente a Francia 

![img 5](_images/cap_5.png)

Nos fijamos en el número de los paquetes obtenidos y quitamos el filtro para ver los paquetes anteriores donde vemos que el paquete 299 es el que encuentra la página.

![img 6](_images/cap_6.png)

### Q5. What was the CMS used to generate the page 'shop.honeynet.sg/catalog/'? (Three words, space in between)

Filtramos los paquetes con "http.request.uri "/catalog/"" de forma que nos muestra el paquete exacto en el que tenemos dicha parte de la url.

![img 7](_images/cap_7.png)

Hacemos click derecho sobre el paquete y seleccionamos "Follow > HTTP Stream" de forma que podemos ver lo que ha ido sucediendo en el paquete HTTP. Siguiendo la "conversación" es donde observamos el CMS que se ha utilizado.

![img 8](_images/cap_8.png)

### Q6. What is the number of the packet that indicates that 'show.php' will not try to infect the same host twice?

Añadiendo "path" al filtro anterior podemos encontrar "/fg/show.php", esto lo sabemos puesto que lo hemos visto durante la investigación y para encontrar el paquete exacto de forma más rápida utilizamos dicho filtro.

![img 9](_images/cap_9.png)

Con este filtro vemos que hay dos paquetes con el mismo host, por lo que nos quedamos con los paquetes y seguimos sus conversaciones.

En primer lugar, el paquete 157 recibe respuesta en el paquete 174. Así que accedemos a "Follow > HTTP Stream" del paquete 174 y vemos que tiene un script con JavaScript ofuscado por lo que aquí realiza la infección.

![img 10](_images/cap_10.png)

![img 11](_images/cap_11.png)

Ahora cambiamos al paquete 358 el cual tiene la respuesta en el 366. Así que repetimos los pasos anteriores para ver si lo infecta de nuevo.

![img 12](_images/cap_12.png)

![img 13](_images/cap_13.png)

En este caso vemos que no encuentra la página así que podemos afirmar que solo infecta una vez a cada host.

### Q7. One of the exploits being served targets a vulnerability in "msdds.dll". Provide the corresponding CVE number.

Para encontrar el número CVE de la vulnerabilidad "msdds.dll" basta con realizar una búsqueda en nuestro navegador y lo obtendremos.

![img 14](_images/cap_14.png)

### Q8. What is the name of the executable being served via 'http://sploitme.com.cn/fg/load.php?e=8'?

Para saber esto lo que vamos a tener que hacer es desofuscar el JavaScript.
En primer lugar, buscaremos la primera respuesta que tenemos con OK en el protocolo HTTP siguiendo los paquetes, con un click derecho sobre el HTML seleccionamos "Show PAckeet Bytes".

![img 15](_images/cap_15.png)

Tras esto nos guardamos el archivo.

![img 16](_images/cap_16.png)

Ahora con una simple línea de comandos acotamos el archivo para desofuscarlo de forma correcta y lo guardamos con otro nombre.

![img 17](_images/cap_17.png)

Con el comando "js" ya lo tendremos desofuscado.

![img 18](_images/cap_18.png)

A continuación, vamos a la web de CyberChef y con la opción de "Jvascript Beautify" pasamos nuestro código y lo guardamos para poder leerlo en claro.

![img 19](_images/cap_19.png)

A partir de este punto, miramos las diferentes variables que nos encontramos y las pasamos por CyberChef con las opciones "Swap Endianness" y "From Hex". La variable que contiene el ejecutable que estamos buscando es "shellcode".

![img 20](_images/cap_20.png)

Nos guardamos la variable y nos cambiamos a una máquina Windows 10 donde tenemos la herramienta "scdbg" y le pasamos el archivo con las opciones "UnlimitedSteps" y "FindSc".

![img 21](_images/cap_21.png)

Por último, cuando termina la ejecución, obtenemos el nombre del ejecutable.

![img 22](_images/cap_22.png)

### Q9. One of the malicious files was first submitted for analysis on VirusTotal at 2010-02-17 11:02:35 and has an MD5 hash ending with '78873f791'. Provide the full MD5 hash.

Con la aplicación Network Miner obtenemos el MD5 entero del fichero malicioso.

![img 23](_images/cap_23.png)

### Q10. What is the name of the function that hosted the shellcode relevant to 'http://sploitme.com.cn/fg/load.php?e=3'?

Reutilizando el Javascript desofuscado anteriormente, volvemos a buscar entre las funciones con CyberChef y encontramos que la función que nos piden es "aolwinmap".

![img 24](_images/cap_24.png)

### Q11. Deobfuscate the JS at 'shop.honeynet.sg/catalog/' and provide the value of the 'click' parameter in the resulted URL.

Buscamos el paquete que contiene la url mencionada.

![img 25](_images/cap_25.png)

Con "Follow > HTTP Stream" buscamos el código JavaScript.

![img 26](_images/cap_26.png)

Con la herremienta online "Javascript Playground" le pasamod el código cambiando "document.write" por "console.log" y este nos saca el valor de "click".

![img 27](_images/cap_27.png)

### Q12. Deobfuscate the JS at 'rapidshare.com.eyu32.ru/login.php' and provide the value of the 'click' parameter in the resulted URL.

Buscamos el primer paquete que contenga la URL que se nos pide.

![img 28](_images/cap_28.png)

Como hemos hecho en el paso anterior lo pasamos a la herramienta "Javascript Playground" añadiendo "console.log" entre los parámetros "eval" y "function".

![img 29](_images/cap_29.png)

![img 30](_images/cap_30.png)

El recultado, lo pasaremos por CyberChef con la opción "URL Decode" el cual nos otorga el valor de "click" 

![img 31](_images/cap_31.png)

### Q13. What was the version of 'mingw-gcc' that compiled the malware?

Siguiendo el "HTTP Stream" del ejercicio anterior buscando por la palabro "ming" encontramos la versión.

![img 32](_images/cap_32.png)

### Q14. The shellcode used a native function inside 'urlmon.dll' to download files from the internet to the compromised host. What is the name of the function?

Pasandole el archivo de la pregunta 8 o el de la pregunta 10 por la herramienta "scdbg" obtenemos la función nativa dentro de "urlmon.dll" --> "URLDownloadToFile"

![img 33](_images/cap_33.png)
