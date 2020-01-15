#format wiki #language en
#pragma section-numbers 2
<<TableOfContents(4)>>
<<Anchor(core_ecs)>>
= Core ECS =
<<Anchor(http_header)>>
== HTTP Header ==
=== Application specific headers ===
 X-EcsAuthId:: Has to be a valid participant id. In a standard ECS
 configuration this HTTP header will be attached by the authentication process
 running on the proxy server.

 X-EcsReceiverCommunities:: Has to be a valid community id/ids or community name/names.
 Adresses all participants joined the comimunity/communities. You are able to note
 multiple communities, either by name or by id, spaced by comma. Only allowed by `POST`.  
 
 X-EcsReceiverMemberships:: Has to be a valid membership id/ids. Adresses all
 listed memberships. You are able to note multiple membersips spaced by comma.
 Only allowed by `POST`.

 X-EcsSender:: Beschreibt bei einem `GET` auf eine Message/Queue Ressource
 den Absender der Nachricht in Form seiner Membership-ID. Wird ausschließlich
 vom ECS gesetzt/beschrieben. Gleichzeitig setzt der ECS
 `X-EcsReceiverCommunities` auf die entsprechende Community des Senders.
 Falls der Absender Teinehmer mehrerer Communities ist, bezeichnet X-!EcsSender
 eine Liste von Membership-IDs, entsprechend enthält auch
 `X-EcsReceiverCommunities` eine korrespondierende Liste an Communities.

 X-EcsQueryStrings:: Used to provide [[#querystrings|querystrings]]. 
=== HTTP standard header ===
 Accept:: Content-Types that are acceptable.
 Content-Type:: The mime type of the body of the request (used with POST and PUT requests).
 If-None-Match:: Allows a 304 Not Modified to be returned if content is unchanged.
 Cookie:: An HTTP cookie previously sent by the server with Set-Cookie.
 Content-Type:: The mime type of this content.
 ETag:: An identifier for a specific version of a resource.
 Location:: Used in redirection, or when a new resource has been created.
 Set-Cookie:: An HTTP cookie.

== Addressing ==
In order to communicate to each other you have to provide a unique address.
These addresses can either be a so called membership id or a community id or
community name.

=== Membership IDs ===
These are unique ids in the scope of an ECS. They establish a relationship
between a participant and a community:

`+------------+ 1        N +-----------+ N        1 +-----------+`<<BR>> 
`|participants|------------|memberships|------------|communities|`<<BR>>
`+------------+            +-----------+            +-----------+`

Therefore a participant can be associated to different communities. Every
participant can inquire his membership ids by calling the
[[#memberships|memberships]]
ressource.

=== Community names and ids ===
A community can be referenced by his community id (cid) or his community name.
If you address a community you implicit address all members of the community.
This applies also to the sender joining the receiver community if the sender
has set his ''community_selfrouting'' flag (default off), otherwise the sender
will be implicitly excluded from the receiver list. Every participant
can inquire his communities memberships by calling the
[[#memberships|memberships]]
ressource. 

=== Create a ressource ===
If you want to `POST` to a ressource you have to provide either a
`X-EcsReceiverMemberships` or `X-EcsReceiverCommunities` header or both
together. 

If you want to address a single membership or a dedicated number of
memberships you have to set the {{{X-EcsReceiverMemberships}}} header. This
header can have a list of values, e.g.
{{{
X-EcsReceiverMemberships: 3,6,47
}}}

If you want to address a community you have to set the
{{{X-EcsReceiverCommunities}}} header. This header can have a list of values,
e.g.
{{{
X-EcsReceiverCommunities: SWS,23,25
}}}

=== Get a ressource ===
If you `GET` a ressource then the ECS set the `X-EcsSender` and the
`X-EcsReceiverCommunities` header to show you from whom and where your received
message comes. If there is a list of `X-EcsReceiverCommunities` values than
there is also a list of corresponding `X-EcsSender` values, i.e. the sending
participant is member of multiple communities and addressed his message to
multiple communities also, e.g.
{{{
X-EcsSender: 3,19
X-EcsReceiverCommunities: UnisBW,SUV
}}}
This means that this message is addressed to you through two communities
(UnisBW, SUV) and the sender has the membership id 3 in UnisBW and 19 in SUV.

== Participants ==
=== Community selfrouting ===
If community selfrouting is activated at the participant (administration area)
you can decide if you also want to receive the message which you send to an
appropriate community, i.e. you get an event notification (if events on this
resource is activated) and you get it listed by its [[#list-ressource|list resource]]
and could access it through its [[#queue-ressource|queue resource]]. Of
course, as sender of the message you can always access it by its
[[#message-resource|message resource]].
=== Authentication ===
All participants have to be authenticated in order to use ECS services. A
participant is deemed to be authenticated if the `X-EcsAuthId` header is set
and the ECS knows it. The real authentication take place ''in front of'' the
ECS, normally at the Webserver (Proxy). But this depends on
configuration/installation of ECS:
                  
`   +-----------+    .        `<<BR>> 
`   |   ECS     |   /_\       `<<BR>> 
`   | (RAILS)   |    |        `<<BR>> 
`   +-----------+    |        `<<BR>> 
`   |Rack Module|    |        `<<BR>> 
`   | (optional)|    |        `<<BR>> 
`   +-----------+    | Message`<<BR>> 
`   | Webserver |    | Flow   `<<BR>> 
`   |  (Proxy)  |    |        `<<BR>> 
`   +-----------+    |        `<<BR>> 
`         |          |        `<<BR>> 
`   +-----------+    |        `<<BR>> 
`   |Participant|    |        `<<BR>> 
`   +-----------+             `

 * Basic Auth ([[/nginx]], [[/apache]])
 * X.509 certificates ([[/nginx]], [[/apache]])
                                     
=== Anonymous participants ===
The creation of a new anonymous participant automatically takes place by every
call to an ECS ressource if the calling participant didn't set `X-EcsAuthId` or
`Cookie` header, by setting a "Set-Cookie:" header in the response. On
subsequent calls the participant has to provide this cookie in a "Cookie"
header in order to be identified as the previously calling participant.
Additionally those participants were automatically joined to the `public`
community. Further their lifetime will be limited and all ressources will be
silently deleted after this lifetime becomes zero. With succesional accesses to
ECS this lifetime will be refreshed. For general cookie handling see also
[[http://en.wikipedia.org/wiki/HTTP_cookie|HTTP cookie]]. See also [[/examples|examples]].

== Ressource access restrictions ==
Of course this is best done on webserver. 

 * [[/nginx]], [[/apache]]

<<Anchor(statische_ressourcen)>>
<<Anchor(system_ressourcen)>>
<<Anchor(system_resources)>>
== System Ressources ==
These are default ressources provided by any ECS installation. 

<<Anchor(events)>>
=== Events queue ===
[[http://freeit.de/documents/ecsa/index.html#events_resource|see central documentation place]]

<<Anchor(memberships)>>
=== Memberships ===
[[http://freeit.de/documents/ecsa/index.html#memberships_resource|see central documentation place]]

=== Authorization through One Touch Tokens ===
[[http://freeit.de/documents/ecsa/index.html#auths_resource|see central documentation place]]

<<Anchor(application_resources)>>
== Anwendungsspezifische Ressourcen ==
Alle Ressourcen die explizit für eine Anwendung bereitgestellt werden sollen,
müssen vorab beim ECS konfiguriert werden. Dabei stellt der
ECS drei Typen von Ressourcen zur Verfügung:

 1. Message Ressource
 2. Listen Ressource
 3. Queue Ressource

Allgemein steht hinter dem Ressourcenbegriff ein abstraktes Konzept. Ressourcen sind:

 * eindeutig identifizierbar (im HTTP Kontext durch URIs)
 * haben eine oder mehrere Repräsentationen (z.B. JSON, XML, Text, ...)

Für den Ressourcen Begriff ist es unerheblich, wie ihre Repräsentation
zustande kommt. Ob dies durch eine statische Datei oder durch ein Skript
(dynamisch) geschieht ist vollkommen egal. Einer URI/Ressource
ist dies von aussen nicht anzusehen (black box) und soll es auch nicht
sein (API, Entkopplung). Eine Bewertung einer Ressource aufgrund
''innerer'' Vorgänge und Gegebenheiten ist somit ebenfalls unerheblich,
ja sogar unzulässig.

Für alle weiteren Betrachtungen soll folgendes gelten:

 * POST: Liefert {{{201}}} bei erfolgreichem anlegen der Ressource.
 * Bei Fehlern wird generell immer ein  {{{4XX}}} oder {{{5XX}}} Status Code zurückgegeben. 
 * Bei nichtvorhandensein der Ressource ein {{{404}}} Status.
 * Bei Conditional-GET gibt {{{304}}} (Not Modified) an, dass sich die Ressource nicht geändert hat.
 * Entsprechende Ressource existiert und Operation ist erfolgreich, andernfalls Fehlermeldungen wie vorstehend beschrieben.


<<Anchor(resource_structure)>> 
=== Ressourcenstruktur ===
{{{
/<projectnamespace>/<name>
/<projectnamespace>/<name>/details
/<projectnamespace>/<name>/<id>
/<projectnamespace>/<name>/<id>/receivers
/<projectnamespace>/<name>/<id>/details
/<projectnamespace>/<name>/fifo
/<projectnamespace>/<name>/lifo
}}}



<<Anchor(persistent-messages)>>
<<Anchor(message-ressource)>>
=== Message Ressource ===
In einer Message Ressource können Nachrichten für einen bestimmten Empfänger
abgelegt werden. Der Empfänger kann diese Nachrichten ebenfalls über diese
Ressource abholen. Eine Message Ressource kann so konfiguriert werden, dass die
in ihr abgelegten Nachrichten "persistent" gehalten werden, um so
neuhinzukommenden Participanten gegebenenfalls zum Abruf zur Verfügung zu stehen
(siehe [[#postrouting|Postrouting]]). Dies erlaubt den Aufbau einfacher und
verteilt genutzter Objektdatenbanken.

''GET'': Liefert Nachricht und gibt "200" Status Code. <<BR>>
''DELETE'': Löscht Nachricht und gibt "200" Status Code. Im Body wird die Repräsentation der gelöschten Ressource zurückgegeben.<<BR>>
''PUT'': Erneuert Nachricht und gibt "200" Status Code. <<BR>>
''POST'': unzulässig ("405" Method Not Allowed Statuscode).

Ressourcenstruktur: `/<projectnamespace>/<name>/<id>` <<BR>>
Bsp. einer Message-Ressource: `http://isblab.rus.uni-stuttgart.de/numlab/exercises/33`

<<Anchor(list_resources)>>
<<Anchor(list-ressource)>>
<<Anchor(listen-ressource)>>
==== Subresource "details" ====
You can ask for detailed (meta) information of a posted
message. Only the original sender or a receiver can do that:

GET: /<projectnamespace>/<name>/<id>/details

You will get back something like this:

`{`<<BR>>
`  "receivers": [`<<BR>>
`    {`<<BR>>
`      "itsyou": false,`<<BR>>
`      "mid": 1,`<<BR>>
`      "cid": 2`<<BR>>
`      "pid": 19,`<<BR>>
`    },`<<BR>>
`    {`<<BR>>
`      "itsyou": false,`<<BR>>
`      "mid": 4,`<<BR>>
`      "cid": 3`<<BR>>
`      "pid": 29,`<<BR>>
`    }`<<BR>>
`  ],`<<BR>>
`  "senders": [`<<BR>>
`    {`<<BR>>
`      "mid": 5`<<BR>>
`    },`<<BR>>
`    {`<<BR>>
`      "mid": 7`<<BR>>
`    }`<<BR>>
`  ],`<<BR>>
`  "url": "courselinks/10",`<<BR>>
`  "content_type": "application/json"`<<BR>>
`  "owner": {`<<BR>>
`    "pid": 3,`<<BR>>
`    "itsyou": true`<<BR>>
`  }`<<BR>>
`}`<<BR>>

The "receivers" and "senders" have corresponding arrays: The first array
entry in "senders" has been addressed the first array entry of
"receivers" and so on.

=== Listen Ressource ===
''GET'': Liefert URI Nachrichten-Liste und "200" Status Code. Wenn Liste leer (leerer HTTP-Body, "Content-Length: 0") trotzdem "200" Statuscode. <<BR>>
''DELETE'': unzulaessig ("405" Method Not Allowed Statuscode). <<BR>>
''PUT'': unzulaessig ("405" Method Not Allowed Statuscode). <<BR>>
''POST'': Legt neue Message-Ressource an.

Ressourcenstruktur: `/<projectnamespace>/<name>` <<BR>>
Bsp. einer Listen-Ressource: `http://isblab.rus.uni-stuttgart.de/numlab/exercises` <<BR>>
The returned `Content-Type` will be `text/uri-list`. The URI list will be
represented by [[http://www.rfc-ref.org/RFC-TEXTS/3986/chapter5.html|relative references]].
URIs are specified in [[http://www.rfc-ref.org/RFC-TEXTS/3986|RFC3986]].

==== Subresource "details" ====
Now it's possible to ask for detailed (meta) information of a list resource.
All querystrings supported my normal list resources could be used.
Only the original sender can do that:

GET: /<namespace>/<name>/details

You will get back something like this:

`[`<<BR>>
`  {`<<BR>>
`    "senders": [`<<BR>>
``<<BR>>
`    ],`<<BR>>
`    "receivers": [`<<BR>>
``<<BR>>
`    ],`<<BR>>
`    "url": "courselinks/35",`<<BR>>
`    "content_type": "text/plain",`<<BR>>
`    "owner": {`<<BR>>
`      "pid": 3,`<<BR>>
`      "itsyou": true`<<BR>>
`    }`<<BR>>
`  },`<<BR>>
`  {`<<BR>>
`    "senders": [`<<BR>>
`      {`<<BR>>
`        "mid": 2`<<BR>>
`      }`<<BR>>
`    ],`<<BR>>
`    "receivers": [`<<BR>>
`      {`<<BR>>
`        "mid": 19,`<<BR>>
`        "cid": 2,`<<BR>>
`        "pid": 19,`<<BR>>
`        "itsyou": false`<<BR>>
`      }`<<BR>>
`    ],`<<BR>>
`    "url": "courselinks/36",`<<BR>>
`    "content_type": "text/plain",`<<BR>>
`    "owner": {`<<BR>>
`      "pid": 3,`<<BR>>
`      "itsyou": true`<<BR>>
`    }`<<BR>>
`  },`<<BR>>
`  {`<<BR>>
`    "senders": [`<<BR>>
`      {`<<BR>>
`        "mid": 2`<<BR>>
`      }`<<BR>>
`    ],`<<BR>>
`    "receivers": [`<<BR>>
`      {`<<BR>>
`        "mid": 19,`<<BR>>
`        "cid": 2,`<<BR>>
`        "pid": 19,`<<BR>>
`        "itsyou": false`<<BR>>
`      }`<<BR>>
`    ],`<<BR>>
`    "url": "courselinks/37",`<<BR>>
`    "content_type": "text/plain",`<<BR>>
`    "owner": {`<<BR>>
`      "pid": 3,`<<BR>>
`      "itsyou": true`<<BR>>
`    }`<<BR>>
`  }`<<BR>>
`]`<<BR>>

The first element of the returned array of the details list subresource
probably needs some explanation. Both senders and receivers are empty
lists. This mean that the appropriate message isn't any more addressed
to any participant. This further implies that all participants which
had been addressed in the past have been received the message from their
message queue. But why don't has been deleted the message then from the
ECS ? Because the resource has been configured to be "postrouted". If
that has not been the matter, ECS would has been removed the message.

<<Anchor(querystrings)>>
==== Querystrings ====
To affect the returned representation you could assign the following
querystrings to {{{X-EcsQueryStrings}}} header variable: 

'''receiver'''<<BR>>
It's possible to filter the returned index from a list resource to only
those items to which the calling participant was formerly an addressed receiver:<<BR>>
`curl .... -H 'X-EcsQueryStrings: receiver=true' -X GET https://server/<namespace>/<name>`<<BR>>
This is also the default, therefore it could be omited.

'''sender'''<<BR>>
It's possible to filter the returned index from a list resource to
only those items to which the calling participant is the original
sender:<<BR>>
`curl .... -H 'X-EcsQueryStrings: sender=true' -X GET https://server/<namespace>/<name>`

'''all'''<<BR>>
It's possible to filter the returned index from a list resource to
show all messages either as addressed receiver or as original sender:<<BR>>
`curl .... -H 'X-EcsQueryStrings: all=true' -X GET https://server/<namespace>/<name>`

Using the {{{X-EcsQueryStrings}}} header variable is the recommended way to use querystrings.
If you have to assign multiple querystrings please delimit the querystrings by comma (,).

Of course you can also specify the querystring by appending it to the end of the resource url, e.g.

`curl .... -X GET https://server/<namespace>/<name>?all=true`<<BR>>


<<Anchor(message-queues)>>
<<Anchor(queue-ressource)>>
=== Queue Ressource ===
Die Queue Ressource ist als Subressource einer Listen Ressource realisiert und kann entweder im ''lifo'' oder ''fifo'' Mode angesprochen werden:

''GET'': Liefert erste (fifo) oder letzte (lifo) Message-Ressource zurück. Gibt "200" Status Code zurück auch wenn keine Nachricht in Queue (leerer HTTP-Body, "Content-Length: 0"). <<BR>>
''DELETE'': unzulaessig ("405" Method Not Allowed Statuscode). <<BR>>
''POST'': Liefert erste (fifo) oder letzte (lifo) Message-Ressource zurück und löscht diese. Gibt "200" Status Code zurück auch wenn keine Nachricht in Queue (leerer HTTP-Body, "Content-Length: 0"). <<BR>>  
''PUT'': unzulaessig ("405" Method Not Allowed Statuscode).

Ressourcenstruktur: `/<projectnamespace>/<name>/fifo` oder `/<projectnamespace>/<name>/lifo` <<BR>>
Bsp. einer Queue-Ressource: `http://isblab.rus.uni-stuttgart.de/numlab/solutions/fifo`

<<Anchor(postrouting)>>
=== Postrouting ===
If a resource has set his ''postroute'' flag, then all new participants will
get postrouted this resources e.g. if you have posted some messages to a
community named ''testcommunity'' and later joins a new participant to this
community, it will get postrouted the former posted messages.

<<Anchor(participant_cluster)>>
== Participant Cluster ==
Nachfolgend wird beschrieben, wie der ECS einen Cluster über eine beliebige
Anzahl von Participants bildet, diesen Cluster zur Benutzung bereitstellt und
wie dieser angesprochen wird.

=== Cluster-Bildung ===
Zunächst eine Grafik zur Verdeutlichung der Topologie:

`  +---------+  +---------+  +---------+`<<BR>> 
`  | Partic. |  | Partic. |  | Partic. |`<<BR>>
`  |    A    |  |    B    |  |    C    |`<<BR>>
`  +---------+  +---------+  +---------+`<<BR>>
`       |            |            |     `<<BR>> 
`       |            |            |     `<<BR>> 
`  +-----------------------------------+`<<BR>>
`  |                ECS                |`<<BR>>
`  +-----------------------------------+`<<BR>>
`                    |                  `<<BR>> 
`                    |                  `<<BR>> 
`  +-----------------------------------+`<<BR>>
`  | virtueller Participant (Cluster)  |`<<BR>>
`  +--------+--------+--------+--------+`<<BR>>
`  | Cluster| Cluster| Cluster| Cluster|`<<BR>>
`  | Partic.| Partic.| Partic.| Partic.|`<<BR>>
`  |    1   |    2   |   3    |   n    |`<<BR>>
`  +--------+--------+--------+--------+`

 1. Der ECS registriert einen virtuellen Participanten d.h. es wird nur ein
 Zertifikat (Cluster-Zertifikat) generiert und ausgegeben. Dieses eine
 Zertifikat wird für alle Participanten benutzt, die den gewünschten Cluster
 bilden sollen. Das reduziert zum einen die Zertifikatsverwaltung (generieren,
 ausgeben, erneuern, ...) und zum anderen läßt sich der Cluster vollkommen
 transparent skalieren, indem man einfach einen weiteren Cluster-Participanten
 mit dem Cluster-Zertifikat konfiguriert. Dazu sind keine Registrierungen oder
 Konfigurationen am ECS notwendig.

 2. Nachrichten an den Cluster werden über eine Queue-Ressource
 geschickt. Die Adressierung des Clusters erfolgt einfach über den virtuellen
 Participanten. Jeder Cluster-Participant besorgt sich durch ein {{{DELETE}}}
 auf die Queue-Ressource eine neue Nachricht zur Bearbeitung. Dieses Abholen
 einer neuen Nachricht läuft innerhalb des Clusters konkurrierend unter allen
 Cluster-Participanten ab (message dispatching). 

{{{#!wiki comment/dotted
@stephan: virtuell heisst scheinbar. Dahinter steckt natürlich keinerlei extra
Instanz. Also keine zusätzliche Middleware oder sonstiges. Der Cluster wird
alleinig durch Verwendung eines gemeinsamen Zertifikats gebildet.

@stephan: Es ist eben gerade keine Änderung im "Backend" notwendig. Die
Cluster-Participants (Computation-Clients) können so bleiben wie sie sind,
benutzen aber dasselbe Zertifikat und werden vom ECS dadurch als ein einziger
Participant gesehen. Dies bezeichne ich als "virtuellen" Participant.
}}}

=== Cluster-Broadcasting ===
Da die Cluster-Participanten selbst nicht direkt/einzeln adressiert werden
können aber man trotzdem einzelne Cluster-Participanten ansprechen möchte,
bedient man sich dem sogenannten "broadcasting". Dazu wird eine
Message-Ressource (im weiteren als Broadcast-Ressource bezeichnet)
verwendet, in die ein Participant eine Nachricht schreibt.  Aus dem Cluster
heraus wird dann folgendermaßen auf diese Ressource zugegriffen:
 1. Jeder Cluster-Participant prüft die Broadcast-Ressource, ob dort eine neue
 Nachricht eingetroffen ist.  Ist dies der Fall wird die Nachricht idempotent
 aus der Broadcast-Ressource ausgelesen und überprüft, ob diese für den
 jeweiligen Cluster-Participanten gedacht ist. Wenn ja verarbeitet der
 Cluster-Participant die Nachricht.  
 2. Der ECS wird die Nachrichten in der Broadcast-Ressource nach
 voreingestellter Zeitdauer löschen (garbage collection).
 <<Anchor(cluster_broadcasting_gc)>>

<<Anchor(filter_plugins)>>
== Filter Plugins ==
Messages können über zur Laufzeit hinzufügbare Filter-Plugins manipuliert
werden. Ein Filter-Plugin (zukünftig als Filter bezeichnet) kann 5
unterschiedlichen Filter-Queues zugeordnet werden, die jeweils durch eine bestimmte
HTTP Operation getriggert werden. Die Filter-Queues werden dabei auf das
Dateisystem unterhalb des jeweiligen Basisverzeichnisses der Anwendung
abgebildet. Dabei richtet sich die Verzeichnisstruktur nach der
Ressourcen-URL ({{{/<project-name-space>/<name>}}} bzw. {{{/<project-name-space>/<name>/<id>}}}):  

 1. {{{show}}} Filter-Queue. Wird getriggert, sobald eine Queue/Message-Ressource abgerufen wird (GET): <<BR>>
 Filter-Queue Pfad: {{{app/controllers/<project-name-space>/filter/<name>/show}}}
 1. {{{index}}} Filter-Queue. Wird getriggert, sobald eine Listen-Ressource abgerufen wird (GET): <<BR>>
 Filter-Queue Pfad: {{{app/controllers/<project-name-space>/filter/<name>/index}}}
 1. {{{create}}} Filter-Queue. Wird getriggert, sobald eine Listen-Ressource aufgerufen wird (POST): <<BR>>
 Filter-Queue Pfad: {{{app/controllers/<project-name-space>/filter/<name>/create}}}
 1. {{{update}}} Filter-Queue. Wird getriggert, sobald eine Queue/Message-Ressource aufgerufen wird (PUT): <<BR>>
 Filter-Queue Pfad: {{{app/controllers/<project-name-space>/filter/<name>/update}}}
 1. {{{delete}}} Filter-Queue. Wird getriggert, sobald eine Queue/Message-Ressource aufgerufen wird (DELETE): <<BR>>
 Filter-Queue Pfad: {{{app/controllers/<project-name-space>/filter/<name>/delete}}}

Es können beliebig viele Filter in den jeweiligen Filter-Queues abgelegt
werden, wobei alle Filter entsprechend ihrer lexikalischen Sortierreihenfolge
nacheinander aufgerufen werden:

`unfiltered +-------+   +-------+   +-------+ filtered `<<BR>>   
`---------->| 1-fil |-->| 2-fil |...| n-fil |--------->`<<BR>>    
`message    +-------+   +-------+   +-------+ message  `

Wird ein Filter in die entsprechende Filter-Queue (Verzeichnis im Dateisystem)
kopiert, wird dieser automatisch zur Laufzeit dem ECS hinzugefügt. Es ist keine
Konfiguration oder gar ein Neustart des ECS notwenig (zero configuration;
convention over configuration).

=== Filter Template ===
Der Filter muß als Ruby Modul vorliegen und folgenden Aufbau besitzen:
{{{
modul BeliebigerModulname
  def self.start
    # hier kommt der eigentliche Filter
  end
end
}}}
Natürlich kann über die Startmethode des Moduls auch eine entsprechende Klasse
für eine objektorientierte Filterimplementierung angestoßen werden. Generell
darf zur Realisierung des Filters auf die komplette Ruby Funktionalität
zurückgegriffen werden.

Als API zum ECS wird durch die Konstante {{{FILTER_API}}} ein entsprechendes
Objekt bereitgestellt, mit dessen Hilfe auf die notwendigen ECS Objekte
zugegriffen werden kann. Momentan sind dies:

 1. {{{FILTER_API.params}}} Über diesen Hash kann unter anderem auf den Querystring des HTTP Aufrufs zugegriffen werden:
 {{{
 http://ecs.rus.uni-stuttgart.de/numlab/exercises/23?properties=name,description
 ...
 elements = FILTER_API.params["elements"].split(",")
 ...
 }}}

 1. {{{FILTER_API.record}}} Dieses Objekt erlaubt unter anderem den Zugriff auf die zu filternde Message:
 {{{
 message = FILTER_API.record.body
 }}}

== Transportprotokoll und Anwendungsprotokoll ==
Als Transport-/Anwendungsprotokoll dient HTTP 1.1 .

== Beispiele ==
Wie man den ECS konkret nutzt und als Client bedient, wird anhand einiger einfacher [[/examples|Beispiele]] erläutert.
