---
lab:
  title: Verwenden eines Apache Spark-Notebooks in einer Pipeline
  ilt-use: Lab
---

# Verwenden eines Apache Spark-Notebooks in einer Pipeline

In dieser Übung erstellen wir eine Azure Synapse Analytics-Pipeline, die eine Aktivität zum Ausführen eines Apache Spark-Notebooks enthält.

Diese Übung dauert ca. **30** Minuten.

## Vorbereitung

Sie benötigen ein [Azure-Abonnement](https://azure.microsoft.com/free), in dem Sie Administratorzugriff besitzen.

## Bereitstellen eines Azure Synapse Analytics-Arbeitsbereichs

Sie benötigen einen Azure Synapse Analytics-Arbeitsbereich mit Zugriff auf den Data Lake-Speicher und einen Spark-Pool.

In dieser Übung verwenden Sie eine Kombination aus einem PowerShell-Skript und einer ARM-Vorlage, um einen Azure Synapse Analytics-Arbeitsbereich bereitzustellen.

1. Melden Sie sich beim [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com` an.
2. Verwenden Sie rechts neben der Suchleiste oben auf der Seite die Schaltfläche **[\>_]**, um eine neue Cloud Shell-Instanz im Azure-Portal zu erstellen. Wählen Sie eine ***PowerShell***-Umgebung aus, und erstellen Sie Speicher, falls Sie dazu aufgefordert werden. Die Cloud Shell bietet eine Befehlszeilenschnittstelle in einem Bereich am unteren Rand des Azure-Portals, wie hier gezeigt:

    ![Azure-Portal mit einem Cloud Shell-Bereich](./images/cloud-shell.png)

    > **Hinweis**: Wenn Sie zuvor eine Cloud Shell erstellt haben, die eine *Bash*-Umgebung verwendet, ändern Sie diese mithilfe des Dropdownmenüs oben links im Cloud Shell-Bereich zu ***PowerShell***.

3. Beachten Sie, dass Sie die Größe der Cloud Shell durch Ziehen der Trennzeichenleiste oben im Bereich ändern oder den Bereich mithilfe der Symbole „—“, **&#9723;** und **X** oben rechts minimieren, maximieren und schließen können. Weitere Informationen zur Verwendung von Azure Cloud Shell finden Sie in der [Azure Cloud Shell-Dokumentation](https://docs.microsoft.com/azure/cloud-shell/overview).

4. Geben Sie im PowerShell-Bereich die folgenden Befehle ein, um dieses Repository zu klonen:

    ```powershell
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Nachdem das Repository geklont wurde, geben Sie die folgenden Befehle ein, um in den Ordner für diese Übung zu wechseln. Führen Sie das darin enthaltene Skript **setup.ps1** aus:

    ```powershell
    cd dp-203/Allfiles/labs/11
    ./setup.ps1
    ```
    
6. Wenn Sie dazu aufgefordert werden, wählen Sie aus, welches Abonnement Sie verwenden möchten (dies geschieht nur, wenn Sie Zugriff auf mehrere Azure-Abonnements haben).
7. Wenn Sie dazu aufgefordert werden, geben Sie ein geeignetes Kennwort ein, das für Ihren Azure Synapse SQL-Pool festgelegt werden soll.

    > **Hinweis**: Merken Sie sich unbedingt dieses Kennwort!

8. Warten Sie, bis das Skript abgeschlossen ist. Dies dauert in der Regel etwa 10 Minuten, in einigen Fällen kann es jedoch länger dauern. Während Sie warten, lesen Sie den Artikel [Azure Synapse-Pipelines](https://learn.microsoft.com/en-us/azure/data-factory/concepts-data-flow-performance-pipelines) in der Dokumentation zu Azure Synapse Analytics.

## Interaktives Ausführen eines Spark-Notebooks

Bevor Sie einen Datentransformationsprozess mit einem Notebook automatisieren, kann es hilfreich sein, das Notebook interaktiv auszuführen, um den Prozess, den Sie später automatisieren werden, besser zu verstehen.

1. Wechseln Sie nach Abschluss des Skripts im Azure-Portal zur erstellten Ressourcengruppe dp203-xxxxxxx, und wählen Sie Ihren Synapse-Arbeitsbereich aus.
2. Wählen Sie auf der Seite **Übersicht** für Ihren Synapse-Arbeitsbereich in der Karte **Synapse Studio öffnen** die Option **Öffnen** aus, um Synapse Studio in einer neuen Browserregisterkarte zu öffnen. Melden Sie sich an, wenn Sie dazu aufgefordert werden.
3. Verwenden Sie im linken Bereich von Synapse Studio das Symbol ››, um das Menü zu erweitern. Dadurch werden die verschiedenen Seiten in Synapse Studio angezeigt.
4. Zeigen Sie auf der Seite **Daten** die Registerkarte „Verknüpft“ an, und stellen Sie sicher, dass Ihr Arbeitsbereich einen Link zu Ihrem Azure Data Lake Storage Gen2-Speicherkonto enthält, dessen Name **synapsexxxxxxx (Primary - datalakexxxxx)** ähneln sollte.
5. Erweitern Sie Ihr Speicherkonto, und stellen Sie sicher, dass es einen Dateisystemcontainer mit dem Namen **files (primary)** enthält.
6. Wählen Sie den Dateicontainer aus, und beachten Sie, dass er einen Ordner mit dem Namen **data** enthält, der die zu transformierenden Datendateien enthält.
7. Öffnen Sie den Ordner **data**** und zeigen Sie die darin enthaltenen CSV-Dateien an. Klicken Sie mit der rechten Maustaste auf eine der Dateien, und wählen Sie **Vorschau** aus, um ein Beispiel der Daten anzuzeigen. Schließen Sie die Vorschau, wenn Sie fertig sind.
8. Erweitern Sie in Synapse Studio auf der Seite **Entwickeln** die Option **Notebooks**, und öffnen Sie das Notebook **Spark Transform**.

    > **Hinweis**: Wenn Sie feststellen, dass das Notebook während des Ausführungsskripts nicht hochgeladen wird, sollten Sie die Datei mit dem Namen Spark Transform.ipynb von GitHub [Allfiles/labs/11/notebooks](https://github.com/MicrosoftLearning/dp-203-azure-data-engineer/tree/master/Allfiles/labs/11/notebooks) herunterladen und sie in Synapse hochladen.

9. Überprüfen Sie den im Notebook enthaltenen Code, und beachten Sie, dass er:
    - Eine Variable festlegt, um einen eindeutigen Ordnernamen zu definieren.
    - Die CSV-Bestelldaten aus dem Ordner **/data** liest.
    - Die Daten durch Aufteilen des Kundennamens in mehrere Felder transformiert.
    - Die transformierten Daten im Parquet-Format im eindeutig benannten Ordner speichert.
10. Fügen Sie das Notebook über die Symbolleiste des Notebooks an Ihren Spark-Pool **spark*xxxxxxx*** an, und verwenden Sie dann die Schaltfläche **&#9655; Alle ausführen**, um alle Codezellen im Notebook auszuführen.
  
    Es kann einige Minuten dauern, bis die Spark-Sitzung starten und die Codezellen ausführen kann.

11. Nachdem alle Notebookzellen ausgeführt wurden, notieren Sie sich den Namen des Ordners, in dem die transformierten Daten gespeichert wurden.
12. Wechseln Sie zur Registerkarte **Dateien** (die noch geöffnet sein sollte), und zeigen Sie den Stammordner **Dateien** an. Wählen Sie ggf. im Menü **Mehr** die Option **Aktualisieren** aus, um den neuen Ordner anzuzeigen. Öffnen Sie ihn dann, um zu überprüfen, ob er Parquet-Dateien enthält.
13. Kehren Sie zum Ordner Stammordner **Dateien** zurück, und wählen Sie dann den vom Notebook generierten eindeutig benannten Ordner aus, und wählen Sie im Menü **Neues SQL-Skript** die Option **Die ersten 100 Zeilen** aus.
14. Legen Sie im Bereich **Die ersten 100 Zeilen** den Dateityp auf **Parquet-Format** fest, und wenden Sie die Änderung an.
15. Verwenden Sie im daraufhin geöffneten Bereich „Neues SQL-Skript“ die Schaltfläche **&#9655; Ausführen**, um den SQL-Code auszuführen, und stellen Sie sicher, dass er die transformierten Bestelldaten zurückgibt.

## Ausführen des Notebooks in einer Pipeline

Nachdem Sie den Transformationsprozess verstanden haben, können Sie ihn automatisieren, indem Sie das Notebook in einer Pipeline kapseln.

### Erstellen einer Parameterzelle

1. Kehren Sie in Synapse Studio zur Registerkarte **Spark Transform** zurück, die das Notebook enthält, und wählen Sie in der Symbolleiste im Menü **...** am rechten Ende die Option **Ausgabe löschen** aus.
2. Wählen Sie die erste Codezelle (die den Code zum Festlegen der Variable **folderName** enthält) aus.
3. Wählen Sie in der Popupsymbolleiste oben rechts in der Codezelle im Menü **...** die Option **\[@] Parameterzelle ein/aus** aus. Vergewissern Sie sich, dass das Wort **Parameter** unten rechts in der Zelle angezeigt wird.
4. Verwenden Sie die Schaltfläche **Veröffentlichen** in der Symbolleiste, um die Änderungen zu speichern.

### Erstellen einer Pipeline

1. Wählen Sie in Synapse Studio die Seite **Integrieren** aus. Wählen Sie dann im Menü **+** die Option **Pipeline** aus, um eine neue Pipeline zu erstellen.
2. Ändern Sie im Bereich **Eigenschaften** für die neue Pipeline den Namen von **Pipeline1** in **Verkaufsdaten transfom**. Verwenden Sie dann die Schaltfläche **Eigenschaften** oberhalb des Bereichs **Eigenschaften**, um sie auszublenden.
3. Erweitern Sie im Bereich **Aktivitäten** die Option **Synapse**, und ziehen Sie dann eine **Notebook**-Aktivität auf die Pipelineentwurfsoberfläche, wie hier gezeigt:

    ![Screenshot: Pipeline mit einer Notebook-Aktivität.](images/notebook-pipeline.png)

4. Ändern Sie auf der Registerkarte **Allgemein** für die Notebookaktivität den Namen in **Spark Transform ausführen**.
5. Legen Sie auf der Registerkarte **Einstellungen** für die Notebookaktivität die folgenden Eigenschaften fest:
    - **Notebook**: Wählen Sie das Notebook **Spark Transform** aus.
    - **Basisparameter**: Erweitern Sie diesen Abschnitt, und definieren Sie einen Parameter mit den folgenden Einstellungen:
        - **Name**: folderName
        - **Typ**: Zeichenfolge
        - **Wert**: Wählen Sie **Dynamischen Inhalt hinzufügen** aus, und legen Sie den Parameterwert auf die Systemvariable *Pipelineausführungs-ID* fest (`@pipeline().RunId`)
    - **Spark-Pool**: Wählen Sie den Pool **spark*xxxxxxx*** aus.
    - **Executor-Größe**: Wählen Sie **Klein (4 virtuelle Kerne, 28 GB Arbeitsspeicher)** aus.

    Ihr Pipelinebereich sollte in etwa so aussehen:

    ![Screenshot: Pipeline mit einer Notebookaktivität mit Einstellungen.](images/notebook-pipeline-settings.png)

### Veröffentlichen und Ausführen der Pipeline

1. Verwenden Sie die Schaltfläche **Alle veröffentlichen**, um die Pipeline (und alle anderen nicht gespeicherten Ressourcen) zu veröffentlichen.
2. Wählen Sie oben im Pipeline-Designerbereich im Menü **Trigger hinzufügen** die Option **Jetzt auslösen** aus. Wählen Sie dann **OK** aus, um zu bestätigen, dass Sie die Pipeline ausführen möchten.

    **Hinweis**: Sie können auch einen Trigger erstellen, um die Pipeline zu einem geplanten Zeitpunkt oder als Reaktion auf ein bestimmtes Ereignis auszuführen.

3. Wenn die Pipeline gestartet wurde, zeigen Sie auf der Seite **Überwachen** die Registerkarte **Pipelineausführungen** an, und überprüfen Sie den Status der Pipeline **Verkaufsdaten transformieren**.
4. Wählen Sie die Pipeline **Verkaufsdaten transformieren** aus, um ihre Details anzuzeigen, und notieren Sie sich die Pipelineausführungs-ID im Bereich **Aktivitätsausführungen**.

    Es kann fünf Minuten oder länger dauern, bis die Pipeline abgeschlossen ist. Sie können die Schaltfläche **&#8635; Aktualisieren** in der Symbolleiste verwenden, um den Status zu überprüfen.

5. Wenn die Pipelineausführung erfolgreich war, navigieren Sie auf der Seite **Daten** zum Speichercontainer **Dateien**, und vergewissern Sie sich, dass ein neuer Ordner mit der Pipelineausführungs-ID im Namen erstellt wurde und dass er Parquet-Dateien für die transformierten Verkaufsdaten enthält.
   
## Löschen von Azure-Ressourcen

Wenn Sie sich mit Azure Synapse Analytics vertraut gemacht haben, sollten Sie die erstellten Ressourcen löschen, um unnötige Azure-Kosten zu vermeiden.

1. Schließen Sie die Registerkarte mit Synapse Studio, und kehren Sie zum Azure-Portal zurück.
2. Wählen Sie auf der **Startseite** des Azure-Portals die Option **Ressourcengruppen** aus.
3. Wählen Sie die Ressourcengruppe **dp203-*xxxxxxx*** für Ihren Synapse Analytics-Arbeitsbereich aus (nicht die verwaltete Ressourcengruppe), und vergewissern Sie sich, dass sie den Synapse-Arbeitsbereich, das Speicherkonto und den Spark-Pool für Ihren Arbeitsbereich enthält.
4. Wählen Sie oben auf der Seite **Übersicht** für Ihre Ressourcengruppe die Option **Ressourcengruppe löschen** aus.
5. Geben Sie den Namen der Ressourcengruppe **dp203-*xxxxxxx*** ein, um zu bestätigen, dass Sie sie löschen möchten, und wählen Sie **Löschen** aus.

    Nach einigen Minuten werden die Ressourcengruppe in Ihrem Azure Synapse-Arbeitsbereich und die damit verknüpfte Ressourcengruppe im verwalteten Arbeitsbereich gelöscht.
