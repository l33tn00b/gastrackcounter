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
Zur Konfiguration: Offensichtlich muss das Ding dann erstmal ins Netz. Das bitte auch nach Doku von ESPEasy machen.   

### Main Settings  

![grafik](https://github.com/l33tn00b/gastrackcounter/assets/28904067/e1e77922-67a2-491b-9a26-5179810748ab)

### Hardware Settings
![grafik](https://github.com/l33tn00b/gastrackcounter/assets/28904067/7cb571a1-0916-4ad0-9e87-d00f0fdd4cad)
### Devices
Hier einfach auf das Gas (Task 3) achten.  

![grafik](https://github.com/l33tn00b/gastrackcounter/assets/28904067/476f504f-5d5d-48ec-bde7-6b6dfead1712)  


Task 3 (Gas) im Detail:
![grafik](https://github.com/l33tn00b/gastrackcounter/assets/28904067/625848c2-e0fa-415b-8dd9-e5b35c03811c)




