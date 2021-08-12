# Übersicht

Dieses Dokument beschreibt die Schnittstelle seitens des Serviceportals, welche durch das 
Gradle-Plugin angesprochen wird.

Informationen zum Gradle-Plugin und der Vorlage siehe
 [die readme.md der Vorlagendokumentation](readme.md)

# Schnittstellendokumentation

## Berechtigungen

Für die Benutzung der Schnittstellen müssen die Benutzer entsprechende Rechte zugeteilt bekommen. 
Diese sind in den Abschnitten zu den einzelnen Aufrufen dokumentiert. Darüber hinaus sind die Rechte
bereits standardmäßig auf allen Umgebungen angelegten Rollen und Benutzergruppen zusammengefasst:    

* Rollen
  * `Prozessentwickler - hochladen via Schnittstelle`
  * `Prozessentwickler - deployment via Schnittstelle`
* Benutzergruppen 
  * `Prozessentwickler - hochladen via Schnittstelle`
  * `Prozessentwickler - deployment via Schnittstelle`

### Token

Die Authentifizierung erfolgt über ein Bearer-Token eines Admincenter-Benutzers (**nicht** Webservice-Benutzers). 
Das Authentifizierungstoken benötigt den Scope `autoDeployment` und muss zu dem Mandanten ausgestellt 
worden sein, unter welchem die Aktion ausgeführt werden soll. 

Das Token kann über die Rest-Schnittstelle (per Rest-Dokumentation bzw. direkten `curl`-Aufruf) oder 
mittels des `getAuthentifizierungsToken`-Task erstellt werden (siehe Dokumentation des Gradle-Plugins)

## Schnittstelle zum Hochladen und Deployen von Formularen

### Allgemein
Die Schnittstelle ermöglicht das Hochladen eines Archives mit den zu einer Formularversion gehörenden
Formulardateien (Sprachen) sowie das anschließende Deployen der Formularversion.
Der Aufruf muss als POST ausgeführt werden.

Durch Setzen des Header-Parameters `X-SP-Deploy: true` können die Formulare direkt im Anschluss an das
Anlegen der Formulare im Admincenter deployt werden.

### Pfad
`{pfad zum sgw}/formulare/management/formulare/api/upload`

### Headerparameter
| **Name**              |  **Beschreibung** |
| --------------------- | ----------------- |
| X-SP-Formular-Name    | Name des Formulars |
| X-SP-Formular-Version | Version des Formulars |
| X-SP-Formular-Stage   | WorkflowStage des Formulars<br />`MODELLING`, `QUALITY_ASSURANCE`, `CERTIFIED` |
| X-SP-Formular-Deploy  | `true`, wenn das Formular deployt werden soll, default: `false` |
| X-SP-Mandant          | ID des Mandanten |
| Content-Type          | `application/zip` |

### Body
Die Schnittstelle erwartet als Body eine ZIP-Datei. 
Dabei enthält das Archiv für jede Sprache eine `json`-Datei mit dem 
Namen `(de|en|fr).json`(entsprechend der angedachten Sprache) im Stammverzeichnis. 
```
/
    de.json
    fr.json
    en.json
```

### Berechtigungen

Um diese Schnittstelle aufrufen zu können benötigt der Benutzer folgende Rechte:
*  **hochladen**: formular:formulare.api.Hochladen
*  **deployen**: formular:formulare.api.Deployen

Die Rechte sind in den oben genannten Rollen und Benutzergruppen enthalten.

Für jede Umgebung ist eine maximale Stufe definiert. Wird versucht 
Formulare in eine Stufe höher der angegebenen hochzuladen und/oder zu deployen, wird ein Fehler zurückgegeben.

### Rückgabewerte 

Einen Stream (`application/txt`) mit einem Protokoll der ausgeführten Aktionen.

----------------------------------------------------------------------------------------------------

## Schnittstelle zum Hochladen von Prozessen

### Allgemein

Die Schnittstelle ermöglicht das Hochladen eines Archives mit den zu einer Prozessmodellversion gehörenden Prozessmodelldateien.
Der Aufruf muss als POST ausgeführt werden.

Bei Prozessen sind die Schritte Hochladen und Deployen in zwei Schritte getrennt, 
um nach dem Hochladen des Prozesses in einem separaten Aufruf die zugehörigen Prozessparameterdefinitionen hochladen zu können.

Anschließend können beide gemeinsam (das Deployen von Prozessparameterdefinitionen ist 
im Backend an das Deplyoen eines Prozesses gekoppelt und wird automatisch mit ausgeführt) deployt werden

### Pfad
`{pfad zum sgw}/prozessmanagement/prozessmodelle/api/upload`

### Headerparameter
| **Name**             | **Beschreibung** |
| -------------------- | ---------------- |
| X-SP-Process-Name    | Name des Prozesses |
| X-SP-Process-Version | Version des Prozesses |
| X-SP-Process-Stage   | WorkflowStage des Prozesses<br />`FUNCTIONAL_ANALYSIS`, `TECHNICAL_IMPLEMENTATION` , `QUALITY_ASSURANCE`, `CERTIFIED` |
| X-SP-Mandant         | ID des Mandanten |
| Content-Type         | `application/zip` |

### Body
Die Schnittstelle erwartet als Body eine ZIP-Datei, welche vom Aufbau her dem Business Archive von Activiti entspricht (https://www.activiti.org/userguide/#_business_archives):

Die Schnittstelle verarbeitet die im Hauptordner des Archives abgelegten `*.bpmn20.xml`-Dateien. 
```
/
    mainProcess.bpmn20.xml
    subProcess1.bpmn20.xml
    subProcess2.bpmn20.xml
    [...]
```

### Berechtigungen

Um diesse Schnittstelle aufrufen zu können benötigt der Benutzer folgende Rechte:
*  **hochladen**: prozessmanagement:prozessmodelle.api.Hochladen

Das Recht ist in den oben genannten Rollen und Benutzergruppen enthalten.

Für jede Umgebung ist eine maximale Stufe definiert. Wird versucht eine Prozessmodellversion in eine Stufe höher 
als der angegebenen hochzuladen, wird ein Fehler zurückgegeben.

### Rückgabewerte 

Einen Stream (`application/txt`) mit einem Protokoll der ausgeführten Aktionen.

----------------------------------------------------------------------------------------------------

## Schnittstelle zum Deployen von Prozess-Versionen

### Allgemein
Die Schnittstelle ermöglicht das Deployen einer Prozessmodellversion auf eine spezifische Prozess-Engine.
Ist keine Engine angegeben, wird auf die Standard-Prozess-Engine deployt.
Der Aufruf muss als POST ausgeführt werden.

### Pfad
`{pfad zum sgw}/prozessmanagement/prozessmodelle/versions/api/deploy`

### Headerparameter
| **Name**             | **Beschreibung** |
| -------------------- | ---------------- | 
| X-SP-Process-Name    | Name des Prozesses |
| X-SP-Process-Version | Version des Prozesses |
| X-SP-Process-Stage   | WorkflowStage des Prozesses<br />`FUNCTIONAL_ANALYSIS`, `TECHNICAL_IMPLEMENTATION` , `QUALITY_ASSURANCE`, `CERTIFIED` |
| X-SP-Process-Engine  | ID der Prozess-Engine, Standard-Prozess-Engine wenn nicht gesetzt |
| X-SP-Mandant         | ID des Mandanten |

### Body
Die Schnittstelle erwartet einen leeren Body. 

### Berechtigungen

Um diese Schnittstelle Aufrufen zu können benötigt der Benutzer folgende Rechte:
*  **deployen**: prozessmanagement:prozessmodelle.api.Deployen

Das Recht ist in den oben genannten Rollen und Benutzergruppen enthalten.

Für jede Umgebung ist eine maximale Stufe definiert. Wird versucht eine Prozessmodellversion in eine Stufe höher 
der angegebenen deployt, wird ein Fehler zurückgegeben.

### Rückgabewerte 

Ein Objekt mit den Informationen zu den Vorgängen (`application/json`).
```json
{
    "undeployed": false,
    "undeployedInstanceCount": 0,
    "deployResponse": {
        "status": "SUCCESS",
        "deploymentId": "41896",
        "duplicateKeys": [
            "m1.p212.alleformularfeldtypen"
        ]
    }
}
```

----------------------------------------------------------------------------------------------------

## Schnittstelle zum Hochladen von Prozessparameterdefinitionen

### Allgemein
Die Schnittstelle ermöglicht das Hochladen eines Archives mit den zu einem Prozessmodell gehörenden Prozessmodelldateien.
Der Aufruf muss als POST ausgeführt werden.

Die Schnittstelle bietet keine Option zum Deployen der Prozessparameterdefinitionen. Dieses ist an
das Deployen der zugehörigen Prozessmodellversion gekoppelt. Da ein Hochladen der Prozessparameterdefinition 
daher die Möglichkeit eines Deployments implizit enthält, wird zum Hochladen das Deploy-Recht benötigt.  

### Pfad
`{pfad zum sgw}/prozessmanagement/prozessparameterdefinitionen/api/deploy`

### Headerparameter
| **Name**             | **Beschreibung**      |
| -------------------- | --------------------- |
| X-SP-Process-Name    | Name des Prozesses    |
| X-SP-Process-Version | Version des Prozesses |
| X-SP-Mandant         | ID des Mandanten      |
| Content-Type         | `application/zip`     |

Da Prozessparameterdefinitionen zu einer Prozessmodellversion und nicht einer spezifischen Stufe gehören,
muss keine Stufe angegeben werden 

### Body
Die Schnittstelle als Body eine JSON-Datei, welche vom Aufbau her derjenigen des Hoch- und Herunterladens im Admincenter entspricht. 

### Berechtigungen

Um diese Schnittstelle aufrufen zu können benötigt der Benutzer folgende Rechte:
*  **hochladen**: prozessmanagement:prozessparametervariablen.api.Hochladen
*  **deployen**: prozessmanagement:prozessparametervariablen.api.Deployen

Die Rechte sind in den oben genannten Rollen und Benutzergruppen enthalten.

### Rückgabewerte 

Ein Objekt mit einer Liste der angelegten Prozessparameterdefinitionen (`application/json`),
entspricht den hochgeladenen Daten plus die vergebenen id's 

----------------------------------------------------------------------------------------------------

## Schnittstelle zum Abfragen der Liste aktiver Prozess-Engines

### Allgemein
Die Schnittstelle ermöglicht es die Liste der aktiven Prozess-Engines zu holen.
Der Aufruf muss als GET ausgeführt werden.

### Pfad
`{pfad zum sgw}/prozess/engine`

### Body
Die Schnittstelle erwartet einen leeren Body.

### Berechtigungen
Um die Schnittstelle aufrufen zu können sind keine weiteren Rechte notwendig.

### Rückgabewerte
Eine Liste mit den IDs und Namen der Prozess-Engines (`application/json`).

```json
[
  {
    "id": "defaultEngine",
    "name": "Standard-Prozess-Engine"
  },
  {
    "id": "secondEngine",
    "name": "Zweite Prozess-Engine"
  }
]
```
