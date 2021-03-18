## Nginx + RTMP Modul installieren

## Standard Konfiguration(en)

## Recordings in MP4 umwandeln

Die direkte Aufnahme als MP4 wird [von diesem Modul derzeit nicht unterstützt](https://github.com/arut/nginx-rtmp-module/issues/957).
Als Alternative wird ein Hook mit `record end` angesprochen, welcher das erzeugte fla-File mit ffmpeg umwandelt, sobald die Aufnahme abgeschlossen ist.
Eine entsprechende Zeile muss dazu im rtmp Abschnitt der Webservice Konfigration angefügt werden.
([-> Wiki](https://github.com/arut/nginx-rtmp-module/wiki/Directives#exec_record_done))

```sh
exec_record_done ffmpeg -y -i $path -acodec libmp3lame -ar 44100 -ac 1 -vcodec libx264 $dirname/$basename.mp4;
```

Gesamtes Beispiel:
```json

...
rtmp {
  server {
    listen 1935;

    application live {
      # Disable livestreaming
      live off;

      # Enable recording
      record all; 
      record_path /path/to/directory/; 
      record_suffix -%H-%M-%d-%m-%y.flv;
      
      # Allow only this machine to play back the stream
      allow play 127.0.0.1;
      deny play all;

      # Convert the recording
      exec_record_done ffmpeg -y -i $path -acodec libmp3lame -ar 44100 -ac 1 -vcodec libx264 $dirname/$basename.mp4;
    }
  }
}
...

```

## Recording manuell starten

