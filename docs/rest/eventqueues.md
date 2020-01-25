## eventqueues
`GET, eventqueues:`:Informiert einen Participanten über etwaige Zustandsänderungen seiner zugriffsberechtigten `econtents` Objekte. Ein Abruf der `eventqueues` Resource liefert alle dem ECS zum Zeitpunkt des Abrufes bekannten Zustandsänderungen der jeweiligen `econtents` Objekte in Form einer `eid` (Econtents ID) und eines `op` Codes (operation code) dem aufrufenden Participanten zurück. Die `eid` repräsentiert natürlich den zustandsveränderten Econtent und der `op` Code gibt die Art der Veränderung an, welche die Resource auf dem ECS erfahren hat. Dabei haben die `op` Codes folgende Bedeutung:

 * "create": Auf dem ECS wurde ein neuer Econtent angelegt. Dem Participanten steht deshalb ein neuer Econtent zur Verfügung.
 * "update": Ein bestehender Econtent hat auf dem ECS eine Zustandsänderung erfahren. Zustandsänderung kann in diesem Fall jedwede Aenderung eines Metadatums sein. Dies kann für den Participanten unter anderem ein Löschen oder neu Anlegen eines Econtentlinks bedeuten oder eben ein aktualisieren eines bestehenden Links.
 * "delete": Der Econtent wurde auf dem ECS gelöscht. Alle Econtentlinks die diesen Econtent betreffen müssen auf dem Participanten gelöscht werden. 

Es obliegt dem Participanten, ob und wann eine Aktualisierung seiner Econtent Objekte durch sukzessives Abrufen der `econtents/<eid>` Objekte vom ECS erfolgt. Ruft ein Participant seine `econtents` Resource ab (alle für ihn zugriffsberechtigte Econtent-Objkete werden zurückgegeben), dann setzt der ECS die Eventqueue des Participanten zurück (alle Einträge werden gelöscht). Dies sollte immer dann erfolgen, wenn ein Participant sich neu beim ECS anmeldet. Erst nach mindestens einmaligem Abruf der `econtents` Resource (Erstsynchronisation) enthält die jeweilige Eventqueue gültige Daten für ihren Participanten und kann dann für die weitere Synchronisation verwendet werden. Frägt der Participant daraufhin seine eventqueues Resource ab, bekommt er eine leere Liste. Dasselbe geschieht auch nach Abruf der `eventqueues` Resource. Ein fehlerfreier Aufruf wird mit einem HTTP Status Code "200 OK" quittiert. Bekommt ein Participant nach Abruf einer `eventqueues/<eid>` Resource den  HTTP Status Code "404 Not Found" zurück, hat er dies so zu behandeln, als hätte er über seine Eventqueue einen `"op":"delete"` bzgl. der <eid> erhalten. 

```
https://ecs.uni-stuttgart.de:7923/eventqueues
[
{"econtents":{"eid":45,"op":"create"}},
{"econtents":{"eid":31,"op":"update"}},
...
]
```

## Commandqueue

Die Eventqueue wird erweitert zu einer Commandqueue.<BR>
Die neuen Objekte der Eventqueue (Commands) sollen eine Art "remote control" bzgl. der Participanten erm&ouml;glichen. Dies dient in erster Linie dazu, eine Neusynchronisation zwischen Participant und ECS herzustellen, ohne dass dazu ein h&auml;ndisches Eingreifen seitens der Administratoren der Participanten vonn&ouml;ten w&auml;re. Diese Commands werden ausschlie&szlig;lich vom ECS Admin manuell getriggert. 

 1. Der Participant liest die Resourcen `memberships` und `econtents` neu ein:
 ```
 {"cmd":{"admin":"reset"}}
 ```

 2. Der Participant reexportiert (l&ouml;schen und neuanlegen) seinen freigegebenen Econtent und liest anschlie&szlig;end die Resourcen `memberships` und `econtents` neu ein:
```
 {"cmd":{"admin":"reset_all"}}
```

Vornehmlich f&uuml;r Testzwecke kann jeder Participant die jeweiligen Commands auch selbst triggern, indem er einfach folgende Resourcen aufruft (POST):

 1. `POST, eventqueues/adminReset`
 2. `POST, eventqueues/adminResetAll`

Ein fehlerfreier Aufruf wird mit einem HTTP Status Code "201 Created" quittiert andernfalls "500 Internal Server Error". Danach kann der Participant &uuml;ber seine `eventqueue` Resource die Kommandos abholen und ausf&uuml;hren.

Ein `curl` Aufruf zum Triggern der `eventqueues/adminReset` Resource f&uuml;r den `beluga` Participanten k&ouml;nnte demnach folgenderma&szlig;en aussehen:
```
curl --cacert freeit-ca-cert.pem --cert beluga-cert.pem  --key beluga-key.pem  --pass "nixfuerungut" -H "Content-Type: application/json"  -H "Accept: application/json"  -i -X POST https://ecs.uni-stuttgart.de:7923/eventqueues/adminReset
```
