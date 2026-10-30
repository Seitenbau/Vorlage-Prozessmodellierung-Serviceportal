# Vorlage zum Erstellen und automatisierten Hochladen eines Serviceportal-Prozessmodells

## Übersicht
Die Dokumentation besteht aus folgenden Teilen:

* Projektvorlage (dieses Repository)
* [Gradle-Plugin (aus Gradle-Plugin-Repository)](https://doku.pmp.seitenbau.com/pages/viewpage.action?pageId=56559435)
* [Schnittstellendokumentation](https://doku.pmp.seitenbau.com/display/DFO/Schnittstellendokumentation+Serviceportale) 
* [Änderungen](https://doku.pmp.seitenbau.com/display/DFO/Changelog+Gradle-Plugin+Serviceportale)
* [Breaking Changes & Migration Guides](https://doku.pmp.seitenbau.com/display/DFO/Breaking+Changes+und+Migration+Guides)

## Ankündigungen

### Deprecation von Projektstufen am 29.02.2024

Projektstufen werden ausgebaut und sind nicht mehr nutzbar. Plugins mit Version < `2023.06.19-0`
sind dadurch nicht mehr nutzbar. Zusätzlich muss die Projektstruktur angepasst werden.
Siehe [Breaking Changes & Migration Guides](https://doku.pmp.seitenbau.com/display/DFO/Breaking+Changes+und+Migration+Guides) 

# Projektvorlage

Arbeitsablauf:

1. Ein neues leeres GIT-Repository auf dem Server für das Projekt erstellen. 
2. Das neue Projekt clonen und einen initialen Commit machen.
3. Dieses Repository (also die Vorlage, in welcher Sie gerade lesen) als ein zweites remote-Repository 
(vorgeschlagener Name: `template`) zu Ihrem Projekt hinzufügen: `git remote add template {link zu diesem repo}`
4. `git fetch --all` aufrufen
5. `git merge template/master --allow-unrelated-histories` alle Dateien aus der Vorlage in der Projekt mergen.
6. Den Projektnamen "Prozessmodellierungsvorlage" in den folgenden Dateien in den tatsächlichen Namen des Projektes ändern:
   - `settings.gradle`
   - `config/project.json`
7. Ggf. in der Datei `build.gradle` die gewünschte Version des Gradle-Plugins setzen     
8. Die Submodule (sofern vorhanden und benutzt) initialisieren: `git submodule update --init --recursive`
9. `git push` die gemachten Änderungen
10. Anlegen der Formulare, Scripte, Prozessparameterdefinitionen, Datenschutzerklärung und Scripte 

Alternativ kann auch dieses Repository ausgecheckt werden und die benötigten Dateien 
und Verzeichnisse per copy&paste in das Projektverzeichnis übertragen werden.

## Konfiguration und Projektstruktur

Die Konfiguration des Plugins und die Projektstruktur wird in der Dokumentation des Gradle-Plugins
[v1](https://doku.pmp.seitenbau.com/spaces/DFO/pages/182683488/Konfiguration+und+Projektstruktur)
und der Dokumentation der 
[v2](https://doku.pmp.seitenbau.com/spaces/DFOV2/pages/176788085/Konfiguration+und+Projektstruktur)
beschrieben.

## Breaking Changes & Migration Guides

Informationen über Änderungen am Plugin, die ein Anpassen der Konfigurationsdateien 
oder sonstiger Projektbestandteile erfordern, werden in [Breaking Changes](https://doku.pmp.seitenbau.com/display/DFO/Breaking+Changes+und+Migration+Guides) beschrieben.
