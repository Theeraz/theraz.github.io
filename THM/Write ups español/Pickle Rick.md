# Pickle Rick WriteUp

He de decir que este es mi primer write up, sé que tengo mucho por mejorar y que las imágenes no son de la mejor calidad posible. Pienso esforzarme para mejorar la calidad de estos a medida que los vaya haciendo. Y ahora...¡al lío!  

Nos facilitan la ip --> 10.10.171.63  
Hago uso del comando export IP=10.10.171.63 para asignarle una variable y hacerlo más cómodo.  
Uso nmap en busca de puertos abiertos, uso el siguiente comando: 
`nmap -sC -sV 10.10.171.63`  

![PortScan](https://github.com/Theeraz/theraz.github.io/assets/90190970/eb92a7c1-e126-42c9-9117-74d225ee6b48)  

Los puertos 22 (ssh) y 80 (tcp) están abiertos.  
Mientras el escáner actúa busco en el source code ``(ctr+U)`` de la página y encuentro un username.  

![SourceCode](https://github.com/Theeraz/theraz.github.io/assets/90190970/d5604384-f6a9-40db-ac74-c7e5c18e0aad)  

Decido probar con Hydra para ver si puedo acceder mediante fuerza bruta pero antes me aseguro de que tengo el diccionario rockyou con el comando locate:  

![LocateRockyou](https://github.com/Theeraz/theraz.github.io/assets/90190970/1419ac6c-ae51-4f97-9baf-37fc93eeb63c)  

Me muevo al directorio en el que se encuentra el diccionario y procedo a insertar el comando de Hydra desde ahí:  

![DirectorioHydra](https://github.com/Theeraz/theraz.github.io/assets/90190970/10597dd9-45d9-45ea-b9e9-54ddbdc387ba)  

No sirve de nada, así que busco información sobre qué hacer y decido usar Gobuster que es una herramienta para usar fuerza bruta en directorios y archivos almacenados en una página web. Busco `common.txt` con el comando `locate` para usarlo cómo diccionario.  
Pruebo el siguiente comando:  
`gobuster dir -u http://10.10.171.63 -w /usr/share/wordlists/dirb/common.txt 2> /dev/null`  

![gobuster](https://github.com/Theeraz/theraz.github.io/assets/90190970/4618971c-dbfc-4232-83bd-5d7c19f2c526)  

Hay varios directorios a los que pruebo a acceder mediante la página web. Pruebo con robots.txt:  

![Wubbalubba](https://github.com/Theeraz/theraz.github.io/assets/90190970/59b44a8d-7f04-45c5-bf7e-564bdc763cea)  

Esto es lo que encuentro, quizás sea un credencial, decido guardarlo ya es posible que me sirva en algún momento. Miro en el directorio assets:  

![Assets](https://github.com/Theeraz/theraz.github.io/assets/90190970/e8db56f2-ddd7-498e-9847-c977a4f6d3e4)  

No encuentro nada que me sirva así que uso Gobuster con otro diccionario enfocado a directorios contenidos en páginas web. El nombre de este es ``directory-list-lowercase-2.3-medium.txt``.
Pruebo el comando anteriormente usado pero con este diccionario:  
`gobuster dir -u http://10.10.171.63 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt`  

![Gobuster2](https://github.com/Theeraz/theraz.github.io/assets/90190970/c75be516-04f3-4056-9774-7277126c723b)  

Han pasado 10 minutos y todavía no ha terminado, así que me pongo a trastear con nmap. Descubro que puedo usar scripts del propio nmap y pruebo con el puerto 80 a usar un script que me enumere los directorios asignados a ese puerto con un script http-enum:  

![Enum](https://github.com/Theeraz/theraz.github.io/assets/90190970/4736966a-8cc7-4015-b001-c9f1be2430bd)  

Ha sido mucho más rápido y me ha localizado lo que parece ser el login de la página web. Pruebo a acceder mediante el username del código fuente de la página y la palabra que encontré en el robot.txt de antes y efectivamente gano acceso con el usuario ``R1ckRul3s`` y la contraseña ``Wubbalubbadubdub``.  
Lo primero que me encuentro es un panel de comandos, trato de acceder a los demás directorios pero son inaccesibles, intuyo que sólo el admin puede acceder a ellos. Vuelvo al panel de comandos y uso `ls`:  

![Ls](https://github.com/Theeraz/theraz.github.io/assets/90190970/aa251b59-5959-4068-958a-b89451f3ca85)  

Trato de mirar el contenido de lo que parece un archivo que me dará una flag (Sup3rS3cretPickl3Tngred.txt) pero el comando ``cat`` está deshabilitado. Pruebo con el comando ``less`` y encuentro la primera flag:  

![Flag1](https://github.com/Theeraz/theraz.github.io/assets/90190970/13be8e19-ad7f-4a58-814c-00ce9d0da8a1)  

Decido hacer lo mismo con clue.txt:  

![Clue](https://github.com/Theeraz/theraz.github.io/assets/90190970/98f27d04-0128-4fcc-b89f-2e433441c238)  

Pruebo a navegar entre directorios y no me deja. Debo hacer one liners para conseguir la información que necesito. Pruebo con "ls ../../../home/" (retrocedo 3 veces ya que en el pwd salgo en /var/www/html y sólo puedo moverme ahí). En home encuentro 2 directorios:  

![Directorios](https://github.com/Theeraz/theraz.github.io/assets/90190970/44c8586a-43a8-40ae-95e8-50ad07e96a1d)  

Pruebo a hacer lo mismo, esta vez entrando en el directorio "Rick". Encuentro un archivo que parece que me dirá la 2º flag. No encuentro el modo de acceder a ese archivo.  
Parece que la única manera de conseguir acceso es mediante una reverse shell. Encuentro una cheat sheet en ironhackers, me dispongo a probar esos comandos:  

![Revershell](https://github.com/Theeraz/theraz.github.io/assets/90190970/d40c81c6-6b53-4e42-9bee-0d9926dbcefe)    
![Notwork](https://github.com/Theeraz/theraz.github.io/assets/90190970/4eb1c3c4-7071-463e-9f51-9a6f6153ede8)  

No parece funcionar. Voy a intentarlo con una shell diferente de la misma cheat sheet de ironhackers. Esta vez lo intento con Perl, eligiendo el puerto 4444:  

![Works](https://github.com/Theeraz/theraz.github.io/assets/90190970/20decef1-1b6d-462a-b1b5-9a851e7f3e58)  
![Works2](https://github.com/Theeraz/theraz.github.io/assets/90190970/95282c98-b037-4747-be14-f98f292c182a)  

Funciona, ya puedo moverme por el sistema sin limitación de comandos. Accedo al directorio que encontré en la propia web mediante comandos y consigo la 2º flag:  

![Flag2](https://github.com/Theeraz/theraz.github.io/assets/90190970/9655e461-7398-4c12-bb0d-ee978c79c023)

Para buscar la última flag, decido probar a escalar a root con "sudo su". Y...¡funciona! Así que miro en la carpeta "/root" y encuentro un archivo que parece ser la 3º flag:  

![Flag3](https://github.com/Theeraz/theraz.github.io/assets/90190970/3a885640-bfce-429d-8f94-a3817f10e578)  
![Flag3'1](https://github.com/Theeraz/theraz.github.io/assets/90190970/9d6aa74f-8030-4086-a841-b02b27016a9e)  

Con esto consigo obtener todas las flags de la máquina.

Recursos: https://ironhackers.es/herramientas/reverse-shell-cheat-sheet/
