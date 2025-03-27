# Intune Configuration as Code Automation

[![Azure DevOps Build Status](https://img.shields.io/azure-devops/build/your-organization/your-project/your-pipeline-name?style=flat-square)](https://dev.azure.com/your-organization/your-project/_build?definitionId=your-pipeline-id)
[![GitHub Release](https://img.shields.io/github/v/release/your-username/your-repository?style=flat-square)](https://github.com/your-username/your-repository/releases)

Dieses Projekt automatisiert die **Sicherung und Verwaltung Ihrer Microsoft Intune Konfigurationen** mithilfe von **Configuration as Code (CaC)**, **IntuneCD** und **Azure DevOps Pipelines** [1-4].

## Ziel

Dieses Projekt implementiert eine **automatische Datensicherungslösung für Intune**, um Datenverlust im Falle von Cyberangriffen oder Ausfällen zu verhindern [5, 6].

## Funktionsweise

Die **Azure Pipeline** führt folgende Schritte aus [7, 8]:

*   **Exportiert Ihre aktuelle Intune-Konfiguration** in einen Backup-Ordner [7].
*   **Erkennt Änderungen** in den Konfigurationsdateien [7, 8].
*   **Ermittelt die Autoren** dieser Änderungen anhand des Intune-Auditprotokolls [7, 8].
*   **Speichert jede Änderung separat** in Ihrem Git-Repository, wobei der Autor der Änderung als Commit-Autor verwendet wird [7, 9].
*   **Erstellt einen Git TAG** mit dem Datum und den Autoren der Änderungen [9, 10].

graph TD
    A[Intune Konfiguration] -->|Änderung erkannt| B(Azure Pipeline);
    B --> C{Konfiguration exportieren};
    C --> D{Änderungen vorhanden?};
    D -- Ja --> E{Autor ermitteln};
    E --> F{Änderungen speichern (mit Autor)};
    F --> G{Git TAG erstellen (mit Autoren)};
    D -- Nein --> H[Pipeline beendet];
    B --> I{Benachrichtigung bei Fehler};
Vorteile
•
Automatische und regelmässige Sicherung Ihrer Intune-Konfiguration.
•
Nachverfolgbarkeit von Änderungen und deren Autoren.
•
Verbesserte Konsistenz und Wiederholbarkeit Ihrer Intune-Einstellungen.
•
Schnellere Wiederherstellung Ihrer Intune-Umgebung im Notfall.
•
Sichere Speicherung der Sicherungsdaten in Azure Repos.
Voraussetzungen
•
Azure DevOps mit aktiviertem Azure Repos.
•
Eine Azure DevOps Pipeline.
•
Eine Azure Active Directory (Azure AD) Workloadidentität (Service Principal) mit folgenden Microsoft Graph API Anwendungsberechtigungen:
◦
DeviceManagementApps.Read.All
◦
DeviceManagementConfiguration.ReadWrite.All
◦
DeviceManagementManagedDevices.Read.All
◦
DeviceManagementServiceConfig.Read.All
◦
Group.Read.All
◦
Policy.Read.All
◦
Policy.Read.ConditionalAccess
◦
DeviceManagementRBAC.Read.All
Einrichtung
1.
Erstellen Sie eine neue Azure DevOps Pipeline und wählen Sie Ihr Repository als Quelle.
2.
Wählen Sie die YAML-Datei aus diesem Repository als Vorlage.
3.
Konfigurieren Sie die Pipeline-Variablen unter "Variables" in Ihrer Pipeline:
◦
BACKUP_FOLDER: Name des Ordners für die Sicherung (z.B. prod-backup).
◦
TENANT_NAME: Ihre Intune Tenant-ID.
◦
SERVICE_CONNECTION_NAME: Name Ihrer Azure Service Connection mit den erforderlichen Berechtigungen.
◦
USER_EMAIL und USER_NAME: Für Git-Operationen innerhalb der Pipeline.
4.
Stellen Sie sicher, dass Ihre Azure Service Connection die oben genannten Graph API Berechtigungen besitzt.
5.
Konfigurieren Sie einen Zeitplan (Trigger) für die Pipeline, z.B. täglich um 1 Uhr.
Nutzung
Nach der Einrichtung wird die Pipeline automatisch ausgeführt. Änderungen an Ihrer Intune-Konfiguration werden erkannt, gespeichert und in Ihrem Git-Repository nachvollziehbar gemacht.
Wiederherstellung
Im Falle einer Wiederherstellung haben Sie zwei Optionen:
1.
Verwendung der IntuneCD-Updatefunktion: Erstellen Sie einen neuen Ordner, kopieren Sie die gewünschten Backup-Dateien und führen Sie IntuneCD-startupdate --path "PfadZumBackupOrdner" aus.
2.
Manuelles Erstellen der Konfigurationen: Verwenden Sie die JSON-Dateien im Backup, um die Konfigurationen manuell in Intune wiederherzustellen.
Wichtige Hinweise zur Autorenerkennung
Die Ermittlung des Änderungsautors ist nicht immer 100% genau. Gründe dafür sind:
•
Zeitliche Verzögerungen zwischen Änderung und Backup-Lauf.
•
Das Auditprotokoll erfasst nicht alle Änderungen.
•
Änderungen durch Anwendungen werden als "(SP)" gekennzeichnet.
•
Unterschiedliche ID-Formate in Intune können die Zuordnung erschweren.
Für höchste Genauigkeit verwenden Sie bei Bedarf den Befehl Get-MgDeviceManagementAuditEvent.
Anpassen der Datensicherung
Sie können festlegen, welche Intune-Konfigurationen gesichert werden sollen, indem Sie den Parameter --exclude im Skript der Aufgabe "Create Intune backup" in der Pipeline-Konfiguration bearbeiten.
# Beispielausschnitt zum Ändern der ausgeschlossenen Elemente
- task: Bash@3
  displayName: Create Intune backup
  inputs:
    targetType: 'inline'
    script: |
      # ... vorheriger Code ...
      IntuneCD-startbackup \
          -t $(accessToken) \
          --mode=1 \
          --output=json \
          --path="$(Build.SourcesDirectory)/$(BACKUP_FOLDER)" \
          --exclude CompliancePartnerHeartbeat ManagedGooglePlay DeviceConfigurations # Beispiel für zusätzlichen Ausschluss
          --append-id \
          --ignore-omasettings
    workingDirectory: '$(Build.SourcesDirectory)'
    failOnStderr: true
Verantwortlichkeiten
Das Workplace-Team ist verantwortlich für die Verwaltung und Überwachung der DevOps Pipelines.
Fazit
Dieses Projekt bietet eine automatisierte und verbesserte Möglichkeit, Ihre Microsoft Intune Konfigurationen zu sichern und zu verwalten.
