#auths
`POST, auths:` Der ECS speichert einen eindeutigen `hash` Wert zur späteren Autorisation. Zusätzlich müssen noch die `mid` des konsumierenden Participanten und `eid` des Econtent angegeben werden. Optional sind `sov` (start of validation time) und `eov` (end of validation time). Ein `auth` Eintrag hat bei Nichtangabe von `sov, eov` eine vom ECS bestimmte Lebensdauer (Standard 30 Sekunden), d.h. wird nach Anlegen eines Auth Objekts dieses nicht innerhalb der nächsten 30sec abgerufen, verfällt der Auth Eintrag und ein nachträglicher Autorisationsversuch schlägt fehl. Diese Zeitspanne sollte für den anschliessenden Forward und die Anfrage des econtentanbietenden Participanten ausreichen. Werden hingegen `sov, eov` angegeben, steht der Auth Eintrag nur innerhalb dieser beiden Zeiten für eine Autorisation zur Verfügung (asynchrones konsumieren von Econtentlinks). Liefert als HTTP Status Code entweder "201 Created" für eine erfolgreiches Anlegen oder "400 bzw. 404" für Fehler. Auth Einträge können nur von Participanten gemacht werden, dessen `mid` als `owner` für den jeweiligen `eid` im entsprechenden Econtent eingetragen sind (ansonsten HTTP Status Code 404). {#auths_abr} Dem anfragenden Participanten wird zusätzlich ein optionales und eindeutiges Kürzel `abr` des  konsumierenden Participanten zur Verfügung gestellt. Dies kann, wenn vorhanden, z.B. zur zusätzlichen Kennzeichnung von neu anzulegenden Accounts auf dem Econtent anbietenden Participanten verwendet werden (siehe auch [memberships](memberships#abr)).

Wertebereiche bzw. Infos für einzelne Attribute:

 * "hash" : Eine einfache Möglichkeit zur Generierung eines eindeutigen Hash's: {{{ echo "`date` ecs.uni-stuttgart.de" | sha1sum}}}, also einen sha1 Hash über das aktuelle Systemdatum und seinen jeweiligen DNS. Der Hash darf eine Länge von 256 Zeichen nicht überschreiten. Der Algorithmus zur Erzeugung des Hashwertes sollte geheim und/oder Zufallskomponenten enthalten.
 * "sov" : steht für {{{start of validation}}} time. Zeitangabe im Format nach rfc3339. Optionales Datum. Muss immer gemeinsam mit {{{eov}}} angegeben werden.
 * "eov" : steht für {{{end of validation}}} time. Zeitangabe im Format nach rfc3339. Optionales Datum. Muss immer gemeinsam mit {{{sov}}} angegeben werden.

```
https://infolms.rus.uni-stuttgart.de:7923/auths
[
{
  "hash":  "87c1141e5ec3adf194b2ea600974b79f494027ff",
  "mid" :  1,
  "eid" :  1,
  "sov" :  "2008-01-07T13:30+01:00",
  "eov" :  "2008-02-14T15:00+01:00"
},
...
]
```
`GET, auths/<hash>:`: Gibt entsprechend nachgefragte {{{auths}}} Resource zurück. Bei vorhandenem Hash (<hash>) wird  als HTTP Status Code "200 OK" und der entsprechende Auths-Eintrag zurückgegeben, ansonsten HTTP Status Code "404" für eine unberechtigte Anfrage.
```
https://infolms.rus.uni-stuttgart.de:7923/auths/87c1141e5ec3adf194b2ea600974b79f494027ff

[
{
  "hash":  "87c1141e5ec3adf194b2ea600974b79f494027ff",
  "mid" :  1,
  "eid" :  1,
  "sov" :  "2008-01-07T13:30+01:00",
  "eov" :  "2008-02-14T15:00+01:00",
  "abr" :  "S"
}
]
```
