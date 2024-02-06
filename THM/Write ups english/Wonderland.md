# Wonderland WriteUp

I start as I usually do, nmap port scanning with the commands that I already know and that usually work for me:
`nmap -sC -sV 10.10.7.36`.
- `-sC`: To activate script scanning and get as much information as possible about services, specific configurations, etc.
- `-sV`: To detect the versions of services on open ports, this helps to identify specific vulnerabilities or get more information on running services

![scan](https://github.com/Theeraz/theraz.github.io/assets/90190970/99c1cf9e-7c0f-4ee8-abbf-5f46c878c256)

Both port 22 and port 80 are open, so I'm guessing we'll have to use ssh to compromise it and find the flags. In the meantime I've looked on the web in case I can find something that might help me:

  ![rabbit1](https://github.com/Theeraz/theraz.github.io/assets/90190970/a49f3583-d936-4f6a-9226-56c10537cc02)

I take a look at the source code of the page hoping to find a username or password, but I can't find anything. The logical thing to do is to use gobuster to find directories on this page where I can probably find a clue:  

 `gobuster dir -u http://10.10.7.36 -w /usr/share/wordlists/dirb/common.txt 2> /dev/null`.
 - `gobuster dir`: Specifies that a directory scan will be performed.
 - `-w /usr/share/wordlists/dirb/common.txt`: Specifies the list of words to brute-force for directories
 - `2> /dev/null`: Redirects error messages to the null device, which means that error messages will not be displayed in the standard output.

![Gobuster](https://github.com/Theeraz/theraz.github.io/assets/90190970/12515126-cb44-4a60-8f37-2f4582c7d916)  

I find 3 directories. I access the first one (`/img`) where I find 3 images and 2 of them are the same but with a different colour and format, that catches my attention:  

![dir r](https://github.com/Theeraz/theraz.github.io/assets/90190970/4e9cb0be-00bf-4179-8c21-9374c60302f5)  

In the second `/index.html` I find the same directory I visited before, with a sentence (Follow the white rabbit) and a picture of the rabbit. I download the photo in case it contains any hidden clues.  
In the third `/r` I find another directory where there is only one sentence, in none of the 3 I have found anything in the source code:  

![f083eeebe66675c4814e215a7a213b26](https://github.com/Theeraz/theraz.github.io/assets/90190970/73d59fac-aa8b-4b5c-af71-4677913b9d77)  

I decide to use another dictionary with gobuster in the hope that maybe it will list a directory that hasn't been listed before:  
`gobuster dir -u http://10.10.7.36 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt 2> /dev/null`.   

![gobuster2](https://github.com/Theeraz/theraz.github.io/assets/90190970/09688e98-57e7-4295-8370-4609bbb7dfd4)  

I find several directories that didn't appear before, but the only accessible one is `/poem`. I investigate the source code a bit, but I can't find anything either. I decide to try steganographing the picture of the rabbit I found before:
`steghide extract -sf "file.jpg"`
- `extract`: To extract the hidden information.
- `-sf`: To indicate the file on which we want to execute the command.
It asks for a passphrase, but just pressing ``enter`` gives me permission to extract the track:

![58923ddaaa64af91cba3d4f86da9694f](https://github.com/Theeraz/theraz.github.io/assets/90190970/4728b5de-5ac6-49f3-a714-4386b4a4d4c5)  

Therein lies a clue, which I think makes it clear what the next step is, accessing the /r/a/b/b/i/t directory:  

![438e1063e1c98e70b93c4d1c0016646c](https://github.com/Theeraz/theraz.github.io/assets/90190970/22a970bf-8d85-4ca0-a510-98d5e7595700)  

In that directory is one of the images I found earlier with several sentences. Inspecting the source code I find what appears to be a user/password:

![a3c79c3094d6ad12c3317eca6b6009ba](https://github.com/Theeraz/theraz.github.io/assets/90190970/e70404e4-329f-4c65-bfe2-93a4bff2bfca)  

With those credentials I should be able to connect to the victim machine via SSH:  

![bc0f00922328428519d79b089cee4e40](https://github.com/Theeraz/theraz.github.io/assets/90190970/cf9aadfd-84a8-433f-b165-f25cbdaacb3f)  

I'm in. I look at the files in the current directory and there seems to be a flag with the name `root.txt`. There is also a `.py` file. I don't have permissions to open `root.txt` but I can read the `.py` file, which turns out to be another poem from which I can't draw any conclusions. Something that catches my attention is that in this user directory there is a root file, plus the clue in the room tells us that "everything here is topsy-turvy" so I try looking for a user file in the root directory: 

![2e39231c7454311d094efa93c9424e5f](https://github.com/Theeraz/theraz.github.io/assets/90190970/0c54ed1d-26e4-4fcb-a196-effa4e2c55d7)  

Bingo, there I find the first flag. I try to escalate to sudo but this user has no permissions. The next thing is to look at the permissions of the files in the user directory:  

![78a7791c798799c2880ab619fee3deac](https://github.com/Theeraz/theraz.github.io/assets/90190970/986c655c-7251-46ec-b184-e9a812607c47)  

Obviously root.txt can only be read by root, however, I don't see the point in finding a `.py` file and not being able to run it. I try `sudo -l` for privileges:  

![f72bbc99a5272e4621222357cf86087a](https://github.com/Theeraz/theraz.github.io/assets/90190970/f6a08795-7041-4895-8b9f-7384d5448308)

There are 2 important facts. The first is that `rabbit` seems to have privileges over the `.py` file. The second is that when doing a `cat` of this file, the first line imports a random module. After looking for a long time for information about possible options, I decide to create a fake module with the same name that will act when executing the file and that, being in the same directory, will be executed before the random module itself. To do this we will have to give it execution permissions with `chmod +x` and add the code we want it to execute:  
`vim random.py`  

![e3cefbe29474fb671133c664d5461fa9](https://github.com/Theeraz/theraz.github.io/assets/90190970/5a41aaf9-0d6b-4295-9012-21a422867aa5)  

With this line of code we can run a bash shell of a Python script. I save with `Esc + :x` and the next thing is to run the command with the user `rabbit` (using sudo to specify it) and entering the absolute paths we found before (basically use the command that appeared as output 2 images ago):
`sudo -u rabbit /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py`  

![7c0bd0645eac566fbcdf62b16b96ce71](https://github.com/Theeraz/theraz.github.io/assets/90190970/dfa6ec16-6720-4f35-ab57-705799f27dd0)  

It worked! Next I look in this user's directory and find a file that is marked in red. It seems that files listed with red background and white text indicate that the setuid bit has been reversed. This means that the file/script/program will run as the user it belongs to, not the user running it. I try running it as a script and it doesn't work, so I try `cat`:  

![fb46abe8633b855a0df067773d271e5e](https://github.com/Theeraz/theraz.github.io/assets/90190970/25d6fee9-8aaf-4824-9bac-1692baaaa85e)  

I don't understand much about scripting and the only thing that catches my attention is this line:  

![70a644d13ad3afe1671149bb96422627](https://github.com/Theeraz/theraz.github.io/assets/90190970/a3e5d8b2-458d-4325-b553-77331e02ae02)  

This is the line that appears when we run it and it seems that it is configured to give an output from one hour onwards and not the current one. It seems that the `date` command lacks an absolute path. This is when I discovered the "path hijacking" technique. Apparently I have to modify the PATH environment variable so that it runs the directory I specify and not the default:  
- `echo "/bin/bash" > date`: To add the line of code to the `date` command.  
- `chmod +x date`: To give it execution permissions and specify which file to give it to  
- `export PATH=/home/rabbit:$PATH`: Using `export` with an environment variable (PATH) allows me to change the directory and have it run in this one. It is important to export it in the directory where the file we want to run is, as I tried exporting it in home and it didn't work:  

![c14c8590373b9e33c392bb1de3d4d9f0](https://github.com/Theeraz/theraz.github.io/assets/90190970/02664787-6dcc-4a70-a181-b53b6d3e968b)  

When running the file this time it changes us to the user "hatter". I try looking in this user's directory for the following hint:  

![ae710c9ecf31c0088dd23d8034656e1c](https://github.com/Theeraz/theraz.github.io/assets/90190970/a58b1171-1143-4b4f-8097-3c570f4c3e1f)  

I find a password and try to escalate to `root` with it, but to no avail. I can't use sudo -l either, and this user is not in the sudousers list. I keep trying things and somehow it feels like I have even less privileges than before: 

![532a95a37ae64504d05cbfcf23f254f6](https://github.com/Theeraz/theraz.github.io/assets/90190970/7207ab83-edda-4d8b-9b00-0252b0c55bab)  

It seems that I am not currently the "hatter" user completely, I decide to "change user" using the password I have found with this same user:

![c91334c59f6c97f300b9c7d835cc4257](https://github.com/Theeraz/theraz.github.io/assets/90190970/8402f86d-2e7b-4ecf-ba5b-1d4ff454c447)  

And now I have all the permissions of the current user. I discover that there is a script called LinPEAS and that it is a security auditing tool for Linux systems. Its main function is to perform an automatic scan of the whole Linux system to identify insecure configurations, privilege settings, etc... This script helps you find executables with capabilities, which allow a process to perform specific privileged operations without needing to be `root`. The idea was to use LinPEAS but for some reason it doesn't work (I can't understand why it doesn't run). I find a command that does a similar function and it would have saved me a lot of time:  

![0d307c7d77e6322576645d4cf585f094](https://github.com/Theeraz/theraz.github.io/assets/90190970/bc55092a-898b-472c-8787-a1be417d3197)  

`getcap -r`: This command lists the extended capability permissions on some files and directories, meaning that it lists commands that we can use as `root` without being `root`. `-r` indicates that it will be executed recursively on directories and subdirectories. Since my options are limited to `perl` as these are two of my three possible options for changing the UID, I look for a possible way to exploit this vulnerability.  I manage to find a command that allows me to change the `setuid` of `perl` so that I can scale to `root`. This "technique" is known as "Privilege Escalation using capabilities":  
`/usr/bin/perl -e `use POSIX (setuid); POSIX::setuid(0); exec "/bin/bash";'`

![f15c4f0e581c2a49403a69350b69a16f](https://github.com/Theeraz/theraz.github.io/assets/90190970/1df9297d-a4fa-4287-a4aa-a8e0513579f5)

I manage to change the UID to 0 (`root`)...and it works! We have `root` privileges and we can access the flag we found at the beginning:  

![c5303978e4274c1d7f043ae64fde695a](https://github.com/Theeraz/theraz.github.io/assets/90190970/7e038f0a-86f1-45f2-af84-c2f6e58cb948)  

I have to say that this is the first time I have dealt with privilege escalation and path hijacking and although I felt lost at times, it was not too tedious to find the answers to my questions and the commands that allowed me to perform these actions.

Resources: https://www.hackingarticles.in/linux-for-pentester-perl-privilege-escalation/#:~:text=Capabilities%20in%20Privilege%20Escalation&text=Capabilities%20are%20those%20permissions%20that,to%20perform%20specific%20privileged%20tasks  

https://hacklido.com/blog/162-privileges-escalation-techniques-basic-to-advanced-in-linux-part-2



