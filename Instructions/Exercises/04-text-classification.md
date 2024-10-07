---
lab:
  title: Classificazione personalizzata del testo
  module: Module 3 - Getting Started with Natural Language Processing
---

# Classificazione personalizzata del testo

Lingua di Azure AI offre diverse funzionalità di elaborazione del linguaggio naturale, tra cui l'identificazione delle frasi chiave, il riepilogo del testo e l'analisi del sentiment. Il servizio Lingua offre anche funzionalità personalizzate, ad esempio la risposta alle domande personalizzate e la classificazione personalizzata del testo.

Per testare la classificazione personalizzata del testo del servizio Lingua di Azure AI, si configurerà il modello usando Language Studio e quindi si userà una piccola applicazione da riga di comando eseguita in Cloud Shell per testarla. Lo stesso modello e le stesse funzionalità usate in questo esempio possono essere usati per le applicazioni reali.

## Effettuare il provisioning di una risorsa *Lingua di Azure AI*

Se non ne è già disponibile una nella sottoscrizione, è necessario effettuare il provisioning di una risorsa del **servizio Lingua di Azure AI**. Inoltre, per usare la classificazione personalizzata del testo, è necessario abilitare la funzionalità **Estrazione e classificazione personalizzata del testo**.

1. In un browser aprire il portale di Azure all'indirizzo `https://portal.azure.com` e accedere con il proprio account Microsoft.
1. Selezionare il campo di ricerca nella parte superiore del pulsante del portale, cercare `Azure AI services` e creare una risorsa del **servizio Lingua**.
1. Selezionare la casella che include **Classificazione personalizzata del testo**. Selezionare quindi **Continua a creare la risorsa**.
1. Creare una risorsa con le impostazioni seguenti:
    - **Sottoscrizione**: *la sottoscrizione di Azure usata*.
    - **Gruppo di risorse**: *Selezionare o creare un gruppo di risorse*.
    - **Area**: *scegliere una qualsiasi area disponibile*:
    - **Nome**: *immettere un nome univoco*.
    - **Piano tariffario**: Selezionare **F0** (*gratuito*) o **S** (*standard*) se F non è disponibile.
    - **Account di archiviazione**: Nuovo account di archiviazione
      - **Nome account di archiviazione**: *immettere un nome univoco*.
      - **Tipo di account di archiviazione: ** Archiviazione con ridondanza locale Standard
    - **Avviso per intelligenza artificiale responsabile**: opzione selezionata

1. Selezionare **Rivedi e crea**, quindi **Crea** per effettuare il provisioning della risorsa.
1. Attendere il completamento della distribuzione e quindi passare alla risorsa distribuita.
1. Visualizzare la pagina **Chiavi ed endpoint**. Le informazioni contenute in questa pagina saranno necessarie più avanti nell'esercizio.

## Caricare articoli di esempio

Dopo aver creato il servizio Lingua di Azure AI e l'account di archiviazione, sarà necessario caricare articoli di esempio per eseguire il training del modello in un secondo momento.

1. In una nuova scheda del browser scaricare gli annunci di esempio da `https://aka.ms/classification-articles` ed estrarre i file in una cartella di propria scelta.

1. Nel portale di Azure passare all'account di archiviazione creato e selezionarlo.

1. Nell'account di archiviazione selezionare **Configurazione** sotto **Impostazioni**. Nella schermata Configurazione abilitare l'opzione **Consenti accesso anonimo ai BLOB** e quindi selezionare **Salva**.

1. Selezionare **Contenitori** nel menu a sinistra, sotto **Archiviazione dati**. Nella schermata visualizzata selezionare **+ Contenitore**. Assegnare al contenitore il nome `articles`, quindi impostare **Livello di accesso anonimo** su **Contenitore (accesso in lettura anonimo per contenitori e BLOB)**.

    > **NOTA**: quando si configura un account di archiviazione per una soluzione reale, prestare attenzione ad assegnare il livello di accesso appropriato. Per altre informazioni su ogni livello di accesso, vedere la [documentazione su Archiviazione di Azure](https://learn.microsoft.com/azure/storage/blobs/anonymous-read-access-configure).

1. Dopo aver creato il contenitore, selezionarlo e quindi selezionare il pulsante **Carica**. Selezionare **Cerca file** per cercare gli articoli di esempio scaricati. Quindi selezionare **Carica**.

## Creare un progetto di classificazione personalizzata del testo

Al termine della configurazione, creare un progetto di classificazione personalizzata del testo. Questo progetto offre un luogo di lavoro per compilare il modello, eseguirne il training e quindi distribuirlo.

> **NOTA**: In questo lab si usa **Language Studio**, ma è anche possibile creare, compilare il modello, eseguirne il training e distribuirlo tramite l'API REST.

1. In una nuova scheda del browser aprire il portale Language Studio di Azure per intelligenza artificiale all'indirizzo `https://language.cognitive.azure.com/` e accedere usando l'account Microsoft associato alla sottoscrizione di Azure.
1. Se viene richiesto di scegliere una risorsa del servizio Lingua, selezionare le impostazioni seguenti:

    - **Directory di Azure**: directory di Azure contenente la sottoscrizione.
    - **Sottoscrizione di Azure**: sottoscrizione di Azure in uso.
    - **Tipo di risorsa**: lingua.
    - **Risorsa della lingua**: risorsa Lingua di Azure AI creata in precedenza.

    Se <u>non</u> viene chiesto di scegliere una risorsa Lingua, è possibile che nella sottoscrizione siano presenti più risorse di questo tipo e in tal caso:

    1. Sulla barra nella parte superiore della pagina selezionare il pulsante **Impostazioni (&#9881;)**.
    2. Nella pagina **Impostazioni** visualizzare la scheda **Risorse**.
    3. Selezionare la risorsa del servizio Lingua appena creata e fare clic su **Cambiare risorsa**.
    4. Nella parte superiore della pagina fare clic su **Language Studio** per tornare alla home page di Language Studio

1. Nella parte superiore del portale scegliere **Classificazione personalizzata del testo** dal menu **Crea nuovo**.
1. Viene visualizzata la pagina **Connetti archiviazione**. Tutti i valori saranno già stati compilati. Quindi, selezionare **Avanti**.
1. Nella pagina **Seleziona tipo di progetto** selezionare **Classificazione etichetta singola**. Quindi seleziona **Avanti**.
1. Nel riquadro **Immetti informazioni di base** impostare quanto segue:
    - **Nome**: `ClassifyLab`  
    - **Lingua principale testo**: Inglese (US)
    - **Descrizione**: `Custom text lab`

1. Selezionare **Avanti**.
1. Nella pagina **Scegli contenitore** impostare l'elenco a discesa **Contenitore archivio BLOB** sul contenitore *articles*.
1. Selezionare l'opzione **No, è necessario assegnare etichette ai file nell'ambito di questo progetto**. Quindi seleziona **Avanti**.
1. Seleziona **Crea progetto**.

> **Suggerimento**: se viene visualizzato un errore relativo alla mancata autorizzazione per eseguire questa operazione, è necessario aggiungere un'assegnazione di ruolo. Per risolvere questo problema, per l'utente che esegue il lab, è stato aggiunto il ruolo "Collaboratore ai dati del BLOB di archiviazione" nell'account di archiviazione. Per informazioni dettagliate, vedere la [pagina della documentazione](https://learn.microsoft.com/azure/ai-services/language-service/custom-named-entity-recognition/how-to/create-project?tabs=portal%2Clanguage-studio#enable-identity-management-for-your-resource)

## Assegnare etichette ai dati

Dopo aver creato il progetto, è necessario assegnare etichette ai dati, ovvero contrassegnarli, per eseguire il training del modello al fine di classificare il testo.

1. A sinistra selezionare **Etichettatura dei dati**, se non è già selezionata. Viene visualizzato un elenco dei file caricati nell'account di archiviazione.
1. Sul lato destro, nel riquadro **Attività** selezionare **+ Aggiungi classe**.  Gli articoli di questo lab rientrano in quattro classi da creare, ovvero `Classifieds`, `Sports`, `News` e `Entertainment`.

    ![Screenshot che mostra la pagina per l'etichettatura dei dati con il pulsante per aggiungere una classe.](../media/tag-data-add-class-new.png#lightbox)

1. Dopo aver creato le quattro classi, selezionare **Articolo 1** per iniziare. Qui è possibile leggere l'articolo e definire la classe in cui si trova il file e il set di dati (training o test) a cui assegnarlo.
1. Assegnare ogni articolo alla classe e al set di dati appropriati (training o test) usando il riquadro **Attività** a destra.  È possibile selezionare un'etichetta dall'elenco a destra e impostare ogni articolo su **training** o **test** usando le opzioni nella parte inferiore del riquadro Attività. Selezionare **Documento successivo** per passare al documento successivo. Ai fini di questo lab, si definiranno gli elementi da usare per il training e per il test del modello:

    | Articolo  | Classe  | Set di dati  |
    |---------|---------|---------|
    | Articolo 1 | Sport | Formazione |
    | Articolo 10 | Novità | Formazione |
    | Articolo 11 | Entertainment | Test in corso |
    | Articolo 12 | Novità | Test in corso |
    | Articolo 13 | Sport | Test in corso |
    | Articolo 2 | Sport | Formazione |
    | Articolo 3 | Classificati | Formazione |
    | Articolo 4 | Classificati | Formazione |
    | Articolo 5 | Entertainment | Formazione |
    | Articolo 6 | Entertainment | Formazione |
    | Articolo 7 | Novità | Formazione |
    | Articolo 8 | Novità | Formazione |
    | Articolo 9 | Entertainment | Formazione |

    > **NOTA** I file in Language Studio sono elencati alfabeticamente, motivo per cui l'elenco precedente non è in ordine sequenziale. Quando si assegnano etichette agli articoli, assicurarsi di visitare entrambe le pagine dei documenti.

1. Selezionare **Salva etichette** per salvare le etichette.

## Eseguire il training del modello

Dopo aver assegnato etichette ai dati, è necessario eseguire il training del modello.

1. Selezionare **Processi di training** dal menu a sinistra.
1. Selezionare **Avvia un processo di training**.
1. Eseguire il training di un nuovo modello denominato `ClassifyArticles`.
1. Scegliere **Usa una suddivisione manuale dei dati di training e test**.

    > **SUGGERIMENTO** Nei progetti di classificazione personalizzati il servizio Lingua di Azure AI divide automaticamente il set di test in base alla percentuale, una funzionalità utile per i set di dati di grandi dimensioni. Con set di dati di dimensioni minori, è importante eseguire il training con la distribuzione della classe corretta.

1. Selezionare **Training**

> **IMPORTANTE** A volte il training del modello può richiedere alcuni minuti. Al termine dell'operazione, verrà visualizzata una notifica.

## Valutare il modello

Nelle applicazioni reali della classificazione del testo è importante valutare e migliorare il modello per verificare che venga eseguito come previsto.

1. Selezionare **Prestazioni del modello** e quindi il modello **ClassifyArticles** È possibile visualizzare il punteggio del modello, le metriche delle prestazioni e il momento in cui è stato eseguito il training. Se il punteggio del modello non è 100%, significa che uno dei documenti usati per il test non ha valutato l'etichetta. Questi errori consentono di comprendere le aree da migliorare.
1. Selezionare la scheda **Dettagli set di test**. In caso di errori, questa scheda consente di visualizzare gli articoli indicati per il test e quali sono i valori stimati dal modello e se tale valore è in conflitto con l'etichetta di test. Per impostazione predefinita, la scheda mostra solo stime non corrette. È possibile attivare l'opzione **Mostra solo le mancate corrispondenze** per visualizzare tutti gli articoli indicati per il test e le relative previsioni.

## Distribuire il modello

Quando il training del modello restituisce risultati soddisfacenti, è possibile distribuire il modello in modo da iniziare a classificare il testo tramite l'API.

1. Nel riquadro a sinistra selezionare **Distribuzione del modello**.
1. Selezionare **Aggiungi distribuzione**, quindi immettere `articles` nel campo **Crea un nuovo nome di distribuzione** e selezionare **ClassifyArticles** nel campo **Modello**.
1. Selezionare **Distribuisci** per distribuire il modello
1. Dopo aver distribuito il modello, lasciare aperta la pagina. Nel passaggio successivo saranno necessari il nome del progetto e della distribuzione.

## Prepararsi a sviluppare un'app in Visual Studio Code

Per testare le funzionalità di classificazione personalizzata del testo del servizio Lingua di Azure AI, si svilupperà una semplice applicazione console in Visual Studio Code.

> **Suggerimento**: se il repository **mslearn-ai-language** è già stato clonato, aprirlo in Visual Studio Code. In caso contrario, eseguire questi passaggi per clonarlo nell'ambiente di sviluppo.

1. Avviare Visual Studio Code.
2. Aprire il riquadro comandi (MAIUSC+CTRL+P) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/mslearn-ai-language` in una cartella locale. Non è importante usare una cartella specifica.
3. Dopo la clonazione del repository, aprire la cartella in Visual Studio Code.

    > **Nota**: Se Visual Studio Code visualizza un messaggio popup per chiedere se si considera attendibile il codice che si apre, fare clic sull'opzione **Sì, considero attendibili gli autori** nel popup.

4. Attendere il completamento dell'installazione di file aggiuntivi per supportare i progetti in codice C# nel repository.

    > **Nota**: se viene richiesto di aggiungere gli asset necessari per la compilazione e il debug, selezionare **Non adesso**.

## Configurare l'applicazione

Sono state fornite applicazioni sia per C# che per Python, oltre a un file di testo di esempio che verrà usato per testare il riepilogo. Entrambe le app presentano la stessa funzionalità. Prima di tutto, verranno completate alcune parti chiave dell'applicazione per consentirle di usare la risorsa Lingua di Azure AI.

1. Nel riquadro **Esplora risorse** di Visual Studio Code passare alla cartella **Labfiles/04-text-classification**, espandere la cartella **CSharp** o **Python** in base al linguaggio scelto ed espandere la cartella **classify-text** al suo interno. Ogni cartella contiene i file specifici del linguaggio per un'app in cui si intende integrare la funzionalità di classificazione del testo di Lingua di Azure AI.
1. Fare clic con il pulsante destro del mouse sulla cartella **classify-text** contenente i file di codice e aprire un terminale integrato. Installare quindi il pacchetto Text Analytics SDK di Lingua di Azure AI eseguendo il comando appropriato per il linguaggio scelto:

    **C#:**

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.3.0
    ```

    **Python**:

    ```
    pip install azure-ai-textanalytics==5.3.0
    ```

1. Nel riquadro **Esplora risorse**, nella cartella **classify-text** aprire il file di configurazione per il linguaggio preferito

    - **C#**: appsettings.json
    - **Python**: .env
    
1. Aggiornare i valori di configurazione per includere l'**endpoint** e una **chiave** dalla risorsa Lingua di Azure creata (disponibile nella pagina **Chiavi ed endpoint** per la risorsa Lingua di Azure AI nel portale di Azure). Il file deve contenere già i nomi di progetto e distribuzione per il modello di classificazione testo.
1. Salvare il file di configurazione.

## Aggiungere codice per classificare i documenti

A questo punto è possibile usare il servizio Lingua di Azure AI per classificare documenti.

1. Espandere la cartella **articles** nella cartella **classify-text** per visualizzare gli articoli di testo che l'applicazione classificherà.
1. Nella cartella **classify-text** aprire il file di codice per l'applicazione client:

    - **C#**: Program.cs
    - **Python**: classify-text.py

1. Individuare il commento **Import namespaces**. In questo commento aggiungere quindi il codice seguente specifico per la lingua per importare gli spazi dei nomi necessari per usare Text Analytics SDK:

    **C#**: Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.TextAnalytics;
    ```

    **Python**: classify-text.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. Nella funzione **Main** notare che il codice per caricare l'endpoint e la chiave del servizio Lingua di Azure AI e i nomi del progetto e della distribuzione indicati nel file di configurazione sono già stati forniti. Individuare quindi il commento **Create client using endpoint and key** e aggiungere il codice seguente per creare un client per l'API Analisi del testo:

    **C#**: Programs.cs

    ```csharp
    // Create client using endpoint and key
    AzureKeyCredential credentials = new AzureKeyCredential(aiSvcKey);
    Uri endpoint = new Uri(aiSvcEndpoint);
    TextAnalyticsClient aiClient = new TextAnalyticsClient(endpoint, credentials);
    ```

    **Python**: classify-text.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

1. Nella funzione **Main** notare che il codice esistente legge tutti i file nella cartella **articles** e crea un elenco del loro contenuto. Individuare quindi il commento **Get Classifications** e aggiungere il codice seguente:

    **C#**: Program.cs

    ```csharp
    // Get Classifications
    ClassifyDocumentOperation operation = await aiClient.SingleLabelClassifyAsync(WaitUntil.Completed, batchedDocuments, projectName, deploymentName);

    int fileNo = 0;
    await foreach (ClassifyDocumentResultCollection documentsInPage in operation.Value)
    {
        
        foreach (ClassifyDocumentResult documentResult in documentsInPage)
        {
            Console.WriteLine(files[fileNo].Name);
            if (documentResult.HasError)
            {
                Console.WriteLine($"  Error!");
                Console.WriteLine($"  Document error code: {documentResult.Error.ErrorCode}");
                Console.WriteLine($"  Message: {documentResult.Error.Message}");
                continue;
            }

            Console.WriteLine($"  Predicted the following class:");
            Console.WriteLine();

            foreach (ClassificationCategory classification in documentResult.ClassificationCategories)
            {
                Console.WriteLine($"  Category: {classification.Category}");
                Console.WriteLine($"  Confidence score: {classification.ConfidenceScore}");
                Console.WriteLine();
            }
            fileNo++;
        }
    }
    ```
    
    **Python**: classify-text.py

    ```Python
    # Get Classifications
    operation = ai_client.begin_single_label_classify(
        batchedDocuments,
        project_name=project_name,
        deployment_name=deployment_name
    )

    document_results = operation.result()

    for doc, classification_result in zip(files, document_results):
        if classification_result.kind == "CustomDocumentClassification":
            classification = classification_result.classifications[0]
            print("{} was classified as '{}' with confidence score {}.".format(
                doc, classification.category, classification.confidence_score)
            )
        elif classification_result.is_error is True:
            print("{} has an error with code '{}' and message '{}'".format(
                doc, classification_result.error.code, classification_result.error.message)
            )
    ```

1. Salvare le modifiche apportate al file di codice.

## Testare l'applicazione

A questo punto l’applicazione è pronta per il test.

1. Nel terminale integrato per la cartella **classify-text** immettere il comando seguente per eseguire il programma:

    - **C#**: `dotnet run`
    - **Python**: `python classify-text.py`

    > **Suggerimento**: è possibile usare l'icona **Ingrandisci dimensioni pannello** (**^**) nella barra degli strumenti del terminale per visualizzare altro testo della console.

1. Osservare l'output. L'applicazione deve elencare un punteggio di classificazione e di attendibilità per ogni file di testo.


## Eseguire la pulizia

Quando il progetto non è più necessario, è possibile eliminarlo dalla pagina **Progetti** in Language Studio. È anche possibile rimuovere il servizio Lingua di Azure AI e l'account di archiviazione associato nel [portale di Azure](https://portal.azure.com).
