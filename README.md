# NAS-Debian
NAS Debian
NAS Debian
Introduction et Contextualisation
Notre projet consiste à configurer un serveur NAS (Network Attached Storage) sous Debian sans interface graphique, en utilisant des services comme SFTP, WebDAV, et en créant un espace partagé (Dossier Public). Vous devez également gérer plusieurs utilisateurs avec des dossiers personnels, une session administrateur, et éventuellement ajouter des fonctionnalités avancées comme la virtualisation, la sauvegarde (rsync), et la configuration RAID.
Sommaire
1. Préparation du système
Installation NAS Debian
Mettre à jour le système
Configuration des services ( SSH, APACHE ) 
2. Création des utilisateurs et dossiers
Création un utilisateur administrateur
Création des utilisateurs standards (exemple pour un utilisateur "user1")
Création des dossiers personnels pour chaque utilisateur 
Création d’un dossier public partagé
3. Configuration des services
SFTP (Secure File Transfer Protocol)
WebDAV ( Installation Apache et le module WebDAV )
4. Configuration RAID (optionnel)
5. Configuration Raid 5
6. Sauvegarde (optionnel)
7. Tests et vérifications
8. Virtualisation ( optionnel )
9. Conclusion

Voici les étapes clés à suivre pour réaliser ce projet depuis notre serveur NAS :
1. Préparation du système
*Installation Debian
Installation Debian sur notre serveur en mode "sans interface graphique"


Écran de choix des programmes à installer ( "Serveur SSH" et "Utilitaires système standard" lors de l'installation).




















*Ajout de disque dans notre NAS:
















































*Mise à jour du système


sudo apt update && sudo apt upgrade -y


Met à jour la liste des paquets disponibles en téléchargeant les informations depuis les dépôts pour garder le système sécurisé et optimisé.


*Configuration des services ( SSH, APACHE ) 










 Configuration l'accès SSH, pour pouvoir administrer le serveur à distance.








Apache est utilisé pour activer WebDAV, permettant ainsi aux utilisateurs d'accéder de manière 
sécurisée aux fichiers hébergés sur le serveur.


2. Création des utilisateurs et dossiers
Créer un utilisateur administrateur: 
Cette section explique comment configurer les comptes utilisateurs et organiser les dossiers, en créant un compte administrateur. Ce compte permet un accès privilégié pour gérer les configurations système, les autorisations d’accès, et assurer la sécurité des données.
sudo adduser admin
sudo usermod -aG sudo admin




















*Créer des utilisateurs standards (exemple pour un utilisateur "user1")
Créer des utilisateurs standards (exemple pour un utilisateur "user1") :
sudo adduser user1






















*Créer des dossiers personnels pour chaque utilisateur 
sudo mkdir /home/user1/private
sudo chown user1:user1 /home/user1/private




Cette commande est utilisée pour :
1.Isoler et sécuriser les données : Chaque utilisateur a un espace dédié, protégé par des permissions spécifiques, évitant les accès non autorisés.
2.Organiser les fichiers : Structurer les données par utilisateur simplifie la gestion, les sauvegardes et la recherche d’informations.
3.Personnaliser l’environnement : Permet de stocker des configurations, documents et préférences propres à chaque utilisateur.
4.Faciliter l’administration : Les administrateurs peuvent auditer, contrôler les quotas ou appliquer des politiques de sécurité de manière ciblée.
Exemple concret :
Sur un système Linux, cela correspond au dossier /home/utilisateur, où chaque utilisateur a des droits exclusifs sur son répertoire personnel.
*Créer un dossier public partagé
sudo mkdir /srv/public
sudo chmod 777 /srv/public




Cette commande est utilisée pour :
Faciliter la collaboration : Permettre à plusieurs utilisateurs d’accéder et de modifier des fichiers communs dans un espace centralisé.
Simplifier le partage : Éviter la duplication des données en centralisant les documents partagés (exemple : rapports, modèles, ressources d’équipe).
Contrôler les accès : Définir des permissions spécifiques (lecture/écriture) pour garantir la sécurité tout en autorisant un accès public ou restreint.
Organiser les ressources communes : Centraliser les fichiers utilisés par plusieurs services ou projets (exemple : dossiers "Public" sur Windows ou "/shared" sur Linux).
Exemple concret :
Dans une entreprise, un dossier public partagé peut stocker des documents internes (procédures, calendriers) accessibles à tous les employés, mais protégés contre les modifications non autorisées.


3. Configuration des services
 Le SFTP (Secure File Transfer Protocol) est un protocole sécurisé permettant le transfert de fichiers via SSH.
Éditez le fichier /etc/ssh/sshd_config :
sudo nano /etc/ssh/sshd_config




























Dans cet éditeur de texte, on a les exécutions:
    Match Group sftpusers
    ChrootDirectory /home/%u
    ForceCommand internal-sftp
    AllowTcpForwarding no
    X11Forwarding no:


Redémarrage  SSH :
Redémarrez SSH :sudo systemctl restart sshd


On redémarre le SSH pour appliquer les mises à jour ou modifications et optimiser les performances.


*WebDAV
Le WebDAV (Web Distributed Authoring and Versioning) est une extension du protocole HTTP qui permet la gestion des fichiers à distance dans notre serveur web.
Installez Apache et le module WebDAV :
sudo apt install apache2
sudo a2enmod dav
sudo a2enmod dav_fs















*Configuration WebDAV pour le dossier public :
 Pourquoi configurer WebDAV pour un dossier public ?
Partage de fichiers : WebDAV (Web Distributed Authoring and Versioning) permet de partager, éditer et gérer des fichiers à distance via HTTP/HTTPS, comme un système de stockage en réseau.
Accès universel : Un dossier public configuré en WebDAV devient accessible depuis n'importe quel appareil (ordinateur, smartphone) avec des clients compatibles (ex : Windows Explorer, macOS Finder, logiciels dédiés).
Collaboration : Idéal pour le travail d'équipe, car il permet à plusieurs utilisateurs de modifier des fichiers simultanément.


Édition du fichier /etc/apache2/sites-available/000-default.conf :
Alias /webdav /srv/public
<Directory /srv/public>
    DAV On
    Options Indexes FollowSymLinks
    AuthType Basic
    AuthName "WebDAV"
    AuthUserFile /etc/apache2/webdav.password
    Require valid-user
</Directory>






























Création un utilisateur WebDAV & Redémarrage Apache2 :
sudo htpasswd -c /etc/apache2/webdav.password user1








4. Configuration RAID (optionnel)
sudo apt install mdadm










sudo mdadm --create --verbose /dev/md0 --level=5 --raid-devices=3 /dev/sdb /dev/sdc /dev/sdd














sudo mkfs.ext4 /dev/md0










sudo mount /dev/md0 /mnt/raid







Lsblk




























5. Configuration Raid 5


Utilisation rsync pour sauvegarder les données sur un autre serveur :
rsync -avz /srv/public user@backup_server:/backup/




6. Sauvegarde (optionnel)


Utilisez rsync pour sauvegarder les données sur un autre serveur :
rsync -avz /srv/public user@backup_server:/backup/


Automatisation de la sauvegarde avec cron :
crontab -e


Ajoutez cette ligne pour une sauvegarde quotidienne à minuit :
0 0 * * * rsync -avz /srv/public user@backup_server:/backup/




7. Tests et vérifications
Teste de chaque service :
SFTP : Connectez-vous avec un client SFTP en utilisant les identifiants d'un utilisateur.
WebDAV : Accédez à http://adresse_IP/webdav via un navigateur ou un client WebDAV.
Dossier public : Vérification que les utilisateurs peuvent lire/écrire dans /srv/public.
Vérification des permissions des dossiers personnels et publics.








8. Virtualisation (optionnel)
Utilisation d’un hyperviseur comme KVM
sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager




9. Conclusion


Ce projet de serveur NAS sous Debian a permis de mettre en place une solution robuste et flexible pour le stockage et le partage de fichiers. En configurant des services comme SFTP, WebDAV et Samba, nous avons assuré un accès sécurisé et personnalisé pour plusieurs utilisateurs. La gestion des dossiers personnels et publics, ainsi que les options avancées comme le RAID et la sauvegarde, renforcent la fiabilité et la redondance du système. Une documentation complète a été élaborée pour faciliter l'utilisation et la maintenance du serveur. Ce NAS offre une solution complète pour les besoins de stockage et de collaboration en réseau.


