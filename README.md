# Speicherlecks

### Copyright Geschichte

[Dr.Memory](http://drmemory.org/docs/index.html) steht unter der LGPL v2.1 (1999). [Link zu Dr.Memory auf github](https://github.com/DynamoRIO/drmemory)<BR>
Ich bin mir nicht sicher, ob das Pflicht ist, meine Uebersetzung mit der gleichen Lizenz zu versehen. Deshalb lasse ich das mal bleiben und werde mir im Anschluss Gedanken machen, welche Lizenz ich diesem Text gebe.

Weil das im Grunde eine Uebersetzungsuebung ist ;), ist es mir egal, wer das wo verwendet.

## Einfuehrung

Das Verstaendnis vom Speicheraufbau und der Funktionsweise des Programmes ist eine wichtige Grundlage, um Speicherlecks im Quelltext von Programmen zu finden. Um das hier eine reine Uebersetzung zu halten, koennen durchaus fachliche Fehler enthalten sein, die eben meiner Unkompetenz entspricht. Dies ist keine wissenschaftliche Arbeit und ist kein Ersatz fuer eine Theoriegrundlage, sondern eine Hilfestellung in der Praxis.<BR>
Ich werde mich fuer alle Hinweise auf Fehler bedanken. Ich bin per Hochschul-Email erreichbar.<BR>
Weil man mit Dr.Memory keine Java Programme auf Speicherfehlzugriffe untersuchen kann, teile ich den Java Teil in ein anderes Repository ein. Ein weiterer Grund fuer die Teilung ist der Unterschied zwischen den Lizenzen von Java und Dr.Memory.

## Speicherleck

Wenn ein Programm Speicher reserviert, nicht wieder freigibt und stattdessen auch noch weiterhin Speicher reserviert, wird dies als Speicherleck bezeichnet. Man sagt das Programm "verliert" Speicher. Gemeint ist damit, dass sich der fuer das Programm verfuegbare Speicher immer weiter verkleinert. Das Programm muss neu gestartet werden, denn sonst wird es auf Grund des Speichermangels zu Systemfehlern kommen. (Quelle: [www.itwissen.info](http://www.itwissen.info/definition/lexikon/Speicherleck-memory-leak.html))

## Was ist Dr.Memory?

Das Dr. steht fuer DynamoRIO.<BR>
Das genannte Programm ist ein Speicher Ueberwachungswerkzeug, das spezialisiert ist fehlerhaften Speicherzugriff von geschriebenen C/C++ Programmen zu finden.
Der Debugger ist kompatibel mit Windows und ist sehr einfach installierbar. Er ist ebenfalls auf Linux und Mac OS installierbar.<BR>
Laut Angaben auf der Homepage von Dr.Memory ist die Debug-Geschwindigkeit sogar hoeher als sein Verwandter valgrind auf Linux.

## Welche Voraussetzungen muessen Programme erfuellen?

* geschrieben in C/C++
* derzeit beschraenkt auf 32-bit
* debug-Informationen in den ausfuehrbaren Dateien notwendig (siehe Tabelle im Anschluss)

| Dateityp (Betriebssystem)     | Debug-Information |
| ----------------------------- | ----------------- |
| ELF (Linux)                   | DWARF2            |
| Mach-O (Mac OS)               | DWARF2            |
| PDB (Windows / Visual Studio) | PDB               |
| PECOFF (Windows / MinGW gcc)  | DWARF2            |

Auf 64-bit Betriebssystemen impliziert das eine Installation von 32-bit Library.<BR>
Und man muss eventuell seine Programme besonders kompilieren.
* Unter gcc sind die Optionen -m32 -g Pflicht. Moeglichst mit einen -O0 kompilieren.
* Unter Visual Studio (bin noch kein Verwender) kann man sich durchklicken, dass das /Zi bzw /ZI Option uebergeben wird und zusaetzlich die /DEBUG (ja nicht /DEBUG:OPTIMIZATION).

### Was kann Dr.Memory nicht?

* 64-bit Programme debuggen
* Programm pausieren
* Systemfehler von Programmfehlern unterscheiden (manche Fehler aus Systembibliotheken werden schon unterdrueckt)

## Aufruf von Dr.Memory

Es muss die entsprechende PATH-Variable auf den bin-Ordner im installierten Ordner gesetzt sein. Das passiert mit der Windows-.msi-Installation automatisch.<BR>
Sonst kann man drmemory auch mit dem vollstaendigen Pfad aufrufen (, wie es moeglich ist fuer jede andere Anwendung).

Der Aufruf lautet folgendermassen:
    drmemory [Dr-Memory-Optionen] -- Pfad-zur-ausfuerbaren-Datei [Anwendungsargumente fuer die zu testende Anwendung]

Fuer alle Kommandozeilen-Neulinge / -Feinde: Man kann es in Visual Studio integrieren. Wenn ich das machen muss, dann schreibe ich meinen Weg dorthin. 

Vorneweg: Alles Optionen, die Sachen anschalten, koennen bei Bedarf mit kleiner Aenderung ausgeschaltet werden. Man muss dafuer die Option mit 
    -no
ausfuehren. Z.B. -count_leaks (zaehlen von entdeckten Speicherlecks) kann zu -no_count_leaks umgeschrieben werden.

Nuetzliche Dr.Memory-Optionen:
    -logdir /Pfad/zum/Ordner
Der angegebene Ordner muss existieren. In dem Ordner wird pro Ausfuehrung eines Tests ein Ordner mit dem Namen der getesten Anwendung erzeugt. Alte Ordner werden nicht ueberschrieben. 

    -batch
Unterdrueckt den Aufruf von einem Editor. Unter windows wird ohne diese Option sonst ein Notepad Fenster mit dem Testergebnis geöffnet.

    -nudge <processid>
Die angegebene Prozess-ID muss zu einem Prozess gehoeren, der von Dr.Memory beobachtet wird. Dr.Memory wertet den Speicher aus und schreibt die Speicher-Fehler in die Logdatei result.txt auf. Dies passiert unter normalen Umstaenden erst beim Programmende. Ein Anwendungsbeispiel sind Daemons.<BR>
Diese Option ist nicht vorhanden in MacOS.

    -suppress /Pfad/zur/suppress.txt
Die Regeln in der suppress.txt Datei werden eingelesen. Die Defaultsupress-Datei befindet sich unter Win7 in Installationsordner/bin/suppress-default.txt

    -log_suppressed_errors
Um alle unterdrueckten Fehlermeldungen zu dokumentieren kann diese Option verwendet werden. Die Datei, die alle Fehlermeldungen enthaelt hat global in ihrem Namen (, wenn man -logdir verwendet).

suppress Dateien sind so zu lesen:

Es muss immer der vollstaendige Dateiname angegeben werden.

    Modul-Dateiname!Funktion

Das Ausrufezeichen trennt den Dateinamen von der Funktion. Es werden die wildcards '*' und '?' zum bezeichnen unterstuetzt. Alle entdeckten Fehler in der Programmierung der Funktion oder der Funktionen werden unterdrueckt.<BR>
Die Funktionsaufrufe von *!__* unterdruecken alle Fehler in allen Modulen in den Funktionen, die mit doppelten Unterstrich beginnen (Systemfunktionen).<BR>
Der Wildcard *!echo? unterdrueckt alle echo-Funktionen mit einem Zeichen dahinter (echo1, echo2, echo3, echoa)<BR>
Eine weitere nuetzliche Variante ist das unterdruecken vom Aufrufscheck von einem Modul per Modulname!*<BR>


## Weitere Optionen zum Aufruf von Dr.Memory
    -light [-count_leaks]
Dieser Modus dient dazu die Leistung von Dr.Memory zu erhoehen durch verringern der detektierten Fehlersorten. Es werden nicht mehr uninitialisierte Lesezugriffe und Speicherlecks detektiert. In dem Modus detektiert Dr.Memory unadressierbare Zugriffe (Adressen ausserhalb vom gueltigen Adressraum verstehe ich darunter), ungueltige Heapargumente und GDI (Graphics device interface) Fehler und Warnungen.<BR>
Das -count-leaks bewirkt, dass wieder Speicherlecks detektiert werden.

    -report_max <int>
Gibt die maximale Anzahl von zurueckzugegebenen NICHT-Speicherlecks-Fehler an. Wird -1 angegeben wird keine Grenze gesetzt.

    -report_leak_max <int>
Dies ist analog zu interpretieren.

## Die Logdateien

Die Logdateien sind entweder im Standardordner oder im Ordner, der per -logdir angegeben ist. Die Unterordnernamen enthalten den Namen der ausfuehrbaren Datei und die Prozess-ID von der von Dr.Memory ueberwachten Ausfuehrung der genannten Datei.<BR>
Die result.txt enthaelt alle nicht unterdrueckten fehlerhaften Speicherzugriffe. Fuer die besonders Gruendlichen sorgt die Option -log\_suppressed\_errors fuer das schreiben aller fehlerhaften Speicherzugriffe in den global.XXX.log.

* Unadressierbarer Zugriff (Unaddressable Access): Das ist ein Zugriff auf Speicher, der nicht allokiert ist. Beispiele sind: Buffer overflow (meine Uebersetzung: Puffer Ueberlauf), lesen hinter dem Ende eines Arrays, lesen oder schreiben von freigegebenen Speicher (free / delete), lesen von ueber dem Stack.<BR>  Die Ausgabe gibt dabei erst Speicherwerte aus, wo gelesen wird. Dr.Memory gibt auch zusaetzlich Speicherwerte in der Naehe aus, wo allokierte Speicherwerte liegen koennen. Dies ist ein Hinweis darauf, wo der Programmierer womoeglich (man weiss es nicht) zugreifen wollte.
* Lesen von nicht-initialisierten Speicher (Uninitialized Read): Der Speicher wurde allokiert. Allerdings wurde kein Wert eingeschrieben. Bis mit dem Speicher ein Lesezugriff, auch im Zusammenhang mit Vergleichen geschieht, wird Dr.Memory keinen Fehler ausgeben. Ebenso wird die Uebergabe eines nicht-initialisierten Speicher bei Funktionsuebergabe.<BR> Es werden Speicheradressen oder bei groesseren Datenstrukturen Speicherbereiche mitangegeben. Ebenso werden Modulnamen!Methodenname ausgedruckt. Dabei muss man dann in der Methode suchen, wo der Fehler liegt.<BR> Es kann hier zu falschen positiven Ergebnissen kommen, denn Variablen, die kleiner sind als ein Wort, werden in einem Speicherplatz so gross wie ein Wort gespeichert. Der nicht benoetigte Platz ist, allerdings nicht-initialisiert und kann Fehlermeldungen verursachen.
* Ungueltiges Heapargument (Invalid Heap Argument): Am einfachsten ist ein Beispiel: INVALID HEAP ARGUMENT: free 0x00001234 <BR> Es drueckt aus, dass die Adresse kein gueltiger Speicher ist. <BR> Es wird als Hilfe fuer die Fehlersuche der Modulname!Funktionsname ausgedruckt.
* Speicherlecks (Memory Leaks): Diese werden unterteilt in moegliche Speicherlecks und definitiv unerreichbare Speicher. Das Kriterium fuer moegliche Speicherlecks ist, dass der Zeiger (mit dem der Speicher allokiert wurde) in die Mitte des allokierten Speicherbereich zeigt. <BR> Analog sind definitiv unerreichbare Speicher Zeiger, die nicht mehr auf ihren allokierten Speicher zeigen.<BR> Die Speicherlecks werden jeweils noch weiter unterteilt in indirekte und direkte Speicherlecks. Indirekte Speicherlecks sind Speicherbereiche, die verwiesen werden. Allerdings sind die Verweise nicht mehr ansprechbar durch fehlende Zeiger. Ein direkter Speicherleck fehlt der Zeiger, der direkt auf den allokierten Speicher verweist.
* Warnung (Warning): Dies sind Meldungen, die den Entwickler evtl. interessieren koennen.
* GDI (Graphical Device Interface) Usage Errors & Handle Leaks: Dies ist fuer die Vollstaendigkeit erwaehnt. Diese Fehler waren primaer nicht im Fokus als ich mich mit dem Thema befasse wollte. Die Handle Leaks sind in der Bedeutung ein fehlen von Ausnahmebehandlungen unter Windows.

### Ausblick

Ich lese es wieder je nachdem, was ich von der Doku brauche. Wenn Unklarheiten bestehen, besser ich es auf. <BR>
Wenn ich Visual Studio Benutzer bin, ergaenze ich die Anleitung zur Verwendung von Dr.Memory in Visual Studio.<BR>
Anmerkungen wegen Fehlern sind mir immer sehr willkommen. Die, die mich kennen, koennen mich per Email erreichen.<BR>
Code selber fuer Logbeispiele zu schreiben faellt mir gerade schwer, weil ich einen genialen Quelltext neulich abgeschrieben habe mit angeblich allen moeglichen Fehlern von Speicherzugriffen. Sprich, ich gerate in die Gefahr eines Copyright-Deliktes, wenn ich nicht nachfrage. Verbrechen rauben mir den Schlaf.