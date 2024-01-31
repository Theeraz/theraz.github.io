# Wonderland WriteUp

Comienzo como usualmente hago, escaneo de puertos de nmap con los comandos que ya conozco y que suelen funcionarme:
`nmap -sC -sV 10.10.7.36`
- `-sC`: Para activar el escaneo de scripts y conseguir la máxima información posible de servicios, configuraciones específicas, etc
- `-sV`: Para detectar las versiones de los servicios en los puertos abiertos, esto ayuda a identificar vulnerabilidades específicas u obtener más información de los servicios en ejecución

![scan](https://github.com/Theeraz/theraz.github.io/assets/90190970/99c1cf9e-7c0f-4ee8-abbf-5f46c878c256)

 Tanto el puerto 22 cómo el 80 están abiertos, eso me da a entender que tendremos que usar ssh para comprometerla y encontrar las flags. Mientras tanto he mirado en la web por si acaso encontrase algo que me sirva de ayuda:  

 ![rabbit1](https://github.com/Theeraz/theraz.github.io/assets/90190970/a49f3583-d936-4f6a-9226-56c10537cc02)

Echo un vistazo en el código fuente de la página esperando encontrar algún username o password, pero no encuentro nada. Lo más lógico es usar gobuster para encontrar directorios de esta página en los que seguramente sí pueda encontrar alguna pista:  

 `gobuster dir -u http://10.10.7.36 -w /usr/share/wordlists/dirb/common.txt 2> /dev/null`
 - `gobuster dir`: Especifica que se realizará un escaneo de directorios
 - `-w /usr/share/wordlists/dirb/common.txt`: Especifica la lista de palabras para la fuerza bruta de directorios
 - `2> /dev/null`: Redirige los mensajes de error al dispositivo nulo, lo que significa que los mensajes de error no se mostrarán en la salida estándar

![Gobuster](https://github.com/Theeraz/theraz.github.io/assets/90190970/12515126-cb44-4a60-8f37-2f4582c7d916)  

 Encuentro 3 directorios. Accedo al primero (`/img`) en el que encuentro 3 imágenes y 2 de ellas son la misma pero les cambia el color y el formato, eso llama mi atención:  

 ![dir r](https://github.com/Theeraz/theraz.github.io/assets/90190970/4e9cb0be-00bf-4179-8c21-9374c60302f5)  

 
En el segundo `/index.html` encuentro el mismo directorio que ya visité anteriormente, en el que sale una frase (Follow the white rabbit) y la foto del conejo. Descargo la foto por si acaso contuviese alguna pista oculta.
  En el tercero `/r` encuentro otro directorio en el que sólo hay una frase, en ninguno de los 3 he encontrado nada en el source code:  

  ![f083eeebe66675c4814e215a7a213b26](https://github.com/Theeraz/theraz.github.io/assets/90190970/73d59fac-aa8b-4b5c-af71-4677913b9d77)  

  Decido usar otro diccionario con gobuster con la esperanza de que quizás me liste algún directorio el cuál no se haya listado antes:  
  `gobuster dir -u http://10.10.7.36 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt 2> /dev/null`  

  ![gobuster2](https://github.com/Theeraz/theraz.github.io/assets/90190970/09688e98-57e7-4295-8370-4609bbb7dfd4)

Encuentro varios directorios que antes no aparecieron, pero el único accesible es `/poem`. Investigo un poco el source code pero tampoco encuentro nada. Decido probar a hacerle esteganografía a la foto del conejo que encontré antes:
`steghide extract -sf "file.jpg"
- `extract`: Para extraer la información oculta
- `-sf`: Para indicar el archivo sobre el que queremos ejecutar el comando
Me pide una passphrase, pero simplemente presionando enter me da permiso para poder extraer la pista:

![58923ddaaa64af91cba3d4f86da9694f](https://github.com/Theeraz/theraz.github.io/assets/90190970/4728b5de-5ac6-49f3-a714-4386b4a4d4c5)  

Ahí reside una pista, la cuál creo que deja bien claro cuál es el siguiente paso, acceder al directorio /r/a/b/b/i/t:  

![438e1063e1c98e70b93c4d1c0016646c](https://github.com/Theeraz/theraz.github.io/assets/90190970/22a970bf-8d85-4ca0-a510-98d5e7595700)  

En ese directorio aparece una de las imágenes que encontré anteriormente con varias frases. Al inspeccionar el código fuente encuentro lo que parece ser un user/password:

![a3c79c3094d6ad12c3317eca6b6009ba](https://github.com/Theeraz/theraz.github.io/assets/90190970/e70404e4-329f-4c65-bfe2-93a4bff2bfca)  

Con esas credenciales debería poder conectarme a la máquina víctima por medio de SSH:  

![bc0f00922328428519d79b089cee4e40](https://github.com/Theeraz/theraz.github.io/assets/90190970/cf9aadfd-84a8-433f-b165-f25cbdaacb3f)

Estoy dentro. Miro los archivos que hay en el directorio actual y parece que hay una flag con el nombre `root.txt`. También hay un archivo `.py`. No tengo permisos para poder abrir `root.txt` pero sí puedo leer el archivo `.py`, que resulta ser otro poema del que no saco ninguna conclusión. Algo que llama mi atención es que en este directorio de user se encuentre un archivo de root, además de que la pista de la sala nos dice que "todo aquí está patas arriba" así que pruebo a buscar un archivo de user en el directorio de root: 

![2e39231c7454311d094efa93c9424e5f](https://github.com/Theeraz/theraz.github.io/assets/90190970/0c54ed1d-26e4-4fcb-a196-effa4e2c55d7)  

Bingo, ahí encuentro la primera flag. Intento escalar a sudo pero este usuario no tiene permiso. Lo siguiente es ver los permisos de los archivos del directorio user:  

![78a7791c798799c2880ab619fee3deac](https://github.com/Theeraz/theraz.github.io/assets/90190970/986c655c-7251-46ec-b184-e9a812607c47)  

Obviamente root.txt sólo puede ser leído por root, sin embargo, no le veo sentido a encontrar un archivo `.py` y que no se pueda ejecutar. Pruebo con `sudo -l` para ver privilegios:  

![f72bbc99a5272e4621222357cf86087a](https://github.com/Theeraz/theraz.github.io/assets/90190970/f6a08795-7041-4895-8b9f-7384d5448308)

Hay 2 datos importantes. El primero es que `rabbit` parece tener privilegios sobre el archivo `.py`. Lo segundo es que al hacer un `cat` de este archivo, la primera línea importa un módulo random. Después de buscar durante un largo rato información sobre posibles opciones, decido crear un módulo falso con el mismo nombre que actúe al ejecutar el archivo y que hará que al estar en el mismo directorio, se ejecute antes que el propio módulo random. Para ello habrá que darle permisos de ejecución con `chmod +x` y agregarle el código que queremos que este ejecute:  
`vim random.py`  

![e3cefbe29474fb671133c664d5461fa9](https://github.com/Theeraz/theraz.github.io/assets/90190970/5a41aaf9-0d6b-4295-9012-21a422867aa5)  

Con esta línea de código podremos ejecutar una shell de bash de un script de Python. Guardo con `Esc + :x` y lo siguiente es ejecutar el comando con el usuario `rabbit` (usando sudo para especificarlo) e introduciendo las rutas absolutas que encontramos antes (básicamente usar el comando que apareció 2 imágenes atrás):
`sudo -u rabbit /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py`  

![7c0bd0645eac566fbcdf62b16b96ce71](https://github.com/Theeraz/theraz.github.io/assets/90190970/dfa6ec16-6720-4f35-ab57-705799f27dd0)

Funcionó! Lo siguiente es mirar en el directorio de este usuario y encuentro un archivo que está marcado en rojo. Parece que el fondo rojo indica que tiene SUID. Pruebo a ejecutarlo y no funciona, así que pruebo a hacerle `cat`:  

![fb46abe8633b855a0df067773d271e5e](https://github.com/Theeraz/theraz.github.io/assets/90190970/25d6fee9-8aaf-4824-9bac-1692baaaa85e)  

No entiendo mucho de scripting y lo único que llama mi atención es esta línea:  

![70a644d13ad3afe1671149bb96422627](https://github.com/Theeraz/theraz.github.io/assets/90190970/a3e5d8b2-458d-4325-b553-77331e02ae02)  

Es la línea que aparece cuándo lo ejecutamos y parece que está configurado para que el output que da, sea de una hora en adelante y no la actual. Parece que el comando `date` carece de ruta absoluta. Es aquí cuándo investigando descubro el path hijacking. Al parecer debo modificar la variable de entorno PATH para que se ejecute el directorio que yo le indique y no el que ejecutaría por defecto:  
- `echo "/bin/bash" > date`: Para añadir la línea de código al comando `date`  
- `chmod +x date`: Para darle permisos de ejecución y especificar a qué archivo se lo queremos dar  
- `export PATH=/home/rabbit:$PATH`:  Usar `export` con una variable de entorno (PATH) me permite cambiarla de directorio y que se ejecute en este. Importante exportarla en el directorio en el que está el archivo que queremos ejecutar, ya que probé exportándola en home y no funcionó:  

![c14c8590373b9e33c392bb1de3d4d9f0](https://github.com/Theeraz/theraz.github.io/assets/90190970/02664787-6dcc-4a70-a181-b53b6d3e968b)  

Al ejecutar el archivo esta vez nos cambia al usuario "hatter". Pruebo a mirar en el directorio de este usuario en busca de la siguiente pista:  

![ae710c9ecf31c0088dd23d8034656e1c](https://github.com/Theeraz/theraz.github.io/assets/90190970/a58b1171-1143-4b4f-8097-3c570f4c3e1f)  

Encuentro una contraseña e intento escalar a `root` con ella, pero es inútil. Tampoco puedo usar sudo -l y es que este usuario no se encuentra en la lista de sudousers. Sigo probando cosas y de algún modo es cómo si tuviese incluso menos privilegios que antes: 

![532a95a37ae64504d05cbfcf23f254f6](https://github.com/Theeraz/theraz.github.io/assets/90190970/7207ab83-edda-4d8b-9b00-0252b0c55bab)  

Parece que actualmente no soy el usuario "hatter" por completo, decido "cambiar de usuario" usando la contraseña que he encontrado con este mismo usuario:

![c91334c59f6c97f300b9c7d835cc4257](https://github.com/Theeraz/theraz.github.io/assets/90190970/8402f86d-2e7b-4ecf-ba5b-1d4ff454c447)  

Y ahora sí tengo todos los permisos del usuario actual. Descubro que existe un script llamado LinPEAS y que trabaja con escalado de privilegios. Este script te ayuda a encontrar capabilities, que permiten a un proceso realizar operaciones privilegiadas específicas sin necesidad de ser `root`. La idea era usar LinPEAS pero no funciona, me hubiese ahorrado tiempo el haber conocido este comando antes:  

![0d307c7d77e6322576645d4cf585f094](https://github.com/Theeraz/theraz.github.io/assets/90190970/bc55092a-898b-472c-8787-a1be417d3197)  

`getcap -r`: Este comando lista los permisos de capacidad extendidos en algunos archivos y directorios, es decir, lista los comandos que podemos usar cómo `root` sin serlo. `-r` indica que se ejecutará de manera recursiva en directorios y subdirectorios. Buscando encuentro un comando que permite cambiar el `setuid` de `perl` y así poder escalar a `root`:  
`/usr/bin/perl -e 'use POSIX (setuid); POSIX::setuid(0); exec "/bin/bash";'`

![f15c4f0e581c2a49403a69350b69a16f](https://github.com/Theeraz/theraz.github.io/assets/90190970/1df9297d-a4fa-4287-a4aa-a8e0513579f5)

Consigo cambiar el UID a 0 (`root`)...y funciona! Tenemos privilegios de `root` y podemos acceder a la flag que encontramos al principio:  

![c5303978e4274c1d7f043ae64fde695a](https://github.com/Theeraz/theraz.github.io/assets/90190970/7e038f0a-86f1-45f2-af84-c2f6e58cb948)

He de decir que es la primera vez que me he enfrentado al escalado de privilegios y al path hijacking y aunque por algunos momentos me he sentido perdido, no me ha resultado muy tedioso encontrar las respuestas a mis preguntas y los comandos que me han permitido completar dicho escalado.

Recursos: https://www.hackingarticles.in/linux-for-pentester-perl-privilege-escalation/#:~:text=Capabilities%20in%20Privilege%20Escalation&text=Capabilities%20are%20those%20permissions%20that,to%20perform%20specific%20privileged%20tasks  

https://hacklido.com/blog/162-privileges-escalation-techniques-basic-to-advanced-in-linux-part-2
