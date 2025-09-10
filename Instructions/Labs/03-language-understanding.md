---
lab:
  title: Creare un modello di comprensione del linguaggio con il servizio Lingua di Azure AI (anteprima)
  description: 'Creare un modello di comprensione del linguaggio personalizzato per interpretare l''input, stimare la finalità e identificare le entità.'
---

# Creare un modello di comprensione del linguaggio con il servizio Lingua

Il servizio Lingua di Azure AI consente di definire un modello di *comprensione del linguaggio di conversazione* che le applicazioni possono usare per interpretare le *espressioni* in linguaggio naturale degli utenti (input di testo o vocali), stimare la *finalità* degli utenti, ossia cosa vogliono ottenere, e identificare le *entità* a cui applicare la finalità.

Un modello linguistico per conversazioni per un'applicazione orologio potrebbe ad esempio elaborare un input simile al seguente:

*What is the time in London?*

Questo tipo di input è un esempio di *espressione* (qualcosa che un utente potrebbe dire o digitare) la cui *finalità* è recuperare l'orario in una località specifica (un'*entità *), in questo caso Londra.

> **NOTA**: l'attività di un modello linguistico per conversazioni è stimare la finalità dell'utente e identificare le entità a cui si applica. <u>Non</u> è compito di un modello linguistico per conversazioni eseguire effettivamente le azioni necessarie per soddisfare la finalità. Ad esempio, un'applicazione orologio può usare un modello linguistico per conversazioni per comprendere che l'utente vuole conoscere l'ora di Londra, ma è l'applicazione client stessa che deve quindi implementare la logica per determinare l'ora corretta e presentarla all'utente.

In questo esercizio si userà il servizio Lingua di Azure AI per creare un modello di comprensione del linguaggio di conversazione e si userà l'SDK Python per implementare un'app client che lo usa.

Anche se questo esercizio è basato su Python, è possibile sviluppare applicazioni di comprensione delle conversazioni usando più SDK specifici del linguaggio, tra cui:

- [Libreria client di conversazioni di Azure AI per Python](https://pypi.org/project/azure-ai-language-conversations/)
- [Libreria client di conversazioni di Azure AI per .NET](https://www.nuget.org/packages/Azure.AI.Language.Conversations)
- [Libreria client di conversazioni di Azure AI per JavaScript](https://www.npmjs.com/package/@azure/ai-language-conversations)

Questo esercizio richiede circa **35** minuti.

## Effettuare il provisioning di una risorsa *Lingua di Azure AI*

Se non ne è già disponibile una nella sottoscrizione, è necessario effettuare il provisioning di una risorsa del servizio **Lingua di Azure AI** nella sottoscrizione di Azure.

1. Aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.
1. Selezionare **Crea una risorsa**.
1. Nel campo di ricerca, cercare **Servizio di linguaggio**. Quindi, nei risultati selezionare **Crea** in **Servizio linguistico**.
1. Effettuare il provisioning della risorsa usando le impostazioni seguenti:
    - **Sottoscrizione**: *la sottoscrizione di Azure usata*.
    - **Gruppo di risorse**: *Selezionare o creare un gruppo di risorse*.
    - **Area**: *scegliere tra una delle aree seguenti*\*
        - Australia orientale
        - India centrale
        - Cina orientale 2
        - Stati Uniti orientali
        - Stati Uniti orientali 2
        - Europa settentrionale
        - Stati Uniti centro-meridionali
        - Svizzera settentrionale
        - Regno Unito meridionale
        - Europa occidentale
        - Stati Uniti occidentali 2
        - Stati Uniti occidentali 3
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

### Preparare lo sviluppo di un'app in Cloud Shell

Si svilupperà l'app di comprensione del linguaggio usando Cloud Shell nel portale di Azure. I file di codice per l'app sono stati forniti in un repository GitHub.

1. Nel portale di Azure, usare il pulsante **[\>_]** a destra della barra di ricerca nella parte superiore della pagina per creare una nuova Cloud Shell nel portale di Azure, selezionando un ambiente ***PowerShell***. Cloud Shell fornisce un'interfaccia della riga di comando in un riquadro nella parte inferiore del portale di Azure.

    > **Nota**: se in precedenza è stata creata una sessione Cloud Shell che usa un ambiente *Bash*, passare a ***PowerShell***.

1. Nella barra degli strumenti di Cloud Shell scegliere **Vai alla versione classica** dal menu **Impostazioni**. Questa operazione è necessaria per usare l'editor di codice.

    **<font color="red">Verificare di passare alla versione classica di Cloud Shell prima di continuare.</font>**

1. Nel riquadro PowerShell immettere i comandi seguenti per clonare il repository GitHub per questo esercizio:

    ```
   rm -r mslearn-ai-language -f
   git clone https://github.com/microsoftlearning/mslearn-ai-language
    ```

    > **Suggerimento**: quando si incollano i comandi in CloudShell, l'ouput può richiedere una grande quantità di buffer dello schermo. È possibile cancellare la schermata immettendo il `cls` comando per rendere più semplice concentrarsi su ogni attività.

1. Dopo aver clonato il repository, passare alla cartella contenente i file di codice dell'applicazione:  

    ```
   cd mslearn-ai-language/Labfiles/03-language/Python/clock-client
    ```

### Configurare l'applicazione

1. Nel riquadro della riga di comando eseguire il comando seguente per visualizzare i file di codice nella cartella **clock-client**:

    ```
   ls -a -l
    ```

    I file includono un file di configurazione (con estensione **env**) e un file di codice (**clock-client.py**).

1. Creare un ambiente virtuale Python e installare il pacchetto SDK di conversazioni di Lingua di Azure AI e altri pacchetti necessari eseguendo il comando seguente:

    ```
   python -m venv labenv
    ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-language-conversations==1.1.0
    ```
1. Immettere il comando seguente per modificare il file di configurazione:

    ```
   code .env
    ```

    Il file viene aperto in un editor di codice.

1. Aggiornare i valori di configurazione per includere l'**endpoint** e una **chiave** dalla risorsa Lingua di Azure creata (disponibile nella pagina **Chiavi ed endpoint** per la risorsa Lingua di Azure AI nel portale di Azure).
1. Dopo aver sostituito i segnaposto con l'editor di codice, usare il comando **CTRL+S** o **Fare clic con il pulsante destro del mouse > Salva** per salvare le modifiche e quindi usare il comando **CTRL+Q** o **Fare clic con il pulsante destro del mouse > Esci** per chiudere l'editor di codice mantenendo aperta la riga di comando di Cloud Shell.

### Aggiungere codice all'applicazione

1. Immettere il comando seguente per modificare il file di codice dell'applicazione:

    ```
   code clock-client.py
    ```

1. Esaminare il codice esistente. Si aggiungerà il codice per usare SDK di conversazioni di Lingua di Azure AI.

    > **Suggerimento**: Quando si aggiunge codice al file di codice, assicurarsi di mantenere il rientro corretto.

1. Nella parte superiore del file di codice, sotto i riferimenti allo spazio dei nomi esistenti, trovare il commento **Importa spazi dei nomi** e aggiungere il codice seguente per importare gli spazi dei nomi necessari per usare SDK di conversazioni di Lingua di Azure AI:

    ```python
   # Import namespaces
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.language.conversations import ConversationAnalysisClient
    ```

1. Nella funzione **main** si noti che il codice per caricare l'endpoint di previsione e la chiave dal file di configurazione è già stato fornito. Trovare quindi il commento **Creare a client per il modello del servizio Lingua** e aggiungere il codice seguente per creare un client di analisi delle conversazioni per il servizio Lingua di Azure AI:

    ```python
   # Create a client for the Language service model
   client = ConversationAnalysisClient(
        ls_prediction_endpoint, AzureKeyCredential(ls_prediction_key))
    ```

1. Si noti che il codice nella funzione **main** richiede input da parte dell'utente finché l'utente non immette "quit". All'interno di questo ciclo trovare il commento **Call the Language service model to get intent and entities** e aggiungere il codice seguente:

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

    La chiamata al modello di comprensione delle conversazioni restituisce una previsione o un risultato, che include la finalità principale (più probabile) e tutte le entità rilevate nell'espressione di input. L'applicazione client deve ora usare la previsione per determinare ed eseguire l'azione appropriata.

1. Trovare il commento **Apply the appropriate action** e aggiungere il codice seguente, che verifica la presenza di finalità supportate dall'applicazione (**GetTime**, **GetDate** e **GetDay**) e determina se sono state rilevate entità pertinenti, prima di chiamare una funzione esistente per produrre una risposta appropriata.

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

1. Salvare le modifiche (CTRL+S), quindi immettere il comando seguente per eseguire il programma (ingrandire il riquadro Cloud Shell e ridimensionare i pannelli per visualizzare altro testo nel riquadro della riga di comando):

    ```
   python clock-client.py
    ```

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

1. Chiudere il riquadro Azure Cloud Shell.
1. Nel portale di Azure passare alla risorsa Lingua di Azure AI creata in questo lab.
1. Nella pagina della risorsa, selezionare **Elimina** e seguire le istruzioni per eliminare la risorsa.

## Ulteriori informazioni

Per altre informazioni sulla comprensione del linguaggio di conversazione in Lingua di Azure AI, vedere la [documentazione di Lingua di Azure AI](https://learn.microsoft.com/azure/ai-services/language-service/conversational-language-understanding/overview).
