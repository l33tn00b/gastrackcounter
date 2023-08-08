# gastrackcounter
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

### Device, um Daten von den ESPs, die ESPEasy am laufen haben, zu sammeln:  
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

### Zählerstand tracken
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
# Megastumpfes Phython Script zur Auswertung:  

Nimmt als Argument den Namen eines fhem-Logfiles (s.o.), das dann nach Tagen ausgewertet wird. Gasverbrauch eines Tages (im Kubikmetern) ist also der Stand am endes des Tages minus der Stand am Anfang des Tages. Spuckt die Ergebnisse der Berechnungen in einer csv-Datei (Name der Eingabedatei + '_gasprotag_kubik.csv') aus. Die kann dann in Exel hübsch gemacht werden. U.A. mit Umrechnung (siehe Gasrechnung) der Kubik in Kilowattstunden, um dann eine Abschätzung für den Strombedarf bei Umstellung auf Wärmepumpe zu machen. 

```
# Auswertung von FHEM Logfiles um eine Abschätzung des Energiebedarfs
# pro Tag  zu erhalten.

# zur Auswertung der virtuellen Zählerstände, die in FHEM generiert werden
# das Problem bei den ESP-Easy Zählern ist, dass die dne Zählerstand verlieren
# würden, wenn sie mal einen Schluckauf bei der Stromversorgung oder einen
# Reset aus anderen Gründen haben

# also zählen wir mit dem ESPEasy nicht absolut sondern lassen uns alle paar
# sekunden die in der zurückliegenden Periode gemessenen Werte liefern

# dann haben wir in fhem ein doif, das bei neuen werten einen dummy hochzählt,
# und damit unseren virtuellen zählerstand generiert.

# den werten wir hier pro Tag aus

# TODO/BUG: Am Anfang des Monats wird falsch gerechnet, weil
# da natürlich vom letzten Monat die Referenz fehlt, um das Delta
# berechnen zu können (liegt einfach daran, dass die Logfiles monatsweise
# vorliegen)
# muss dann halt in excel manuell korrigiert werden

#python 3
import argparse,sys
from datetime import datetime, time, timedelta
#from string import split

parser = argparse.ArgumentParser(description='Process Command Line Arguments.')
parser.add_argument('filename', type=str,
                   help='name of file that is to be processed')

args = parser.parse_args()
print(args.filename)
# read from specified file
infile = open(args.filename,'r')

# set up output files
# gas (kubik) pro Tag
outfilename = args.filename.split('.')[0]+'_gasprotag_kubik.csv'
print(outfilename)
outfile = open(outfilename,'w')

# starttag bestimmen
#set timestamp for startup to collate entries with identical timestamps
#whereas timestamp is not the sdm's timestamp
#but fhem's logging timestamp
line = infile.readline()
prevDate = datetime.fromisoformat(line.split(' ')[0])
print ("Startdatum: ", prevDate.date().isoformat())
# impulse des vorangegangenen Tages
# die müssen wir dann von denen des momentanen Tages abziehen
# TODO: mal gucken, ob ein int reicht :P
#ZaehlerVorangegangenerTag = int(line.split(' ')[2])
ZaehlerVorangegangenerTag = 0
print("Startwert Zaehler: ", ZaehlerVorangegangenerTag)
#infile.close()
#outfile.close()
#sys.exit()

#reset file read counter
infile.seek(0);
# read lines until no more lines available
# exiting loop via break
while True:
    line = infile.readline()
    if not line:
        break
    #items are separated by whitespace
    lineSplit = line.split(' ')
    # der erste wert ist unser datetime
    dtString = lineSplit[0]
    # haben wir noch den gleichen Tag? dann wert lesen
    # current date
    cdate = datetime.fromisoformat(dtString)
    # vorsicht bei vergleich, wir dürfen nicht die Objekte
    # vergleichen (das wäre ein "=")
    if cdate.date() == prevDate.date():
        #print("Noch der gleiche Tag", cdate.date().isoformat(), " ", line)
        letzterZaehlerstandMomTag = int(lineSplit[2])
        #print (letzterZaehlerstandMomTag)
    else:
        print ("Neuer Tag: ", cdate.date().isoformat())
        # letzten Zaehlerstand des vorangegangenen - Tages schreiben
        outfile.write(prevDate.date().isoformat()+";"+str(letzterZaehlerstandMomTag - ZaehlerVorangegangenerTag)+"\n")
        
        # neues datum setzen
        prevDate = cdate
        ZaehlerVorangegangenerTag = letzterZaehlerstandMomTag
        
 

# und am ende kommt kein neuer tag
# da müssen wir dann halt die werte noch von Hand schreiben...
outfile.write(prevDate.date().isoformat()+";"+str(letzterZaehlerstandMomTag - ZaehlerVorangegangenerTag)+"\n")
infile.close()
outfile.close()
sys.exit()

```
# Resultat in Excel   
Das sieht dann geplottet etwa so aus:
![grafik](https://github.com/l33tn00b/gastrackcounter/assets/28904067/27466c5d-8a76-428b-9682-da3ce35e3e6f)
Klein und grau ist die Außentemperatur. Da kann man schon erkennen, dass mit geringerer Temperatur der Bedarf an Heizenergie steigt. Ach was.
