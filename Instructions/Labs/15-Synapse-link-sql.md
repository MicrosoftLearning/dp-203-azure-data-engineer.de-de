---
lab:
  title: Verwenden von Azure Synapse Link für SQL
  ilt-use: Suggested demo
---

# Verwenden von Azure Synapse Link für SQL

Mit Azure Synapse Link für SQL können Sie eine transaktionale Datenbank in SQL Server oder Azure SQL-Datenbank mit einem dedizierten SQL-Pool in Azure Synapse Analytics synchronisieren. Mit dieser Synchronisierung können Sie latenzarme analytische Workloads in Synapse Analytics ausführen, ohne dass in der Quellbetriebsdatenbank ein Abfrageaufwand entsteht.

Diese Übung dauert ca. **35** Minuten.

## Vor der Installation

Sie benötigen ein [Azure-Abonnement](https://azure.microsoft.com/free), in dem Sie Administratorzugriff besitzen.

## Bereitstellen von Azure-Ressourcen

In dieser Übung synchronisieren Sie Daten aus einer Azure SQL-Datenbankressource mit einem Azure Synapse Analytics-Arbeitsbereich. Sie verwenden zunächst ein Skript, um diese Ressourcen in Ihrem Azure-Abonnement bereitzustellen.

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
    cd dp-203/Allfiles/labs/15
    ./setup.ps1
    ```

6. Wenn Sie dazu aufgefordert werden, wählen Sie aus, welches Abonnement Sie verwenden möchten (dies geschieht nur, wenn Sie Zugriff auf mehrere Azure-Abonnements haben).
7. Wenn Sie dazu aufgefordert werden, geben Sie ein geeignetes Kennwort für Ihre Azure SQL-Datenbank ein.

    > **Hinweis**: Merken Sie sich unbedingt dieses Kennwort!

8. Warten Sie, bis das Skript abgeschlossen ist. Dies dauert in der Regel etwa 15 Minuten, in einigen Fällen kann es jedoch länger dauern. Während Sie warten, lesen Sie den Artikel [Was ist Azure Synapse Link für SQL?](https://docs.microsoft.com/azure/synapse-analytics/synapse-link/sql-synapse-link-overview) in der Dokumentation zu Azure Synapse Analytics.

## Konfigurieren von Azure SQL-Datenbank

Bevor Sie Azure Synapse Link für Ihre Azure SQL-Datenbank einrichten können, müssen Sie sicherstellen, dass die erforderlichen Konfigurationseinstellungen auf Ihrem Azure SQL-Datenbank-Server angewendet wurden.

1. Navigieren Sie im [Azure-Portal](https://portal.azure.com) zur Ressourcengruppe **dp203-*xxxxxxx***, die vom Setupskript erstellt wurde, und wählen Sie Ihren Azure SQL-Server **sqldb*xxxxxx*** aus.

    > **Hinweis:** Achten Sie darauf, dass Sie die Azure SQL-Serverressource **sqldb*xxxxxxxx***) und den dedizierten SQL-Pool von Azure Synapse Analytics (** sql*xxxxxxxx***) nicht verwechseln.

2. Wählen Sie auf der Seite für Ihre Azure SQL-Serverressource im Bereich auf der linken Seite im Abschnitt **Sicherheit** (am unteren Rand) die Option **Identität** aus. Legen Sie dann unter **Systemseitig zugewiesene verwaltete Identität** die Option **Status** auf **Ein** fest. Verwenden Sie dann das Symbol **&#128427; Speichern**, um die Konfigurationsänderung zu speichern.

    ![Screenshot: Seite „Identität“ für den Azure SQL-Server im Azure-Portal](./images/sqldb-identity.png)

3. Wählen Sie im linken Bereich im Abschnitt **Sicherheit** die Option **Netzwerk**. Wählen Sie dann unter **Firewallregeln** die Ausnahme **Azure-Diensten und -Ressourcen den Zugriff auf diesen Server gestatten**.

4. Verwenden Sie die Schaltfläche **#65291; Eine Firewallregel hinzufügen** hinzu, um eine neue Firewallregel mit den folgenden Einstellungen hinzuzufügen:

    | Regelname | Start-IP | End-IP |
    | -- | -- | -- |
    | AllClients | 0.0.0.0 | 255.255.255.255 |

    > **Hinweis**: Diese Regel ermöglicht den Zugriff auf Ihren Server von jedem mit dem Internet verbundenen Computer. Wir aktivieren diese Option, um die Übung zu vereinfachen, aber in einem Produktionsszenario sollten Sie den Zugriff auf Netzwerkadressen beschränken, die Ihre Datenbanken verwenden müssen.

5. Klicken Sie auf die Schaltfläche **Speichern**, um Ihre Konfigurationsänderungen zu speichern.

    ![Screenshot: Seite „Netzwerk“ für den Azure SQL-Server im Azure-Portal](./images/sqldb-network.png)

## Erkunden der transaktionalen Datenbank

Ihr Azure SQL-Server hostet eine Beispieldatenbank mit dem Namen **AdventureWorksLT**. Diese Datenbank stellt eine transaktionale Datenbank dar, die für betriebsbereite Anwendungsdaten verwendet wird.

1. Wählen Sie unten auf der Seite **Übersicht** für Ihren Azure SQL-Server die Datenbank **AdventureWorksLT** aus:
2. Wählen Sie auf der Datenbankseite **AdventureWorksLT** die Registerkarte **Abfrage-Editor** aus, und melden Sie sich mithilfe der SQL-Server-Authentifizierung mit den folgenden Anmeldeinformationen an:
    - **Login** SQLUser
    - **Kennwort**: *Das Kennwort, das Sie beim Ausführen des Setupskripts angegeben haben.*
3. Wenn der Abfrage-Editor geöffnet wird, erweitern Sie den Knoten **Tabelle**, und zeigen Sie die Liste der Tabellen in der Datenbank an. Beachten Sie, dass sie Tabellen in einem **SalesLT**-Schema enthält (zum Beispiel **SalesLT.Customer**).

## Konfigurieren von Azure Synapse Link

Jetzt können Sie Azure Synapse Link für SQL in Ihrem Synapse Analytics-Arbeitsbereich konfigurieren.

### Starten des dedizierten SQL-Pools

1. Schließen Sie im Azure-Portal den Abfrage-Editor für Ihre Azure SQL-Datenbank (ohne Änderungen zu speichern), und kehren Sie zur Seite für Ihre Ressourcengruppe **dp203-*xxxxxxx*** zurück.
2. Öffnen Sie den Synapse-Arbeitsbereich **Synapse*xxxxxxx***, und wählen Sie auf der Seite **Übersicht** in der Karte **Synapse Studio öffnen** die Option **Öffnen** aus, um Synapse Studio in einer neuen Browserregisterkarte zu öffnen. Melden Sie sich an, wenn Sie dazu aufgefordert werden.
3. Verwenden Sie im linken Bereich von Synapse Studio das Symbol **&rsaquo;&rsaquo;**, um das Menü zu erweitern. Dadurch werden die verschiedenen Seiten in Synapse Studio angezeigt.
4. Wählen Sie auf der Seite **Verwalten** auf der Registerkarte **SQL-Pools** die Zeile für den dedizierten SQL-Pool **sql*xxxxxxx*** aus, und klicken Sie auf das zugehörige Symbol **&#9655;**, um ihn zu starten. Bestätigen Sie, dass Sie ihn fortsetzen möchten, wenn Sie dazu aufgefordert werden.
5. Warten Sie, bis der SQL-Pool fortgesetzt wird. Dies kann einige Minuten dauern. Sie können den Status mithilfe der Schaltfläche **&#8635; Aktualisieren** regelmäßig überprüfen. Der Status wird als **Online** angezeigt, wenn der SQL-Pool bereit ist.

### Erstellen des Zielschemas

1. Erweitern Sie in Synapse Studio auf der Seite **Daten** auf der Registerkarte **Arbeitsbereich** **SQL-Datenbanken**, und wählen Sie Ihre Datenbank **sql*xxxxxxx*** aus.
2. Wählen Sie im Menü **…** für die Datenbank **sql*xxxxxxx*** die Option **Neues SQL-Skript** > **Leeres Skript** aus.
3. Geben Sie im Bereich **SQL-Skript 1** den folgenden SQL-Code ein, und verwenden Sie die Schaltfläche **&#9655; Ausführen**, um ihn auszuführen.

    ```sql
    CREATE SCHEMA SalesLT;
    GO
    ```

4. Warten Sie auf den erfolgreichen Abschluss der Abfrage. Dieser Code erstellt ein Schema mit dem Namen **SalesLT** in der Datenbank für Ihren dedizierten SQL-Pool. Dadurch können Sie Tabellen im Schema mit diesem Namen aus Ihrer Azure SQL-Datenbank synchronisieren.

### Erstellen einer Linkverbindung

1. Wählen Sie in Synapse Studio auf der Seite **Integrieren** im Dropdownmenü **&#65291;** die Option **Linkverbindung** aus. Erstellen Sie anschließend eine neue verknüpfte Verbindung mit den folgenden Einstellungen:
    - **Quelltyp**: Azure SQL-Datenbank
    - **Verknüpfter Quelldienst**: Fügen Sie einen neuen verknüpften Dienst mit den folgenden Einstellungen hinzu (eine neue Registerkarte wird geöffnet):
        - **Name**: SqlAdventureWorksLT
        - **Beschreibung**: Verbindung mit der Datenbank „AdventureWorksLT“
        - **Verbindung über Integration Runtime herstellen**: AutoResolveIntegrationRuntime
        - **Version:** Alt
        - **Verbindungszeichenfolge**: Ausgewählt
        - **Aus Azure-Abonnement**: Ausgewählt
        - **Azure-Abonnement**: *Wählen Sie Ihr Azure-Abonnement aus.*
        - **Servername**: *Wählen Sie Ihren Azure SQL-Server **sqldbxxxxxxx** aus*.
        - **Datenbankname**: AdventureWorksLT
        - **Authentifizierungstyp**: SQL-Authentifizierung
        - **Benutzername**: SQLUser
        - **Kennwort**: *Das Kennwort, das Sie beim Ausführen des Setupskripts festgelegt haben*

        *Verwenden Sie die Option **Verbindung testen**, um sicherzustellen, dass Ihre Verbindungseinstellungen korrekt sind, bevor Sie fortfahren! Klicken Sie anschließend auf **Erstellen**.*

    - **Quelltabellen**: Wählen Sie die folgenden Tabellen aus:
        - **SalesLT.Customer**
        - **SalesLT.Product**
        - **SalesLT.SalesOrderDetail**
        - **SalesLT.SalesOrderHeader**

        *Fahren Sie mit der Konfiguration der folgenden Einstellungen fort:*

    > **Hinweis**: Bei einigen Zieltabellen wird ein Fehler angezeigt, der auf die Verwendung benutzerdefinierter Datentypen oder darauf zurückzuführen ist, dass Daten in der Quelltabelle nicht mit dem Standardstrukturtyp des *gruppierten Columnstore-Index* kompatibel sind.

    - **Zielpool**: *Wählen Sie Ihren dedizierten SQL-Pool **sqlxxxxxxx** aus.*

        *Fahren Sie mit der Konfiguration der folgenden Einstellungen fort:*

    - **Linkverbindungsname**: sql-adventureworkslt-conn
    - **Kernanzahl**: 4 (+ 4 Treiberkerne)

2. Zeigen Sie auf der erstellten Seite **sql-adventureworkslt-conn** die Tabellenzuordnungen an, die erstellt wurden. Sie können mithilfe der Schaltfläche **Eigenschaften** (die dem Element **&#128463;<sub>*</sub>** ähnelt) den Bereich **Eigenschaften** ausblenden, um die Sichtbarkeit der Inhalte zu verbessern. 

3. Ändern Sie die Strukturtypen in den Tabellenzuordnungen wie folgt:

    | Quelltabelle | Zieltabelle | Verteilungstyp | Verteilungsspalte | Strukturtyp |
    |--|--|--|--|--|
    | SalesLT.Customer **&#8594;** | \[SalesLT]. \[Customer] | Roundrobin | - | Gruppierter Columnstore-Index |
    | SalesLT.Product **&#8594;** | \[SalesLT]. \[Product] | Roundrobin | - | Heap |
    | SalesLT.SalesOrderDetail **&#8594;** | \[SalesLT]. \[SalesOrderDetail] | Roundrobin | - | Gruppierter Columnstore-Index |
    | SalesLT.SalesOrderHeader **&#8594;** | \[SalesLT]. \[SalesOrderHeader] | Roundrobin | - | Heap |

4. Verwenden Sie oben auf der erstellten Seite **sql-adventureworkslt-conn** die Schaltfläche **&#9655; Starten**, um die Synchronisierung zu starten. Wenn Sie dazu aufgefordert werden, wählen Sie **OK** aus, um die Linkverbindung zu veröffentlichen und zu starten.
5. Zeigen Sie nach dem Starten der Verbindung auf der Seite **Überwachen** die Registerkarte **Linkverbindungen** an, und wählen Sie die Verbindung **sql-adventureworkslt-conn** aus. Sie können den Status mithilfe der Schaltfläche **&#8635; Aktualisieren** regelmäßig aktualisieren. Es kann mehrere Minuten dauern, bis der erste Kopiervorgang der Momentaufnahme abgeschlossen und die Replikation gestartet wird. Danach werden alle Änderungen in den Tabellen der Quelldatenbank automatisch in den synchronisierten Tabellen wiedergegeben.

### Anzeigen der replizierten Daten

1. Nachdem sich der Status der Tabellen in **Ausgeführt** geändert hat, wählen Sie die Seite **Daten** aus, und verwenden Sie das Symbol **&#8635;** oben rechts, um die Ansicht zu aktualisieren.
2. Erweitern Sie auf der Registerkarte **Arbeitsbereich** **SQL-Datenbanken**, Ihre Datenbank **sql*xxxxxxx*** und den zugehörigen Ordner **Tabellen**, um die replizierten Tabellen anzuzeigen.
3. Wählen Sie im Menü **…** für die Datenbank **sql*xxxxxxx*** die Option **Neues SQL-Skript** > **Leeres Skript** aus. Geben Sie anschließend auf der Seite für das neue Skript den folgenden SQL-Code ein:

    ```sql
    SELECT  oh.SalesOrderID, oh.OrderDate,
            p.ProductNumber, p.Color, p.Size,
            c.EmailAddress AS CustomerEmail,
            od.OrderQty, od.UnitPrice
    FROM SalesLT.SalesOrderHeader AS oh
    JOIN SalesLT.SalesOrderDetail AS od 
        ON oh.SalesOrderID = od.SalesOrderID
    JOIN  SalesLT.Product AS p 
        ON od.ProductID = p.ProductID
    JOIN SalesLT.Customer as c
        ON oh.CustomerID = c.CustomerID
    ORDER BY oh.SalesOrderID;
    ```

4. Verwenden Sie Schaltfläche **&#9655; Ausführen**, um das Skript auszuführen und die Ergebnisse anzuzeigen. Die Abfrage wird für die replizierten Tabellen im dedizierten SQL-Pool und nicht für die Quelldatenbank ausgeführt. Dadurch können Sie analytische Abfragen ohne Auswirkungen auf Geschäftsanwendungen ausführen.
5. Wenn Sie fertig sind, halten Sie auf der Seite **Verwalten** den dedizierten SQL-Pool **sql*xxxxxxx*** an.

## Löschen von Azure-Ressourcen

Wenn Sie sich mit Azure Synapse Analytics vertraut gemacht haben, sollten Sie die erstellten Ressourcen löschen, um unnötige Azure-Kosten zu vermeiden.

1. Schließen Sie die Registerkarte mit Synapse Studio, und kehren Sie zum Azure-Portal zurück.
2. Wählen Sie auf der **Startseite** des Azure-Portals die Option **Ressourcengruppen** aus.
3. Wählen Sie die Ressourcengruppe **dp203-*xxxxxxx*** aus, die vom Setupskript am Anfang dieser Übung erstellt wurde.
4. Wählen Sie oben auf der Seite **Übersicht** für Ihre Ressourcengruppe die Option **Ressourcengruppe löschen** aus.
5. Geben Sie den Namen der Ressourcengruppe **dp203-*xxxxxxx*** ein, um zu bestätigen, dass Sie sie löschen möchten, und wählen Sie **Löschen** aus.

    Nach einigen Minuten werden Ihre Ressourcengruppe und alle zugehörigen Ressourcen gelöscht.
