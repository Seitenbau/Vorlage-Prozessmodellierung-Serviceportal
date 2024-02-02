# Breaking Changes

## Index

1. [Umstellung von Projektstufen zu Statuswerten](#umstellung-projektstufen-zu-statuswerten)
   1. [Migration Guide](#migration-guide)
   2. [Tabelle: Mapping der alten Stufen auf neue Statuswerte](#tabelle-mapping-der-alten-stufen-auf-neue-statuswerte)

## Umstellung Projektstufen zu Statuswerten

Mit der aktuellsten version des Gradle Plugins wurden die Projektstufen entfernt und durch
Statuswerte ersetzt. 
Die Schnittstelle selbst funktioniert übergangsweise sowohl mit der Stufe, als auch dem neuen Status.
Es darf jedoch nur einer der beiden Werte an die Schnittstelle übertragen werden.

**Deprecated:** Ab dem 29.02.2024, wird die Verwendung der Projektstufen vollständig ausgebaut 
und komplett auf die neuen Statuswerte umgestellt. Daher ist nicht gewährleistet das die Endpunkte
zukünftig weiterhin mit den Projektstufen funktionieren.


### Migration Guide

Um zur aktuellsten Plugin Version zu migrieren, müssen folgende aktionen durchgeführt werden.

1. Konfigurationsdateien anpassen. Statt dem Attribut `projectStage` sollte das Attribut `status` verwendet werden 
2. Die neuen Statuswerte statt den ehemaligen Stufen verwenden. Siehe hierfür die folgenden Tabellen

Die Werte des veralteten `projectStage` Attributs können anhand der nachfolgenden Tabelle
in die neuen Statuswerte übertragen werden.

#### Beispiel
##### Alte config.json

```json
{
   "url": "http://example.com:81/sgw",
   "projectStage": "TECHNICAL_IMPLEMENTATION"
}
```

##### Neue config.json

```json
{
   "url": "http://example.com:81/sgw",
   "status": "EDIT"
}
```

### Tabelle: Mapping der alten Stufen auf neue Statuswerte

Die folgenden Tabellen stellen dar welcher Statuswert die selbe Funktionalität bietet wie in der
vorherigen Plugin version eine bestimmte Projektstufe.

#### Formulare

| Projektstufe      | Status |
|-------------------|--------|
| MODELLING         | EDIT   |
| QUALITY_ASSURANCE | TEST   |
| CERTIFIED         | FINAL  |

#### Prozesse

| Projektstufe             | Status |
|--------------------------|--------|
| FUNCTIONAL_ANALYSIS      | EDIT   |
| TECHNICAL_IMPLEMENTATION | EDIT   |
| QUALITY_ASSURANCE        | TEST   |
| CERTIFIED                | FINAL  |


