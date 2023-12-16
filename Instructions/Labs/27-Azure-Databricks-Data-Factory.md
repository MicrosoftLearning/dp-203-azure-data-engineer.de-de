---
lab:
  title: Automatisieren von Azure Databricks-Notebooks mit Azure Data Factory
  ilt-use: Suggested demo
---

# Automatisieren von Azure Databricks-Notebooks mit Azure Data Factory

Sie können Notebooks in Azure Databricks verwenden, um Datentechnikaufgaben auszuführen, z. B. das Verarbeiten von Datendateien und das Laden von Daten in Tabellen. Wenn Sie diese Aufgaben als Teil einer Datentechnikpipeline koordinieren müssen, können Sie Azure Data Factory verwenden.

Diese Übung dauert ca. **40** Minuten.

## Vorbereitung

Sie benötigen ein [Azure-Abonnement](https://azure.microsoft.com/free), in dem Sie Administratorzugriff besitzen.

## Bereitstellen von Azure-Ressourcen

In dieser Übung verwenden Sie ein Skript, um einen neuen Azure Databricks-Arbeitsbereich und eine Azure Data Factory-Ressource in Ihrem Azure-Abonnement bereitzustellen.

> **Tipp**: Wenn Sie bereits über einen *Standard* - oder *Testarbeitsbereich* für Azure Databricks <u>sowie</u> eine eine Azure Data Factory v2-Ressource verfügen, können Sie dieses Verfahren überspringen.

1. Melden Sie sich in einem Webbrowser am [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com` an.
2. Verwenden Sie rechts neben der Suchleiste oben auf der Seite die Schaltfläche **[\>_]**, um eine neue Cloud Shell-Instanz im Azure-Portal zu erstellen. Wählen Sie eine ***PowerShell***-Umgebung aus, und erstellen Sie Speicher, wenn Sie dazu aufgefordert werden. Die Cloud Shell bietet eine Befehlszeilenschnittstelle in einem Bereich am unteren Rand des Azure-Portal, wie hier gezeigt:

    ![Azure-Portal mit einem Cloud Shell-Bereich](./images/cloud-shell.png)

    > **Hinweis**: Wenn Sie zuvor eine Cloud Shell erstellt haben, die eine *Bash*-Umgebung verwendet, verwenden Sie das Dropdownmenü oben links im Bereich der Cloud Shell, um sie in ***PowerShell*** zu ändern.

3. Beachten Sie, dass Sie die Größe der Cloud Shell durch Ziehen der Trennzeichenleiste oben im Bereich ändern können, oder den Bereich mithilfe der Symbole **&#8212;** , **&#9723;** und **X** oben rechts minimieren, maximieren und schließen können. Weitere Informationen zur Verwendung von Azure Cloud Shell finden Sie in der [Azure Cloud Shell-Dokumentation](https://docs.microsoft.com/azure/cloud-shell/overview).

4. Geben Sie im PowerShell Bereich die folgenden Befehle ein, um dieses Repository zu klonen:

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

7. Warten Sie, bis das Skript abgeschlossen ist. Dies dauert in der Regel etwa 5 Minuten, aber in einigen Fällen kann es länger dauern. Während Sie warten, lesen Sie den Artikel [Was ist Azure Data Factory?](https://docs.microsoft.com/azure/data-factory/introduction).
8. Wenn das Skript abgeschlossen ist, schließen Sie den Cloud Shell-Bereich, und navigieren Sie zur Ressourcengruppe **dp203-*xxxxxxx***, die vom Skript erstellt wurde, um zu überprüfen, ob es einen Azure Databricks-Arbeitsbereich und eine Azure Data Factory (V2)-Ressource enthält (Möglicherweise müssen Sie die Ressourcengruppenansicht aktualisieren).

## Importieren eines Notebooks

Sie können Notebooks in Ihrem Azure Databricks-Arbeitsbereich erstellen, um Code auszuführen, der in einer Reihe von Programmiersprachen geschrieben wurde. In dieser Übung importieren Sie ein vorhandenes Notebook, das Python-Code enthält.

1. Navigieren Sie im Azure-Portal zur Ressourcenruppe **dp203-*xxxxxxx***, die vom Skript erstellt wurde (oder die Ressourcengruppe, die Ihren vorhandenen Azure Databricks-Arbeitsbereich enthält).
1. Wählen Sie Ihre Azure Databricks-Dienstressource (mit dem Namen **Databricks*xxxxxxx*** aus, wenn Sie das Setupskript zum Erstellen verwendet haben).
1. Verwenden Sie auf der Seite **Übersicht** für Ihren Arbeitsbereich die Schaltfläche **Arbeitsbereich starten**, um Ihren Azure Databricks-Arbeitsbereich auf einer neuen Browserregisterkarte zu öffnen. Melden Sie sich an, wenn Sie dazu aufgefordert werden.

    > **Tipp**: Während Sie das Databricks-Arbeitsbereichsportal verwenden, werden möglicherweise verschiedene Tipps und Benachrichtigungen angezeigt. Schließen Sie diese, und folgen Sie den Anweisungen, um die Aufgaben in dieser Übung auszuführen.

1. Zeigen Sie das Azure Databricks-Arbeitsbereichsportal an, und beachten Sie, dass die Randleiste auf der linken Seite Symbole für die verschiedenen Aufgaben enthält, die Sie ausführen können.
1. Wählen Sie in der Randleiste auf der linken Seite **Arbeitsbereich** aus. Wählen Sie dann den Ordner **⌂ Start** aus.
1. Wählen Sie oben auf der Seite im Menü **⋮** neben Ihrem Benutzernamen **Importieren** aus. Wählen Sie dann im Dialogfeld **Importieren** den Eintrag **URL** aus, und importieren Sie das Notebook von `https://github.com/MicrosoftLearning/dp-203-azure-data-engineer/raw/master/Allfiles/labs/27/Process-Data.ipynb`
1. Überprüfen Sie den Inhalt des Notebooks, in dem Python-Codezellen enthalten sind:
    - Rufen Sie einen Parameter mit dem Namen **folder** ab, wenn er übergeben wurde (andernfalls verwenden Sie den Standardwert *data*).
    - Laden Sie Daten von GitHub herunter, und speichern Sie sie im angegebenen Ordner im Databricks File System (DBFS).
    - Verlassen Sie das Notebook, und geben Sie den Pfad zurück, in dem die Daten als Ausgabe gespeichert wurden.

    > **Tipp**: Das Notebook kann praktisch jede benötigte Datenverarbeitungslogik enthalten. Dies ist ein einfaches Beispiel, das das Hauptprinzip veranschaulicht.

## Aktivieren der Integration von Azure Databricks in Azure Data Factory

Um Azure Databricks aus einer Azure Data Factory-Pipeline zu verwenden, müssen Sie einen verknüpften Dienst in Azure Data Factory erstellen, der den Zugriff auf Ihren Azure Databricks-Arbeitsbereich ermöglicht.

### Erstellen von einem Zugriffstoken

1. Wählen Sie im Azure Databricks-Portal in der oberen rechten Menüleiste den Benutzernamen aus, und wählen Sie dann in der Dropdownliste **Benutzereinstellungen** aus.
1. Wählen Sie auf der Seite **Benutzereinstellungen** die Option **Entwickler** aus. Wählen Sie dann neben **Zugriffstoken** die Option **Verwalten** aus.
1. Wählen Sie **Neues Token genieren** aus, und generieren Sie ein neues Token mit dem Kommentar *Data Factory* und einer leeren Lebensdauer (sodass das Token nicht abläuft). Achten Sie darauf, **das angezeigte Token zu kopieren, <u>bevor</u> Sie *Done*** auswählen.
1. Fügen Sie das kopierte Token in eine Textdatei ein, damit Sie es später in dieser Übung zur Verfügung haben.

### Erstellen eines verknüpften Diensts in Azure Data Factory

1. Kehren Sie zum Azure-Portal zurück, und wählen Sie in der Ressourcengruppe **dp203-*xxxxx*** die Azure Data Factory-Ressource **adf*xxxxxxx*** aus.
2. Wählen Sie auf der Seite **Übersicht** die Option **Studio starten** aus, um das Azure Data Factory Studio zu öffnen. Melden Sie sich an, wenn Sie dazu aufgefordert werden.
3. Verwenden Sie in Azure Data Factory Studio das Symbol **>>**, um den Navigationsbereich auf der linken Seite zu erweitern. Wählen Sie dann die Seite **Verwalten** aus.
4. Wählen Sie auf der Seite **Verwalten** auf der Registerkarte **Verknüpfte Dienste** die Option **+ Neu** aus, um einen neuen verknüpften Dienst hinzuzufügen.
5. Wählen Sie im Bereich **Neuer verknüpfter Dienst** die Registerkarte **Compute** aus. Wählen Sie dann **Azure Databricks** aus.
6. Fahren Sie fort, und erstellen Sie den verknüpften Dienst mit den folgenden Einstellungen:
    - **Name**: AzureDatabricks
    - **Beschreibung**: Azure Databricks-Arbeitsbereich
    - **Verbindung über Integration Runtime herstellen**: AutoResolveIntegrationRuntime
    - **Kontoauswahlmethode**: Aus Azure-Abonnement
    - **Azure-Abonnement**: *Wählen Sie Ihr Abonnement aus*
    - **Databricks-Arbeitsbereich**: *Wählen Sie Ihren Arbeitsbereich **databricksxxxxxxx** aus*
    - **Cluster auswählen**: Neuer Auftragscluster
    - **Databricks-Arbeitsbereichs-URL**: *Automatische Festlegung auf Ihre Databricks-Arbeitsbereichs-URL*
    - **Authentifizierungstyp**: Zugriffstoken
    - **Zugriffstoken**: *Fügen Sie Ihr Zugriffstoken ein*
    - **Clusterversion**: 13.3 LTS (Spark 3.4.1, Scala 2.12)
    - **Clusterknotentyp**: Standard_DS3_v2
    - **Python-Version**: 3
    - **Worker-Optionen**: Behoben
    - **Worker**: 1

## Ausführen des Azure Databricks-Notebooks mithilfe einer Pipeline

Nachdem Sie nun einen verknüpften Dienst erstellt haben, können Sie ihn in einer Pipeline verwenden, um das zuvor angezeigte Notebook auszuführen.

### Erstellen einer Pipeline

1. Wählen Sie in Data Factory Studio im linken Navigationsbereich die Option **Autor** aus.
2. Verwenden Sie auf der Seite **Autor** im Bereich **Factoryressourcen** das Symbol ****+, um eine **Pipeline** hinzuzufügen.
3. Ändern Sie im Bereich **Eigenschaften** für die neue Pipeline ihren Namen in **Daten mit Databricks verarbeiten**. Verwenden Sie dann die Schaltfläche **Eigenschaften** (die ähnlich aussieht wie **<sub>*</sub>**) rechts auf der Symbolleiste, um den Bereich **Eigenschaften** auszublenden.
4. Erweitern Sie im Bereich **Aktivitäten** die Option **Databricks**, und ziehen Sie eine **Notebook**-Aktivität auf die Oberfläche des Pipeline-Designers.
5. Wenn die neue Aktivität **Notebook1** ausgewählt ist, legen Sie die folgenden Eigenschaften im unteren Bereich fest:
    - **General**:
        - **Name**: Daten verarbeiten
    - **Azure Databricks**:
        - **Mit Databricks verknüpfter Dienst**: *Wählen Sie den zuvor erstellten verknüpften **AzureDatabricks**-Dienst aus.*
    - **Einstellungen**:
        - **Notebook-Pfad**: *Navigieren Sie zum Ordner **Users/your_user_name**, und wählen Sie das Notebook **Daten verarbeiten** aus*
        - **Basisparameter**: *Fügen Sie einen neuen Parameter mit dem Namen **folder** und dem Wert **product_data** hinzu*
6. Verwenden Sie die Schaltfläche **Überprüfen** oberhalb der Pipeline-Designer-Benutzeroberfläche, um die Pipeline zu überprüfen. Verwenden Sie dann die Schaltfläche **Alle veröffentlichen**, um sie zu veröffentlichen (speichern).

### Führen Sie die Pipeline aus.

1. Wählen Sie oberhalb der Pipeline-Designer-Benutzeroberfläche **Trigger hinzufügen** und dann **Jetzt auslösen** aus.
2. Wählen Sie im Bereich **Pipelineausführung**die Schaltfläche **OK** aus, um die Pipeline auszuführen.
3. Wählen Sie im linken Navigationsbereich **Überwachen** und beachten Sie die Pipeline **Daten mit Databricks verarbeiten** auf der Registerkarte **Pipelineausführungen**. Die Ausführung kann eine Weile dauern, da ein Spark-Cluster dynamisch erstellt und das Notebook ausgeführt wird. Sie können die Schaltfläche **↻ Aktualisieren** auf der Seite **Pipelineausführungen** verwenden, um den Status zu aktualisieren.

    > **Hinweis**: Wenn Ihre Pipeline fehlschlägt, weist Ihr Abonnement möglicherweise ein unzureichendes Kontingent in der Region auf, in der Ihr Azure Databricks-Arbeitsbereich bereitgestellt wird, um einen Auftragscluster zu erstellen. Anleitungen finden Sie unter [Der Grenzwert für CPU-Kerne verhindert die Clustererstellung](https://docs.microsoft.com/azure/databricks/kb/clusters/azure-core-limit). In diesem Fall können Sie versuchen, Ihren Arbeitsbereich zu löschen und in einer anderen Region einen neuen zu erstellen. Sie können einen Bereich als Parameter für das Setupskript wie folgt angeben: `./setup.ps1 eastus`

4. Wenn die Ausführung erfolgreich ist, wählen Sie den Namen aus, um die Ausführungsdetails anzuzeigen. Wählen Sie dann auf der Seite **Daten mit Databricks verarbeiten** im Abschnitt **Aktivitätsausführung** die Aktivität **Daten verarbeiten** aus. Verwenden Sie das Symbol ***Ausgabe***, um die JSON-Ausgabe der Aktivität anzuzeigen, die wie folgt aussehen sollte:
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

5. Beachten Sie den Wert **runOutput**, bei dem es sich um die *path*-Variable handelt, in der das Notebook die Daten gespeichert hat.

## Löschen von Azure Databricks-Ressourcen

Nachdem Sie die Azure Data Factory-Integration mit Azure Databricks erkundet haben, müssen Sie die von Ihnen erstellten Ressourcen löschen, um unnötige Azure-Kosten zu vermeiden und Kapazität in Ihrem Abonnement freizugeben.

1. Schließen Sie den Azure Databricks-Arbeitsbereich und die Azure Data Factory Studio-Browserregisterkarten, und kehren Sie zum Azure-Portal zurück.
2. Wählen Sie auf der **Startseite** des Azure-Portals die Option **Ressource erstellen** aus.
3. Wählen Sie die Ressourcengruppe **dp203-*xxxxx*** aus, die Ihren Azure Databricks- und Azure Data Factory-Arbeitsbereich (nicht die verwaltete Ressourcengruppe) enthält.
4. Wählen Sie oben auf der Seite **Übersicht** für Ihre Ressourcengruppe die Option **Ressourcengruppe löschen** aus.
5. Geben Sie den Namen der Ressourcengruppe ein, um zu bestätigen, dass Sie sie löschen möchten, und wählen Sie **Löschen** aus.

    Nach ein paar MInuten werden Ihre Ressourcengruppe und die damit verbundene verwaltete Ressourcengruppe des Arbeitsbereichs gelöscht.
