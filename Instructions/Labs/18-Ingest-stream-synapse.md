---
lab:
  title: Erfassen von Echtzeitdaten mit Azure Stream Analytics und Azure Synapse Analytics
  ilt-use: Lab
---

# Erfassen von Echtzeitdaten mit Azure Stream Analytics und Azure Synapse Analytics

Datenanalyselösungen umfassen häufig eine Anforderung zum Erfassen und Verarbeiten von *Datenströmen*. Die Verarbeitung von Datenströmen unterscheidet sich in der Hinsicht von der Batchverarbeitung, dass Datenströme im Allgemeinen *endlos* sind, d. h. es handelt sich um fortlaufende Datenquellen, die fortwährend und nicht in festen Intervallen verarbeitet werden müssen.

Azure Stream Analytics bietet einen Clouddienst, mit dem Sie eine *Abfrage* definieren können, die auf einem Datenstrom aus einer Streamingquelle wie etwa Azure Event Hubs oder einem Azure IoT Hub ausgeführt wird. Sie können eine Azure Stream Analytics-Abfrage verwenden, um den Datenstrom direkt in einen Datenspeicher aufzunehmen, um weitere Analysen durchzuführen oder die Daten basierend auf zeitlichen Fenstern zu filtern, zu aggregieren und zusammenzufassen.

In dieser Übung verwenden Sie Azure Stream Analytics, um einen Datenstrom von Bestelldaten zu verarbeiten, wie er etwa von einer Online-Einzelhandelsanwendung erzeugt werden könnte. Die Bestelldaten werden an Azure Event Hubs gesendet, von wo aus Ihre Azure Stream Analytics-Aufträge die Daten lesen und in Azure Synapse Analytics aufnehmen.

Diese Übung dauert ca. **45** Minuten.

## Vorbereitung

Sie benötigen ein [Azure-Abonnement](https://azure.microsoft.com/free), in dem Sie Administratorzugriff besitzen.

## Bereitstellen von Azure-Ressourcen

In dieser Übung benötigen Sie einen Azure Synapse Analytics-Arbeitsbereich mit Zugriff auf den Data Lake-Speicher und einen dedizierten SQL-Pool. Außerdem benötigen Sie einen Azure Event Hubs-Namespace, an den die Bestelldaten des Streamings gesendet werden können.

Sie verwenden eine Kombination aus einem PowerShell-Skript und einer ARM-Vorlage, um diese Ressourcen bereitzustellen.

1. Melden Sie sich beim [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com` an.
2. Verwenden Sie rechts neben der Suchleiste oben auf der Seite die Schaltfläche **[\>_]**, um eine neue Cloud Shell-Instanz im Azure-Portal zu erstellen. Wählen Sie eine ***PowerShell***-Umgebung aus, und erstellen Sie Speicher, falls Sie dazu aufgefordert werden. Die Cloud Shell bietet eine Befehlszeilenschnittstelle in einem Bereich am unteren Rand des Azure-Portals, wie hier gezeigt:

    ![Azure-Portal mit einem Cloud Shell-Bereich](./images/cloud-shell.png)

    > **Hinweis**: Wenn Sie zuvor eine Cloud Shell erstellt haben, die eine *Bash*-Umgebung verwendet, ändern Sie diese mithilfe des Dropdownmenüs oben links im Cloud Shell-Bereich zu ***PowerShell***.

3. Beachten Sie, dass Sie die Größe der Cloud Shell durch Ziehen der Trennzeichenleiste oben im Bereich ändern können oder den Bereich mithilfe der Symbole **&#8212;**, **&#9723;** und **X** oben rechts minimieren, maximieren und schließen können. Weitere Informationen zur Verwendung von Azure Cloud Shell finden Sie in der [Azure Cloud Shell-Dokumentation](https://docs.microsoft.com/azure/cloud-shell/overview).

4. Geben Sie im PowerShell-Bereich die folgenden Befehle ein, um das Repository mit dieser Übung zu klonen:

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Nachdem das Repository geklont wurde, geben Sie die folgenden Befehle ein, um in den Ordner für diese Übung zu wechseln. Führen Sie das darin enthaltene Skript **setup.ps1** aus:

    ```
    cd dp-203/Allfiles/labs/18
    ./setup.ps1
    ```

6. Wenn Sie dazu aufgefordert werden, wählen Sie aus, welches Abonnement Sie verwenden möchten (dies geschieht nur, wenn Sie Zugriff auf mehrere Azure-Abonnements haben).
7. Wenn Sie dazu aufgefordert werden, geben Sie ein geeignetes Kennwort ein, das für Ihren Azure Synapse SQL-Pool festgelegt werden soll.

    > **Hinweis**: Merken Sie sich unbedingt dieses Kennwort!

8. Warten Sie, bis das Skript abgeschlossen ist. Dies dauert in der Regel etwa 15 Minuten, in einigen Fällen kann es jedoch länger dauern. Während Sie warten, lesen Sie den Artikel [Willkommen bei Azure Stream Analytics](https://learn.microsoft.com/azure/stream-analytics/stream-analytics-introduction) in der Dokumentation zu Azure Stream Analytics.

## Aufnehmen von Streamingdaten in einen dedizierten SQL-Pool

Beginnen wir damit, einen Datenstrom direkt in eine Tabelle in einem dedizierten SQL-Pool von Azure Synapse Analytics aufzunehmen.

### Anzeigen der Streamingquelle und Datenbanktabelle

1. Wenn die Ausführung des Setupskripts abgeschlossen ist, minimieren Sie den Cloud Shell-Bereich (Sie kehren später dorthin zurück). Wechseln Sie dann im Azure-Portal zur erstellten Ressourcengruppe **dp203-*xxxxxxx***, und beachten Sie, dass diese Ressourcengruppe einen Azure Synapse-Arbeitsbereich, ein Storage-Konto für Ihren Data Lake, einen dedizierten SQL-Pool und einen Event Hubs-Namespace enthält.
2. Wählen Sie Ihren Synapse-Arbeitsbereich aus, und klicken Sie auf der Seite **Übersicht** der Karte **Synapse Studio öffnen** auf die Option **Öffnen**, um Synapse Studio in einer neuen Browserregisterkarte zu öffnen. Synapse Studio ist eine webbasierte Schnittstelle, die Sie zum Arbeiten mit Ihrem Synapse Analytics-Arbeitsbereich verwenden können.
3. Verwenden Sie auf der linken Seite von Synapse Studio das Symbol **&rsaquo;&rsaquo;**, um das Menü zu erweitern. Dadurch werden die verschiedenen Seiten in Synapse Studio angezeigt, die Sie zum Verwalten von Ressourcen und zum Ausführen von Datenanalyseaufgaben verwenden.
4. Wählen Sie auf der Seite **Verwalten** im Abschnitt **SQL-Pools** die Zeile für den dedizierten SQL-Pool **sql*xxxxxxx*** aus, und verwenden Sie dann das zugehörige Symbol **&#9655;**, um sie fortzusetzen.
5. Während Sie warten, bis der SQL-Pool gestartet wird, wechseln Sie zurück zur Browserregisterkarte mit dem Azure-Portal, und öffnen Sie den Cloud Shell-Bereich erneut.
6. Geben Sie im Cloud Shell-Bereich den folgenden Befehl ein, um eine Client-App auszuführen, die 100 simulierte Bestellungen an Azure Event Hubs sendet:

    ```
    node ~/dp-203/Allfiles/labs/18/orderclient
    ```

7. Beobachten Sie die Bestelldaten, während sie gesendet werden. Jede Bestellung besteht aus einer Produkt-ID und einer Menge.
8. Nachdem die Bestellclient-App fertig ist, minimieren Sie den Cloud Shell-Bereich, und wechseln Sie zurück zur Registerkarte mit Synapse Studio.
9. Stellen Sie in Synapse Studio auf der Seite **Verwalten** sicher, dass Ihr dedizierter SQL-Pool den Status **Online** hat. Wechseln Sie dann zur Seite **Daten**, und erweitern Sie im Bereich **Arbeitsbereich** **SQL-Datenbank**, Ihren SQL-Pool **sql*xxxxxxx*** und **Tabellen**, um die Tabelle **dbo.FactOrder** anzuzeigen.
10. Wählen Sie im Menü **…** für die Tabelle **dbo.FactOrder** die Option **Neues SQL-Skript** > **Die ersten 100 Zeilen auswählen** aus, und überprüfen Sie die Ergebnisse. Beachten Sie, dass die Tabelle Spalten für **OrderDateTime**, **ProductID** und **Quantity** enthält, es jedoch derzeit keine Datenzeilen gibt.

### Erstellen eines Azure Stream Analytics-Auftrags zum Erfassen von Bestelldaten

1. Wechseln Sie zurück zur Browserregisterkarte mit dem Azure-Portal, und notieren Sie sich die Region, in der Ihre Ressourcengruppe **dp203-*xxxxxxx*** bereitgestellt wurde. Sie erstellen Ihren Stream Analytics-Auftrag in derselben <u>Region</u>.
2. Wählen Sie auf der **Startseite** von Azure **+ Ressource erstellen**, und suchen Sie nach `Stream Analytics job`. Erstellen Sie dann einen **Stream Analytics-Auftrag** mit den folgenden Eigenschaften:
    - **Grundlagen**:
        - **Abonnement**: Ihr Azure-Abonnement
        - **Ressourcengruppe**: Wählen Sie die vorhandene Ressourcengruppe **dp203-*xxxxxxx*** aus.
        - **Name**: `ingest-orders`
        - **Region**: Wählen Sie <u>dieselbe Region</u> aus, in der Ihr Synapse Analytics-Arbeitsbereich bereitgestellt wird.
        - **Hostumgebung**: Cloud
        - **Streamingeinheiten**: 1
    - **Speicher**:
        - **Speicherkonto hinzufügen**: Ausgewählt
        - **Abonnement**: Ihr Azure-Abonnement
        - **Speicherkonten**: Wählen Sie das Speicherkonto **datalake*xxxxxxx*** aus.
        - **Authentifizierungsmodus**: Verbindungszeichenfolge
        - **Private Daten im Speicherkonto sichern**: Ausgewählt
    - **Tags:**
        - *Keine*
3. Warten Sie, bis die Bereitstellung abgeschlossen ist, und wechseln Sie dann zur bereitgestellten Stream Analytics-Auftragsressource.

### Erstellen einer Eingabe für den Ereignis-Datenstrom

1. Wählen Sie auf der Übersichtsseite **ingest-orders** die Seite **Eingaben** aus. Verwenden Sie das Menü **Eingabe hinzufügen**, um eine **Event Hub**-Eingabe mit den folgenden Eigenschaften hinzuzufügen:
    - **Eingabealias**: `orders`
    - **Event Hub aus Ihren Abonnements auswählen**: Ausgewählt
    - **Abonnement**: Ihr Azure-Abonnement
    - **Event Hub-Namespace**: Wählen Sie den Event Hubs-Namespace **events*xxxxxxx*** aus.
    - **Event Hub-Name**: Wählen Sie den vorhandenen Event Hub **eventhub*xxxxxxx*** aus.
    - **Event Hub-Consumergruppe**: Wählen Sie **Vorhandene verwenden** und dann die Consumergruppe **$Default** aus.
    - **Authentifizierungsmodus**: Erstellen Sie eine systemseitig zugewiesene verwaltete Identität.
    - **Partitionsschlüssel**: *Leer lassen*
    - **Ereignisserialisierungsformat**: JSON
    - **Codierung**: UTF-8
2. Speichern Sie die Eingabe, und warten Sie, während sie erstellt wird. Es werden mehrere Benachrichtigungen angezeigt. Warten Sie auf die Benachrichtigung **Verbindungstest erfolgreich**.

### Erstellen einer Ausgabe für die SQL-Tabelle

1. Zeigen Sie die Seite **Ausgaben** für den Stream Analytics-Auftrag **ingest-orders** an. Verwenden Sie dann das Menü **Ausgabe hinzufügen**, um eine **Azure Synapse Analytics**-Ausgabe mit den folgenden Eigenschaften hinzuzufügen:
    - **Ausgabealias**: `FactOrder`
    - **Azure Synapse Analytics aus Ihren Abonnements auswählen**: Ausgewählt
    - **Abonnement**: Ihr Azure-Abonnement
    - **Datenbank**: Wählen Sie die Datenbank **sql*xxxxxxx* (synapse*xxxxxxx *)** aus.
    - **Authentifizierungsmodus**: SQL Server-Authentifizierung
    - **Benutzername**: SQLUser
    - **Kennwort**: *Das Kennwort, das Sie für Ihren SQL-Pool beim Ausführen des Setupskripts angegeben haben*
    - **Tabelle**: `FactOrder`
2. Speichern Sie die Ausgabe, und warten Sie, während sie erstellt wird. Es werden mehrere Benachrichtigungen angezeigt. Warten Sie auf die Benachrichtigung **Verbindungstest erfolgreich**.

### Erstellen einer Abfrage zum Erfassen des Ereignisdatenstroms

1. Zeigen Sie die Seite **Abfrage** für den Stream Analytics-Auftrag **ingest-orders** an. Warten Sie dann einige Augenblicke, bis die Eingabevorschau angezeigt wird (basierend auf den zuvor im Event Hub erfassten Bestellereignissen).
2. Beachten Sie, dass die Eingabedaten die Felder **ProductID** und **Menge** in den von der Client-App übermittelten Nachrichten sowie zusätzliche Event Hubs-Felder enthalten, einschließlich des Felds **EventProcessedUtcTime**, das angibt, wann das Ereignis dem Event Hub hinzugefügt wurde.
3. Ändern Sie die Standardabfrage wie folgt:

    ```
    SELECT
        EventProcessedUtcTime AS OrderDateTime,
        ProductID,
        Quantity
    INTO
        [FactOrder]
    FROM
        [orders]
    ```

    Beachten Sie, dass diese Abfrage Felder aus der Eingabe (Event Hub) abruft und direkt in die Ausgabe (SQL-Tabelle) schreibt.

4. Speichern Sie die Abfrage.

### Ausführen des Streamingauftrags zum Erfassen von Bestelldaten

1. Zeigen Sie die Seite **Übersicht** für den Stream Analytics-Auftrag **ingest-orders** an, und überprüfen Sie auf der Registerkarte **Eigenschaften** die **Eingaben**, **Abfrage**, **Ausgaben** und **Funktionen** für den Auftrag. Wenn die Anzahl der **Eingaben** und **Ausgaben** 0 ist, verwenden Sie die Schaltfläche **&#8635; Aktualisieren** auf der Seite **Übersicht**, um die Eingabe **orders** und die Ausgabe **FactTable** anzuzeigen.
2. Wählen Sie die Schaltfläche **&#9655; Starten**, und starten Sie jetzt den Streamingauftrag. Warten Sie auf eine Benachrichtigung, dass der Streamingauftrag erfolgreich gestartet wurde.
3. Öffnen Sie den Cloud Shell-Bereich erneut, und führen Sie dann den folgenden Befehl erneut aus, um weitere 100 Bestellungen zu senden.

    ```
    node ~/dp-203/Allfiles/labs/18/orderclient
    ```

4. Wechseln Sie während der Ausführung der Bestellclient-App zur Registerkarte mit Synapse Studio, und zeigen Sie die Abfrage an, die Sie zuvor ausgeführt haben, um die ersten 100 Zeilen aus der Tabelle **dbo.FactOrder** auszuwählen.
5. Verwenden Sie Schaltfläche **&#9655; Ausführen**, um die Abfrage erneut auszuführen, und überprüfen Sie, ob die Tabelle jetzt Bestelldaten aus dem Ereignisdatenstrom enthält (falls nicht, warten Sie eine Minute, und führen Sie die Abfrage erneut aus). Der Stream Analytics-Auftrag verschiebt alle neuen Ereignisdaten in die Tabelle, solange der Auftrag ausgeführt wird und Bestellereignisse an den Event Hub gesendet werden.
6. Halten Sie auf der Seite **Verwalten** den dedizierten SQL-Pool **sql*xxxxxxx*** an (um unnötige Azure-Gebühren zu vermeiden).
7. Kehren Sie zur Browserregisterkarte mit dem Azure-Portal zurück, und minimieren Sie den Cloud Shell-Bereich. Verwenden Sie dann die Schaltfläche **&#128454; Beenden**, um den Stream Analytics-Auftrag zu beenden, und warten Sie auf die Benachrichtigung, dass der Stream Analytics-Auftrag erfolgreich beendet wurde.

## Zusammenfassen von Streamingdaten in einem Data Lake

Bisher haben Sie erfahren, wie Sie mit einem Stream Analytics-Auftrag Nachrichten aus einer Streamingquelle in eine SQL-Tabelle aufnehmen können. Sehen wir uns nun an, wie Azure Stream Analytics zum Aggregieren von Daten über Zeitfenster verwendet wird – in diesem Fall, um die Gesamtzahl der verkauften Produkte alle 5 Sekunden zu berechnen. Außerdem werden wir untersuchen, wie Sie eine andere Art von Ausgabe für den Auftrag verwenden, indem Sie die Ergebnisse im CSV-Format in einem Data Lake-Blob-Speicher schreiben.

### Erstellen eines Azure Stream Analytics-Auftrags zum Aggregieren von Bestelldaten

1. Wählen Sie auf der **Startseite** im Azure-Portal **+ Ressource erstellen**, und suchen Sie nach `Stream Analytics job`. Erstellen Sie dann einen **Stream Analytics-Auftrag** mit den folgenden Eigenschaften:
    - **Grundlagen**:
        - **Abonnement**: Ihr Azure-Abonnement
        - **Ressourcengruppe**: Wählen Sie die vorhandene Ressourcengruppe **dp203-*xxxxxxx*** aus.
        - **Name**: `aggregate-orders`
        - **Region**: Wählen Sie <u>dieselbe Region</u> aus, in der Ihr Synapse Analytics-Arbeitsbereich bereitgestellt wird.
        - **Hostumgebung**: Cloud
        - **Streamingeinheiten**: 1
    - **Speicher**:
        - **Speicherkonto hinzufügen**: Ausgewählt
        - **Abonnement**: Ihr Azure-Abonnement
        - **Speicherkonten**: Wählen Sie das Speicherkonto **datalake*xxxxxxx*** aus.
        - **Authentifizierungsmodus**: Verbindungszeichenfolge
        - **Private Daten im Speicherkonto sichern**: Ausgewählt
    - **Tags:**
        - *Keine*

2. Warten Sie, bis die Bereitstellung abgeschlossen ist, und wechseln Sie dann zur bereitgestellten Stream Analytics-Auftragsressource.

### Erstellen einer Eingabe für die Rohdaten einer Bestellung

1. Wählen Sie auf der Übersichtsseite **aggregate-orders** die Seite **Eingaben** aus. Verwenden Sie das Menü **Eingabe hinzufügen**, um eine **Event Hub**-Eingabe mit den folgenden Eigenschaften hinzuzufügen:
    - **Eingabealias**: `orders`
    - **Event Hub aus Ihren Abonnements auswählen**: Ausgewählt
    - **Abonnement**: Ihr Azure-Abonnement
    - **Event Hub-Namespace**: Wählen Sie den Event Hubs-Namespace **events*xxxxxxx*** aus.
    - **Event Hub-Name**: Wählen Sie den vorhandenen Event Hub **eventhub*xxxxxxx*** aus.
    - **Event Hub-Consumergruppe**: Wählen Sie die vorhandene Consumergruppe **$Default** aus.
    - **Authentifizierungsmodus**: Erstellen Sie eine systemseitig zugewiesene verwaltete Identität.
    - **Partitionsschlüssel**: *Leer lassen*
    - **Ereignisserialisierungsformat**: JSON
    - **Codierung**: UTF-8
2. Speichern Sie die Eingabe, und warten Sie, während sie erstellt wird. Es werden mehrere Benachrichtigungen angezeigt. Warten Sie auf die Benachrichtigung **Verbindungstest erfolgreich**.

### Erstellen einer Ausgabe für den Data Lake-Speicher

1. Zeigen Sie die Seite **Ausgaben** für den Stream Analytics-Auftrag **aggregate-orders** an. Verwenden Sie dann das Menü **Ausgabe hinzufügen**, um eine **Blob-Speicher/ADLS Gen2**-Ausgabe mit den folgenden Eigenschaften hinzuzufügen:
    - **Ausgabealias**: `datalake`
    - **Blob-Speicher/ADLS Gen2 aus Ihren Abonnements auswählen**: Ausgewählt
    - **Abonnement**: Ihr Azure-Abonnement
    - **Speicherkonto**: Wählen Sie das Speicherkonto **datalake*xxxxxxx*** aus.
    - **Container**: Wählen Sie **Vorhandenen verwenden** aus und wählen Sie in der Liste den Container **Dateien** aus.
    - **Authentifizierungsmodus**: Verbindungszeichenfolge
    - **Ereignisserialisierungsformat**: CSV – Komma (,)
    - **Codierung**: UTF-8
    - **Schreibmodus**: Beim Eintreffen der Ergebnisse anfügen
    - **Pfadmuster**: `{date}`
    - **Datumsformat**: JJJJ/MM/TT
    - **Zeitformat**: *Nicht zutreffend*
    - **Mindestanzahl von Zeilen**: 20
    - **Maximale Zeit**: 0 Stunden, 1 Minute, 0 Sekunden
2. Speichern Sie die Ausgabe, und warten Sie, während sie erstellt wird. Es werden mehrere Benachrichtigungen angezeigt. Warten Sie auf die Benachrichtigung **Verbindungstest erfolgreich**.

### Erstellen einer Abfrage zum Aggregieren der Ereignisdaten

1. Zeigen Sie die Seite **Abfrage** für den Stream Analytics-Auftrag **aggregate-orders** an.
2. Ändern Sie die Standardabfrage wie folgt:

    ```
    SELECT
        DateAdd(second,-5,System.TimeStamp) AS StartTime,
        System.TimeStamp AS EndTime,
        ProductID,
        SUM(Quantity) AS Orders
    INTO
        [datalake]
    FROM
        [orders] TIMESTAMP BY EventProcessedUtcTime
    GROUP BY ProductID, TumblingWindow(second, 5)
    HAVING COUNT(*) > 1
    ```

    Beachten Sie, dass diese Abfrage **System.Timestamp** (basierend auf dem Feld **EventProcessedUtcTime**) verwendet, um den Anfang und das Ende jedes 5-sekündigen *rollierenden* (nicht überlappenden sequenziellen) Fensters zu definieren, in dem die Gesamtmenge für jede Produkt-ID berechnet wird.

3. Speichern Sie die Abfrage.

### Ausführen des Streamingauftrags zum Aggregieren von Bestelldaten

1. Zeigen Sie die Seite **Übersicht** für den Stream Analytics-Auftrag **aggregate-orders** an, und überprüfen Sie auf der Registerkarte **Eigenschaften** die **Eingaben**, **Abfrage**, **Ausgaben** und **Funktionen** für den Auftrag. Wenn die Anzahl der **Eingaben** und **Ausgaben** 0 ist, verwenden Sie die Schaltfläche **&#8635; Aktualisieren** auf der Seite **Übersicht**, um die Eingabe **orders** und die Ausgabe **datalake** anzuzeigen.
2. Wählen Sie die Schaltfläche **&#9655; Starten**, und starten Sie jetzt den Streamingauftrag. Warten Sie auf eine Benachrichtigung, dass der Streamingauftrag erfolgreich gestartet wurde.
3. Öffnen Sie den Cloud Shell-Bereich erneut, und führen Sie dann den folgenden Befehl erneut aus, um weitere 100 Bestellungen zu senden:

    ```
    node ~/dp-203/Allfiles/labs/18/orderclient
    ```

4. Wenn die Bestell-App fertig ist, minimieren Sie den Cloud Shell-Bereich. Wechseln Sie dann zur Registerkarte mit Synapse Studio, und erweitern Sie auf der Seite **Daten** auf der Registerkarte **Verknüpft** **Azure Data Lake Storage Gen2** > **synapse*xxxxxxx* (primary - datalake*xxxxxxx *)**, und wählen Sie den Container **Dateien (Primär)** aus.
5. Wenn der Container **Dateien** leer ist, warten Sie eine Minute, und verwenden Sie dann die Schaltfläche **&#8635 Aktualisieren**, um die Ansicht zu aktualisieren. Schließlich sollte ein Ordner, der nach dem aktuellem Jahr benannt ist, angezeigt werden. Dieser wiederum enthält Ordner für den Monat und den Tag.
6. Wählen Sie den Ordner für das Jahr und im Menü **Neues SQL-Skript** die Option **Die ersten 100 Zeilen auswählen** aus. Legen Sie dann den **Dateityp** auf **Textformat** fest, und wenden Sie die Einstellungen an.
7. Ändern Sie im daraufhin geöffneten Abfragebereich die Abfrage, um den Parameter `HEADER_ROW = TRUE` hinzuzufügen, wie hier gezeigt:

    ```sql
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/2023/**',
            FORMAT = 'CSV',
            PARSER_VERSION = '2.0',
            HEADER_ROW = TRUE
        ) AS [result]
    ```

8. Verwenden Sie die Schaltfläche **&#9655; Ausführen**, um die SQL-Abfrage auszuführen und die Ergebnisse anzuzeigen, die die Menge der einzelnen bestellten Produkte in Fünf-Sekunden-Zeiträumen angeben.
9. Kehren Sie zur Browserregisterkarte mit dem Azure-Portal zurück, und verwenden Sie dann die Schaltfläche **&#128454; Beenden**, um den Stream Analytics-Auftrag zu beenden, und warten Sie auf die Benachrichtigung, dass der Stream Analytics-Auftrag erfolgreich beendet wurde.

## Löschen von Azure-Ressourcen

Wenn Sie sich mit Azure Stream Analytics vertraut gemacht haben, sollten Sie die erstellten Ressourcen löschen, um unnötige Azure-Kosten zu vermeiden.

1. Schließen Sie die Registerkarte mit Azure Synapse Studio, und kehren Sie zum Azure-Portal zurück.
2. Wählen Sie auf der **Startseite** des Azure-Portals die Option **Ressourcengruppen** aus.
3. Wählen Sie die Ressourcengruppe **dp203-*xxxxxxx*** aus, die Ihre Azure Synapse-, Event Hubs- und Stream Analytics-Ressourcen enthält (nicht die verwaltete Ressourcengruppe).
4. Wählen Sie oben auf der Seite **Übersicht** für Ihre Ressourcengruppe die Option **Ressourcengruppe löschen** aus.
5. Geben Sie den Namen der Ressourcengruppe **dp203-*xxxxxxx*** ein, um zu bestätigen, dass Sie sie löschen möchten, und wählen Sie **Löschen** aus.

    Nach einigen Minuten werden die in dieser Übung erstellten Ressourcen gelöscht.
