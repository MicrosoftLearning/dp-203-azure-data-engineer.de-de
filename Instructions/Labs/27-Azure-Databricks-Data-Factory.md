---
lab:
  title: Automatisieren eines Azure Databricks-Notebooks mit Azure Data Factory
  ilt-use: Suggested demo
---

# Automatisieren eines Azure Databricks-Notebooks mit Azure Data Factory

Sie können Notebooks in Azure Databricks verwenden, um Datentechnikaufgaben auszuführen, wie etwa das Verarbeiten von Datendateien und das Laden von Daten in Tabellen. Wenn Sie diese Aufgaben als Teil einer Datentechnikpipeline orchestrieren müssen, können Sie Azure Data Factory verwenden.

Diese Übung dauert ca. **40** Minuten.

## Vorbereitung

Sie benötigen ein [Azure-Abonnement](https://azure.microsoft.com/free), in dem Sie Administratorzugriff besitzen.

## Bereitstellen von Azure-Ressourcen

In dieser Übung verwenden Sie ein Skript, um einen neuen Azure Databricks-Arbeitsbereich und eine Azure Data Factory-Ressource in Ihrem Azure-Abonnement bereitzustellen.

1. Melden Sie sich in einem Webbrowser beim [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com` an.
2. Verwenden Sie rechts neben der Suchleiste oben auf der Seite die Schaltfläche **[\>_]**, um eine neue Cloud Shell-Instanz im Azure-Portal zu erstellen. Wählen Sie eine ***PowerShell***-Umgebung aus, und erstellen Sie Speicher, falls Sie dazu aufgefordert werden. Die Cloud Shell bietet eine Befehlszeilenschnittstelle in einem Bereich am unteren Rand des Azure-Portals, wie hier gezeigt:

    ![Azure-Portal mit einem Cloud Shell-Bereich](./images/cloud-shell.png)

    > **Hinweis**: Wenn Sie zuvor eine Cloud Shell erstellt haben, die eine *Bash*-Umgebung verwendet, ändern Sie diese mithilfe des Dropdownmenüs oben links im Cloud Shell-Bereich zu ***PowerShell***.

3. Beachten Sie, dass Sie die Größe der Cloud Shell durch Ziehen der Trennzeichenleiste oben im Bereich ändern können oder den Bereich mithilfe der Symbole **&#8212;**, **&#9723;** und **X** oben rechts minimieren, maximieren und schließen können. Weitere Informationen zur Verwendung von Azure Cloud Shell finden Sie in der [Azure Cloud Shell-Dokumentation](https://docs.microsoft.com/azure/cloud-shell/overview).

4. Geben Sie im PowerShell-Bereich die folgenden Befehle ein, um dieses Repository zu klonen:

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Nachdem das Repository geklont wurde, geben Sie die folgenden Befehle ein, um in den Ordner für dieses Lab zu wechseln. Führen Sie das darin enthaltene Skript **setup.ps1** aus:

    ```
    cd dp-203/Allfiles/labs/27
    ./setup.ps1
    ```

6. Wenn Sie dazu aufgefordert werden, wählen Sie aus, welches Abonnement Sie verwenden möchten (dies geschieht nur, wenn Sie Zugriff auf mehrere Azure-Abonnements haben).

7. Warten Sie, bis das Skript abgeschlossen ist. Dies dauert in der Regel etwa 5 Minuten, in einigen Fällen kann es jedoch länger dauern. Während Sie warten, lesen Sie den Artikel [Was ist Azure Data Factory?](https://docs.microsoft.com/azure/data-factory/introduction)
8. Wenn das Skript abgeschlossen ist, schließen Sie den Cloud Shell-Bereich, und navigieren Sie zur Ressourcengruppe **dp203-*xxxxxxx***, die vom Skript erstellt wurde, um zu überprüfen, ob sie einen Azure Databricks-Arbeitsbereich und eine Azure Data Factory(V2)-Ressource enthält (Möglicherweise müssen Sie die Ressourcengruppenansicht aktualisieren).

## Importieren eines Notebooks

Sie können Notizbücher in Ihrem Azure Databricks-Arbeitsbereich erstellen, um Code auszuführen, der in einer Reihe von Programmiersprachen geschrieben wurde. In dieser Übung importieren Sie ein vorhandenes Notebook, das Python-Code enthält.

1. Navigieren Sie im Azure-Portal zur Ressourcengruppe **dp203-*xxxxxxx***, die vom Skript erstellt wurde, das Sie ausgeführt haben.
2. Wählen Sie die Azure Databricks Service-Ressource **databricks*xxxxxxx*** aus.
3. Verwenden Sie auf der Seite **Übersicht** für **databricks*xxxxxxx*** die Schaltfläche **Arbeitsbereich starten**, um Ihren Azure Databricks-Arbeitsbereich in einer neuen Browserregisterkarte zu öffnen. Melden Sie sich an, wenn Sie dazu aufgefordert werden.
4. Wenn die Nachricht **Was ist Ihr aktuelles Datenprojekt?** angezeigt wird, wählen Sie **Fertigstellen** aus, um es zu schließen. Zeigen Sie dann das Azure Databricks-Arbeitsbereichsportal an, und beachten Sie, dass die Seitenleiste auf der linken Seite Symbole für die verschiedenen Aufgaben enthält, die Sie ausführen können.

    >**Tipp**: Während Sie das Databricks-Arbeitsbereichsportal verwenden, werden möglicherweise verschiedene Tipps und Benachrichtigungen angezeigt. Schließen Sie diese, und folgen Sie den Anweisungen, um die Aufgaben in dieser Übung auszuführen.

1. Wählen Sie in der Seitenleiste auf der linken Seite **Arbeitsbereiche** aus. Wählen Sie dann den Ordner **&#8962; Basis** aus.
1. Wählen Sie oben auf der Seite im Menü **&#8942;** neben Ihrem Benutzernamen die Option **Importieren** aus. Wählen Sie dann im Dialogfeld **Importieren** die Option **URL** aus, und importieren Sie das Notebook aus `https://github.com/MicrosoftLearning/dp-203-azure-data-engineer/raw/master/Allfiles/labs/27/Process-Data.ipynb`.
1. Überprüfen Sie den Inhalt des Notebooks, in dem einige Python-Codezellen enthalten sind:
    - Rufen Sie einen Parameter mit dem Namen **folder** ab, wenn er übergeben wurde (andernfalls verwenden Sie den Standardwert *data*).
    - Laden Sie Daten von GitHub herunter, und speichern Sie sie im angegebenen Ordner im Databricks File System (DBFS).
    - Beenden Sie das Notebook, und geben Sie den Pfad zurück, in dem die Daten als Ausgabe gespeichert wurden.

    > **Tipp**: Das Notebook kann praktisch jede benötigte Datenverarbeitungslogik enthalten. Dies ist ein einfaches Beispiel, das das Prinzip veranschaulicht.

## Aktivieren der Azure Databricks-Integration in Azure Data Factory

Um Azure Databricks aus einer Azure Data Factory-Pipeline zu verwenden, müssen Sie einen verknüpften Dienst in Azure Data Factory erstellen, der den Zugriff auf Ihren Azure Databricks-Arbeitsbereich ermöglicht.

### Erstellen eines Zugriffstokens

1. Wählen Sie im Azure Databricks-Portal in der oberen rechten Menüleiste den Benutzernamen aus, und wählen Sie dann in der Dropdownliste **Benutzereinstellungen** aus.
1. Wählen Sie auf der Seite **Benutzereinstellungen** die Option **Entwickler** aus. Wählen Sie dann neben **Zugriffstoken** die Option **Verwalten** aus.
1. Wählen Sie **Neues Token generieren** aus, und generieren Sie ein neues Token mit dem Kommentar *Data Factory* und einer leeren Lebensdauer (damit das Token nicht abläuft). Achten Sie darauf, das Token zu **kopieren, wenn es angezeigt wird, <u>bevor</u> Sie *Fertig*** auswählen.
1. Fügen Sie das kopierte Token in eine Textdatei ein, damit Sie es später in dieser Übung zur Hand haben.

### Erstellen eines verknüpften Diensts in Azure Data Factory

1. Kehren Sie zum Azure-Portal zurück, und wählen Sie in der Ressourcengruppe **dp203-*xxxxxxx*** die Azure Data Factory-Ressource **adf*xxxxxxx*** aus.
2. Wählen Sie auf der Seite **Übersicht** die Option **Studio starten** aus, um das Azure Data Factory Studio zu öffnen. Melden Sie sich an, wenn Sie dazu aufgefordert werden.
3. Verwenden Sie in Azure Data Factory Studio das Symbol **>>**, um den Navigationsbereich auf der linken Seite zu erweitern. Wählen Sie dann die Seite **Verwalten** aus.
4. Wählen Sie auf der Seite **Verwalten** auf der Registerkarte **Verknüpfte Dienste** die Option **+ Neu** aus, um einen neuen verknüpften Dienst hinzuzufügen.
5. Wählen Sie im Bereich **Neuer verknüpfter Dienst** die Registerkarte **Compute** oben aus. Wählen Sie dann **Azure Databricks** aus.
6. Fahren Sie fort, und erstellen Sie den verknüpften Dienst mit den folgenden Einstellungen:
    - **Name**: AzureDatabricks
    - **Beschreibung**: Azure Databricks-Arbeitsbereich
    - **Verbindung über Integration Runtime herstellen**: AutoResolveInegrationRuntime
    - **Kontoauswahlmethode**: Aus Azure-Abonnement
    - **Azure-Abonnement**: *Wählen Sie Ihr Abonnement aus.*
    - **Databricks-Arbeitsbereich**: *Wählen Sie Ihren Arbeitsbereich **databricksxxxxxxx** aus.*
    - **Cluster auswählen**: Neuer Auftragscluster
    - **Databrick-Arbeitsbereichs-URL**: *Automatisch auf Ihre Databricks-Arbeitsbereichs-URL festgelegt*
    - **Authentifizierungstyp**: Zugriffstoken
    - **Zugriffstoken**: *Fügen Sie Ihr Zugriffstoken ein*
    - **Clusterversion**: 12.2 LTS (Scala 2.12, Spark 3.2.2)
    - **Clusterknotentyp**: Standard_DS3_v2
    - **Python-Version**: 3
    - **Workeroptionen**: Behoben
    - **Worker**: 1

## Verwenden einer Pipeline zum Ausführen des Databricks-Notebooks

Nachdem Sie nun einen verknüpften Dienst erstellt haben, können Sie ihn in einer Pipeline verwenden, um das zuvor angezeigte Notebook auszuführen.

### Erstellen einer Pipeline

1. Wählen Sie in Azure Data Factory Studio im Navigationsbereich die Option **Autor** aus.
2. Verwenden Sie auf der Seite **Autor** im Bereich **Factory-Ressourcen** das Symbol **+**, um eine **Pipeline** hinzuzufügen.
3. Ändern Sie im Bereich **Eigenschaften** für die neue Pipeline ihren Namen in **Daten mit Databricks verarbeiten**. Verwenden Sie dann die Schaltfläche **Eigenschaften** (die **&#128463;<sub>*</sub>** ähnelt) am rechten Ende der Symbolleiste verwenden, um den Bereich **Eigenschaften** auszublenden.
4. Erweitern Sie im Bereich **Aktivitäten** die Option **Databricks**, und ziehen Sie eine **Notebook**-Aktivität auf die Oberfläche des Pipeline-Designers, wie hier gezeigt:
5. Wenn die neue **Notebook1**-Aktivität ausgewählt ist, legen Sie die folgenden Eigenschaften im unteren Bereich fest:
    - **Allgemein**:
        - **Name**: Daten verarbeiten
    - **Azure Databricks**:
        - **Mit Databricks verknüpfter Dienst**: *Wählen Sie den verknüpften Dienst **AzureDatabricks** aus, den Sie zuvor erstellt haben.*
    - **Einstellungen**:
        - **Notebookpfad**: *Navigieren Sie zum Ordner **Users/Ihr_Benutzername**, und wählen Sie das Notebook **Daten verarbeiten** aus.*
        - **Basisparameter**: *Fügen Sie einen neuen Parameter namens **folder** mit dem Wert **product_data** hinzu.*
6. Verwenden Sie die Schaltfläche **Validieren** oberhalb der Pipeline-Designeroberfläche, um die Pipeline zu validieren. Verwenden Sie dann die Schaltfläche **Alle veröffentlichen**, um sie zu veröffentlichen (speichern).

### Führen Sie die Pipeline aus.

1. Wählen Sie oberhalb der Oberfläche des Pipeline-Designers die Option **Trigger hinzufügen** und dann **Jetzt auslösen** aus.
2. Wählen Sie im Bereich **Pipelineausführung** die Option **OK** aus, um die Pipeline auszuführen.
3. Wählen Sie im Navigationsbereich auf der linken Seite **Überwachen** aus, und beobachten Sie die Pipeline **Daten mit Databricks verarbeiten** auf der Registerkarte **Pipelineausführungen**. Die Ausführung kann eine Weile dauern, da die Pipeline dynamisch einen Spark-Cluster erstellt und das Notebook ausführt. Sie können die Schaltfläche **&#8635; Aktualisieren** auf der Seite **Pipelineausführungen** verwenden, um den Status zu aktualisieren.

    > **Hinweis**: Wenn Ihre Pipeline nicht ausgeführt werden kann, verfügt Ihr Abonnement möglicherweise über ein unzureichendes Kontingent in der Region, in der Ihr Azure Databricks-Arbeitsbereich bereitgestellt wird, um einen Auftragscluster zu erstellen. Details finden Sie unter [Der Grenzwert für CPU-Kerne verhindert die Clustererstellung](https://docs.microsoft.com/azure/databricks/kb/clusters/azure-core-limit). In diesem Fall können Sie versuchen, Ihren Arbeitsbereich zu löschen und in einer anderen Region einen neuen zu erstellen. Sie können einen Bereich als Parameter für das Setupskript wie folgt angeben: `./setup.ps1 eastus`

4. Wenn die Ausführung erfolgreich ist, wählen Sie ihren Namen aus, um die Ausführungsdetails anzuzeigen. Wählen Sie dann auf der Seite **Daten mit Databricks verarbeiten** im Abschnitt **Aktivitätsausführungen** die Aktivität **Daten verarbeiten** aus, und verwenden Sie das zugehörige Symbol ***Ausgabe***, um den JSON-Ausgabe-Code aus der Aktivität anzuzeigen. Dieser sollte wie folgt aussehen:
    ```json
    {
        "runPageUrl": "https://adb-..../run/...",
        "runOutput": "dbfs:/product_data/products.csv",
        "effectiveIntegrationRuntime": "AutoResolveIntegrationRuntime (East US)",
        "executionDuration": 61,
        "durationInQueue": {
            "integrationRuntimeQueue": 0
        },
        "billingReference": {
            "activityType": "ExternalActivity",
            "billableDuration": [
                {
                    "meterType": "AzureIR",
                    "duration": 0.03333333333333333,
                    "unit": "Hours"
                }
            ]
        }
    }
    ```

5. Notieren Sie sich den Wert **runOutput**, bei dem es sich um die Variable *path* handelt, in der das Notebook die Daten gespeichert hat.

## Löschen von Databricks-Ressourcen

Nachdem Sie sich mit der Azure Data Factory-Integration in Azure Databricks vertraut gemacht haben, müssen Sie die von Ihnen erstellten Ressourcen löschen, um unnötige Azure-Kosten zu vermeiden und Kapazität in Ihrem Abonnement freizugeben.

1. Schließen Sie den Azure Databricks-Arbeitsbereich und die Registerkarte mit Azure Data Factory Studio, und kehren Sie zum Azure-Portal zurück.
2. Wählen Sie auf der **Startseite** des Azure-Portals die Option **Ressourcengruppen** aus.
3. Wählen Sie die Ressourcengruppe **dp203-*xxxxxxx*** aus, die Ihren Azure Databricks- und Azure Data Factory-Arbeitsbereich (nicht die verwaltete Ressourcengruppe) enthält.
4. Wählen Sie oben auf der Seite **Übersicht** für Ihre Ressourcengruppe die Option **Ressourcengruppe löschen** aus.
5. Geben Sie den Namen der Ressourcengruppe ein, um zu bestätigen, dass Sie sie löschen möchten, und wählen Sie **Löschen** aus.

    Nach einigen Minuten werden Ihre Ressourcengruppe und die damit verknüpfte Ressourcengruppe im verwalteten Arbeitsbereich gelöscht.
