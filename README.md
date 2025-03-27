# CaC-Intune
Configuration as Code for Intune

# Intune-Konfigurationsautomatisierung

Dieses Projekt automatisiert die Sicherung, Änderungsverfolgung und Dokumentation Ihrer Microsoft Intune-Konfiguration mithilfe einer Azure Pipeline.

## Funktionsweise der Azure Pipeline

Die Azure Pipeline führt die folgenden Schritte aus [1, 2]:

*   **IntuneCD Tool installieren** [1]
*   **Intune-Konfigurationssicherung in den Ordner `prod-backup` exportieren** [1]
*   **Erkennung von Konfigurationsänderungen:**
    *   Wenn eine Änderung erkannt wird [1]
    *   werden für jede geänderte Datei die Autoren ermittelt [1]
    *   durchsucht die Pipeline das Intune-Auditprotokoll nach bestimmten `ResourceId`-Änderungen seit dem Datum des letzten Konfigurations-Commits [1, 3]
    *   werden die geänderten Dateien nach den Autoren gruppiert [1, 3]
    *   und jede Gruppe separat committet, wobei die Namen der Autoren als Commit-Autoren verwendet werden [1, 4]
*   **Optionale Generierung von Dokumentation:**
    *   Erstellung einer Intune-Konfigurationsdokumentationsdatei im Markdown-Format [2] (nur wenn die Aufgabe "Markdown-Dokument generieren & committen" aktiviert ist)
    *   Erstellung eines Git TAG, wobei die Namen aller Änderungsautoren in die TAG-Beschreibung aufgenommen werden [2, 4]
    *   Generierung von HTML- und PDF-Dateien aus der Intune-Konfigurationsdokumentationsdatei und Speicherung als Pipeline-Artefakte [2] (nur wenn die entsprechenden Aufgaben aktiviert sind und der Ordner `md2pdf` im Repository-Root vorhanden ist)
*   **Beendigung der Pipeline**, wenn keine Änderungen vorhanden sind [5]

## Benachrichtigung bei Pipeline-Fehlern

Es ist eingerichtet, dass **Benachrichtigungen bei Pipeline-Fehlern versendet werden** [5]. Dies kann beispielsweise bei einem abgelaufenen Anwendungsgeheimnis hilfreich sein [5]. Die Einrichtung erfolgt unter `Projekteinstellungen > Benachrichtigungen > Neues Abonnement > Build > Ein Build schlägt fehl` [5].

## Wiederherstellungsprozess

Grundsätzlich gibt es zwei Möglichkeiten, eine Sicherung wiederherzustellen [6]:

1.  **Verwendung der IntuneCD-Updatefunktion:**
    *   Erstellen Sie einen neuen Ordner [6].
    *   Kopieren Sie nur die Ordnerstruktur mit den wiederherzustellenden JSON-Dateien in diesen Ordner [6].
    *   Führen Sie `"IntuneCD-startupdate" mit dem Argument "--path "PfadZumBackupOrdner""` aus [6].
2.  **Manuelles Erstellen der gewünschten Intune-Konfigurationen:**
    *   Füllen Sie die Werte in den JSON-Dateien Ihres Backups ein [6].

## Ändern der Datensicherungselemente

Um festzulegen, welche Intune-Konfigurationen gesichert werden sollen, müssen Sie den Abschnitt "**Create Intune backup task**" im Pipeline-Konfigurationscode bearbeiten [6, 7]. Gehen Sie zu `Pipelines > <IhrPipelineName> > Bearbeiten > Suchen Sie nach einer Aufgabe mit dem Namen Create Intune backup` und ändern Sie deren Skript-Abschnitt, indem Sie den Parameter `--exclude` modifizieren [7]. Standardmässig ist `CompliancePartner` ausgeschlossen [7].

## Ermittlung des Änderungsautors

Wenn eine Änderung in einer JSON-Konfigurationsdatei erkannt wird, führt die Pipeline folgende Schritte zur Ermittlung des Autors aus [3, 7]:

1.  Abrufen aller geänderten Dateien [3]
2.  Abrufen des Datums des letzten Konfigurations-Backup-Commits (`$lastCommitDate`) [3] (falls kein Commit vorhanden ist, wird das gesamte Intune-Auditprotokoll durchsucht [3])
3.  Für jede geänderte Datei:
    *   Extrahieren der Ressourcen-ID aus dem Dateinamen [3]
    *   Suchen dieser ID im Intune-Auditprotokoll (zwischen `$lastCommitDate` und dem Startzeitpunkt des aktuellen Backups) [3]
    *   Notieren aller Autoren, die dort Änderungen vorgenommen haben [3]
4.  Gruppieren der geänderten Dateien nach Autoren und Erstellen von Commits [3, 4]
5.  Verwendung aller gefundenen Autoren im Commit-Namen und als Commit-Autoren für jede Gruppe [4]
6.  Aufnahme aller gefundenen Autoren in die Git-Tag-Beschreibung [4]

**Wichtiger Hinweis:** Die Liste der ermittelten Autoren ist möglicherweise nicht zu 100 % korrekt [4]. Für absolute Genauigkeit (z. B. bei Sicherheitsvorfällen) wird die Verwendung des Befehls `"Get-MgDeviceManagementAuditEvent"` empfohlen [4].

**Ungenauigkeiten können auftreten, wenn:**

*   Jemand eine Änderung direkt nach dem Start des Backups vornimmt. Die Änderung wird erfasst, der Autor jedoch möglicherweise nicht, da das Auditprotokoll nur bis zum Startzeitpunkt des Backups durchsucht wird [8].
*   Die Commit-Autoren zeigen lediglich, wer Änderungen an den commiteten Dateien vorgenommen hat (seit dem letzten Commit), nicht wer welche spezifische Änderung vorgenommen hat [9].
*   Eine Änderung von einer Anwendung vorgenommen wurde. In diesem Fall wird der Name der Anwendung mit dem Suffix `(SP)` als Autorenname verwendet (z. B. `"Modern Workplace Management (SP)"`) [9].
*   Bestimmte Intune-Konfigurationsänderungen (z. B. an ESP-Profilen) nicht im Intune-Auditprotokoll erfasst werden. In solchen Fällen wird `"unbekannt"` als Autor verwendet [9, 10].
*   Intune verschiedene ID-Formate verwendet (nicht nur `<GUID>`, sondern z. B. `<GUID>_<GUID>`, `<GUID>_<Zeichenfolge>`, `T_<GUID>`) [10].


