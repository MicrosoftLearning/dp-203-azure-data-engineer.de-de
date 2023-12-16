---
lab:
  title: Verwenden von Delta Lake in Azure Databricks
  ilt-use: Optional demo
---

# Verwenden von Delta Lake in Azure Databricks

Delta Lake ist ein Open-Source-Projekt, um eine Transaktionsdatenspeicherebene für Spark auf einem Data Lake zu erstellen. Delta Lake bietet Unterstützung für relationale Semantik für Batch- und Streamingdatenvorgänge und ermöglicht die Erstellung einer *Lakehouse* -Architektur, in der Apache Spark zum Verarbeiten und Abfragen von Daten in Tabellen verwendet werden kann, die auf zugrunde liegenden Dateien in einem Data Lake basieren.

Diese Übung dauert ca. **40** Minuten.

## Vorbereitung

Sie benötigen ein [Azure-Abonnement](https://azure.microsoft.com/free), in dem Sie Administratorzugriff besitzen.

## Bereitstellen eines Azure Databricks-Arbeitsbereichs

In dieser Übung verwenden Sie ein Skript, um einen neuen Azure Databricks-Arbeitsbereich bereitzustellen.

> **Tipp**: Wenn Sie bereits über einen *Standard* - oder *Testarbeitsbereich* für Azure Databricks verfügen, können Sie dieses Verfahren überspringen.

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
    cd dp-203/Allfiles/labs/25
    ./setup.ps1
    ```

6. Wenn Sie dazu aufgefordert werden, wählen Sie aus, welches Abonnement Sie verwenden möchten (dies geschieht nur, wenn Sie Zugriff auf mehrere Azure-Abonnements haben).

7. Warten Sie, bis das Skript abgeschlossen ist. Dies dauert in der Regel etwa 5 Minuten, aber in einigen Fällen kann es länger dauern. Während Sie warten, lesen Sie den Artikel [Einführung in Delta Technologies](https://learn.microsoft.com/azure/databricks/introduction/delta-comparison) in der Dokumentation zu Azure Databricks.

## Erstellen eines Clusters

Azure Databricks ist eine verteilte Verarbeitungsplattform, die Apache Spark-*Cluster* verwendet, um Daten parallel auf mehreren Knoten zu verarbeiten. Jeder Cluster besteht aus einem Treiberknoten, um die Arbeit zu koordinieren, und Arbeitsknoten zum Ausführen von Verarbeitungsaufgaben.

> **Tipp**: Wenn Sie bereits über einen Cluster mit einer 13.3 LTS-Laufzeitversion in Ihrem Azure Databricks-Arbeitsbereich verfügen, können Sie damit diese Übung abschließen und dieses Verfahren überspringen.

1. Navigieren Sie im Azure-Portal zur Ressourcenruppe **dp203-*xxxxxxx***, die vom Skript erstellt wurde (oder die Ressourcengruppe, die Ihren vorhandenen Azure Databricks-Arbeitsbereich enthält).
1. Wählen Sie Ihre Azure Databricks-Dienstressource (mit dem Namen **Databricks*xxxxxxx*** aus, wenn Sie das Setupskript zum Erstellen verwendet haben).
1. Verwenden Sie auf der Seite **Übersicht** für Ihren Arbeitsbereich die Schaltfläche **Arbeitsbereich starten**, um Ihren Azure Databricks-Arbeitsbereich auf einer neuen Browserregisterkarte zu öffnen. Melden Sie sich an, wenn Sie dazu aufgefordert werden.

    > **Tipp**: Während Sie das Databricks-Arbeitsbereichsportal verwenden, werden möglicherweise verschiedene Tipps und Benachrichtigungen angezeigt. Schließen Sie diese, und folgen Sie den Anweisungen, um die Aufgaben in dieser Übung auszuführen.

1. Zeigen Sie das Azure Databricks-Arbeitsbereichsportal an, und beachten Sie, dass die Randleiste auf der linken Seite Symbole für die verschiedenen Aufgaben enthält, die Sie ausführen können.

1. Wählen Sie die Aufgabe **(+) Neu** und dann **Cluster** aus.
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

## Erkunden des Delta Lakes mithilfe eines Notebooks

In dieser Übung verwenden Sie einen Code in einem Notebook, um den Delta Lake in Azure Databricks zu erkunden.

1. Wählen Sie im Azure Databricks-Arbeitsbereichsportal für Ihren Arbeitsbereich in der Randleiste auf der linken Seite **Arbeitsbereich** aus. Wählen Sie dann den Ordner **⌂ Start** aus.
1. Wählen Sie oben auf der Seite im Menü **⋮** neben Ihrem Benutzernamen **Importieren** aus. Wählen Sie dann im Dialogfeld **Importieren** den Eintrag ** URL** aus, und importieren Sie das Notebook von `https://github.com/MicrosoftLearning/dp-203-azure-data-engineer/raw/master/Allfiles/labs/25/Delta-Lake.ipynb`
1. Verbinden Sie das Notebook mit Ihrem Cluster, und folgen Sie den darin enthaltenen Anweisungen. Führen Sie die darin enthaltenen Zellen aus, um die Delta-Lake-Funktionalität zu erkunden.

## Löschen von Azure Databricks-Ressourcen

Nachdem Sie die Erkundung von Delta Lake in Azure Databricks abgeschlossen haben, müssen Sie die von Ihnen erstellten Ressourcen löschen, um unnötige Azure-Kosten zu vermeiden und Kapazität in Ihrem Abonnement freizugeben.

1. Schließen Sie die Browserregisterkarte für den Databricks-Arbeitsbereich, und kehren Sie zum Azure-Portal zurück.
2. Wählen Sie auf der **Startseite** des Azure-Portals die Option **Ressource erstellen** aus.
3. Wählen Sie die Ressourcengruppe **dp203-*xxxxx*** (nicht die verwaltete Ressourcengruppe) aus, und stellen Sie sicher, dass sie Ihren Azure Databricks-Arbeitsbereich enthält.
4. Wählen Sie oben auf der Seite **Übersicht** für Ihre Ressourcengruppe die Option **Ressourcengruppe löschen** aus.
5. Geben Sie den Ressourcengruppennamen **dp203-*xxxxxxx*** ein, um zu bestätigen, dass Sie ihn löschen möchten, und wählen Sie dann **Löschen** aus.

    Nach ein paar Minuten werden Ihre Ressourcengruppe und die damit verbundenen verwalteten Ressourcengruppen des Arbeitsbereichs gelöscht.
