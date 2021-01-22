# Verwendung/Administration:

Standardmässig wird das Webinterface auf Port 4455 zur Verfügung gestellt.

# Installation:

#TODO: Node.js installieren

--> https://github.com/josephdadams/TallyArbiter#installing-the-software

Den Code der aktuellen Version herunterladen:
```
git clone https://github.com/josephdadams/TallyArbiter.git tallyarbiter
```
Ins Verzeichnis wechseln:
```
cd tallyarbiter/
```
Benötigte Software-Abhängigkeiten installieren:
```
npm install
```

### Installation als Autostart-Daemon:

pm2 global installieren:
```
sudo npm install -g pm2
```
Autostart-Daemon mit dem Namen TallyArbiter erstellen:
```
pm2 start index.js --name TallyArbiter
```
Den Daemon anweisen, beim Start des Pis automatisch zu starten:
```
pm2 startup
```
