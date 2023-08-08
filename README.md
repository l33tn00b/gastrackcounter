# gastrackcounter
Gasverbrauch mit ESPEasy, fhem und etwas Bastelei im Blick.

# Ziel
Den Gasverbrauch im Blick behalten. Der Zähler hängt halt im Keller und eine manuelle Statistik ist doof.

# Fazit
Funktioniert. Ist über zwei Jahre recht stabil gelaufen. Jetzt ist der Gasanschluss weg, damit wird jetzt auf ähnliche Art die Wasseruhr im Blick behalten. Dazu woanders und später vielleicht mehr.

# Zutaten
- fhem am Laufen
- Hardware für ESPEasy, z.B. ein WEMOS o.ä.
- Reed Sensor (Meder, MK04-1A66B-500W)

# Umsetzung
## Hardware
Aus einem anderen Projekt hatte ich noch eine Trägerplatine für ESP-01-Module, die eigentlich für den Einbau in Unterputzdosen vorgesehen war. Diese Platine hat einen Weitbereichseingang, kann bis 36V Versorgungsspannung ab und macht daraus die 3,3V für den ESP. War eh da, also verwendet. Grundsätzlich tut es aber wie oben geschrieben auch jede andere Platine, auf der ESPEasy läuft. Die Platine hat aufgrund der beschränkten Anzahl von Pins des ESP-01 nur zwei digitale I/Os (SDA und SCL), wenn man den seriellen Port nutzen möchte. Reicht aber völlig aus.

Der Reed-Sensor wird zwischen digitalen Eingang und Masse geklemmt. Damit zieht er beim Durchschalten (wenn das Metall im Zähler vorbeirauscht) den Eingang auf Masse. 

## ESPEasy
Sorry, keine Ausführungen zur Installation von ESPEasy. Bitte auf den Seiten dort nachschlagen.
Zur Konfiguration:

