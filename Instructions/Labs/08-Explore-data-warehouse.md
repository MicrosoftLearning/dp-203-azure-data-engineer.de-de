---
lab:
  title: Erkunden eines relationalen Data Warehouse
  ilt-use: Suggested demo
---

# Erkunden eines relationalen Data Warehouse

Azure Synapse Analytics basiert auf einer skalierbaren Set-Funktion zur Unterstützung von Enterprise Data Warehouse; einschließlich dateibasierter Datenanalysen in einem Data Lake sowie relationalen Data Warehouses im großen Maßstab sowie der zum Laden verwendeten Datenübertragungs- und Transformationspipelinen. In dieser Übung erfahren Sie, wie Sie einen dedizierten SQL-Pool in Azure Synapse Analytics verwenden, um Daten in einem relationalen Data Warehouse zu speichern und abzufragen.

Dieses Lab dauert ungefähr **45** Minuten.

## Vor der Installation

Sie benötigen ein [Azure-Abonnement](https://azure.microsoft.com/free), in dem Sie Administratorzugriff besitzen.

## Bereitstellen eines Azure Synapse Analytics-Arbeitsbereichs

Ein Azure Synapse Analytics- *Arbeitsbereich* bietet einen zentralen Punkt für die Verwaltung von Daten und Datenverarbeitungslaufzeiten. Sie können einen Arbeitsbereich mithilfe der interaktiven Benutzeroberfläche im Azure-Portal bereitstellen, oder Sie können einen Arbeitsbereich und Ressourcen darin mithilfe eines Skripts oder einer Vorlage bereitstellen. In den meisten Produktionsszenarien empfiehlt es sich, die Bereitstellung mit den Skripts und Vorlagen zu automatisieren, damit Sie die Ressourcenbereitstellung in einen wiederholbaren *DevOps-Prozess* (Development/Operations, Entwicklung/Betrieb) integrieren können.

In dieser Übung verwenden Sie eine Kombination aus einem PowerShell-Skript und einer ARM-Vorlage, um Azure Synapse Analytics bereitzustellen.

1. Melden Sie sich beim [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com` an.
2. Verwenden Sie rechts neben der Suchleiste oben auf der Seite die Schaltfläche **[\>_]** , um eine neue Cloud Shell-Instanz im Azure-Portal zu erstellen. Wählen Sie eine ***PowerShell***-Umgebung aus, und erstellen Sie Speicher, falls Sie dazu aufgefordert werden. Die Cloud Shell bietet eine Befehlszeilenschnittstelle in einem Bereich am unteren Rand des Azure-Portals, wie hier gezeigt:

    ![Azure-Portal mit einem Cloud Shell-Bereich](./images/cloud-shell.png)

    > **Hinweis**: Wenn Sie zuvor eine Cloudshell erstellt haben, die eine *Bash-Umgebung* verwendet, verwenden Sie das Dropdownmenü oben links im Bereich der Cloudshell, um sie in*** Power Shell ***zu ändern.

3. Beachten Sie, dass Sie die Größe der Cloud Shell durch Ziehen der Trennzeichenleiste oben im Bereich ändern können oder den Bereich mithilfe der Symbole **&#8212;**, **&#9723;** und **X** oben rechts minimieren, maximieren und schließen können. Weitere Informationen zur Verwendung von Azure Cloud Shell finden Sie in der [Azure Cloud Shell-Dokumentation](https://docs.microsoft.com/azure/cloud-shell/overview).

4. Geben Sie im PowerShell Bereich die folgenden Befehle ein, um dieses Repository zu klonen:

    ```
    rm -r dp203 -f
    git clone  https://github.com/MicrosoftLearning/Dp-203-azure-data-engineer dp203
    ```

5. Nachdem das Repository geklont wurde, geben Sie die folgenden Befehle ein, um in den Ordner für dieses Lab zu wechseln. Führen Sie das darin enthaltene Skript **setup.ps1** aus:

    ```
    cd dp203/Allfiles/labs/08
    ./setup.ps1
    ```

6. Wenn Sie dazu aufgefordert werden, wählen Sie aus, welches Abonnement Sie verwenden möchten (dies geschieht nur, wenn Sie Zugriff auf mehrere Azure-Abonnements haben).
7. Wenn Sie dazu aufgefordert werden, geben Sie ein geeignetes Kennwort ein, das für Ihren Azure Synapse SQL-Pool festgelegt werden soll.

    > **Hinweis**: Merken Sie sich unbedingt das Kennwort!

8. Warten Sie, bis das Skript abgeschlossen ist – dies dauert in der Regel etwa 15 Minuten, in einigen Fällen kann es jedoch länger dauern. Während Sie warten, lesen Sie den Artiekl [Was ist dedizierter SQL-Pool in Azure Synapse Analytics?](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is) in der Dokumentation zu Azure Synapse Analytics.

## Erkunden des Data Warehouse-Schemas

Ein dedizierter SQL Pool wird für Data Warehouses in Azure Synapse Analytics verwendet.

### Starten des dedizierten SQL-Pools

1. Wechseln Sie nach Abschluss des Skripts im Azure-Portal zur erstellten Ressourcengruppe **dp203-*xxxxxxx***, und wählen Sie Ihren Synapse-Arbeitsbereich aus.
2. Wählen Sie auf der Seite **Übersicht** für Ihren Synapse-Arbeitsbereich auf der Karte **Open Synapse Studio** die Option**Öffnen** aus, um Synapse Studio auf einer neuen Browserregisterkarte zu öffnen.
3. Verwenden Sie im linken Bereich von Synapse Studio das Symbol **&rsaquo;&rsaquo;** , um das Menü zu erweitern. Dadurch werden die verschiedenen Seiten in Synapse Studio angezeigt, die Sie zur Verwaltung von Ressourcen und zur Durchführung von Datenanalyseaufgaben verwenden werden.
4. Stellen Sie auf der Seite **Verwalten** sicher, dass die Registerkarte **SQL-Pools** ausgewählt ist, und wählen Sie dann den dedizidierten SQL-Pool ***xxxxx*** aus, und verwenden Sie das zugehörige Symbol **▷**, um ihn zu starten. Bestätigen Sie, dass Sie ihn fortsetzen möchten, wenn Sie dazu aufgefordert werden.
5. Warten Sie, bis der SQL-Pool fortgesetzt wird. Dies kann einige Minuten dauern. Verwenden Sie die Schaltfläche **↻ Aktualisieren**  um den Status regelmäßig zu überprüfen. Der Status wird als **Online** angezeigt, wenn er bereit ist.

### Anzeigen der Tabellen in der Datenbank

1. Wählen Sie in Synapse Studio die Seite **Daten** aus, und stellen Sie sicher, dass die Registerkarte **Arbeitsbereich** ausgewählt ist und eine **SQL-Datenbankkategorie** enthält.
2. Erweitern Sie **die SQL-Datenbank**, den **SQL*xxxxxxx***-Pool und den zugehörigen **Tabellenordner** , um die Tabellen in der Datenbank anzuzeigen.

    Ein relationales Data Warehouse besteht in der Regel aus *Fakt-* und *Dimensionstabellen*. Die Tabellen sind für analytische Abfragen optimiert, in denen numerische Metriken in den Faktentabellen durch Attribute der Entitäten aggregiert werden, die durch die Dimensionstabellen dargestellt werden , z. B. die Möglichkeit, Internetumsatz nach Produkt, Kunde, Datum usw. zu aggregieren.
    
3. Erweitern Sie die Tabelle **Dbo. FactInternetSales** und deren Ordner **Spalten** , um die Spalten in dieser Tabelle anzuzeigen. Beachten Sie, dass viele der Spalten * Schlüssel*sind, die auf Zeilen in den Dimensionstabellen verweisen. Andere Werte sind numerische Werte (*Measures*) für die Analyse.
    
    Die Schlüssel werden verwendet, um eine Faktentabelle mit einer oder mehreren Bemaßungstabellen zu verknüpfen, häufig in einem *Sternschema* ; in dem die Faktentabelle direkt mit jeder Dimensionstabelle verknüpft ist (bilden einen mehrzackigen "Stern" mit der Faktentabelle in der Mitte).

4. Zeigen Sie die Spalten für die Tabelle **dbo.DimPromotion** und beachten Sie, dass sie über einen eindeutigen **PromotionKey** verfügt, der jede Zeile in der Tabelle eindeutig identifiziert. Sie verfügt auch über einen **AlternateKey**.

    In der Regel wurden Daten in einem Data Warehouse aus einer oder mehreren Transaktionsquellen importiert. Der *alternative* Schlüssel gibt den Geschäftsbezeichner für die Instanz dieser Entität in der Quelle wieder, aber ein eindeutiger numerischer *Ersatzschlüssel* wird normalerweise generiert, um jede Zeile in der Data Warehouse-Dimensionstabelle eindeutig zu identifizieren. Einer der Vorteile dieses Ansatzes besteht darin, dass das Data Warehouse mehrere Instanzen derselben Entität zu verschiedenen Zeitpunkten enthalten kann (z. B. Datensätze für denselben Kunden, die ihre Adresse zum Zeitpunkt der Bestellung widerspiegeln).

5. Zeigen Sie die Spalten für **dbo.DimProduct**an, und beachten, dass sie eine Spalte **ProductSubcategoryKey** enthalten, die auf die Tabelle **dbo.DimProductSubcategory**verweist, die wiederum eine Spalte **ProductCategoryKey** enthält, die auf die Tabelle  **dbo.DimProductCategory** verweist.

    In einigen Fällen werden Dimensionen teilweise in mehrere verknüpfte Tabellen normalisiert, um unterschiedliche Granularitätsebenen zu ermöglichen , z. B. Produkte, die in Unterkategorien und Kategorien gruppiert werden können. Dies führt dazu, dass ein einfacher Stern auf ein *Schneeflockenschema* erweitert wird, in dem die zentrale Faktentabelle mit einer Bemaßungstabelle verknüpft ist, die sich auf weitere Dimensionstabellen bezieht.

6. Zeigen Sie die Spalten für die Tabelle **Dbo.DimDate** an und beachten Sie, dass sie mehrere Spalten enthält, die unterschiedliche zeitliche Attribute eines Datums widerspiegeln – einschließlich Wochentag, Monat, Monat, Jahr, Tagname, Monatsname usw.

    Zeitabmessungen in einem Data Warehouse werden in der Regel als Dimensionstabelle implementiert, die eine Zeile für jede der kleinsten zeitlichen Einheiten der Granularität enthält (häufig als*Grain* der Dimension bezeichnet) mit der Sie die Measures in den Faktentabellen aggregieren möchten. In diesem Fall ist das niedrigste Grain, bei dem Measures aggregiert werden können, ein einzelnes Datum, und die Tabelle enthält eine Zeile für jedes Datum von der ersten bis zum letzten Datum, auf das in den Daten verwiesen wird. Mit den Attributen in der Tabelle **DimDate** können Analysten Measures basierend auf einem beliebigen Datumsschlüssel in der Faktentabelle mithilfe einer konsistenten Gruppe zeitlicher Attribute aggregieren (z. B. anzeigen von Bestellungen nach Monat basierend auf dem Bestelldatum). Die Tabelle **FactInternetSales** enthält drei Schlüssel, die sich auf die Tabelle **DimDate**: **OrderDateKey**, **DueDateKey**, and **ShipDateKey** bezieht.

## Abfragen von Data Warehouse-Tabellen

Nachdem Sie nun einige der wichtigeren Aspekte des Data Warehouse-Schemas untersucht haben, können Sie die Tabellen abfragen und einige Daten abrufen.

### Abfragen von Fakten- und Dimensionstabellen

Numerische Werte in einem relationalen Data Warehouse werden in Faktentabellen mit verknüpften Dimensionstabellen gespeichert, mit denen Sie die Daten über mehrere Attribute aggregieren können. Dieses Design bedeutet, dass die meisten Abfragen in einem relationalen Data Warehouse das Aggregieren und Gruppieren von Daten (mithilfe von Aggregatfunktionen und GROUP BY-Klauseln) für verwandte Tabellen (mithilfe von JOIN-Klauseln) umfassen.

1. Wählen Sie auf der Seite **Daten** den **sql*xxxxxxx*** SQL Pool und sein Menü **...**, wählen Sie **Neues SQL-Skript** > **Leeres Skript**.
2. Wenn eine neue Registerkarte **SQL Script 1** geöffnet wird, ändern Sie im Bereich **Eigenschaften** den Namen des Skripts in  **Internetumsatz analysieren** und ändern Sie die **Ergebniseinstellungen pro Abfrage** , um alle Zeilen zurückzugeben. Verwenden Sie dann die Schaltfläche **Veröffentlichen** auf der Symbolleiste, um das Skript zu speichern, und verwenden die Schaltfläche **Eigenschaften** (die aussieht wie **.**) auf der rechten Seite der Symbolleiste, um den Bereich **Eigenschaften** zu schließen, sodass Sie den Skriptbereich sehen können.
3. Fügen Sie dem Skript  den folgenden Code hinzu.

    ```sql
    SELECT  d.CalendarYear AS Year,
            SUM(i.SalesAmount) AS InternetSalesAmount
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    GROUP BY d.CalendarYear
    ORDER BY Year;
    ```

4. Verwenden Sie die Schaltfläche **▷ Ausführen**, um die Abfrage auszuführen und die Ergebnisse anzuzeigen, die den Gesamtumsatz für jedes Jahr anzeigen sollten. Diese Abfrage verknüpft die Faktentabelle für Internetverkäufe zu einer Zeitdimensionstabelle basierend auf dem Bestelldatum und aggregiert das Verkaufsbetragsmaß in der Faktentabelle nach dem Kalendermonatsattribut der Dimensionstabelle.

5. Ändern Sie die Abfrage wie folgt, um das Monatsattribut aus der Zeitdimension hinzuzufügen, und führen Sie dann die geänderte Abfrage aus.

    ```sql
    SELECT  d.CalendarYear AS Year,
            d.MonthNumberOfYear AS Month,
            SUM(i.SalesAmount) AS InternetSalesAmount
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    GROUP BY d.CalendarYear, d.MonthNumberOfYear
    ORDER BY Year, Month;
    ```

    Beachten Sie, dass Sie mit den Attributen in der Zeitdimension die Measures in der Faktentabelle auf mehreren Hierarchieebenen – in diesem Fall Jahr und Monat – aggregieren können. Dies ist ein gängiges Muster in Data Warehouses.

6. Ändern Sie die Abfrage wie folgt, um den Monat zu entfernen und der Aggregation eine zweite Dimension hinzuzufügen, und führen Sie sie dann aus, um die Ergebnisse anzuzeigen (die jährliche Internetumsatzsummen für jede Region anzeigen):

    ```sql
    SELECT  d.CalendarYear AS Year,
            g.EnglishCountryRegionName AS Region,
            SUM(i.SalesAmount) AS InternetSalesAmount
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    JOIN DimCustomer AS c ON i.CustomerKey = c.CustomerKey
    JOIN DimGeography AS g ON c.GeographyKey = g.GeographyKey
    GROUP BY d.CalendarYear, g.EnglishCountryRegionName
    ORDER BY Year, Region;
    ```

    Beachten Sie, dass es sich bei der Geografie um eine *Schneeflocken*-Dimension handelt, die mit der Internet-Verkaufs-Faktentabelle über die Kundendimension verknüpft ist. Daher benötigen Sie zwei Verknüpfungen in der Abfrage, um Internetverkäufe nach Geografie zu aggregieren.

7. Ändern sie die Abfrage, und führen Sie sie erneut aus, um eine weitere Schneeflockendimension hinzuzufügen und die jährliche regionale Umsätze nach Produktkategorie zu aggregieren:

    ```sql
    SELECT  d.CalendarYear AS Year,
            pc.EnglishProductCategoryName AS ProductCategory,
            g.EnglishCountryRegionName AS Region,
            SUM(i.SalesAmount) AS InternetSalesAmount
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    JOIN DimCustomer AS c ON i.CustomerKey = c.CustomerKey
    JOIN DimGeography AS g ON c.GeographyKey = g.GeographyKey
    JOIN DimProduct AS p ON i.ProductKey = p.ProductKey
    JOIN DimProductSubcategory AS ps ON p.ProductSubcategoryKey = ps.ProductSubcategoryKey
    JOIN DimProductCategory AS pc ON ps.ProductCategoryKey = pc.ProductCategoryKey
    GROUP BY d.CalendarYear, pc.EnglishProductCategoryName, g.EnglishCountryRegionName
    ORDER BY Year, ProductCategory, Region;
    ```

    Diesmal erfordert die Schneeflockendimension für die Produktkategorie drei Verknüpfungen, um die hierarchische Beziehung zwischen Produkten, Unterkategorien und Kategorien widerzuspiegeln.

8. Veröffentlichen Sie das Skript, um es zu speichern.

### Rangfolgefunktionen verwenden

Eine weitere häufige Anforderung beim Analysieren großer Datenmengen besteht darin, die Daten nach Partitionen zu gruppieren und den *Rang* jeder Entität in der Partition basierend auf einer bestimmten Metrik zu bestimmen.

1. Fügen Sie unter der vorhandenen Abfrage die folgende SQL hinzu, um Verkaufswerte für 2022 über Partitionen basierend auf dem Namen des Landes/der Region abzurufen:

    ```sql
    SELECT  g.EnglishCountryRegionName AS Region,
            ROW_NUMBER() OVER(PARTITION BY g.EnglishCountryRegionName
                              ORDER BY i.SalesAmount ASC) AS RowNumber,
            i.SalesOrderNumber AS OrderNo,
            i.SalesOrderLineNumber AS LineItem,
            i.SalesAmount AS SalesAmount,
            SUM(i.SalesAmount) OVER(PARTITION BY g.EnglishCountryRegionName) AS RegionTotal,
            AVG(i.SalesAmount) OVER(PARTITION BY g.EnglishCountryRegionName) AS RegionAverage
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    JOIN DimCustomer AS c ON i.CustomerKey = c.CustomerKey
    JOIN DimGeography AS g ON c.GeographyKey = g.GeographyKey
    WHERE d.CalendarYear = 2022
    ORDER BY Region;
    ```

2. Wählen Sie nur den neuen Abfragecode aus, und verwenden Sie die Schaltfläche**▷ Ausführen**, um ihn auszuführen. Überprüfen Sie dann die Ergebnisse, die ähnlich wie in der folgenden Tabelle aussehen sollten:

    | Region | RowNumber | OrderNo | LineItem | SalesAmount | RegionTotal | RegionAverage |
    |--|--|--|--|--|--|--|
    |Australien|1|SO73943|2|2.2900|2172278.7900|375.8918|
    |Australien|2|SO74100|4|2.2900|2172278.7900|375.8918|
    |...|...|...|...|...|...|...|
    |Australien|5779|SO64284|1|2443.3500|2172278.7900|375.8918|
    |Kanada|1|SO66332|2|2.2900|563177.1000|157.8411|
    |Kanada|2|SO68234|2|2.2900|563177.1000|157.8411|
    |...|...|...|...|...|...|...|
    |Kanada|3568|SO70911|1|2443.3500|563177.1000|157.8411|
    |Frankreich|1|SO68226|3|2.2900|816259.4300|315.4016|
    |Frankreich|2|SO63460|2|2.2900|816259.4300|315.4016|
    |...|...|...|...|...|...|...|
    |Frankreich|2588|SO69100|1|2443.3500|816259.4300|315.4016|
    |Deutschland|1|SO70829|3|2.2900|922368.2100|352.4525|
    |Deutschland|2|SO71651|2|2.2900|922368.2100|352.4525|
    |...|...|...|...|...|...|...|
    |Deutschland|2617|SO67908|1|2443.3500|922368.2100|352.4525|
    |Vereinigtes Königreich|1|SO66124|3|2.2900|1051560.1000|341.7484|
    |Vereinigtes Königreich|2|SO67823|3|2.2900|1051560.1000|341.7484|
    |...|...|...|...|...|...|...|
    |Vereinigtes Königreich|3077|SO71568|1|2443.3500|1051560.1000|341.7484|
    |USA|1|SO74796|2|2.2900|2905011.1600|289.0270|
    |USA|2|SO65114|2|2.2900|2905011.1600|289.0270|
    |...|...|...|...|...|...|...|
    |USA|10051|SO66863|1|2443.3500|2905011.1600|289.0270|

    Beachten Sie die folgenden Fakten zu diesen Ergebnissen:

    - Für jede Verkaufsauftragsposition gibt es eine Zeile.
    - Die Zeilen sind in Partitionen basierend auf der Geografie, in der der Verkauf getätigt wurde, organisiert.
    - Die Zeilen innerhalb jeder geografischen Partition werden in Reihenfolge des Verkaufsbetrags nummeriert (von kleinstem bis höchsten Wert).
    - Für jede Zeile werden der Verkaufsbetrag der Position sowie die regionalen Gesamt- und durchschnittlichen Verkaufsbeträge einbezogen.

3. Fügen Sie unter den vorhandenen Abfragen den folgenden Code hinzu, um Fensterfunktionen in einer GROUP BY-Abfrage anzuwenden und die Städte in jeder Region basierend auf ihrem Gesamtumsatz zu bewerten:

    ```sql
    SELECT  g.EnglishCountryRegionName AS Region,
            g.City,
            SUM(i.SalesAmount) AS CityTotal,
            SUM(SUM(i.SalesAmount)) OVER(PARTITION BY g.EnglishCountryRegionName) AS RegionTotal,
            RANK() OVER(PARTITION BY g.EnglishCountryRegionName
                        ORDER BY SUM(i.SalesAmount) DESC) AS RegionalRank
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    JOIN DimCustomer AS c ON i.CustomerKey = c.CustomerKey
    JOIN DimGeography AS g ON c.GeographyKey = g.GeographyKey
    GROUP BY g.EnglishCountryRegionName, g.City
    ORDER BY Region;
    ```

4. Wählen Sie nur den neuen Abfragecode aus, und verwenden Sie die Schaltfläche**▷ Ausführen**, um ihn auszuführen. Überprüfen Sie dann die Ergebnisse und beachten Sie die folgenden Punkte:
    - Die Ergebnisse enthalten eine Zeile für jede Stadt, gruppiert nach Region.
    - Der Gesamtumsatz (Summe einzelner Verkaufsbeträge) wird für jede Stadt berechnet.
    - Die Gesamtsumme des regionalen Umsatzes (die Summe der Verkaufsbeträge für jede Stadt in der Region) wird basierend auf der regionalen Partition berechnet.
    - Die Rangfolge für jede Stadt innerhalb ihrer regionalen Partition wird berechnet, indem der Gesamtumsatz pro Stadt in absteigender Reihenfolge sortiert wird.

5. Veröffentlichen Sie das aktualisierte Skript, um die Änderungen zu speichern.

> **Tipp**: ROW_NUMBER und RANK sind Beispiele für Bewertungsfunktionen, die in Transact-SQL verfügbar sind. Weitere Informationen finden Sie in der [Rangfolgefunktionen](https://docs.microsoft.com/sql/t-sql/functions/ranking-functions-transact-sql) Referenz in der  Transact-SQL-Sprachdodkumentation.

### Abrufen einer ungefähren Anzahl

Bei der Untersuchung sehr großer Datenmengen kann die Ausführung von Abfragen erhebliche Zeit und Ressourcen in Anspruch nehmen. Häufig erfordert die Datenanalyse keine absolut genauen Werte – ein Vergleich der ungefähren Werte reicht möglicherweise aus.

1. Fügen Sie unter den vorhandenen Abfragen den folgenden Code hinzu, um die Anzahl der Bestellungen für jedes Kalenderjahr abzurufen:

    ```sql
    SELECT d.CalendarYear AS CalendarYear,
        COUNT(DISTINCT i.SalesOrderNumber) AS Orders
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    GROUP BY d.CalendarYear
    ORDER BY CalendarYear;
    ```

2. Wählen Sie nur den neuen Abfragecode aus, und verwenden Sie die Schaltfläche**▷ Ausführen**, um ihn auszuführen. Überprüfen Sie dann die zurückgegebene Ausgabe:
    - Zeigen Sie auf der Registerkarte **Ergebnisse **unter der Abfrage die Anzahl der Reihenfolgen für jedes Jahr an.
    - Zeigen Sie auf der Registerkarte**Nachrichten** die Gesamtausführungszeit für die Abfrage an.
3. Ändern Sie die Abfrage wie folgt, um eine ungefähre Anzahl für jedes Jahr zurückzugeben. Dann führen Sie die Abfrage erneut aus.

    ```sql
    SELECT d.CalendarYear AS CalendarYear,
        APPROX_COUNT_DISTINCT(i.SalesOrderNumber) AS Orders
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    GROUP BY d.CalendarYear
    ORDER BY CalendarYear;
    ```

4. Überprüfen Sie die zurückgegebene Ausgabe:
    - Zeigen Sie auf der Registerkarte **Ergebnisse **unter der Abfrage die Anzahl der Reihenfolgen für jedes Jahr an. Diese sollten innerhalb von 2 % der tatsächlichen Anzahl, die von der vorherigen Abfrage abgerufen wurden, betragen.
    - Zeigen Sie auf der Registerkarte**Nachrichten** die Gesamtausführungszeit für die Abfrage an. Diese sollte kürzer sein als für die vorherige Abfrage.

5. Veröffentlichen Sie das Skript, um die Änderungen zu speichern.

> **Tipp**: Weitere Details finden Sie in der Dokumentation zur [APPROX_COUNT_DISTINCT](https://docs.microsoft.com/sql/t-sql/functions/approx-count-distinct-transact-sql) Funktion.

## Herausforderung – Analysieren des Vertriebs von Wiederverkäufern

1. Erstellen Sie ein neues leeres Skript für den **SQL*xxxxxxx*** SQL-Pool, und speichern Sie es mit dem Namen **Analysieren des Vertriebs von Wiederverkäufern**.
2. Erstellen Sie SQL-Abfragen im Skript, um die folgenden Informationen basierend auf der Tabelle **FactResellerSales-Fakten** und den Dimensionstabellen zu finden, mit denen sie verknüpft ist:
    - Die Gesamtmenge der pro Geschäftsjahr und Quartal verkauften Artikel.
    - Die Gesamtmenge der verkauften Artikel pro Geschäftsjahr, Quartal und Vertriebsgebiet Region, die dem Mitarbeiter zugeordnet ist, der den Verkauf getätigt hat.
    - Die Gesamtmenge der pro Geschäftsjahr, Quartal und Vertriebsgebietsregion verkauften Artikel nach Produktkategorie.
    - Der Rang jedes Vertriebsgebiets pro Geschäftsjahr basierend auf dem Gesamtumsatzbetrag für das Jahr.
    - Die ungefähre Anzahl der Verkaufsbestellungen pro Jahr in jedem Vertriebsgebiet.

    > **Tipp**: Vergleichen Sie Ihre Abfragen mit den Abfragen im **Lösungs-** Skript auf der Seite **Entwickeln** in Synapse Studio.

3. Experimentieren Sie mit Abfragen, um die restlichen Tabellen im Data Warehouse-Schema als Ihre Freizeit zu erkunden.
4. Wenn Sie fertig sind, stoppen Sie auf der Seite **Verwalten** den **sql*xxxxxxx*** dedizidierten SQL Pool.

## Löschen von Azure-Ressourcen

Wenn Sie sich mit Azure Synapse Analytics vertraut gemacht haben, sollten Sie die erstellten Ressourcen löschen, um unnötige Azure-Kosten zu vermeiden.

1. Schließen Sie die Registerkarte mit Synapse Studio, und kehren Sie zum Azure-Portal zurück.
2. Wählen Sie auf der **Startseite** des Azure-Portals die Option **Ressourcengruppen** aus.
3. Wählen Sie die Ressourcengruppe **dp203-*xxxxxxx*** für Ihren Synapse Analytics-Arbeitsbereich aus (nicht die verwaltete Ressourcengruppe), und vergewissern Sie sich, dass sie den Synapse-Arbeitsbereich, das Speicherkonto und den dedizierten SQL-Pool für Ihren Arbeitsbereich enthält.
4. Wählen Sie oben auf der Seite **Übersicht** für Ihre Ressourcengruppe die Option **Ressourcengruppe löschen** aus.
5. Geben Sie den Namen der Ressourcengruppe **dp203-*xxxxxxx*** ein, um zu bestätigen, dass Sie sie löschen möchten, und wählen Sie **Löschen** aus.

    Nach einigen Minuten werden die Ressourcengruppe in Ihrem Azure Synapse-Arbeitsbereich und die damit verknüpfte Ressourcengruppe im verwalteten Arbeitsbereich gelöscht.
