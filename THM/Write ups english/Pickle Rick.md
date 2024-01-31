# Pickle Rick WriteUp  

I have to say that this is my first write up, I know that I have a lot to improve and that the images are not of the best possible quality. I will make an effort to improve the quality of these as I do them. And now...¡let's get down to business!  

They give us the ip --> 10.10.171.63  
I use the command export IP=10.10.171.63 to assign a variable and make it more comfortable.  
I use nmap to search for open ports, I use the following command: 
`nmap -sC -sV 10.10.171.63`.  

![PortScan](https://github.com/Theeraz/theraz.github.io/assets/90190970/eb92a7c1-e126-42c9-9117-74d225ee6b48)  

Ports 22 (ssh) and 80 (tcp) are open.  
While the scanner is running I search the source code ``(ctr+U)`` on the page and find a username.  

![SourceCode](https://github.com/Theeraz/theraz.github.io/assets/90190970/d5604384-f6a9-40db-ac74-c7e5c18e0aad)  

I decide to try with Hydra to see if I can brute force my way in but first I make sure I have the rockyou dictionary with the locate command:  

![LocateRockyou](https://github.com/Theeraz/theraz.github.io/assets/90190970/1419ac6c-ae51-4f97-9baf-37fc93eeb63c)  

I move to the directory where the dictionary is located and proceed to insert the Hydra command from there:  

![DirectorioHydra](https://github.com/Theeraz/theraz.github.io/assets/90190970/10597dd9-45d9-45ea-b9e9-54ddbdc387ba)

It's no use, so I look for information on what to do and decide to use Gobuster which is a tool to use brute force on directories and files stored on a web page. I search for `common.txt` with the `locate` command to use it as a dictionary.  
I try the following command:  
`gobuster dir -u http://10.10.171.63 -w /usr/share/wordlists/dirb/common.txt 2> /dev/null`  

![gobuster](https://github.com/Theeraz/theraz.github.io/assets/90190970/4618971c-dbfc-4232-83bd-5d7c19f2c526)  

There are several directories that I try to access via the website. I try robots.txt:  

![Wubbalubba](https://github.com/Theeraz/theraz.github.io/assets/90190970/59b44a8d-7f04-45c5-bf7e-564bdc763cea) 

This is what I find, maybe it is a credential, I decide to save it, it is possible that it will be useful for me at some point. I look in the assets directory:  

![Assets](https://github.com/Theeraz/theraz.github.io/assets/90190970/e8db56f2-ddd7-498e-9847-c977a4f6d3e4)  

I can't find anything that works for me so I use Gobuster with another dictionary focused on directories contained in web pages. The name of this one is ``directory-list-lowercase-2.3-medium.txt``.
I try the previously used command but with this dictionary:  
`gobuster dir -u http://10.10.171.63 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt`

![Gobuster2](https://github.com/Theeraz/theraz.github.io/assets/90190970/c75be516-04f3-4056-9774-7277126c723b)  

It's been 10 minutes and it still hasn't finished, so I start messing around with nmap. I discover that I can use nmap's own scripts and I try using port 80 to use a script that lists the directories assigned to that port with an http-enum script:  

![Enum](https://github.com/Theeraz/theraz.github.io/assets/90190970/4736966a-8cc7-4015-b001-c9f1be2430bd)  

It has been much quicker and has located what appears to be the login for the website. I try to access using the username from the source code of the page and the word I found in the robot.txt from before and indeed I gain access with the user ``R1ckRul3s`` and the password ``Wubbalubbadubdub``.  
The first thing I find is a command panel, I try to access the other directories but they are inaccessible, I guess that only the admin can access them. I go back to the command panel and use `ls`: 

![Ls](https://github.com/Theeraz/theraz.github.io/assets/90190970/aa251b59-5959-4068-958a-b89451f3ca85)  

I try to look at the contents of what looks like a file that will give me a flag (Sup3rS3cretPickl3Tngred.txt) but the ``cat`` command is disabled. I try the ``less`` command and find the first flag:  

![Flag1](https://github.com/Theeraz/theraz.github.io/assets/90190970/13be8e19-ad7f-4a58-814c-00ce9d0da8a1)  

I decide to do the same with clue.txt:  

![Clue](https://github.com/Theeraz/theraz.github.io/assets/90190970/98f27d04-0128-4fcc-b89f-2e433441c238)  

I try to navigate between directories and it won't let me. I have to do one liners to get the information I need. I try with "ls ../.../.../home/" (I go back 3 times because in the pwd I go to /var/www/html and I can only move there). In home I find 2 directories:  

![Directorios](https://github.com/Theeraz/theraz.github.io/assets/90190970/44c8586a-43a8-40ae-95e8-50ad07e96a1d)  

I try to do the same, this time going into the "Rick" directory. I find a file that looks like it will tell me the 2nd flag. I can't find a way to access that file.  
It looks like the only way to get access is through a reverse shell. I find a cheat sheet on ironhackers, I set out to try those commands:  

![Revershell](https://github.com/Theeraz/theraz.github.io/assets/90190970/d40c81c6-6b53-4e42-9bee-0d9926dbcefe)    
![Notwork](https://github.com/Theeraz/theraz.github.io/assets/90190970/4eb1c3c4-7071-463e-9f51-9a6f6153ede8) 

It doesn't seem to work. I'm going to try a different shell from the same ironhackers cheat sheet. This time I try it with Perl, choosing port 4444: 

![Works](https://github.com/Theeraz/theraz.github.io/assets/90190970/20decef1-1b6d-462a-b1b5-9a851e7f3e58)  
![Works2](https://github.com/Theeraz/theraz.github.io/assets/90190970/95282c98-b037-4747-be14-f98f292c182a)  

It works, I can now move around the system without any limitation of commands. I access the directory that I found on the website using commands and I get the 2nd flag: 

![Flag2](https://github.com/Theeraz/theraz.github.io/assets/90190970/9655e461-7398-4c12-bb0d-ee978c79c023)

To find the last flag, I decide to try to escalate to root with "sudo su". And... it works! So I look in the "/root" folder and find a file that seems to be the 3rd flag: 

![Flag3](https://github.com/Theeraz/theraz.github.io/assets/90190970/3a885640-bfce-429d-8f94-a3817f10e578)  
![Flag3'1](https://github.com/Theeraz/theraz.github.io/assets/90190970/9d6aa74f-8030-4086-a841-b02b27016a9e) 

With this I manage to get all the flags of the machine.

Recursos: https://ironhackers.es/herramientas/reverse-shell-cheat-sheet/
