# Intune Configuration as Code Automation

[![Azure DevOps Build Status](https://img.shields.io/azure-devops/build/your-organization/your-project/your-pipeline-name?style=flat-square)](https://dev.azure.com/your-organization/your-project/_build?definitionId=your-pipeline-id)
[![GitHub Release](https://img.shields.io/github/v/release/your-username/your-repository?style=flat-square)](https://github.com/your-username/your-repository/releases)

**Dieses Projekt automatisiert die Sicherung und Verwaltung Ihrer Microsoft Intune Konfigurationen mithilfe von Configuration as Code (CaC)-Prinzipien, IntuneCD und Azure DevOps Pipelines.** [1-3]

## Ausgangslage und Ziel [4-6]

Das Fehlen einer automatisierten Datensicherungslösung für Intune birgt im Falle eines Cyberangriffs oder eines Rechenzentrumsausfalls ein erhebliches Risiko für den Verlust aller Konfigurationseinstellungen des digitalen Arbeitsplatzes [6]. **Dieses Projekt implementiert eine umfassende Datensicherungslösung, um dieses Risiko zu minimieren.** [6]

## Funktionsweise [7, 8]

Die **Azure Pipeline** in diesem Projekt führt folgende Hauptaufgaben aus:

*   **Installiert das IntuneCD Tool.** [7]
*   **Exportiert die aktuelle Intune-Konfiguration in den Ordner `prod-backup`.** [7, 9]
*   **Überprüft, ob Änderungen in der Konfiguration vorhanden sind.** [7]
*   **Ermittelt für jede geänderte Datei den/die Autor(en) anhand des Intune-Auditprotokolls seit dem letzten Backup-Commit.** [7, 10]
    *   Die Ressourcen-ID wird aus dem Dateinamen extrahiert und im Auditprotokoll gesucht. [10]
    *   Änderungen durch Benutzer und Service Principals werden berücksichtigt. [11]
*   **Gruppiert die geänderten Dateien nach den jeweiligen Autoren.** [7]
*   **Erstellt für jede Gruppe separate Commits, wobei der Name des/der Autors/Autoren als Commit-Autor verwendet wird.** [7, 12]
*   **Erstellt optional eine Intune-Konfigurationsdokumentationsdatei im Markdown-Format.** [8]
*   **Erstellt einen Git TAG.** [8]
    *   Die Namen aller Änderungsautoren werden in die TAG-Beschreibung aufgenommen. [8, 12]
*   **Generiert optional HTML- und PDF-Dateien aus der Markdown-Dokumentation und speichert sie als Pipeline-Artefakte.** [8]
*   **Beendet die Pipeline, falls keine Änderungen vorhanden sind.** [13]
*   **Sendet eine Benachrichtigung bei Pipeline-Fehlern.** [13]

graph TD
    A[Intune Konfiguration] -->|Änderung erkannt| B(Azure Pipeline);
    B --> C{IntuneCD installieren};
    C --> D{Konfiguration exportieren nach prod-backup};
    D --> E{Änderungen vorhanden?};
    E -- Ja --> F{Autor ermitteln};
    F --> G{Dateien nach Autor gruppieren};
    G --> H{Commits erstellen (Autor als Autor)};
    H --> I{Git TAG erstellen (mit Autoren)};
    I --> J{Dokumentation erstellen (optional)};
    J --> K{Artefakte speichern (optional)};
    K --> L[Pipeline beendet];
    E -- Nein --> M[Pipeline beendet];
    B --> N{Benachrichtigung bei Fehler};
    
Vorteile
•
Verbesserte Konsistenz und Wiederholbarkeit: Intune-Einstellungen werden konsistent angewendet.
•
Vereinfachte Zusammenarbeit: Konfigurationen können einfach geteilt und verwaltet werden.
•
Verbesserte Sichtbarkeit und Kontrolle: Änderungen an der Umgebung sind nachvollziehbar.
•
Automatisierte Bereitstellung: Schnelle und effiziente Aktualisierung von Richtlinien.
•
Reduzierte Fehlerquote: Minimierung menschlicher Fehler durch Automatisierung.
•
Schnellere Reaktion auf sich ändernde Anforderungen: Einfache Implementierung neuer Richtlinien.
•
Sichere Speicherung der Sicherungen: In Azure Repository.
•
Einfache Wiederherstellung im Notfall.
Voraussetzungen
•
Azure DevOps:
◦
Ein Projekt mit aktiviertem Azure Repos.
•
Azure DevOps Pipeline:
•
Azure Active Directory (Azure AD) Workloadidentität (Service Principal):
◦
Mit den erforderlichen Graph API Anwendungsberechtigungen:
▪
DeviceManagementApps.Read.All
▪
DeviceManagementConfiguration.ReadWrite.All
▪
DeviceManagementManagedDevices.Read.All
▪
DeviceManagementServiceConfig.Read.All
▪
Group.Read.All
▪
Policy.Read.All
▪
Policy.Read.ConditionalAccess
▪
DeviceManagementRBAC.Read.All
Einrichtung
1.
Erstellen Sie eine neue Azure DevOps Pipeline.
2.
Wählen Sie als Quelle Ihr Repository aus.
3.
Wählen Sie die YAML-Vorlage aus diesem Repository.
4.
Konfigurieren Sie die Pipeline-Variablen:
◦
BACKUP_FOLDER: Der Name des Ordners für die Sicherung (Standard: prod-backup).
◦
TENANT_NAME: Die ID Ihres Intune Tenants.
◦
SERVICE_CONNECTION_NAME: Der Name Ihrer Azure Service Connection zu Ihrem Azure-Abonnement.
◦
USER_EMAIL: Die E-Mail-Adresse für Git-Operationen innerhalb der Pipeline.
◦
USER_NAME: Der Name für Git-Operationen innerhalb der Pipeline.
5.
Stellen Sie sicher, dass Ihre Azure Service Connection über die oben genannten Graph API Berechtigungen verfügt.
6.
Konfigurieren Sie den Trigger für die Pipeline:
◦
Ein Zeitplan (z.B. täglich um 1 Uhr).
◦
Standardmässig wird die Pipeline nur für Änderungen im main-Branch ausgeführt.
7.
Passen Sie optional die Liste der ausgeschlossenen Konfigurationselemente im Schritt "Create Intune backup" an, indem Sie den Parameter --exclude im Skript modifizieren. Standardmässig ausgeschlossen sind CompliancePartnerHeartbeat, ManagedGooglePlay und VPPusedLicenseCount.
# Beispielausschnitt aus der Azure DevOps Pipeline YAML-Datei
variables:
  - name: BACKUP_FOLDER
    value: prod-backup
  - name: TENANT_NAME
    value: your-intune-tenant-id
  - name: SERVICE_CONNECTION_NAME
    value: your-azure-service-connection-name
  - name: USER_EMAIL
    value: pipeline-user@example.com
  - name: USER_NAME
    value: Pipeline User

schedules:
- cron: "0 1 * * *"
  displayName: "Tägliche Sicherung um 1 Uhr"
  branches:
    include:
    - main
  always: true

steps:
- task: Bash@3
  displayName: Create Intune backup
  inputs:
    targetType: 'inline'
    script: |
      mkdir -p "$(Build.SourcesDirectory)/$(BACKUP_FOLDER)"

      BACKUP_START=`date +%Y.%m.%d:%H.%M.%S`
      echo "##vso[task.setVariable variable=BACKUP_START]$BACKUP_START"

      IntuneCD-startbackup \
          -t $(accessToken) \
          --mode=1 \
          --output=json \
          --path="$(Build.SourcesDirectory)/$(BACKUP_FOLDER)" \
          --exclude CompliancePartnerHeartbeat ManagedGooglePlay VPPusedLicenseCount \
          --append-id \
          --ignore-omasettings
    workingDirectory: '$(Build.SourcesDirectory)'
    failOnStderr: true
Nutzung
Nach der erfolgreichen Einrichtung wird die Azure Pipeline gemäss Ihrem konfigurierten Zeitplan automatisch ausgeführt.
•
Bei Änderungen in Ihrer Intune-Konfiguration:
◦
Die Pipeline exportiert die aktuellen Einstellungen.
◦
Ermittelt die Autoren der Änderungen seit dem letzten Backup.
◦
Erstellt separate Git-Commits pro Autor mit den entsprechenden Änderungen.
◦
Erstellt einen neuen Git TAG, der das Datum und die Autoren der Änderungen enthält.
◦
Optional wird eine Dokumentation der Konfiguration erstellt und als Artefakt gespeichert.
•
Ohne Änderungen: Die Pipeline wird beendet, ohne Commits oder Tags zu erstellen.
Sie können die Historie Ihrer Intune-Konfigurationen im Git-Repository einsehen. Jeder Commit repräsentiert eine Änderung, und der Commit-Autor gibt an, wer diese Änderung vorgenommen hat (sofern im Auditprotokoll gefunden).
Wiederherstellungsprozess
Im Falle einer notwendigen Wiederherstellung haben Sie zwei grundlegende Möglichkeiten:
1.
Verwendung der IntuneCD-Updatefunktion:
◦
Erstellen Sie einen neuen Ordner.
◦
Kopieren Sie nur die Ordnerstruktur mit den JSON-Dateien, die Sie wiederherstellen möchten, in diesen Ordner.
◦
Führen Sie IntuneCD-startupdate mit dem Argument --path "PfadZumBackupOrdner" aus.
2.
Manuelles Erstellen der gewünschten Intune-Konfigurationen:
◦
Füllen Sie die Werte in den JSON-Dateien Ihres Backups manuell in Ihrer Intune-Umgebung aus.
Der Wiederherstellungsprozess mithilfe der Azure DevOps Pipeline umfasst das Wiederherstellen der exportierten Daten aus Azure Repo und das anschliessende Importieren in Intune mithilfe von PowerShell-Cmdlets.
Wichtig: Die Wiederherstellung einer früheren Intune-Instanz muss gemäss der aktuellen Richtlinien genehmigt werden. Die Umsetzung erfolgt durch einen zuständigen Mitarbeiter.
Ungenauigkeiten bei der Ermittlung des Änderungsautors
Bitte beachten Sie, dass die Ermittlung des Änderungsautors nicht immer 100% genau ist.
•
Änderungen, die kurz nach dem Start des Backups, aber vor der Protokollabfrage erfolgen, werden möglicherweise nicht dem korrekten Autor zugeordnet.
•
Die Liste der Commit-Autoren zeigt, wer Änderungen an den Dateien vorgenommen hat, aber nicht die spezifischen Änderungen jeder Person.
•
Änderungen, die von Anwendungen vorgenommen wurden, werden mit dem Namen der Anwendung und dem Suffix "(SP)" gekennzeichnet.
•
Bestimmte Intune-Konfigurationsänderungen werden nicht im Intune-Auditprotokoll erfasst (z.B. Änderungen am ESP-Profil). In solchen Fällen wird "unbekannt" als Autor verwendet.
•
Intune verwendet verschiedene ID-Formate, was die Identifizierung im Auditprotokoll komplex machen kann.
Für absolute Genauigkeit (z.B. bei Sicherheitsvorfällen) wird die Verwendung des Befehls Get-MgDeviceManagementAuditEvent empfohlen.
Ändern der Datensicherungselemente
Um festzulegen, welche Intune-Konfigurationen gesichert werden sollen, bearbeiten Sie den Abschnitt "Create Intune backup task" in der Pipeline-Konfiguration. Suchen Sie nach der Aufgabe "Create Intune backup" und ändern Sie deren Skript, indem Sie den Parameter --exclude anpassen, um bestimmte Elemente auszuschliessen oder zu entfernen.
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
          --exclude CompliancePartnerHeartbeat ManagedGooglePlay VPPusedLicenseCount DeviceConfigurations # Zusätzlicher Ausschluss
          --append-id \
          --ignore-omasettings
    workingDirectory: '$(Build.SourcesDirectory)'
    failOnStderr: true
DevOps Pipelines
Die Verantwortung für die Verwaltung und Überwachung der DevOps-Pipelines liegt beim Workplace-Team. Benachrichtigungen über den Erfolg oder Misserfolg der Pipeline können bei Bedarf an weitere Teammitglieder erweitert werden.
Fazit
Dieses Projekt bietet eine automatisierte und zuverlässige Lösung für die Sicherung und Verwaltung Ihrer Microsoft Intune Konfigurationen als Code. Durch die Verwendung von IntuneCD und Azure DevOps Pipelines profitieren Sie von verbesserter Konsistenz, Nachvollziehbarkeit und der Möglichkeit einer schnellen Wiederherstellung im Notfall
