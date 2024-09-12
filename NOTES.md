# Fix Red Hat Bugzilla – Bug 1801539
sudo rpm-ostree kargs --append=rd.luks.options=discard
systemctl reboot


# Install RPM Fusion repos
sudo rpm-ostree install --assumeyes --apply-live \
  https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm \
  https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm

# Unlock and update versioned packages
sudo rpm-ostree update --apply-live \
  --uninstall rpmfusion-free-release \
  --uninstall rpmfusion-nonfree-release \
  --install rpmfusion-free-release \
  --install rpmfusion-nonfree-release
systemctl reboot


# Hardware codecs with Intel(recent)
sudo rpm-ostree install intel-media-driver
systemctl reboot


# ALSA Audio tinny and quiet sound fix for bass speakers and internal microphone
grep -iq '^options snd-sof-intel-hda-generic hda_model=alc287-yoga9-bass-spk-pin' \
  /etc/modprobe.d/snd.conf \
  || echo "options snd-sof-intel-hda-generic hda_model=alc287-yoga9-bass-spk-pin" \
  | sudo tee -a /etc/modprobe.d/snd.conf


# Switch to Tuned Power Manager
# Todo: Create /etc/tuned/ppd.conf before
sudo systemctl mask power-profiles-daemon.service
sudo rpm-ostree override remove power-profiles-daemon --install=tuned-ppd --install=tuned-gtk
systemctl reboot


### Verzeichnis .bashrc.d anlegen
mkdir -p ~/.bashrc.d/


# Fedora 40 Toolbox erstellen
toolbox create -y
podman image pull registry.fedoraproject.org/fedora-toolbox:40


# AppImage Deamon einrichten
rm "$HOME"/.local/share/applications/appimage*
[ -f ~/.config/systemd/user/default.target.wants/appimagelauncherd.service ] && rm ~/.config/systemd/user/default.target.wants/appimagelauncherd.service
mkdir -p ~/Applications
wget -c https://github.com/$(wget -q https://github.com/probonopd/go-appimage/releases/expanded_assets/continuous -O - | grep "appimaged-.*-x86_64.AppImage" | head -n 1 | cut -d '"' -f 2) -P ~/Applications/
chmod +x ~/Applications/appimaged-*.AppImage
~/Applications/appimaged-*.AppImage


#https://www.reddit.com/r/Fedora/comments/z2kk88/fedora_silverblue_replace_the_fedora_flatpak_repo/?tl=de
# Flatpak Repo Fedora auf Flathub umstellen
sudo flatpak remote-modify --no-filter --enable flathub
sudo flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
#flatpak install --reinstall flathub \
#    $( flatpak list --columns=application,origin | awk '/fedora$/' | awk '!/org.fedoraproject/' | awk '{print $1}' )
flatpak install --reinstall flathub \
    $( flatpak list --columns=application,origin | awk '/fedora$/' | awk '/org.gnome/' | awk '{print $1}' )


# Firefox RPM deaktiviern
sudo mkdir -p /usr/local/share/applications/
Fedora 40 and later:
sudo cp /usr/share/applications/org.mozilla.firefox.desktop /usr/local/share/applications/
sudo sed -i "2a\\NotShowIn=GNOME;KDE" /usr/local/share/applications/org.mozilla.firefox.desktop
sudo update-desktop-database /usr/local/share/applications/


# Flatpak Applikationen aus flathub installieren
flatpak install -y flathub \
  org.mozilla.firefox \
  org.gnome.Evolution \
  org.gnome.Photos \
  org.gnome.Totem \
  org.libreoffice.LibreOffice \
  org.gimp.GIMP \
  org.blender.Blender \
  org.gnome.Boxes \
  org.gnome.Boxes.Extension.OsinfoDb \
  org.gnome.Shotwell \
  org.gnome.World.PikaBackup \
  io.typora.Typora \
  net.codeindustry.MasterPDFEditor \
  com.hamrick.VueScan \
  `#com.bitwarden.desktop` \
  `#com.visualstudio.code` \
  `#com.visualstudio.code.tool.podman` \
  `#com.visualstudio.code.tool.git-lfs` \
  com.google.Chrome \
  com.spotify.Client \
  org.localsend.localsend_app \
  com.github.flxzt.rnote \
  com.github.tchx84.Flatseal \
  com.mattjakeman.ExtensionManager


# Custom Gnome Terminal for host
#
#Setup a new Profile named Fedora (--profile Fedora)
sudo cp /usr/share/applications/org.gnome.Terminal.desktop /usr/local/share/applications/
# Replace "Exec=gnome-terminal" with "Exec=gnome-terminal --profile Fedora"
sudo update-desktop-database /usr/local/share/applications/


##adw GT3 Support / Layer adw-gtk3
#rpm-ostree install gnome-tweaks adw-gtk3-theme --apply-live
#flatpak install -y org.gtk.Gtk3theme.adw-gtk3 org.gtk.Gtk3theme.adw-gtk3-dark
#gsettings set org.gnome.desktop.interface gtk-theme 'adw-gtk3' && gsettings set #org.gnome.desktop.interface color-scheme 'default'
##gsettings set org.gnome.desktop.interface gtk-theme 'adw-gtk3-dark' && gsettings set #org.gnome.desktop.interface color-scheme 'prefer-dark'
##gsettings set org.gnome.desktop.interface gtk-theme 'Adwaita' && gsettings set #org.gnome.desktop.interface color-scheme 'default'


# Gnome shell extension
#rpm-ostree override remove gnome-shell-extension-apps-menu gnome-classic-session gnome-classic-session-xsession gnome-shell-#extension-window-list gnome-shell-extension-background-logo gnome-shell-extension-launch-new-instance #gnome-shell-extension-places-menu

rpm-ostree override remove open-vm-tools-desktop open-vm-tools qemu-guest-agent spice-vdagent spice-webdavd virtualbox-guest-additions gnome-shell-extension-apps-menu gnome-classic-session gnome-classic-session-xsession gnome-shell-extension-window-list gnome-shell-extension-background-logo gnome-shell-extension-launch-new-instance gnome-shell-extension-places-menu --install gnome-tweaks


# rclone installation und setup
sudo -v ; curl https://rclone.org/install.sh | sudo bash


# VSCode Toolbox
toolbox run sudo dnf update -y
toolbox run sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
toolbox run sudo sh -c 'echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/vscode.repo'
toolbox run sudo dnf check-update
toolbox run sudo dnf install -y code
##-------------------------------------
#cat >$HOME/.local/share/applications/code.desktop<<EOF
#[Desktop Entry]
#Type=Application
#Version=1.0 # you can replace the version
#Name=Visual Studio Code
#Exec=toolbox run code
#Icon=com.visualstudio.code
#Terminal=false
#EOF
##-------------------------------------
cp code*.desktop /home/akurpanek/.local/share/applications/
mkdir -p /home/akurpanek/.local/share/icons/
cp /usr/share/pixmaps/vscode.png /home/akurpanek/.local/share/icons/
#replace 'Exec=code' mit 'Exec=toolbox run code'
#nano /home/akurpanek/.local/share/applications/code*.desktop
sudo update-desktop-database $HOME/.local/share/applications







