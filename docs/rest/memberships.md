##master-page:HomepagePrivatePageTemplate
##master-date:Unknown-Date
#acl Project/AdminGroup:admin,read,write,delete,revert EcsGroup:read,write,delete All:read
#format wiki
#language

=== memberships ===
Memberships stellen eine eindeutige Beziehung zwischen einem Participanten
(E-Learningplattform) und einer Community her und werden vom ECS Administrator
auf Anfrage/Antrag vergeben/eingetragen. Sollte ein Participant erstmalig in
einer Community des ECS in Erscheinung treten, wird ihm vom ECS Admin ein
entsprechendes Zertifikat (X509, SSL) ausgestellt, was ihm den sicheren und
authentifizierten Zugang zum ECS ermöglicht. Für alle etwaigen weiteren
Community Mitgliedschaften auf diesem ECS wird dasselbe Zertifikat verwendet
und es muss vom ECS Admin nur ein weiterer Communityeingetrag gemacht werden.
Der ECS Admin wird dem Participanten seine {{{certid}}}, seine neuen Community
Mitgliedschaften sowie seine damit verknüpften MembershipIDs explizit per email
mitteilen (diese Status-Info geht auch an alle bisherigen Community
Mitglieder).  Für Plattformen die nicht mehrere {{{communities}}} unterstützen,
sollten diese ihren Community Eintrag anhand ihrer zugewiesenen MembershipID
({{{mid}}}) oder ihrer {{{certid}}} aus der {{{memberships}}} Repräsentation
extrahieren und entsprechend anzeigen. Die {{{certid}}} wird direkt aus dem
ausgestellten Zertifikat gewonnen und kann auch vom Participanten vonselben
ermittelt werden ({{{certid}}} entspricht "Serial Number:" des Zertifikats).
<<Anchor(memberships_abr)>> Zusätzlich führt der ECS ein eindeutiges Kürzel
{{{abr}}} (abreviation) für jeden Participanten. Dieses Kürzel wird vom ECS nur
optional angeboten, doch falls vorhanden, ist dieses eindeutig bzgl. des
anbietenden ECS (siehe auch [[../auths#auths_abr|Autorisation]]).


'''GET, memberships:''': Gibt alle Participanten zugeordent zu ihrer jeweiligen Community zurück, in der auch der anfragende Participant Mitglied ist. 
{{{
https://infolms.rus.uni-stuttgart.de:7923/memberships
[
{
  "community": {
    "name": "SUV",
    "description": "Süddeutscher Universitäts Verbund"
  },
  "participants": [
    {
      "participantname": "Universität Stuttgart",
      "description": "Allgemeine eLearningplattform betrieben am Rechenzentrum Uni Stuttgart.",
      "dns": "ecs.uni-stuttgart.de",
      "email: "ecs@uni-stuttgart.de",
      "organization": "Uni Stuttgart",
      "mid": 5,
      "certid: "F55B205682709393",
      "abr": "S"
    },
      "participantname": "Universität Ulm",
      "description": "Zentrale ELearningplattform.",
      "dns": "ecs.uni-ulm.de",
      "email: "ecs@uni-ulm.de",
      "organization": "Uni Ulm",
      "mid": 24
      "certid": "F55B205682709394",
      "abr": "UL"
    }
  ]
},
{
  "community": {
    "name": "WAREM",
    "description": "Internationally Oriented Master of Science Program Water Resources Engineering and Management - WAREM"
  },
  "participants": [
    {
      "participantname": "Universität Stuttgart",
      "description": "Allgemeine eLearningplattform betrieben am Rechenzentrum Uni Stuttgart.",
      "dns": "ecs.uni-stuttgart.de",
      "email: "warem@uni-stuttgart.de",
      "organization": "Uni Stuttgart",
      "mid": 9
      "certid": "F55B205682709393",
      "abr": "S"
    },
      "participantname": "Universität Karlsruhe",
      "description": "E-Learningplattform der Fakultät Informatik.",
      "dns": "ecs.uni-karlsruhe.de",
      "email: "ecs@uni-karlsruhe.de",
      "organization": "Uni Karlsruhe",
      "mid": 12
      "certid": "F55B205682709395",
      "abr": "KA"
    }
  ]
}
]
}}}
'''GET, memberships/<mid>:''' Gibt eine Liste von Community / Participanten Paare aus, bei der der nachgefragte Participant Mitglied ist.
{{{
https://infolms.rus.uni-stuttgart.de:7923/memberships/5
[
{
  "community": {
    "name": "SUV",
    "description": "Süddeutscher Universitäts Verbund"
  },
  "participants": [
    {
      "participantname": "Universität Stuttgart",
      "description": "Allgemeine eLearningplattform betrieben am Rechenzentrum Uni Stuttgart.",
      "dns": "ecs.uni-stuttgart.de",
      "email: "ecs@uni-stuttgart.de",
      "organization": "Uni Stuttgart",
      "mid": 5
      "certid: "F55B205682709393",
      "abr: "S"
    }
  ]
}
]
}}}
'''DELETE, memberships/<id>''' Löscht Membership mit der mid=<id>. Nur der Besitzer der Membership kann diese auch löschen :
{{{
https://infolms.rus.uni-stuttgart.de:7923/memberships/5
}}}
