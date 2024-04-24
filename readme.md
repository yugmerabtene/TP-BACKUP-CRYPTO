#### TP : 

Enoncé :

1. Installez la pile LAMP sur votre machine virtuelle Ubuntu Server et sécurisez-la.
2. Installez WordPress sur votre machine virtuelle Ubuntu Server via le terminal.
3. Créez un script en Python qui effectue une sauvegarde quotidienne en cronjob sur le répertoire FTP contenant votre application WordPress (c'est-à-dire www), ainsi que la base de données complète du serveur MySQL. Le tout doit être chiffré avec une clé symétrique pour garantir la sécurité.
4. Envoyez la sauvegarde chiffrée en FTP vers le deuxième serveur de sauvegarde, qui se trouve sur le même réseau virtuel que votre serveur web principal.

Réponses :

1. Pour installer la pile LAMP sur votre machine virtuelle Ubuntu Server et la sécuriser, vous pouvez utiliser les commandes suivantes :

```bash
sudo apt update
sudo apt install apache2
sudo apt install mysql-server
sudo apt install php libapache2-mod-php php-mysql
sudo mysql_secure_installation
```

2. Pour installer WordPress via le terminal sur votre machine virtuelle Ubuntu Server, vous pouvez suivre ces étapes :

```bash
sudo apt update
sudo apt install wordpress
```

3. le script de sauvegarde et de chiffrement :

```python
import os
import shutil
import datetime
import subprocess
import ftplib

source_dir = '/var/www/'
backup_dir = '/path/to/backup/'
backup_name = 'wordpress_backup_' + datetime.datetime.now().strftime('%Y-%m-%d') + '.tar.gz'

# Sauvegarde MySQL
mysql_user = 'your_mysql_username'
mysql_password = 'your_mysql_password'
mysql_database = 'your_mysql_database_name'
mysql_backup_file = os.path.join(backup_dir, 'mysql_backup.sql')

mysql_dump_command = f"mysqldump -u {mysql_user} -p{mysql_password} {mysql_database} > {mysql_backup_file}"
subprocess.call(mysql_dump_command, shell=True)

# Chiffrement avec une clé symétrique (exemple avec openssl)
encryption_key = 'your_encryption_key'
encrypted_backup_file = os.path.join(backup_dir, 'encrypted_backup.tar.gz')

openssl_encrypt_command = f"openssl enc -aes-256-cbc -salt -in {mysql_backup_file} -out {encrypted_backup_file} -k {encryption_key}"
subprocess.call(openssl_encrypt_command, shell=True)

# Transfert FTP
ftp_server = 'ftp.example.com'
ftp_username = 'your_ftp_username'
ftp_password = 'your_ftp_password'

ftp = ftplib.FTP(ftp_server)
ftp.login(ftp_username, ftp_password)

with open(encrypted_backup_file, 'rb') as file:
    ftp.storbinary('STOR ' + os.path.basename(encrypted_backup_file), file)

ftp.quit()
```

1. Ouvrez votre terminal.
2. Tapez la commande suivante pour ouvrir le fichier cronjobs dans l'éditeur de texte de votre choix :

```bash
crontab -e
```

3. Ajoutez la ligne suivante à la fin du fichier pour exécuter votre script de sauvegarde Python tous les jours à minuit (assurez-vous de remplacer `/chemin/vers/backup.py` par le chemin absolu de votre script de sauvegarde Python) :

```bash
0 0 * * * /chemin/vers/backup.py
```

4. Enregistrez et fermez le fichier.
