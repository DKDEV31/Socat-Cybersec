# Projet Socat
## WriteUp Room Toss a coin
- Scan
  - Resultat

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


  - 80 - SSH
  - 22 - http
- Dirbuster
  - navigation
- SSH - Jaskier
- User.txtc
- Yen
- Geralt
- root.txt

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
