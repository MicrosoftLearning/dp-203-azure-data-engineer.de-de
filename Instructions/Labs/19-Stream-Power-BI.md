---
lab:
  title: Erstellen eines Echtzeitberichts mit Azure Stream Analytics und Microsoft Power BI
  ilt-use: Suggested demo
---

# Erstellen eines Echtzeitberichts mit Azure Stream Analytics und Microsoft Power BI

Datenanalyselösungen umfassen häufig eine Anforderung zum Erfassen und Verarbeiten von *Datenströmen*. Die Verarbeitung von Datenströmen unterscheidet sich in der Hinsicht von der Batchverarbeitung, dass Datenströme im Allgemeinen *endlos* sind, d. h. es handelt sich um fortlaufende Datenquellen, die fortwährend und nicht in festen Intervallen verarbeitet werden müssen.

Azure Stream Analytics bietet einen Clouddienst, mit dem Sie eine *Abfrage* definieren können, die auf einem Datenstrom aus einer Streamingquelle wie etwa Azure Event Hubs oder einem Azure IoT Hub ausgeführt wird. Sie können eine Azure Stream Analytics-Abfrage verwenden, um einen Datenstrom zu verarbeiten und die Ergebnisse direkt zur Echtzeitvisualisierung an Microsoft Power BI zu senden.

In dieser Übung verwenden Sie Azure Stream Analytics, um einen Datenstrom von Bestelldaten zu verarbeiten, wie er etwa von einer Online-Einzelhandelsanwendung erzeugt werden könnte. Die Bestelldaten werden an Azure Event Hubs gesendet, von wo aus Ihr Azure Stream Analytics-Auftrag die Daten liest und zusammenfasst, bevor sie an Power BI gesendet werden, wo Sie die Daten in einem Bericht visualisieren.

Diese Übung dauert ca. **45** Minuten.

## Vorbereitung

Sie benötigen ein [Azure-Abonnement](https://azure.microsoft.com/free), in dem Sie Administratorzugriff besitzen.

Außerdem benötigen Sie Zugriff auf den Microsoft Power BI-Dienst. Möglicherweise wird dies von Ihrer Bildungseinrichtung oder Organisation bereits angeboten. Alternativ können Sie sich als [Einzelperson für den Power BI-Dienst registrieren](https://learn.microsoft.com/power-bi/fundamentals/service-self-service-signup-for-power-bi).

## Bereitstellen von Azure-Ressourcen

In dieser Übung benötigen Sie einen Azure Synapse Analytics-Arbeitsbereich mit Zugriff auf den Data Lake-Speicher und einen dedizierten SQL-Pool. Außerdem benötigen Sie einen Azure Event Hubs-Namespace, an den die Bestelldaten des Streamings gesendet werden können.

Sie verwenden eine Kombination aus einem PowerShell-Skript und einer ARM-Vorlage, um diese Ressourcen bereitzustellen.

1. Melden Sie sich beim [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com` an.
2. Verwenden Sie rechts neben der Suchleiste oben auf der Seite die Schaltfläche **[\>_]**, um eine neue Cloud Shell-Instanz im Azure-Portal zu erstellen. Wählen Sie eine ***PowerShell***-Umgebung aus, und erstellen Sie Speicher, falls Sie dazu aufgefordert werden. Die Cloud Shell bietet eine Befehlszeilenschnittstelle in einem Bereich am unteren Rand des Azure-Portals, wie hier gezeigt:

    ![Screenshot des Azure-Portals mit einem Cloud Shell-Bereich](./images/cloud-shell.png)

    > **Hinweis**: Wenn Sie zuvor eine Cloud Shell erstellt haben, die eine *Bash*-Umgebung verwendet, ändern Sie diese mithilfe des Dropdownmenüs oben links im Cloud Shell-Bereich zu ***PowerShell***.

3. Beachten Sie, dass Sie die Größe der Cloud Shell durch Ziehen der Trennzeichenleiste oben im Bereich ändern können oder den Bereich mithilfe der Symbole **&#8212;**, **&#9723;** und **X** oben rechts minimieren, maximieren und schließen können. Weitere Informationen zur Verwendung von Azure Cloud Shell finden Sie in der [Azure Cloud Shell-Dokumentation](https://docs.microsoft.com/azure/cloud-shell/overview).

4. Geben Sie im PowerShell-Bereich die folgenden Befehle ein, um das Repository mit dieser Übung zu klonen:

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Nachdem das Repository geklont wurde, geben Sie die folgenden Befehle ein, um in den Ordner für diese Übung zu wechseln. Führen Sie das darin enthaltene Skript **setup.ps1** aus:

    ```
    cd dp-203/Allfiles/labs/19
    ./setup.ps1
    ```

6. Wenn Sie dazu aufgefordert werden, wählen Sie aus, welches Abonnement Sie verwenden möchten (dies geschieht nur, wenn Sie Zugriff auf mehrere Azure-Abonnements haben).

7. Während Sie auf den Abschluss des Skripts warten, fahren Sie mit der nächsten Aufgabe fort.

## Erstellen eines Power BI-Arbeitsbereichs

Im Power BI-Dienst organisieren Sie DataSets, Berichte und andere Ressourcen in *Arbeitsbereichen*. Jeder Power BI-Benutzer und jede Power BI-Benutzerin verfügt über einen Standardarbeitsbereich mit dem Namen **Mein Arbeitsbereich**, den Sie in dieser Übung verwenden können. In der Regel empfiehlt es sich jedoch, einen Arbeitsbereich für jede separate Berichterstellungslösung zu erstellen, die Sie verwalten möchten.

1. Melden Sie sich beim Power BI-Dienst unter [https://app.powerbi.com/](https://app.powerbi.com/) mit Ihren Anmeldeinformationen für den Power BI-Dienst an.
2. Wählen Sie auf der Menüleiste auf der linken Seite **Arbeitsbereiche** aus (Symbol ähnelt &#128455;).
3. Erstellen Sie einen neuen Arbeitsbereich mit einem aussagekräftigen Namen (z. B. *mslearn-streaming*), und wählen Sie den Lizenzierungsmodus **Pro** aus.

    > **Hinweis**: Wenn Sie ein Testkonto verwenden, müssen Sie möglicherweise zusätzliche Testfeatures aktivieren.

4. Notieren Sie sich beim Anzeigen ihres Arbeitsbereichs die GUID (Globally Unique Identifier) in der Seiten-URL (die `https://app.powerbi.com/groups/<GUID>/list` ähneln sollte). Sie werden diese GUID später benötigen.

## Verwenden von Azure Stream Analytics zum Verarbeiten von Streamingdaten

Ein Azure Stream Analytics-Auftrag definiert eine unbefristete Abfrage, die auf Streamingdaten aus einer oder mehreren Eingaben angewendet wird und die Ergebnisse an eine oder mehrere Ausgaben sendet.

### Erstellen eines Stream Analytics-Auftrags

1. Wechseln Sie zurück zur Browserregisterkarte mit dem Azure-Portal, und notieren Sie sich nach Abschluss des Skripts die Region, in der Ihre Ressourcengruppe **dp203-*xxxxxxx*** bereitgestellt wurde.
2. Wählen Sie auf der **Startseite** des Azure-Portals **+ Ressource erstellen** aus, und suchen Sie nach `Stream Analytics job`. Erstellen Sie dann einen **Stream Analytics-Auftrag** mit den folgenden Eigenschaften:
    - **Abonnement**: Ihr Azure-Abonnement
    - **Ressourcengruppe**: Wählen Sie die vorhandene Ressourcengruppe **dp203-*xxxxxxx*** aus.
    - **Name**: `stream-orders`
    - **Region**: Wählen Sie die Region aus, in der Ihr Synapse Analytics-Arbeitsbereich bereitgestellt wird.
    - **Hostumgebung**: Cloud
    - **Streamingeinheiten**: 1
3. Warten Sie, bis die Bereitstellung abgeschlossen ist, und wechseln Sie dann zur bereitgestellten Stream Analytics-Auftragsressource.

### Erstellen einer Eingabe für den Ereignis-Datenstrom

1. Wählen Sie auf der Übersichtsseite **stream-orders** die Seite **Eingaben** aus, und verwenden Sie das Menü **Eingabe hinzufügen**, um eine **Event Hub**-Eingabe mit den folgenden Eigenschaften hinzuzufügen:
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

### Erstellen einer Ausgabe für den Power BI-Arbeitsbereich

1. Zeigen Sie die Seite **Ausgaben** für den Stream Analytics-Auftrag **stream-orders** an. Verwenden Sie dann das Menü **Ausgabe hinzufügen**, um eine **Power BI**-Ausgabe mit den folgenden Eigenschaften hinzuzufügen:
    - **Ausgabealias**: `powerbi-dataset`
    - **Power BI-Einstellungen manuell auswählen**: Ausgewählt
    - **Gruppenarbeitsbereich**: *Die GUID für Ihren Arbeitsbereich*
    - **Authentifizierungsmodus**: *Wählen Sie* **Benutzertoken** *aus, und verwenden Sie dann die Schaltfläche* **Autorisieren** *unten, um sich bei Ihrem Power BI-Konto anzumelden.*
    - **DataSet-Name**: `realtime-data`
    - **Tabellenname**: `orders`

2. Speichern Sie die Ausgabe, und warten Sie, während sie erstellt wird. Es werden mehrere Benachrichtigungen angezeigt. Warten Sie auf die Benachrichtigung **Verbindungstest erfolgreich**.

### Erstellen einer Abfrage zum Zusammenfassen des Ereignisdatenstroms

1. Zeigen Sie die Seite **Abfrage** für den Stream Analytics-Auftrag **stream-orders** an.
2. Ändern Sie die Standardabfrage wie folgt:

    ```
    SELECT
        DateAdd(second,-5,System.TimeStamp) AS StartTime,
        System.TimeStamp AS EndTime,
        ProductID,
        SUM(Quantity) AS Orders
    INTO
        [powerbi-dataset]
    FROM
        [orders] TIMESTAMP BY EventEnqueuedUtcTime
    GROUP BY ProductID, TumblingWindow(second, 5)
    HAVING COUNT(*) > 1
    ```

    Beachten Sie, dass diese Abfrage **System.Timestamp** (basierend auf dem Feld **EventEnqueuedUtcTime**) verwendet, um den Anfang und das Ende jedes 5-sekündigen *rollierenden* (nicht überlappenden sequenziellen) Fensters zu definieren, in dem die Gesamtmenge für jede Produkt-ID berechnet wird.

3. Speichern Sie die Abfrage.

### Ausführen des Streamingauftrags zum Verarbeiten von Bestelldaten

1. Zeigen Sie die Seite **Übersicht** für den Stream Analytics-Auftrag **stream-orders** an, und überprüfen Sie auf der Registerkarte **Eigenschaften** die **Eingaben**, **Abfrage**, **Ausgaben** und **Funktionen** für den Auftrag. Wenn die Anzahl der **Eingaben** und **Ausgaben** 0 ist, verwenden Sie die Schaltfläche **&#8635; Aktualisieren** auf der Seite **Übersicht**, um die Eingabe **orders** und die Ausgabe **powerbi-dataset** anzuzeigen.
2. Wählen Sie die Schaltfläche **&#9655; Starten**, und starten Sie jetzt den Streamingauftrag. Warten Sie auf eine Benachrichtigung, dass der Streamingauftrag erfolgreich gestartet wurde.
3. Öffnen Sie den Cloud Shell-Bereich erneut, und führen Sie den folgenden Befehl aus, um 100 Bestellungen zu senden.

    ```
    node ~/dp-203/Allfiles/labs/19/orderclient
    ```

4. Wechseln Sie während der Ausführung der Bestellclient-App zur Browserregisterkarte der Power BI-App, und zeigen Sie Ihren Arbeitsbereich an.
5. Aktualisieren Sie die Seite der Power BI-App, bis das DataSet **realtime-data** in Ihrem Arbeitsbereich angezeigt wird. Dieses DataSet wird vom Azure Stream Analytics-Auftrag generiert.

## Visualisieren der Streamingdaten in Power BI

Nachdem Sie nun über ein DataSet für die Bestelldaten des Streamings verfügen, können Sie ein Power BI-Dashboard erstellen, das es visuell darstellt.

1. Kehren Sie zur Browserregisterkarte mit Power BI zurück.

2. Wählen Sie im Dropdownmenü **+ Neu** für Ihren Arbeitsbereich **Dashboard** aus, und erstellen Sie ein neues Dashboard mit dem Namen **Bestellverfolgung**.

3. Wählen Sie im Dashboard **Bestellverfolgung** das Menü **&#9999;&#65039; Bearbeiten** und dann **+ Kachel hinzufügen** aus. Wählen Sie dann im Bereich **Kachel hinzufügen** die Option **Benutzerdefinierte Streamingdaten** und dann **Weiter** aus.

4. Wählen Sie im Bereich **Kachel für benutzerdefinierte Streamingdaten hinzufügen** unter **Ihre DataSets** das DataSet **realtime-data** und **Weiter** aus.

5. Ändern Sie den Standardvisualisierungstyp in **Liniendiagramm**. Legen Sie dann die folgenden Eigenschaften fest, und wählen Sie **Weiter** aus:
    - **Achse**: EndTime
    - **Wert**: Bestellungen
    - **Das anzuzeigende Zeitfenster**: 1 Minute

6. Legen Sie im Bereich **Kacheldetails** den **Titel** auf **Anzahl der Echtzeitbestellungen** fest, und wählen Sie **Übernehmen**.

7. Wechseln Sie zurück zur Browserregisterkarte mit dem Azure-Portal, und öffnen Sie ggf. den Cloud Shell-Bereich erneut. Führen Sie dann den folgenden Befehl erneut aus, um weitere 100 Bestellungen zu senden.

    ```
    node ~/dp-203/Allfiles/labs/19/orderclient
    ```

8. Wechseln Sie während der Ausführung des Skripts zur Bestellübermittlung zurück zur Browserregisterkarte mit dem Power BI-Dashboard **Bestellverfolgung**, und beobachten Sie, wie die Visualisierung aktualisiert wird, um die neuen Bestelldaten anzuzeigen, während sie vom Stream Analytics-Auftrag (der weiterhin ausgeführt werden sollte) verarbeitet werden.

    ![Screenshot eines Power BI-Berichts, der einen Echtzeitdatenstrom mit Bestelldaten anzeigt.](./images/powerbi-line-chart.png)

    Sie können das Skript **orderclient** erneut ausführen und die erfassten Daten im Echtzeit-Dashboard beobachten.

## Löschen von Ressourcen

Wenn Sie sich mit Azure Stream Analytics und Power BI vertraut gemacht haben, sollten Sie die erstellten Ressourcen löschen, um unnötige Azure-Kosten zu vermeiden.

1. Schließen Sie die Browserregisterkarte mit dem Power BI-Bericht. Wählen Sie dann im Bereich **Arbeitsbereiche** im Menü **&#8942;** für Ihren Arbeitsbereich **Arbeitsbereichseinstellungen** aus, und löschen Sie den Arbeitsbereich.
2. Kehren Sie zur Browserregisterkarte mit dem Azure-Portal zurück, schließen Sie den Cloud Shell-Bereich, und verwenden Sie die Schaltfläche **&#128454; Beenden**, um den Stream Analytics-Auftrag zu beenden. Warten Sie auf eine Benachrichtigung, dass der Stream Analytics-Auftrag erfolgreich beendet wurde.
3. Wählen Sie auf der **Startseite** des Azure-Portals die Option **Ressourcengruppen** aus.
4. Wählen Sie die Ressourcengruppe **dp203-*xxxxxxx*** aus, die Ihre Azure Event Hub- und Stream Analytics-Ressourcen enthält.
5. Wählen Sie oben auf der Seite **Übersicht** für Ihre Ressourcengruppe die Option **Ressourcengruppe löschen** aus.
6. Geben Sie den Namen der Ressourcengruppe **dp203-*xxxxxxx*** ein, um zu bestätigen, dass Sie sie löschen möchten, und wählen Sie **Löschen** aus.

    Nach einigen Minuten werden die in dieser Übung erstellten Ressourcen gelöscht.
