# Pickle Rick WriteUp

Nos facilitan la ip --> 10.10.171.63  
Hago uso del comando export IP=10.10.171.63 para asignarle una variable y hacerlo más cómodo.  
Uso nmap en busca de puertos abiertos, uso el siguiente comando: 
`nmap -sC -sV 10.10.171.63`  
![PortScan](https://github.com/Theeraz/theraz.github.io/assets/90190970/eb92a7c1-e126-42c9-9117-74d225ee6b48)  
Los puertos 22 (ssh) y 80 (tcp) están abiertos.  
Mientras el escáner actúa busco en el source code (ctr+U) de la página y encuentro un username.  
![SourceCode](https://github.com/Theeraz/theraz.github.io/assets/90190970/d5604384-f6a9-40db-ac74-c7e5c18e0aad)  
Decido probar con Hydra para ver si puedo acceder mediante fuerza bruta pero antes me aseguro de que tengo el diccionario rockyou con el comando locate:  
![LocateRockyou](https://github.com/Theeraz/theraz.github.io/assets/90190970/1419ac6c-ae51-4f97-9baf-37fc93eeb63c)  
Me muevo al directorio en el que se encuentra el diccionario y procedo a insertar el comando de Hydra desde ahí:  
![DirectorioHydra](https://github.com/Theeraz/theraz.github.io/assets/90190970/10597dd9-45d9-45ea-b9e9-54ddbdc387ba)
No sirve de nada, así que busco información sobre qué hacer y decido usar Gobuster que es una herramienta para usar fuerza bruta en directorios y archivos almacenados en una página web. Busco common.txt con el comando "locate" para usarlo cómo diccionario.  
Pruebo el siguiente comando:  


