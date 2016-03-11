# Auslesen eines Drehstromzählers
1. Allgemeines
2. Datenaustausch
  * HDLC Typ 3


## 1. Allgemeines
Der verwendete Zähler in der Testumgebung ist ein EMH LZQJ-XC, mit dem man über
das häufig für Drehstromzähler verwendete Datenprotokoll DLMS kommunizieren kann.
Die Kommunikation zwischen unserem Programm und dem Zähler kann prinzipiell mit
dem Client-Server-Modell dargestellt werden. Der Zähler übernimmt dabei die Rolle
 des Servers und unser Programm die des Clients.

## 2. Datenaustausch
Zur veranschaulichung des Datenaustausches beziehen wir uns auf das
 OSI-Referenzmodell und beschreiben im nachfolgenden die für uns wichtigsten Schichten. Die erste OSI-Schicht (Bitübertragungsschicht _engl.:Physical Layer_), wird in unserem Fall durch das TCP/IP Protokoll realisiert,
 welches ein einfaches Ethernet Kabel als Kopplungseinheit nutzt.
 Das High-Level Data Link Control (_kurz: HDLC_) Protokoll vom Typ 3
 stellt die zweite OSI-Schicht (Sicherungsschicht _engl.:Data Link Layer_)
 dar. Auf das HDLC Protokoll wird später intensiv eingegangen, da dieses den Blockaufbau der übertragenen Frames festlegt und somit wichtig für das Programm ist.
 Unser Programm stellt die siebte Schicht (Anwendungsschicht _engl.:Application Layer_ ) dar. Für die anderen Schichten, sei auf die untere Abbildung (Abb. 1.1) verwiesen.

<dl>
<img src="https://raw.githubusercontent.com/Ac719237/ACS/master/OSI.jpg" alt="https://raw.githubusercontent.com/Ac719237/ACS/master/OSI.jpg" style="width: 600px;"/>
</dl>

### HDLC Typ 3
Das HDLC Typ 3 Protokoll Wie bereits erwähnt
