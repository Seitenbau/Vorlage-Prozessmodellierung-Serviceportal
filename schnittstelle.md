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
| **Name**              | **Beschreibung**                                                                                                                                                                                                                                                        |
|-----------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| X-SP-Formular-Name    | Name des Formulars                                                                                                                                                                                                                                                      |
| X-SP-Formular-Version | Version des Formulars                                                                                                                                                                                                                                                   |
| X-SP-Formular-Stage   | WorkflowStage des Formulars<ul><li>`MODELLING`</li><li>`QUALITY_ASSURANCE`</li><li>`CERTIFIED`</li></ul> <b>*Deprecated:</b> Header wird mit folgenden Versionen vollständig entfernt. Migration erforderlich falls noch genutzt, siehe [Migration Guide](migration.md) |
| X-SP-Formular-Status  | Status des Formulars<ul><li>`EDIT`</li><li>`TEST`</li><li>`FINAL`</li></ul>                                                                                                                                                                                             |
| X-SP-Formular-Deploy  | `true`, wenn das Formular deployt werden soll, default: `false`                                                                                                                                                                                                         |
| X-SP-Mandant          | ID des Mandanten                                                                                                                                                                                                                                                        |
| Content-Type          | `application/zip`                                                                                                                                                                                                                                                       |

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

Für jede Umgebung ist ein maximaler Status definiert. Wird versucht 
Formulare in einem Status hochzuladen und/oder zu deployen, der höher ist als der maximale, 
wird ein Fehler zurückgegeben.

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
im Backend an das Deployen eines Prozesses gekoppelt und wird automatisch mit ausgeführt) deployt werden

### Pfad
`{pfad zum sgw}/prozessmanagement/prozessmodelle/api/upload`

### Headerparameter
| **Name**             | **Beschreibung**                                                                                                                                                                                                                                                                                                           |
|----------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| X-SP-Process-Name    | Name des Prozesses                                                                                                                                                                                                                                                                                                         |
| X-SP-Process-Version | Version des Prozesses                                                                                                                                                                                                                                                                                                      |
| X-SP-Process-Stage   | WorkflowStage des Prozesses<br /><ul><li>`FUNCTIONAL_ANALYSIS`</li><li>`TECHNICAL_IMPLEMENTATION`</li><li>`QUALITY_ASSURANCE`</li><li>`CERTIFIED`</li></ul> <b>*Deprecated:</b> Header wird mit folgenden Versionen vollständig entfernt. Migration erforderlich falls noch genutzt, siehe [Migration Guide](migration.md) |
| X-SP-Process-Status  | Status des Prozesses<br /><ul><li>`EDIT`</li><li>`TEST`</li> <li>`FINAL`</li></ul>                                                                                                                                                                                                                                         |
| X-SP-Mandant         | ID des Mandanten                                                                                                                                                                                                                                                                                                           |
| Content-Type         | `application/zip`                                                                                                                                                                                                                                                                                                          |

### Body
Die Schnittstelle erwartet als Body eine ZIP-Datei, die vom Aufbau her einer Erweiterung des Business Archive von Activiti (https://www.activiti.org/userguide/#_business_archives) entspricht:

Die Schnittstelle verarbeitet die im Hauptordner des Archives abgelegten `*.bpmn20.xml`-Dateien
und die `metadata.json`-Datei.
```
/
    metadata.json
    mainProcess.bpmn20.xml
    subProcess1.bpmn20.xml
    subProcess2.bpmn20.xml
    [...]
```

### Berechtigungen

Um diese Schnittstelle aufrufen zu können benötigt der Benutzer folgende Rechte:
*  **hochladen**: prozessmanagement:prozessmodelle.api.Hochladen

Das Recht ist in den oben genannten Rollen und Benutzergruppen enthalten.

Für jede Umgebung ist ein maximaler Status definiert. 
Wird versucht eine Prozessmodellversion in einem Status hochzuladen, der höher ist als der maximale Status, 
wird ein Fehler zurückgegeben.

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
| **Name**             | **Beschreibung**                                                  |
|----------------------|-------------------------------------------------------------------| 
| X-SP-Process-Name    | Name des Prozesses                                                |
| X-SP-Process-Version | Version des Prozesses                                             |
| X-SP-Process-Engine  | ID der Prozess-Engine, Standard-Prozess-Engine wenn nicht gesetzt |
| X-SP-Mandant         | ID des Mandanten                                                  |

### Body
Die Schnittstelle erwartet einen leeren Body. 

### Berechtigungen

Um diese Schnittstelle Aufrufen zu können benötigt der Benutzer folgende Rechte:
*  **deployen**: prozessmanagement:prozessmodelle.api.Deployen

Das Recht ist in den oben genannten Rollen und Benutzergruppen enthalten.

Für jede Umgebung ist ein maximaler Status definiert. Wird versucht eine Prozessmodellversion in einem Status
zu deployen, der höher ist als der maximale, wird ein Fehler zurückgegeben.

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
|----------------------|-----------------------|
| X-SP-Process-Name    | Name des Prozesses    |
| X-SP-Process-Version | Version des Prozesses |
| X-SP-Mandant         | ID des Mandanten      |
| Content-Type         | `application/zip`     |

Da Prozessparameterdefinitionen zu einer Prozessmodellversion und nicht einem spezifischen Status gehören,
muss kein Status angegeben werden. 

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

---------------------------------------------------------------------------------------------------

### Schnittstelle zum ermitteln der Deployment-ID eines Onlinedienstes

#### Allgemein

Die Schnittstelle ermittelt die Deployment-ID eines Onlinedienstes.

Der Aufruf muss als **GET** ausgeführt werden.

#### Pfad

`{URL der Umgebung}/prozessmanagement/prozessmodelle/deployment/{processModelName}/{processModelVersionName}`

#### Path-Parameter

| **Name**                 | **Pflicht** | **Typ** | **Beschreibung**                                                                     |
|--------------------------|-------------|---------|--------------------------------------------------------------------------------------|
| processModelName         | Ja          | String  | Der Names der Prozessmodells für den die Deployment-ID ermittelt werden soll.        |
| processModelVersionName  | Ja          | String  | Der Versionsname des Prozessmodells für das die Deployment-ID ermittelt werden soll. |


#### Rückgabewerte

Die Rückgabe ist vom Typ (`application/json`), es wird jedoch nur die ermittelte Deployment-ID zurückgegeben.<br />

```
Sa6DGsfXOud4fWSpFPwOLD
```

---------------------------------------------------------------------------------------------------

### Schnittstelle zum auflisten aller erstellten zeitgesteuerte Undeployments

#### Allgemein

Die Schnittstelle listet alle zeitgesteuerten Undeployments für den übergebenen Mandanten auf.

Der Aufruf muss als **GET** ausgeführt werden.

#### Pfad

`{URL der Umgebung}/prozess/scheduled/undeployment/list`

#### Header-Parameter

| **Name**     | **Pflicht** | **Typ**  | **Beschreibung** |
|--------------|-------------|----------|------------------|
| X-SP-Mandant | Ja          | Ganzzahl | ID des Mandanten |

#### Rückgabewerte

Ein Objekt (`application/json`), das eine Liste der erstellten zeitgesteuerte Undeployments des Mandanten enthält.<br />

```json
{
  "value": [
      {
          "deploymentId": "Sa6DGsfXOud4fWSpFPwOLD",
          "undeploymentDate": 1707519600000,
          "undeploymentMessage": {},
          "undeploymentAnnounceMessage": {}
      },
      {
          "deploymentId": "FFb0ffdVnt9VmUN6AtT7BQ",
          "undeploymentDate": 1707519603500,
          "undeploymentMessage": {
              "subject": "Undeployment des Prozesses",
              "body": "Der Prozess muss leider undeployed werden..."
          },
          "undeploymentAnnounceMessage": {}
      }
  ],
    "complete": true
}
```

---------------------------------------------------------------------------------------------------

### Schnittstelle zum erstellen eines zeitgesteuertes Undeployments

#### Allgemein

Die Schnittstelle erstellt ein zeitgesteuertes Undeployment und setzt Parameter die später zum Benachrichtigen der User verwendet werden.

Der Aufruf muss als **POST** ausgeführt werden.

#### Pfad

`{URL der Umgebung}/prozess/scheduled/undeployment`

#### Header-Parameter

| **Name**     | **Pflicht** | **Typ**  | **Beschreibung** |
|--------------|-------------|----------|------------------|
| X-SP-Mandant | Ja          | Ganzzahl | ID des Mandanten |

#### Request-Body

- Die Schnittstelle erwartet als Body ein JSON-Objekt mit folgender Struktur:

```json
{
    "deploymentId": "deploymentId des Prozessmodells",
    "undeploymentDate": "2024-02-17",
    "undeploymentAnnounceMessage": {
        "subject": "Betreff der Nachricht",
        "body": "Inhalt der Nachricht"
    },
    "undeploymentMessage": {
        "subject": "Betreff der Nachricht",
        "body": "Inhalt der Nachricht"
    }
}
```

| **Name**                    | **Pflicht** | **Typ** | **Beschreibung**                                                                                                      |
|-----------------------------|-------------|---------|-----------------------------------------------------------------------------------------------------------------------|
| deploymentId                | Ja          | String  | Deployment-ID des Online-Dienstes, der undeployt werden soll.                                                         |
| undeploymentDate            | Ja          | Date    | Das Datum, an dem der Online-Dienst undeployt werden soll (TT.MM.YYYY).                                               |
| undeploymentAnnounceMessage | Nein        | Message | Eine Nachricht die 1, 7 und 14 Tage vor dem eigentlichen Undeployment verschickt wird und das Undeployment ankündigt. |
| undeploymentMessage         | Nein        | Message | Eine Nachricht die beim Undeployment des Prozessmodells verschickt wird.                                              |

##### Message Objekt

| **Name** | **Pflicht** | **Typ** | **Maximale Anzahl Zeichen** | **Beschreibung**                                                                                                                   |
|----------|-------------|---------|-----------------------------|------------------------------------------------------------------------------------------------------------------------------------|
| subject  | Nein        | Objekt  | 255                         | Der Betreff der Nachricht.                                                                                                         |
| body     | Nein        | Objekt  | 2000                        | Der Inhalt der Nachricht. Hier können Platzhalter verwenden siehe hierzu "[Platzhalter Message Body](#Platzhalter-Message-Body)".  |

###### Platzhalter Message Body

| Name                      | Beschreibung                                         | Beispiel                                                                            |
|---------------------------|------------------------------------------------------|-------------------------------------------------------------------------------------|
| name                      | Wird aufgelöst zum Namen des Empfängers.             | Max Mustermann                                                                      |
| tageBisUndeployment       | Anzahl der Tage bis der Onlinedienst undeployt wird. | 14                                                                                  |
| linkAufAktuellenProzess   | Ein Verweis auf den aktuellen Prozess.               | {protal-url}/onlineantraege/onlineantrag?processInstanceId=zsoh_zgxiZaUpzuA6eLqzQ   |

#### Rückgabewerte

Der Endpunkt liefert bei Erfolg den HTTP-Status `204 No content` ohne Response Body zurück.

---------------------------------------------------------------------------------------------------

### Schnittstelle zum Löschen von zeitgesteuerte Undeployments

#### Allgemein

Die Schnittstelle löscht ein zeitgesteuertes Undeployment.

Der Aufruf muss als **DELETE** ausgeführt werden.

#### Pfad

`{URL der Umgebung}/prozess/scheduled/undeployment/{deploymentId}`

#### Path-Parameter

| **Name**     | **Pflicht** | **Typ** | **Beschreibung**                                                                                 |
|--------------|-------------|---------|--------------------------------------------------------------------------------------------------|
| deploymentId | Ja          | String  | Deployment-ID des Online-Dienstes, für den das zeitgesteuerte Undeployment gelöscht werden soll. |


#### Header-Parameter

| **Name**     | **Pflicht** | **Typ**  | **Beschreibung** |
|--------------|-------------|----------|------------------|
| X-SP-Mandant | Ja          | Ganzzahl | ID des Mandanten |


#### Rückgabewerte

Der Endpunkt liefert bei Erfolg den HTTP-Status `204 No content` ohne Response Body zurück.