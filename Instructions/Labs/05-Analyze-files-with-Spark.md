---
lab:
  title: Analysieren von Daten in einem Data Lake mit Spark
  ilt-use: Suggested demo
---
# Analysieren von Daten in einem Data Lake mit Spark

Apache Spark ist eine Open-Source-Engine für verteilte Datenverarbeitung und wird häufig verwendet, um große Datenmengen in Data Lake Storage zu untersuchen, zu verarbeiten und zu analysieren. Spark ist als Verarbeitungsoption in vielen Datenplattformprodukten verfügbar (einschließlich Azure HDInsight, Azure Databricks, Azure Synapse Analytics und Microsoft Fabric). Einer der Vorteile von Spark ist die Unterstützung für eine Vielzahl von Programmiersprachen (einschließlich Java, Scala, Python und SQL). Dies macht Spark zu einer sehr flexiblen Lösung für Datenverarbeitungsworkloads (einschließlich Datenbereinigung und -manipulation, statistischer Analysen, maschinellem Lernen, Datenanalysen und Visualisierungen).

Dieses Lab dauert ungefähr **45** Minuten.

## Vor der Installation

Sie benötigen ein [Azure-Abonnement](https://azure.microsoft.com/free), in dem Sie Administratorzugriff besitzen.

## Bereitstellen eines Azure Synapse Analytics-Arbeitsbereichs

Sie benötigen einen Azure Synapse Analytics-Arbeitsbereich mit Zugriff auf den Datenspeicher und einen Apache Spark-Pool, den Sie zum Abfragen und Verarbeiten von Dateien im Data Lake verwenden können.

In dieser Übung verwenden Sie eine Kombination aus einem PowerShell-Skript und einer ARM-Vorlage, um einen Azure Synapse Analytics-Arbeitsbereich bereitzustellen.

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
    cd dp203/Allfiles/labs/05
    ./setup.ps1
    ```

6. Wenn Sie dazu aufgefordert werden, wählen Sie aus, welches Abonnement Sie verwenden möchten (dies geschieht nur, wenn Sie Zugriff auf mehrere Azure-Abonnements haben).
7. Wenn Sie dazu aufgefordert werden, geben Sie ein geeignetes Kennwort ein, das für Ihren Azure Synapse SQL-Pool festgelegt werden soll.

    > **Hinweis**: Merken Sie sich unbedingt das Kennwort!

8. Warten Sie, bis das Skript abgeschlossen ist. Dies dauert in der Regel etwa 10 Minuten, kann aber in manchen Fällen auch länger dauern. Während Sie warten, lesen Sie den Artikel [Apache Spark in Azure Synapse Analytics](https://docs.microsoft.com/azure/synapse-analytics/spark/apache-spark-overview) in der Dokumentation zu Azure Synapse Analytics.

## Abfragen von Daten in Dateien

Das Skript stellt einen Azure Synapse Analytics-Arbeitsbereich und ein Azure Storage-Konto zum Hosten des Datensees bereit und lädt dann einige Datendateien in den Data Lake hoch.

### Anzeigen von Dateien im Data Lake

1. Wechseln Sie nach Abschluss des Skripts im Azure-Portal zur erstellten Ressourcengruppe **dp203-*xxxxxxx***, und wählen Sie Ihren Synapse-Arbeitsbereich aus.
2. Wählen Sie auf der Seite **Übersicht** für Ihren Synapse-Arbeitsbereich in der Karte **Synapse Studio öffnen** die Option **Öffnen** aus, um Synapse Studio in einer neuen Browserregisterkarte zu öffnen. Melden Sie sich an, wenn Sie dazu aufgefordert werden.
3. Verwenden Sie im linken Bereich von Synapse Studio das Symbol **&rsaquo;&rsaquo;** , um das Menü zu erweitern. -dadurch werden die verschiedenen Seiten in Synapse Studio angezeigt, die Sie zur Verwaltung von Ressourcen und zur Durchführung von Datenanalyseaufgaben verwenden werden.
4. Wählen Sie auf der Seite **Verwalten** die Registerkarte **Apache Spark Pools** aus, und beachten Sie, dass ein Spark-Pool mit einem Namen wie **spark*xxxxxxx*** im Arbeitsbereich bereitgestellt wurde. Später verwenden Sie diesen Spark-Pool, um Daten aus Dateien im Datenspeicher für den Arbeitsbereich zu laden und zu analysieren.
5. Zeigen Sie auf der Seite **Daten** die Registerkarte **Verknüpft** an, und vergewissern Sie sich, dass Ihr Arbeitsbereich einen Link zu Ihrem Azure Data Lake Storage Gen2-Speicherkonto enthält, das einen Namen wie **Synapse*xxxxxxx* (Primary - datalake*xxxxxxx*)**hat.
6. Erweitern Sie Ihr Speicherkonto, und stellen Sie sicher, dass es einen Dateisystemcontainer mit Namen **Dateien** enthält.
7. Wählen Sie den **Dateicontainer** aus, und beachten Sie, dass er Ordner mit dem Namen **Sales** und **Synapse** enthält. Der Ordner **Synapse** wird von Azure Synapse verwendet, und der Ordner **Vertrieb** enthält die Datendateien, die Sie abfragen möchten.
8. Öffnen Sie den Ordner**Vertrieb** und den darin enthaltenen Ordner **Bestellungen** und beachten Sie, dass der Ordner**Bestellungen** CSV-Dateien für drei Jahre Verkaufsdaten enthält.
9. Klicken Sie mit der rechten Maustaste auf eine der Dateien, und wählen Sie **Vorschau** aus, um die darin enthaltenen Daten anzuzeigen. Beachten Sie, dass die Dateien keine Kopfzeile enthalten, sodass Sie die Auswahl der Option zum Anzeigen von Spaltenüberschriften aufheben können.

### Verwenden Sie Spark zum Erkunden von Daten

1. Wählen Sie eine der Dateien im Ordner**Bestellungen**  aus, und wählen Sie dann in der Liste **Neues Notebook** auf der Symbolleiste **In DataFrameladen** aus. Ein Datenframe ist eine Struktur in Spark, die ein tabellarisches Dataset darstellt.
2. Wählen Sie auf der neuen Registerkarte**Notebook 1** die geöffnet wird, in der Liste **Anfügen an** Ihren Spark-Pool (**spark*xxxxxxx***) aus. Verwenden Sie dann die Schaltfläche **▷ Alle ausführen**, um alle Zellen im Notebook auszuführen (derzeit nur eine!).

    Hinweis: Da Sie Spark-Code zum ersten Mal in dieser Sitzung ausführen, muss der Spark-Pool gestartet werden. Dies bedeutet, dass die erste Ausführung in der Sitzung etwa eine Minute dauern kann. Nachfolgende Ausführungen erfolgen schneller.

3. Während Sie auf die Initialisierung der Spark-Sitzung warten, überprüfen Sie den generierten Code. das sieht ähnlich wie folgt aus:

    ```Python
    %%pyspark
    df = spark.read.load('abfss://files@datalakexxxxxxx.dfs.core.windows.net/sales/orders/2019.csv', format='csv'
    ## If header exists uncomment line below
    ##, header=True
    )
    display(df.limit(10))
    ```

4. Wenn der Code ausgeführt wurde, überprüfen Sie die Ausgabe unter der Zelle im Notebook. Sie zeigt die ersten zehn Zeilen in der ausgewählten Datei mit automatischen Spaltennamen im Formular **_c0**, **_c1**, **_c2** usw. an.
5. Ändern Sie den Code so, dass die Funktionen **Spark.read.load** Daten aus <u>allen</u> CSV-Dateien im Ordner liest, und die **Anzeigefunktion** zeigt die ersten 100 Zeilen an. Ihr Code sollte wie folgt aussehen (wobei *datalakexxxxxxx mit* dem Namen Ihres Datenspeichers übereinstimmen):

    ```Python
    %%pyspark
    df = spark.read.load('abfss://files@datalakexxxxxxx.dfs.core.windows.net/sales/orders/*.csv', format='csv'
    )
    display(df.limit(100))
    ```

6. Verwenden Sie die Schaltfläche **▷** links neben der Codezelle, um nur diese Zelle auszuführen, und überprüfen Sie die Ergebnisse.

    Der Dataframe enthält jetzt Daten aus allen Dateien, aber die Spaltennamen sind nicht hilfreich. Spark verwendet einen "schema-on-read"-Ansatz, um geeignete Datentypen für die Spalten basierend auf den darin enthaltenen Daten zu ermitteln, und wenn eine Kopfzeile in einer Textdatei vorhanden ist, kann sie verwendet werden, um die Spaltennamen zu identifizieren (durch Angeben eines **Header=True-Parameters** in der **Ladefunktion** ). Alternativ können Sie ein explizites Schema für den Dataframe definieren.

7. Ändern Sie den Code wie folgt (ersetzen Sie *datalakexxxxxxx),* um ein explizites Schema für den Dataframe zu definieren, der die Spaltennamen und Datentypen enthält. Führen Sie den Code in der Zelle erneut aus.

    ```Python
    %%pyspark
    from pyspark.sql.types import *
    from pyspark.sql.functions import *

    orderSchema = StructType([
        StructField("SalesOrderNumber", StringType()),
        StructField("SalesOrderLineNumber", IntegerType()),
        StructField("OrderDate", DateType()),
        StructField("CustomerName", StringType()),
        StructField("Email", StringType()),
        StructField("Item", StringType()),
        StructField("Quantity", IntegerType()),
        StructField("UnitPrice", FloatType()),
        StructField("Tax", FloatType())
        ])

    df = spark.read.load('abfss://files@datalakexxxxxxx.dfs.core.windows.net/sales/orders/*.csv', format='csv', schema=orderSchema)
    display(df.limit(100))
    ```

8. Verwenden Sie unter den Ergebnissen das Symbol **＋ Code**, um dem Notebook eine neue Codezelle hinzuzufügen. Fügen Sie dann in der neuen Zelle den folgenden Code hinzu, um das Dataframeschema anzuzeigen:

    ```Python
    df.printSchema()
    ```

9. Führen Sie die neue Zelle aus, und stellen Sie sicher, dass das Dataframe-Schema dem von Ihnen definierten **OrderSchema** entspricht. Die **printSchema**-Funktion kann nützlich sein, wenn Sie einen Datenrahmen mit einem automatisch abgeleiteten Schema verwenden.

## Analysieren von Daten in einem Dataframe

Das **Dataframe**-Objekt in Spark ähnelt einem Pandas-Dataframe in Python und enthält eine breite Palette von Funktionen, mit denen Sie die enthaltenen Daten manipulieren, filtern, gruppieren und anderweitig analysieren können.

### Filtern eines Dataframes

1. Fügen Sie dem Notebook eine neue Codezelle hinzu, und geben Sie darin den folgenden Code ein:

    ```Python
    customers = df['CustomerName', 'Email']
    print(customers.count())
    print(customers.distinct().count())
    display(customers.distinct())
    ```

2. Führen Sie die neue Codezelle aus, und überprüfen Sie die Ergebnisse. Beachten Sie die folgenden Details:
    - Wenn Sie einen Vorgang für einen Dataframe ausführen, ist das Ergebnis ein neuer Dataframe (in diesem Fall wird ein neuer **customers**-Dataframe erstellt, indem eine bestimmte Teilmenge von Spalten aus dem **df**-Dataframe ausgewählt wird).
    - Dataframes bieten Funktionen wie **count** und **distinct**, die zum Zusammenfassen und Filtern der darin enthaltenen Daten verwendet werden können.
    - Die `dataframe['Field1', 'Field2', ...]` Syntax ist eine praktische Möglichkeit zum Definieren einer Teilmenge von Spalten. Sie können auch die **select**-Methode verwenden, sodass die erste Zeile des obigen Codes wie folgt geschrieben werden kann: `customers = df.select("CustomerName", "Email")`.

3. Ändern Sie den Code wie folgt:

    ```Python
    customers = df.select("CustomerName", "Email").where(df['Item']=='Road-250 Red, 52')
    print(customers.count())
    print(customers.distinct().count())
    display(customers.distinct())
    ```

4. Führen Sie den geänderten Code aus, um die Kunden anzuzeigen, die das Produkt *Road-250 Red, 52* erworben haben. Beachten Sie, dass Sie mehrere Funktionen verketten können, sodass die Ausgabe einer Funktion zur Eingabe für die nächste wird. In diesem Fall ist der von der **select**-Methode erstellte Dataframe der Quelldataframe für die **where**-Methode, die zum Anwenden von Filterkriterien verwendet wird.

### Aggregieren und Gruppieren von Daten in einem Dataframe

1. Fügen Sie dem Notebook eine neue Codezelle hinzu, und geben Sie darin den folgenden Code ein:

    ```Python
    productSales = df.select("Item", "Quantity").groupBy("Item").sum()
    display(productSales)
    ```

2. Führen Sie die hinzugefügte Codezelle aus, und beachten Sie, dass in den Ergebnissen die Summe der Bestellmengen nach Produkt gruppiert angezeigt wird. Die **groupBy**-Methode gruppiert die Zeilen nach *Item*, und die nachfolgende **sum**-Aggregatfunktion wird auf alle verbleibenden numerischen Spalten angewendet (in diesem Fall *Quantity*).

3. Fügen Sie dem Notebook eine weitere Codezelle hinzu, und geben Sie darin den folgenden Code ein:

    ```Python
    yearlySales = df.select(year("OrderDate").alias("Year")).groupBy("Year").count().orderBy("Year")
    display(yearlySales)
    ```

4. Führen Sie die hinzugefügte Codezelle aus, und beachten Sie, dass in den Ergebnissen die Anzahl der Verkaufsaufträge pro Jahr angezeigt wird. Beachten Sie, dass die **Select**-Methode eine SQL **Jahres** funktion enthält, um die Jahreskomponente des  Felds*OrderDate* zu extrahieren, und dann wird eine **Alias** Methode verwendet, um dem extrahierten Jahreswert einen Spaltennamen zuzuweisen. Die Daten werden dann nach der abgeleiteten *Year*-Spalte gruppiert, und die Anzahl der Zeilen in jeder Gruppe wird berechnet, bevor schließlich die **orderBy**-Methode verwendet wird, um den resultierenden Dataframe zu sortieren.

## Abfragen von Daten mithilfe der Spark SQL

Wie Sie gesehen haben, ermöglichen Ihnen die nativen Methoden des Dataframeobjekts, Daten aus einer Datei effektiv abzufragen und zu analysieren. Viele Data Analysts arbeiten jedoch bevorzugt mit Tabellen, die sie mithilfe von SQL-Syntax abfragen können. Spark SQL ist eine SQL-Sprach-API in Spark, die Sie zum Ausführen von SQL-Anweisungen oder sogar zum Speichern von Daten in relationalen Tabellen verwenden können.

### Verwenden von Spark SQL im PySpark-Code

Die Standardsprache in Azure Synapse Studio-Notebooks ist PySpark, eine Spark-basierte Python-Laufzeit. Innerhalb dieser Laufzeit können Sie die Bibliothek **Spark.sql** verwenden, um Spark SQL-Syntax in Ihren Python-Code einzubetten und mit SQL-Konstrukten wie Tabellen und Ansichten zu arbeiten.

1. Fügen Sie dem Notebook eine neue Codezelle hinzu, und geben Sie darin den folgenden Code ein:

    ```Python
    df.createOrReplaceTempView("salesorders")

    spark_df = spark.sql("SELECT * FROM salesorders")
    display(spark_df)
    ```

2. Führen Sie die Zelle aus, und überprüfen Sie die Ergebnisse. Beachten Sie, Folgendes:
    - Der Code speichert die Daten im Dataframe **DF** als temporäre Ansicht namens **Salesorders**. Spark SQL unterstützt die Verwendung temporärer Ansichten oder beibehaltener Tabellen als Quellen für SQL-Abfragen.
    - Die Methode **Spark.sql** wird dann verwendet, um eine SQL-Abfrage für die **Salesorders**-Ansicht auszuführen.
    - Die Abfrageergebnisse werden in einem Dataframe gespeichert.

### Ausführen von SQL-Code in einer Zelle

Obwohl es nützlich ist, SQL-Anweisungen in eine Zelle einzubetten, die PySpark-Code enthält, bevorzugen Data Analysts oft die Arbeit direkt in SQL.

1. Fügen Sie dem Notebook eine neue Codezelle hinzu, und geben Sie darin den folgenden Code ein:

    ```sql
    %%sql
    SELECT YEAR(OrderDate) AS OrderYear,
           SUM((UnitPrice * Quantity) + Tax) AS GrossRevenue
    FROM salesorders
    GROUP BY YEAR(OrderDate)
    ORDER BY OrderYear;
    ```

2. Führen Sie die Zelle aus, und überprüfen Sie die Ergebnisse. Beachten Sie, Folgendes:
    - Die Zeile `%%sql` am Anfang der Zelle (wird als *Magic-Befehl* bezeichnet) gibt an, dass anstelle von PySpark die Spark SQL-Runtime verwendet werden soll, um den Code in dieser Zelle auszuführen.
    - Der SQL-Code verweist auf die Tabelle **salesorders**, die Sie zuvor mithilfe von PySpark erstellt haben.
    - Die Ausgabe der SQL-Abfrage wird automatisch als Ergebnis unter der Zelle angezeigt.

> **Hinweis:** Weitere Informationen zu Spark SQL und Dataframes finden Sie in der [Spark SQL-Dokumentation](https://spark.apache.org/docs/2.2.0/sql-programming-guide.html).

## Visualisieren von Daten mit Spark

Ein Bild sagt sprichwörtlich mehr als tausend Worte, und ein Diagramm ist oft besser als tausend Datenzeilen. Notebooks in Fabric enthalten zwar eine integrierte Diagrammansicht für Daten, die aus einem Dataframe oder einer Spark SQL-Abfrage angezeigt werden, die Ansicht ist jedoch nicht für die umfassende Diagrammdarstellung konzipiert. Sie können jedoch Python-Grafikbibliotheken wie **matplotlib** und **seaborn** verwenden, um Diagramme aus Daten in Dataframes zu erstellen.

### Anzeigen von Ergebnissen als Diagramm

1. Fügen Sie dem Notebook eine neue Codezelle hinzu, und geben Sie darin den folgenden Code ein:

    ```sql
    %%sql
    SELECT * FROM salesorders
    ```

2. Führen Sie den Code aus, und beachten Sie, dass er die Daten aus der **salesorders**-Ansicht zurückgibt, die Sie zuvor erstellt haben.
3. Ändern Sie im Ergebnisabschnitt unterhalb der Zelle die Option **Ansicht** von **Tabelle** in **Diagramm**.
4. Verwenden Sie die Schaltfläche **Ansichtsoptionen** oben rechts im Diagramm, um den Optionsbereich für das Diagramm anzuzeigen. Legen Sie dann die Optionen wie folgt fest, und klicken Sie auf **Anwenden**:
    - **Diagrammtyp:** Balkendiagramm
    - **Schlüssel:** Element
    - **Werte:** Menge
    - **Reihengruppe:** *Leer lassen*
    - **Aggregation:** Summe
    - **Gestapelt:** *Nicht aktiviert*

5. Stellen Sie sicher, dass das Diagramm in etwa wie folgt aussieht:

    ![Ein Balkendiagramm der Produkte nach Gesamtbestellungsmenge](./images/notebook-chart.png)

### Erste Schritte mit **matplotlib**

1. Fügen Sie dem Notebook eine neue Codezelle hinzu, und geben Sie darin den folgenden Code ein:

    ```Python
    sqlQuery = "SELECT CAST(YEAR(OrderDate) AS CHAR(4)) AS OrderYear, \
                    SUM((UnitPrice * Quantity) + Tax) AS GrossRevenue \
                FROM salesorders \
                GROUP BY CAST(YEAR(OrderDate) AS CHAR(4)) \
                ORDER BY OrderYear"
    df_spark = spark.sql(sqlQuery)
    df_spark.show()
    ```

2. Führen Sie den Code aus, und überprüfen Sie, ob ein Spark-Dataframe mit dem Jahresumsatz zurückgegeben wird.

    Um die Daten als Diagramm zu visualisieren, verwenden Sie zunächst die Python-Bibliothek **matplotlib**. Diese Bibliothek ist die zentrale Bibliothek für Darstellungen, auf der viele andere basieren, und sie bietet hohe Flexibilität beim Erstellen von Diagrammen.

3. Fügen Sie dem Notebook eine neue Codezelle hinzu, und fügen Sie darin den folgenden Code ein:

    ```Python
    from matplotlib import pyplot as plt

    # matplotlib requires a Pandas dataframe, not a Spark one
    df_sales = df_spark.toPandas()

    # Create a bar plot of revenue by year
    plt.bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'])

    # Display the plot
    plt.show()
    ```

4. Führen Sie die Zelle aus, und überprüfen Sie die Ergebnisse, die sich aus einem Säulendiagramm mit dem Gesamtbruttoumsatz für jedes Jahr zusammensetzen. Beachten Sie die folgenden Features des Codes, der zum Erstellen dieses Diagramms verwendet wird:
    - Die **matplotlib**-Bibliothek erfordert einen *Pandas*-Dataframe, weshalb Sie den *Spark*-Dataframe, der von der Spark SQL-Abfrage zurückgegeben wird, in dieses Format konvertieren müssen.
    - Der Kern der **matplotlib**-Bibliothek ist das **pyplot**-Objekt. Dies ist die Grundlage für die meisten Darstellungsfunktionen.
    - Die Standardeinstellungen führen zu einem verwendbaren Diagramm, aber es gibt einige Optionen, um es weiter anzupassen.

5. Ändern Sie den Code, um das Diagramm wie folgt darzustellen:

    ```Python
    # Clear the plot area
    plt.clf()

    # Create a bar plot of revenue by year
    plt.bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'], color='orange')

    # Customize the chart
    plt.title('Revenue by Year')
    plt.xlabel('Year')
    plt.ylabel('Revenue')
    plt.grid(color='#95a5a6', linestyle='--', linewidth=2, axis='y', alpha=0.7)
    plt.xticks(rotation=45)

    # Show the figure
    plt.show()
    ```

6. Führen Sie die Codezelle erneut aus, und zeigen Sie die Ergebnisse an. Das Diagramm enthält nun etwas mehr Informationen.

    Plots sind technisch gesehen in **Abbildungen** enthalten. In den vorherigen Beispielen wurde die Abbildung implizit für Sie erstellt, aber Sie können sie auch explizit erstellen.

7. Ändern Sie den Code, um das Diagramm wie folgt darzustellen:

    ```Python
    # Clear the plot area
    plt.clf()

    # Create a Figure
    fig = plt.figure(figsize=(8,3))

    # Create a bar plot of revenue by year
    plt.bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'], color='orange')

    # Customize the chart
    plt.title('Revenue by Year')
    plt.xlabel('Year')
    plt.ylabel('Revenue')
    plt.grid(color='#95a5a6', linestyle='--', linewidth=2, axis='y', alpha=0.7)
    plt.xticks(rotation=45)

    # Show the figure
    plt.show()
    ```

8. Führen Sie die Codezelle erneut aus, und zeigen Sie die Ergebnisse an. Die Abbildung bestimmt die Form und Größe des Plots.

    Eine Abbildung kann mehrere Teilplots enthalten (jeweils auf einer eigenen *Achse*).

9. Ändern Sie den Code, um das Diagramm wie folgt darzustellen:

    ```Python
    # Clear the plot area
    plt.clf()

    # Create a figure for 2 subplots (1 row, 2 columns)
    fig, ax = plt.subplots(1, 2, figsize = (10,4))

    # Create a bar plot of revenue by year on the first axis
    ax[0].bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'], color='orange')
    ax[0].set_title('Revenue by Year')

    # Create a pie chart of yearly order counts on the second axis
    yearly_counts = df_sales['OrderYear'].value_counts()
    ax[1].pie(yearly_counts)
    ax[1].set_title('Orders per Year')
    ax[1].legend(yearly_counts.keys().tolist())

    # Add a title to the Figure
    fig.suptitle('Sales Data')

    # Show the figure
    plt.show()
    ```

10. Führen Sie die Codezelle erneut aus, und zeigen Sie die Ergebnisse an. Die Abbildung enthält die im Code angegebenen Teilplots.

> **Hinweis:** Weitere Informationen zur Darstellung mit „matplotlib“ finden Sie in der [matplotlib-Dokumentation](https://matplotlib.org/).

### Verwenden der **seaborn**-Bibliothek

Mit **matplotlib** können Sie zwar komplexe Diagramme mit mehreren Typen erstellen, es kann jedoch komplexen Code erfordern, um die besten Ergebnisse zu erzielen. Aus diesem Grund wurden im Laufe der Jahre viele neue Bibliotheken auf der Basis von „matplotlib“ entwickelt, um die Komplexität dieser Bibliothek zu abstrahieren und ihre Funktionen zu verbessern. Eine dieser Bibliotheken ist **seaborn**.

1. Fügen Sie dem Notebook eine neue Codezelle hinzu, und geben Sie darin den folgenden Code ein:

    ```Python
    import seaborn as sns

    # Clear the plot area
    plt.clf()

    # Create a bar chart
    ax = sns.barplot(x="OrderYear", y="GrossRevenue", data=df_sales)
    plt.show()
    ```

2. Führen Sie den Code aus, und überprüfen Sie, ob bei Verwendung der seaborn-Bibliothek ein Balkendiagramm angezeigt wird.
3. Fügen Sie dem Notebook eine neue Codezelle hinzu, und geben Sie darin den folgenden Code ein:

    ```Python
    # Clear the plot area
    plt.clf()

    # Set the visual theme for seaborn
    sns.set_theme(style="whitegrid")

    # Create a bar chart
    ax = sns.barplot(x="OrderYear", y="GrossRevenue", data=df_sales)
    plt.show()
    ```

4. Führen Sie den geänderten Code aus, und beachten Sie, dass Sie mit Seaborn ein konsistentes Farbdesign für Ihre Plots festlegen können.

5. Fügen Sie dem Notebook eine neue Codezelle hinzu, und geben Sie darin den folgenden Code ein:

    ```Python
    # Clear the plot area
    plt.clf()

    # Create a bar chart
    ax = sns.lineplot(x="OrderYear", y="GrossRevenue", data=df_sales)
    plt.show()
    ```

6. Führen Sie den geänderten Code aus, um den Jahresumsatz als Liniendiagramm anzuzeigen.

> **Hinweis:** Weitere Informationen zur Darstellung mithilfe der seaborn-Bibliothek finden Sie in der [seaborn-Dokumentation](https://seaborn.pydata.org/index.html).

## Löschen von Azure-Ressourcen

Wenn Sie sich mit Azure Synapse Analytics vertraut gemacht haben, sollten Sie die erstellten Ressourcen löschen, um unnötige Azure-Kosten zu vermeiden.

1. Schließen Sie die Registerkarte mit Synapse Studio, und kehren Sie zum Azure-Portal zurück.
2. Wählen Sie auf der **Startseite** des Azure-Portals die Option **Ressourcengruppen** aus.
3. Wählen Sie die Ressourcengruppe **dp203-*xxxxxxx*** für Ihren Synapse Analytics-Arbeitsbereich aus (nicht die verwaltete Ressourcengruppe), und vergewissern Sie sich, dass sie den Synapse-Arbeitsbereich, das Speicherkonto und den Spark-Pool für Ihren Arbeitsbereich enthält.
4. Wählen Sie oben auf der Seite **Übersicht** für Ihre Ressourcengruppe die Option **Ressourcengruppe löschen** aus.
5. Geben Sie den Namen der Ressourcengruppe **dp203-*xxxxxxx*** ein, um zu bestätigen, dass Sie sie löschen möchten, und wählen Sie **Löschen** aus.

    Nach einigen Minuten werden die Ressourcengruppe in Ihrem Azure Synapse-Arbeitsbereich und die damit verknüpfte Ressourcengruppe im verwalteten Arbeitsbereich gelöscht.
