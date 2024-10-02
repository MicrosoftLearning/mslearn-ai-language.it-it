---
lab:
  title: Creare un modello di comprensione del linguaggio con il servizio Lingua di Azure AI (anteprima)
  module: Module 5 - Create language understanding solutions
---

# Creare un modello di comprensione del linguaggio con il servizio Lingua

> **NOTA**: la funzionalità di comprensione del linguaggio di conversazione del servizio Lingua di Azure AI è attualmente in anteprima ed è soggetta a modifiche. In alcuni casi, il training del modello può non riuscire. In tal caso, riprovare.  

Il servizio Lingua di Azure AI consente di definire un modello di *comprensione del linguaggio di conversazione* che le applicazioni possono usare per interpretare l'input in linguaggio naturale degli utenti, stimare la *finalità* degli utenti, ossia cosa vogliono ottenere, e identificare le *entità* a cui applicare la finalità.

Un modello linguistico per conversazioni per un'applicazione orologio potrebbe ad esempio elaborare un input simile al seguente:

*What is the time in London?*

Questo tipo di input è un esempio di *espressione* (qualcosa che un utente potrebbe dire o digitare) la cui *finalità* è recuperare l'orario in una località specifica (un'*entità *), in questo caso Londra.

> **NOTA**: l'attività di un modello linguistico per conversazioni è stimare la finalità dell'utente e identificare le entità a cui si applica. <u>Non</u> è compito di un modello linguistico per conversazioni eseguire effettivamente le azioni necessarie per soddisfare la finalità. Ad esempio, un'applicazione orologio può usare un modello linguistico per conversazioni per comprendere che l'utente vuole conoscere l'ora di Londra, ma è l'applicazione client stessa che deve quindi implementare la logica per determinare l'ora corretta e presentarla all'utente.

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

## Creare un progetto di comprensione del linguaggio di conversazione

Ora che è stata creata una risorsa di creazione, è possibile usarla per creare un progetto di comprensione del linguaggio di conversazione.

1. In una nuova scheda del browser aprire il portale Studio di Lingua di Azure AI all'indirizzo `https://language.cognitive.azure.com/` e accedere usando l'account Microsoft associato alla sottoscrizione di Azure.

1. Se viene richiesto di scegliere una risorsa del servizio Lingua, selezionare le impostazioni seguenti:

    - **Directory di Azure**: directory di Azure contenente la sottoscrizione.
    - **Sottoscrizione di Azure**: sottoscrizione di Azure in uso.
    - **Tipo di risorsa**: lingua.
    - **Risorsa della lingua**: risorsa Lingua di Azure AI creata in precedenza.

    Se <u>non</u> viene chiesto di scegliere una risorsa Lingua, è possibile che nella sottoscrizione siano presenti più risorse di questo tipo e in tal caso:

    1. Nella barra nella parte superiore della pagina, selezionare il **Settings (&#9881;)**.
    2. Nella pagina **Impostazioni** visualizzare la scheda **Risorse**.
    3. Selezionare la risorsa del servizio Lingua appena creata e fare clic su **Cambiare risorsa**.
    4. Nella parte superiore della pagina fare clic su **Language Studio** per tornare alla home page di Language Studio

1. Nella parte superiore del portale scegliere **Comprensione del linguaggio di conversazione** dal menu **Crea nuovo**.

1. Nella finestra di dialogo **Create a project**, nella pagina **Enter basic information** immettere i dati seguenti e quindi selezionare **Avanti**:
    - **Nome**: `Clock`
    - **Utterances primary language** (Lingua principale espressioni): Inglese
    - **Enable multiple languages in project?** (Abilitare più lingue nel progetto?): *opzione deselezionata*
    - **Descrizione**: `Natural language clock`

1. Nella pagina **Rivedi e termina**, selezionare **Crea**.

### Creare finalità

La prima cosa da fare nel nuovo progetto è definire alcune finalità. Il modello prevede in ultima analisi quale di queste finalità richiede un utente durante l'invio di un'espressione in linguaggio naturale.

> **Suggerimento**: Quando si lavora al progetto, se vengono visualizzati alcuni suggerimenti, leggerli e selezionare **OK** per eliminarli oppure selezionare **Ignora tutto**.

1. Nella scheda **Definizione dello schema**, nella scheda **Finalità**, selezionare **&#65291; Aggiungi** per aggiungere una nuova finalità denominata `GetTime`.
1. Verificare che la finalità **GetTime** sia elencata (insieme alla finalità **None** predefinita ). Aggiungere quindi le finalità aggiuntive seguenti:
    - `GetDay`
    - `GetDate`

### Etichettare ogni finalità con espressioni di esempio

Per consentire al modello di stimare la finalità richiesta da un utente, è necessario etichettare ogni finalità con alcune espressioni di esempio.

1. Nel riquadro a sinistra selezionare la pagina **Etichettatura dati**.

> **Suggerimento**: È possibile espandere il riquadro con l'icona **>>** per visualizzare i nomi delle pagine e nasconderlo di nuovo con l'icona **<<**.

1. Selezionare la nuova finalità **GetTime** e immettere l'espressione `what is the time?`. In questo modo, l'espressione viene aggiunta come input di esempio per la finalità.
1. Aggiungere le espressioni aggiuntive seguenti per la finalità **GetTime**:
    - `what's the time?`
    - `what time is it?`
    - `tell me the time`

    > **NOTA** Per aggiungere una nuova espressione, scrivere l'espressione nella casella di testo accanto alla finalità e quindi premere INVIO. 

1. Selezionare la finalità **GetDay** e aggiungere le espressioni seguenti come input di esempio per tale finalità:
    - `what day is it?`
    - `what's the day?`
    - `what is the day today?`
    - `what day of the week is it?`

1. Selezionare la finalità **GetDate** e aggiungerne le espressioni seguenti:
    - `what date is it?`
    - `what's the date?`
    - `what is the date today?`
    - `what's today's date?`

1. Dopo aver aggiunto espressioni per ognuna delle finalità, selezionare **Salva modifiche**.

### Eseguire il training e il test del modello

Ora che sono state aggiunte alcune finalità, è possibile eseguire il training del modello linguistico e verificare se riesce a stimarle correttamente dall'input dell'utente.

1. Nel riquadro a sinistra selezionare **Processi di training**. Poi selezionare **+Avvia un processo di training**.

1. Nella finestra di dialogo **Avvia un processo di training**, selezionare l'opzione per eseguire il training di un nuovo modello, denominarla `Clock`. Selezionare **Modalità di training standard** e le opzioni di **suddivisione dei dati** predefinite.

1. Per iniziare il processo di training del modello, fare clic su **Esegui il training**.

1. Al termine del training (che può richiedere alcuni minuti) lo **stato** del processo cambierà in **Training succeeded**.

1. Selezionare la pagina **Prestazioni modello** e quindi selezionare il modello **Orologio**. Visualizzare le metriche di valutazione complessive e per finalità (*precisione*, *richiamo* e *punteggio F1*) e la *matrice di confusione* generata dalla valutazione eseguita durante il training (si noti che a causa del numero ridotto di espressioni di esempio, nei risultati potrebbero non essere incluse tutte le finalità).

    > **NOTA**: per altre informazioni sulle metriche di valutazione, vedere la [documentazione](https://learn.microsoft.com/azure/ai-services/language-service/conversational-language-understanding/concepts/evaluation-metrics)

1. Passare alla pagina **Distribuzione di un modello** e quindi selezionare **Aggiungi distribuzione**.

1. Nella finestra di dialogo **Aggiungi distribuzione**, selezionare **Crea un nuovo nome di distribuzione** e quindi immettere `production`.

1. Selezionare il modello **Orologio** nel campo **Modello** e quindi selezionare **Distribuisci**. La distribuzione potrebbe richiedere alcuni minuti.

1. Quando il modello è stato distribuito, selezionare la pagina **Test distribuzioni** e quindi selezionare la distribuzione **produzione** nel campo **Nome distribuzione**.

1. Immettere il testo seguente nella casella di testo vuota e quindi selezionare **Esegui il test**:

    `what's the time now?`

    Esaminare il risultato restituito, notando che include la finalità prevista (che dovrebbe essere **GetTime**) e un punteggio di attendibilità che indica la probabilità calcolata dal modello per la finalità prevista. La scheda JSON mostra l'attendibilità comparativa per ogni finalità potenziale (quella con il punteggio di attendibilità più alto è la finalità stimata)

1. Deselezionare la casella di testo e quindi eseguire un altro test con il testo seguente:

    `tell me the time`

    Di nuovo, esaminare la finalità prevista e il punteggio di attendibilità.

1. Provare il testo seguente:

    `what's the day today?`

    Il modello dovrebbe prevedere la finalità **GetDay**.

## Aggiungere entità

Finora sono stati definite alcune espressioni semplici associate a finalità. La maggior parte delle applicazioni reali include espressioni più complesse da cui è necessario estrarre entità di dati specifiche per ottenere più contesto per la finalità.

### Aggiungere un'entità di apprendimento

Il tipo più comune di entità è quella di *apprendimento*, in cui il modello impara a identificare i valori dell'entità in base agli esempi.

1. In Language Studio tornare alla pagina **Definizione schema** e quindi, nella scheda **Entità**, selezionare **&#65291; Aggiungi** per aggiungere una nuova entità.

1. Nella finestra di dialogo **Aggiungi un'entità** immettere il nome di entità `Location` e verificare che sia selezionata l'opzione **Appreso**. Selezionare quindi **Aggiungi entità**.

1. Dopo aver creato l'entità **Posizione** , tornare alla pagina **Etichettatura dati**.
1. Selezionare la finalità **GetTime** e immettere la nuova espressione di esempio seguente:

    `what time is it in London?`

1. Dopo avere aggiunto l'espressione, selezionare la parola **London** e nell'elenco a discesa visualizzato selezionare **Location** per indicare che "London" è un esempio di località.

1. Aggiungere un'altra espressione di esempio per la finalità **GetTime** :

    `Tell me the time in Paris?`

1. Dopo avere aggiunto l'espressione, selezionare la parola **Paris** ed eseguirne il mapping all'entità **Location**.

1. Aggiungere un'altra espressione di esempio per la finalità **GetTime** :

    `what's the time in New York?`

1. Dopo avere aggiunto l'espressione, selezionare le parole **New York** ed eseguirne il mapping all'entità **Location**.

1. Selezionare **Salva le modifiche** per salvare le nuove espressioni.

### Aggiungere un'entità *elenco*

In alcuni casi, i valori validi per un'entità possono essere limitati a un elenco di termini e sinonimi specifici. Questo consente all'app di identificare più facilmente le istanze dell'entità nelle espressioni.

1. In Language Studio tornare alla pagina **Definizione schema** e quindi, nella scheda **Entità**, selezionare **&#65291; Aggiungi** per aggiungere una nuova entità.

1. Nella finestra di dialogo **Aggiungi un'entità**, immettere il nome di entità `Weekday` e selezionare il tipo di entità **Elenco**. Selezionare quindi **Aggiungi entità**.

1. Nella pagina per l'entità **Giorno feriale**, nella sezione **Appreso**, verificare che **Non richiesto** sia selezionato. Quindi, nella sezione **Elenco**, selezionare **&#65291; Aggiungi un nuovo elenco**. Immettere quindi il valore e il sinonimo seguenti e selezionare **Salva**:

    | List key | sinonimi|
    |-------------------|---------|
    | `Sunday` | `Sun` |

    > **NOTA** Per immettere i campi del nuovo elenco, inserire il valore `Sunday` nel campo di testo, quindi fare clic sul campo in cui viene visualizzato "Digitare il valore e premere Invio...", immettere i sinonimi e premere INVIO.

1. Ripetere il passaggio precedente per aggiungere i componenti dell'elenco seguenti:

    | Valore | sinonimi|
    |-------------------|---------|
    | `Monday` | `Mon` |
    | `Tuesday` | `Tue, Tues` |
    | `Wednesday` | `Wed, Weds` |
    | `Thursday` | `Thur, Thurs` |
    | `Friday` | `Fri` |
    | `Saturday` | `Sat` |

1. Dopo aver aggiunto e salvato i valori dell'elenco, tornare alla pagina **Etichettatura dati**.
1. Selezionare la finalità **GetDate** e immettere la nuova espressione di esempio seguente:

    `what date was it on Saturday?`

1. Dopo avere aggiunto l'espressione, selezionare la parola ***Saturday*** e nell'elenco a discesa visualizzato selezionare **Weekday**.

1. Aggiungere un'altra espressione di esempio per la finalità **GetDate**:

    `what date will it be on Friday?`

1. Dopo avere aggiunto l'espressione, eseguire il mapping di **Friday** all'entità **Weekday**.

1. Aggiungere un'altra espressione di esempio per la finalità **GetDate**:

    `what will the date be on Thurs?`

1. Dopo avere aggiunto l'espressione, eseguire il mapping di **Thurs** all'entità **Weekday**.

1. selezionare **Salva le modifiche** per salvare le nuove espressioni.

### Aggiungere un'entità *predefinita*

Il servizio Lingua di Azure AI fornisce un set di entità *predefinite* comunemente usate nelle applicazioni di conversazione.

1. In Language Studio tornare alla pagina **Definizione schema** e quindi, nella scheda **Entità**, selezionare **&#65291; Aggiungi** per aggiungere una nuova entità.

1. Nella finestra di dialogo **Aggiungi un'entità**, immettere il nome di entità `Date` e selezionare il tipo di entità **Predefinita**. Selezionare quindi **Aggiungi entità**.

1. Nella pagina per l'entità **Data**, nella sezione **Appreso**, assicurarsi che **Non richiesto** sia selezionato. Quindi, nella sezione **Predefinita** selezionare **&#65291; Aggiungere un nuovo elemento predefinito**.

1. Nell'elenco **Seleziona predefinita**, selezionare **Data/Ora** e quindi fare clic su **Salva**.
1. Dopo aver aggiunto l'entità predefinita, tornare alla pagina **Etichettatura dati**
1. Selezionare la finalità **GetDay** e immettere la nuova espressione di esempio seguente:

    `what day was 01/01/1901?`

1. Dopo avere aggiunto l'espressione, selezionare ***01/01/1901*** e nell'elenco a discesa visualizzato selezionare **Date**.

1. Aggiungere un'altra espressione di esempio per la finalità**GetDay**:

    `what day will it be on Dec 31st 2099?`

1. Dopo avere aggiunto l'espressione, eseguire il mapping di **Dec 31st 2099** all'entità **Date**.

1. Selezionare **Salva le modifiche** per salvare le nuove espressioni.

### Ripetere il training del modello

Ora che è stato modificato lo schema, è necessario ripetere il training e il test del modello.

1. Nella pagina **Esegui training del modello** selezionare **Avvia un processo di training**.

1. Nella pagina **Avvia un processo di training**, selezionare **Sovrascrivi il modello esistente** e specificare il modello **Orologio**. Per eseguire il training del modello, selezionare **Esegui training**. Se richiesto, verificare di voler sovrascrivere il modello esistente.

1. Al termine del training, lo **stato** del processo verrà aggiornato in **Training succeeded**.

1. Selezionare la pagina **Prestazioni modello** e quindi selezionare il modello **Orologio** . Visualizzare le metriche di valutazione (*precisione*, *richiamo* e *punteggio F1*) e la *matrice di confusione* generata dalla valutazione eseguita durante il training (si noti che a causa del numero ridotto di espressioni di esempio, nei risultati potrebbero non essere incluse tutte le finalità).

1. Nella pagina **Distribuzione di un modello**, selezionare **Aggiungi distribuzione**.

1. Nella finestra di dialogo **Add deployment** selezionare **Override an existing deployment name** e quindi selezionare **production**.

1. Selezionare il modello **Orologio** nel campo **Modello** e quindi selezionare **Distribuisci** per distribuirlo. Questa operazione potrebbe richiedere tempo.

1. Quando il modello viene distribuito, nella pagina **Test di distribuzioni**, selezionare la distribuzione di **produzione** nel campo **Nome distribuzione** e quindi testarla con il testo seguente:

    `what's the time in Edinburgh?`

1. Esaminare il risultato restituito, che dovrebbe stimare la finalità **GetTime** e un'entità **Location** con il valore di testo "Edinburgh".

1. Provare a testare le espressioni seguenti:

    `what time is it in Tokyo?`

    `what date is it on Friday?`

    `what's the date on Weds?`

    `what day was 01/01/2020?`

    `what day will Mar 7th 2030 be?`

## Usare il modello da un'app client

In un progetto reale si perfezionerebbero in modo iterativo le finalità e le entità, ripetendo il training e i test finché non si è soddisfatti delle prestazioni di previsione. Dopo avere eseguito il test, se le prestazioni di stima sono soddisfacenti è possibile usare il modello in un'app client chiamando la relativa interfaccia REST.

### Preparare lo sviluppo di un'app in Visual Studio Code

Si svilupperà l'app di comprensione del linguaggio usando Visual Studio Code. I file di codice per l'app sono stati forniti in un repository GitHub.

> **Suggerimento**: se il repository **mslearn-ai-language** è già stato clonato, aprirlo in Visual Studio Code. In caso contrario, eseguire questi passaggi per clonarlo nell'ambiente di sviluppo.

1. Avviare Visual Studio Code.
2. Aprire il riquadro comandi (MAIUSC+CTRL+P) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/mslearn-ai-language` in una cartella locale. Non è importante usare una cartella specifica.
3. Dopo la clonazione del repository, aprire la cartella in Visual Studio Code.

    > **Nota**: Se Visual Studio Code visualizza un messaggio popup per chiedere se si considera attendibile il codice che si apre, fare clic sull'opzione **Sì, considero attendibili gli autori** nel popup.

4. Attendere il completamento dell'installazione di file aggiuntivi per supportare i progetti in codice C# nel repository.

    > **Nota**: se viene richiesto di aggiungere gli asset necessari per la compilazione e il debug, selezionare **Non adesso**.

### Configurare l'applicazione

Sono state fornite applicazioni sia per C# che per Python, oltre a un file di testo di esempio che verrà usato per testare il riepilogo. Entrambe le app presentano la stessa funzionalità. Prima di tutto, verranno completate alcune parti chiave dell'applicazione per abilitarla all'uso della risorsa Lingua di Azure AI.

1. Nel riquadro **Esplora risorse** di Visual Studio Code, passare alla cartella **Labfiles/03-language** ed espandere la cartella **CSharp** o **Python**, in base alle preferenze della lingua e alla cartella **clock-client** che contiene. Ogni cartella contiene i file specifici della lingua per un'app in cui si intende integrare la funzionalità di risposta alle domande di Lingua di Azure AI.
2. Fare clic con il pulsante destro del mouse sulla cartella **clock-client** contenente i file di codice e aprire un terminale integrato. Installare quindi il pacchetto l'SDK di comprensione del linguaggio di conversazione di Lingua di Azure AI eseguendo il comando appropriato per il linguaggio scelto:

    **C#:**

    ```
    dotnet add package Azure.AI.Language.Conversations --version 1.1.0
    ```

    **Python**:

    ```
    pip install azure-ai-language-conversations
    ```

3. Nel riquadro **Esplora risorse**, aprire il file di configurazione per la lingua preferita nella cartella **clock-client**

    - **C#**: appsettings.json
    - **Python**: .env
    
4. Aggiornare i valori di configurazione per includere l'**endpoint** e una **chiave** dalla risorsa Lingua di Azure creata (disponibile nella pagina **Chiavi ed endpoint** per la risorsa di Lingua di Azure AI).
5. Salvare il file di configurazione.

### Aggiungere codice all'applicazione

È ora possibile aggiungere il codice necessario per importare le librerie SDK necessarie, stabilire una connessione autenticata al progetto distribuito e inviare domande.

1. Si noti che la cartella **clock-client** contiene un file di codice per l'applicazione client:

    - **C#**: Program.cs
    - **Python**: clock-client.py

    Aprire il file di codice e nella parte superiore, sotto i riferimenti agli spazi dei nomi esistenti, individuare il commento **Import namespaces**. In questo commento aggiungere quindi il codice seguente specifico per la lingua per importare gli spazi dei nomi necessari per usare Text Analytics SDK:

    **C#**: Programs.cs

    ```c#
    // import namespaces
    using Azure;
    using Azure.AI.Language.Conversations;
    ```

    **Python**: clock-client.py

    ```python
    # Import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.language.conversations import ConversationAnalysisClient
    ```

1. Nella funzione **Main** si noti che il codice per caricare l'endpoint di previsione e la chiave dal file di configurazione è già stato fornito. Trovare quindi il commento **Create a client for the Language service model** e aggiungere il codice seguente per creare un client di previsione per l'app Language Service:

    **C#**: Programs.cs

    ```c#
    // Create a client for the Language service model
    Uri endpoint = new Uri(predictionEndpoint);
    AzureKeyCredential credential = new AzureKeyCredential(predictionKey);

    ConversationAnalysisClient client = new ConversationAnalysisClient(endpoint, credential);
    ```

    **Python**: clock-client.py

    ```python
    # Create a client for the Language service model
    client = ConversationAnalysisClient(
        ls_prediction_endpoint, AzureKeyCredential(ls_prediction_key))
    ```

1. Si noti che il codice nella funzione **Main** richiede input da parte dell'utente finché l'utente non immette "quit". All'interno di questo ciclo trovare il commento **Call the Language service model to get intent and entities** e aggiungere il codice seguente:

    **C#**: Programs.cs

    ```c#
    // Call the Language service model to get intent and entities
    var projectName = "Clock";
    var deploymentName = "production";
    var data = new
    {
        analysisInput = new
        {
            conversationItem = new
            {
                text = userText,
                id = "1",
                participantId = "1",
            }
        },
        parameters = new
        {
            projectName,
            deploymentName,
            // Use Utf16CodeUnit for strings in .NET.
            stringIndexType = "Utf16CodeUnit",
        },
        kind = "Conversation",
    };
    // Send request
    Response response = await client.AnalyzeConversationAsync(RequestContent.Create(data));
    dynamic conversationalTaskResult = response.Content.ToDynamicFromJson(JsonPropertyNames.CamelCase);
    dynamic conversationPrediction = conversationalTaskResult.Result.Prediction;   
    var options = new JsonSerializerOptions { WriteIndented = true };
    Console.WriteLine(JsonSerializer.Serialize(conversationalTaskResult, options));
    Console.WriteLine("--------------------\n");
    Console.WriteLine(userText);
    var topIntent = "";
    if (conversationPrediction.Intents[0].ConfidenceScore > 0.5)
    {
        topIntent = conversationPrediction.TopIntent;
    }
    ```

    **Python**: clock-client.py

    ```python
    # Call the Language service model to get intent and entities
    cls_project = 'Clock'
    deployment_slot = 'production'

    with client:
        query = userText
        result = client.analyze_conversation(
            task={
                "kind": "Conversation",
                "analysisInput": {
                    "conversationItem": {
                        "participantId": "1",
                        "id": "1",
                        "modality": "text",
                        "language": "en",
                        "text": query
                    },
                    "isLoggingEnabled": False
                },
                "parameters": {
                    "projectName": cls_project,
                    "deploymentName": deployment_slot,
                    "verbose": True
                }
            }
        )

    top_intent = result["result"]["prediction"]["topIntent"]
    entities = result["result"]["prediction"]["entities"]

    print("view top intent:")
    print("\ttop intent: {}".format(result["result"]["prediction"]["topIntent"]))
    print("\tcategory: {}".format(result["result"]["prediction"]["intents"][0]["category"]))
    print("\tconfidence score: {}\n".format(result["result"]["prediction"]["intents"][0]["confidenceScore"]))

    print("view entities:")
    for entity in entities:
        print("\tcategory: {}".format(entity["category"]))
        print("\ttext: {}".format(entity["text"]))
        print("\tconfidence score: {}".format(entity["confidenceScore"]))

    print("query: {}".format(result["result"]["query"]))
    ```

    La chiamata al modello di servizio di linguaggio restituisce una previsione o un risultato, che include la finalità principale (più probabile) e tutte le entità rilevate nell'espressione di input. L'applicazione client deve ora usare la previsione per determinare ed eseguire l'azione appropriata.

1. Trovare il commento **Apply the appropriate action** e aggiungere il codice seguente, che verifica la presenza di finalità supportate dall'applicazione (**GetTime**, **GetDate** e **GetDay**) e determina se sono state rilevate entità pertinenti, prima di chiamare una funzione esistente per produrre una risposta appropriata.

    **C#**: Programs.cs

    ```c#
    // Apply the appropriate action
    switch (topIntent)
    {
        case "GetTime":
            var location = "local";           
            // Check for a location entity
            foreach (dynamic entity in conversationPrediction.Entities)
            {
                if (entity.Category == "Location")
                {
                    //Console.WriteLine($"Location Confidence: {entity.ConfidenceScore}");
                    location = entity.Text;
                }
            }
            // Get the time for the specified location
            string timeResponse = GetTime(location);
            Console.WriteLine(timeResponse);
            break;
        case "GetDay":
            var date = DateTime.Today.ToShortDateString();            
            // Check for a Date entity
            foreach (dynamic entity in conversationPrediction.Entities)
            {
                if (entity.Category == "Date")
                {
                    //Console.WriteLine($"Location Confidence: {entity.ConfidenceScore}");
                    date = entity.Text;
                }
            }            
            // Get the day for the specified date
            string dayResponse = GetDay(date);
            Console.WriteLine(dayResponse);
            break;
        case "GetDate":
            var day = DateTime.Today.DayOfWeek.ToString();
            // Check for entities            
            // Check for a Weekday entity
            foreach (dynamic entity in conversationPrediction.Entities)
            {
                if (entity.Category == "Weekday")
                {
                    //Console.WriteLine($"Location Confidence: {entity.ConfidenceScore}");
                    day = entity.Text;
                }
            }          
            // Get the date for the specified day
            string dateResponse = GetDate(day);
            Console.WriteLine(dateResponse);
            break;
        default:
            // Some other intent (for example, "None") was predicted
            Console.WriteLine("Try asking me for the time, the day, or the date.");
            break;
    }
    ```

    **Python**: clock-client.py

    ```python
    # Apply the appropriate action
    if top_intent == 'GetTime':
        location = 'local'
        # Check for entities
        if len(entities) > 0:
            # Check for a location entity
            for entity in entities:
                if 'Location' == entity["category"]:
                    # ML entities are strings, get the first one
                    location = entity["text"]
        # Get the time for the specified location
        print(GetTime(location))

    elif top_intent == 'GetDay':
        date_string = date.today().strftime("%m/%d/%Y")
        # Check for entities
        if len(entities) > 0:
            # Check for a Date entity
            for entity in entities:
                if 'Date' == entity["category"]:
                    # Regex entities are strings, get the first one
                    date_string = entity["text"]
        # Get the day for the specified date
        print(GetDay(date_string))

    elif top_intent == 'GetDate':
        day = 'today'
        # Check for entities
        if len(entities) > 0:
            # Check for a Weekday entity
            for entity in entities:
                if 'Weekday' == entity["category"]:
                # List entities are lists
                    day = entity["text"]
        # Get the date for the specified day
        print(GetDate(day))

    else:
        # Some other intent (for example, "None") was predicted
        print('Try asking me for the time, the day, or the date.')
    ```

1. Salvare le modifiche e tornare al terminale integrato per la cartella **clock-client**, quindi immettere il comando seguente per eseguire il programma:

    - **C#**: `dotnet run`
    - **Python**: `python clock-client.py`

    > **Suggerimento**: È possibile usare l'icona **Ingrandisci dimensioni pannello** (**^**) nella barra degli strumenti del terminale per visualizzare altro testo della console.

1. Quando richiesto, immettere espressioni per testare l'applicazione. Ad esempio, provare:

    *Ciao*

    *What time is it?*

    *What's the time in London?*

    *What's the date?*

    *What date is Sunday?*

    *What day is it?*

    *What day is 01/01/2025?*

    > **Nota**: la logica nell'applicazione è volutamente semplice e presenta una serie di limitazioni. Ad esempio, per il recupero dell'ora è supportato solo un set limitato di città e l'ora legale viene ignorata. Lo scopo è vedere un esempio di un modello tipico di uso del servizio di linguaggio in cui l'applicazione deve:
    >   1. Connettersi a un endpoint di previsione.
    >   2. Inviare un'espressione per ottenere una previsione.
    >   3. Implementare logica per rispondere in modo appropriato alla finalità prevista e alle entità.

1. Al termine del test, immettere *quit*.

## Pulire le risorse

Se si è terminato di esplorare il servizio Lingua di Azure AI, è possibile eliminare le risorse create in questo esercizio. In tal caso, eseguire la procedura seguente:

1. Aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.
2. Passare alla risorsa Lingua di Azure AI creata in questo lab.
3. Nella pagina della risorsa selezionare **Elimina** e seguire le istruzioni per eliminare la risorsa.

## Ulteriori informazioni

Per altre informazioni sulla comprensione del linguaggio di conversazione in Lingua di Azure AI, vedere la [documentazione di Lingua di Azure AI](https://learn.microsoft.com/azure/ai-services/language-service/conversational-language-understanding/overview).
