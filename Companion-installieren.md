# 0. Ein ARM64-image auf die SD-Karte flashen:

* Entweder das Rasperrian mit Desktopoberfläche: https://downloads.raspberrypi.org/raspios_arm64/images/
* oder die Headless-Variante (nur mit Konsole): https://downloads.raspberrypi.org/raspios_lite_arm64/images/

Empfohlen ist die Headless-Variante, da nur die Installation in der Konsole erfolgt. Die Konfiguration des Companion erfolgt über die Weboberfläche. Eine Desktopoberfläche kann auch nachträglich noch installiert werden.

#TODO: how to flash an SD-Card

# 1. Passwort ändern und Netzwerkzugriff sicherstellen.

Den Raspberry mit Bildschirm, Tastatur, Maus und Companion anschliessen. (Companion kann auch bereits angeschlossen werden, braucht es aber in diesem Schritt noch nicht). Wenn das Gerät via LAN im Netzwerk erreichbar sein soll, dieses ebenfalls bereits anschliessen.

In der Raspbery Konsole anmelden, Standard-Login ist User: pi, Passwort: raspberry

Anschliessend mit `sudo raspi-config` das Konfigurationsmenü aufrufen und 1) das Passwort ändern, sowie 2) unter "Configure Network Interface" das SSH-Interface aktivieren. Falls der Companion via WLAN im Netzwerk sein soll, muss dieses ebenfalls im Konfigurationsmenü aktiviert werden. Das Konfigurationsmenü anschliessend verlassen.

Die IP-Adresse(n) können mit `ip addr show` abgerufen werden. Bei einer Verbindung via LAN ist diese unter "eth0" zu finden, mit WLAN unter "#TODO". Die Adresse, mit welcher der Raspberry in Zukunft erreichbar sein wird. Diese Adresse notieren oder merken.

Anschliessend `sudo reboot`. Bildschirm, Tastatur und Maus können anschliessend abgesteckt werden.

# 2. Remote auf den Raspberry zugreifen.

Unter Windows ein Programm wie putty verwenden, und als Ziel die IP-Adresse des Raspberry angeben.
Unter Linuxartigen-Betriebssystemen wird die verbindung mit `ssh pi@<<IP-Adresse>>` hergesstellt. Wenn der Nutzername bereits geändert wurde, muss dies auch im ssh-Command geändert werden: `ssh <<user>>@<<IP-Adresse>>`.

# 3. Updates und Software installieren:

Paketliste aktualisieren:
```
sudo apt update
```
Update durchführen:
```
sudo apt dist-upgrade -y
```
Zusätzliche Software installieren, welche im späteren Prozess gebraucht wird:
```
sudo apt-get install -y gcc g++ make libgusb-dev libudev-dev libusb-1.0-0-dev build-essential cmake git pkg-config libglib2.0-0 libexpat1-dev
```
Veraltete Pakete entfernen:
```
sudo apt autoclean && sudo apt autoremove
```
Raspberry neu starten. Die Verbindung zum Raspberry muss nach kurzem warten neu hergestellt werden (siehe Schritt 2)
```
sudo reboot
```
# 4. Raspberry Firmware updaten (muss nicht bei jeder Betriebssystem installation gemacht werden).

Paketliste aktualisieren und Updates installieren:
```
sudo apt update && sudo apt upgrade
```
Firmware Update aktivieren:
```
sudo sed -i -e 's/^FIRMWARE_RELEASE_STATUS=.*/FIRMWARE_RELEASE_STATUS=beta/' /etc/default/rpi-eeprom-update
```
Firmware Update durchführen:
```
sudo rpi-eeprom-update -a
```
Anschliessend Raspberry neu starten. Die Verbindung zum Raspberry muss nach kurzem warten neu hergestellt werden (siehe Schritt 2)
```
sudo reboot
```

# 5. Raspberry System-Härtung:

#TODO

# 6. Node.JS installieren:

Node.JS 14 (derzeitige LTS-Version) herunterladen und ausführbar deklarieren:
```
curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -
```
Anschliessend installieren:
```
sudo apt-get install -y nodejs
```

# 7. Yarn installieren

GPG-Key der Yarn Paketliste herunterladen und akzeptieren:
```
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
```
Yarn-Paketliste herunterladen und aktivieren:
```
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
```
Anschliessend installieren:
```
sudo apt update && sudo apt install yarn
```

# 8. VIPS manuell installieren:
Aktuelle Version herunterladen:
```
wget https://github.com/libvips/libvips/releases/download/v8.10.5/vips-8.10.5.tar.gz
```
Entpacken:
```
tar xf vips-8.10.5.tar.gz
```
In das heruntergeladene Quellcode-Verzeichnis wechseln:
```
cd vips-8.10.5/
```
Konfigurieren: #TODO: eventuell müssen zusätzliche Pakete hier Konfiguriert werden um einzelne Funktionen zu ermöglichen. Das weiss ich aber noch nicht.
```
./configure
```
Paket bauen (dauert ca 20 Minuten):
```
sudo make -j3
```
Paket installieren:
```
sudo make install
```
Aktivieren:
```
sudo ldconfig
```

# 9. UDEV Regeln hinzufügen um den Zugriff auf das USB-Device (Streamdeck) zu erlauben.

Konfiguration mit `sudo nano /etc/udev/rules.d/50-companion.rules` erstellen und öffnen und den folgenden Text hineinkopieren (CTRL-SHIFT-V).

```
SUBSYSTEM=="input", GROUP="input", MODE="0666"
SUBSYSTEM=="usb", ATTRS{idVendor}=="0fd9", ATTRS{idProduct}=="0060", MODE:="666", GROUP="plugdev"
KERNEL=="hidraw", ATTRS{idVendor}=="0fd9", ATTRS{idProduct}=="0060", MODE:="666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="ffff", ATTRS{idProduct}=="1f40", MODE:="666", GROUP="plugdev"
KERNEL=="hidraw", ATTRS{idVendor}=="ffff", ATTRS{idProduct}=="1f40", MODE:="666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="0fd9", ATTRS{idProduct}=="0063", MODE:="666", GROUP="plugdev"
KERNEL=="hidraw", ATTRS{idVendor}=="0fd9", ATTRS{idProduct}=="0063", MODE:="666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="0fd9", ATTRS{idProduct}=="006c", MODE:="666", GROUP="plugdev"
KERNEL=="hidraw", ATTRS{idVendor}=="0fd9", ATTRS{idProduct}=="006c", MODE:="666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="0fd9", ATTRS{idProduct}=="006d", MODE:="666", GROUP="plugdev"
KERNEL=="hidraw", ATTRS{idVendor}=="0fd9", ATTRS{idProduct}=="006d", MODE:="666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="ffff", ATTRS{idProduct}=="1f41", MODE:="666", GROUP="plugdev"
KERNEL=="hidraw", ATTRS{idVendor}=="ffff", ATTRS{idProduct}=="1f41", MODE:="666", GROUP="plugdev"
```
Anschliessend mit `CTRL-X` schliessen anregen und mit `SHIFT-Y` das speichern bestätigen.

UDEV-Regeln neu laden:
```
sudo udevadm control --reload-rules
```

# 10. Node.JS 12 installieren (Companion kompatible version)

n (Paketmanager) installieren: 
```
sudo npm install n -g
```
node.js installieren:
```
sudo n 12.19.1
```
Ins Home-Verzeichnis wechseln:
```
cd ~
```
Companion herunterladen:
```
git clone https://github.com/bitfocus/companion.git
```
In das Companion Verzeichnis wechseln:
```
cd companion/
```
Sharp bereitstellen:
```
npm install sharp
```
Mit Yarn submodule installieren:
```
yarn update
```
und Companion bereitstellen:
```
./tools/build_writefile.sh
```

# 11. Autostart Konfigurieren:

Mit `sudo nano /etc/systemd/system/companion.service` Konfigurationsdatei erstellen und öffnen und den folgenden Text hineinkopieren (CTRL-SHIFT-V).

```
[Unit]
Description=Bitfocus Companion (v2.0.0)
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=pi
WorkingDirectory=/home/pi/companion
ExecStart=node /home/pi/companion/headless_ip.js 0.0.0.0
Restart=on-failure
KillSignal=SIGINT
TimeoutStopSec=60

[Install]
WantedBy=multi-user.target
```
Anschliessend mit `CTRL-X` schliessen anregen und mit `SHIFT-Y` das speichern bestätigen.

Service im System aktivieren:
```
sudo systemctl enable companion.service
```
Gerät neu starten
```
sudo reboot
```

# 11. Companion Konfigurieren

Nach kurzem Warten sollte die Web Oberfläche unter der gemerkten IP-Adresse auf Port 8000 erreichbar sein. Dazu im Browser <<IP-Adresse>>:8000 eingeben.
