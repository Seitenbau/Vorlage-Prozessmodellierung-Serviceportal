# Vorlage zum Erstellen und automatisierten hochladen eines Serviceportal-Prozessmodells

## Übersicht
Die Dokumentation besteht aus folgenden Teilen:

* Projektvorlage (dieses Repository)
* [Gradle-Plugin (aus Gradle-Plugin-Repository)](plugin.md)
* [Schnittstellendokumentation](schnittstelle.md) 

# Projektvorlage

Arbeitsablauf:

1. Ein neues leeres GIT-Repository auf dem Server für das Projekt erstellen. 
1. Das neue Projekt clonen und einen initialen Commit machen.
1. Dieses Repository (also die Vorlage, in welcher Sie gerade lesen) als ein zweites remote-Repository 
(vorgeschlagener Name: `template`) zu Ihrem Projekt hinzufügen: `git remote add template {link zu diesem repo}`
1. `git fetch --all` aufrufen
1. `git merge template/master --allow-unrelated-histories` alle Dateien aus der Vorlage in der Projekt mergen.
1. Den Projektnamen "Prozessmodellierungsvorlage" in den folgenden Dateien in den tatsächlichen Namen des Projektes ändern:
   - `settings.gradle`
   - `config/project.json`
1. in der `settings.gradle` die gewünschte Version des Gradle-Plugins setzen     
1. Die Submodule (sofern vorhanden und benutzt) initialisieren: `git submodule update --init --recursive`
1. `git push` die gemachten Änderungen
1. Anlegen der Formulare, Scripte, Prozessparameterdefinitionen und Scripte 

## Konfiguration und Projektstruktur

Das Gradle-Plugin wird über mehrere Json-Dateien konfiguriert, die in einer vorgegebenen 
Ordnerstruktur abgelegt werden müssen. Dabei können mehrere Dateien für verschiedene Umgebungen
definiert werden. 

Beim Ausführen der Gradle-Tasks kann dann die gewünschte Umgebung per Aufrufparameter mitgegeben werden.

Die Prozessmodelle, Skripte und Formulare werden in einer definierten Ordnerstruktur erwartet.

## Ordnerstruktur

Über die erwartete Ordnerstruktur wird im Folgenden ein Überblick gegeben, 
danach werden die einzelnen Unterordner und Dateien im detailliert beschrieben.
Hierbei sind nicht für alle Gradle-Tasks alle beschriebenen Dateien und Ordner notwendig.

* **config** Enthält die Konfigurationsdateien
  * **project.json** Enthält allgemeine Informationen zum Projekt, z.B. Prozessname und Mandant
  * **auth.json** Enthält Tokens zum Authentifizieren gegen die Serviceportal-Schnittstellen.
  * **default.json** Enthält die Definition der Standardumgebung (URL, Projektstufe, ...)
  * **${umgebungsname}.json** Enthält die Definition der Umgebung ${umgebungsname} 
* **scripts** Enthält die Skripte, die in die Prozessmodell-Dateien eingefügt werden sollen
  * **${prozessmodelId}** Enthält die Skripte für den Prozess mit der Id ${prozessmodelId}
    * **${scriptname}.groovy** Enthält den Inhalt des Skripts mit der Id ${scriptname} in der
      Prozessmodell-Datei ${prozessmodelName}.bpmn 
  * **commons** Enthält Klassen, die in allen Skripten des Prozessmodells verwendet werden können
* **commons** Enthält Unterordner mit externen Klassen, die in den Skripten dieses Prozessmodells 
  und anderer Projekte verwendet werden können
* **models** Enthält die Prozessmodell-Dateien (mit leeren Skript Tasks)
  * **${dateiname}.bpmn** bzw. **${dateiname}.bpmn20.xml** Enthält eine Prozessmodell-Datei
* **forms** Enthält die Formulare, die zum Prozess gehören
  * **${dateiname}.json** Enthält eine Formulardefinition
* **parameterdefinitions** Enthält die Definitionen der Prozessparameter
  * **default.json** Fallback, wird verwendet, wenn keine Parameterdefinition für eine bestimmte
    Umgebung vorhanden ist
  * **${umgebungsname}.json** Enthält die Parameterdefinition für die Umgebung ${umgebungsname}
  
### Ordner config

Enthält die Konfigurationsdateien für das Gradle-Plugin. Die enthaltenen Dateien beeinflussen nur
das Verhalten des Gradle-Plugins.

#### hierachische Projekte

Wenn mehrere Projekte in einem Repository zusammen gelegt werden, können diese über die Gradle-Standart-Methoden angesteuert werden (siehe https://docs.gradle.org/current/userguide/multi_project_builds.html). Dabei wird auch ie Projektkonfiguration entsprechend erst aus dem root-Projekt gelesen, und deren Werte mit denen des Unterprojektes ergänzt.

Dadurch kann z.B. die Konfiguration für alle Projekte im Root-Prjekt erfolgen, und die Konfigurationsdateien der Unterprojekte enthalten nur noch die abweichenden Größen wie z.B. den Namen des Projektes.

Dabei gilt: Manuell per Aufrufpfad gesetzte Parameter überschreiben die Werte aus den Konfigdateien (z.B. -Pmandant=42), und die Angaben im Unterprojekt überschreiben (wenn gesetzt) die im Root-Projekt.

Eine Ausnahme hierbei sind die Daten zur Identifizierung: der Task getAuthorizationToken schreibt die Daten ins Root Projekt und nimmt diese auch bevorzugt von hier.

#### Ordner config - Datei project.json

Die Datei project.json enthält Informationen darüber, an welcher Stelle der Prozess im Admincenter
abgelegt wird. Enthalten sind folgende Informationen:
* **projectName**: Der Name des Prozessmodells im Admincenter.
* **projectVersion**: Die Version des Prozessmodells, an der gerade gearbeitet wird.
* **mandant**: Die ID des Mandanten, unter dem das Prozessmodell und die Formulare im Admincenter 
  abgelegt werden. Die Angabe ist optional, allerdings muss dann in der Umgebungskonfiguration oder als Parameter ein Mandant gesetzt werden.

Beispielsweise könnte die Datei project.json so aussehen:
```json
{
  "projectName" : "TestInternal",
  "projectVersion": "v1.0",
  "mandant": "1",
}
```

#### Ordner config - Datei auth.json

Die Informationen in dieser Datei authentifizieren den Benutzer gegen die entsprechende Umgebung.

**Die Authentifizierungsinformationen sind sensible persönliche Informationen
und entsprechen vom Stellenwert her persönlichen Passwörtern.** 

Das bedeutet, 
* jeder Entwickler sollte eigene Authentifizierungstokens benutzen, 
* sollte diese Datei **nicht**  unter Versionskontrolle stehen 
* und **nicht** an andere Personen weitergegeben werden
* wenn möglich, sollte die Datei außerhalb der Projekte angelegt und in das Projekt hinein 
verlinkt werden. 

Sie enthalten folgende Struktur:

* eine Map der Umgebungen
  * darin je eine Map der Mandanten mit Key Mandanten-Id, darin
      * **authorizationHeader** Parameter zur Authentifizierung (`Bearer-Token`, `Basic auth`...)
      * **headerParams** optionale zusätzliche Header-Parameter. Diese werden direkt wie sie aufgelistet sind bei allen http-Aufrufen mit gesendet. 

Datei-Beispiel `auth.json`für die Authentifizierung für den Mandant 1 in der Umgebung default:
```json
{
  "default": {
    "1": {
      "authorizationHeader": 
      [
        "Bearer eyJhbGciOiJIUzI1NiJ9[...]"
      ]
    }
  }
}
```

Damit die Token funktionieren, gelten folgende Voraussetzungen:

* Der entsprechende Nutzer muss auf der Umgebung angelegt sein
* die Funktionen müssen auf der Umgebung freigeschaltet sein
* dem Nutzer müssen die entsprechenden Rechte über entsprechende Rollen/ Benutzergruppen gewährt werden.
* Die Token laufen nach einer umgebungsspezifischen Frist ab. Diese beträgt standardmäßig meist 24h.
* **ein Zugriff mittels Webservice-Benutzern ist nicht möglich.**

Manche Umgebungen schränken aus Sicherheitsgründen einzelne Funktionen ein.
z.B. wird es nicht möglich sein, auf die Produktivumgebungen mittels der Schnittstelle
zu deployen. 

Diese Beschränkungen werden über den Betrieb auf Anweisung des IM/SK festgelegt.

##### Erzeugung der Token

##### Automatische Erstellung via getAuthorizationToken

Um den manuellen Prozess zu vereinfachen, kann man mit dem Task  `getAuthorizationToken`
für eine Umgebung ein neues Bearer-Token in die Konfigurationsdatei `auth.json` eintragen lassen.

##### manuelle Erstellung über das SGW und eintragen in die Auth.json

Die Token können auch manuell über das Servicegateway der Umgebung erzeugt werden. 

Dafür kann die Oberfläche der Rest-Dokumentation aufgerufen werden: 
* `Zuständigkeitsfinder: Schnittstellen-Benutzer` 
* `/benutzer/token`.

Alternativ gibt folgender  `curl` -Aufruf das gewünschte Token zurück:

```shell script
curl -X POST \
  --header "Content-Type: text/plain" \
  --header "Accept: text/plain" \
  --header "X-SP-Mandant: ${mandanten-id}" \
  -d "${passwort des Nutzers}" \
  "https://${url des sgw}:443/benutzer/token?admincenterbenutzername=${benutzername}&scope=autoDeployment"
```

Im Body der Antwort bekommt man das benötigte Bearer-Token, z.B.:

```Bearer eyJhbGciOiJIUzI1NiJ9[...]```

#### Ordner config - Datei default.json und andere Umgebungsdefinitionen

Die Umgebungsdefinitionen dienen dazu, die Informationen für unterschiedliche Umgebungen abzulegen.
Wird keine Umgebung beim Aufruf der Tasks angegeben, so wird die Umgebung "default" gewählt.
Der Name der Umgebung ist in dem Dateinamen kodiert. So wird z.B. für die Umgebung "prozesstest"
die Umgebungsdefinitionsdatei prozesstest.json angezogen.

Die Dateien enthalten folgende Informationen:
* **url**: Die URL des Servicegateways der Zielumgebung
* **projectStage**: Die Stufe, in die das Prozessmodell und die Formulare 
  auf der entsprechenden Umgebung abgelegt werden sollen. Gültige Werte sind prinzipiell
  FUNCTIONAL_ANALYSIS, TECHNICAL_IMPLEMENTATION, QUALITY_ASSURANCE, CERTIFIED, es kann jedoch sein,
  dass manche Werte auf manchen Umgebungen nicht zur Verfügung stehen.
* **mandant**: Die ID des Mandanten für die Umgebung. Überschreibr - wenn gesetzt - den Mandant aus der Datei 
  project.json überschrieben. Angabe optional, es muss dann aber anderweitig ein Mandant gesetzt werden. 

Beispiel:
```
{
  "url": "https://sgwtest.service-bw.de",
  "projectStage": "TECHNICAL_IMPLEMENTATION",
  "mandant": "42"
}
```
Die in den Umgebungsdefinitionen im Ordner `config` gesetzten Attribute überschreiben die der 
Umgebungsdefinitionen im `globalEnvConfigurationPath` (s.o).

### Ordner models

Dieser Ordner enthält die Prozessmodell-Dateien als `*.bpmn`oder `*.bpmn20.xml`. 
Die Skript-Tasks der Prozessmodelle werden dabei komplett leer gelassen und aus dem Ordner `scripts`befüllt.

### Ordner scripts

Dieser Ordner enthält Groovy-Dateien, aus denen die Groovy-Skripte für die Skript-Tasks der 
Prozessmodelle erzeugt werden.

 
Der Ordner enthält einen Unterordner `commons`, der Groovy-Klassen enthält, die in mehreren 
Skripten verwendet werden. 

Des Weiteren enthält er für jede Prozessmodelldatei im Ordner `models` einen Unterordner
mit Namen `${processModelId}`. Dabei muss der Ordnername dem Feld `id` aus der zugehörigen Modelldatei entsprechen.
 
In diesen Ordnern liegt für jeden Skript-Task der Prozessmodelle eine Datei mit Namen
`${scriptTaskId}.groovy`. Anhand des Dateinamens wird diese Datei in den Skript-Task im 
Prozessmodell eingefügt.

### Ordner forms

In diesem Ordner werden die Formulare, die im Prozess verwendet werden, als JSON-Datei abgelegt.

Dabei werden die Formulare wie folgt abgelegt:
* **Variante 1** als Dateien im Stammverzeichnis
* **Variante 2** als Ordnerstruktur

#### Variante 1
Die Dateien werden als `{formularname}-{version}-{sprache}.json` (entspricht dem Schema beim 
Herunterladen von Formularen aus dem Admincenter) abgelegt. 

Das Plugin parst den Dateinamen und lädt die Formulare entsprechend hoch. 

In dieser Variante können die Formularnamen und Versionen alle Buchstaben, Umlaute und `_` 
sowie `.` enthalten. Als Sprachen können `de|en|fr` angegeben werden.

#### Variante 2
Die Dateien werden in einer Ordnerstruktur mit dem Aufbau `{formularname}/{version}/{sprache}.json` abgelegt. Dabei ist es möglich in den Formularnamen zusätzlich das Zeichen `-` zu verwenden.

Das Plugin parst den Pfad zu den Dateien und lädt die Formulare entsprechend hoch.

### Ordner parameterdefinitions

Enthält die Prozessparameter-Definition für einen Prozess. 

Die Standard-Konfiguration der Prozessparameter sollte in der Datei default.json abgelegt werden.
Werden für einzelne Umgebungen andere Prozessparameterdefinitionen benötigt, so können diese
in Dateien ${umgebungsname}.json abgelegt werden, die dann für die benannten Umgebungen 
automatisch statt default.json angezogen werden.

In einer Parameterdefinition werden die folgenden Informationen gespeichert:

* **name**: Der programmatische Name des Prozessparameters, muss unique für ein Prozessmodell sein
* **description**: Die Beschreibung des Parameters für die Prozessverwalter, die den Parameter 
  setzen wollen.
* **type**: Der Typ des Prozessparameters. Erlaubt sind folgende Werte:
  * **STRING**: Der im ProcessParameter gespeicherte String wird as is verwendet
  * **JSON_STRING_MAP**: Der im ProcessParameter gespeicherte String wird im Prozess als 
   JSON-Map<String, String> geparst
  * **JSON_OBJECT**: Der im ProcessParameter gespeicherte String wird im Prozess als
   JSON-Objekt geparst 
  * **BINARY**: Der im ProcessParameter gespeicherte String wird als Base64 kodierter Byte-Array
   interpretiert und als Byte-Array zurückgegeben.
* **defaultValue**: Der Defaultwert des Parameters. Wird verwendet, falls der Prozessverwalter 
  bei der Aktivierung des Prozesses nichts anderes angibt. 
  Wenn defaultValue nicht gesetzt ist, gibt es keinen Defaultwert.
* **required**: true, falls der Aktivierung des Prozesses immer ein Wert angegeben werden muss, 
  oder false, wenn nicht. Default ist false.
* **hidden**: true, wenn der Prozessparameter bei der Aktivierung nicht sichtbar sein soll,
  oder false, wenn der Prozessparameter bei der Aktivierung sichtbar sein soll. Default ist false.
  
Beispielhafter Inhalt einer Prozessparameter-Definitions-Datei:
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

Diese Dateien entsprechen vom Format den Dateien, welche auch im Admincenter hoch bzw. heruntergeladen werden können.


