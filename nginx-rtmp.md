## Nginx + RTMP Modul installieren

## Standard Konfiguration(en)

## Recordings in MP4 umwandeln

Die direkte Aufnahme als MP4 wird [von diesem Modul derzeit nicht unterstützt](https://github.com/arut/nginx-rtmp-module/issues/957).
Als Alternative wird ein Hook mit `record end` angesprochen, welcher das erzeugte fla-File mit ffmpeg umwandelt, sobald die Aufnahme abgeschlossen ist.
Eine entsprechende Zeile muss dazu im rtmp Abschnitt der Webservice Konfigration angefügt werden. Die Variablen sind dabei in `exec_record_done` integriert ([siehe Wiki](https://github.com/arut/nginx-rtmp-module/wiki/Directives#exec_record_done))

```sh
exec_record_done ffmpeg -y -i $path -acodec libmp3lame -ar 44100 -ac 1 -vcodec libx264 $dirname/$basename.mp4;
```

Gesamtes Beispiel:
```

...
rtmp {
  server {
    listen 1935;

    application live {
      
      live on;

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

Das Recording kann manuell durch ein Server-Script, oder eine entsprechende HTTP-Direktive kontrolliert werden. Dazu wird ein Webserver geöffnet und das RTMP-Modul angewiesen, Aufzeichnungen manuell zu starten. ([-> Wiki](https://github.com/arut/nginx-rtmp-module/wiki/Control-module)). Ebenfalls **muss** das RTMP Modul einen `application` Container aufweisen, da der HTTP Request sonst nicht an die richtige Stelle verwiesen werden kann (im Beispiel unten ist der application Container _live_ benannt).

Das RTMP Modul wird wie folgend angepasst:
```
record all manual; 
```

Die Control-Direktive im Webserver lautet:
```
rtmp_control all; 
```


Gesamtes Beispiel:

```
...

http {
  server {
    listen 8080;
    server_name localhost;
    location /control {
      rtmp_control all;
    }
  }
}

rtmp {
  server {
    listen 1935;

    application live {
    
      live on;

      # Enable recording, but only manual:
      record all manual; 
      record_path /path/to/directory/; 
      record_suffix -%H-%M-%d-%m-%y.flv;
      
      # Allow only this machine to play back the stream
      allow play 127.0.0.1;
      deny play all;
    }
  }
}
...

```

Das Recording wird in diesem Beispiel mit `http://<ip-address>:8080/control/record/start?app=live&name=mystream` im Browser (oder via curl/wget) gestartet, bzw äquivalent mit `http://<ip-address>:8080/control/record/start?app=live&name=mystream` gestoppt. Mit `name` wird hierbei der Streamkey benannt (in diesem Fall _mystream_)!

Es ist auch möglich mehrere `recorder` anzulegen, und diese unabhängig voneinander zu starten/stoppen. Der HTTP-Call wird dann mit einem `rec`-Parameter erweitert: `http://<ip-address>:8080/control/record/start?app=live&name=mystream&rec=rec1`


```
rtmp {
  server {
    listen 1935;
    application live {
      live on;
      recorder rec1 {
        record all manual;
        record_suffix rec1.flv;
        record_path /tmp/rec;
        record_unique on;
      }
      recorder rec2 {
        record all manual;
        record_suffix rec2.flv;
        record_path /tmp/rec;
        record_unique on;
      }
      recorder rec3 {
        record all manual;
        record_suffix rec3.flv;
        record_path /tmp/rec;
        record_unique on;
      }
    }
  }
}
```
