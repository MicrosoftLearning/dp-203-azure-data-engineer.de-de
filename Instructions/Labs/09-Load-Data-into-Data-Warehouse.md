---
lab:
  title: Laden von Daten in ein relationales Data Warehouse
  ilt-use: Lab
---

# Laden von Daten in ein relationales Data Warehouse

In dieser Übung laden Sie Daten in einen dedizierten SQL-Pool.

Diese Übung dauert ca. **30** Minuten.

## Vorbereitung

Sie benötigen ein [Azure-Abonnement](https://azure.microsoft.com/free), in dem Sie Administratorzugriff besitzen.

## Bereitstellen eines Azure Synapse Analytics-Arbeitsbereichs

Sie benötigen einen Azure Synapse Analytics-Arbeitsbereich mit Zugriff auf den Data Lake-Speicher und einen dedizierten SQL-Pool, in dem ein Data Warehouse gehostet wird.

In dieser Übung verwenden Sie eine Kombination aus einem PowerShell-Skript und einer ARM-Vorlage, um einen Azure Synapse Analytics-Arbeitsbereich bereitzustellen.

1. Melden Sie sich beim [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com` an.
2. Verwenden Sie rechts neben der Suchleiste oben auf der Seite die Schaltfläche **[\>_]**, um eine neue Cloud Shell-Instanz im Azure-Portal zu erstellen. Wählen Sie eine ***PowerShell***-Umgebung aus, und erstellen Sie Speicher, falls Sie dazu aufgefordert werden. Die Cloud Shell bietet eine Befehlszeilenschnittstelle in einem Bereich am unteren Rand des Azure-Portals, wie hier gezeigt:

    ![Azure-Portal mit einem Cloud Shell-Bereich](./images/cloud-shell.png)

    > **Hinweis**: Wenn Sie zuvor eine Cloud Shell erstellt haben, die eine *Bash*-Umgebung verwendet, ändern Sie diese mithilfe des Dropdownmenüs oben links im Cloud Shell-Bereich zu ***PowerShell***.

3. Sie können die Größe der Cloud Shell durch Ziehen der Trennzeichenleiste oben im Bereich ändern oder den Bereich mithilfe der Symbole „—“, **&#9723;** und **X** oben rechts minimieren, maximieren und schließen. Weitere Informationen zur Verwendung von Azure Cloud Shell finden Sie in der [Azure Cloud Shell-Dokumentation](https://docs.microsoft.com/azure/cloud-shell/overview).

4. Geben Sie im PowerShell-Bereich die folgenden Befehle ein, um dieses Repository zu klonen:

    ```powershell
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Nachdem das Repository geklont wurde, geben Sie die folgenden Befehle ein, um in den Ordner für diese Übung zu wechseln. Führen Sie das darin enthaltene Skript **setup.ps1** aus:

    ```powershell
    cd dp-203/Allfiles/labs/09
    ./setup.ps1
    ```

6. Wenn Sie dazu aufgefordert werden, wählen Sie aus, welches Abonnement Sie verwenden möchten (dies geschieht nur, wenn Sie Zugriff auf mehrere Azure-Abonnements haben).
7. Wenn Sie dazu aufgefordert werden, geben Sie ein geeignetes Kennwort ein, das für Ihren Azure Synapse SQL-Pool festgelegt werden soll.

    > **Hinweis**: Merken Sie sich unbedingt dieses Kennwort!

8. Warten Sie, bis das Skript abgeschlossen ist. Dies dauert in der Regel etwa 10 Minuten, in einigen Fällen kann es jedoch länger dauern. Während Sie warten, lesen Sie den Artikel [Strategien zum Laden von Daten für dedizierten SQL-Pool in Azure Synapse Analytics](https://learn.microsoft.com/azure/synapse-analytics/sql-data-warehouse/design-elt-data-loading) in der Dokumentation zu Azure Synapse Analytics.

## Vorbereitung zum Laden von Daten

1. Wechseln Sie nach Abschluss des Skripts im Azure-Portal zur erstellten Ressourcengruppe **dp203-*xxxxxxx***, und wählen Sie ihren Synapse-Arbeitsbereich aus.
2. Wählen Sie auf der Seite **Übersicht** für Ihren Synapse-Arbeitsbereich in der Karte **Synapse Studio öffnen** die Option **Öffnen** aus, um Synapse Studio in einer neuen Browserregisterkarte zu öffnen. Melden Sie sich an, wenn Sie dazu aufgefordert werden.
3. Verwenden Sie im linken Bereich von Synapse Studio das Symbol ››, um das Menü zu erweitern. Dadurch werden die verschiedenen Seiten in Synapse Studio angezeigt, die Sie zur Verwaltung von Ressourcen und zur Durchführung von Datenanalyseaufgaben verwenden werden.
4. Wählen Sie auf der Seite **Verwalten** auf der Registerkarte **SQL-Pools** die Zeile für den dedizierten SQL-Pool **sql*xxxxxxx*** aus, der das Data Warehouse für diese Übung hostet, und klicken Sie auf das zugehörige Symbol **&#9655;**, um ihn zu starten. Bestätigen Sie, dass Sie ihn fortsetzen möchten, wenn Sie dazu aufgefordert werden.

    Das Fortsetzen eines Pools kann einige Minuten dauern. Sie können den Status mithilfe der Schaltfläche **&#8635; Aktualisieren** regelmäßig überprüfen. Der Status wird als **Online** angezeigt, wenn der SQL-Pool bereit ist. Fahren Sie in der Zwischenzeit mit den nachstehenden Schritten fort, um die Datendateien, die Sie laden werden, anzuzeigen.

5. Zeigen Sie auf der Seite **Daten** die Registerkarte **Verknüpft** an, und stellen Sie sicher, dass Ihr Arbeitsbereich einen Link zu Ihrem Azure Data Lake Storage Gen2-Speicherkonto enthält, dessen Name **synapsexxxxxxx (Primary - datalakexxxxx)** ähneln sollte.
6. Erweitern Sie Ihr Speicherkonto, und stellen Sie sicher, dass es einen Dateisystemcontainer mit dem Namen **files (primary)** enthält.
7. Wählen Sie den Dateicontainer aus, und beachten Sie, dass er einen Ordner mit dem Namen **data** enthält. Dieser Ordner enthält die Datendateien, die Sie in das Data Warehouse laden werden.
8. Öffnen Sie den Ordner **data**, und beachten Sie, dass er CSV-Dateien mit Kunden- und Produktdaten enthält.
9. Klicken Sie mit der rechten Maustaste auf eine der Dateien, und wählen Sie **Vorschau** aus, um die darin enthaltenen Daten anzuzeigen. Beachten Sie, dass die Dateien eine Kopfzeile enthalten, damit Sie die Option zum Anzeigen von Spaltenüberschriften auswählen können.
10. Kehren Sie zur Seite **Verwalten** zurück, und stellen Sie sicher, dass Ihr dedizierter SQL-Pool online ist.

## Laden von Data Warehouse-Tabellen

Sehen wir uns einige SQL-basierte Ansätze zum Laden von Daten in das Data Warehouse an.

1. Wählen Sie auf der Seite **Daten** die Registerkarte **Arbeitsbereich** aus.
2. Erweitern Sie **SQL-Datenbank**, und wählen Sie Ihre Datenbank **sql*xxxxxxx*** aus. Wählen Sie dann im Menü **…** die Option **Neues SQL-Skript** > 
**Leeres Skript** aus.

Sie haben nun eine leere SQL-Seite, die mit der Instanz für die folgenden Übungen verbunden ist. Sie verwenden dieses Skript, um verschiedene SQL-Techniken zu untersuchen, mit denen Sie Daten laden können.

### Laden von Daten aus einem Data Lake mithilfe der COPY-Anweisung

1. Geben Sie in Ihrem SQL-Skript den folgenden Code in das Fenster ein.

    ```sql
    SELECT COUNT(1) 
    FROM dbo.StageProduct
    ```

2. Verwenden Sie auf der Symbolleiste die Schaltfläche **&#9655; Ausführen** aus, um den SQL-Code auszuführen, und vergewissern Sie sich, dass derzeit **0** Zeilen in der Tabelle **StageProduct** enthalten sind.
3. Ersetzen Sie den Code durch die folgende COPY-Anweisung (und ersetzen Sie **datalake*xxxxxx*** durch den Namen Ihres Data Lake):

    ```sql
    COPY INTO dbo.StageProduct
        (ProductID, ProductName, ProductCategory, Color, Size, ListPrice, Discontinued)
    FROM 'https://datalakexxxxxx.blob.core.windows.net/files/data/Product.csv'
    WITH
    (
        FILE_TYPE = 'CSV',
        MAXERRORS = 0,
        IDENTITY_INSERT = 'OFF',
        FIRSTROW = 2 --Skip header row
    );


    SELECT COUNT(1) 
    FROM dbo.StageProduct
    ```

4. Führen Sie das Skript aus, und überprüfen Sie die Ergebnisse. 11 Zeilen sollten in die Tabelle **StageProduct** geladen worden sein.

    Nun verwenden wir die gleiche Technik zum Laden einer anderen Tabelle, und protokollieren diesmal Fehler, die auftreten könnten.

5. Ersetzen Sie den SQL-Code im Skriptbereich durch den folgenden Code, und ändern Sie **datalake*xxxxxx*** sowohl in der Klausel ```FROM``` als auch in der Klausel ```ERRORFILE``` in den Namen Ihres Data Lake:

    ```sql
    COPY INTO dbo.StageCustomer
    (GeographyKey, CustomerAlternateKey, Title, FirstName, MiddleName, LastName, NameStyle, BirthDate, 
    MaritalStatus, Suffix, Gender, EmailAddress, YearlyIncome, TotalChildren, NumberChildrenAtHome, EnglishEducation, 
    SpanishEducation, FrenchEducation, EnglishOccupation, SpanishOccupation, FrenchOccupation, HouseOwnerFlag, 
    NumberCarsOwned, AddressLine1, AddressLine2, Phone, DateFirstPurchase, CommuteDistance)
    FROM 'https://datalakexxxxxx.dfs.core.windows.net/files/data/Customer.csv'
    WITH
    (
    FILE_TYPE = 'CSV'
    ,MAXERRORS = 5
    ,FIRSTROW = 2 -- skip header row
    ,ERRORFILE = 'https://datalakexxxxxx.dfs.core.windows.net/files/'
    );
    ```

6. Führen Sie das Skript aus, und überprüfen Sie die darauf folgende Nachricht. Die Quelldatei enthält eine Zeile mit ungültigen Daten, sodass eine Zeile abgelehnt wird. Der obige Code gibt einen Höchstwert von **5** Fehlern an, sodass ein einziger Fehler nicht das Laden der gültigen Zeilen verhindert haben sollte. Sie können die *geladenen* Zeilen anzeigen, indem Sie die folgende Abfrage ausführen.

    ```sql
    SELECT *
    FROM dbo.StageCustomer
    ```

7. Zeigen Sie auf der Registerkarte **Dateien** den Stammordner Ihres Data Lake an, und überprüfen Sie, ob ein neuer Ordner mit dem Namen **_rejectedrows** erstellt wurde (wenn dieser Ordner nicht angezeigt wird, wählen Sie im Menü **Mehr** die Option **Aktualisieren** aus, um die Ansicht zu aktualisieren).
8. Öffnen Sie den Ordner **_rejectedrows** sowie den darin enthaltenen Unterordner mit Datum und Uhrzeit, und beachten Sie, dass Dateien erstellt wurden, deren Namen ***QID123_1_2*.Error.Txt** und ***QID123_1_2*.Row.Txt** ähneln. Sie können auf jede dieser Dateien mit der rechten Maustaste klicken und **Vorschau** auswählen, um Details des Fehlers und der abgelehnten Zeile anzuzeigen.

    Die Verwendung von Stagingtabellen ermöglicht es Ihnen, Daten zu validieren oder zu transformieren, bevor Sie sie verschieben oder an vorhandene Dimensionstabellen anfügen bzw. ein Upsert ausführen. Die COPY-Anweisung bietet eine einfache, aber leistungsstarke Technik, mit der Sie Daten aus Dateien in einem Data Lake einfach in Stagingtabellen laden und – wie Sie gesehen haben – ungültige Zeilen identifizieren und umleiten können.

### Verwenden einer CREATE TABLE AS-Anweisung (CTAS)

1. Kehren Sie zum Skriptbereich zurück, und ersetzen Sie den darin enthaltenen Code durch den folgenden Code:

    ```sql
    CREATE TABLE dbo.DimProduct
    WITH
    (
        DISTRIBUTION = HASH(ProductAltKey),
        CLUSTERED COLUMNSTORE INDEX
    )
    AS
    SELECT ROW_NUMBER() OVER(ORDER BY ProductID) AS ProductKey,
        ProductID AS ProductAltKey,
        ProductName,
        ProductCategory,
        Color,
        Size,
        ListPrice,
        Discontinued
    FROM dbo.StageProduct;
    ```

2. Führen Sie das Skript aus, das eine neue Tabelle namens **DimProduct** aus den eingesetzten Produktdaten erstellt, die **ProductAltKey** als Hashverteilungsschlüssel verwendet und über einen gruppierten Columnstore-Index verfügt.
4. Verwenden Sie die folgende Abfrage, um den Inhalt der neuen Tabelle **DimProduct** anzuzeigen.

    ```sql
    SELECT ProductKey,
        ProductAltKey,
        ProductName,
        ProductCategory,
        Color,
        Size,
        ListPrice,
        Discontinued
    FROM dbo.DimProduct;
    ```

    Der Ausdruck „CREATE TABLE AS SELECT“ (CTAS) hat verschiedene Verwendungsmöglichkeiten, darunter folgende:

    - Neuverteilen des Hashschlüssels einer Tabelle, um sie mit anderen Tabellen zu synchronisieren und dadurch eine bessere Abfrageleistung zu erzielen.
    - Zuweisen eines Ersatzschlüssels zu einer Stagingtabelle basierend auf vorhandenen Werten nach der Durchführung einer Delta-Analyse.
    - Schnelles Erstellen von aggregierten Tabellen für Berichtszwecke.

### Kombinieren von INSERT- und UPDATE-Anweisungen zum Laden einer langsam veränderlichen Dimensionstabelle

Die Tabelle **DimCustomer** unterstützt langsam veränderliche Dimensionen (Slowly Changing Dimensions, SCDs) des Typ 1 und Typ 2, wobei Änderungen des Typ 1 ein direktes Update einer vorhandenen Zeile zur Folge haben, und Änderungen des Typ 2 eine neue Zeile, um die neueste Version einer bestimmten Dimensionsentitätsinstanz anzugeben. Das Laden dieser Tabelle erfordert eine Kombination aus INSERT-Anweisungen (zum Laden neuer Kundinnen und Kunden) und UPDATE-Anweisungen (zum Anwenden von Änderungen des Typ 1 oder Typ 2).

1. Ersetzen Sie den vorhandenen SQL-Code im Abfragebereich durch den folgenden Code:

    ```sql
    INSERT INTO dbo.DimCustomer ([GeographyKey],[CustomerAlternateKey],[Title],[FirstName],[MiddleName],[LastName],[NameStyle],[BirthDate],[MaritalStatus],
    [Suffix],[Gender],[EmailAddress],[YearlyIncome],[TotalChildren],[NumberChildrenAtHome],[EnglishEducation],[SpanishEducation],[FrenchEducation],
    [EnglishOccupation],[SpanishOccupation],[FrenchOccupation],[HouseOwnerFlag],[NumberCarsOwned],[AddressLine1],[AddressLine2],[Phone],
    [DateFirstPurchase],[CommuteDistance])
    SELECT *
    FROM dbo.StageCustomer AS stg
    WHERE NOT EXISTS
        (SELECT * FROM dbo.DimCustomer AS dim
        WHERE dim.CustomerAlternateKey = stg.CustomerAlternateKey);

    -- Type 1 updates (change name, email, or phone in place)
    UPDATE dbo.DimCustomer
    SET LastName = stg.LastName,
        EmailAddress = stg.EmailAddress,
        Phone = stg.Phone
    FROM DimCustomer dim inner join StageCustomer stg
    ON dim.CustomerAlternateKey = stg.CustomerAlternateKey
    WHERE dim.LastName <> stg.LastName OR dim.EmailAddress <> stg.EmailAddress OR dim.Phone <> stg.Phone

    -- Type 2 updates (address changes triggers new entry)
    INSERT INTO dbo.DimCustomer
    SELECT stg.GeographyKey,stg.CustomerAlternateKey,stg.Title,stg.FirstName,stg.MiddleName,stg.LastName,stg.NameStyle,stg.BirthDate,stg.MaritalStatus,
    stg.Suffix,stg.Gender,stg.EmailAddress,stg.YearlyIncome,stg.TotalChildren,stg.NumberChildrenAtHome,stg.EnglishEducation,stg.SpanishEducation,stg.FrenchEducation,
    stg.EnglishOccupation,stg.SpanishOccupation,stg.FrenchOccupation,stg.HouseOwnerFlag,stg.NumberCarsOwned,stg.AddressLine1,stg.AddressLine2,stg.Phone,
    stg.DateFirstPurchase,stg.CommuteDistance
    FROM dbo.StageCustomer AS stg
    JOIN dbo.DimCustomer AS dim
    ON stg.CustomerAlternateKey = dim.CustomerAlternateKey
    AND stg.AddressLine1 <> dim.AddressLine1;
    ```

2. Führen Sie das Skript aus, und überprüfen Sie die Ausgabe.

## Durchführen der Optimierung nach dem Laden

Nach dem Laden neuer Daten in das Data Warehouse empfiehlt es sich, die Indizes der Tabellenspalten neu zu erstellen und die Statistiken für häufig abgefragte Spalten zu aktualisieren.

1. Ersetzen Sie den Code im Skriptbereich durch den folgenden Code:

    ```sql
    ALTER INDEX ALL ON dbo.DimProduct REBUILD;
    ```

2. Führen Sie das Skript aus, um die Indizes in der Tabelle **DimProduct** neu zu erstellen.
3. Ersetzen Sie den Code im Skriptbereich durch den folgenden Code:

    ```sql
    CREATE STATISTICS customergeo_stats
    ON dbo.DimCustomer (GeographyKey);
    ```

4. Führen Sie das Skript aus, um Statistiken in der Spalte **GeographyKey** der Tabelle **DimCustomer** zu erstellen oder zu aktualisieren.

## Löschen von Azure-Ressourcen

Wenn Sie sich mit Azure Synapse Analytics vertraut gemacht haben, sollten Sie die erstellten Ressourcen löschen, um unnötige Azure-Kosten zu vermeiden.

1. Schließen Sie die Registerkarte mit Synapse Studio, und kehren Sie zum Azure-Portal zurück.
2. Wählen Sie auf der **Startseite** des Azure-Portals die Option **Ressourcengruppen** aus.
3. Wählen Sie die Ressourcengruppe **dp203-*xxxxxxx*** für Ihren Synapse Analytics-Arbeitsbereich aus (nicht die verwaltete Ressourcengruppe), und vergewissern Sie sich, dass sie den Synapse-Arbeitsbereich, das Speicherkonto und den Spark-Pool für Ihren Arbeitsbereich enthält.
4. Wählen Sie oben auf der Seite **Übersicht** für Ihre Ressourcengruppe die Option **Ressourcengruppe löschen** aus.
5. Geben Sie den Namen der Ressourcengruppe **dp203-*xxxxxxx*** ein, um zu bestätigen, dass Sie sie löschen möchten, und wählen Sie **Löschen** aus.

    Nach einigen Minuten werden die Ressourcengruppe in Ihrem Azure Synapse-Arbeitsbereich und die damit verknüpfte Ressourcengruppe im verwalteten Arbeitsbereich gelöscht.
