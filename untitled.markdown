# Auslesen eines Drehstromzählers
1. Allgemeines
2. Datenaustausch
  * High-Level Data Link Control Protokoll
3. Das Programm
  * Verbindungsaufbau und Auslesen


## 1. Allgemeines
Der verwendete Zähler in der Testumgebung ist ein EMH LZQJ-XC, mit dem man über
das häufig für Drehstromzähler verwendete Datenprotokoll DLMS kommunizieren kann.
Die Kommunikation zwischen unserem Programm und dem Zähler kann prinzipiell mit
dem Client-Server-Modell dargestellt werden. Der Zähler übernimmt dabei die Rolle
 des Servers und unser Programm die des Clients.  

## 2. Datenaustausch
Zur veranschaulichung des Datenaustausches beziehen wir uns auf das
 OSI-Referenzmodell und beschreiben im nachfolgenden die für uns wichtigsten Schichten.     
Die erste OSI-Schicht (Bitübertragungsschicht _engl.:Physical Layer_), wird in unserem Fall durch das TCP/IP Protokoll realisiert,
 welches ein einfaches Ethernet Kabel als Kopplungseinheit nutzt.
 Das High-Level Data Link Control (_kurz: HDLC_) Protokoll
 stellt die zweite Schicht des OSI-Referenzmodells (Sicherungsschicht _engl.:Data Link Layer_)
 dar. Auf das HDLC Protokoll wird später intensiv eingegangen, da dieses den Blockaufbau der übertragenen Frames festlegt, welche unter anderem die auszulesenden Informationen enthält und somit essentiell wichtig für das Programm ist.
 Unser Programm stellt die siebte Schicht (Anwendungsschicht _engl.:Application Layer_ ) dar.   
 Für die anderen Schichten, sei auf die untere Abbildung (Abb. 2.1) verwiesen.  

<dl>
<img src="https://raw.githubusercontent.com/Ac719237/ACS/master/OSI.jpg" alt="https://raw.githubusercontent.com/Ac719237/ACS/master/OSI.jpg" style="width: 600px;"/>
</dl>
_Abb. 2.1 OSI Referenzmodell, bezogen auf das DLMS Protokoll_

### High-Level Data Link Control Protokoll  
Das HDLC Protokoll, ist ein sogenanntes bitorientiertes Netzwerkprotokoll, dessen Aufgabe es unter anderem ist, den Datenfluss zu steuern und Übertragungsfehler durch Blockprüfung zu erkennen. Auch enthalten die vom Server kommenden Frames, die für uns wichtigen Messdaten, sodass eine genauere Betrachtung dieses Protokolls wichtig ist. Der Blockaufbau eines HDLC Frames ist in Abb. 2.2 dargestellt.  
Es folgt eine detailierte Beschreibung der einzelnen Bestandteile eines Frames.     <dl><img src="https://raw.githubusercontent.com/Ac719237/ACS/master/Frame%20format.png" alt="https://raw.githubusercontent.com/Ac719237/ACS/master/OSI.jpg" style="width: 650px;"/>
</dl>
_Abb. 2.2 Blockaufbau eines HDLC Frames_

**FLAG**  
Das Flag kennzeichnet immer den Anfangs-und den Endpunkt eines Datenpaketes und besteht aus einem Byte mit dem Hexadezimalen Wert  
  > **0x7E** (=01111110 Binär).  

Dabei kann das Endflag eines Frames gleichzeitig das Anfangsflag, des darauffolgenden Frames sein. Um Verwechslungen auszuschließen, wird 'Bit-Stuffing' verwendet.

**Frame Format**  
Das Frame Format Feld, besteht aus zwei Bytes, wovon das erste das Format des Frames und das zweite die Länge des Datenpaketes (ohne Start- und Endflag) angibt.   
* In unserem Fall ist das Format-Byte immer
>**0xA0** (HDLC Typ 3).  

  Das zweite Byte variiert je nach Länge des Datenpaketes.

**Destination Adress und Source Adress**  
Das Destination Adress Feld gibt die Zieladresse des Datenpaketes an und das Source Adress Feld die Adresse von der das Datenpaket kommt. Häufig werden beide Felder zusammengefasst und als 'Adress Field' bezeichnet. Zu beachten ist hierbei, dass Client und Server Adresse je nach Richtung des Frames mal Zieladresse und mal Startadresse sein können.  
 Das LSB (Least Significant Bit) eines jeden Bytes im Adress Field gibt an, ob ein erweitertes Adressfeld benutzt wird, wobei eine '0' auf ein erweitertes Feld hinweist.  

 * Die Adresse des Clients wird immer durch ein Byte ausgedrückt und ist in unserem Fall
 >**0x21** (=100001 Binär).  

  Das letzte Bit gibt hier also nur an, dass kein erweitertes Adressfeld benutzt wird.
 * Die Adresse des Servers besteht aus einer logischen Adresse (Upper HDLC Adress) und einer physikalischen Adresse (Lower HDLC Adress), welche jeweils durch zwei Byte ausgedrückt werden.  
 Die logische Adresse ist in unserem Fall immer 16
 > **0x0020** (=0000 0000 0010 0000 Binär)

  Die physikalische Adresse berechnet sich zu
             (SerialNumber) mod 10000 +256

Für unseren Zähler folgt 5190. In Hexadezimalzahlen ausgedrückt  
> **0x508D** (=0101 0000 1000 1101 Binär)

**Control Field**  
Das Control Field gibt an um was für eine Art von Datenpaket es sich handelt.  
 Dabei wird unterschieden zwischen Information Frames (zur Datenübertragung), Supervisory Frames (zur Steuerung des Datenflusses) und Unnumbered Frames (zur Steuerung der Verbindung). Die Länge des Feldes variiert je nach Art zwischen einem und zwei Bytes.

**Header Check Sequence (HCS)**  
Das HCS Feld besteht aus zwei Bytes und beeinhaltet eine Prüfsumme, die sich aus allen Bits zwischen Anfangsflag und HCS Feld ergibt. Datenpakete die keine Information transportieren (z.B. Supervisory Frames), besitzen kein HCS Feld, sondern lediglich ein FCS (Frame Check Sequenze Feld).  
 Die Berechnung dieser Prüfsumme ist sehr komplex, trotzdem wird kurz auf die Berechnungsweise eingegangen. Die übertragenen Bits werden mittels Polynomdivision durch ein vorher festgelegtes Generatorpolynom (für HDLC: CRC-CCITT ) dividiert. Der Rest dieser Division ergibt die Prüfsumme. Der Empfänger kennt das Generatorpolynom und kann daraufhin wieder eine Division mit dem empfangenen Bitstring und dem Generatorpolynom durchführen. Falls der Rest dieser Division Null ist, wurde das Datenpaket korrekt übertragen.

 **Information Field**  
 Das Information Field enthält die übertragenen Informationen. In unserem Falle wären das die Messdaten des Drehstromzählers.  
 Wie bereits oben erwähnt, besitzen nicht alle Datenpakete ein Information Field.

 **Frame Check Sequenze Field**  
 Das FCS Feld enthält wie das HCS Feld eine Prüfsumme und berechnet sich auch genau gleich. Der Unterschied zum HCS Feld ist, dass die Bits des Information Feldes mit in die Berechnungen einbezogen werden.  

 ## 3. Funktionsweise des Programmes
Im folgenden wird auf die Funktionsweise des Programmes eingegangen. Das Programm wurde mithilfe des Gurux Frameworks erstellt, welches eine Open Source API zum Verbinden und Auslesen von Smart Metern darstellt.  
Es sei angemerkt, dass wir uns nun in der siebten (und damit in der höchsten) Protokollschicht des OSI-Modells befinden.
### Verbindungsaufbau und Auslesen
Zunächst wird die Betriebsart des HDLC-Protokolls festgelegt. Grundsätzlich gibt es 3 verschiedene Betriebsarten, wobei wir hier den Halbduplex Modus (Normal Response Mode) verwenden. Dies bedeutet, dass Daten abwechselnd, aber nicht gleichzeitig, in beide Richtungen fließen können.  
Dazu wird ein SNRM (Set Normal Response Mode) Frame verschickt.  

> **SNRM Frame**  7E A0 OA 00 20 50 8D 21 93 DB CD 7E

Danach wird ein AARQ (A-Association Request) Frame verschickt.
  Als antwort erhalten wir ein Datenpaket, welches für die Verbindung wichtige Informationen enthält und diese in Form eines PDU (Protocol Data Unit) an die tieferliegenden Schichten weitergibt.  
Das Gurux Framework benötigt nur diese beiden Frames um eine Verbindung zwischen Server und Client aufzubauen.   
Das Auslesen erfolgt durch versenden von Frames, die auf bestimmte Daten innerhalb des Zählers verweisen. Dazu ist wichtig zu verstehen, dass ein physikalischer Zähler aus mehreren Logischen Objekten besteht. Diese Logischen Objekte enthalten die von uns gewünschten Informationen und können durch HDLC Frames addressiert werden. Um herauszufinden in welchem Logischen Objekt sich welche Informationen befinden, kann man Frame-Sniffer verwenden. Für unsere Zwecke reicht der von Gurux zur Verfügung gestellte GXDLMSDirector aus.  
Für den Verbrauch in Kwh verschicken wir zum Beispiel das Frame  
> 7E A0 14 00 20 50 8D 21 32 B0 4A E6 E6 00 05 01 02 07 08 68 52 7E

und erhalten das Frame  
> 7E A0 17 21 00 20 50 8D 52 68 4C E6 E7 00 0C 01 00 05 00 00 00 XX B1 EF 7E

zurück. Die Stelle XX kennzeichnet hierbei den Verbrauch in Kwh (nach Umwandlung von Hexadezimal in Dezimal).  
Die Leistung wird danach in eine MySQL Datenbank abgespeichert.
