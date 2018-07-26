# Nightscout-on-Raspberry-PI
Installation einer Nightscout Instanz auf einem lokalen Raspberry PI
Installation of a nightscout-intance on a local Raspberry PI



Daten in <> durch eigene Daten ersetzen!!!

Schritt 1 - Raspbian Betriebssystem wird vorausgesetzt
 
	Evtl. den ssh Dienst auf dem Pi starten, je nach heimischem Aufbau
	Den Pi ins lokale Netzwerk bringen und mit einer festen IP versehen.
	Optional - Den Pi über einen DynDNS Anbieter über das Internet erreichbar machen.
	Portfreigabe für tcp/1337 im Internetrouter einrichten, um Daten zu empfangen und die Webseite aufrufen zu können.

Schritt 2 - Raspbian aktualisieren
	sudo apt-get update
	sudo apt-get upgrade

Schritt 3 - MongoDB installieren

		sudo apt install mongodb-server
		
	Admin User anlegen: 
		Auf der Mongo Shell - mongo:
		
		sudo mongo -shell
		
			use admin
			db.addUser( { user: <"USER">, pwd: <"PASSWORD">, roles: [ "userAdminAnyDatabase" ] } )
	
	Nightscout DB und Nightscout User anlegen: 
	
			use nightscout
			db.addUser( { user: <"USER_NS">, pwd: <"PASSWORD_NS">, roles: [ "readWrite", "dbAdmin" ] } )

Schritt 4 - NodeJS 
	Genutzt habe ich diese Anleitung: https://github.com/audstanley/NodeJs-Raspberry-Pi/
	
		sudo wget -O - https://raw.githubusercontent.com/audstanley/NodeJs-Raspberry-Pi/master/Install-Node.sh | sudo bash node -v

Schritt 5 - Nightscout
	Aktuelle Nightscout Version herunterladen aus dem GIT - https://github.com/nightscout/cgm-remote-monitor/releases (auf dem pi mit git clone)
	
	Kopiere die Sourcen z.B. ins Home Verzechnis des Benutzers pi.
	wget https://github.com/nightscout/cgm-remote-monitor/archive/0.10.2.zip
	
	Entpacke die Sourcen
		sudo unzip /home/pi/0.10.2.zip
		
	Damit später einfacher ein Update installiert, oder eine andere Version zum testen gesetzt werden kann, wird ein symbolischer Link auf das aktuelle Sourcen Verzeichnis gesetzt.
		sudo ln -s cgm-remote-monitor-0.10.2 nightscout
		sudo chmod 777 cgm-remote-monitor-0.10.2
		
	Ins Verzchnis nightscout wechseln und das setup starten.
			./setup.sh
		
	Der Installationsvorgang sollte nun ohne Fehler durchlaufen.
	Falls nicht schauen ob evtl. Abhängigkeiten nachinstalliert werden müssen.

Schritt 6 - my.env
	Bevor Nightscout gestartet werden kann muss im Nightscout Verzeichnis eine Konfigurationsdatei angelegt werden. (my.env)
 
		sudo nano my.env	

	Dort werden alle Einstellungen (Konfig, Plugins, etc vorgenommen)
	z.B.

					MONGO_CONNECTION=mongodb://<"USER_NS">:<"PASSWORD_NS">@localhost:27017/nightscout
					API_SECRET=<PASSWORD>

					DISPLAY_UNITS=mg/dl

					ENABLE="delta direction timeago devicestatus ar2 profile careportal boluscalc food rawbg iob cob bwp cage sage iage treatmentnotify basal pump openaps upbat errorcodes simplealarms bridge mmconnect loop"
					DISABLE=""

					TIME_FORMAT=24
					NIGHT_MODE=off
					SHOW_RAWBG=always
					THEME=colors

					ALARM_TIMEAGO_WARN=on
					ALARM_TIMEAGO_WARN_MINS=15
					ALARM_TIMEAGO_URGENT=on
					ALARM_TIMEAGO_URGENT_MINS=30

					PROFILE_HISTORY=off
					PROFILE_MULTIPLE=off

					BWP_WARN=0.50
					BWP_URGENT=1.00
					BWP_SNOOZE_MINS=10
					BWP_SNOOZE=0.10

					CAGE_ENABLE_ALERTS=true
					CAGE_INFO=44
					CAGE_WARN=48
					CAGE_URGENT=72
					CAGE_DISPLAY=hours

					SAGE_ENABLE_ALERTS=false
					SAGE_INFO=144
					SAGE_WARN=164
					SAGE_URGENT=166

					IAGE_ENABLE_ALERTS=false
					IAGE_INFO=44
					IAGE_WARN=48
					IAGE_URGENT=72
					BRIDGE_USER_NAME=
					BRIDGE_PASSWORD=
					BRIDGE_INTERVAL=150000
					BRIDGE_MAX_COUNT=1
					BRIDGE_FIRST_FETCH_COUNT=3
					BRIDGE_MAX_FAILURES=3
					BRIDGE_MINUTES=1400

					MMCONNECT_USER_NAME=
					MMCONNECT_PASSWORD=
					MMCONNECT_INTERVAL=60000
					MMCONNECT_MAX_RETRY_DURATION=32
					MMCONNECT_SGV_LIMIT=24
					MMCONNECT_VERBOSE=false
					MMCONNECT_STORE_RAW_DATA=false

					DEVICESTATUS_ADVANCED="true"

					PUMP_ENABLE_ALERTS=true
					PUMP_FIELDS="reservoir battery clock status"
					PUMP_RETRO_FIELDS="reservoir battery clock"
					PUMP_WARN_CLOCK=30
					PUMP_URGENT_CLOCK=60
					PUMP_WARN_RES=50
					PUMP_URGENT_RES=10
					PUMP_WARN_BATT_P=30
					PUMP_URGENT_BATT_P=20
					PUMP_WARN_BATT_V=1.35
					PUMP_URGENT_BATT_V=1.30

					OPENAPS_ENABLE_ALERTS=false
					OPENAPS_WARN=30
					OPENAPS_URGENT=60
					OPENAPS_FIELDS="status-symbol status-label iob meal-assist rssi freq"
					OPENAPS_RETRO_FIELDS="status-symbol status-label iob meal-assist rssi"

					LOOP_ENABLE_ALERTS=false
					LOOP_WARN=30
					LOOP_URGENT=60

					SHOW_PLUGINS=careportal
					SHOW_FORECAST="ar2 openaps"

					LANGUAGE=en
					SCALE_Y=log
					EDIT_MODE=on



		
Schritt 7 - Start/Stop Skript - Runlevel

	Im Nightscout Verzeichnise wird ein startskript angelegt:
		sudo touch start.sh

	Folgender Inhalt:
		#!/bin/sh
		(eval $(cat my.env | sed 's/^/export /') && PORT=1337 node server.js) &

	Skript ausführbar machen:
 		sudo chmod +x start.sh

	Mit dem Aufruf im Installationsverzeichnis wird Nightscout gestartet.

	./start.sh

Optional für automatischen Start von Nightscout bei Pi-Neustart.
(Hat bei mir noch nicht funktioniert.)

	Unter /etc/init.d eine Datei (nightscout) anlegen mit folgendem Inhalt. 
	WICHTIG - Pfade im Sktipt evtl. anpassen!
	cd /etc/init.d
	sudo touch nightscout
	
	
	#! /bin/bash
      	# Carry out specific functions when asked to by the system
	case "$1" in
 		start)
			echo "Starting Nightscout..."
			sudo -u pi bash -c 'cd /home/pi/nightscout/ && ./start.sh' >/dev/null 2>&1
    		;;
  		stop)
    		echo "Stopping Nightscout..."
			killall node
   				sleep 2
    	;;
 		*)
    	echo "Usage: /etc/init.d/nightscout {start|stop}"
    exit 1
    ;;
	esac
	exit 0	

	Damit Nightscout beim neustarten direkt mit gestartet wird, muss noch folgender Befehl als root ausgeführt werden:
		sudo update-rc.d nightscout defaults

	So wird das Startskript in die runlevel eingetragen.

Schritt 8 - Es sollte laufen 🙂
	Nach einem Neustart des Pi sollte dieser im Netzwerk erreichbar sein unter der zuvor festgelegten IP.
		http://local-ip:1337
