---
lab:
  title: Verwenden von Delta Lake in Azure Synapse Analytics
  ilt-use: Lab
---

# Verwenden von Delta Lake mit Spark in Azure Synapse Analytics

Delta Lake ist ein Open-Source-Projekt, um eine Transaktionsdatenspeicherschicht auf einem Data Lake zu erstellen. Delta Lake bietet Unterstützung für relationale Semantik für Batch- und Streamingdatenvorgänge und ermöglicht die Erstellung einer *Lakehouse*-Architektur, in der Apache Spark zum Verarbeiten und Abfragen von Daten in Tabellen verwendet werden kann, die auf zugrunde liegenden Dateien im Data Lake basieren.

Diese Übung dauert ca. **40** Minuten.

## Vor der Installation

Sie benötigen ein [Azure-Abonnement](https://azure.microsoft.com/free), in dem Sie Administratorzugriff besitzen.

## Bereitstellen eines Azure Synapse Analytics-Arbeitsbereichs

Sie benötigen einen Azure Synapse Analytics-Arbeitsbereich mit Zugriff auf den Datenspeicher und einen Apache Spark-Pool, den Sie zum Abfragen und Verarbeiten von Dateien im Data Lake verwenden können.

In dieser Übung verwenden Sie eine Kombination aus einem PowerShell-Skript und einer ARM-Vorlage, um einen Azure Synapse Analytics-Arbeitsbereich bereitzustellen.

1. Melden Sie sich beim [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com` an.
2. Verwenden Sie rechts neben der Suchleiste oben auf der Seite die Schaltfläche **[\>_]**, um eine neue Cloud Shell-Instanz im Azure-Portal zu erstellen. Wählen Sie eine ***PowerShell***-Umgebung aus, und erstellen Sie Speicher, falls Sie dazu aufgefordert werden. Die Cloud Shell bietet eine Befehlszeilenschnittstelle in einem Bereich am unteren Rand des Azure-Portals, wie hier gezeigt:

    ![Azure-Portal mit einem Cloud Shell-Bereich](./images/cloud-shell.png)

    > **Hinweis**: Wenn Sie zuvor eine Cloud Shell erstellt haben, die eine *Bash*-Umgebung verwendet, ändern Sie diese mithilfe des Dropdownmenüs oben links im Cloud Shell-Bereich zu ***PowerShell***.

3. Beachten Sie, dass Sie die Größe der Cloud Shell durch Ziehen der Trennzeichenleiste oben im Bereich ändern können oder den Bereich mithilfe der Symbole **&#8212;**, **&#9723;** und **X** oben rechts minimieren, maximieren und schließen können. Weitere Informationen zur Verwendung von Azure Cloud Shell finden Sie in der [Azure Cloud Shell-Dokumentation](https://docs.microsoft.com/azure/cloud-shell/overview).

4. Geben Sie im PowerShell-Bereich die folgenden Befehle ein, um dieses Repository zu klonen:

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Nachdem das Repository geklont wurde, geben Sie die folgenden Befehle ein, um in den Ordner für diese Übung zu wechseln. Führen Sie das darin enthaltene Skript **setup.ps1** aus:

    ```
    cd dp-203/Allfiles/labs/07
    ./setup.ps1
    ```

6. Wenn Sie dazu aufgefordert werden, wählen Sie aus, welches Abonnement Sie verwenden möchten (dies geschieht nur, wenn Sie Zugriff auf mehrere Azure-Abonnements haben).
7. Wenn Sie dazu aufgefordert werden, geben Sie ein geeignetes Kennwort ein, das für Ihren Azure Synapse SQL-Pool festgelegt werden soll.

    > **Hinweis**: Merken Sie sich unbedingt dieses Kennwort!

8. Warten Sie, bis das Skript abgeschlossen ist. Dies dauert in der Regel etwa 10 Minuten und in Ausnahmefällen auch länger. Während Sie warten, lesen Sie den Artikel [Was ist Delta Lake?](https://docs.microsoft.com/azure/synapse-analytics/spark/apache-spark-what-is-delta-lake) in der Dokumentation zu Azure Synapse Analytics.

## Erstellen von Deltatabellen

Das Skript stellt einen Azure Synapse Analytics-Arbeitsbereich und ein Azure Storage-Konto zum Hosten des Data Lake bereit und lädt dann eine Datei mit Daten in den Data Lake hoch.

### Erkunden der Daten im Data Lake

1. Wechseln Sie nach Abschluss des Skripts im Azure-Portal zur erstellten Ressourcengruppe **dp203-*xxxxxxx***, und wählen Sie Ihren Synapse-Arbeitsbereich aus.
2. Wählen Sie auf der Seite **Übersicht** für Ihren Synapse-Arbeitsbereich in der Karte **Synapse Studio öffnen** die Option **Öffnen** aus, um Synapse Studio in einer neuen Browserregisterkarte zu öffnen. Melden Sie sich an, wenn Sie dazu aufgefordert werden.
3. Verwenden Sie auf der linken Seite von Synapse Studio das Symbol **&rsaquo;&rsaquo;**, um das Menü zu erweitern. Dadurch werden die verschiedenen Seiten in Synapse Studio angezeigt, die Sie zum Verwalten von Ressourcen und zum Ausführen von Datenanalyseaufgaben verwenden.
4. Zeigen Sie auf der Seite **Daten** die Registerkarte **Verknüpft** an, und stellen Sie sicher, dass Ihr Arbeitsbereich einen Link zu Ihrem Azure Data Lake Storage Gen2-Speicherkonto enthält, dessen Name **synapse*xxxxxxx* (Primary - datalake*xxxxxxx*)** ähneln sollte.
5. Erweitern Sie Ihr Speicherkonto, und stellen Sie sicher, dass es einen Dateisystemcontainer mit dem Namen **Dateien** enthält.
6. Wählen Sie den Container **Dateien** aus und beachten Sie, dass er einen Ordner mit dem Namen **Produkte** enthält. Dieser Ordner enthält die Daten, mit denen Sie in dieser Übung arbeiten werden.
7. Öffnen Sie den Ordner **Produkte**, und beachten Sie, dass er eine Datei mit dem Namen **products.csv** enthält.
8. Wählen Sie **products.csv** und anschließend in der Liste **Neues Notebook** auf der Symbolleiste **In Datenframe laden** aus.
9. Wählen Sie im daraufhin angezeigten Bereich **Notebook 1** in der Liste **Anfügen an** den Spark-Pool **sparkxxxxxxx** aus, den Sie zuvor erstellt haben, und stellen Sie sicher, dass die **Sprache** auf **PySpark (Python)** festgelegt ist.
10. Überprüfen Sie den Code in der ersten (und einzigen) Zelle des Notebooks, der wie folgt aussehen sollte:

    ```Python
    %%pyspark
    df = spark.read.load('abfss://files@datalakexxxxxxx.dfs.core.windows.net/products/products.csv', format='csv'
    ## If header exists uncomment line below
    ##, header=True
    )
    display(df.limit(10))
    ```

11. Aufheben der Auskommentierung der Zeile *,header=True* (da die products.csv-Datei die Spaltenüberschriften in der ersten Zeile enthält), sodass Ihr Code wie folgt aussieht:

    ```Python
    %%pyspark
    df = spark.read.load('abfss://files@datalakexxxxxxx.dfs.core.windows.net/products/products.csv', format='csv'
    ## If header exists uncomment line below
    , header=True
    )
    display(df.limit(10))
    ```

12. Verwenden Sie das Symbol **&#9655;** links neben der Codezelle, um sie auszuführen, und warten Sie auf die Ergebnisse. Wenn Sie eine Zelle zum ersten Mal in einem Notebook ausführen, wird der Spark-Pool gestartet. Es kann also etwa eine Minute dauern, bis Ergebnisse zurückgegeben werden. Letztlich sollten die Ergebnisse unterhalb der Zelle angezeigt werden und in etwa wie folgt aussehen:

    | ProductID | ProductName | Kategorie | ListPrice |
    | -- | -- | -- | -- |
    | 771 | Mountain-100 Silver, 38 | Mountainbikes | 3399.9900 |
    | 772 | Mountain-100 Silver, 42 | Mountainbikes | 3399.9900 |
    | ... | ... | ... | ... |

### Laden der Dateidaten in eine Deltatabelle

1. Verwenden Sie unter den von der ersten Codezelle zurückgegebenen Ergebnissen die Schaltfläche **+ Code**, um eine neue Codezelle hinzuzufügen. Geben Sie dann den folgenden Code in die neue Zelle ein, und führen Sie sie aus:

    ```Python
    delta_table_path = "/delta/products-delta"
    df.write.format("delta").save(delta_table_path)
    ```

2. Verwenden Sie auf der Registerkarte **Dateien** das Symbol **&#8593;** in der Symbolleiste, um zum Stamm des Containers **Dateien** zurückzukehren, und beachten Sie, dass ein neuer Ordner namens **Delta** erstellt wurde. Öffnen Sie diesen Ordner und die darin enthaltene Tabelle **products-delta**, in der jetzt die Parquet-Datei(en) mit den enthaltenen Daten angezeigt werden sollten.

3. Kehren Sie zur Registerkarte **Notebook 1** zurück und fügen Sie eine weitere neue Codezelle hinzu. Fügen Sie dann in der neuen Zelle den folgenden Code hinzu, und führen Sie ihn aus:

    ```Python
    from delta.tables import *
    from pyspark.sql.functions import *

    # Create a deltaTable object
    deltaTable = DeltaTable.forPath(spark, delta_table_path)

    # Update the table (reduce price of product 771 by 10%)
    deltaTable.update(
        condition = "ProductID == 771",
        set = { "ListPrice": "ListPrice * 0.9" })

    # View the updated data as a dataframe
    deltaTable.toDF().show(10)
    ```

    Die Daten werden in ein **DeltaTable**-Objekt geladen und aktualisiert. Sie können die Aktualisierung an den veränderten Abfrageergebnisse erkennen.

4. Fügen Sie eine weitere Codezelle mit dem folgenden Code hinzu und führen Sie den Code aus:

    ```Python
    new_df = spark.read.format("delta").load(delta_table_path)
    new_df.show(10)
    ```

    Der Code lädt die Delta-Tabellendaten aus seiner Position im Data Lake in einen Datenframe und überprüft, ob die Änderung, die Sie über ein **DeltaTable**-Objekt vorgenommen haben, beibehalten wurde.

5. Ändern Sie den soeben ausgeführten Code wie folgt, wobei Sie die Option auswählen, die *Zeitreise*-funktion von Delta Lake zum Anzeigen einer früheren Version der Daten zu verwenden.

    ```Python
    new_df = spark.read.format("delta").option("versionAsOf", 0).load(delta_table_path)
    new_df.show(10)
    ```

    Wenn Sie den geänderten Code ausführen, zeigen die Ergebnisse die ursprüngliche Version der Daten an.

6. Fügen Sie eine weitere Codezelle mit dem folgenden Code hinzu und führen Sie den Code aus:

    ```Python
    deltaTable.history(10).show(20, False, True)
    ```

    Es wird nun der Verlauf der letzten 20 Änderungen an der Tabelle angezeigt – es sollten zwei zu sehen sein (die ursprüngliche Erstellung und die von Ihnen vorgenommene Aktualisierung).

## Erstellen von Katalogtabellen

Bisher haben Sie mit Delta-Tabellen gearbeitet, indem Sie Daten aus dem Ordner mit den Parquet-Dateien laden, auf denen die Tabelle basiert. Sie können *Katalogtabellen* definieren, die die Daten kapseln und eine benannte Tabellenentität bereitstellen, auf die Sie im SQL-Code verweisen können. Spark unterstützt zwei Arten von Katalogtabellen für Delta Lake:

- *Externe* Tabellen, die durch den Pfad zu den Parquet-Dateien mit den Tabellendaten definiert werden.
- *Verwaltete* Tabellen, die im Hive-Metaspeicher für den Spark-Pool definiert sind.

### Erstellen einer externen Tabelle

1. Fügen Sie den folgenden Code in einer neuen Codezelle hinzu, und führen Sie ihn aus:

    ```Python
    spark.sql("CREATE DATABASE AdventureWorks")
    spark.sql("CREATE TABLE AdventureWorks.ProductsExternal USING DELTA LOCATION '{0}'".format(delta_table_path))
    spark.sql("DESCRIBE EXTENDED AdventureWorks.ProductsExternal").show(truncate=False)
    ```

    Dieser Code erstellt eine neue Datenbank namens **AdventureWorks** und erstellt dann eine externe Tabelle namens **ProductsExternal** in dieser Datenbank, basierend auf dem Pfad zu den zuvor definierten Parquet-Dateien. Anschließend wird eine Beschreibung der Eigenschaften der Tabelle angezeigt. Beachten Sie, dass die Eigenschaft **Speicherort** der von Ihnen angegebene Pfad ist.

2. Fügen Sie eine neue Codezelle hinzu, und geben Sie anschließend den folgenden Code ein und führen ihn aus:

    ```sql
    %%sql

    USE AdventureWorks;

    SELECT * FROM ProductsExternal;
    ```

    Der Code verwendet SQL, um den Kontext zur **AdventureWorks**-Datenbank zu wechseln (die keine Daten zurückgibt) und dann die Tabelle **ProductsExternal** abzufragen (die Ergebnisse der Abfrage enthalten die Produktdaten in der Delta Lake-Tabelle).

### Erstellen einer verwalteten Tabelle

1. Fügen Sie den folgenden Code in einer neuen Codezelle hinzu, und führen Sie ihn aus:

    ```Python
    df.write.format("delta").saveAsTable("AdventureWorks.ProductsManaged")
    spark.sql("DESCRIBE EXTENDED AdventureWorks.ProductsManaged").show(truncate=False)
    ```

    Dieser Code erstellt eine verwaltete Tabelle namens **ProductsManaged** basierend auf dem Datenframe, den Sie ursprünglich aus der Datei **products.csv** geladen haben (bevor Sie den Preis des Produkts 771 aktualisiert haben). Sie geben keinen Pfad für die von der Tabelle verwendeten Parquet-Dateien an – dies wird für Sie im Hive-Metastore verwaltet und wird in der Eigenschaft **Speicherort** in der Tabellenbeschreibung (im Pfad **Dateien/synapse/workspaces/synapsexxxxxxx/warehouse**) angezeigt.

2. Fügen Sie eine neue Codezelle hinzu, und geben Sie anschließend den folgenden Code ein und führen ihn aus:

    ```sql
    %%sql

    USE AdventureWorks;

    SELECT * FROM ProductsManaged;
    ```

    Der Code verwendet SQL, um die Tabelle **ProductsManaged** abzufragen.

### Vergleichen externer und verwalteter Tabellen

1. Fügen Sie den folgenden Code in einer neuen Codezelle hinzu, und führen Sie ihn aus:

    ```sql
    %%sql

    USE AdventureWorks;

    SHOW TABLES;
    ```

    Dieser Code listet die Tabellen in der Datenbank **AdventureWorks** auf.

2. Ändern Sie die Codezelle wie folgt, und führen Sie sie aus:

    ```sql
    %%sql

    USE AdventureWorks;

    DROP TABLE IF EXISTS ProductsExternal;
    DROP TABLE IF EXISTS ProductsManaged;
    ```

    Dieser Code legt die Tabellen aus dem Metaspeicher ab.

3. Kehren Sie zur Registerkarte **Dateien** zurück und zeigen Sie den Ordner **Dateien/delta/products-delta** an. Beachten Sie, dass die Datendateien noch immer an diesem Speicherort gespeichert sind. Durch Ablegen der externen Tabelle wurde die Tabelle aus dem Metastore entfernt, die Datendateien blieben jedoch erhalten.
4. Zeigen Sie den Ordner **Dateien/synapse/workspaces/synapsexxxxxxx/warehouse** an und beachten Sie, dass es keinen Ordner für die **ProductsManaged**-Tabellendaten gibt. Durch Ablegen einer verwalteten Tabelle wird die Tabelle aus dem Metastore entfernt und auch die Datendateien der Tabelle gelöscht.

### Erstellen einer Tabelle mit SQL

1. Fügen Sie eine neue Codezelle hinzu, und geben Sie anschließend den folgenden Code ein und führen ihn aus:

    ```sql
    %%sql

    USE AdventureWorks;

    CREATE TABLE Products
    USING DELTA
    LOCATION '/delta/products-delta';
    ```

2. Fügen Sie eine neue Codezelle hinzu, und geben Sie anschließend den folgenden Code ein und führen ihn aus:

    ```sql
    %%sql

    USE AdventureWorks;

    SELECT * FROM Products;
    ```

    Beachten Sie, dass die neue Katalogtabelle für den vorhandenen Tabellenordner Delta Lake erstellt wurde, der die zuvor vorgenommenen Änderungen widerspiegelt.

## Verwenden von Delta-Tabellen zum Streamen von Daten

Delta Lake unterstützt das Streamen von Daten. Deltatabellen können eine *Senke* oder *Quelle* für Datenströme sein, die mit der Spark Structured Streaming-API erstellt wurden. In diesem Beispiel verwenden Sie eine Deltatabelle als Senke für einige Streamingdaten in einem simulierten IoT-Szenario (Internet der Dinge).

1. Kehren Sie zur Registerkarte **Notebook 1** zurück und fügen Sie eine neue Codezelle hinzu. Fügen Sie dann in der neuen Zelle den folgenden Code hinzu, und führen Sie ihn aus:

    ```python
    from notebookutils import mssparkutils
    from pyspark.sql.types import *
    from pyspark.sql.functions import *

    # Create a folder
    inputPath = '/data/'
    mssparkutils.fs.mkdirs(inputPath)

    # Create a stream that reads data from the folder, using a JSON schema
    jsonSchema = StructType([
    StructField("device", StringType(), False),
    StructField("status", StringType(), False)
    ])
    iotstream = spark.readStream.schema(jsonSchema).option("maxFilesPerTrigger", 1).json(inputPath)

    # Write some event data to the folder
    device_data = '''{"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev2","status":"error"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"error"}
    {"device":"Dev2","status":"ok"}
    {"device":"Dev2","status":"error"}
    {"device":"Dev1","status":"ok"}'''
    mssparkutils.fs.put(inputPath + "data.txt", device_data, True)
    print("Source stream created...")
    ```

    Stellen Sie sicher, dass die Meldung *Quelldatenstrom erstellt...* ausgegeben wird. Der Code, den Sie gerade ausgeführt haben, hat eine Streamingdatenquelle basierend auf einem Ordner erstellt, in dem einige Daten gespeichert wurden, die Messwerte von hypothetischen IoT-Geräten darstellen.

2. Fügen Sie den folgenden Code in einer neuen Codezelle hinzu, und führen Sie ihn aus:

    ```python
    # Write the stream to a delta table
    delta_stream_table_path = '/delta/iotdevicedata'
    checkpointpath = '/delta/checkpoint'
    deltastream = iotstream.writeStream.format("delta").option("checkpointLocation", checkpointpath).start(delta_stream_table_path)
    print("Streaming to delta sink...")
    ```

    Dieser Code schreibt die Streaminggerätedaten im Delta-Format.

3. Fügen Sie den folgenden Code in einer neuen Codezelle hinzu, und führen Sie ihn aus:

    ```python
    # Read the data in delta format into a dataframe
    df = spark.read.format("delta").load(delta_stream_table_path)
    display(df)
    ```

    Dieser Code liest die gestreamten Daten im Delta-Format in einen Datenframe vor. Beachten Sie, dass sich der Code zum Laden von Streamingdaten nicht von dem unterscheidet, der zum Laden statischer Daten aus einem Delta-Ordner verwendet wird.

4. Fügen Sie den folgenden Code in einer neuen Codezelle hinzu, und führen Sie ihn aus:

    ```python
    # create a catalog table based on the streaming sink
    spark.sql("CREATE TABLE IotDeviceData USING DELTA LOCATION '{0}'".format(delta_stream_table_path))
    ```

    Dieser Code erstellt, basierend auf dem Delta-Ordner, eine Katalogtabelle mit dem Namen **IotDeviceData** (in der Datenbank **Standard**). Auch hier ist dieser Code identisch mit dem, der für Nicht-Streaming-Daten verwendet würde.

5. Fügen Sie den folgenden Code in einer neuen Codezelle hinzu, und führen Sie ihn aus:

    ```sql
    %%sql

    SELECT * FROM IotDeviceData;
    ```

    Dieser Code fragt die Tabelle **IotDeviceData** ab, die die Gerätedaten aus der Streamingquelle enthält.

6. Fügen Sie den folgenden Code in einer neuen Codezelle hinzu, und führen Sie ihn aus:

    ```python
    # Add more data to the source stream
    more_data = '''{"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"error"}
    {"device":"Dev2","status":"error"}
    {"device":"Dev1","status":"ok"}'''

    mssparkutils.fs.put(inputPath + "more-data.txt", more_data, True)
    ```

    Dieser Code schreibt weitere hypothetische Gerätedaten in die Streamingquelle.

7. Fügen Sie den folgenden Code in einer neuen Codezelle hinzu, und führen Sie ihn aus:

    ```sql
    %%sql

    SELECT * FROM IotDeviceData;
    ```

    Dieser Code fragt die **IotDeviceData**-Tabelle erneut ab, die nun die zusätzlichen Daten enthalten sollte, die der Streamingquelle hinzugefügt wurden.

8. Fügen Sie den folgenden Code in einer neuen Codezelle hinzu, und führen Sie ihn aus:

    ```python
    deltastream.stop()
    ```

    Dieser Code beendet den Stream.

## Abfragen einer Delta-Tabelle aus einem serverlosen SQL-Pool

Zusätzlich zu Spark-Pools umfasst Azure Synapse Analytics einen integrierten serverlosen SQL-Pool. Sie können das relationale Datenbankmodul in diesem Pool verwenden, um Delta-Tabellen mithilfe von SQL abzufragen.

1. Navigieren Sie auf der Registerkarte **Dateien** zum Ordner **Dateien/Delta**.
2. Wählen Sie den Ordner **Products-Delta** aus, und wählen Sie auf der Symbolleiste in der Dropdownliste **Neues SQL-Skript** die Option **Erste 100 Zeilen auswählen** aus.
3. Wählen Sie im Bereich **Erste 100 Zeilen auswählen** in der Liste **Dateityp** **Delta-Format** und dann **Übernehmen** aus.
4. Überprüfen Sie den generierten SQL-Code, der wie folgt aussehen sollte:

    ```sql
    -- This is auto-generated code
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/delta/products-delta/',
            FORMAT = 'DELTA'
        ) AS [result]
    ```

5. Verwenden Sie das Symbol **&#9655; Ausführen**, um das Skript auszuführen und überprüfen Sie die Ergebnisse. Sie sollten ungefähr folgendermaßen aussehen:

    | ProductID | ProductName | Kategorie | ListPrice |
    | -- | -- | -- | -- |
    | 771 | Mountain-100 Silver, 38 | Mountainbikes | 3059.991 |
    | 772 | Mountain-100 Silver, 42 | Mountainbikes | 3399.9900 |
    | ... | ... | ... | ... |

    Dies veranschaulicht, wie Sie einen serverlosen SQL-Pool verwenden können, um mit Spark erstellte Dateien im Delta-Format abzufragen und die Ergebnisse für die Berichterstellung oder Analyse zu verwenden.

6. Ersetzen Sie die Abfrage durch den folgenden SQL-Code:

    ```sql
    USE AdventureWorks;

    SELECT * FROM Products;
    ```

7. Führen Sie den Code aus, und beachten Sie, dass Sie auch den serverlosen SQL-Pool verwenden können, um Delta Lake-Daten in Katalogtabellen abzufragen, die im Spark-Metastore definiert sind.

## Löschen von Azure-Ressourcen

Wenn Sie sich mit Azure Synapse Analytics vertraut gemacht haben, sollten Sie die erstellten Ressourcen löschen, um unnötige Azure-Kosten zu vermeiden.

1. Schließen Sie die Registerkarte mit Synapse Studio, und kehren Sie zum Azure-Portal zurück.
2. Wählen Sie auf der **Startseite** des Azure-Portals die Option **Ressourcengruppen** aus.
3. Wählen Sie die Ressourcengruppe **dp203-*xxxxxxx*** für Ihren Synapse Analytics-Arbeitsbereich aus (nicht die verwaltete Ressourcengruppe), und vergewissern Sie sich, dass sie den Synapse-Arbeitsbereich, das Speicherkonto und den Spark-Pool für Ihren Arbeitsbereich enthält.
4. Wählen Sie oben auf der Seite **Übersicht** für Ihre Ressourcengruppe die Option **Ressourcengruppe löschen** aus.
5. Geben Sie den Namen der Ressourcengruppe **dp203-*xxxxxxx*** ein, um zu bestätigen, dass Sie sie löschen möchten, und wählen Sie **Löschen** aus.

    Nach einigen Minuten werden die Ressourcengruppe in Ihrem Azure Synapse-Arbeitsbereich und die damit verknüpfte Ressourcengruppe im verwalteten Arbeitsbereich gelöscht.
