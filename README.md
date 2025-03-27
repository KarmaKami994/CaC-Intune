# Intune Konfiguration als Code Automation

Dieses Projekt automatisiert die **Sicherung und Verwaltung Ihrer Microsoft Intune Konfigurationen** mithilfe von **Configuration as Code (CaC)**, **IntuneCD** und **Azure DevOps Pipelines**.

## Ziel

Implementierung einer **automatischen Datensicherungslösung für Intune**, um Datenverlust vorzubeugen [1].

## Funktionsweise

Die **Azure Pipeline** [2]:

*   **Exportiert** die Intune-Konfiguration [2].
*   **Erkennt Änderungen** und **ermittelt Autoren** [2, 3].
*   **Speichert Änderungen** im Git-Repository (mit Autoren) [2, 4].
*   **Erstellt Git TAGs** (mit Autoren) [4, 5].

## Vorteile

*   **Automatische, regelmässige Sicherung** [6].
*   **Nachverfolgbarkeit von Änderungen** und Autoren [2, 3].
*   **Verbesserte Konsistenz** und Wiederholbarkeit [7, 8].
*   **Schnelle Wiederherstellung** im Notfall [1, 9].
*   **Sichere Speicherung** in Azure Repos [9, 10].

## Voraussetzungen

*   **Azure DevOps** mit **Azure Repos** [9, 11].
*   Eine **Azure DevOps Pipeline** [11].
*   Eine **Azure AD Workloadidentität (Service Principal)** mit erforderlichen **Graph API Berechtigungen** [11, 12].

## Einrichtung (Kurz)

1.  **Azure DevOps Pipeline erstellen** und YAML-Datei auswählen [10, 13].
2.  **Pipeline-Variablen konfigurieren** (`BACKUP_FOLDER`, `TENANT_NAME`, `SERVICE_CONNECTION_NAME`, `USER_EMAIL`, `USER_NAME`) [14].
3.  **Service Connection mit Graph API Berechtigungen** sicherstellen [12, 15].
4.  **Zeitplan (Trigger)** für die Pipeline festlegen [14].

## Wiederherstellung (Kurz)

1.  **IntuneCD-Updatefunktion:** Ordner erstellen, Dateien kopieren, `IntuneCD-startupdate` ausführen [16].
2.  **Manuelle Konfiguration:** JSON-Dateien im Backup verwenden [16].

## Wichtige Hinweise zur Autorenerkennung

Die Autorenerkennung ist **nicht immer 100% genau** (z.B. bei Änderungen kurz nach Backup-Start oder fehlenden Einträgen im Auditprotokoll) [4, 17, 18]. Für höchste Genauigkeit `Get-MgDeviceManagementAuditEvent` verwenden [4].

## Anpassen der Datensicherung

Zu sichernde Konfigurationen über den Parameter `--exclude` in der Aufgabe "Create Intune backup" anpassen [19-21].
