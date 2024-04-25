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
continuer a installer wordpress
```

3. le script de sauvegarde et de chiffrement :

**Installatin des dependances*
```bash
sudo apt install python3-pip
python3 -m pip install paramiko 

```


```python
import shutil
import subprocess
import os
import datetime
import paramiko

source_dir = '/var/www/'
backup_dir = '/home/yug/'
backup_name = 'wordpress_backup_' + datetime.datetime.now().strftime('%Y-%m-%d') + '.tar.gz'

print("Connecting to MySQL server")

# Backup MySQL database
mysql_user = 'yug'
mysql_password = 'doranco2024'
mysql_database = 'wordpress'

mysql_backup_file = os.path.join(backup_dir, 'mysql_backup.sql')

try:
    mysql_dump_command = f"mysqldump -u {mysql_user} -p{mysql_password} {mysql_database} > {mysql_backup_file}"
    subprocess.call(mysql_dump_command, shell=True)
    print("MySQL backup completed successfully")
except Exception as e:
    print("Error occurred while backing up MySQL:", e)

# Copy FTP content to backup directory
try:
    shutil.copytree(source_dir, os.path.join(backup_dir, 'www'))
    print("Website directory copied successfully")
except Exception as e:
    print("Error occurred while copying website directory:", e)

# Create archive including MySQL backup and website content
try:
    shutil.make_archive(backup_name.split('.')[0], 'gztar', backup_dir)
    print("Archive created successfully")
except Exception as e:
    print("Error occurred while creating archive:", e)

# Copy encryption key to backup directory
encryption_key = 'aes_key.txt'
encryption_key_backup = os.path.join(backup_dir, encryption_key)

try:
    shutil.copy(encryption_key, encryption_key_backup)
    print("Encryption key copied successfully")
except Exception as e:
    print("Error occurred while copying encryption key:", e)

# Encrypt backup file with OpenSSL
encrypted_backup_file = os.path.join(backup_dir, 'encrypted_backup.tar.gz')

try:
    openssl_encrypt_command = f"openssl enc -aes-256-cbc -salt -in {backup_name} -out {encrypted_backup_file} -pass file:{encryption_key_backup}"
    subprocess.call(openssl_encrypt_command, shell=True)
    print("Encryption completed successfully")
except Exception as e:
    print("Error occurred while encrypting backup file:", e)

# SFTP Connection details
sftp_host = '192.168.1.13'
sftp_port = 22
sftp_username = 'yug'
sftp_password = 'doranco'

# Connect to SFTP server and upload encrypted backup file and encryption key
try:
    transport = paramiko.Transport((sftp_host, sftp_port))
    transport.connect(username=sftp_username, password=sftp_password)
    sftp = paramiko.SFTPClient.from_transport(transport)
    print("Connected to SFTP server")

    sftp.put(encrypted_backup_file, os.path.basename(encrypted_backup_file))
    sftp.put(encryption_key_backup, os.path.basename(encryption_key_backup))
    print("Backup and encryption key uploaded successfully")

    sftp.close()
    transport.close()
except paramiko.AuthenticationException:
    print("Authentication failed. Please check your SFTP credentials.")
except Exception as e:
    print("Error occurred during SFTP transfer:", e)


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


5. decrypter sur le server de sauvgarde :  
```bash
sudo openssl enc -d -aes-256-cbc -in encrypted_backup.tar.gz -out decrypted -pass file:aes_key.txt
```
```bash
tar -xzf decrypted
```

**Version améliorée du script de sauvegarde :**
```python
import shutil
import subprocess
import os
import datetime
import paramiko
import random
import string

# Répertoire source contenant les données à sauvegarder
source_dir = '/var/www/'

# Répertoire de sauvegarde où seront stockées les sauvegardes
backup_dir = '/home/yug/'

# Nom de base de l'archive de sauvegarde
backup_base_name = 'wordpress_backup_' + datetime.datetime.now().strftime('%Y-%m-%d')

# Fonction pour générer une clé de chiffrement aléatoire
def generate_encryption_key():
    return ''.join(random.choices(string.ascii_letters + string.digits, k=16))

# Génération de la clé de chiffrement
encryption_key = generate_encryption_key()

# Chemin vers le fichier contenant la clé de chiffrement
encryption_key_file = os.path.join(backup_dir, 'encryption_key.txt')

# Écriture de la clé de chiffrement dans un fichier
with open(encryption_key_file, 'w') as key_file:
    key_file.write(encryption_key)

print("Encryption key generated successfully")

# Sauvegarde de la base de données MySQL dans un fichier SQL
mysql_user = 'yug'
mysql_password = 'doranco2024'
mysql_database = 'wordpress'

mysql_backup_file = os.path.join(backup_dir, 'mysql_backup.sql')

try:
    mysql_dump_command = f"mysqldump -u {mysql_user} -p{mysql_password} {mysql_database} > {mysql_backup_file}"
    subprocess.call(mysql_dump_command, shell=True)
    print("MySQL backup completed successfully")
except Exception as e:
    print("Error occurred while backing up MySQL:", e)

# Copie du contenu du répertoire FTP dans le répertoire de sauvegarde
try:
    shutil.copytree(source_dir, os.path.join(backup_dir, 'www'))
    print("Website directory copied successfully")
except Exception as e:
    print("Error occurred while copying website directory:", e)

# Création de l'archive contenant la sauvegarde MySQL et le contenu FTP
backup_name = f"{backup_base_name}_{datetime.datetime.now().strftime('%H-%M-%S')}.tar.gz"

try:
    shutil.make_archive(os.path.join(backup_dir, backup_name.split('.')[0]), 'gztar', backup_dir)
    print("Archive created successfully")
except Exception as e:
    print("Error occurred while creating archive:", e)

# Chiffrement de l'archive avec OpenSSL
encrypted_backup_file = os.path.join(backup_dir, f"encrypted_{backup_name}")

try:
    openssl_encrypt_command = f"openssl enc -aes-256-cbc -salt -in {backup_name} -out {encrypted_backup_file} -pass file:{encryption_key_file}"
    subprocess.call(openssl_encrypt_command, shell=True)
    print("Encryption completed successfully")
except Exception as e:
    print("Error occurred while encrypting backup file:", e)

# Détails de connexion SFTP pour le serveur distant
sftp_host = '192.168.1.13'
sftp_port = 22
sftp_username = 'yug'
sftp_password = 'doranco'

# Connexion au serveur SFTP et envoi de l'archive chiffrée et de la clé de chiffrement
try:
    transport = paramiko.Transport((sftp_host, sftp_port))
    transport.connect(username=sftp_username, password=sftp_password)
    sftp = paramiko.SFTPClient.from_transport(transport)
    print("Connected to SFTP server")

    sftp.put(encrypted_backup_file, os.path.basename(encrypted_backup_file))
    sftp.put(encryption_key_file, os.path.basename(encryption_key_file))
    print("Backup and encryption key uploaded successfully")

    sftp.close()
    transport.close()
except paramiko.AuthenticationException:
    print("Authentication failed. Please check your SFTP credentials.")
except Exception as e:
    print("Error occurred during SFTP transfer:", e)
finally:
    # Nettoyage : suppression des fichiers temporaires et des répertoires
    os.remove(encryption_key_file)
    os.remove(encrypted_backup_file)
    os.remove(mysql_backup_file)
    shutil.rmtree(os.path.join(backup_dir, 'www'))
    print("Temporary files and directories deleted successfully")
```

### Spécifications du Programme de Sauvegarde :

1. **Objectif :** Le programme effectue une sauvegarde complète d'une base de données MySQL et du contenu d'un répertoire FTP, chiffre les données sauvegardées, les transfère sur un serveur distant via SFTP, puis nettoie les fichiers temporaires locaux.

2. **Fonctionnalités :**
   - Sauvegarde de la base de données MySQL dans un fichier SQL.
   - Copie du contenu du répertoire FTP dans un répertoire de sauvegarde local.
   - Génération aléatoire d'une clé de chiffrement pour sécuriser les données.
   - Création d'une archive contenant la sauvegarde MySQL et le contenu FTP.
   - Chiffrement de l'archive avec OpenSSL en utilisant la clé de chiffrement.
   - Transfert de l'archive chiffrée et de la clé de chiffrement sur un serveur distant via SFTP.
   - Suppression des fichiers temporaires locaux après le transfert.

3. **Technologies Utilisées :**
   - Python pour l'automatisation des tâches.
   - Shell pour l'exécution de commandes système.
   - Paramiko pour la connexion et le transfert via SFTP.
   - OpenSSL pour le chiffrement des données sensibles.
   - Système de fichiers Linux pour la gestion des fichiers et répertoires.

4. **Configuration Requise :**
   - Accès à la base de données MySQL avec des privilèges de sauvegarde.
   - Accès en écriture au répertoire FTP et aux fichiers de sauvegarde.
   - Installation de Python et des modules Paramiko.
   - Disponibilité de OpenSSL pour le chiffrement des données.

5. **Utilisation :**
   - Personnalisation des chemins des répertoires et des informations d'identification dans le code.
   - Exécution du programme de sauvegarde sur le serveur où les données sont stockées.
   - Automatisation possible via des tâches planifiées ou des scripts d'automatisation.




