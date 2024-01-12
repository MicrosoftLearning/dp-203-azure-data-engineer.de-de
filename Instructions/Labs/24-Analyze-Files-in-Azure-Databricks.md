---
lab:
  title: Verwenden von Spark in Azure Databricks
  ilt-use: Lab
---

# Verwenden von Spark in Azure Databricks

Azure Databricks ist eine Microsoft Azure-basierte Version der beliebten Open-Source-Databricks-Plattform. Azure Databricks basiert auf Apache Spark und bietet eine hoch skalierbare Lösung für Datentechnik- und Analyseaufgaben, die das Arbeiten mit Daten in Dateien beinhalten. Einer der Vorteile von Spark ist die Unterstützung für eine Vielzahl von Programmiersprachen (einschließlich Java, Scala, Python und SQL). Dies macht Spark zu einer sehr flexiblen Lösung für Datenverarbeitungsworkloads (einschließlich Datenbereinigung und -manipulation, statistischer Analysen, maschinellem Lernen, Datenanalysen und Visualisierungen).

Diese Übung dauert ca. **45** Minuten.

## Vorbereitung

Sie benötigen ein [Azure-Abonnement](https://azure.microsoft.com/free), in dem Sie Administratorzugriff besitzen.

## Bereitstellen eines Azure Databricks-Arbeitsbereichs

In dieser Übung verwenden Sie ein Skript, um einen neuen Azure Databricks-Arbeitsbereich bereitzustellen.

1. Melden Sie sich in einem Webbrowser beim [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com` an.
2. Verwenden Sie rechts neben der Suchleiste oben auf der Seite die Schaltfläche **[\>_]**, um eine neue Cloud Shell-Instanz im Azure-Portal zu erstellen. Wählen Sie eine ***PowerShell***-Umgebung aus, und erstellen Sie Speicher, falls Sie dazu aufgefordert werden. Die Cloud Shell bietet eine Befehlszeilenschnittstelle in einem Bereich am unteren Rand des Azure-Portals, wie hier gezeigt:

    ![Azure-Portal mit einem Cloud Shell-Bereich](./images/cloud-shell.png)

    > **Hinweis**: Wenn Sie zuvor eine Cloud Shell erstellt haben, die eine *Bash*-Umgebung verwendet, ändern Sie diese mithilfe des Dropdownmenüs oben links im Cloud Shell-Bereich zu ***PowerShell***.

3. Beachten Sie, dass Sie die Größe der Cloud Shell durch Ziehen der Trennzeichenleiste oben im Bereich ändern können oder den Bereich mithilfe der Symbole **&#8212;**, **&#9723;** und **X** oben rechts minimieren, maximieren und schließen können. Weitere Informationen zur Verwendung von Azure Cloud Shell finden Sie in der [Azure Cloud Shell-Dokumentation](https://docs.microsoft.com/azure/cloud-shell/overview).

4. Geben Sie im PowerShell-Bereich die folgenden Befehle ein, um dieses Repository zu klonen:

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. Nachdem das Repository geklont wurde, geben Sie die folgenden Befehle ein, um in den Ordner für dieses Lab zu wechseln. Führen Sie das darin enthaltene Skript **setup.ps1** aus:

    ```
    cd dp-203/Allfiles/labs/24
    ./setup.ps1
    ```

6. Wenn Sie dazu aufgefordert werden, wählen Sie aus, welches Abonnement Sie verwenden möchten (dies geschieht nur, wenn Sie Zugriff auf mehrere Azure-Abonnements haben).

7. Warten Sie, bis das Skript abgeschlossen ist. Dies dauert in der Regel etwa 5 Minuten, in einigen Fällen kann es jedoch länger dauern. Während Sie warten, lesen Sie den Artikel [Explorative Datenanalyse in Azure Databricks](https://learn.microsoft.com/azure/databricks/exploratory-data-analysis/) in der Dokumentation zu Azure Databricks.

## Erstellen eines Clusters

Azure Databricks ist eine verteilte Verarbeitungsplattform, die Apache Spark-*Cluster* verwendet, um Daten parallel auf mehreren Knoten zu verarbeiten. Jeder Cluster besteht aus einem Treiberknoten zur Koordination der Arbeit und Workerknoten zum Ausführen von Verarbeitungsaufgaben.

> **Hinweis**: In dieser Übung erstellen Sie einen *Einzelknotencluster*, um die in der Labumgebung verwendeten Computeressourcen (wo Ressourcen möglicherweise eingeschränkt sind) zu minimieren. In einer Produktionsumgebung erstellen Sie in der Regel einen Cluster mit mehreren Workerknoten.

1. Navigieren Sie im Azure-Portal zur Ressourcengruppe **dp203-*xxxxxxx***, die vom Skript erstellt wurde, das Sie ausgeführt haben.
2. Wählen Sie die Azure Databricks Service-Ressource **databricks*xxxxxxx*** aus.
3. Verwenden Sie auf der Seite **Übersicht** für **databricks*xxxxxxx*** die Schaltfläche **Arbeitsbereich starten**, um Ihren Azure Databricks-Arbeitsbereich in einer neuen Browserregisterkarte zu öffnen. Melden Sie sich an, wenn Sie dazu aufgefordert werden.
4. Wenn die Nachricht **Was ist Ihr aktuelles Datenprojekt?** angezeigt wird, wählen Sie **Fertigstellen** aus, um es zu schließen. Zeigen Sie dann das Azure Databricks-Arbeitsbereichsportal an, und beachten Sie, dass die Seitenleiste auf der linken Seite Symbole für die verschiedenen Aufgaben enthält, die Sie ausführen können.

    >**Tipp**: Während Sie das Databricks-Arbeitsbereichsportal verwenden, werden möglicherweise verschiedene Tipps und Benachrichtigungen angezeigt. Schließen Sie diese, und folgen Sie den Anweisungen, um die Aufgaben in dieser Übung auszuführen.

1. Wählen Sie die Aufgabe **(+) Neu** und dann **Cluster** aus.
1. Erstellen Sie auf der Seite **Neuer Cluster** einen neuen Cluster mit den folgenden Einstellungen:
    - **Clustername**: Cluster des *Benutzernamens* (der Standardclustername)
    - **Clustermodus**: Einzelknoten
    - **Zugriffsmodus**: Einzelner Benutzer (*Ihr Benutzerkonto ist ausgewählt*)
    - **Databricks-Runtimeversion**: 12.2 LTS (Scala 2.12, Spark 3.2.2)
    - **Photonbeschleunigung verwenden**: Ausgewählt
    - **Knotentyp**: Standard_DS3_v2
    - **Nach** *30* **Minuten Inaktivität beenden**

7. Warten Sie, bis der Cluster erstellt wurde. Es kann ein oder zwei Minuten dauern.

> **Hinweis**: Wenn Ihr Cluster nicht gestartet werden kann, verfügt Ihr Abonnement möglicherweise über ein unzureichendes Kontingent in der Region, in der Ihr Azure Databricks-Arbeitsbereich bereitgestellt wird. Details finden Sie unter [Der Grenzwert für CPU-Kerne verhindert die Clustererstellung](https://docs.microsoft.com/azure/databricks/kb/clusters/azure-core-limit). In diesem Fall können Sie versuchen, Ihren Arbeitsbereich zu löschen und in einer anderen Region einen neuen zu erstellen. Sie können einen Bereich als Parameter für das Setupskript wie folgt angeben: `./setup.ps1 eastus`

## Erkunden von Daten mithilfe eines Notebooks

Wie in vielen Spark-Umgebungen unterstützt Databricks die Verwendung von Notebooks zum Kombinieren von Notizen und interaktiven Codezellen, mit denen Sie Daten untersuchen können.

1. Wählen Sie in der Seitenleiste auf der linken Seite **Arbeitsbereiche** aus. Wählen Sie dann den Ordner **&#8962; Basis** aus.
1. Wählen Sie oben auf der Seite im Menü **&#8942;** neben Ihrem Benutzernamen die Option **Importieren** aus. Wählen Sie dann im Dialogfeld **Importieren** die Option **URL** aus, und importieren Sie das Notebook aus `https://github.com/MicrosoftLearning/dp-203-azure-data-engineer/raw/master/Allfiles/labs/24/Databricks-Spark.ipynb`.
1. Verbinden Sie das Notebook mit Ihrem Cluster aus, und folgen Sie den darin enthaltenen Anweisungen. Führen Sie die darin enthaltenen Zellen aus, um Daten in Dateien zu erkunden.

## Löschen von Databricks-Ressourcen

Nachdem Sie sich mit Azure Databricks vertraut gemacht haben, müssen Sie die von Ihnen erstellten Ressourcen löschen, um unnötige Azure-Kosten zu vermeiden und Kapazität in Ihrem Abonnement freizugeben.

1. Schließen Sie die Registerkarte mit dem Azure Databricks-Arbeitsbereich, und kehren Sie zum Azure-Portal zurück.
2. Wählen Sie auf der **Startseite** des Azure-Portals die Option **Ressourcengruppen** aus.
3. Wählen Sie die Ressourcengruppe **dp203-*xxxxxxx*** (nicht die verwaltete Ressourcengruppe), und vergewissern Sie sich, dass sie Ihren Azure Databricks-Arbeitsbereich enthält.
4. Wählen Sie oben auf der Seite **Übersicht** für Ihre Ressourcengruppe die Option **Ressourcengruppe löschen** aus.
5. Geben Sie den Namen der Ressourcengruppe **dp203-*xxxxxxx*** ein, um zu bestätigen, dass Sie sie löschen möchten, und wählen Sie **Löschen** aus.

    Nach einigen Minuten werden Ihre Ressourcengruppe und die damit verknüpften Ressourcengruppen im verwalteten Arbeitsbereich gelöscht.
