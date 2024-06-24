---
lab:
  title: Analizzare il testo
  module: Module 3 - Develop natural language processing solutions
---

# Analizzare il testo

**Lingua di Azure** supporta l'analisi del testo, tra cui il rilevamento della lingua, l'analisi del sentiment, l'estrazione di frasi chiave e il riconoscimento di entità.

Si supponga, ad esempio, che un'agenzia di viaggi voglia elaborare recensioni di alberghi inviate al sito Web della società. Il servizio Lingua di Azure AI consente di determinare la lingua in cui è scritta ogni recensione, la valutazione (positiva, neutra o negativa) delle recensioni, le frasi chiave che possono indicare gli argomenti principali discussi nella recensione e le entità denominate, ad esempio località, luoghi di interesse o persone citate nelle recensioni.

## Effettuare il provisioning di una risorsa *Lingua di Azure AI*

Se non ne è già disponibile una nella sottoscrizione, è necessario effettuare il provisioning di una risorsa del servizio **Lingua di Azure AI** nella sottoscrizione di Azure.

1. Aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.
1. Nel campo di ricerca nella parte superiore cercare i **Servizi di Azure AI**. Quindi, nei risultati selezionare **Crea** in **Servizio linguistico**.
1. Selezionare **Continua a creare la risorsa**.
1. Effettuare il provisioning della risorsa usando le impostazioni seguenti:
    - **Sottoscrizione**: *la sottoscrizione di Azure usata*.
    - **Gruppo di risorse**: *Selezionare o creare un gruppo di risorse*.
    - **Area**: *scegliere una qualsiasi area disponibile*
    - **Nome**: *immettere un nome univoco*.
    - **Piano tariffario**: Selezionare **F0** (*gratuito*) o **S** (*standard*) se F non è disponibile.
    - **Informativa sull'Intelligenza artificiale responsabile**: D'accordo.
1. Selezionare **Rivedi e crea**, quindi **Crea** per effettuare il provisioning della risorsa.
1. Attendere il completamento della distribuzione e quindi passare alla risorsa distribuita.
1. Visualizzare la pagina **Chiavi ed endpoint**. Le informazioni contenute in questa pagina saranno necessarie più avanti nell'esercizio.

## Preparare lo sviluppo di un'app in Visual Studio Code

Si svilupperà l'app di analisi del testo usando Visual Studio Code. I file di codice per l'app sono stati forniti in un repository GitHub.

> **Suggerimento**: Se il repository **mslearn-ai-language** è già stato clonato, aprirlo in Visual Studio Code. In caso contrario, seguire questa procedura per clonarla nell'ambiente di sviluppo.

1. Avviare Visual Studio Code.
2. Aprire il riquadro comandi (MAIUSC+CTRL+P) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/mslearn-ai-language` in una cartella locale. Non è importante usare una cartella specifica.
3. Dopo la clonazione del repository, aprire la cartella in Visual Studio Code.

    > **Nota**: Se Visual Studio Code visualizza un messaggio popup per chiedere se si considera attendibile il codice che si apre, fare clic sull'opzione **Sì, considero attendibili gli autori** nel popup.

4. Attendere il completamento dell'installazione di file aggiuntivi per supportare i progetti in codice C# nel repository.

    > **Nota**: se viene richiesto di aggiungere gli asset necessari per la compilazione e il debug, selezionare **Non adesso**.

## Configurare l'applicazione

Sono state fornite applicazioni sia per C# che per Python, oltre a un file di testo di esempio che verrà usato per testare il riepilogo. Entrambe le app presentano la stessa funzionalità. Prima di tutto, verranno completate alcune parti chiave dell'applicazione per abilitarla all'uso della risorsa Lingua di Azure AI.

1. Nel riquadro **Esplora risorse** di Visual Studio Code, passare alla cartella **Labfiles/01-analyze-text** ed espandere la cartella **CSharp** o **Python**, in base alle preferenze della lingua e alla cartella **text-analysis** che contiene. Ogni cartella contiene i file specifici della lingua per un'app in cui si intende integrare la funzionalità di analisi del testo di Lingua di Azure AI.
2. Fare clic con il pulsante destro del mouse sulla cartella **text-analysis** contenente i file di codice e aprire un terminale integrato. Installare quindi il pacchetto Text Analytics SDK di Lingua di Azure AI eseguendo il comando appropriato per il linguaggio scelto. Per l'esercizio Python, installare anche il pacchetto `dotenv`:

    **C#:**

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.3.0
    ```

    **Python**:

    ```
    pip install azure-ai-textanalytics==5.3.0
    pip install python-dotenv
    ```

3. Nel riquadro **Esplo risorse**, nella cartella **text-analysis**, aprire il file di configurazione per la lingua preferita

    - **C#**: appsettings.json
    - **Python**: .env
    
4. Aggiornare i valori di configurazione per includere l'**endpoint** e una **chiave** dalla risorsa Lingua di Azure creata (disponibile nella pagina **Chiavi ed endpoint** per la risorsa Lingua di Azure AI)
5. Salvare il file di configurazione.

6. Si noti che la cartella **text-analysis** contiene un file di codice per l'applicazione client:

    - **C#**: Program.cs
    - **Python**: text-analysis.py

    Aprire il file di codice e nella parte superiore, sotto i riferimenti agli spazi dei nomi esistenti, individuare il commento **Import namespaces**. In questo commento aggiungere quindi il codice seguente specifico per la lingua per importare gli spazi dei nomi necessari per usare Text Analytics SDK:

    **C#**: Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.TextAnalytics;
    ```

    **Python**: text-analysis.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    ```

7. Nella funzione **Main** si noti che il codice per caricare l'endpoint e la chiave di servizi Lingua di Azure AI dal file di configurazione è già stato fornito. Individuare quindi il commento **Create client using endpoint and key** e aggiungere il codice seguente per creare un client per l'API Analisi del testo:

    **C#**: Programs.cs

    ```C#
    // Create client using endpoint and key
    AzureKeyCredential credentials = new AzureKeyCredential(aiSvcKey);
    Uri endpoint = new Uri(aiSvcEndpoint);
    TextAnalyticsClient aiClient = new TextAnalyticsClient(endpoint, credentials);
    ```

    **Python**: text-analysis.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

8. Salvare le modifiche e tornare al terminale integrato per la cartella **text-analysis**, quindi immettere il comando seguente per eseguire il programma:

    - **C#**: `dotnet run`
    - **Python**: `python text-analysis.py`

    > **Suggerimento**: È possibile usare l'icona **Ingrandisci dimensioni pannello** (**^**) nella barra degli strumenti del terminale per visualizzare altro testo della console.

9. Osservare l'output perché il codice deve essere eseguito senza errori, visualizzando i contenuti di ogni file di testo di recensione nella cartella **reviews**. L'applicazione crea correttamente un client per l'API Analisi del testo, ma non la usa. Questo problema verrà risolto nella procedura successiva.

## Aggiungere codice per rilevare la lingua

Ora che è stato creato un client per l'API, è possibile usarlo per rilevare la lingua in cui viene scritta ogni recensione.

1. Nella funzione **Main** per il programma, individuare il commento **Get language**. Sotto questo commento aggiungere quindi il codice necessario per rilevare la lingua in ogni recensione:

    **C#**: Programs.cs

    ```csharp
    // Get language
    DetectedLanguage detectedLanguage = aiClient.DetectLanguage(text);
    Console.WriteLine($"\nLanguage: {detectedLanguage.Name}");
    ```

    **Python**: text-analysis.py

    ```python
    # Get language
    detectedLanguage = ai_client.detect_language(documents=[text])[0]
    print('\nLanguage: {}'.format(detectedLanguage.primary_language.name))
    ```

     > **Nota**: *in questo esempio ogni recensione viene analizzata singolarmente e di conseguenza viene effettuata una chiamata separata al servizio per ogni file. Un approccio alternativo consiste nel creare una raccolta di documenti e passarli al servizio in un'unica chiamata. In entrambi gli approcci, la risposta restituita dal servizio è costituita da una raccolta di documenti. È per questo motivo che nel codice Python precedente viene specificato l'indice del primo (e unico) documento incluso nella risposta ([0]).*

1. Salva le modifiche. Tornare quindi al terminale integrato per la cartella **text-analysis** ed eseguire di nuovo il programma.
1. Osservare l'output, notando che questa volta viene identificata la lingua per ogni recensione.

## Aggiungere codice per valutare il sentiment

L'*analisi valutazione* è una tecnica usata in genere per classificare il testo come *positivo* o *negativo* (oppure possibile *neutro* o *misto*). Viene usato in genere per analizzare post di social media, recensioni di prodotti e altri elementi in cui la valutazione del testo può fornire utili informazioni dettagliate.

1. Nella funzione **Main** per il programma, individuare il commento **Get sentiment**. Sotto questo commento aggiungere quindi il codice necessario per rilevare la valutazione di ogni recensione:

    **C#**: Program.cs

    ```csharp
    // Get sentiment
    DocumentSentiment sentimentAnalysis = aiClient.AnalyzeSentiment(text);
    Console.WriteLine($"\nSentiment: {sentimentAnalysis.Sentiment}");
    ```

    **Python**: text-analysis.py

    ```python
    # Get sentiment
    sentimentAnalysis = ai_client.analyze_sentiment(documents=[text])[0]
    print("\nSentiment: {}".format(sentimentAnalysis.sentiment))
    ```

1. Salva le modifiche. Tornare quindi al terminale integrato per la cartella **text-analysis** ed eseguire di nuovo il programma.
1. Osservare l'output, notando che viene rilevata la valutazione delle recensioni.

## Aggiungere codice per identificare le frasi chiave

Può essere utile identificare le frasi chiave nel corpo di un testo per determinare i principali argomenti trattati.

1. Nella funzione **Main** per il programma, individuare il commento **Get key phrases**. Sotto questo commento aggiungere quindi il codice necessario per rilevare le frasi chiave in ogni recensione:

    **C#**: Program.cs

    ```csharp
    // Get key phrases
    KeyPhraseCollection phrases = aiClient.ExtractKeyPhrases(text);
    if (phrases.Count > 0)
    {
        Console.WriteLine("\nKey Phrases:");
        foreach(string phrase in phrases)
        {
            Console.WriteLine($"\t{phrase}");
        }
    }
    ```

    **Python**: text-analysis.py

    ```python
    # Get key phrases
    phrases = ai_client.extract_key_phrases(documents=[text])[0].key_phrases
    if len(phrases) > 0:
        print("\nKey Phrases:")
        for phrase in phrases:
            print('\t{}'.format(phrase))
    ```

1. Salva le modifiche. Tornare quindi al terminale integrato per la cartella **text-analysis** ed eseguire di nuovo il programma.
1. Osservare l'output, notando che ogni documento contiene frasi chiave che forniscono alcune informazioni dettagliate sul contenuto della recensione.

## Aggiungere codice per estrarre le entità

Spesso nei documenti o in altri corpi del testo vengono citati persone, luoghi, periodi di tempo o altre entità. L'API Analisi del testo è in grado di rilevare più categorie (e sottocategorie) dell'entità nel testo.

1. Nella funzione **Main** per il programma, individuare il commento **Get entities**. Sotto questo commento aggiungere quindi il codice necessario per identificare le entità citate in ogni recensione:

    **C#**: Program.cs

    ```csharp
    // Get entities
    CategorizedEntityCollection entities = aiClient.RecognizeEntities(text);
    if (entities.Count > 0)
    {
        Console.WriteLine("\nEntities:");
        foreach(CategorizedEntity entity in entities)
        {
            Console.WriteLine($"\t{entity.Text} ({entity.Category})");
        }
    }
    ```

    **Python**: text-analysis.py

    ```python
    # Get entities
    entities = ai_client.recognize_entities(documents=[text])[0].entities
    if len(entities) > 0:
        print("\nEntities")
        for entity in entities:
            print('\t{} ({})'.format(entity.text, entity.category))
    ```

1. Salva le modifiche. Tornare quindi al terminale integrato per la cartella **text-analysis** ed eseguire di nuovo il programma.
1. Osservare l'output, notando le entità che sono state rilevate nel testo.

## Aggiungere codice per estrarre le entità collegate

Oltre alle entità classificate, l'API Analisi del testo è in grado di rilevare le entità per cui sono presenti collegamenti noti alle origini dati, ad esempio Wikipedia.

1. Nella funzione **Main** per il programma, individuare il commento **Get linked entities**. Sotto questo commento aggiungere quindi il codice necessario per identificare le entità collegate citate in ogni recensione:

    **C#**: Program.cs

    ```csharp
    // Get linked entities
    LinkedEntityCollection linkedEntities = aiClient.RecognizeLinkedEntities(text);
    if (linkedEntities.Count > 0)
    {
        Console.WriteLine("\nLinks:");
        foreach(LinkedEntity linkedEntity in linkedEntities)
        {
            Console.WriteLine($"\t{linkedEntity.Name} ({linkedEntity.Url})");
        }
    }
    ```

    **Python**: text-analysis.py

    ```python
    # Get linked entities
    entities = ai_client.recognize_linked_entities(documents=[text])[0].entities
    if len(entities) > 0:
        print("\nLinks")
        for linked_entity in entities:
            print('\t{} ({})'.format(linked_entity.name, linked_entity.url))
    ```

1. Salva le modifiche. Tornare quindi al terminale integrato per la cartella **text-analysis** ed eseguire di nuovo il programma.
1. Osservare l'output, notando le entità collegate che sono state identificate.

## Pulire le risorse

Se si è terminato di esplorare il servizio Lingua di Azure AI, è possibile eliminare le risorse create in questo esercizio. In tal caso, eseguire la procedura seguente:

1. Aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.

2. Passare alla risorsa Lingua di Azure AI creata in questo lab.

3. Nella pagina della risorsa selezionare **Elimina** e seguire le istruzioni per eliminare la risorsa.

## Ulteriori informazioni

Per altre informazioni sull'uso di **Lingua di Azure AI**, vedere la [documentazione](https://learn.microsoft.com/azure/ai-services/language-service/).
