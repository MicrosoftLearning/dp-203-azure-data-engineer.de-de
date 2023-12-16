---
lab:
  title: Verwenden eines SQL-Warehouse in Azure Databricks
  ilt-use: Optional demo
---

# Verwenden eines SQL-Warehouse in Azure Databricks

SQL ist eine Branchenstandardsprache zum Abfragen und Bearbeiten von Daten. Viele Datenanalysten führen Datenanalysen mithilfe von SQL aus, um Tabellen in einer relationalen Datenbank abzufragen. Azure Databricks umfasst SQL-Funktionen, die auf Spark- und Delta Lake-Technologien basieren, um eine relationale Datenbankschicht über Dateien in einem Data Lake bereitzustellen.

Diese Übung dauert ca. **30** Minuten.

## Vorbereitung

Sie benötigen ein [Azure-Abonnement](https://azure.microsoft.com/free), in dem Sie Zugriff auf Administratorebene und ein ausreichendes Kontingent in mindestens einer Region haben, um ein Azure Databricks SQL-Warehouse bereitzustellen.

## Bereitstellen eines Azure Databricks-Arbeitsbereichs

In dieser Übung benötigen Sie einen Azure Databricks-Arbeitsbereich auf Premiumebene.

> **Tipp**: Wenn Sie bereits über einen *Premium-* oder *Testarbeitsbereich* für Azure Databricks verfügen, können Sie dieses Verfahren überspringen.

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
    cd dp-203/Allfiles/labs/26
    ./setup.ps1
    ```

6. Wenn Sie dazu aufgefordert werden, wählen Sie aus, welches Abonnement Sie verwenden möchten (dies geschieht nur, wenn Sie Zugriff auf mehrere Azure-Abonnements haben).

7. Warten Sie, bis das Skript abgeschlossen ist. Dies dauert in der Regel etwa 5 Minuten, aber in einigen Fällen kann es länger dauern. Während Sie warten, lesen Sie den Artikel [Was ist Data Warehousing in Azure Databricks?](https://learn.microsoft.com/azure/databricks/sql/) in der Dokumentation zu Azure Databricks.

## Anzeigen und Starten eines SQL-Warehouse

1. Wenn die Azure Databricks-Arbeitsbereichsressource bereitgestellt wurde, wechseln Sie im Azure-Portal zu dieser Ressource.
1. Verwenden Sie auf der Seite **Übersicht** für Ihren Azure Databricks-Arbeitsbereich die Schaltfläche **Arbeitsbereich starten**, um Ihren Azure Databricks-Arbeitsbereich auf einer neuen Browserregisterkarte zu öffnen. Melden Sie sich an, wenn Sie dazu aufgefordert werden.

    > **Tipp**: Während Sie das Databricks-Arbeitsbereichsportal verwenden, werden möglicherweise verschiedene Tipps und Benachrichtigungen angezeigt. Schließen Sie diese, und folgen Sie den Anweisungen, um die Aufgaben in dieser Übung auszuführen.

1. Zeigen Sie das Azure Databricks-Arbeitsbereichsportal an, und beachten Sie, dass die Randleiste auf der linken Seite die Namen der Aufgabenkategorien enthält.
1. Wählen Sie in der Randleiste unter **SQL** die Option **SQL-Warehouses** aus.
1. Beachten Sie, dass der Arbeitsbereich bereits ein SQL-Warehouse mit dem Namen **Starter Warehouse** enthält.
1. Im Menü **Aktionen** (**⁝**) für das SQL-Warehouse wählen Sie **Bearbeiten** aus. Legen Sie dann die Eigenschaft **Clustergröße** auf **2X-Klein** fest, und speichern Sie Ihre Änderungen.
1. Verwenden Sie die Schaltfläche **Start**, um das SQL Warehouse zu starten (was eine Minute oder zwei dauern kann).

> **Hinweis**: Wenn Ihr SQL-Warehouse nicht gestartet werden kann, verfügt Ihr Abonnement möglicherweise über ein unzureichendes Kontingent in der Region, in der Ihr Azure Databricks-Arbeitsbereich bereitgestellt wird. Weitere Informationen finden Sie unter [Erforderliches Azure vCPU-Kontingent](https://docs.microsoft.com/azure/databricks/sql/admin/sql-endpoints#required-azure-vcpu-quota). In diesem Fall können Sie versuchen, wie in der Fehlermeldung beschrieben eine Kontingenterhöhung anzufordern, wenn das Warehouse nicht gestartet werden kann. Alternativ können Sie versuchen, Ihren Arbeitsbereich zu löschen und einen neuen Arbeitsbereich in einer anderen Region zu erstellen. Sie können einen Bereich als Parameter für das Setupskript wie folgt angeben: `./setup.ps1 eastus`

## Erstellen eines Datenbankschemas

1. Wenn Ihr SQL-Warehouse * ausgeführt* wird, wählen Sie **SQL-Editor** in der Randleiste aus.
2. Beachten Sie im Bereich **Schemabrowser**, dass der Katalog *hive_metastore* eine Datenbank mit dem Namen **default** enthält.
3. Geben Sie im Bereich **Neue Abfrage** den folgenden SQL-Code ein:

    ```sql
    CREATE SCHEMA adventureworks;
    ```

4. Verwenden Sie die Schaltfläche **►Ausführen (1 Ausführung)**, um den SQL-Code auszuführen.
5. Wenn der Code erfolgreich ausgeführt wurde, verwenden Sie im Bereich **Schemabrowser** die Schaltfläche „Aktualisieren“ am unteren Rand des Bereichs, um die Liste zu aktualisieren. Erweitern Sie dann **hive_metastore** und **adventureworks**. Die Datenbank wurde zwar erstellt, enthält jedoch keine Tabellen.

Sie können die **default**-Datenbank für Ihre Tabellen verwenden, aber beim Erstellen eines analytischen Datenspeichers empfiehlt es sich, benutzerdefinierte Datenbanken für bestimmte Daten zu erstellen.

## Erstellen einer Tabelle

1. Laden Sie die Datei [products.csv](https://raw.githubusercontent.com/MicrosoftLearning/dp-203-azure-data-engineer/master/Allfiles/labs/26/data/products.csv)** auf Ihren lokalen Computer herunter, und speichern Sie sie als **products.csv**.**
1. Im Azure Databricks-Arbeitsbereichsportal wählen Sie in der Randleiste **(+) Neu** und dann **Dateiupload** aus und laden die Datei **products.csv** hoch, die Sie auf Ihren Computer heruntergeladen haben.
1. Wählen Sie auf der Seite **Daten hochladen** das Schema **adventureworks** aus, und legen Sie den Tabellennamen auf **products** fest. Wählen Sie dann in der unteren linken Ecke der Seite **Tabelle erstellen** aus.
1. Wenn die Tabelle erstellt wurde, überprüfen Sie die Details.

Die Möglichkeit zum Erstellen einer Tabelle durch Importieren von Daten aus einer Datei erleichtert das Auffüllen einer Datenbank. Sie können auch Spark SQL verwenden, um Tabellen mithilfe von Code zu erstellen. Die Tabellen selbst sind Metadatendefinitionen im Hive-Metastore, und die darin enthaltenen Daten werden im Delta-Format im Databricks File System (DBFS)-Speicher gespeichert.

## Erstellen einer Abfrage

1. Klicken Sie in der Seitenleiste auf **(+) Neu**, und wählen Sie **Abfrage** aus.
2. Im Bereich **Schemabrowser** erweitern Sie **hive_metastore** und **adventureworks**, und stellen sicher, dass die Tabelle **products** aufgeführt ist.
3. Geben Sie im Bereich **Neue Abfrage** den folgenden SQL-Code ein:

    ```sql
    SELECT ProductID, ProductName, Category
    FROM adventureworks.products; 
    ```

4. Verwenden Sie die Schaltfläche **►Ausführen (1 Ausführung)**, um den SQL-Code auszuführen.
5. Überprüfen Sie nach Abschluss der Abfrage die Ergebnistabelle.
6. Verwenden Sie die Schaltfläche **Speichern** oben rechts im Abfrage-Editor, um die Abfrage als **Produkte und Kategorien** zu speichern.

Durch das Speichern einer Abfrage können dieselben Daten zu einem späteren Zeitpunkt wieder abgerufen werden.

## Erstellen eines Dashboards

1. Wählen Sie in der Randleiste **(+) Neu** und dann **Dashboard** aus.
2. Geben Sie im Dialogfeld **Neues Dashboard** den Namen **Adventure Works-Produkte** ein, und wählen Sie **Speichern** aus.
3. Wählen Sie im Dashboard **Adventure Works-Produkte** in der Dropdownliste **Hinzufügen** die Option **Visualisierung** aus.
4. Wählen Sie im Dialogfeld **Visualisierungs-Widget hinzufügen** die Abfrage **Produkte und Kategorien** aus. Wählen Sie dann **Neue Visualisierung erstellen** aus, legen Sie den Titel auf **Produkte pro Kategorie** fest, und wählen Sie **Visualisierung erstellen** aus.
5. Legen Sie im Visualisierungs-Editor die folgenden Eigenschaften fest:
    - **Visualisierungstyp**: Balken
    - **Horizontales Diagramm**: Ausgewählt
    - **Y-Spalte**: Kategorie
    - **X-Spalten**: Produkt-ID : Anzahl
    - **Gruppieren nach**: *Leer lassen*
    - **Stapeln**: Deaktiviert
    - **Normalisieren von Werten in Prozent**: <u>Nicht</u>ausgewählt
    - **Fehlende und NULL-Werte**: Im Diagramm nicht anzeigen

6. Speichern Sie die Visualisierung, und zeigen Sie sie im Dashboard an.
7. Wählen Sie **Bearbeitung abgeschlossen** aus, um das Dashboard so anzuzeigen, wie es für Benutzer sichtbar sein wird.

Dashboards sind eine hervorragende Möglichkeit, Datentabellen und Visualisierungen mit Geschäftsbenutzern zu teilen. Sie können planen, dass die Dashboards regelmäßig aktualisiert und per E-Mail an Abonnenten gesendet werden.

## Löschen von Azure Databricks-Ressourcen

Nachdem Sie SQL-Warehouses in Azure Databricks untersucht haben, müssen Sie die von Ihnen erstellten Ressourcen löschen, um unnötige Azure-Kosten zu vermeiden und Kapazität in Ihrem Abonnement freizugeben.

1. Schließen Sie die Browserregisterkarte für den Databricks-Arbeitsbereich, und kehren Sie zum Azure-Portal zurück.
2. Wählen Sie auf der **Startseite** des Azure-Portals die Option **Ressource erstellen** aus.
3. Wählen Sie die Ressourcengruppe aus, die Ihren Azure Databricks-Arbeitsbereich enthält (nicht die verwaltete Ressourcengruppe).
4. Wählen Sie oben auf der Seite **Übersicht** für Ihre Ressourcengruppe die Option **Ressourcengruppe löschen** aus.
5. Geben Sie den Namen der Ressourcengruppe ein, um zu bestätigen, dass Sie sie löschen möchten, und wählen Sie **Löschen** aus.

    Nach ein paar MInuten werden Ihre Ressourcengruppe und die damit verbundene verwaltete Ressourcengruppe des Arbeitsbereichs gelöscht.
