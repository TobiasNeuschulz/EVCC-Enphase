# Anbindung einer Enphase-IQ-Battery an EVCC

## Option 1) Nutzung EVCC Standard Template

Die EVCC.yaml sieht wie folgt aus:

```yml
meters:
- type: template
  template: enphase
  usage: grid
  host: <Enter Enphase IQ Gateway IP here>
  token: <Enter Enphase Token here>
  name: grid1
- type: template
  template: enphase
  usage: pv
  host: <Enter Enphase IQ Gateway IP here>
  token: <Enter Enphase Token here>
  name: pv2
- type: template
  template: enphase
  usage: battery
  host: <Enter Enphase IQ Gateway IP here>
  token: <Enter Enphase Token here>
  name: battery3

[...]

site:
  title: EVCC-Example-Site
  meters:
    grid: grid1
    pv:
    - pv1
    - pv2
    battery:
    - battery3
  residualPower: 100
```
Vorteile:
- einfache Konfiguration

Nachteile:
- offiziell nur für Enphase "AC Battery" ("AC" ist hierbei ein Produkttyp) und diese Konfiguration unterstützt nur teilweise die neueren Enphase Batterien des Typs "IQ Battery"
- Innerhalb von EVCC wird eine IQ Battery zwar mit dem aktuellen Ladestand (SoC) angezeigt, jedoch bleiben die Lade- bzw. Entladeströme in (W) bei 0.

![EVCC no charging information](images/EVCC-EnphaseIQBattery-NoChargeInfo-small.png)

## Option 2) Nutzung EVCC Custom-Template für die Enphase IQ Battery

Der in der Standard Konfiguration erzeugte Block für eine Enphase Batterie
```yml
- type: template
  template: enphase
  usage: battery
  host: <Enter Enphase IQ Gateway IP here>
  token: <Enter Enphase Token here>
  name: battery3
```

wird ersetzt durch

```yml
- name: battery3
  type: custom
  power:
    source: http
    uri: https://<Enter Enphase IQ Gateway IP here>/ivp/livedata/status
    method: GET
    auth:
      type: bearer
      password: <Enter Enphase Token here>
    insecure: true
    jq: .meters.storage.agg_p_mw
    scale: 0.001
  soc:
    source: http
    uri: https://<Enter Enphase IQ Gateway IP here>/ivp/livedata/status
    method: GET
    auth:
      type: bearer
      password: <Enter Enphase Token here>
    insecure: true
    jq: .meters.soc
  capacity: 3.5 # kWh
```

In der "Enlighten" App kann der Live Status immer nur für 15 Minuten aktiviert werden. Anschließend sieht EVCC mit dieser Konfiguration keine aktuellen Werte mehr.

![Enlighten Live Status](images/Enlighten-Live-Status-small.png)

Wie kann das verhindert werden?
Mittels Cron-Daemon wird zeitbasierten die Ausführung eines Befehls als wiederkehrende Aufgabe automatisiert, welche den Live Status alle 14 Minuten aktiviert.

```
sudo crontab -e
```

```
*/14 * * * * curl -f -k -H "Authorization:bearer <Enter Enphase Token here>" -H "Content-Type:application/json" -d "{\"enable\":1}" https://<Enter Enphase IQ Gateway IP here>/ivp/livedata/stream
```

Wer damit leben kann, dass die Daten nur am Tag aktualisiert werden (hier im Beispiel 7-20 Uhr) kann den Cronjob auch wie folgt anlegen:
```
*/14 7-20 * * * curl -f -k -H "Authorization:bearer <Enter Enphase Token here>" -H "Content-Type:application/json" -d "{\"enable\":1}" https://<Enter Enphase IQ Gateway IP here>/ivp/livedata/stream
```

Der curl Befehl kann zunächst auch einfach mittels SSH auf dem Linux System getestet werden. Die Rückgabe nach Ausführung sollte sein:
```
[...]
"sc_stream": "enabled"
```

That's it! EVCC sollte jetzt alle Werte (SoC und Lade- bzw. Entladeströme) korrekt anzeigen:
![EVCC with Enphase IQ Battery information](images/EVCC-EnphaseIQBattery-ChargeInfo-small.png)
