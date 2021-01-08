# Übersicht

Übersicht siehe [Übersichtsdatei](plugin.md)

# Zugang

Das Plugin ist im Gradle Plugin-Repository unter der id https://plugins.gradle.org/plugin/de.seitenbau.serviceportal zu finden.

# Versionen

* 1.0

# Verwendung des Gradle-Plugin

## Hintergrund und Ziele des Gradle-Plugins zur Prozessmodellierung

Da die Entwicklung von Prozessen immer komplexer wird und die existierenden Tools den Ansprüchen 
nicht gerecht werden, soll mittels eines Gradle-Plugins die Möglichkeit geschaffen werden
einzelne Teile getrennt voneinander zu entwickeln und so die Komplexität beherrschbar zu machen. 

Das Gradle-Plugin fügt die einzelnen Teile zu einem Modell zum Betrieb auf der Plattform zusammen
und spielt dieses Gesamtmodell zum Testen auf eine Zielplattform ein.

Dieser Ansatz ist für fortgeschrittenen Entwickler gedacht, die lokal arbeiten können
und wollen und nicht den Umweg über die Weboberflächen gehen möchten. 
Die Weboberflächen sollen weiterhin als gleichwertige Alternative für weniger technisch orientierte
Entwickler erhalten bleiben und weiterentwickelt werden. 

Der Ansatz trennt folgende Bereiche:

* Prozessmodelle ohne Skripte (via eigenem Tool)
* Skripte als "Software" mit entsprechenden Möglichkeiten und Hilfen zur Entwicklung 
  (via beliebiger IDE)
* Formulare (via eigenem Tool)
* Zusammenfügen, hochladen und deployen (dieses Gradle-Plugin)

## System-Anforderungen

* Das Plugin ist mit Gradle 6 getestet. Eventuell sind die Funktionen auch mit niedrigeren 
Gradle-Versionen verfügbar.
* Das verwendete Gradle muss minimal unter Java 11 laufen.

## Parameter für alle Tasks

Alle vom Plugin bereitgestellten Gradle-Tasks können mit den folgenden Parametern konfiguriert 
werden:

* **mandant**: Optional. Überschreibt den im config-Verzeichnis festgelegten Mandanten.

* **environment**: Optional. Falls nicht gesetzt, wird die Umgebung "default" verwendet.
* **debug** Optional. Falls der Parameter den Wert `true`hat, werden auf der Kommandozeile Logeinträge 
ab Loglevel DEBUG ausgegeben. Falls nicht gesetzt, werden Logeinträge ab Loglevel INFO ausgegeben.

## Bereitgestellte Gradle-Tasks

### Task _mergeScripts_
 
Zweck dieses Tasks ist es, dass der Quellcode eines Groovy-Script-Tasks auf mehrere Dateien
aufgeteilt werden kann. 
Der Task fügt mithilfe der `import`-Anweisungen in den Skripten rekursiv mehrere Groovy-Dateien 
zu einer Datei zusammen, die in sich geschlossen ist.
 
Dabei werden alle Prozessmodell-Dateien im Verzeichnis `models` 
gescannt. In den gefundenen Prozessmodellen werden die Skript-Tasks durchlaufen und im Ordner `scripts` nach 
nach entsprechenden groovy-Dateien `scripts/[processmodel-id]/[skript_task_id].groovy` gesucht.
Dort werden die verwendeten Imports durchlaufen und eine in sich geschlossen lauffähige Scriptsdatei erstellt.

Wird innerhalb eines Quellcodebausteins ein weiterer Import definiert, 
so wird dieser rekursiv ebenfalls importiert. Wird dabei ein bereits behandelter
Import erreicht, so wird dies erkannt und der Import wird nicht erneut eingefügt.

Die importierten Klassen werden an zwei Stellen gesucht
* Im Verzeichnis `scripts/commons`. Dort werden typischerweise projektinterne Klassen abgelegt, 
  diese Klassen müssen das package commons haben. 
  D.h. die Zeile  `import commons.example` führt dazu, 
  dass der Inhalt der Datei `scripts/commons/example.groovy` verwendet wird.
* in beliebigen Unterordnern des Verzeichnisses `commons`. 
  In dem Verzeichnis `commons` werden typischerweise Projekt-externe Verzeichnisse (z.B. als git 
  Submodule) gemounted. Alle Unterordner werden als valide Quellverzeichnisse 
  für projektübergreifende Repositorys angesehen, die wieder dieselbe Ordnerstruktur 
  wie das eigentliche Projekt haben. Daher können beliebig viele Repositorys eingebunden werden.
  
Projektübergreifende Dateien müssen das Package `commons.[package].[subpackage].[Scriptdateiname]`
  haben. D.h. die Zeile `import commons.[package].[subpackage].[Scriptdateiname]` führt dazu, dass
  die Datei `commons/[package]/scripts/commons/[package]/[subpackage]/[scriptdateiname].groovy`
  angezogen wird. 

Das zusammengefügte Skript wird im Verzeichnis 
`build/scripts/${prozessModellId}/[ID des Script-Tasks].groovy` abgelegt.
Der Inhalt dieser Datei kann dann im Task _buildModel_ (s.u.) anschließend als Script für den 
Script-Task eingetragen werden.

Die Verwendung von `import`-Anweisungen erlaubt es der verwendeten IDE,
die entsprechenden Klassen für ihre Codevervollständigung nutzen. 
So kann der Compiler schon während der Entwicklung auf Fehler hinweisen. 

Bei der Ablage der Quellcodedateien wird zwischen projektinternem und projektübergreifendem Code
unterschieden.

### Task _buildModel_

Dieser Task erzeugt aus den Prozessmodelldateien im Ordner `models` und den Skripten im Ordner `build/scripts/`
die vollständigen Prozessmodelle.

Dabei wird in den Skript-Task mit ID `${skriptTaskId}` des Prozessmodells mit ID `${prozessModellId}` der Inhalt
der Datei `build/scripts/${prozessModellId}/${skriptTaskId}.groovy` als Skript eingefügt. 
Kann die Datei nicht gelesen werden, wird der Task mit einem Fehler abgebrochen. 

Die vollständigen Prozessmodelle werden im Ordner `build/models` als `*.bpmn20.xml`-Dateien abgelegt.

### Task _uploadProcessModelFiles_

Dieser Task dient dazu, die Prozessmodelldateien für ein Prozessmodell ins Admincenter hochzuladen. 
Dabei wird das Prozessmodell nicht sofort deployed; das Deployment erfolgt über einen separaten Task.

Beim erstmaligen Hochladen wird für den in der Konfiguration bzw. im Parameter `environment` angegebenen Mandanten im
 Admincenter ein Prozessmodell mit dem in der Datei `project.json` unter `projectName`definierten Namen angelegt.
In diesem wird die unter `projectVersion` definierte Version angelegt. 
Anschließend werden alle Dateien im Ordner `build/models` für alle Stufen bis zur in `$
{nameDerUmgebung}.json` bzw. `default.json` unter `project` definierten Stufe importiert.

Falls für das angegebene Prozessmodell und die angegebene Version
* bereits Dateien in der angegebenen Stufe vorhanden sind, so gilt
  * hat eine Datei in der Stufe den gleichen Namen wie eine Datei in `build/models`, wird diese
   ersetzt
  * alle anderen vorhandenen Dateien werde nicht verändert
* bereits Dateien in einer höheren als der angegebenen Stufe vorhanden sind, ist ein Hochladen in
 die angegebene Stufe nicht möglich.

### Task _uploadParameterDefinition_

Dieser Task dient dazu, eine Prozessparameterdefinition für einen Prozess in das Admincenter 
hochzuladen. Eine eventuell bereits vorhandene Parameterdefinition wird überschrieben. 
Die Prozessparameterdefinition wird in diesem Task NICHT sofort deployed; das Deployment
findet beim Deployment des Prozesses statt. 

Wird dem Task über den Parameter `environment` eine Umgebung mitgegeben, so werden die 
Prozessparameterdefinitionen aus der Datei `parameterdefinitions/${nameDerUmgebung}.json` verwendet. 
Existiert diese Datei nicht oder wird keine Umgebung angegeben, so werden die 
Prozessparameterdefinitionen aus der Datei `parameterdefinitions/default.json` verwendet.

Wenn das angegebene Prozessmodell noch nicht existiert, wird ein Fehler geworfen.

### Task _deployProcessModelVersion_

Dieser Task dient dazu, eine im Admincenter hochgeladene Prozessmodellversion in einer Stufe zu deployen.

Für den Task kann über den Parameter `environment` angegeben werden, für welche Umgebung das Deployment
ausgeführt werden soll. Wird dieser nicht angegeben, so wird die in der Datei `config/default.json`
angegebene Umgebung verwendet.

Ist die Version bereits auf der Umgebung deployed, so wird sie zunächst undeployed, wobei alle 
laufenden Instanzen gelöscht werden.

Ist die Prozessmodellversion in der angegebenen Stufe nicht auf der Umgebung vorhanden, so wird ein 
Fehler geworfen.

Beim Deployen werden neben der Prozessmodellversion auch die hochgeladene Prozessparameterdefinition
veröffentlicht.

### Task _uploadFormularFiles_

Dieser Task dient dazu, die Fomulardateien für ein Prozessmodell ins Admincenter hochzuladen. 
Dabei wird das Prozessmodell nicht sofort deployed; das Deployment erfolgt über einen separaten Task.

Beim erstmaligen Hochladen wird für den in der Konfiguration bzw. im Parameter `environment` angegebenen Mandanten im
 Admincenter ein Formularmodell mit dem Namen des Formulares angelegt. Eventuell gleichbenannte Formulare aus anderen Projekten werden dabei aktualisiert.

In diesem wird die unter `projectVersion` definierte Version angelegt, wobei die Stufen _Fachliche Modellierung_ und _Technische Implementierung_ zu _Modellierung_ gewandelt werden.

Anschließend werden alle Dateien im Ordner `forms` für alle Stufen bis zur in `$
{nameDerUmgebung}.json` bzw. `default.json` unter `project` definierten Stufe importiert.

Falls für das angegebene Formulare und die angegebene Version
* bereits Dateien in der angegebenen Stufe vorhanden sind, so gilt
  * hat eine Datei in der Stufe den gleichen Namen wie eine Datei in `build/models`, wird diese
   ersetzt
  * alle anderen vorhandenen Dateien werde nicht verändert
* bereits Dateien in einer höheren als der angegebenen Stufe vorhanden sind, ist ein Hochladen in
 die angegebene Stufe nicht möglich.

### Task _uploadAndDeployFormularFiles_

Wie _uploadFormularFiles_, nur wird das Formular direkt nach dem Hochladen deployed.

### Task _getAuthorizationToken_

Um den manuellen Prozess zu vereinfachen, kann man mit diesem Task automatisiert für eine mittels 
`-Penvironment` angegebene Umgebung ein neues Bearer-Token in die Konfigurationsdatei `auth.json` 
eintragen lassen.

Benutzername und Passwort werden dabei in einer der von Gradle verendeten `gradle.properties` 
(https://docs.gradle.org/current/userguide/build_environment.html) als Variablen mit den Namen `serviceportal.username`
bzw. `serviceportal.password` gesucht. 

Durch Angabe eines Umgebungspostfix (`serviceportal.password.umgebung`) können für verschiedene 
Umgebungen unterschiedliche Zugangsdaten gesetzt werden. 

Es empfiehlt sich dabei eine Ablagedatei außerhalb des Projektordners bzw. - wenn innerhalb - diese in
der `.gitignore` von der Versionierung aus zu schließen, um die Zugangsdaten zu schützen.