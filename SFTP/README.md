SFTP
====

Le groupe des users
-------------------
Par défaut l'authentification est par mot de passe.
```
root@SFTP1 $> groupadd sftpusers
root@SFTP1 $> useradd -g applicatifs -G sftpusers -d /FILER/sftp/cedric -M -s /usr/libexec/openssh/sftp-server cedric
root@SFTP1 $> passwd cedric
```
Le chemin chrooté doit obligatoirement appartenir à "root:root" avec les droits 755
```
root@SFTP1 $> install -d -o root -g root -m 755 /FILER/sftp
root@SFTP1 $> mount /FILER/sftp/cedric
root@SFTP1 $> chown root:root /FILER/sftp/cedric
root@SFTP1 $> chmod 755 /FILER/sftp/cedric
```
Ce répertoire est en RO pour le user cedric.
Pour un répertoire en RW pour le group server
```
root@SFTP1 $> install -d -o cedric -g server -m 775 /FILER/sftp/cedric
```

Authentification par clef
-------------------------
Mise en place de l’authentification par clef
```
root@SFTP1 $> install -d -o cedric -g sftpusers -m 750 /FILER/sftp/cedric/.ssh
root@SFTP1 $> touch /FILER/sftp/cedric/.ssh/authorized_keys
root@SFTP1 $> ssh-keygen -t rsa -f /root/.ssh/id_rsa_cedric # pour tester
root@SFTP1 $> chmod 600 /root/.ssh/id_rsa_cedric
root@SFTP1 $> cat /root/.ssh/id_rsa_cedric.pub > /FILER/sftp/cedric/.ssh/authorized_keys
root@SFTP1 $> chmod 644 /FILER/sftp/cedric/.ssh/authorized_keys
root@SFTP1 $> chown root:root /FILER/sftp/cedric/.ssh/authorized_keys
```

Exemple de configuration SSHD
----------------------------

[Ici](sshd_config)

IPVS
----
example de conf [ici](keepalived.conf)
```
root@LB $> ipvsadm --clear
root@LB $> ipvsadm --add-service --tcp-service 192.168.1.20:2200 --scheduler rr
root@LB $> ipvsadm --add-server  --tcp-service 192.168.1.20:2200 --real-server 192.168.1.101:2200  --weight 1
```
Evidemment,  il ne faut pas oublier de mettre le [LVS tunning](sysctl.conf) en place
