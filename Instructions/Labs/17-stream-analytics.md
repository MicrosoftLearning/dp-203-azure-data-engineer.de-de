---
lab:
  title: Erste Schritte mit Azure Stream Analytics
  ilt-use: Suggested demo
---

# Erste Schritte mit Azure Stream Analytics

In dieser Übung stellen Sie einen Azure Stream Analytics-Auftrag in Ihrem Azure-Abonnement bereit und verwenden ihn zum Abfragen und Zusammenfassen von Echtzeit-Ereignisdaten und zum Speichern der Ergebnisse in Azure Storage.

Diese Übung dauert ca. **15** Minuten.

## Vor der Installation

Sie benötigen ein [Azure-Abonnement](https://azure.microsoft.com/free), in dem Sie Administratorzugriff besitzen.

## Bereitstellen von Azure-Ressourcen

In dieser Übung erfassen Sie einen Datenstrom simulierter Verkaufstransaktionsdaten, verarbeiten sie und speichern die Ergebnisse in einem Blob-Container in Azure Storage. Sie benötigen einen Azure Event Hubs-Namespace, an den Streamingdaten gesendet werden können, und ein Azure Storage-Konto, in dem die Ergebnisse der Datenstromverarbeitung gespeichert werden.

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
    cd dp-203/Allfiles/labs/17
    ./setup.ps1
    ```

6. Wenn Sie dazu aufgefordert werden, wählen Sie aus, welches Abonnement Sie verwenden möchten (dies geschieht nur, wenn Sie Zugriff auf mehrere Azure-Abonnements haben).
7. Warten Sie, bis das Skript abgeschlossen ist. Dies dauert in der Regel etwa 5 Minuten, in einigen Fällen kann es jedoch länger dauern. Während Sie warten, lesen Sie den Artikel [Willkommen bei Azure Stream Analytics](https://learn.microsoft.com/azure/stream-analytics/stream-analytics-introduction) in der Dokumentation zu Azure Stream Analytics.

## Anzeigen der Streamingdatenquelle

Bevor Sie einen Azure Stream Analytics-Auftrag zum Verarbeiten von Echtzeitdaten erstellen, werfen wir einen Blick auf den Datenstrom, den er abfragen muss.

1. Wenn die Ausführung des Setupskripts abgeschlossen ist, ändern Sie die Größe des Cloud Shell-Bereichs oder minimieren Sie ihn, damit Sie das Azure-Portal sehen können (Sie kehren später zur Cloud Shell zurück). Wechseln Sie dann im Azure-Portal zur erstellten Ressourcengruppe **dp203-*xxxxxxx***, und beachten Sie, dass diese Ressourcengruppe ein Azure Storage-Konto und einen Event Hubs-Namespace enthält.

    Notieren Sie sich den **Speicherort**, an dem die Ressourcen bereitgestellt wurden. Sie werden zu einem späteren Zeitpunkt einen Azure Stream Analytics-Auftrag an demselben Speicherort erstellen.

2. Öffnen Sie den Cloud Shell-Bereich erneut, und geben Sie den folgenden Befehl ein, um eine Client-App auszuführen, die 100 simulierte Bestellungen an Azure Event Hubs sendet:

    ```
    node ~/dp-203/Allfiles/labs/17/orderclient
    ```

3. Beobachten Sie die Bestelldaten, während sie gesendet werden. Jede Bestellung besteht aus einer Produkt-ID und einer Menge. Die App wird nach dem Senden von 1000 Bestellungen beendet, was etwa eine Minute dauert.

## Erstellen eines Azure Stream Analytics-Auftrags

Sie können jetzt einen Azure Stream Analytics-Auftrag erstellen, um die Verkaufstransaktionsdaten bei ihrem Eintreffen im Event Hub zu verarbeiten.

1. Wählen Sie im Azure-Portal auf der Seite **dp203-*xxxxxxx*** die Option **+ Erstellen** aus, und suchen Sie nach `Stream Analytics job`. Erstellen Sie dann einen **Stream Analytics-Auftrag** mit den folgenden Eigenschaften:
    - **Grundlagen**:
        - **Abonnement**: Ihr Azure-Abonnement
        - **Ressourcengruppe**: Wählen Sie die vorhandene Ressourcengruppe **dp203-*xxxxxxx*** aus.
        - **Name**: `process-orders`
        - **Region**: Wählen Sie die Region aus, in der Ihre anderen Azure-Ressourcen bereitgestellt werden.
        - **Hostumgebung**: Cloud
        - **Streamingeinheiten**: 1
    - **Speicher**:
        - **Speicherkonto hinzufügen**: Nicht ausgewählt
    - **Tags:**
        - *Keine*
2. Warten Sie, bis die Bereitstellung abgeschlossen ist, und wechseln Sie dann zur bereitgestellten Stream Analytics-Auftragsressource.

## Erstellen einer Eingabe für den Ereignisdatenstrom

Ihr Azure Stream Analytics-Auftrag muss Eingabedaten aus dem Event Hub abrufen, in dem die Bestellungen aufgezeichnet werden.

1. Wählen Sie auf der Übersichtsseite **process-orders** die Option **Eingabe hinzufügen** aus. Verwenden Sie dann auf der Seite **Eingaben** das Menü **Streamingeingabe hinzufügen**, um eine **Event Hub**-Eingabe mit den folgenden Eigenschaften hinzuzufügen:
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

## Erstellen einer Ausgabe für den BLOB-Speicher

Sie speichern die aggregierten Bestelldaten im JSON-Format in einem Azure Storage-Blob-Container.

1. Zeigen Sie die Seite **Ausgaben** für den Stream Analytics-Auftrag **process-orders** an. Verwenden Sie dann das Menü **Hinzufügen**, um eine **Blob-Speicher/ADLS Gen2**-Ausgabe mit den folgenden Eigenschaften hinzuzufügen:
    - **Ausgabealias**: `blobstore`
    - **Blob-Speicher/ADLS Gen2 aus Ihren Abonnements auswählen**: Ausgewählt
    - **Abonnement**: Ihr Azure-Abonnement
    - **Speicherkonto**: Wählen Sie das Speicherkonto **store*xxxxxxx*** aus.
    - **Container**: Wählen Sie den vorhandenen **Daten**-Container aus.
    - **Authentifizierungstyp**: Verwaltete Identität: Systemseitig zugewiesen
    - **Ereignisserialisierungsformat**: JSON
    - **Format**: Separate Zeile
    - **Codierung**: UTF-8
    - **Schreibmodus**: Beim Eintreffen der Ergebnisse anfügen
    - **Pfadmuster**: `{date}`
    - **Datumsformat**: JJJJ/MM/TT
    - **Zeitformat**: *Nicht zutreffend*
    - **Mindestanzahl von Zeilen**: 20
    - **Maximale Zeit**: 0 Stunden, 1 Minute, 0 Sekunden
2. Speichern Sie die Ausgabe, und warten Sie, während sie erstellt wird. Es werden mehrere Benachrichtigungen angezeigt. Warten Sie auf die Benachrichtigung **Verbindungstest erfolgreich**.

## Erstellen einer Abfrage

Nachdem Sie nun eine Eingabe und eine Ausgabe für Ihren Azure Stream Analytics-Auftrag definiert haben, können Sie eine Abfrage verwenden, um Daten aus der Eingabe auszuwählen, zu filtern und zu aggregieren und die Ergebnisse an die Ausgabe zu senden.

1. Zeigen Sie die Seite **Abfrage** für den Stream Analytics-Auftrag **process-orders** an. Warten Sie dann einige Augenblicke, bis die Eingabevorschau angezeigt wird (basierend auf den zuvor im Event Hub erfassten Bestellereignissen).
2. Beachten Sie, dass die Eingabedaten die Felder **ProductID** und **Menge** in den von der Client-App übermittelten Nachrichten sowie zusätzliche Event Hubs-Felder enthalten, einschließlich des Felds **EventProcessedUtcTime**, das angibt, wann das Ereignis dem Event Hub hinzugefügt wurde.
3. Ändern Sie die Standardabfrage wie folgt:

    ```
    SELECT
        DateAdd(second,-10,System.TimeStamp) AS StartTime,
        System.TimeStamp AS EndTime,
        ProductID,
        SUM(Quantity) AS Orders
    INTO
        [blobstore]
    FROM
        [orders] TIMESTAMP BY EventProcessedUtcTime
    GROUP BY ProductID, TumblingWindow(second, 10)
    HAVING COUNT(*) > 1
    ```

    Beachten Sie, dass diese Abfrage **System.Timestamp** (basierend auf dem Feld **EventProcessedUtcTime**) verwendet, um den Anfang und das Ende jedes 10-sekündigen *rollierenden* (nicht überlappenden sequenziellen) Fensters zu definieren, in dem die Gesamtmenge für jede Produkt-ID berechnet wird.

4. Verwenden Sie Schaltfläche **&#9655; Abfrage testen**, um die Abfrage zu überprüfen, und stellen Sie sicher, dass der Status **Testergebnisse** den Wert **Erfolgreich** hat (auch wenn keine Zeilen zurückgegeben werden).
5. Speichern Sie die Abfrage.

## Ausführen des Streamingauftrags

Jetzt können Sie den Auftrag ausführen und einige Echtzeit-Bestelldaten verarbeiten.

1. Zeigen Sie die Seite **Übersicht** für den Stream Analytics-Auftrag **process-orders** an, und überprüfen Sie auf der Registerkarte **Eigenschaften** die **Eingaben**, **Abfrage**, **Ausgaben** und **Funktionen** für den Auftrag. Wenn die Anzahl der **Eingaben** und **Ausgaben** 0 ist, verwenden Sie die Schaltfläche **&#8635; Aktualisieren** auf der Seite **Übersicht**, um die Eingabe **orders** und die Ausgabe **blobstore** anzuzeigen.
2. Wählen Sie die Schaltfläche **&#9655; Starten**, und starten Sie jetzt den Streamingauftrag. Warten Sie auf eine Benachrichtigung, dass der Streamingauftrag erfolgreich gestartet wurde.
3. Öffnen Sie den Cloud Shell-Bereich erneut, stellen Sie bei Bedarf erneut eine Verbindung her, und führen Sie dann den folgenden Befehl erneut aus, um weitere 1000 Bestellungen zu senden.

    ```
    node ~/dp-203/Allfiles/labs/17/orderclient
    ```

4. Kehren Sie während der Ausführung der App im Azure-Portal auf die Seite für die Ressourcengruppe **dp203-*xxxxxxx*** zurück, und wählen Sie das Speicherkonto **store*xxxxxxxxxxxx*** aus.
6. Wählen Sie im Bereich auf der linken Seite des Blatts „Speicherkonto“ die Registerkarte **Container** aus.
7. Öffnen Sie den Container **Daten**, und verwenden Sie die Schaltfläche **&#8635; Aktualisieren**, um die Ansicht zu aktualisieren, bis ein Ordner mit dem Namen des aktuellen Jahres angezeigt wird.
8. Navigieren Sie im Container **Daten** durch die Ordnerhierarchie, die den Ordner für das aktuelle Jahr mit Unterordnern für den Monat und den Tag enthält.
9. Suchen Sie im Ordner für die Stunde die erstellte Datei, die einen ähnlichen Namen haben sollte wie **0_xxxxxxxxxxxxxxxx.json**.
10. Wählen Sie im Menü **…** für die Datei (rechts von den Details) die Option **Anzeigen/bearbeiten** aus, und überprüfen Sie den Inhalt der Datei. Dieser sollte aus einem JSON-Datensatz für jeden 10-Sekunden-Zeitraum bestehen, der die Anzahl der für jede Produkt-ID verarbeiteten Bestellungen anzeigt, etwa so:

    ```
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":6,"Orders":13.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":8,"Orders":15.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":5,"Orders":15.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":1,"Orders":16.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":3,"Orders":10.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":2,"Orders":25.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":7,"Orders":13.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":4,"Orders":12.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":10,"Orders":19.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":9,"Orders":8.0}
    {"StartTime":"2022-11-23T18:16:35.0000000Z","EndTime":"2022-11-23T18:16:45.0000000Z","ProductID":6,"Orders":41.0}
    {"StartTime":"2022-11-23T18:16:35.0000000Z","EndTime":"2022-11-23T18:16:45.0000000Z","ProductID":8,"Orders":29.0}
    ...
    ```

11. Warten Sie im Azure Cloud Shell-Bereich, bis die Bestellclient-App fertig ist.
12. Aktualisieren Sie im Azure-Portal die Datei, um alle erzeugten Ergebnisse anzuzeigen.
13. Kehren Sie zur Ressourcengruppe **dp203-*xxxxxxx*** zurück, und öffnen Sie erneut den Stream Analytics-Auftrag **process-orders**.
14. Verwenden Sie die im oberen Bereich der Seite für den Stream Analytics-Auftrag angezeigte Schaltfläche **&#11036; Beenden**, um den Auftrag zu beenden. Bestätigen Sie den Vorgang, wenn Sie dazu aufgefordert werden.

## Löschen von Azure-Ressourcen

Wenn Sie sich mit Azure Stream Analytics vertraut gemacht haben, sollten Sie die erstellten Ressourcen löschen, um unnötige Azure-Kosten zu vermeiden.

1. Wählen Sie auf der **Startseite** des Azure-Portals die Option **Ressourcengruppen** aus.
2. Wählen Sie die Ressourcengruppe **dp203-*xxxxxxx*** aus, die Ihre Azure Storage-, Event Hubs- und Stream Analytics-Ressourcen enthält.
3. Wählen Sie oben auf der Seite **Übersicht** für Ihre Ressourcengruppe die Option **Ressourcengruppe löschen** aus.
4. Geben Sie den Namen der Ressourcengruppe **dp203-*xxxxxxx*** ein, um zu bestätigen, dass Sie sie löschen möchten, und wählen Sie **Löschen** aus.

    Nach einigen Minuten werden die in dieser Übung erstellten Ressourcen gelöscht.
