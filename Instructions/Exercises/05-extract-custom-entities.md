---
lab:
  title: Estrarre entità personalizzate
  module: Module 3 - Getting Started with Natural Language Processing
---

# Estrarre entità personalizzate

Oltre ad altre funzionalità di elaborazione del linguaggio naturale, il servizio Lingua di Azure AI consente di definire entità personalizzate ed estrarre istanze di tali entità dal testo.

Per testare l'estrazione di entità personalizzata, verrà creato un modello e ne verrà eseguito il training tramite Azure AI Language Studio, quindi verrà usata un'applicazione da riga di comando per testarlo.

## Effettuare il provisioning di una risorsa di *Lingua di Azure AI*

Se non ne è già disponibile una nella sottoscrizione, è necessario effettuare il provisioning di una risorsa del servizio **Lingua di Azure AI**. Inoltre, per usare la classificazione personalizzata del testo, è necessario abilitare la funzionalità **Estrazione e classificazione personalizzata del testo**.

1. In un browser, aprire il portale di Azure all'indirizzo `https://portal.azure.com` e accedere con il proprio account Microsoft.
1. Selezionare il pulsante **Crea una risorsa**, cercare *Lingua*, quindi creare una risorsa del servizio **Lingua**. Una volta nella pagina in *Seleziona funzionalità aggiuntive*, selezionare la funzionalità personalizzata contenente l'**estrazione di riconoscimento entità denominata personalizzata**. Creare la risorsa con le impostazioni seguenti:
    - **Sottoscrizione**: *la sottoscrizione di Azure usata*
    - **Gruppo di risorse**: *selezionare o creare un gruppo di risorse*
    - **Area**: *scegliere una qualsiasi area disponibile*
    - **Nome**: *immettere un nome univoco*
    - **Piano tariffario**: selezionare **F0** (*gratuito*) o **S** (*standard*) se F non è disponibile.
    - **Account di archiviazione**: nuovo account di archiviazione:
      - **Nome account di archiviazione**: *immettere un nome univoco*.
      - **Tipo di account di archiviazione: ** Archiviazione con ridondanza locale Standard
    - **Avviso per intelligenza artificiale responsabile**: opzione selezionata

1. Selezionare **Rivedi e crea**, quindi **Crea** per effettuare il provisioning della risorsa.
1. Attendere il completamento della distribuzione e quindi passare alla risorsa distribuita.
1. Visualizzare la pagina **Chiavi ed endpoint**. Le informazioni contenute in questa pagina saranno necessarie più avanti nell'esercizio.

## Caricare annunci di esempio

Dopo aver creato il servizio Lingua di Azure AI e l'account di archiviazione, sarà necessario caricare annunci di esempio per eseguire il training del modello in un secondo momento.

1. In una nuova scheda del browser, scaricare gli annunci classificati di esempio da `https://aka.ms/entity-extraction-ads` ed estrarre i file nella cartella desiderata.

2. Nel portale di Azure passare all'account di archiviazione creato e selezionarlo.

3. Nell'account di archiviazione, selezionare **Configurazione**, che si trova sotto **Impostazioni** e abilitare l'opzione **Consenti accesso anonimo BLOB**, quindi selezionare **Salva**.

4. Dal menu a sinistra selezionare **Contenitori** che si trova sotto **Archiviazione dati**. Nella schermata visualizzata selezionare **+ Contenitore**. Assegnare al contenitore il nome `classifieds`, quindi impostare **Livello di accesso anonimo** su **Contenitore (accesso in lettura anonimo per contenitori e BLOB)**.

    > **NOTA**: Quando si configura un account di archiviazione per una soluzione reale, assicurarsi di assegnare il livello di accesso appropriato. Per altre informazioni su ogni livello di accesso, vedere la [documentazione di Archiviazione di Azure](https://learn.microsoft.com/azure/storage/blobs/anonymous-read-access-configure).

5. Dopo aver creato il contenitore, selezionarlo e fare clic sul pulsante **Carica** per caricare gli annunci di esempio scaricati.

## Creare un progetto di Riconoscimento entità denominata personalizzata

A questo punto è possibile creare un progetto di riconoscimento entità denominata personalizzata. Questo progetto offre un luogo di lavoro per compilare il modello, eseguirne il training e quindi distribuirlo.

> **NOTA**: È anche possibile creare, compilare, eseguire il training e distribuire il modello tramite l'API REST.

1. In una nuova scheda del browser aprire il portale Studio di Lingua di Azure AI all'indirizzo `https://language.cognitive.azure.com/` e accedere usando l'account Microsoft associato alla sottoscrizione di Azure.
1. Se viene richiesto di scegliere una risorsa del servizio Lingua, selezionare le impostazioni seguenti:

    - **Directory di Azure**: directory di Azure contenente la sottoscrizione.
    - **Sottoscrizione di Azure**: sottoscrizione di Azure in uso.
    - **Tipo di risorsa**: lingua.
    - **Risorsa linguistica**: risorsa di Lingua di Azure AI creata in precedenza.

    Se <u>non</u> viene chiesto di scegliere una risorsa Lingua, è possibile che nella sottoscrizione siano presenti più risorse di questo tipo e in tal caso:

    1. Nella barra nella parte superiore della pagina, selezionare il pulsante **Impostazioni (&#9881;)**.
    2. Nella pagina **Impostazioni** visualizzare la scheda **Risorse**.
    3. Selezionare la risorsa del servizio Lingua appena creata e fare clic su **Cambiare risorsa**.
    4. Nella parte superiore della pagina fare clic su **Language Studio** per tornare alla pagina iniziale di Language Studio.

1. Nella parte superiore del portale, nel menu **Crea nuovo** selezionare **Riconoscimento entità denominata personalizzata**.

1. Creare un nuovo progetto con le seguenti impostazioni:
    - **Archiviazione Connessione**: *questo valore è probabilmente già compilato. Se non lo è, immettere l'account di archiviazione*
    - **Informazioni di base:**
    - **Nome**: `CustomEntityLab`
        - **Lingua principale testo**: Inglese (US)
        - **Il set di dati include documenti che non sono nella stessa lingua?** : *No*
        - **Descrizione**: `Custom entities in classified ads`
    - **Contenitore**:
        - **Contenitore dell'archivio BLOB**: classificazioni
        - **I file sono etichettati con classi?**: no, è necessario assegnare etichette ai file nell'ambito di questo progetto

## Assegnare etichette ai dati

Dopo aver creato il progetto, è necessario assegnare etichette ai dati per eseguire il training del modello al fine di identificare le entità.

1. Se la pagina **Etichettatura dei dati** non è già aperta, nel riquadro a sinistra selezionare **Etichettatura dei dati**. Viene visualizzato un elenco dei file caricati nell'account di archiviazione.
1. Sul lato destro, nel riquadro **Attività**, selezionare**Aggiungi entità** e aggiungere una nuova entità denominata `ItemForSale`.
1.  Ripetere il passaggio precedente o creare le entità seguenti:
    - `Price`
    - `Location`
1. Dopo aver creato le tre entità, selezionare **Ad 1.txt** in modo da poterla leggere.
1. In *Ad 1.txt*: 
    1. Evidenziare il testo *face cord of firewood* e selezionare l'entità **ItemForSale**.
    1. Evidenziare il testo *Denver, CO* e selezionare l'entità **Location**.
    1. Evidenziare il testo *$90* e selezionare l'entità **Price**.
1. Nel riquadro **Attività**, si noti che questo documento verrà aggiunto al set di dati per il training del modello.
1. Usare il pulsante **Documento successivo** per passare al documento successivo e continuare ad assegnare testo alle entità appropriate per l'intero set di documenti, aggiungendoli tutti al set di dati di training.
1. Dopo aver etichettato l'ultimo documento (*Ad 9.txt*), salvare le etichette.

## Eseguire il training del modello

Dopo aver assegnato etichette ai dati, è necessario eseguire il training del modello.

1. Selezionare **Processi di training** nel riquadro a sinistra.
2. Selezionare **Avvia un processo di training**
3. Eseguire il training di un nuovo modello denominato `ExtractAds`
4. Scegliere **Separare automaticamente il set di test dai dati di training**

    > **SUGGERIMENTO**: Nei progetti di estrazione personalizzati usare la divisione dei test più adatta ai dati. Per dati più coerenti e set di dati più grandi, il servizio Lingua di Azure AI divide automaticamente il set di test in base alla percentuale. Con set di dati più piccoli, è importante eseguire il training con la giusta varietà di documenti di input possibili.

5. Fare clic su **Training**

    > **IMPORTANTE**: Il training del modello può richiedere talvolta diversi minuti. Al termine dell'operazione, verrà visualizzata una notifica.

## Valutare il modello

Nelle applicazioni reali è importante valutare e migliorare il modello per verificare che venga eseguito come previsto. Due pagine a sinistra mostrano i dettagli del modello sottoposto a training e tutti i test non riusciti.

Selezionare **Prestazioni del modello** nel menu a sinistra, quindi selezionare il modello `ExtractAds`. È possibile visualizzare il punteggio del modello, le metriche delle prestazioni e il momento in cui è stato eseguito il training. Sarà possibile verificare se i documenti di test non sono riusciti e questi errori consentono di comprendere gli aspetti da migliorare.

## Distribuire il modello

Quando il training del modello restituisce risultati soddisfacenti, è possibile distribuire il modello in modo da iniziare a estrarre entità tramite l'API.

1. Nel riquadro sinistro selezionare **Distribuzione di un modello**.
2. Selezionare **Aggiungi distribuzione**, quindi immettere il nome `AdEntities` e selezionare il modello **ExtractAds**.
3. Fare clic su **Distribuisci** per distribuire il modello.

## Preparare lo sviluppo di un'app in Visual Studio Code

Per testare le funzionalità di estrazione di entità personalizzate del servizio Lingua di Azure AI, si svilupperà una semplice applicazione console in Visual Studio Code.

> **Suggerimento**: se il repository **mslearn-ai-language** è già stato clonato, aprirlo in Visual Studio Code. In caso contrario, eseguire questi passaggi per clonarlo nell'ambiente di sviluppo.

1. Avviare Visual Studio Code.
2. Aprire il riquadro comandi (MAIUSC+CTRL+P) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/mslearn-ai-language` in una cartella locale. Non è importante usare una cartella specifica.
3. Dopo la clonazione del repository, aprire la cartella in Visual Studio Code.

    > **Nota**: se in Visual Studio Code viene visualizzato un messaggio popup che chiede se si considera attendibile il codice da aprire, fare clic sull'opzione **Sì, mi fido degli autori** nel popup.

4. Attendere il completamento dell'installazione di file aggiuntivi per supportare i progetti in codice C# nel repository.

    > **Nota**: se viene richiesto di aggiungere gli asset necessari per la compilazione e il debug, selezionare **Non adesso**.

## Configurare l'applicazione

Sono state fornite applicazioni per C# e per Python. Entrambe le app hanno la stessa funzionalità. Prima di tutto, verranno completate alcune parti chiave dell'applicazione per consentirle di usare la risorsa Lingua di Azure AI.

1. Nel riquadro **Esplora risorse** di Visual Studio Code, passare alla cartella **Labfiles/05-custom-entity-recognition** ed espandere la cartella **CSharp** o **Python** in base alle preferenze di lingua e alla cartella **custom-entities** che contiene. Ogni cartella contiene i file specifici della lingua per un'app in cui si intende integrare la funzionalità di classificazione del testo di Lingua di Azure AI.
1. Fare clic con il pulsante destro del mouse sulla cartella **custom-entities** contenente i file di codice e aprire un terminale integrato. Installare quindi il pacchetto Text Analytics SDK di Lingua di Azure AI eseguendo il comando appropriato per la lingua scelta:

    **C#:**

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.3.0
    ```

    **Python**:

    ```
    pip install azure-ai-textanalytics==5.3.0
    ```

1. Nel riquadro **Esplora risorse**, aprire il file di configurazione per la lingua preferita nella cartella **custom-entities**

    - **C#**: appsettings.json
    - **Python**: .env
    
1. Aggiornare i valori di configurazione per includere l'**endpoint** e una **chiave** dalla risorsa Lingua di Azure creata (disponibile nella pagina **Chiavi ed endpoint** per la risorsa Lingua di Azure AI nel portale di Azure). Il file deve contenere già i nomi di progetto e distribuzione per il modello di estrazione di entità personalizzate.
1. Salvare il file di configurazione.

## Aggiungere codice per estrarre le entità

A questo punto è possibile usare il servizio Lingua di Azure AI per estrarre entità personalizzate dal testo.

1. Espandere la cartella **ads** nella cartella **custom-entities** per visualizzare gli annunci classificati che l'applicazione analizzerà.
1. Nella cartella **custom-entities**, aprire il file di codice per l'applicazione client:

    - **C#**: Program.cs
    - **Python**: custom-entities.py

1. Individuare il commento **Import namespaces**. In questo commento aggiungere quindi il codice seguente specifico per la lingua per importare gli spazi dei nomi necessari per usare Text Analytics SDK:

    **C#**: Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.TextAnalytics;
    ```

    **Python**: custom-entities.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. Nella funzione **Main**, si noti che il codice per caricare l'endpoint e la chiave del servizio Lingua di Azure AI e i nomi di progetto e distribuzione indicati nel file di configurazione sono già stati forniti. Individuare quindi il commento **Create client using endpoint and key** e aggiungere il codice seguente per creare un client per l'API Analisi del testo:

    **C#**: Programs.cs

    ```csharp
    // Create client using endpoint and key
    AzureKeyCredential credentials = new(aiSvcKey);
    Uri endpoint = new(aiSvcEndpoint);
    TextAnalyticsClient aiClient = new(endpoint, credentials);
    ```

    **Python**: custom-entities.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

1. Nella funzione **Main**, si noti che il codice esistente legge tutti i file nella cartella **ads** e crea un elenco del loro contenuto. Nel caso del codice C#, viene usato un elenco di oggetti **TextDocumentInput** per includere il nome del file come ID e la lingua. In Python viene usato un semplice elenco del contenuto del testo.
1. Individuare il commento **Extract entities** e aggiungere il codice seguente:

    **C#**: Program.cs

    ```csharp
    // Extract entities
    RecognizeCustomEntitiesOperation operation = await aiClient.RecognizeCustomEntitiesAsync(WaitUntil.Completed, batchedDocuments, projectName, deploymentName);

    await foreach (RecognizeCustomEntitiesResultCollection documentsInPage in operation.Value)
    {
        foreach (RecognizeEntitiesResult documentResult in documentsInPage)
        {
            Console.WriteLine($"Result for \"{documentResult.Id}\":");

            if (documentResult.HasError)
            {
                Console.WriteLine($"  Error!");
                Console.WriteLine($"  Document error code: {documentResult.Error.ErrorCode}");
                Console.WriteLine($"  Message: {documentResult.Error.Message}");
                Console.WriteLine();
                continue;
            }

            Console.WriteLine($"  Recognized {documentResult.Entities.Count} entities:");

            foreach (CategorizedEntity entity in documentResult.Entities)
            {
                Console.WriteLine($"  Entity: {entity.Text}");
                Console.WriteLine($"  Category: {entity.Category}");
                Console.WriteLine($"  Offset: {entity.Offset}");
                Console.WriteLine($"  Length: {entity.Length}");
                Console.WriteLine($"  ConfidenceScore: {entity.ConfidenceScore}");
                Console.WriteLine($"  SubCategory: {entity.SubCategory}");
                Console.WriteLine();
            }

            Console.WriteLine();
        }
    }
    ```

    **Python**: custom-entities.py

    ```Python
    # Extract entities
    operation = ai_client.begin_recognize_custom_entities(
        batchedDocuments,
        project_name=project_name,
        deployment_name=deployment_name
    )

    document_results = operation.result()

    for doc, custom_entities_result in zip(files, document_results):
        print(doc)
        if custom_entities_result.kind == "CustomEntityRecognition":
            for entity in custom_entities_result.entities:
                print(
                    "\tEntity '{}' has category '{}' with confidence score of '{}'".format(
                        entity.text, entity.category, entity.confidence_score
                    )
                )
        elif custom_entities_result.is_error is True:
            print("\tError with code '{}' and message '{}'".format(
                custom_entities_result.error.code, custom_entities_result.error.message
                )
            )
    ```

1. Salvare le modifiche apportate al file di codice.

## Testare l'applicazione

A questo punto l'applicazione è pronta per il test.

1. Nel terminale integrato per la cartella **classify-text**, immettere il comando seguente per eseguire il programma:

    - **C#**: `dotnet run`
    - **Python**: `python custom-entities.py`

    > **Suggerimento**: è possibile usare l'icona **Ingrandisci dimensioni pannello** (**^**) nella barra degli strumenti del terminale per visualizzare altro testo della console.

1. Osservare l'output. L'applicazione deve elencare i dettagli delle entità presenti in ogni file di testo.

## Eseguire la pulizia

Quando il progetto non è più necessario, è possibile eliminarlo dalla pagina **Progetti** in Language Studio. È anche possibile rimuovere il servizio Lingua di Azure AI e l'account di archiviazione associato nel [portale di Azure](https://portal.azure.com).
