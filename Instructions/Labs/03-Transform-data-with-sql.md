---
lab:
  title: Transformieren von Daten mithilfe eines serverlosen SQL-Pools
  ilt-use: Lab
---

# Transformieren von Dateien mithilfe eines serverlosen SQL-Pools

Daten*analysten* verwenden häufig SQL, um Daten für Analyse und Berichterstellung abzufragen. Auch Daten*techniker* können SQL zum Bearbeiten und Transformieren von Daten verwenden, häufig als Teil einer Datenaufnahmepipeline oder eines Prozesses zum Extrahieren, Transformieren und Laden (ETL).

In dieser Übung verwenden Sie einen serverlosen SQL-Pool in Azure Synapse Analytics, um Daten in Dateien zu transformieren.

Diese Übung dauert ca. **30** Minuten.

## Vorbereitung

Sie benötigen ein [Azure-Abonnement](https://azure.microsoft.com/free), in dem Sie Administratorzugriff besitzen.

## Bereitstellen eines Azure Synapse Analytics-Arbeitsbereichs

Sie benötigen einen Azure Synapse Analytics-Arbeitsbereich mit Zugriff auf den Datenspeicher. Sie können den integrierten serverlosen SQL-Pool verwenden, um Dateien im Data Lake abzufragen.

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
    cd dp-203/Allfiles/labs/03
    ./setup.ps1
    ```

6. Wenn Sie dazu aufgefordert werden, wählen Sie aus, welches Abonnement Sie verwenden möchten (dies geschieht nur, wenn Sie Zugriff auf mehrere Azure-Abonnements haben).
7. Wenn Sie dazu aufgefordert werden, geben Sie ein geeignetes Kennwort ein, das für Ihren Azure Synapse SQL-Pool festgelegt werden soll.

    > **Hinweis**: Merken Sie sich unbedingt dieses Kennwort!

8. Warten Sie, bis das Skript abgeschlossen ist. Dies dauert in der Regel etwa 10 Minuten und in Ausnahmefällen auch länger. Während Sie warten, lesen Sie den Artikel [CETAS mit Synapse SQL](https://docs.microsoft.com/azure/synapse-analytics/sql/develop-tables-cetas) in der Dokumentation zu Azure Synapse Analytics.

## Abfragen von Daten in Dateien

Das Skript stellt einen Azure Synapse Analytics-Arbeitsbereich und ein Azure Storage-Konto zum Hosten des Data Lake bereit und lädt dann einige Dateien mit Daten in den Data Lake hoch.

### Anzeigen von Dateien im Data Lake

1. Wechseln Sie nach Abschluss des Skripts im Azure-Portal zur erstellten Ressourcengruppe **dp203-*xxxxxxx***, und wählen Sie Ihren Synapse-Arbeitsbereich aus.
2. Wählen Sie auf der Seite **Übersicht** für Ihren Synapse-Arbeitsbereich in der Karte **Synapse Studio öffnen** die Option **Öffnen** aus, um Synapse Studio in einer neuen Browserregisterkarte zu öffnen. Melden Sie sich an, wenn Sie dazu aufgefordert werden.
3. Verwenden Sie auf der linken Seite von Synapse Studio das Symbol **&rsaquo;&rsaquo;**, um das Menü zu erweitern. Dadurch werden die verschiedenen Seiten in Synapse Studio angezeigt, die Sie zum Verwalten von Ressourcen und zum Ausführen von Datenanalyseaufgaben verwenden.
4. Zeigen Sie auf der Seite **Daten** die Registerkarte **Verknüpft** an, und stellen Sie sicher, dass Ihr Arbeitsbereich einen Link zu Ihrem Azure Data Lake Storage Gen2-Speicherkonto enthält, dessen Name **synapse*xxxxxxx* (Primary - datalake*xxxxxxx*)** ähneln sollte.
5. Erweitern Sie Ihr Speicherkonto, und stellen Sie sicher, dass es einen Dateisystemcontainer mit dem Namen **Dateien** enthält.
6. Wählen Sie den Container **Dateien** aus und beachten Sie, dass er einen Ordner mit dem Namen **Sales** enthält. Dieser Ordner enthält die Datendateien, die Sie abfragen möchten.
7. Öffnen Sie den Ordner **Sales** und den darin enthaltenen **CSV**-Ordner und beachten Sie, dass dieser Ordner CSV-Dateien für drei Jahre Verkaufsdaten enthält.
8. Klicken Sie mit der rechten Maustaste auf eine der Dateien, und wählen Sie **Vorschau** aus, um die darin enthaltenen Daten anzuzeigen. Beachten Sie, dass die Dateien eine Kopfzeile enthalten.
9. Schließen Sie die Vorschau, und navigieren Sie dann mit der Schaltfläche **&#8593** zurück zum Ordner **Sales**.

### Verwenden von SQL zum Abfragen von CSV-Dateien

1. Wählen Sie den **CSV**-Ordner und dann in der Liste **Neues SQL-Skript** auf der Symbolleiste **Die ersten 100 Zeilen auswählen** aus.
2. Wählen Sie in der Liste **Dateityp** **Textformat** aus, und wenden Sie dann die Einstellungen an, um ein neues SQL-Skript zu öffnen, das die Daten im Ordner abfragt.
3. Ändern Sie im Bereich **Eigenschaften** für **SQL Skript 1**, das erstellt wird, den Namen in **Sales-CSV-Dateien** und ändern Sie die Ergebniseinstellungen so, dass **Alle Zeilen** angezeigt werden. Wählen Sie dann auf der Symbolleiste **Veröffentlichen** aus, um das Skript zu speichern und verwenden Sie die Schaltfläche **Eigenschaften** (ähnlich wie **&#128463;<sub>*</sub>**) am rechten Ende der Symbolleiste, um den Bereich **Eigenschaften** auszublenden.
4. Überprüfen Sie den generierten SQL Code, der in etwa wie der folgende aussehen sollte:

    ```SQL
    -- This is auto-generated code
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/sales/csv/**',
            FORMAT = 'CSV',
            PARSER_VERSION='2.0'
        ) AS [result]
    ```

    Dieser Code verwendet das OPENROWSET zum Lesen von Daten aus den CSV-Dateien im Sales-Ordner und ruft die ersten 100 Datenzeilen ab.

5. In diesem Fall enthalten die Datendateien die Spaltennamen in der ersten Zeile. Ändern Sie daher die Abfrage so, dass der Klausel ein `HEADER_ROW = TRUE` Parameter `OPENROWSET` hinzugefügt wird, wie hier gezeigt (vergessen Sie nicht, ein Komma nach dem vorherigen Parameter hinzuzufügen):

    ```SQL
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/sales/csv/**',
            FORMAT = 'CSV',
            PARSER_VERSION='2.0',
            HEADER_ROW = TRUE
        ) AS [result]
    ```

6. Vergewissern Sie sich, dass in der Liste **Verbinden mit** die Option **Integriert** ausgewählt ist. Dies entspricht dem integrierten SQL-Pool, der mit Ihrem Arbeitsbereich erstellt wurde. Verwenden Sie dann in der Symbolleiste die Schaltfläche **&#9655; Ausführen**, um den SQL-Code auszuführen, und überprüfen Sie die Ergebnisse, die in etwa wie folgt aussehen sollten:

    | SalesOrderNumber | SalesOrderLineNumber | OrderDate | CustomerName | EmailAddress | Element | Quantity (Menge) | UnitPrice (Stückpreis) | TaxAmount |
    | -- | -- | -- | -- | -- | -- | -- | -- | -- |
    | SO43701 | 1 | 01.07.2019 | Christy Zhu | christy12@adventure-works.com |Mountain-100 Silver, 44 | 1 | 3399.99 | 271.9992 |
    | ... | ... | ... | ... | ... | ... | ... | ... | ... |

7. Veröffentlichen Sie die Änderungen an Ihrem Skript, und schließen Sie dann den Skriptbereich.

## Transformieren von Daten mit CETAS-Anweisungen (CREATE EXTERNAL TABLE AS SELECT)

Eine einfache Möglichkeit, SQL zum Transformieren von Daten in einer Datei und zum Speichern der Ergebnisse in einer anderen Datei zu verwenden, besteht darin, eine CREATE EXTERNAL TABLE AS SELECT (CETAS)-Anweisung zu verwenden. Diese Anweisung erstellt eine Tabelle basierend auf den Anforderungen einer Abfrage, aber die Daten für die Tabelle werden als Dateien in einem Data Lake gespeichert. Die transformierten Daten können dann über die externe Tabelle abgefragt oder direkt im Dateisystem aufgerufen werden (z. B. zum Einfügen in einen nachgelagerten Prozess zum Laden der transformierten Daten in ein Data Warehouse).

### Erstellen einer externen Datenquelle und eines externen Dateiformats

Wenn Sie eine externe Datenquelle in einer Datenbank definieren, können Sie diese verwenden, um auf den Data Lake-Speicherort zu verweisen, an dem Sie Dateien für externe Tabellen speichern möchten. Mit einem externen Dateiformat können Sie das Format für diese Dateien definieren, z. B. Parquet oder CSV. Um diese Objekte für die Arbeit mit externen Tabellen zu verwenden, müssen Sie diese in einer anderen Datenbank als der Standarddatenbank **Master** erstellen.

1. Wählen Sie in Synapse Studio auf der Seite **Entwickeln** im Menü **+** **SQL Skript** aus.
2. Fügen Sie im neuen Skriptbereich den folgenden Code hinzu (ersetzen Sie *datalakexxxxxxxxx* durch den Namen Ihres Data Lake-Speicherkontos), um eine neue Datenbank zu erstellen und eine externe Datenquelle hinzuzufügen.

    ```sql
    -- Database for sales data
    CREATE DATABASE Sales
      COLLATE Latin1_General_100_BIN2_UTF8;
    GO;
    
    Use Sales;
    GO;
    
    -- External data is in the Files container in the data lake
    CREATE EXTERNAL DATA SOURCE sales_data WITH (
        LOCATION = 'https://datalakexxxxxxx.dfs.core.windows.net/files/'
    );
    GO;
    
    -- Format for table files
    CREATE EXTERNAL FILE FORMAT ParquetFormat
        WITH (
                FORMAT_TYPE = PARQUET,
                DATA_COMPRESSION = 'org.apache.hadoop.io.compress.SnappyCodec'
            );
    GO;
    ```

3. Ändern Sie die Skripteigenschaften, um ihren Namen in **Sales DB erstellen** zu ändern und zu veröffentlichen.
4. Stellen Sie sicher, dass das Skript mit dem **integrierten** SQL-Pool und der Datenbank **Master** verbunden ist, und führen Sie es dann aus.
5. Wechseln Sie zurück zur Seite **Daten**, und verwenden Sie die Schaltfläche **&#8635;** oben rechts in Synapse Studio, um die Seite zu aktualisieren. Zeigen Sie dann die Registerkarte **Arbeitsbereich** im Bereich **Daten** an, wo jetzt eine Liste **SQL-Datenbank** angezeigt wird. Erweitern Sie diese Liste, um zu überprüfen, ob die Datenbank **Sales** erstellt wurde.
6. Erweitern Sie die Datenbank **Sales**, den zugehörigen Ordner **Externe Ressourcen** und den Ordner **Externe Datenquellen**, um die von Ihnen erstellte externe Datenquelle **sales_data** anzuzeigen.

### Erstellen einer externen Tabelle

1. Wählen Sie in Synapse Studio auf der Seite **Entwickeln** im Menü **+** **SQL Skript** aus.
2. Fügen Sie im neuen Skriptbereich den folgenden Code hinzu, um Daten aus den CSV-Sales-Dateien mithilfe der externen Datenquelle abzurufen und zu aggregieren. Beachten Sie dabei, dass der Pfad **BULK** relativ zum Ordnerspeicherort ist, in dem die Datenquelle definiert ist:

    ```sql
    USE Sales;
    GO;
    
    SELECT Item AS Product,
           SUM(Quantity) AS ItemsSold,
           ROUND(SUM(UnitPrice) - SUM(TaxAmount), 2) AS NetRevenue
    FROM
        OPENROWSET(
            BULK 'sales/csv/*.csv',
            DATA_SOURCE = 'sales_data',
            FORMAT = 'CSV',
            PARSER_VERSION = '2.0',
            HEADER_ROW = TRUE
        ) AS orders
    GROUP BY Item;
    ```

3. Führen Sie das Skript aus. Das Ergebnis sollte dem folgenden ähneln:

    | Produkt | ItemsSold | NetRevenue |
    | -- | -- | -- |
    | AWC Logo Cap | 1063 | 8791.86 |
    | ... | ... | ... |

4. Ändern Sie den SQL-Code, um die Ergebnisse der Abfrage in einer externen Tabelle wie folgt zu speichern:

    ```sql
    CREATE EXTERNAL TABLE ProductSalesTotals
        WITH (
            LOCATION = 'sales/productsales/',
            DATA_SOURCE = sales_data,
            FILE_FORMAT = ParquetFormat
        )
    AS
    SELECT Item AS Product,
        SUM(Quantity) AS ItemsSold,
        ROUND(SUM(UnitPrice) - SUM(TaxAmount), 2) AS NetRevenue
    FROM
        OPENROWSET(
            BULK 'sales/csv/*.csv',
            DATA_SOURCE = 'sales_data',
            FORMAT = 'CSV',
            PARSER_VERSION = '2.0',
            HEADER_ROW = TRUE
        ) AS orders
    GROUP BY Item;
    ```

5. Führen Sie das Skript aus. Dieses Mal gibt es keine Ausgabe, aber der Code sollte eine externe Tabelle basierend auf den Ergebnissen der Abfrage erstellt haben.
6. Benennen Sie das Skript als **ProductSalesTotals-Tabelle erstellen** und veröffentlichen Sie es.
7. Zeigen Sie auf der Seite **Daten** auf der Registerkarte **Arbeitsbereich** den Inhalt des Ordners **Externe Tabellen** für die SQL-Datenbank **Sales** an, um zu überprüfen, ob eine neue Tabelle mit dem Namen **ProductSalesTotals** erstellt wurde.
8. Wählen Sie im Menü **…** für die Tabelle **ProductSalesTotals** die Option **Neues SQL-Skript** > **Die ersten 100 Zeilen auswählen** aus, und überprüfen Sie die Ergebnisse. Führen Sie dann das resultierende Skript aus, und stellen Sie sicher, dass es die aggregierten Produktumsatzdaten zurückgibt.
9. Zeigen Sie auf der Registerkarte **Dateien**, die das Dateisystem für Ihren Data Lake enthält, den Inhalt des Ordners **Sales** an (aktualisieren Sie die Ansicht bei Bedarf), und stellen Sie sicher, dass ein neuer Ordner **productsales** erstellt wurde.
10. Beachten Sie, dass im Ordner **productsales** eine oder mehrere Dateien erstellt wurden, deren Namen in etwa so aussehen: ABC123DE----.parquet. Diese Dateien enthalten die aggregierten Produktumsatzdaten. Um dies zu beweisen, können Sie eine der Dateien auswählen und das Menü **Neues SQL-Skript** > **Erste 100 Zeilen auswählen** verwenden, um sie direkt abzufragen.

## Kapseln von Datentransformationen in einer gespeicherten Prozedur

Wenn Sie Daten häufig transformieren müssen, können Sie eine gespeicherte Prozedur verwenden, um eine CETAS-Anweisung zu kapseln.

1. Wählen Sie in Synapse Studio auf der Seite **Entwickeln** im Menü **+** **SQL Skript** aus.
2. Fügen Sie im neuen Skriptbereich den folgenden Code hinzu, um eine gespeicherte Prozedur in der Datenbank **Sales** zu erstellen, die Umsätze nach Jahr aggregiert und die Ergebnisse in einer externen Tabelle speichert:

    ```sql
    USE Sales;
    GO;
    CREATE PROCEDURE sp_GetYearlySales
    AS
    BEGIN
        -- drop existing table
        IF EXISTS (
                SELECT * FROM sys.external_tables
                WHERE name = 'YearlySalesTotals'
            )
            DROP EXTERNAL TABLE YearlySalesTotals
        -- create external table
        CREATE EXTERNAL TABLE YearlySalesTotals
        WITH (
                LOCATION = 'sales/yearlysales/',
                DATA_SOURCE = sales_data,
                FILE_FORMAT = ParquetFormat
            )
        AS
        SELECT YEAR(OrderDate) AS CalendarYear,
                SUM(Quantity) AS ItemsSold,
                ROUND(SUM(UnitPrice) - SUM(TaxAmount), 2) AS NetRevenue
        FROM
            OPENROWSET(
                BULK 'sales/csv/*.csv',
                DATA_SOURCE = 'sales_data',
                FORMAT = 'CSV',
                PARSER_VERSION = '2.0',
                HEADER_ROW = TRUE
            ) AS orders
        GROUP BY YEAR(OrderDate)
    END
    ```

3. Führen Sie das Skript aus, um die gespeicherte Prozedur zu erstellen.
4. Fügen Sie unter dem soeben ausgeführten Code den folgenden Code hinzu, um die gespeicherte Prozedur aufzurufen:

    ```sql
    EXEC sp_GetYearlySales;
    ```

5. Wählen Sie nur die `EXEC sp_GetYearlySales;` soeben hinzugefügte Anweisung aus, und verwenden Sie die Schaltfläche **&#9655; Ausführen**, um sie auszuführen.
6. Zeigen Sie auf der Registerkarte **Dateien**, die das Dateisystem für Ihren Data Lake enthält, den Inhalt des Ordners **Sales** an (aktualisieren Sie die Ansicht bei Bedarf), und stellen Sie sicher, dass ein neuer Ordner **yearlysales** erstellt wurde.
7. Beachten Sie, dass im Ordner **yearlysales** eine Parquet-Datei mit den aggregierten Jahresumsatzdaten erstellt wurde.
8. Wechseln Sie zurück zum SQL-Skript, und führen Sie die `EXEC sp_GetYearlySales;` Anweisung erneut aus, und achten Sie auf das Auftreten eines Fehlers.

    Obwohl das Skript die externe Tabelle abbricht, wird der Ordner, der die Daten enthält, nicht gelöscht. Zum erneuten Ausführen der gespeicherten Prozedur (z. B. als Teil einer Pipeline für die geplante Datentransformation) müssen Sie die alten Daten löschen.

9. Wechseln Sie zurück zur Registerkarte **Dateien** und zeigen Sie den Ordner **Sales** an. Wählen Sie dann den Ordner **yearlysales** aus, und löschen Sie diesen.
10. Wechseln Sie zurück zum SQL-Skript, und führen Sie die `EXEC sp_GetYearlySales;` Anweisung erneut aus. Dieses Mal wird der Vorgang erfolgreich ausgeführt, und eine neue Datendatei wird generiert.

## Löschen von Azure-Ressourcen

Wenn Sie sich mit Azure Synapse Analytics vertraut gemacht haben, sollten Sie die erstellten Ressourcen löschen, um unnötige Azure-Kosten zu vermeiden.

1. Schließen Sie die Registerkarte mit Synapse Studio, und kehren Sie zum Azure-Portal zurück.
2. Wählen Sie auf der **Startseite** des Azure-Portals die Option **Ressourcengruppen** aus.
3. Wählen Sie die Ressourcengruppe **dp203-*xxxxxxx*** für Ihren Synapse Analytics-Arbeitsbereich aus (nicht die verwaltete Ressourcengruppe), und vergewissern Sie sich, dass sie den Synapse-Arbeitsbereich und das Speicherkonto für Ihren Arbeitsbereich enthält.
4. Wählen Sie oben auf der Seite **Übersicht** für Ihre Ressourcengruppe die Option **Ressourcengruppe löschen** aus.
5. Geben Sie den Namen der Ressourcengruppe **dp203-*xxxxxxx*** ein, um zu bestätigen, dass Sie sie löschen möchten, und wählen Sie **Löschen** aus.

    Nach einigen Minuten werden die Ressourcengruppe in Ihrem Azure Synapse-Arbeitsbereich und die damit verknüpfte Ressourcengruppe im verwalteten Arbeitsbereich gelöscht.
