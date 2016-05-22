SFTP
====

Le groupe des users
-------------------
Par défaut l'authentification est par mot de passe.
```
$> groupadd sftpusers
$> useradd -g applicatifs -G sftpusers -d /FILER/sftp/cedric -M -s /usr/libexec/openssh/sftp-server cedric
$> passwd cedric
```
Le chemin chrooté doit obligatoirement appartenir à "root:root" avec les droits 755
```
$> install -d -o root -g root -m 755 /FILER/sftp
$> mount /FILER/sftp/cedric
$> chown root:root /FILER/sftp/cedric
$> chmod 755 /FILER/sftp/cedric
```
Ce répertoire est en RO pour le user cedric.
Pour un répertoire en RW pour le group server
```
$> install -d -o cedric -g server -m 775 /FILER/sftp/cedric
```

Authentification par clef
-------------------------
Mise en place de l’authentification par clef
```
$> install -d -o cedric -g sftpusers -m 750 /FILER/sftp/cedric/.ssh
$> touch /FILER/sftp/cedric/.ssh/authorized_keys
$> ssh-keygen -t rsa -f /root/.ssh/id_rsa_cedric # pour tester
$> chmod 600 /root/.ssh/id_rsa_cedric
$> cat /root/.ssh/id_rsa_cedric.pub > /FILER/sftp/cedric/.ssh/authorized_keys
$> chmod 644 /FILER/sftp/cedric/.ssh/authorized_keys
$> chown root:root /FILER/sftp/cedric/.ssh/authorized_keys
```

