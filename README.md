![grafik](https://github.com/l33tn00b/gastrackcounter/assets/28904067/7a2ec030-9d98-4ad1-80ab-a1e832c4fc8f)# gastrackcounter
Gasverbrauch mit ESPEasy, fhem und etwas Bastelei im Blick.

# Ziel
Den Gasverbrauch im Blick behalten. Der Zähler hängt halt im Keller und eine manuelle Statistik ist doof.

# Fazit
Funktioniert. Ist über zwei Jahre recht stabil gelaufen. Jetzt ist der Gasanschluss weg, damit wird jetzt auf ähnliche Art die Wasseruhr im Blick behalten. Dazu woanders und später vielleicht mehr.

# Zutaten
- fhem am Laufen
- ESPEasy (habe ne olle Version: v2.3.0 Build 147, ist ja schon ne Weile gelaufen).
- Hardware für ESPEasy, z.B. ein WEMOS o.ä.
- Reed Sensor (Meder, MK04-1A66B-500W)

# Umsetzung
## Hardware
Aus einem anderen Projekt hatte ich noch eine Trägerplatine für ESP-01-Module, die eigentlich für den Einbau in Unterputzdosen vorgesehen war. Diese Platine hat einen Weitbereichseingang, kann bis 36V Versorgungsspannung ab und macht daraus die 3,3V für den ESP. War eh da, also verwendet. Grundsätzlich tut es aber wie oben geschrieben auch jede andere Platine, auf der ESPEasy läuft. Die Platine hat aufgrund der beschränkten Anzahl von Pins des ESP-01 nur zwei digitale I/Os (SDA und SCL), wenn man den seriellen Port nutzen möchte. Reicht aber völlig aus.

Der Reed-Sensor wird zwischen digitalen Eingang und Masse geklemmt. Damit zieht er beim Durchschalten (wenn das Metall im Zähler vorbeirauscht) den Eingang auf Masse. 

In etwa so:   
![grafik](https://github.com/l33tn00b/gastrackcounter/assets/28904067/3677ed3a-6763-4d05-918e-cd9b7f0705f1)


## ESPEasy
Sorry, keine Ausführungen zur Installation von ESPEasy. Bitte auf den Seiten dort nachschlagen.
Zur Konfiguration: Offensichtlich muss das Ding dann erstmal ins Netz. Das bitte auch nach Doku von ESPEasy machen.   

### Main Settings  

![grafik](https://github.com/l33tn00b/gastrackcounter/assets/28904067/e1e77922-67a2-491b-9a26-5179810748ab)
Klar, der ESP heißt hier espWasser.

### Hardware Settings
![grafik](https://github.com/l33tn00b/gastrackcounter/assets/28904067/7cb571a1-0916-4ad0-9e87-d00f0fdd4cad)
Hier ist der passende GPIO einzutragen....

### Devices
Hier einfach auf das Gas (Task 3) achten.  

![grafik](https://github.com/l33tn00b/gastrackcounter/assets/28904067/476f504f-5d5d-48ec-bde7-6b6dfead1712)  


Task 3 (Gas) im Detail:  

![grafik](https://github.com/l33tn00b/gastrackcounter/assets/28904067/625848c2-e0fa-415b-8dd9-e5b35c03811c)


## fhem
Hier fängt die eigentliche Arbeit an. Da ein absoluter Zählerstand  bei ESPEasy nicht persistent gespeichert wird (reboot? -> Zählerstand wech), muss fhem irgendwie von einem anfänglichen Stand aus hochzählen und die relativen Zählerstände, die nach einem Update-Intervall von ESPEasy kommen, dazuaddieren.

## Device, um Daten von den ESPs, die ESPEasy am laufen haben, zu sammeln:  
```
defmod espBridge ESPEasy bridge
attr espBridge authentication 1
attr espBridge autocreate 1
attr espBridge combineDevices 0
attr espBridge group ESPEasy Bridge
attr espBridge room ESPEasy
```  

Danach noch user und passwort setzen:
```set espBridge user <username>```
```set espBridge pass <password>```

Weil autocreate an ist, sollte der ESP dann automatisch angelegt werden, wenn er sich zum ersten Mal bei fhem meldet.

## Zählerstand tracken
- dummy anlegen, um Zählerstand zu tracken:
  ```
  define gasVirtuellerZaehlertand dummy
  set gasVirtuellerZaehlerstand 0
  attr gasVirtuellerZaehlerstand room Gas
  ```
- DOIF, das den Dummy hochzählt, wenn neue Daten da sind:
  ```
  defmod di_gasTrackCounterRel DOIF ([ESPEasy_espWasser_Gas:Count]) \
	(setreading gasVirtuellerZaehlerstand state {(ReadingsNum("gasVirtuellerZaehlerstand","state","0") + ReadingsNum("ESPEasy_espWasser_Gas","Count","0"))})
  attr di_gasTrackCounterRel do always
  attr di_gasTrackCounterRel room Gas
  ```
  Weil der ESP eben espWasser heißt, und den Task Gas hat, das ganze Ding per autocreate angelegt wird, taucht der Name oben zusammengesetzt (ESPEasy_espWasser_Gas) im DOIF auf.
- Wäre ja doof, wenn wir das nicht auch irgendwo loggen würden, um es dann auszuwerten:
  ```
  defmod FileLog_gasVirtuellerZaehlerstand FileLog ./log/gasVirtuellerZaehlerstand_%Y_%m.log gasVirtuellerZaehlerstand:.*
  attr FileLog_gasVirtuellerZaehlerstand room Energie,Gas
  ```


- 

