# Créer un fichier de pré-configuration Debian
Pour créer un fichier de pré-configuration pour l’installateur debian, la méthode la plus 
simple est de récupérer le template créé par Debian [ici](https://www.debian.org/releases/bookworm/example-preseed.txt). Le but de ce fichier est donc de 
pouvoir remplir automatiquement toutes les informations demandées par l’installateur. 
Les questions concernent le nom de la machine, la connexion au réseau, le 
partitionnement etc … 
Le fichier doit impérativement commencer par la ligne suivante : #_preseed_V1. Il est 
également important de bien nommer le fichier "preseed.cfg" afin de ne pas avoir 
d’erreurs. Au moment du boot, l’installateur va automatiquement chercher si un fichier 
preseed.cfg est présent dans le dossier initrd.gz autrement l’installation ne se lancera 
pas automatiquement. Pour pouvoir intégrer le fichier preseed dans l’image iso veuillez 
suivre les étapes de la création d’une image ISO personnalisée dans la doc [Debian
automatic Install](https://github.com/augustin840/preseed-debian/blob/main/modifier-iso.md) 
## Syntaxe du fichier preseed :
Le fichier de pré-configuration est au format utilisé dans la commande debconf-set-selections. Le format général d’un fichier de pré-configuration est le suivant : 
```html
<owner> <question name> <question type> <value>
```
Quelques règles pour éditer le fichier de pré-configuration de manière correcte :
- Ne mettre qu’un seul espace ou tabulation entre le type de question et la valeur, 
autrement l’installeur prendra le second espace/tab comme valeur.
- Une ligne peut être écrite sur plusieurs lignes différentes si elle est longue, pour cela 
il faut utiliser le caractère « \ » qui indiquera au d-i le retour à la ligne. Attention à le 
placer à un endroit stratégique, et non pas par exemple entre le type de valeur et la 
valeur.
- Pour les variables qui sont utilisées uniquement par le système d’installation, faire 
attention à bien spécifier « d-i » comme owner. (D-i pour debian-installer).
- La plupart des questions attendent une réponse avec des valeurs en anglais, pour 
éviter toutes confusion, il est bon de laisser tout en anglais par défaut. 
- Une ligne peut être commentée à l’aide du caractère « # »

## Création du fichier preseed : 
La manière la plus simple de faire un fichier preseed est de récupérer [l’exemple fournit par debian](https://www.debian.org/releases/bookworm/example-preseed.txt). Dans cet exemple, toutes les questions posées durant l’installation seront accessible depuis ce fichier. Autrement la seconde méthode est de récupérer la configuration actuelle d’un système fonctionnel et de la copier dans un fichier qui servira de preseed pour les installations suivantes. La manière à suivre pour cette option ci est la suivante : 
```bash
$ echo ”#_preseed_V1” > file
$ debconf-get-selections --installer >> file
$ debconf-get-selections >> file
```

Il y a également une commande utile pour vérifier la syntaxe du fichier preseed une fois fini afin d’éviter les erreurs directement durant l’installation :
```bash
debconf-set-selections -c preseed.cfg
```
## Principales questions à remplir:

En premier, il faut définir la locale. Une locale est utilisée pour définir à la fois une langue et un pays. Dans le fichier preseed il faut donc remplir les lignes suivantes :

```bash
# Préconfigurer la locale seule définit la langue, le pays et la locale.
d-i debian-installer/locale string en_US

# Les valeurs peuvent être préconfigurées individuellement.
#d-i debian-installer/language string en
#d-i debian-installer/country string NL
#d-i debian-installer/locale string en_GB.UTF-8
# On peut aussi demander la création d'autres locales.
#d-i localechooser/supported-locales multiselect en_US.UTF-8, fr_FR.UTF-8
```
La clavier peut également être configuré de la manière suivante:

```bash
# Choix du clavier :
d-i keyboard-configuration/xkb-keymap select fr(latin9)
# d-i keyboard-configuration/toggle select No toggling
```
Une fois le pays, la langue et le clavier définit, on peut maintenant configurer le réseau qui sera nécéssaire pour la suite du déploiement de l'os. Pluisieurs paramètres sont possibles pour configurer le réseau, cependant la manière la plus simple est d'utiliser le mode automatique qui cherchera si une interface est déjà connectée au réseau, telle que l'interface ethernet par exemple:

```bash
# Netcfg choisira une interface connectée si possible. Cela empêchera
# d'afficher une liste s'il y a plusieurs interfaces.
d-i netcfg/choose_interface select auto

# Pour utiliser une interface particulière :
#d-i netcfg/choose_interface select eth1
```

Autrement la configuration peut être entièrement manuelle:
```bash
# Par défaut, la configuration du réseau est automatique.
# Si vous préférez configurer vous-même le réseau, décommentez cette ligne
# et les lignes suivantes sur la configuration du réseau.
#d-i netcfg/disable_autoconfig boolean true

# Si vous voulez que le fichier de préconfiguration fonctionne aussi bien
# avec que sans serveur dhcp, décommentez ces lignes et les lignes sur la
# configuration du réseau. 
#d-i netcfg/dhcp_failed note
#d-i netcfg/dhcp_options select Configure network manually

# Configuration du réseau.
#
# exemple pour IPv4
#d-i netcfg/get_ipaddress string 192.168.1.42
#d-i netcfg/get_netmask string 255.255.255.0
#d-i netcfg/get_gateway string 192.168.1.1
#d-i netcfg/get_nameservers string 192.168.1.1
#d-i netcfg/confirm_static boolean true
#
# exemple pour IPv6
#d-i netcfg/get_ipaddress string fc00::2
#d-i netcfg/get_netmask string ffff:ffff:ffff:ffff::
#d-i netcfg/get_gateway string fc00::1
#d-i netcfg/get_nameservers string fc00::1
#d-i netcfg/confirm_static boolean true
```
On peut également définir le nom de la machine ainsi que le nom de domaine de la manière suivante:
```bash
# Remarquez que les valeurs données par DHCP, nom de domaine ou nom de 
# machine, prennent le pas sur les valeurs déclarées ici. Cependant,
# cette déclaration empêche que les questions ne soient posées, même si les
# valeurs viennent de dhcp.
d-i netcfg/get_hostname string unassigned-hostname
d-i netcfg/get_domain string unassigned-domain
```
L’étape suivante consiste à configurer les valeurs pour les miroirs afin de télécharger le système de base et les composants supplémentaires:
```bash
# Protocole pour les miroirs :
# Si vous utilisez ftp, il n'est pas nécessaire d'indiquer la chaîne
# mirror/country.
# Le protocole par défaut est http.
#d-i mirror/protocol string ftp
d-i mirror/country string manual
d-i mirror/http/hostname string http.us.debian.org
d-i mirror/http/directory string /debian
d-i mirror/http/proxy string

# Distribution à installer.
#d-i mirror/suite string testing
# Distribution à utiliser pour charger les composants de l'installateur
# (facultatif).
#d-i mirror/udeb/suite string testing
```
**Remarque:** Ajuster l'url du répo en fonction du lieu. La liste entière des répo officiels est disponible sur le site de Débian à [la page dédiée](https://www.debian.org/mirror/list.fr.html)

Une fois configurés, l’installateur passe au paramétrage des comptes root et utilisateurs. En fonction des besoins, il peut être utile de ne pas activer le compte root, dans ce cas, les simples utilisateurs pourront utiliser sudo pour l’élévation de droit. Ci-dessous quelques configurations possibles incluant le chiffrage des mots de passe directement dans le fichier de configuration pour éviter de laisser les mots de passe en clair:
```bash
# Ne pas créer de compte root (l'utilisateur ordinaire utilisera sudo).
#d-i passwd/root-login boolean false
# On peut aussi ne pas créer de compte d'utilisateur.
#d-i passwd/make-user boolean false

#Le mot de passe de root en clair...
#d-i passwd/root-password password r00tme
#d-i passwd/root-password-again password r00tme
# ... ou chiffré avec un hachage crypt(3)
#d-i passwd/root-password-crypted password [crypt(3) hash]


# Vous pouvez aussi présélectionner le nom de l'utilisateur et son
# identifiant de connexion
#d-i passwd/user-fullname string Utilisateur Debian
#d-i passwd/username string debian
# Mot de passe de l'utilisateur en clair...
#d-i passwd/user-password password insecure
#d-i passwd/user-password-again password insecure
# ... ou chiffré avec un hachage crypt(3)
#d-i passwd/user-password-crypted password [crypt(3) hash]
# Préciser l'UID du premier utilisateur.
#d-i passwd/user-uid string 1010

# Le compte sera ajouté à certains groupes. Pour contrôler ces groupes,
# utilisez cette ligne par exemple :
#d-i passwd/user-default-groups string audio cdrom video
```
Pour chiffrer les mots de passe, on peut utiliser la commande suivante:
```bash
mkpasswd -m sha-512
```
Il est également possible de configurer l'horloge de la manière suivante:
```bash
# Cette commande permet de régler l'horloge matérielle sur UTC :
d-i clock-setup/utc boolean true

# Vous pouvez mettre toute valeur acceptée pour $TZ.
# Voyez ce que contient /usr/share/zoneinfo/ pour les valeurs possibles.
d-i time/zone string Europe/Paris

# La ligne suivante autorise l'utilisation de NTP pour régler l'horloge
# pendant l'installation :
d-i clock-setup/ntp boolean true
# Le serveur NTP à utiliser. Le serveur par défaut est presque
# toujours correct.
#d-i clock-setup/ntp-server string ntp.example.com
```
Une étape très importante et délicate durant la configuration d’un os est le partitionnement de disque. Il peut être fait de trois manières différentes, à savoir 
-	Automatique
-	Guidé
-	Manuel

Pour certains cas il faut diviser les partitions en spécifiant des tailles précises, dans ce cas il faudra utiliser le mode manuel et créer une « recette de partitionnement » en utilisant l’utilitaire partman et sa syntaxe. Les informations essentielles pour créer une « recette » soi-même se trouvent sur le [répertoire source de l'installer Debian](https://salsa.debian.org/installer-team/debian-installer/-/blob/master/doc/devel/partman-auto-recipe.txt). L’exemple ci-dessous montre seulement les principales options cependant si vous devez monter un raid, lvm ou bien des partitions avancées, il faudra aller chercher les informations sur le repo fournit ci-dessus :

```bash
# Si le système possède un espace libre, vous pouvez ne partitionner que
# cet espace.
# Mais il faut que partman-auto/method (ci-dessous) ne soit pas définie.
#d-i partman-auto/init_automatically_partition select biggest_free

# Vous pouvez aussi choisir un disque entier. Si le système ne possède
# qu'un seul disque, l'installateur le choisira automatiquement. Si le
# système possède plusieurs disques, le nom du disque doit être
# donné selon le format traditionnel (par exemple, /dev/sda,
# mais pas /dev/discs/disc0/disc).
# Par exemple, pour utiliser le premier disque SCSI/SATA :
#d-i partman-auto/disk string /dev/sda
# Il faudra aussi indiquer la méthode à utiliser.
# Actuellement les méthodes disponibles sont :
# - regular : utilisation des types de partition habituels.
# - lvm :     utilisation de LVM pour le partitionnement du disque.
# - crypto :  utilisation de LVM à l'intérieur d'une partition chiffrée.
d-i partman-auto/method string lvm

# Vous pouvez définir la quantité d'espace qui sera utilisée
# par le groupe LVM.
# Cela peut être une taille associée à son unité (par exemple 20 GB),
# un pourcentage d'espace libre ou le mot "max".
d-i partman-auto-lvm/guided_size string max

# Si l'un des disques à partitionner automatiquement contient une ancienne
# configuration LVM, l'utilisateur recevra normalement un avertissement.
# Cet avertissement peut être évité :  
d-i partman-lvm/device_remove_lvm boolean true
# De même pour un Raid logiciel existant déjà :
d-i partman-md/device_remove_md boolean true
# Et aussi pour la confirmation concernant la création de partitions lvm :
d-i partman-lvm/confirm boolean true
d-i partman-lvm/confirm_nooverwrite boolean true

# Vous pouvez choisir l'un des trois schémas prédéfinis...
# - atomic : tous les fichiers dans une seule partition
# - home :   partition /home distincte
# - multi :  partitions /home, /var et /tmp séparées
d-i partman-auto/choose_recipe select atomic

# ...ou donner vous-même votre schéma.
# Si vous avez la possibilité d'obtenir un schéma dans l'environnement de
# l'installateur, vous pouvez simplement pointer sur lui.
#d-i partman-auto/expert_recipe_file string /hd-media/recipe

# Sinon, vous pouvez mettre un schéma dans le fichier de préconfiguration
# (une seule ligne logique). L'exemple suivant crée une petite partition
# /boot, une partition swap convenable, et utilise le reste de l'espace libre
# pour la partition racine :
#d-i partman-auto/expert_recipe string                         \
#      boot-root ::                                            \
#              40 50 100 ext3                                  \
#                      $primary{ } $bootable{ }                \
#                      method{ format } format{ }              \
#                      use_filesystem{ } filesystem{ ext3 }    \
#                      mountpoint{ /boot }                     \
#              .                                               \
#              500 10000 1000000000 ext3                       \
#                      method{ format } format{ }              \
#                      use_filesystem{ } filesystem{ ext3 }    \
#                      mountpoint{ / }                         \
#              .                                               \
#              64 512 300% linux-swap                          \
#                      method{ swap } format{ }                \
#              .

# Une documentation complète sur le format des schémas se trouve dans le
# fichier partman-auto-recipe.txt, disponible dans le
# paquet « debian-installer » ou dans les sources de l'installateur.
# On trouve aussi dans ce document la manière d'indiquer les étiquettes
# de systèmes de fichiers, les noms de groupes de volumes ainsi que les
# noms de périphériques physiques à inclure dans les groupes de volumes.

## Partitionnement pour EFI
# Si votre système nécessite une partition EFI, vous pouvez ajouter ceci au
# schéma précédent, en tant que premier élément :
#               538 538 1075 free                              \
#                      $iflabel{ gpt }                         \
#                      $reusemethod{ }                         \
#                      method{ efi }                           \
#                      format{ }                               \
#               .                                              \
#
# Le fragment ci-dessus correspond à l'architecture amd64, 
# certains détails peuvent varier en fonction des architectures.
# Le paquet « partman-auto » dans le dépôt de sources de D-I
# peut contenir des exemples à adapter.

# Si vous avez indiqué la méthode à utiliser, partman créera automatiquement
# les partitions sans demander de confirmation.
d-i partman-partitioning/confirm_write_new_label boolean true
d-i partman/choose_partition select finish
d-i partman/confirm boolean true
d-i partman/confirm_nooverwrite boolean true

# Forcer l'amorçage avec UEFI (la compatibilité avec le BIOS sera perdue),
# cela n'est pas fait par défaut.
#d-i partman-efi/non_efi_system boolean true
# S'assurer que la table de partitionnement est en GPT,
# cela est nécessaire pour l'EFI.
#d-i partman-partitioning/choose_label select gpt
#d-i partman-partitioning/default_label string gpt

# Quand le chiffrage de disque est activé,
# ne pas effacer les partitions avant.
#d-i partman-auto-crypto/erase_disks boolean false
```

Pour les raid:
```bash
# La méthode à indiquer est "raid".
#d-i partman-auto/method string raid
# Indiquez les disques à partitionner. Ils auront tous les mêmes
# caractéristiques, et donc cela ne fonctionnera que s'ils ont tous
# la même taille.
#d-i partman-auto/disk string /dev/sda /dev/sdb

# Ensuite, indiquez les partitions physiques à utiliser. 
#d-i partman-auto/expert_recipe string \
#      multiraid ::                                         \
#              1000 5000 4000 raid                          \
#                      $primary{ } method{ raid }           \
#              .                                            \
#              64 512 300% raid                             \
#                      method{ raid }                       \
#              .                                            \
#              500 10000 1000000000 raid                    \
#                      method{ raid }                       \
#              .

# Enfin vous devez indiquer comment seront utilisées les partitions que
# vous venez de définir. N'oubliez pas de donner les bons numéros pour
# les partitions logiques. Les niveaux RAID 0, 1, 5, 6 et 10 sont acceptés.
# Les noms des périphériques sont séparés par un caractère « # ». 
# Paramètres :
# <raidtype> <devcount> <sparecount> <fstype> <mountpoint> \
#          <devices> <sparedevices>

#d-i partman-auto-raid/recipe string \
#    1 2 0 ext3 /                    \
#          /dev/sda1#/dev/sdb1       \
#    .                               \
#    1 2 0 swap -                    \
#          /dev/sda5#/dev/sdb5       \
#    .                               \
#    0 2 0 ext3 /home                \
#          /dev/sda6#/dev/sdb6       \
#    .

# Une documentation complète se trouve dans le
# fichier partman-auto-raid-recipe.txt, disponible dans le
# paquet « debian-installer » ou dans les sources de l'installateur.

# Pour que partman partitionne automatiquement sans demander de confirmation :
d-i partman-md/confirm boolean true
d-i partman-partitioning/confirm_write_new_label boolean true
d-i partman/choose_partition select finish
d-i partman/confirm boolean true
d-i partman/confirm_nooverwrite boolean true
```
Le setup de /etc/apt/source.list est normalement entièrement automatisé en fonction des réponses aux précédentes questions, cependant quelques paramètres supplémentaires peuvent être ajoutés, à savoir:
```bash
# Choisissez si vous voulez analyser un autre support d’installation
# (par défaut : non (false)).
d-i apt-setup/cdrom/set-first boolean false
# Vous pouvez installer des microprogrammes non libres.
#d-i apt-setup/non-free-firmware boolean true
# Vous pouvez installer des logiciels des distributions non-free et contrib.
#d-i apt-setup/non-free boolean true
#d-i apt-setup/contrib boolean true
# Décommentez cette ligne si vous ne souhaitez pas qu'une image d'installation
# DVD ou BD soit active dans le sources.list du système installé
# (les images de CD ou netinst sont désactivées
# quelque soit la valeur de ce paramètre)
#d-i apt-setup/disable-cdrom-entries boolean true
# Décommentez cette ligne si vous n'utilisez pas de miroir sur le réseau.
#d-i apt-setup/use_mirror boolean false
# Choisissez les services de mise à jour et les miroirs à utiliser.
# Les valeurs ci-après sont les valeurs par défaut :
#d-i apt-setup/services-select multiselect security, updates
#d-i apt-setup/security_host string security.debian.org

# Autres sources disponibles, local[0-9]
#d-i apt-setup/local0/repository string \
#       http://local.server/debian stable main
#d-i apt-setup/local0/comment string local server
# Activer des lignes deb-src
#d-i apt-setup/local0/source boolean true
# URL de la clé publique de la source locale. Vous devez indiquer une clé ; sinon
# apt se plaindra que la source n'est pas authentifiée et laissera la ligne du
# fichier sources.list en commentaire.
#d-i apt-setup/local0/key string http://local.server/key
# ou vous pouvez fournir cette clé en chiffrant son contenu avec
# la commande `base64 -w0` et en l'écrivant ainsi :
#d-i apt-setup/local0/key string base64://LS0tLS1CRUdJTiBQR1AgUFVCTElDIEtFWSBCTE9DSy0tLS0tCi4uLgo=
# Le fichier de clé fourni est vérifié. S'il est au format PGP ASCII avec armure,
# il sera enregistré avec l'extension « .asc »,
# sinon l'extension « .gpg » sera utilisée.
# Le format « keybox database » n'est pas encore pris en charge.
# (consultez generators/60local dans les sources de apt-setup)

# Par défaut, l'installateur demande que les dépôts soient authentifiés par
# une clé gpg connue. On peut se servir de cette commande pour désactiver
# cette authentification.
# Attention : cette commande n'est pas sécurisée ni recommandée.
#d-i debian-installer/allow_unauthenticated boolean true

# Décommentez pour ajouter la configuration multiarch pour i386
#d-i apt-setup/multiarch string i386
```
Ensuite, vient l’installation des packages supplémentaires tels que :
-	Standard (outils standards)
-	Desktop (graphical desktop)
-	Gnome-desktop (Gnome desktop)
-	Xfce-desktop (xfce desktop)
-	Kde-desktop
-	Cinnamon-desktop
-	Mate-dektop
-	Lxde-desktop
-	Web-server
-	Ssh-server

Attention, il est recommandé d’installer au moins le package standard. Cela peut être fait de la manière suivante :
```bash
#tasksel tasksel/first multiselect standard, web-server, kde-desktop

# Ou choisissez de ne pas afficher les écrans de tasksel
# (ce qui n'installe aucun paquet)
#d-i pkgsel/run_tasksel boolean false

# Paquets supplémentaires
#d-i pkgsel/include string openssh-server build-essential
# Mise à jour des paquets après debootstrap.
# Valeurs autorisées : none, safe-upgrade, full-upgrade
#d-i pkgsel/upgrade select none

# Vous pouvez choisir si vous souhaitez signaler les logiciels que
# vous avez installés et ceux que vous utilisez. Par défaut, rien n'est signalé.
# Mais l'envoi de rapport d'installation aide le projet à connaître les logiciels
# populaires qui devraient être inclus sur les premiers CD et DVD.
#popularity-contest popularity-contest/participate boolean false
```
Il y a également l’étape de l’installation du bootloader où il faudra définir : 
-	Si on veut l’installer avec une partition record si jamais il rencontre un autre os installé durant l’installation
-	La partition où il faut l’installer 
-	Si on veut également configurer un mot de passe soit en clair ou bien avec un hash. 

```bash
# Grub is the boot loader (for x86).
# This is fairly safe to set, it makes grub install automatically to the UEFI
# partition/boot record if no other operating system is detected on the machine.
d-i grub-installer/only_debian boolean true
# This one makes grub-installer install to the UEFI partition/boot record, if
# it also finds some other OS, which is less safe as it might not be able to
# boot that other OS.
d-i grub-installer/with_other_os boolean true
# Due notably to potential USB sticks, the location of the primary drive can
# not be determined safely in general, so this needs to be specified:
#d-i grub-installer/bootdev string /dev/sda
# To install to the primary device (assuming it is not a USB stick):
#d-i grub-installer/bootdev string default
# Alternatively, if you want to install to a location other than the UEFI
# parition/boot record, uncomment and edit these lines:
#d-i grub-installer/only_debian boolean false
#d-i grub-installer/with_other_os boolean false
#d-i grub-installer/bootdev string (hd0,1)
# To install grub to multiple disks:
#d-i grub-installer/bootdev string (hd0,1) (hd1,1) (hd2,1)
# Optional password for grub, either in clear text
#d-i grub-installer/password password r00tme
#d-i grub-installer/password-again password r00tme
# or encrypted using an MD5 hash, see grub-md5-crypt(8).
#d-i grub-installer/password-crypted password [MD5 hash]
# Use the following option to add additional boot parameters for the
# installed system (if supported by the bootloader installer).
# Note: options passed to the installer will be added automatically.
#d-i debian-installer/add-kernel-opts string nousb
```
La dernière étape consiste à configurer des paramètres mineurs, tels que le redémarrage automatique durant l'installation, l'activation des consoles vty durant l'isntallation etc ...

```bash
# Lors d'une installation à partir d'une console série, les consoles virtuelles
# (VT1-VT6) sont désactivées dans /etc/inittab. Décommentez la ligne suivante
# pour empêcher la désactivation.
#d-i finish-install/keep-consoles boolean true

# Pour éviter le dernier message disant que l'installation est terminée :
d-i finish-install/reboot_in_progress note

# Pour empêcher l'éjection du CD au moment du redémarrage,
# c'est utile parfois :
#d-i cdrom-detect/eject boolean false

# Pour arrêter l'installateur quand il a terminé, mais sans redémarrer
# le système installé :
#d-i debian-installer/exit/halt boolean true
# Pour éteindre la machine au lieu de simplement l'arrêter :
#d-i debian-installer/exit/poweroff boolean true
```


**Liens utiles:**

[Debian GNU/Linux Installation Guide](https://www.debian.org/releases/stable/amd64/install.en.pdf)

[Debian Installer / debian-installer - GitLab](https://salsa.debian.org/installer-team/debian-installer)

[exemple fichier preseed](https://www.debian.org/releases/bookworm/example-preseed.txt)

[Debian -- Manuels pour les développeurs Debian](https://www.debian.org/doc/devel-manuals)

[Debian Preseed with Encrypted LVM - GitHub](https://gist.github.com/chuckn246/ca24d26c048b3cc4ffa8188708f5dccf)