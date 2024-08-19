# DOCUMENTATION DEBIAN AUTOMATIC INSTALL
## Automatisation de l’installation de debian à l’aide du fichier de pré configuration.
Il est possible d’automatiser l’installation de debian en utilisant un fichier de pré 
configuration nommé preseed.cfg. Ce fichier permet de répondre à toutes (ou presque 
toutes) les questions posées par l’installateur debian lors de l’installation. Un template 
de ce fichier est disponible ici. Pour charger ce fichier, trois options sont possibles :
- Via le réseau
- Via usb média
- Via ISO
Au moment du boot l’utilisateur doit sélectionner automated (graphical) install.
- Install vie le réseau : attendre que l’utilisateur soit invité à sélectionner une 
interface. Une fois connecté au réseau l’utilisateur doit rentrer également le 
chemin du fichier preseed.cfg (exemple : tftp://host/path/to/preseed.cfg). Une 
fois chargé, le fichier va suivre les indications présentes dans le fichier. La 
deuxième méthode pour donner le fichier preseed.cfg au debian installer est de 
spécifier directement le chemin d’accès au fichier dans le menu grub en cliquant 
sur e et en rajoutant à la ligne linux url=/path/to/preseed.cfg auto=true. L’option 
auto permets de charger le fichier de pré configuration avant le boot pour que 
certains paramètres, tels que la langue, la location ainsi que la disposition du 
clavier soient directement prises en charge. Il est également possible de 
spécifier certains paramètres au même moment que lorsque l’on donne le fichier 
preseed. 
- Install via fichier de pré-configuration : Pour cette méthode ci, il faut choisir 
automated install ou graphical automated install. Vous devrez donc sélectionner
la location, le langage du système, la configuration du clavier ainsi que les 
locales avant d’être invité à rentrer le chemin d’accès au fichier de pré-configuration (preseed). Une fois le fichier chargé, l’installation devrait être 
entièrement automatique pour la suite de l’installation. 
- Création d’une image personnalisée : La manière la plus « simple » et plus rapide 
pour l’automatisation de l’installation est de rajouter le fichier preseed.cfg 
directement dans les fichiers de l’image ISO. Pour cela, il faut suivre la démarche 
suivante :
1. Créer son fichier de pré-configuration et faire attention à bien le nommer 
« preseed.cfg ». Il est également possible de vérifier la syntaxe du fichier 
en utilisant la commande suivante dans un terminal linux : debconf-set-
selections -c preseed.cfg. Une fois fait, on peut passer à la modification 
de l’image ISO.
2. Modification de l’image :
a. Il faut d’abord monter l’image officielle à l’aide de la commande : 
```bash
udevil mount 
debian-10.2.0-i386-netinst.iso
```
(s’assurer d’avoir le package udevil installé 
auparavant.
b. Ensuite on copie tous les fichiers qui ont été montés dans un nouveau dossier 
isofiles : 
```bash
cp -rT /media/debian-10.2.0-i386-netinst.iso/ isofiles/
```
c. Une fois fait, pour modifier le fichier initrd.gz qui est protégé en écriture, on 
utilise : 
```bash
chmod +w -R isofiles/install.amd/
```
d. On peut donc maintenant dezipper le fichier initrd.gz qui n’est plus protégé en 
écriture. Le dossier initrd.gz va contenir tous les fichiers utiles lors du boot, c’est 
donc ici qu’on va placer notre fichier de preseed. Pour extraire le fichier on utilise 
l’utilitaire « gunzip » : 
```bash
gunzip isofiles/install.amd/initrd.gz
```
e. Ensuite on va pouvoir placer le fichier preseed.cfg dans la racine de initrd en 
utilisant la commande suivante : 
```bash
echo preseed.cfg | cpio -H newc -o -A -F isofiles/install.amd/initrd
```
f. Une fois fait, un peut compresser à nouveau le fichier initrd en utilisant la 
commande : 
```bash
gzip isofiles/install.amd/initrd
```
g. On peut maintenant enlever les droits d’écriture du dossier install.amd en 
utilisant : 
```bash
chmod -w -R isofiles/install.amd/
```
h. Il faut maintenant regénérer la somme de contrôle du dossier afin d’assurer 
l’intégriter des fichiers. Il faut donc utiliser les commandes suivantes :
```bash
cd isofiles
chmod +w md5sum.txt
find -follow -type f ! -name md5sum.txt -print0 | xargs -0 md5sum > md5sum.txt
chmod -w md5sum.txt
```
i. Une fois l’opération terminée, il faut recréer une nouvelle image ISO bootable. 
Pour faire ceci, nous allons utiliser un utilitaire dédié nommé genisoimage. Il 
faudra donc exécuter la commande:
```bash 
genisoimage -r -J -b isofiles/isolinux/isolinux.bin -c isofiles/isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -o preseed-debian-10.2.0-i386-netinst.iso isofiles
```
j. Pour finir, pour restaurer l’iso a un état bootable, il faut utiliser l’utilitaire 
isohybride inclus dans le package syslinux-utils : 
```bash
isohybrid name_of_your_file.iso
```
k. Le nouveau fichier est peut donc être copié sur une clé usb et nous pouvons 
booter dessus.
3. Booter sur la clé usb contenant le nouvel ISO. Une fois dans le menu grub, 
il faut seulement sélectionner l’option install et l’installation de debian se 
fera automatiquement.
Exemples de paramètres utils à utiliser dans les paramètres de boot.