# Fedora-Silverblue-40-Install-Guide

By André Kurpanek
Erstellt am 08. September 2024. Aktualisiert am 09. September 2024

---

## Hinweise

Verweise:

- <https://www.markdownguide.org/basic-syntax/> "Marcdown Guide: Basic Syntax"
- <https://docs.fedoraproject.org/de/fedora-silverblue/> "Fedora Silverblue User Guide"
- <https://docs.fedoraproject.org/en-US/fedora-silverblue/tips-and-tricks/> " Fedora Silverblue Tips and Tricks"

---

## Noch zu Validieren

Verweise:

- [The Perfect Fedora Silverblue Setup For The XPS 13](https://schykle.medium.com/the-perfect-fedora-silverblue-setup-for-the-xps-13-90c4917003a4)

- [My Fedora Silverblue 40 Setup](https://gist.github.com/queeup/13c97bb059ca533ddca0ee12511c728f)

- [Configure Intel Arc A370M Xe-HPG discrete GPU on Linux](https://unrahul.quarto.pub/technical-ramblings/posts/2022-08-12-arc-dgpu-linux.html)

---

## Informatiomen

### Hardware



| <!-- -->            | <!-- -->                                                     |
| ---                 | ---                                                          |
| Hardwaremodell      | Lenovo Yoga 7 16IAH7 82UF                                    |
| Firmware-Version    | J1CN37WW                                                     |
| Speicher            | 16,0 GiB                                                     |
| Prozessor           | 12th Gen Intel® Core™ i7-12700H × 20                         |
| Grafik              | Intel® Arc™ A370M Graphics (DG2) / Intel® Graphics (ADL GT2) |
| Festlattenkapazität | 1,0 TB                                                       |

---

### Software

Folgende Software wird von mir  verwendet:

| Kategorie / Anwendungszweck | Software | Typ |
| --- | --- | --- |
| *Office** |   |   |
| Office-Paket | [LibreOffice](https://de.libreoffice.org/) | Flatpak |
| Markdown-Editor | [Typora](https://typora.io/) | Flatpak |
| PDF-Betrachter | GNOME Evince | Flatpak |
| PDF-Editor | [MASTER PDF Editor](https://code-industry.net/masterpdfeditor/) | Flatpak |
| Scannprogramm | [VueScan](https://www.hamrick.com/de/) | Flatpak |
| Code-Editor | [VSCode](https://code.visualstudio.com/) | Flatpak |
| **Internet** |   |   |
| Browser |   |   |
| E-Mail, Kontakte, Kalender, Aufgaben, Notizen | GNOME Evolution | Flatpak |
| **Multimedia** |   |   |
| Bildbearbeitung | [GIMP](https://www.gimp.org/) | Flatpak |
| Musikstreaming | [Spotify](https://open.spotify.com/) | Flatpak |
| Videoschnitt | [Blender](https://www.blender.org/) | Flatpak |
| **Tools** |   |   |
| AppImage | [Go AppImage](https://github.com/probonopd/go-appimage) | [AppImage](https://github.com/probonopd/go-appimage/releases/tag/continuous) |
| Passwortmanager | [Bitwarden](https://bitwarden.com/) | Flatpak |
| **Kommunikation** |   |   |
| Messaging | [Beeper](https://www.beeper.com/) | [AppImage](https://download.beeper.com/linux/appImage/x64) |

---

## OS Pre-Installation

---

## OS-Installation

Verweise: 

- <https://docs.fedoraproject.org/en-US/fedora-silverblue/installation/>

---

## OS Post-Installation

### Fedora Silverblue aktualisieren

```shell
# Check for new updates and download and install them
sudo rpm-ostree upgrade
```

Verweise:

- <https://docs.fedoraproject.org/en-US/fedora-silverblue/updates-upgrades-rollbacks/#updating>

---

### RPM Fusion Repositories aktivieren

```shell
# Install RPM Fusion repos
sudo rpm-ostree install --apply-live \
  https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm \
  https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```

```shell
# Unlock and update versioned packages
sudo rpm-ostree update \
  --uninstall rpmfusion-free-release \
  --uninstall rpmfusion-nonfree-release \
  --install rpmfusion-free-release \
  --install rpmfusion-nonfree-release
```

```shell
# Reboot again into new deployment
systemctl reboot
```

Verweise:

- <https://docs.fedoraproject.org/en-US/fedora-silverblue/tips-and-tricks/#_enabling_rpm_fusion_repos>

---

### Intel Media Driver for VAAPI

```shell
# Hardware codecs with Intel(recent)
rpm-ostree install intel-media-driver
```

```shell
# Reboot again into new deployment
systemctl reboot
```

Verweise:

- <https://rpmfusion.org/Howto/OSTree#Hardware_codecs_with_Intel.28recent.29>

---

### Intel 12. Screen flickering

```shell
# Disable Intel PSR
rpm-ostree kargs --append=i915.enable_psr=0
```

```shell
# If fixed, enable Intel PSR for future releases
#rpm-ostree kargs --delete=i915.enable_psr=0
```

```shell
# Reboot again into new deployment
systemctl reboot
```

Verweise:

- <https://wiki.archlinux.org/title/Intel_graphics#Screen_flickering>

---

### GuC / HuC Firmware aktivieren

```shell
# Alder Lake-P (Mobile) and newer
rpm-ostree kargs --append=i915.enable_guc=3
```

```shell
# Reboot again into new deployment
systemctl reboot
```

Verweise:

- <https://wiki.archlinux.org/title/Intel_graphics#Enable_GuC_/_HuC_firmware_loading>

---

### ALSA-Audio Hardware konfigurieren

Die Audio-Ausgabe hat einen blechernden und leisen Klang. Das Problem kann nmit folgendem Fix behoben werden.

```shell
# Apply tinny and quiet sound fix for bass speakers and internal microphone on Lenovo 7i
grep -iq '^options snd-sof-intel-hda-generic hda_model=alc287-yoga9-bass-spk-pin' \
  /etc/modprobe.d/snd.conf \
  || echo "options snd-sof-intel-hda-generic hda_model=alc287-yoga9-bass-spk-pin" \
  | sudo tee -a /etc/modprobe.d/snd.conf
```

Verweise:

- <https://wiki.archlinux.org/title/Lenovo_Yoga_9i_2022_(14AiPI7)#Audio>

---

### Flatpak einrichten

Verweise:

- <https://docs.fedoraproject.org/en-US/fedora-silverblue/getting-started/#flatpak>

---

#### AppImage Deamon einrichten

```shell
# Clear cache
rm "$HOME"/.local/share/applications/appimage*
[ -f ~/.config/systemd/user/default.target.wants/appimagelauncherd.service ] && rm ~/.config/systemd/user/default.target.wants/appimagelauncherd.service

# Optionally, install Firejail (if you want sandboxing functionality)

# Download
mkdir -p ~/Applications
wget -c https://github.com/$(wget -q https://github.com/probonopd/go-appimage/releases/expanded_assets/continuous -O - | grep "appimaged-.*-x86_64.AppImage" | head -n 1 | cut -d '"' -f 2) -P ~/Applications/
chmod +x ~/Applications/appimaged-*.AppImage

# Launch
~/Applications/appimaged-*.AppImage
```

Verweise:

- https://github.com/probonopd/go-appimage

---

### Toolbox einrichten

```shell
# Fedora 40 Toolbox erstellen
toolbox create
# Toolbox aktualisieren
podman image pull registry.fedoraproject.org/fedora-toolbox:40
```

---

### Distrobox einrichten

```shell
rpm-ostree install distrobox
```

### Bash einrichten

```shell
### Verzeichnis .bashrc.d anlegen
mkdir -p ~/.bashrc.d/
```

```shell
# Toolbox
cat > ~/.bashrc.d/10-toolbox.sh <<EOL
alias istoolbx='[ -f "/run/.toolboxenv" ] && grep -oP "(?<=name=\")[^\";]+" /run/.containerenv'
function is_toolbox() {
    if [ -f "/run/.toolboxenv" ]
    then
        TOOLBOX_NAME=$(cat /run/.containerenv | grep -oP "(?<=name=\")[^\";]+")
        echo "[${HOSTNAME} ${TOOLBOX_NAME}]"
    fi
}
EOL
```

---

## Software Installation

### Flatpak Applikationen installieren

```shell
# Fedora Repository
flatpak install -y fedora \
  org.mozilla.firefox \
  org.gnome.Evolution \
  org.gnome.Photos \
  org.gnome.Totem \
  org.gnome.Shotwell \
  org.libreoffice.LibreOffice \
  org.gimp.GIMP \
  org.blender.Blender
```

```shell
# Flathub Repository
flatpak install -y flathub \
  org.gnome.Boxes \
  io.typora.Typora \
  net.codeindustry.MasterPDFEditor \
  com.hamrick.VueScan \
  com.bitwarden.desktop \
  com.visualstudio.code \
  `#com.visualstudio.code.tool.podman` \
  `#com.visualstudio.code.tool.git-lfs` \
  com.google.Chrome \
  com.spotify.Client \
  org.localsend.localsend_app \
  com.github.flxzt.rnote \
  com.github.tchx84.Flatseal \
  org.gnome.World.PikaBackup
```

---

### Flatpak Applikationen konfigurieren

```shell
# GNOME Boxes: disable CoW for images
chattr +C ~/.var/app/org.gnome.Boxes/data/gnome-boxes/images
```

---

### AppImage Applikationen installieren

```shell
# Beeper chat app
curl -O -J -L --output-dir ~/Applications/  https://download.beeper.com/linux/appImage/x64
chmod +x ~/Applications/beeper*.AppImage
```

---

## Software Post-Installation

### Standardbrowser ausblenden (Firefox)

```shell
# Fedora 40 and later:
sudo mkdir -p /usr/local/share/applications/
sudo cp /usr/share/applications/org.mozilla.firefox.desktop /usr/local/share/applications/

grep -iq '^NotShowIn=GNOME;KDE' \
  /usr/local/share/applications/org.mozilla.firefox.desktop \
  || sudo sed -i "2a\\NotShowIn=GNOME;KDE" \
  /usr/local/share/applications/org.mozilla.firefox.desktop

sudo update-desktop-database /usr/local/share/applications/
```

---

### Firefox zurücksetzen

```shell
rm -rf ~/.mozilla/firefox
rm -rf ~/.cache/mozilla/firefox
```

Verweise:

- <https://docs.fedoraproject.org/en-US/fedora-silverblue/tips-and-tricks/#_hiding_the_default_browser_firefox>

---

## Änderungshistorie

