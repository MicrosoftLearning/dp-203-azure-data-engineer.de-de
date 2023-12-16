---
lab:
  title: Einführung in Azure Databricks
  ilt-use: Suggested demo
---

# Einführung in Azure Databricks

Azure Databricks ist eine Microsoft Azure-basierte Version der beliebten Open-Source-Databricks-Plattform.

Ähnlich wie Azure Synapse Analytics bietet ein Azure Databricks-*Arbeitsbereich* einen zentralen Punkt für die Verwaltung von Databricks-Clustern, -Daten und -Ressourcen in Azure.

Diese Übung dauert ca. **30** Minuten.

## Vorbereitung

Sie benötigen ein [Azure-Abonnement](https://azure.microsoft.com/free), in dem Sie Administratorzugriff besitzen.

## Bereitstellen eines Azure Databricks-Arbeitsbereichs

In dieser Übung verwenden Sie ein Skript, um einen neuen Azure Databricks-Arbeitsbereich bereitzustellen.

> **Tipp**: Wenn Sie bereits über einen *Standard* - oder *Testarbeitsbereich* für Azure Databricks verfügen, können Sie dieses Verfahren überspringen und Ihren vorhandenen Arbeitsbereich verwenden.

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
    cd dp-203/Allfiles/labs/23
    ./setup.ps1
    ```

6. Wenn Sie dazu aufgefordert werden, wählen Sie aus, welches Abonnement Sie verwenden möchten (dies geschieht nur, wenn Sie Zugriff auf mehrere Azure-Abonnements haben).

7. Warten Sie, bis das Skript abgeschlossen ist. Dies dauert in der Regel etwa 5 Minuten, aber in einigen Fällen kann es länger dauern. Während Sie warten, lesen Sie den Artikel [Was ist Azure Databricks?](https://learn.microsoft.com/azure/databricks/introduction/) in der Dokumentation zu Azure Databricks.

## Erstellen eines Clusters

Azure Databricks ist eine verteilte Verarbeitungsplattform, die Apache Spark-*Cluster* verwendet, um Daten parallel auf mehreren Knoten zu verarbeiten. Jeder Cluster besteht aus einem Treiberknoten, um die Arbeit zu koordinieren, und Arbeitsknoten zum Ausführen von Verarbeitungsaufgaben.

In dieser Übung erstellen Sie einen *Einzelknotencluster* , um die in der Lab-Umgebung verwendeten Computeressourcen zu minimieren (in denen Ressourcen möglicherweise eingeschränkt werden). In einer Produktionsumgebung erstellen Sie in der Regel einen Cluster mit mehreren Workerknoten.

> **Tipp**: Wenn Sie bereits über einen Cluster mit einer 13.3 LTS-Laufzeitversion in Ihrem Azure Databricks-Arbeitsbereich verfügen, können Sie damit diese Übung abschließen und dieses Verfahren überspringen.

1. Navigieren Sie im Azure-Portal zur Ressourcenruppe **dp203-*xxxxxxx***, die vom Skript erstellt wurde (oder die Ressourcengruppe, die Ihren vorhandenen Azure Databricks-Arbeitsbereich enthält).
1. Wählen Sie Ihre Azure Databricks-Dienstressource (mit dem Namen **Databricks*xxxxxxx*** aus, wenn Sie das Setupskript zum Erstellen verwendet haben).
1. Verwenden Sie auf der Seite **Übersicht** für Ihren Arbeitsbereich die Schaltfläche **Arbeitsbereich starten**, um Ihren Azure Databricks-Arbeitsbereich auf einer neuen Browserregisterkarte zu öffnen. Melden Sie sich an, wenn Sie dazu aufgefordert werden.

    > **Tipp**: Während Sie das Databricks-Arbeitsbereichsportal verwenden, werden möglicherweise verschiedene Tipps und Benachrichtigungen angezeigt. Schließen Sie diese, und folgen Sie den Anweisungen, um die Aufgaben in dieser Übung auszuführen.

1. Zeigen Sie das Azure Databricks-Arbeitsbereichsportal an, und beachten Sie, dass die Randleiste auf der linken Seite Links zu den verschiedenen Aufgabentypen enthält, die Sie ausführen können.

1. Wählen Sie in der Randleiste den Link ** (+) Neu** und dann **Cluster** aus.
1. Erstellen Sie auf der Seite **Neuer Cluster** einen neuen Cluster mit den folgenden Einstellungen:
    - **Clustername**:Cluster des  *Benutzernamens* (Standardclustername)
    - **Clustermodus**: Einzelknoten
    - **Zugriffsmodus**: Einzelner Benutzer (*mit ausgewähltem Benutzerkonto*)
    - **Databricks-Laufzeitversion**: 13.3 LTS (Spark 3.4.1, Scala 2.12)
    - **Photonbeschleunigung verwenden**: Ausgewählt
    - **Knotentyp**: Standard_DS3_v2
    - **Nach***30***Minuten Inaktivität beenden**

1. Warten Sie, bis der Cluster erstellt wurde. Dies kann ein oder zwei Minuten dauern.

> **Hinweis**: Wenn Ihr Cluster nicht gestartet werden kann, verfügt Ihr Abonnement möglicherweise über ein unzureichendes Kontingent in der Region, in der Ihr Azure Databricks-Arbeitsbereich bereitgestellt wird. Anleitungen finden Sie unter [Der Grenzwert für CPU-Kerne verhindert die Clustererstellung](https://docs.microsoft.com/azure/databricks/kb/clusters/azure-core-limit). In diesem Fall können Sie versuchen, Ihren Arbeitsbereich zu löschen und in einer anderen Region einen neuen zu erstellen. Sie können einen Bereich als Parameter für das Setupskript wie folgt angeben: `./setup.ps1 eastus`

## Analysieren einer Datendatei mithilfe von Spark

Wie in vielen Spark-Umgebungen unterstützt Databricks die Verwendung von Notebooks zum Kombinieren von Notizen und interaktiven Codezellen, mit denen Sie Daten untersuchen können.

1. Verwenden Sie in der Randleiste den Link ** (+) Neu**, um ein **Notebook** zu erstellen.
1. Ändern Sie den Standardnamen des Notebooks (**Unbenanntes Notebook *[Datum]***) in **Produkte erkunden**, und wählen Sie in der Dropdownliste **Verbinden** Ihren Cluster aus, wenn er noch nicht ausgewählt ist. Wenn der Cluster nicht ausgeführt wird, kann es eine Minute dauern, bis er gestartet wird.
1. Laden Sie die Datei [**products.csv**](https://raw.githubusercontent.com/MicrosoftLearning/dp-203-azure-data-engineer/master/Allfiles/labs/23/adventureworks/products.csv) auf Ihren lokalen Computer herunter, und speichern Sie sie als **products.csv**. Wählen Sie dann im Notebook **Produkte erkunden** im Menü **Datei** die Option **Daten nach DBFS hochladen** aus.
1. Notieren Sie sich im Dialogfeld **Daten hochladen** das **DBFS-Zielverzeichnis**, in das die Datei hochgeladen wird. Wählen Sie dann den Bereich **Dateien** aus, und laden Sie die Datei **products.csv** hoch, die Sie auf Ihren Computer heruntergeladen haben. Wenn die Datei hochgeladen wurde, wählen Sie **Weiter** aus.
1. Wählen Sie im Bereich **Dateien aus Notebooks abrufen** den PySpark-Beispielcode aus, und kopieren Sie ihn in die Zwischenablage. Hiermit laden Sie die Daten aus der Datei in einen DataFrame. Wählen Sie dann **Fertig** aus.
1. Fügen Sie im Notebook **Produkte erkunden** in der leeren Codezelle den kopierten Code ein, der so oder ähnlich aussehen sollte:

    ```python
    df1 = spark.read.format("csv").option("header", "true").load("dbfs:/FileStore/shared_uploads/user@outlook.com/products.csv")
    ```

1. Verwenden Sie die Menüoption **▸ Zelle ausführen** oben rechts in der Zelle, um sie auszuführen, und fügen Sie den Cluster an, wenn Sie dazu aufgefordert werden.
1. Warten Sie, bis der Spark-Auftrag vom Code ausgeführt wurde. Der Code hat ein *dataframe*-Objekt namens **df1** aus den Daten in der Datei erstellt, die Sie hochgeladen haben.
1. Verwenden Sie unter der vorhandenen Codezelle das Symbol **+**, um eine neue Codezelle hinzuzufügen. Geben Sie dann in der neuen Zelle den folgenden Code ein.

    ```python
    display(df1)
    ```

1. Verwenden Sie die Menüoption **▸ Zelle ausführen** oben rechts in der neuen Zelle, um sie auszuführen. Dieser Code zeigt den Inhalt des Datenframes an, der so oder ähnlich aussehen sollte:

    | ProductID | ProductName | Category | ListPrice |
    | -- | -- | -- | -- |
    | 771 | Mountain-100 Silver, 38 | Mountainbikes | 3399.9900 |
    | 772 | Mountain-100 Silver, 42 | Mountainbikes | 3399.9900 |
    | ... | ... | ... | ... |

1. Wählen Sie oberhalb der Ergebnistabelle **+** und dann **Visualisierung** aus, um den Visualisierungs-Editor anzuzeigen, und wenden Sie dann die folgenden Optionen an:
    - **Visualisierungstyp**: Balken
    - **X-Spalte**: Kategorie
    - **Y-Spalte**: *Fügen Sie eine neue Spalte hinzu und wählen Sie***ProductID**. *Wenden Sie die***Anzahl**-*Aggregation* an.

    Speichern Sie die Visualisierung, und beachten Sie, dass sie im Notebook wie folgt angezeigt wird:

    ![Ein Balkendiagramm mit Produktanzahl nach Kategorie](./images/databricks-chart.png)

## Erstellen und Abfragen einer Tabelle

Während viele Datenanalysen mit Sprachen wie Python oder Scala vertraut sind, um mit Daten in Dateien zu arbeiten, basieren viele Datenanalyselösungen auf relationalen Datenbanken; in denen Daten in Tabellen gespeichert und mithilfe von SQL bearbeitet werden.

1. Verwenden Sie im Notebook **Produkte erkunden** unter der Diagrammausgabe aus der zuvor ausgeführten Codezelle das Symbol **+**, um eine neue Zelle hinzuzufügen.
2. Geben Sie dann den folgenden Code in die neue Zelle ein, und führen Sie ihn aus:

    ```python
    df1.write.saveAsTable("products")
    ```

3. Fügen Sie nach Abschluss der Zelle eine neue Zelle mit dem folgenden Code hinzu:

    ```sql
    %sql

    SELECT ProductName, ListPrice
    FROM products
    WHERE Category = 'Touring Bikes';
    ```

4. Führen Sie die neue Zelle aus, die SQL-Code enthält, um den Namen und Preis der Produkte in der Kategorie *Touring Bikes* zurückzugeben.
5. In der Randleiste wählen Sie den Link **Katalog** und prüfen, ob die Tabelle **products** im Standarddatenbankschema erstellt wurde (die wenig überraschend **default** bezeichnet wird). Es ist möglich, Spark-Code zum Erstellen benutzerdefinierter Datenbankschemas und eines Schemas relationaler Tabellen zu verwenden, mit denen Datenanalysten Daten untersuchen und Analyseberichte generieren können.

## Löschen von Azure Databricks-Ressourcen

Nachdem Sie Azure Databricks erkundet haben, müssen Sie die von Ihnen erstellten Ressourcen löschen, um unnötige Azure-Kosten zu vermeiden und Kapazität in Ihrem Abonnement freizugeben.

1. Schließen Sie die Browserregisterkarte für den Databricks-Arbeitsbereich, und kehren Sie zum Azure-Portal zurück.
2. Wählen Sie auf der **Startseite** des Azure-Portals die Option **Ressource erstellen** aus.
3. Wählen Sie die Ressourcengruppe **dp203-*xxxxx*** (nicht die verwaltete Ressourcengruppe) aus, und stellen Sie sicher, dass sie Ihren Azure Databricks-Arbeitsbereich enthält.
4. Wählen Sie oben auf der Seite **Übersicht** für Ihre Ressourcengruppe die Option **Ressourcengruppe löschen** aus.
5. Geben Sie den Ressourcengruppennamen **dp203-*xxxxxxx*** ein, um zu bestätigen, dass Sie ihn löschen möchten, und wählen Sie dann **Löschen** aus.

    Nach ein paar Minuten werden Ihre Ressourcengruppe und die damit verbundenen verwalteten Ressourcengruppen des Arbeitsbereichs gelöscht.
