# linux kiosk / affichage publique
Une machine sous debian 10 qui fait tourner chromium en mode kiosk.
Il affiche une page web: Site internet, diaporama, images, vidéos...

### Installation
Installer un debian 10 (ou raspberryOS pour un raspberry)
Faire les actions de bases d'une installation (mise à jour, sécurisation, installation d'outils basique)
Une fois prêt on commence l'installation du kiosk.

```bash
sudo apt install chromium xserver-xorg-video-all xserver-xorg-input-all xserver-xorg-core xinit x11-xserver-utils can-utils vsftpd npm nodejs
```
On créer un utilisateur qui lancera le kiosk

```bash
useradd -m screen
```
Cette utilisateur n'a aucun droit sur le système, donc aucun risque qu'un utilisateur malveillant puisse modifier l'installation.

Pour commencer on va dire au gestionnaire d'utilisateur de lancer automatiquement le serveur d'affichage à la connexion uniquement si un écran est connecté et que la session et local (pour éviter un lancement en ssh).

```bash
sudo nano /home/screen/.profile

if  [ -z $DISPLAY ] &&  [ $(tty) = /dev/tty1 ]; then
    startx
fi
```

On configure .xinitrc pour lancer le kiosk

```bash
sudo nano /home/screen/.xinitrc

xset -dpms
xset s off
xset s noblank

chromium /home/screen/index.html --start-fullscreen --kiosk --incognito --noerrdialogs --disable-translate --no-first-run --fast --fast-start --disable-infobars --disable-features=TranslateUI --disk-cache-dir=/dev/null --password-store=basic
```
On crée une page web avec écrit Test

```bash
nano /home/screen/test.html

<h1>Test</h1>
```
Maintenant on va faire en sorte que le kiosk se connecte automatiquement à screen pour que le kiosk s'affiche même en cas de reboot.
Pour ça il faut modifier la ligne `#NAutoVTS=6`.

```bash
sudo nano /etc/systemd/logind.conf

NAutoVTs=1
```
Ensuite on va indiquer au system que faire au lancement pour la connexion automatique

```bash
sudo systemctl edit getty@tty1

[Service]
ExecStart=
ExecStart=-/sbin/agetty --autologin screen --noclear %I 38400 linux
```
Au moment d'enregister faite bien attention à ce que le fichier ait le chemin suivant: `/etc/systemd/system/getty@tty1.service.d/override.conf`.
Ensuite on active le programme: `sudo systemctl enable getty@tty1`
On peut reboot et après le démarrage, un écran avec écrit Test apparait.

Si des problème d'affichage (chromium qui n'occupe pas tout l'écran, chromium en petite fenêtre...) vous pouvez ajouter `--window-size=1920,1080 --window-position=0,0 ` après avant `--start-fullscreen`.
