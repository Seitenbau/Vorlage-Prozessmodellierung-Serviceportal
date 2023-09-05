# Aktualisierungen

## 2023.09.05-0
- Dependency-Updates: gradlew 7.6.1

## 2023.07.20-0
- Verbesserung: bei übergabe einer nicht vorhandenen Umgebungs-Konfigurations-Datei über das Flag
  `-Penvironment=<environmentName>` wird eine eindeutigere Fehlermeldung ausgegeben.

## 2023.06.30-0
- Bugfix: bei Schnittstellen-Aufrufen den HTTP-Header `Accept` auf den von der Schnittstelle
erwarteten Wert setzen

## 2023.06.19-0
- Mit Plugin Version `2023.06.19-0` wurden die Projektstufen ausgebaut.
Für weitere Informationen siehe den [Migration Guide](migration.md).
- `projectStage` ersetzt durch `status` in `default.json`.
- `migration.md` hinzugefügt mit Hinweisen zur Umstellungen der Konfigurationsdateien.
- Dokumentation aktualisiert.

## 2022.12.14-0
- Dependency-Updates: Plugin: de.seitenbau.serviceportal.prozesspipeline 2022.12.14-0, gradlew 7.6

## 2022.10.07-0
- Bugfix: Beim Zusammenführen von Quellcode-Dateien mittels `mergeScripts` prüfen, dass die Imports
  in den erzeugten Groovy-Skripten eindeutig sind

## 2022.09.27-0
- Dependency-Updates: Plugin: de.seitenbau.serviceportal.prozesspipeline 2022.09.27-0, gradlew 7.5.1

## 2022.09.01-0
- Behebung eines Fehlers, der in manchen Fällen im Task `buildModel` dazu geführt hat, dass ein 
Prozessmodell erzeugt wurde, welches auf OZG-Hub Umgebungen nicht deployt werden konnte.

## 2022.03.23-0
- Neues Feature: Dateien im Ordner resources werden jetzt as-is in das Deployment
  des Prozesses aufgenommen
- Dependency-Updates: log4j 2.17.0, guava 31.1-jre, jackson-databind 2.13.2, 
  gson 2.9.0, jsoup 1.14.3, j2html 1.5.0, jdom2 2.0.6.1, commons-io 2.11.0,
  commons-lang3 3.12.0
  
## 2021.12.14-0
- Dependency-Update auf log4j 2.16.0
- Optimierung: Logging beim Laden der Konfigurationsdateien verbessert

## 2021.08.25-0
- Ein Task zum Abfragen der Prozess-Engines wurde hinzugefügt
- Der Task zum deployen einer Prozessmodell-Version wurde dahingehend ergänzt, dass die
  Prozess-Engine, auf die deployt werden soll, angegeben werden kann. (SBW-20143)

## 2021.03.03-0
- Optimierung: Der lokale HTTP-Server listet Dateien mit Bindestrichen auf. (SBW-19921)

## 2021.02.25-1
- Bugfix: Laden und bearbeiten eines leeren Formulars im lokalen Formulardesigner ermöglichen.
- Optimierung: Anzeige, wo die Datei currentServerPort.txt liegt.
