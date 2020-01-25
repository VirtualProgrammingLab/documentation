# ECS REST Interface

Die Kommunikation zwischen ECS und Participants erfolgt via [HTTP/1.1](http://www.w3.org/Protocols/rfc2616/rfc2616.html) als Transport- und Applikationsprotokoll. Für die Datenrepräsentation wird [JSON](http://www.json.org/) verwendet.

## Encoding und Locale
Das Encoding der Resourcenbeschreibung erfolgt in UTF-8. Die Internationalisierung (locale) wird beschrieben durch: `language[_territory]`, wobei language gemäß [ISO-639-1](http://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) und das optionale territory gemäß [ISO3166-1 alpha-2](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-2|ISO3166-1 alpha-2) zu wählen ist. Beispiele: de, de_DE, de_CH, en, en_US, en_GB

## Datums- und Zeitangaben
Alle Datums-/Zeitangaben werden gemäß [rfc3339](http://www.ietf.org/rfc/rfc3339.txt|rfc3339) gemacht (Internet profile of the ISO 8601 [ISO8601]
standard for representation of dates and times using the Gregorian calendar).

## Fehlermeldungen
Alle Fehlermeldungen werden dem Client über einen [[http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10|HTTP/1.1 Statuscode]] (rfc2616, Status Code Definitions) zurückgemeldet. Momentan werden folgende Statuscodes generiert bzw. unterstützt:

 * '''200 OK''' Die Anfrage wurde erfolgreich bearbeitet und das Ergebnis der Anfrage wird in der Antwort übertragen.
  * erfolgreiche Anfrage (GET) an die Resourcen {{{econtents, memberships, auths}}}
  * erfolreiches updaten (PUT) der Resource {{{econtents}}}
  * erfolgreiches Löschen der Resource {{{econtents}}}
 * '''201 Created''' Die Anfrage wurde bearbeitet und eine neue Resource wurde angelegt. Die URL der neuangelegten Resource wird im Location Header zurückgegeben.
 * '''400 Bad Request''' Die Anfrage-Nachricht war fehlerhaft aufgebaut.
  * ungültiges JSON Format
  * beim Anlegen (POST) einer neuen {{{econtents}}} Resource wurden nicht mindestens folgende Elemente angegeben und gültige Werte zugewiesen: title, url, etype, eligibleMembers, status, lang .
  * beim Anlegen (POST) einer neuen {{{auths}}} Resource wurden nicht mindestens folgende Elemente angegeben und gültige Werte zugewiesen: eid, mid, hash .
 * '''404 Not Found''' Die angeforderte Resource wurde nicht gefunden.
  * nach abfragen (GET), updaten (PUT) oder löschen einer {{{econtents}}} Resource durch Angabe einer nicht existenten {{{eid}}} bzw. nicht zugriffsberechtigten/sichtbaren {{{eid}}} aufgrund fehlender community Zugehörigkeit.
  * nach abfragen (GET) einer {{{memberships}}} Resource durch Angabe einer nicht existenten {{{eid}}}.
  * nach abfragen (GET) einer {{{auths}}} Resource durch Angabe einer ungültigen hash id.
  * beim Versuch eine {{{auths}}} Resource anzulegen (POST) für dessen Econtent man nicht der Eigentümer ist.
 * '''406 Not Acceptable''' Die angeforderte Ressource steht nicht in der gewünschten Form zur Verfügung.

## Resourcen Übersicht

|Resource|POST|GET|PUT|DELETE|
|--------|:----:|:---:|:---:|:------:|
|[econtens](/econtents)|x|x|x|x|
|[memberships](/memberships)| |x| |x|
|[auths](/auths)|x|x| | |
|[eventqueues](/eventqueues)| |x| | |

## Metadaten Übersicht

###Econtent
|Name|Typ|Zugriff|
|----|---|:-------:|
|eid|Integer|r|
|eligibleMembers|Integer Array|r/w|
|etype|"application/ecs-course"|r/w|
|lang|[String Locale](#encodingLocale)|r/w|
|owner|Integer|r|
|status|"online" | "offline"|r/w|
|title|String|r/w||
|url|String|r/w||

###Course
|Name|Typ|Zugriff|
|----|---|:-------:|
|courseID|String|r/w|
|courseType|String|r/w|
|credits|String|r/w|
|lecturer|String Array|r/w|
|organization|String|r|
|semesterHours|String|r/w|
|study_courses|String Array|r/w|
|term|String|r/w|
|timePlace|Hash|r/w|
|<center>begin</center>|[String Date](#dateTime)|r/w|
|<center>cycle</center>|String|r/w|
|<center>end</center>|[String Time](#dateTime)|r/w|
|<center>room</center>|String|r/w|
