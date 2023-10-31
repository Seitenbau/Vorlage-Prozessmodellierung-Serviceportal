# Inhaltsverzeichnis

1. [Ziele des Gradle-Plugins zur Prozessmodellierung](#ziele-des-gradle-plugins-zur-prozessmodellierung)
2. [System-Anforderungen](#system-anforderungen)
3. [Zugang](#zugang)
4. [Grundidee](#grundidee)
5. [Aktueller Funktionsumfang](#aktueller-funktionsumfang)
6. [Konfiguration und Projektstruktur](#konfiguration-und-projektstruktur)
   1. [Ordnerstruktur](#ordnerstruktur)
      1. [/config](#ordner-config)
      2. [/scripts](#ordner-scripts)
      3. [/commons](#ordner-commons)
      4. [/models](#ordner-models)
      5. [/forms](#ordner-forms)
      6. [/parameterdefinitions](#ordner-parameterdefinitions)
      7. [/metadata](#ordner-metadata)
   2. [Ermittlung von Konfigurationswerten](#ermittlung-von-konfigurationswerten)
        1. [Umgebung](#umgebung)
        2. [Mandant](#mandant)
        3. [Name und Version des Prozessmodells](#name-und-version-des-prozessmodells)
        4. [Status der Prozessmodell-Version](#status-der-prozessmodell-version)
        5. [Prozess-Engine für Deployment](#prozess-engine-für-deployment)
   3. [Unterstützung von Gradle Multi-Project Builds](#unterstützung-von-gradle-multi-project-builds)
7. [Tasks](#tasks)
    1. [_mergeScripts_](#task-mergescripts)
    2. [_buildModel_](#task-buildmodel)
    3. [_uploadProcessModelFiles_](#task-uploadprocessmodelfiles)
    4. [_uploadParameterDefinition_](#task-uploadparameterdefinition)
    5. [_getActiveProcessEngines_](#task-getactiveprocessengines)
    6. [_deployProcessModelVersion_](#task-deployprocessmodelversion)
    7. [_uploadFormularFiles_](#task-uploadformularfiles)
    8. [_uploadAndDeployFormularFiles_](#task-uploadanddeployformularfiles)
    9. [_getAuthorizationToken_](#task-getauthorizationtoken)
    10. [_startLocalHttpServer_](#task-startlocalhttpserver)
    11. [_stopLocalHttpServer_](#task-stoplocalhttpserver)

# Ziele des Gradle-Plugins zur Prozessmodellierung

Dieses Gradle-Plugin schafft die Möglichkeit, einzelne Prozess-Teile 
(insbesondere die im Prozess verwendeten Skripte) getrennt voneinander zu entwickeln
und so die Komplexität der Prozess-Entwicklung beherrschbar zu machen. 
Das Gradle-Plugin fügt die einzelnen Teile zu einem Modell zum Betrieb auf der Plattform zusammen
und spielt dieses Gesamtmodell zum Testen auf eine Zielplattform ein.

Dieser Ansatz ist für Entwickler gedacht, die sich eine lokale Arbeitsumgebung einrichten möchten
und nicht die Admincenter-Weboberflächen benutzen wollen. 
Die Weboberflächen sollen weiterhin als gleichwertige Alternative für weniger technisch orientierte
Entwickler erhalten bleiben und weiterentwickelt werden. 

Der Ansatz der lokalen Entwicklung ist aufgeteilt in:

* Entwicklung von Prozessmodellen ohne Skripte (z.B. mit Activiti Designer als Eclipse-Plugin)
* Entwicklung von Skripten als "Software" mit entsprechenden Möglichkeiten und Hilfen zur Entwicklung,
  z.B. Aufteilung in Klassen, Erstellen von Unittests usw.
  (via beliebiger IDE)
* Entwicklung von Formularen (wird noch entwickelt. Aktuell möglich ist z.B. textbasierte Entwicklung 
  oder Verwendung des Formulardesigners im Admincenter)
* Zusammenfügen, hochladen und deployen der Prozessmodelle und Formulare (dieses Gradle-Plugin)

# System-Anforderungen

* Das Plugin ist mit Gradle 7.6.2 getestet. Eventuell sind die Funktionen auch mit niedrigeren 
Gradle-Versionen verfügbar.
* Das verwendete Gradle muss minimal unter Java 11 laufen.

# Zugang

Das Plugin ist im Gradle Plugin-Repository unter der id 
https://plugins.gradle.org/plugin/de.seitenbau.serviceportal.prozesspipeline zu finden.

# Grundidee

- Skripte werden nur als Rumpf-Skripte implementiert, die eigentliche Funktionalität
  wird in Groovy-Klassen implementiert, die wie andere Software auch mit Unittests geprüft werden 
  können. Die Klassen werden mit den Rumpf-Skripten über import-Anweisungen verbunden.
- Die entwickelten Skripte werden dann in BPMN-Rumpfdateien in den entsprechenden Skript-Task
  eingefügt. Die Verknüpfung zwischen Skript-Task und Skript erfolgt über die ID des Skript-Tasks
  bzw. dem Dateiname des Skripts.
- Die so entwickelten BPMN-Dateien sowie ebenso im Projekt abgelegte 
  Prozessparameterdefinitionen und Formulare können per Schnittstelle ins Admincenter 
  übertragen und deployt werden.
  
Das Gradle-Plugin unterstützt dieses Vorgehen und stellt Tasks bereit, die das Zusammenfügen der
Skripte, das Einfügen der Skripte in das Prozessmodell und das Hochladen ins Admincenter und
Deployen übernehmen.  

# Aktueller Funktionsumfang

In diesem Release enthalten sind die Tasks:

| Task                                                                 | Beschreibung                                                                                                                                                    |
|----------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [`mergeScripts`](#task-mergescripts)                                 | Fügt die einzelnen Bestandteile der Skripte zu kompilierbaren Groovy-Skripten zusammen                                                                          |
| [`buildModel`](#task-buildmodel)                                     | Fügt die Skripte in das Prozessmodell ein                                                                                                                       | 
| [`uploadProcessModelFiles`](#task-uploadprocessmodelfiles)           | Lädt Prozessmodell-Dateien in das Admincenter hoch                                                                                                              |
| [`uploadParameterDefinition`](#task-uploadparameterdefinition)       | Lädt Prozessparameterdefinitionen in das Admincenter hoch                                                                                                       |
| [`getActiveProcessEngines`](#task-getactiveprocessengines)           | Gibt die Liste der zur Verfügung stehenden Prozess-Engines aus                                                                                                  |
| [`deployProcessModelVersion`](#task-deployprocessmodelversion)       | Deployt eine (zuvor hochgeladene) Prozessversion                                                                                                                |
| [`uploadFormularFiles`](#task-uploadformularfiles)                   | Lädt Formulardateien in das Admincenter hoch                                                                                                                    |
| [`uploadAndDeployFormularFiles`](#task-uploadanddeployformularfiles) | Lädt Formulardateien in das Admincenter hoch und deployt sie                                                                                                    |
| [`getAuthorizationToken`](#task-getauthorizationtoken)               | Erstellt ein Token zum Authentifizieren. Ein solches Token wird zum Hochladen und Deployen von Prozessen, Prozessparameterdefinitionen und Formularen benötigt. |
| [`startLocalHttpServer`](#task-startlocalhttpserver)                 | Startet einen lokalen Server, um lokale Formulardateien mit dem Formulardesigner auf einer definierten Umgebung zu bearbeiten                                   |
| [`stopLocalHttpServer`](#task-stoplocalhttpserver)                   | Stoppt den lokalen Server zum Bearbeiten von lokalen Dateien.                                                                                                   |

# Konfiguration und Projektstruktur

Das Gradle-Plugin wird über mehrere Json-Dateien konfiguriert, die in einer vorgegebenen 
Ordnerstruktur abgelegt werden müssen. Dabei können mehrere Umgebungen definiert werden; 
so können z.B. die Testumgebung und die Prozesstestumgebung als eigene Umgebungen
in der Konfiguration gespeichert werden; beim Ausführen der Gradle-Tasks kann dann die gewünschte
Umgebung mitgegeben werden. Die Prozessmodelle, Skripte und Formulare werden ebenfalls 
in einer definierten Ordnerstruktur erwartet.

## Ordnerstruktur

Über die erwartete Ordnerstruktur wird im Folgenden ein Überblick gegeben, 
danach werden die einzelnen Unterordner und Dateien im detailliert beschrieben.
Hierbei sind nicht für alle Gradle-Tasks alle beschriebenen Dateien und Ordner notwendig.

Es wird empfohlen, die Ordnerstruktur nicht selbst von null aus aufzubauen,
sondern die Vorlage https://github.com/Seitenbau/Vorlage-Prozessmodellierung-Serviceportal
zu verwenden.

Die Ordnerstruktur wird vom Gradle-Plugin wie folgt erwartet:

* **config**: Enthält die Konfigurationsdateien
  * **project.json**: Enthält allgemeine Informationen zum Projekt, z.B. Prozessname und Mandant
  * **auth.json**: Enthält Tokens zum Authentifizieren gegen die Serviceportal-Schnittstellen.
  * **default.json**: Enthält die Definition der Standardumgebung (URL, Status, ...)
  * **${umgebungsname}.json**: Enthält die Definition der Umgebung ${umgebungsname} 
* **scripts**: Enthält die Skripte, die in die Prozessmodell-Dateien eingefügt werden sollen
  * **${prozessmodelId}**: Enthält die Skripte für den Prozess mit der Id ${prozessmodelId}
    * **${skriptname}.groovy**: Enthält den Inhalt des Skripts mit der Id ${skriptname} in der
      Prozessmodell-Datei ${prozessmodelName}.bpmn 
  * **commons**: Enthält Klassen, die in mehreren Skripten des Prozessmodells verwendet werden können
* **commons**: Enthält Unterordner mit externen Klassen, die in den Skripten der Prozessmodelle 
  verschiedener Projekte übergreifend verwendet werden können
* **models**: Enthält die Prozessmodell-Dateien (mit leeren Skript-Tasks)
  * **${dateiname}.bpmn**: Enthält eine Prozessmodell-Datei
* **resources**: Enthält Dateien, die zusätzlich in das Prozess-Deployment mit aufgenommen werden sollen
  z.B. Keystores. 
  Falls keine solchen Dateien benötigt werden, muss der Ordner nicht angelegt werden.
* **forms**: Enthält die Formulare, die zum Prozess gehören
  * **${dateiname}.json**: Enthält eine Formulardefinition
* **parameterdefinitions**: Enthält die Definitionen der Prozessparameter
  * **default.json**: Fallback, wird verwendet, wenn keine Parameterdefinition für eine bestimmte
    Umgebung vorhanden ist
  * **${umgebungsname}.json**: Enthält die Parameterdefinition für die Umgebung ${umgebungsname}
* **metadata**: Enthält die Metadaten-Datei
    * **metadata.json**: Metadaten
  
### Ordner config

Enthält die Konfigurationsdateien für das Gradle-Plugin. Die enthaltenen Dateien beeinflussen nur
das Verhalten des Gradle-Plugins.

<br/>                                 

#### **Datei project.json**

Die Datei project.json enthält Informationen darüber, an welcher Stelle der Prozess
im Admincenter abgelegt wird. Enthalten sind folgende Informationen:

| Attribut        | Beschreibung                                                                                        |
|-----------------|-----------------------------------------------------------------------------------------------------|
| projectName     | Der Name des Prozessmodells im Admincenter.                                                         |
| projectVersion  | Die Version des Prozessmodells, an der gerade gearbeitet wird.                                      |
| mandant         | Die Id des Mandanten, unter dem das Prozessmodell und die Formulare im Admincenter abgelegt werden. | 

Beispielsweise könnte die Datei project.json so aussehen:
```json
{
  "projectName" : "TestInternal",
  "projectVersion": "v1.0",
  "mandant": "1"
}
```
<br/>                        

#### **Datei auth.json**

Die Informationen in dieser Datei authentifizieren den Benutzer gegen die entsprechende Umgebung.
Die Authentifizierungsinformationen sind sensitive persönliche Informationen
und entsprechen vom Stellenwert her persönlichen Passwörtern. 
Das heisst, jeder Entwickler sollte eigene Authentifizierungstokens benutzen, 
daher sollte diese Datei NICHT unter Versionskontrolle stehen und NICHT an andere Personen
weitergegeben werden. 

Die Datei hat folgende Struktur:

* eine Map der Umgebungen mit key Umgebungs-Id
  * darin je eine Map der Mandanten mit Key Mandanten-Id
    * darin folgendes Attribut:
      * **authorizationHeader**: Parameter zur Authentifizierung (`Bearer-Token`)

Damit die Token funktionieren, gelten folgende Voraussetzungen:

* Der entsprechende Nutzer muss auf der Umgebung angelegt sein
* dem Nutzer müssen die entsprechenden Rechte über Rollen/ Benutzergruppen gewährt werden.
* Das Token muss auf der entsprechenden Umgebung ausgestellt worden sein.
* Ein Zugriff mittels Webservice-Benutzern ist nicht möglich.
* Die Benutzertoken haben eine Gültigkeitsdauer von 24h, danach muss ein neues Token erzeugt werden.

Beispiel für die Authentifizierung für den Mandant 1 in der Umgebung default:
```json
{
  "default": {
    "1": {
      "authorizationHeader": 
      [
        "Bearer XYZTokenContent"
      ]
    }
  }
}
```
Die Token können am bequemsten über den Task getAuthorizationToken geholt werden.

Manche Umgebungen schränken aus Sicherheitsgründen einzelne Funktionen ein.
So ist es beispielsweise möglich, auf die Produktivumgebungen Prozesse im Status `TEST` 
hochzuladen, aber nicht im Status `FINAL`. Ebenso ist es nicht möglich mittels des Plugins 
die Prozesse zu deployen. 

Diese Beschränkungen werden über den Betrieb in Absprache mit dem IM/SK für jede Umgebung
festgelegt.
                                                                
<br/>

#### **Datei default.json und andere Umgebungsdefinitionen**

Die Umgebungsdefinitionen dienen dazu, die Informationen für unterschiedliche Umgebungen abzulegen.
Wird beim Aufruf der Tasks keine Umgebung angegeben, so wird die Umgebung "default" gewählt.
Der Name der Umgebung ist in dem Dateinamen kodiert. So wird z.B. für die Umgebung "prozesstest"
die Umgebungsdefinitionsdatei prozesstest.json angezogen.

Die Dateien enthalten folgende Informationen:

| Attribut                  | Beschreibung                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
|---------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| url                       | Die URL des Servicegateways der Zielumgebung                                                                                                                                                                                                                                                                                                                                                                                                                             |
| status                    | Der Status in der das Prozessmodell und die Formulare auf der entsprechenden Umgebung abgelegt werden sollen. Gültige Werte sind: <table><tr><th>Status</th><th>Beschreibung</th></tr><tr><td>EDIT</td><td>In Bearbeitung.</td></tr><tr><td>TEST</td><td>Wird zertifiziert.</td></tr><tr><td>FINAL</td><td>Final veröfentlicht und nicht mehr bearbeitbar.</td></tr></table> Es kann jedoch sein, dass manche Werte auf manchen Umgebungen nicht zur Verfügung stehen.   | 
| mandant                   | Die Id des Mandanten für diese Umgebung. Wenn gesetzt, wird der Mandant aus der Datei project.json überschrieben.                                                                                                                                                                                                                                                                                                                                                        |
| processModelNameExtension | Ein optionales Suffix, der beim Bauen der Prozessmodelle an den Prozessnamen, der in der bpmn-Datei definiert ist, angehängt wird (getrennt durch ein Leerzeichen).                                                                                                                                                                                                                                                                                                      |
| processEngine             | Die ID der Prozess-Engine, auf welche die Prozessmodell-Version deployt werden soll. Ist der Parameter nicht gesetzt, wird die Standard-Prozess-Engine verwendet.                                                                                                                                                                                                                                                                                                        |


Beispiel:
```json
{
  "url": "https://sgwtest.service-bw.de",
  "status": "EDIT",
  "mandant": "42",
  "processModelNameExtension": "DEV",
  "processEngine": "secondEngine"
}
```

### Ordner scripts

Enthält die Groovy-Skripte, die in die Prozessmodelle eingefügt werden,
und darin verwendete Klassen. 

Die Skripte für ein Prozessmodell liegen in einem Unterverzeichnis, dessen Name gleich der Id
des Prozessmodells (Attribut `id` des Elements `process` der BPMN-Datei) und haben den Namen
${Id des Script-Tasks}.groovy.

Sollen skript-tasksübergreifende Groovy-Klassen verwendet werden, können im Unterverzeichnis "commons" 
abgelegt werden.

Sie müssen im Package "commons" deklariert werden und können über Importanweisungen eingebungen 
werden. So wird z.B. die Klasse "commons.Example" in der Datei scripts/commons/Example.groovy 
gesucht.

Für projektübergreifende Klassen ist das Verzeichnis commons direkt im Projekt gedacht, s.u.

### Ordner commons

Enthält beliebige Unterordner mit weiteren Groovy-Klassen, 
die in den Skripten verwendet werden können.
Jeder Unterordner muss dabei den Namen des zweiten Teils des packages haben und
hat dieselbe Projektstruktur wie die hier beschriebene Projektstruktur.
D.h. die Klasse "commons.externalpackage.Example" würde in der Datei
commons/externalpackage/scripts/commons/externalpackage/Example.groovy gefunden werden.

Dieses Verzeichnis ist dafür gedacht, um externe Verzeichnisse, die z.B.
eine Bibliothek von prozessübergreifenden Klassen bereitstellen, 
in das Projekt verlinken zu können (beispielsweise mit git submodules).

### Ordner models

Enthält die Prozessmodell-Dateien des Prozesses mit leeren Skript-Tasks.
Die Prozessmodell-Dateien enden mit .bpmn oder mit .bpmn20.xml.

### Ordner forms

Enthält die Formulare, die zum Prozess gehören. Es gibt zwei Namensschemata für die Formulare:
* `forms/${FormularName}-${FormularVersion}-${Sprachkürzel}.json` 
* `forms/${FormularName}/${FormularVersion}/${Sprachkürzel}.json`

D.h. entweder können FormularName und Formularversion im Dateinamen konfiguriert werden
oder in den Namen der Verzeichnisse.

### Ordner parameterdefinitions

Enthält die Definition der Prozessparameter für den Prozess. 
Die Standard-Konfiguration der Prozessparameter sollte in der Datei default.json abgelegt werden.
Werden für einzelne Umgebungen andere Prozessparameterdefinitionen benötigt, so können diese
in Dateien ${umgebungsname}.json abgelegt werden, die dann für die benannten Umgebungen 
automatisch statt default.json angezogen werden.

**Diese Dateien haben dasselbe Format wie die json-Dateien, 
die auch im Admincenter als Prozessparameterdefinitionen verwendet werden.**

In einer Parameterdefinition werden die folgenden Informationen gespeichert

| Attribut     | Beschreibung                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
|--------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| name         | Der programmatische Name des Prozessparameters, muss unique für ein Prozessmodell sein                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| description  | Die Beschreibung des Parameters für die Prozessverwalter, die den Parameter setzen wollen.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| type         | Der Typ des Prozessparameters. Erlaubt sind folgende Werte:  <table><tr><th>Type</th><th>Beschreibung</th></tr><tr><td>STRING</td><td>Der im ProcessParameter gespeicherte String wird as is verwendet</td></tr><tr><td>JSON_STRING_MAP</td><td>Der im ProcessParameter gespeicherte String wird im Prozess als <code>JSON-Map&lt;String, String&gt;</code> geparst</td></tr><tr><td>JSON_OBJECT</td><td>Der im ProcessParameter gespeicherte String wird im Prozess als JSON-Objekt geparst</td></tr><tr><td>BINARY</td><td>Der im ProcessParameter gespeicherte String wird als Base64 kodierter Byte-Array interpretiert und als Byte-Array zurückgegeben.</td></tr></table> |
| defaultValue | Der Defaultwert des Parameters. Wird verwendet, falls der Prozessverwalter bei der Aktivierung des Prozesses nichts anderes angibt. Wenn defaultValue nicht gesetzt ist, gibt es keinen Defaultwert.                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| required     | true, falls der Aktivierung des Prozesses immer ein Wert angegeben werden muss, oder false, wenn nicht. Default ist false.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| hidden       | true, wenn der Prozessparameter bei der Aktivierung nicht sichtbar sein soll, oder false, wenn der Prozessparameter bei der Aktivierung sichtbar sein soll. Default ist false.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |


Beispielhafter Inhalt einer Prozessparameterdefinitions-Datei:
```json
[
  {
    "name": "nameDesProzessparameters",
    "description": "Beschreibung für die Prozessverwalter, die den Parameter setzen wollen.",
    "type": "STRING",
    "defaultValue": "myDefaultValue",
    "required": false,
    "hidden": false
  },
  {
    "name": "otherProzessparameter",
    "description": "Andere Beschreibung",
    "type": "JSON_OBJECT"
  }
]
```
### Ordner metadata

Enthält eine optionale Definition der Prozess-Metadaten.

| Attribut            | Beschreibung                                                                                                                                                                                                                                               |
|---------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| authenticationTypes | Eine Liste der für diesen Prozess zulässigen Authentisierungsmittel. <br/>Erlaubte Werte sind <ul><li>SERVICEKONTO</li><li>BUND_ID</li><li>MUK</li></ul> **Hinweis:** Für die Produktionssysteme Service-BW und AMT-24 ist derzeit nur SERVICEKONTO aktiv. |

```json
{
  "authenticationTypes": [
    "SERVICEKONTO",
    "BUND_ID",
    "MUK"
  ]
}
```


## Ermittlung von Konfigurationswerten

### Umgebung

Die Umgebung wird, sofern sie vom Task benutzt wird, über den Parameter **environment**
des Tasks gesetzt (mittels `-Penvironment=<umgebung>`). Wird dieser Parameter nicht gesetzt, so wird
die Umgebung `default` verwendet.

### Mandant

Der Mandant wird mit folgender Hierarchie ermittelt. 
Dabei wird von oben nach unten vorgegangen und der erste nicht-null Wert verwendet.
* Parameter **mandant** bei Aufruf des Tasks (mittels `-Pmandant=<mandant>`)
* der in der Konfigurationsdatei der Umgebung definierte Mandant 
* der in der Datei `config/project.json` definierte Mandant 

### Name und Version des Prozessmodells

Name und Version des Prozessmodells werden aus der Datei `config/project.json` gelesen.

### Status der Prozessmodell-Version

Der aktive Status des Prozessmodells wird aus der Konfigurationsdatei der Umgebung gelesen.

### Prozess-Engine für Deployment
Die ID der Prozess-Engine, auf die eine Prozessmodell-Version deployt werden soll, wird, sofern
vorhanden, aus der Konfigurationsdatei der Umgebung gelesen.

## Unterstützung von Gradle Multi-Project Builds

[Gradle Multi-Project Builds](https://docs.gradle.org/current/userguide/multi_project_builds.html)
werden vom Gradle-Plugin insofern unterstützt, als dass die Konfiguration des 
aktuell gebauten Projekts mit der Konfiguration des Root-Projekts ergänzt wird.
D.h. Vorrang hat die Konfiguration des aktuell gebauten Projekts, aber Einstellungen,
die im aktuell gebauten Projekt nicht vorhanden sind, werden aus der Konfiguration
des Root-Projektes geladen, wenn sie dort vorhanden sind.

Dies erlaubt es, die Konfiguration in den einzelnen Projekten minimal zu halten, indem 
wiederkehrende Konfigurationseinstellungen in ein Root-Projekt ausgelagert werden.

Authentifizierungsdaten, die vom Task _getAuthorizationToken_ geschrieben werden, werden
in die Datei config/auth.json des Root-Projektes geschrieben. Authentifizierungsdaten im Unterprojekt
werden dabei vollständig ignoriert.


# Tasks

## Parameter für alle Tasks

Die meisten vom Plugin bereitgestellten Gradle-Tasks können mit den folgenden Parametern
konfiguriert werden:
       
| Parameter   | Pflicht? | Beschreibung                                                                                                                                   |
|-------------|----------|------------------------------------------------------------------------------------------------------------------------------------------------|
| mandant     | Nein     | Überschreibt den im config-Verzeichnis festgelegten Mandanten.                                                                                 |
| environment | Nein     | Definiert die Umgebung, deren Konfiguration verwendet wird. Wird dieser Parameter nicht verwendet, so wird die Umgebung "default" verwendet.   |
| debug       | Nein     | Wenn der Wert des Parameters "true" ist, werden weitere debug-Ausgaben durch das Plugin ausgegeben.                                            |

## Task _mergeScripts_

Zweck dieses Tasks ist es, den Quellcode der Groovy-Skript-Tasks aus mehreren Dateien
zusammenzufügen.
Die Klassennamen in den import-Statements des Basis-Skripts definieren die einzubindenden Dateien, 
in denen typischerweise die importierten Klassen definiert sind. 
Diese eingebundenen Dateien enthalten ggf. weitere import-Statements, 
die wiederum aufgelöst werden usw. (d.h. die Auflösung erfolg rekursiv). 
Wird ein bereits behandelter Import erreicht, so wird dies erkannt 
und der Import wird nicht erneut eingefügt. 
Imports, die nicht mit dem root package `commons` beginnen, führen nicht zu einem Einbinden
einer Datei.

Die importierten Klassen werden an zwei Stellen gesucht:
* Im Verzeichnis `scripts/commons`. Dort werden typischerweise projektinterne Klassen abgelegt. 
  Diese Klassen müssen das package commons haben. 
  D.h. die Zeile  `import commons.Example` führt dazu, 
  dass der Inhalt der Datei `scripts/commons/Example.groovy` verwendet wird.
* in beliebigen Unterordnern des Verzeichnisses `commons`. 
  In dem Verzeichnis `commons` werden typischerweise projektexterne Verzeichnisse (z.B. als git 
  Submodule) gemounted. Alle Unterordner werden als valide Quellverzeichnisse 
  für projektübergreifende Repositorys angesehen, die wieder dieselbe Ordnerstruktur 
  wie das eigentliche Projekt haben. Daher können beliebig viele Repositorys eingebunden werden.\
  Projektübergreifende Dateien müssen das Package `commons.[package].[subpackage].[Scriptdateiname]`
  haben, wobei subpackage optional ist oder selbst aus mehreren packagebestandteilen 
  zusammengesetzt sein kann. 
  D.h. die Zeile `import commons.[package].[subpackage].[Scriptdateiname]` führt dazu, dass
  die Datei `commons/[package]/scripts/commons/[package]/[subpackage]/[scriptdateiname].groovy`
  angezogen wird. 

Das zusammengefügte Skript wird im Verzeichnis 
`build/scripts/[ID des Prozesmodells]/[ID des Script-Tasks].groovy` abgelegt. 
Der Inhalt dieser Datei kann dann im Task _buildModel_ (s.u.) anschließend als Skript für den 
Skript-Task eingetragen werden.

Die Verwendung von `import`-Anweisungen erlaubt es der verwendeten IDE,
die entsprechenden Klassen für Codevervollständigung zu nutzen, zu kompilieren 
und schon während der Entwicklung auf Fehler hinzuweisen. Hierfür sollten in der IDE
die Verzeichnisse `scripts` und alle Unterverzeichnisse des Verzeichnisses `commons` 
als Groovy-Quellverzeichnisse konfiguriert werden.

### Parameter

Der globale Aufrufparameter **debug** beeinflusst den Task wie angegeben.
 
## Task _buildModel_

Dieser Task fügt die (mit dem Task _mergeScripts_ zusammengefügten) Skripte in die Prozessmodelle
ein, die im Verzeichnis `models` liegen, 
und erweitert ggf. den Namen des Prozesses mit der für die angezogene Umgebung
konfigurierten processModelNameExtension.

Die Skripte werden nach folgendem Schema 
`build/scripts/[ID des Prozesmodells]/[ID des Skript-Tasks].groovy`
den Skript-Tasks zugeordnet.  

Die gebauten Prozessmodelle werden in das Verzeichnis `build/models` geschrieben.  

### Aufrufparameter

Die globalen Aufrufparameter **environment** und **debug** beeinflussen den Task wie angegeben.

## Task _uploadProcessModelFiles_

Dieser Task lädt die Prozessmodell-Dateien, die mit dem Task _buildModel_ erstellt wurden,
zusammen mit den Metadaten aus dem Verzeichnis `metadata` in das Admincenter hoch. 
Dabei wird das Prozessmodell nicht sofort deployt; das Deployment erfolgt über den separaten 
Task _deployProcessModelVersion_.

Mandant, Name, Version und aktiver Status für das Hochladen werden wie im Kapitel 
"Ermittlung von Konfigurationswerten" beschrieben aus der Konfiguration gelesen.

Beim erstmaligen Hochladen wird für den konfigurierten Mandanten im Admincenter 
ein Prozessmodell mit dem konfigurierten Namen angelegt. 
In diesem wird die konfigurierte Version angelegt. 
Anschließend werden alle Dateien im Ordner `build/models` für den konfigurierten Status importiert.

Falls für das angegebene Prozessmodell und die angegebene Version
* bereits Dateien mit dem angegebenen Status vorhanden sind, so gilt:
  * hat eine Datei mit einem Status den gleichen Namen wie eine Datei in `build/models`, wird diese
   ersetzt
  * alle anderen vorhandenen Dateien werden NICHT verändert oder gelöscht
* bereits Dateien in einem höheren als dem angegebenen Status vorhanden sind, ist ein Hochladen mit
  dem angegebenen Status nicht möglich.
 
### Aufrufparameter

Alle globalen Aufrufparameter beeinflussen den Task wie angegeben.

## Task _uploadParameterDefinition_

Dieser Task lädt eine Prozessparameterdefinition für einen Prozess in das Admincenter hoch.
Eine eventuell bereits vorhandene Parameterdefinition wird überschrieben. 
Die Prozessparameterdefinition wird in diesem Task NICHT sofort deployt; das Deployment
findet beim Deployment des Prozesses statt. 

Mandant, Name und Version des Prozesses, für den die Parameterdefinition hochgeladen werden soll,
werden wie im Kapitel "Ermittlung von Konfigurationswerten" beschrieben
aus der Konfiguration gelesen.

Der Name der konfigurierten Umgebung bestimmt den Dateinamen der hochgeladenen Parameterdefinition.
Standardmäßig wird die Datei `parameterdefinitions/${nameDerUmgebung}.json` verwendet. 
Existiert diese Datei nicht oder wird keine Umgebung angegeben, so werden die 
Prozessparameterdefinitionen aus der Datei `parameterdefinitions/default.json` verwendet.

Wenn das angegebene Prozessmodell oder die angegebene Version im konfigurierten Mandanten 
im Admincenter nicht existiert, wird ein Fehler geworfen.

### Aufrufparameter

Alle globalen Aufrufparameter beeinflussen den Task wie angegeben.

## Task _getActiveProcessEngines_

Dieser Task gibt die Liste der aktuell zur Verfügung stehenden Prozess-Engines aus. Die ID einer 
Prozess-Engine kann beim Deployment einer Prozessmodell-Version verwendet werden, um gezielt
auf die gewünschte Prozess-Engine zu deployen.

## Task _deployProcessModelVersion_

Dieser Task deployt ein im Admincenter vorhandenes Prozessmodell. Ist das Prozessmodell schon 
deployt, wird es undeployt und neu deployt. 

Mandant, Name, Version und Status des zu deployenden Prozesses und die ID der Prozess-Engine, auf die
deployt werden soll, werden wie im Kapitel "Ermittlung von Konfigurationswerten" beschrieben aus 
der Konfiguration gelesen.

Die Aktion entspricht dabei dem Button "Prozessmodell deployen" im Admincenter.
Ein Triggern der Aktion "Deploy (Prozess-Testumgebung)" über diesen Task ist nicht möglich.

Aus der Menge, der zur Verfügung stehenden Prozess-Engines, kann die Engine, auf die die
Prozessversion deployt werden soll, ausgewählt werden. Ist keine Engine explizit angegeben,
wird die Standard-Prozess-Engine verwendet.

## Task _uploadFormularFiles_

Dieser Task lädt die Formulare des Projekts in das Admincenter hoch.

Die hochzuladenden Formulare werden im Ordner `forms` gesucht. Alle dort liegenden 
und der Namenskonvention entsprechenden Formulare werden über die Schnittstelle
an das Admincenter gesendet. Schon existierende Formulare werden überschrieben, 
nicht existierende Formulare und Versionen werden angelegt.
Falls der aktive Status für die Version des Formulars im Admincenter
höher ist als der angegebene Status in der Konfiguration, 
so ist ein Hochladen in den angegebene Status nicht möglich. 
Formulare, welche nicht dem Namensschema entsprechen, erzeugen einen Fehler.

### Benennung

Die Informationen zu den Formularen können entweder als Ordnerstruktur oder über den Dateinamen*
 dem Task übergeben werden:
* `forms/${FormularName}-${FormularVersion}-${Sprachkürzel}.json` 
* `forms/${FormularName}/${FormularVersion}/${Sprachkürzel}.json`

\* analog dem Namen beim Download eines Formulars aus dem Admincenter

Mandant und der Status für das Hochladen werden wie im Kapitel 
"Ermittlung von Konfigurationswerten" beschrieben aus der Konfiguration gelesen.

### Aufrufparameter

Alle globalen Aufrufparameter beeinflussen den Task wie angegeben.
   
## Task _uploadAndDeployFormularFiles_

Dieser Task dient dazu, die Formulare im Projekt an eine Umgebung zu senden 
und anschliessend zu deployen.
Ist ein Formular schon deployt, so wird es vor dem erneuten Deployment undeployt. 

Die weitere Funktion ist analog zum Task _uploadFormularFiles_.

## Task _getAuthorizationToken_

Mit diesem Task kann ein neues Bearer-Token für eine Umgebung 
in die Konfigurationsdatei `auth.json` geschrieben werden.
Existiert das entsprechende Token schon, so wird mit dem neu geholten Token ersetzt.

Benutzername und Passwort werden dabei aus einer der `gradle.properties` 
(in der systemeigenen Hierarchie) aus den Keys `serviceportal.username` bzw. `serviceportal.password` 
gelesen.

Mandant und Umgebung, unter der der Key abgelegt werden soll, werden wie im Kapitel 
"Ermittlung von Konfigurationswerten" beschrieben aus der Konfiguration gelesen.

### Aufrufparameter

Alle globalen Aufrufparameter beeinflussen den Task wie angegeben.

## Task _startLocalHttpServer_

Startet einen lokalen Server, um auf einer beliebigen Serviceportal-Umgebung die auf dem lokalen 
Rechner des Entwickler liegenden Dateien des Entwicklungsprojektes (Formulare und Prozesse) 
editieren zu können.

In der Konsole wird ein Link zu einer Übersichtsseite angezeigt, auf welcher die zur
Bearbeitung verfügbaren Dateien verlinkt sind. Durch Klick auf den Link wird die entsprechende Seite 
der Umgebung im Browser geöffnet.

Mit dem Parameter `environment` kann (wie bei den anderen Tasks) die gewünschte Umgebung gewählt 
werden, auf welcher die Datei bearbeitet werden soll. Der Parameter `port` kann den Server auf
einem definierten lokalen Port starten (sofern dieser frei ist).

Es kann nur jeweils eine Instanz des Servers gestartet werden. Um den Server zu beenden, kann in der 
Übersichtsseite auf der entsprechende Link verwendet bzw. auf der Konsole der dazu bereitgestellte 
Task verwendet werden.

## Task _stopLocalHttpServer_

Stoppt den lokalen Server zum Bearbeiten von lokalen Dateien.
