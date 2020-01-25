### econtents 
`GET, econtents:` Listet alle verfügbaren eContent-Angebote des anfragenden Participanten auf, sofern er mit einer seiner MembershipIDs (`mid`) in `eligibleMembers` eines `econtents` vorkommt. Das Attribut `eligibleMembers` listet dabei alle MembershipIDs (`mid`) auf, die den Econtent verlinken bzw. referenzieren dürfen. Die dem Attribut `owner` zugeordnete `mid` repräsentiert die Heimatplattform bzw. den eigentlichen Anbieter des Econtent.

```
https://infolms.rus.uni-stuttgart.de:7923/econtents
[
{
"url"    : "https://il3.stgt.de/course23",
"title"  : "HM II Einführung",
"eligibleMembers" : [1,3],
"eid": 1,
"owner": 1,
...
},
{
"url"    : "https://il3.stgt.de/course15",
"title"  : "Datenverarbeitung I",
"eligibleMembers" : [2,3,5],
"eid": 43,
"owner": 2,
...
}
]
```

`GET, econtents/<eid>:` Gibt eContent mit der eContentID= <eid> zurück. Dies funktioniert auch, wenn der aufrufende Participant nicht in `eligibleMembers` des `econtents` gelistet ist, sondern dort als `owner` eingetragen ist:

```
https://infolms.rus.uni-stuttgart.de:7923/econtent/125
[
{
"url"    : "https://il3.stgt.de/course23",
"title"  : "HM II Einführung",
"eid": 125
...
}
]
```

`POST, econtents:` Generiert neuen eContent. Nachfolgend sind alle möglichen Attribute für den Course-Econtent aufgeführt. Es müssen mindestens "url", "title", "eligibleMembers", "etype", "status","lang" und "owner" angegeben werden. Wertebereiche bzw. Infos für einzelne Attribute:

 * "status" : "online" | "offline"
 * "etype" : "application/ecs-course"
 * "eligibleMembers" : gültige MembershipIDs (`mid`). Für gewöhnlich gewonnen aus Abfrage der `memberships` Resource welche dem Dozenten zur Auswahl über Formular                  dargestellt werden.
 * "lang" : "de_DE"
 * "owner": gültige MembershipIDs (`mid`), d.h. der Participant muss dieser `mid` zugeordnet sein.

```
https://infolms.rus.uni-stuttgart.de:7923/econtents

{
  "timePlace": {
    "room": "Raum M 2.11",
    "cycle": "wöch.",
    "begin": "2008-01-10T13:30:00+01:00",
    "end": "2008-01-10T15:00:00+01:00"
  },
  "lecturer": [
    "Prof. Dr. Stephan Nussberger",
    "Dipl.-Ing. Markus Sewald"
  ],
  "courseID": "course5:220",
  "semesterHours": "5",
  "study_courses": [
    "Technische Biologie",
    "Elektrotechnik"
  ],
  "url": "https://ilias3.uni-stuttgart.de/goto.php?target=crs_2737&client_id=Uni_Stuttgart",
  "title": "Hochgeschwindigkeitskommunikationsnetze",
  "abstract": "Moderne media access Verfahren als Wegbereiter der Hochgeschwindigkeitskommunikation.",
  "eligibleMembers": [2,1,3],
  "owner": 1,
  "courseType": "Vorlesung",
  "lang": "de_DE",
  "etype": "application/ecs-course",
  "status": "online",
  "credits": "10",
  "term": "WS 06/07"
}
```

`eligibleMembers` sind [MembershipIDs](../memberships#mid). 

`PUT, econtents/<eid>:` Updated eContent mit der eContentID=\<eid\>:

```
https://infolms.rus.uni-stuttgart.de:7923/econtents/125

{
"lecturer"  : [
  "Prof. Uli Ganzgenau",
  "Dr. Udo Grenzwert",
  "Dipl.-Ing Manfred Durchblick"
],
"credits": 23
}
```

`DELETE, econtents/<eid>` Löscht eContent mit der eContentID=<eid> :
```
https://infolms.rus.uni-stuttgart.de:7923/econtents/125
```
