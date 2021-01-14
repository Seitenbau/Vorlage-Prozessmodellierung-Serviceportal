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

Alternativ kann auch dieses Repository ausgecheckt werden und die benötigten Dateien 
und Verzeichnisse per copy&paste in das Projektverzeichnis übertragen werden.

## Konfiguration und Projektstruktur

Die Konfiguration des Plugins und die Projektstruktur wird in 
[der Dokumentation des Gradle-Plugins](plugin.md) beschrieben.