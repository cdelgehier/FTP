FTPS
====


> **Implicite vs Explicite:**
> - Implicite: Suppose que le serveur attend tout SSL. Cela signifie que lorsque le client se connecte au serveur, il sera immédiatement négocier en SSL. Normalement, les connexions implicites sont également sur un port différent, tel que le port 990
> - Explicite: Une connexion FTP normale est établie, généralement sur le port standard 21. Cependant, après la connexion, le client envoie une commande pour passer en mode SSL. Cette commande est "SSL AUTH". Lorsque cette commande est envoyée le serveur répondra normalement, puis établira une connexion SSL
> Avec les deux, la connexion de données est toujours entièrement cryptée avec SSL.

Installation de VSFTPD
----------------------

```
root@FTP $> yum install vsftpd
root@FTP $> chmod 600 /etc/vsftpd/vsftpd.conf

```

La gestion des users
-------------------

```
root@FTP $> mv /etc/vsftpd/ftpusers /etc/vsftpd/ftpusers.save
root@FTP $> cat /etc/vsftpd/login.txt # users virtuels
cedric
root@FTP $> db_load -T -t hash -f /etc/vsftpd/login.txt /etc/vsftpd/login.db
root@FTP $> chmod 600 /etc/vsftpd/login.*
root@FTP $> cd /etc/vsftpd/vsftpd_user_conf 
root@FTP $> cat cedric
# Download
anon_world_readable_only=NO

# Upload 
anon_upload_enable=YES

# Autorisation des commandes influant sur le systeme de fichier (STOR, DELE, RNFR, RNTO, MKD, RMD, APPE and SITE) pour les locaux
write_enable=YES

local_root=/FILER/ftps/cedric

cmds_allowed=ABOR,ALLO,CDUP,CWD,DELE,EPRT,EPSV,FEAT,HELP,LIST,MODE,NLST,NOOP,OPTS,PASS,PASV,PORT,PWD,QUIT,REIN,REST,RETR,SIZE,STAT,STOR,STOU,STRU,SYST,TYPE,USER,AUTH,ADAT,PBSZ,PROT,CCC,MIC,CONF,ENC,RNFR,RNTO,MKD
```
> **ftpusers et user_list**
>
> Ces 2 fichiers interdisent des users mais a différent niveaux
> - ftpusers: A la connexion d'un utilisateur, PAM vient lire ce fichier et si l'identifiant de connexion utilisé est dans ce fichier, la connexion est refusée. Notons que la demande de mot de passe se fait quand meme. Donc le mot de passe (de root ?) transite en clair sur le reseau.
> - user_list: Il est utilisé directement par vsftpd. Il peut avoir deux usages : soit les seuls utilisateurs contenus dans ce fichier ont le droit de se connecter, soit l'accès leur est systématiquement refusé. Avec lui, vsftpd coupera directement la connexion, sans même demander le mot de passe.

Le fichier vsftpd.conf
---------------------
Voici un exemple de configuration qui:
- Ecoute sur une ip (de vituelle) en Implicite Passif
- Limite le range de port pour la 
- Refuse les users anonymes
- Autorise les users locaux et virtuels
- Force le TLS
- Tourne en permanance (standalone)
```
# Ip de service
listen_address=192.168.1.21
# Port d'ecoute
listen_port=990

# Interdiction d'utiliser le port 20
connect_from_port_20=NO

# Fichier de config PAM
pam_service_name=vsftpd

# Mode standalone
listen=YES

# Banniere d'accueil
ftpd_banner="Welcome !"

# Interdire les connexions anonymes au serveur
anonymous_enable=NO
# Refus des droits d'ecriture pour les anonymes (et donc utilisateurs virtuels) par defaut
# les autorisations seront donnees au cas par cas :
# pas d'upload
anon_upload_enable=NO
# pas de creation de repertoire
anon_mkdir_write_enable=NO
# pas de creation, suppression, renommage de repertoire ...
anon_other_write_enable=NO

# On autorise les connexions des utilisateurs locaux. C'est indispensable
# pour que les utilisateurs virtuels (mappes sur un utilisateur local)
# puissent se connecter (les "vrais" utilisateurs locaux sont ensuite desactives
# avec le fichier user_list
local_enable=YES

# les users virtuels seront mapper sur le compte local "ftp"
guest_enable=YES
guest_username=ftp

# Fichier des users interdits (ex: root)
userlist_file=/etc/vsftpd/user_list
# on charge la liste
userlist_enable=YES
# on refuse toute la liste qu'elle contient
userlist_deny=YES

# Activer les droits specifiques par utilisateur
user_config_dir=/etc/vsftpd/vsftpd_user_conf

# Refus des commandes influant sur le systeme de fichier (STOR, DELE, RNFR, RNTO, MKD, RMD, APPE and SITE) pour les locaux
write_enable=NO
local_umask=022

#user_sub_token=$USER
#local_root=/FILER/ftps/$USER

#Les utilisateurs qui sont chroot dans leur home ne peuvent pas a priori naviguer dans un dossier a l exterieur de leur home.
#La solution est d utiliser la commande mount avec l option bind
#mount --bind dossier_source dossier_ftp
# On pensera a activer le montage dans /etc/fstab
#dossier_source dossier_ftp    none  bind,defaults,auto     0    0

#nombre de connexions simultannee maximum
#max_clients=50

#nombre maximum de connexion avec la meme adresse IP
#max_per_ip=4

# Options for PASV
# connections through firewall.

pasv_enable=YES
#pasv_promiscuous=NO
pasv_min_port=40000
pasv_max_port=40030
#pasv_address=
#port_promiscuous=NO

# Options for SSL
# encrypted connections.

#On autorise le SSL
ssl_enable=YES

#allow_anon_ssl=NO

#On force l'utilisation du SSL
force_local_data_ssl=YES
force_local_logins_ssl=YES
implicit_ssl=YES
require_ssl_reuse=NO
require_cert=NO

#on accepte les differentes versions d'SSL
ssl_tlsv1=YES
ssl_sslv2=NO
ssl_sslv3=NO

#Renseignement du certificat a utiliser
rsa_cert_file=/etc/vsftpd/ftps.crt
rsa_private_key_file=/etc/vsftpd/ftps.key

# Activation de logs
xferlog_enable=YES
xferlog_file=/var/log/vsftpd.xfer.log
xferlog_std_format=YES
log_ftp_protocol=YES
vsftpd_log_file=/var/log/vsftpd.log
dirmessage_enable=YES
```

> **Passif Vs Actif:
> 
> FTP peut fonctionner dans un *actif* ou un mode *passif*, qui détermine comment une connexion de données est établie. Dans les deux cas, un client crée une connexion de contrôle TCP à un port de commande du serveur FTP 21. Donc, en général il n'y a pas de problèmes lors de l'ouverture de la connexion de contrôle. Alors que les autres protocoles utilisent la même connexion à la fois pour le contrôle de session et le transfert de fichiers (données), le protocole FTP utilise une connexion séparée pour ces transferts.
> - Actif: Dans ce mode, le client commence à écouter sur un port aléatoire pour les connexions de données entrantes provenant du serveur. Il informe le serveur de ce port grace à la commande *PORT*. Ce mode est peu utilisé car le client est souvent derriere un par feu ou un firewall et ces derniers refusent les connexions de ce genre.
> - Passif : Ici c'est l'admin sys qui s'occupe de tout coté serveur, plutot qu'individuellement coté client. Le client utilise la directive de contrôle *PASV* et en retour il recoit une ip et un port coté serveur sur lesquels le transfert se fera.

Le PAM
------
Le Pluggable Authentication Modules est celui qui lire notre base de login berkeley
```
root@FTP $> cat /etc/pam.d/vsftpd
#Connexion des user locaux
auth 	sufficient	pam_unix.so
account sufficient	pam_unix.so
# Connexion des user virtuels
auth	required	/lib64/security/pam_userdb.so	db=/etc/vsftpd/login
account required	/lib64/security/pam_userdb.so	db=/etc/vsftpd/login
```

