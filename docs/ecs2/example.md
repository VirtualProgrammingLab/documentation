<<TableOfContents(3)>>

= Beispiele =
Als Client wird [[http://de.wikipedia.org/wiki/CURL|curl]] verwendet.

== Persistente Message erzeugen (X.509 zertifizierter Client) ==
Unter der Ressource {{{https://ecs.uni-stuttgart.de/numlab/exercises}}} neue
persistente Message erzeugen (POST auf
{{{https://ecs.uni-stuttgart.de/numlab/exercises}}}). Message wird an 
participant mit membership id {{{8}}} adressiert:
{{{
curl -i -X POST 
--cacert freeit-root-ca.cert.pem  --cert client.cert.pem --key client.key.pem --pass "ganzgeheim" \
-H "Content-Type: application/json" \
-H "X-EcsReceiverMemberships: 8" \
-d '
{
  "identifier"  : "Numerik II, SS10, Aufgabe 1, v1.0",
  "comment"     : "Um die Studenten mal richtig zu fordern.",
  "name"        : "Aufgabe 1",
  "description" : "Schreiben Sie eine C-Funktion..."
}
' \
https://ecs.uni-stuttgart.de/numlab/exercises
}}}
Mögliche Antwort:
{{{
HTTP/1.1 201 Created
Location: https://ecs.uni-stuttgart.de/numlab/exercises/274
Content-Type: application/json; charset=utf-8

<leerer http-body>
}}}

== Eintrag in Message-Queue erzeugen (anonymer Client) ==
Unter der Ressource {{{https://ecs.uni-stuttgart.de/numlab/solutions}}} neue
Message in Message-Queue erzeugen (POST auf
{{{https://ecs.uni-stuttgart.de/numlab/solutions}}}). Message wird an
participant mit membership id {{{8}}} adressiert. Der Client besitzt zum
Zeitpunkt der Anfrage noch kein gültiges Cookie (anonym). Ihm wird dieses durch
den Respond-Header (Rückantwort vom ECS) mitgeteilt (siehe auch Core ECS):
{{{
curl -i -X POST 
-H "Content-Type: application/json" \
-H "X-EcsReceiverMemberships: 8" \
-d '
{
  "comment" : "Die war aber schwer!",
  "texts" :
    [
      { "name"      : "editable source",
        "editable"  : false,
        "type"      : "C",
        "emphasis"  : "medium",
        "content"   : "void bar() { printf("bar!\n"); }"
      }
    ],
  "solutionPostingTime" : "1985-04-13T17:10:00.52Z"
}
' \
http://ecs.uni-stuttgart.de/numlab/solutions
}}}

Mögliche Antwort:
{{{
HTTP/1.1 201 Created
Location: http://ecs.uni-stuttgart.de/numlab/solutions/89
Set-Cookie: ecs_anonymous=59c3aaeead64514b83abcea9f81cece9; path=/

<leerer http-body>
}}}

== Abfragen einer Message Queue durch authentisierten Participanten ==
Die Queue Ressource {{{https://ecs.uni-stuttgart.de/numlab/solutions}}} soll
von einem authentisierten Participanten (z.B. Computation client) abgefragt
werden. Keine clientseitige zertifikatsbasierte Authentisierung (kein
Clientzertifikat und kein Clientpasswort), sondern nur eine serverseitige
Authentifikation (mit dem freeit-root-ca.cert.pem Rootzertifikat Prüfung des ECS
Serverzertifikats):
{{{
curl -i \
--cacert freeit-root-ca.cert.pem \
-H "X-EcsAuthId: tunichtgut" \
-H "Accept: application/json" \
-X POST https://ecs.uni-stuttgart.de/numlab/solutions/fifo
}}}

Mögliche Antwort:
{{{
HTTP/1.1 200 OK
Connection: close
Date: Thu, 03 Dec 2009 15:08:33 GMT
ETag: "45fd14239a00353b4877cb1bd12822Xt"
Last-Modified: Tue, 13 Oct 2009 15:23:53 GMT
Content-Type: application/json; charset=utf-8
X-Runtime: 32
Content-Length: 310
Cache-Control: private, max-age=5

<BODY>
}}}

== Abfragen einer Message Queue durch anonymen Participanten ==
Die Queue Ressource {{{https://ecs.uni-stuttgart.de/numlab/results}}} soll
von einem durch Cookie identifizierten (Cookie:
ecs_anonymous=92cee57f228a999bbf3c0bbf162d9cf20210a9cb), anonymen Participanten
(z.B. Presentation client) abgefragt werden (serverseitige Authentifikation mit
dem freeit-root-ca.cert.pem Rootzertifikat):
{{{
curl -i \
--cacert freeit-root-ca.cert.pem \
-H "Cookie: ecs_anonymous=59c3aaeead64514b83abcea9f81cece9" \
-H "Accept: application/json" \
-X POST https://ecs.uni-stuttgart.de/numlab/solutions/fifo
}}}

Mögliche Antwort:
{{{
HTTP/1.1 200 OK
Server: nginx/0.7.65
Date: Mon, 29 Mar 2010 23:31:53 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
ETag: "86c90986c7397a6df3779550ccf99b9e"
Last-Modified: Mon, 29 Mar 2010 23:29:32 GMT
X-EcsReceiverCommunities: 3
X-EcsSender: 8
X-Runtime: 202
Content-Length: 25
Set-Cookie: ecs_anonymous=92cee57f228a999bbf3c0bbf162d9cf20210a9cb; path=/; expires=Tue, 30-Mar-2010 00:31:53 GMT
Cache-Control: private, max-age=5

<BODY>
}}}


<<Anchor(exercises_show_filter)>>
== Exercises Filter-Plugin ==
'''Filterung einer Exercises Message-Ressource nach dem {{{name}}} Property:'''
{{{
curl -i -X GET http://ecs.uni-stuttgart.de/numlab/exercises/27?properties=name
}}}
Mögliche Antwort:
{{{
HTTP/1.1 200 OK
Connection: close
Date: Thu, 03 Dec 2009 15:08:33 GMT
ETag: "45fd14239a00353b4877cb1bd12822ed"
Last-Modified: Tue, 13 Oct 2009 15:23:53 GMT
Content-Type: text/html; charset=utf-8
X-Runtime: 32
Content-Length: 310
Cache-Control: private, max-age=5

{ "Exercise" :
  {
  "name" : "Aufgabe 1: C-Funktion zur Integralberechnung"
  }
} 
}}}

'''Filterung einer Exercises Message-Ressource nach dem {{{name}}} und {{{description}}} Property:'''
{{{
curl -i -X GET http://ecs.uni-stuttgart.de/numlab/exercises/27?properties=name,description
}}}
Mögliche Antwort:
{{{
HTTP/1.1 200 OK
Connection: close
Date: Thu, 03 Dec 2009 15:08:33 GMT
ETag: "45fd14239a00353b4877cb1bd12822ed"
Last-Modified: Tue, 13 Oct 2009 15:23:53 GMT
Content-Type: text/html; charset=utf-8
X-Runtime: 32
Content-Length: 310
Cache-Control: private, max-age=5

{ "Exercise" :
  {
  "name" : "Aufgabe 1: C-Funktion zur Integralberechnung",
  "description" : "Schreiben Sie eine C-Funktion trapez(a,b,n,f) zur numerischen Berechnung des Integrals einer Funktion f im Intervall [a,b] mit Hilfe der aufsummierten Sehnentrapezregel mit n äquidistanten Stützstellen."
  }
} 
}}}


{{{{#!wiki comment/dotted

== Liste aller Messages ==
Unter der Ressource {{{http://ecs.uni-stuttgart.de/numlab/exercises}}} alle Messages als Liste anfordern (GET auf {{{http://ecs.uni-stuttgart.de/numlab/exercises}}}):
}}}}
