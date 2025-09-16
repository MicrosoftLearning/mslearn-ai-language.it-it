---
lab:
  title: Estrarre entità personalizzate
  description: Eseguire il training di un modello per estrarre entità personalizzate dall'input di testo con Lingua di Azure AI.
---

# Estrarre entità personalizzate

Oltre ad altre funzionalità di elaborazione del linguaggio naturale, il servizio Lingua di Azure AI consente di definire entità personalizzate ed estrarre istanze di tali entità dal testo.

Per testare l'estrazione di entità personalizzate, verrà creato un modello e ne verrà eseguito il training tramite lo studio Lingua di Azure AI, quindi verrà usata un'applicazione Python per testarlo.

Anche se questo esercizio è basato su Python, è possibile sviluppare applicazioni di classificazione testo usando più SDK specifici del linguaggio, tra cui:

- [Libreria client di Analisi del testo di Azure AI per Python](https://pypi.org/project/azure-ai-textanalytics/)
- [Libreria client di Analisi del testo di Azure AI per .NET](https://www.nuget.org/packages/Azure.AI.TextAnalytics)
- [Libreria client di Analisi del testo di Azure AI per JavaScript](https://www.npmjs.com/package/@azure/ai-text-analytics)

Questo esercizio richiede circa **35** minuti.

## Effettuare il provisioning di una risorsa *Lingua di Azure AI*

Se non ne è già disponibile una nella sottoscrizione, è necessario effettuare il provisioning di una risorsa del **servizio Lingua di Azure AI**. Inoltre, per usare la classificazione personalizzata del testo, è necessario abilitare la funzionalità **Estrazione e classificazione personalizzata del testo**.

1. In un browser, aprire il portale di Azure all'indirizzo `https://portal.azure.com` e accedere con il proprio account Microsoft.
1. Selezionare il pulsante **Crea una risorsa**, cercare *Lingua*, quindi creare una risorsa del servizio **Lingua**. Una volta nella pagina per *Selezionare funzionalità aggiuntive*, selezionare la funzionalità personalizzata contenente **l'estrazione di riconoscimento entità denominata personalizzata**. Creare la risorsa con le impostazioni seguenti:
    - **Sottoscrizione**: *la sottoscrizione di Azure usata*
    - **Gruppo di risorse**: *Selezionare o creare un gruppo di risorse*
    - **Area**: *scegliere tra una delle aree seguenti*\*
        - Australia orientale
        - India centrale
        - Stati Uniti orientali
        - Stati Uniti orientali 2
        - Europa settentrionale
        - Stati Uniti centro-meridionali
        - Svizzera settentrionale
        - Regno Unito meridionale
        - Europa occidentale
        - Stati Uniti occidentali 2
        - Stati Uniti occidentali 3
    - **Nome**: *immettere un nome univoco*
    - **Piano tariffario**: Selezionare **F0** (*gratuito*) o **S** (*standard*) se F non è disponibile.
    - **Account di archiviazione**: Nuovo account di archiviazione:
      - **Nome account di archiviazione**: *immettere un nome univoco*.
      - **Tipo di account di archiviazione: ** Archiviazione con ridondanza locale Standard
    - **Avviso per intelligenza artificiale responsabile**: opzione selezionata

1. Selezionare **Rivedi e crea**, quindi **Crea** per effettuare il provisioning della risorsa.
1. Attendere il completamento della distribuzione e quindi passare alla risorsa distribuita.
1. Visualizzare la pagina **Chiavi ed endpoint**. Le informazioni contenute in questa pagina saranno necessarie più avanti nell'esercizio.

## Configurare l'accesso basato sui ruoli per l'utente

> **NOTA**: se si ignora questo passaggio, si riscontrerà un errore 403 quando si tenta di connettersi al progetto personalizzato. È importante che l'utente corrente abbia questo ruolo per accedere ai dati BLOB dell'account di archiviazione, anche se si è il proprietario dell'account di archiviazione.

1. Accedere alla pagina dell’account di archiviazione nel portale di Azure.
2. Selezionare **Controllo di accesso (IAM)** nel menu di spostamento a sinistra.
3. Selezionare **Aggiungi** per aggiungere assegnazioni di ruolo e scegliere il ruolo di **collaboratore ai dati del BLOB di archiviazione** nell'account di archiviazione.
4. In **Assegna accesso a**, selezionare **Utente, gruppo o entità servizio**.
5. Selezionare **Selezionare i membri**.
6. Selezionare l'utente. È possibile cercare nomi utente nel campo **Seleziona**.

## Caricare annunci di esempio

Dopo aver creato il servizio Lingua di Azure AI e l'account di archiviazione, sarà necessario caricare annunci di esempio per eseguire il training del modello in un secondo momento.

1. In una nuova scheda del browser, scarica gli annunci classificati di esempio da `https://aka.ms/entity-extraction-ads` ed estrai i file in una cartella di tua scelta.

2. Nel portale di Azure passare all'account di archiviazione creato e selezionarlo.

3. Nell'account di archiviazione, selezionare **Configurazione **, che si trova sotto **Impostazioni** e abilitare l'opzione **Consenti accesso anonimo BLOB** e quindi selezionare **Salva**.

4. Selezionare **Contenitori** dal menu a sinistra, che si trova sotto **Archiviazione dati**. Nella schermata visualizzata selezionare **+ Contenitore**. Assegnare al contenitore il nome `classifieds`, quindi impostare **Livello di accesso anonimo** su **Contenitore (accesso in lettura anonimo per contenitori e BLOB)**.

    > **NOTA**: quando si configura un account di archiviazione per una soluzione reale, prestare attenzione ad assegnare il livello di accesso appropriato. Per altre informazioni su ogni livello di accesso, vedere la [documentazione su Archiviazione di Azure](https://learn.microsoft.com/azure/storage/blobs/anonymous-read-access-configure).

5. Dopo aver creato il contenitore, selezionarlo e fare clic sul pulsante **Carica** e caricare gli annunci di esempio scaricati.

## Creare un progetto di Riconoscimento entità denominata personalizzata

A questo punto è possibile creare un progetto di riconoscimento di entità denominato personalizzato. Questo progetto offre un luogo di lavoro per compilare il modello, eseguirne il training e quindi distribuirlo.

> **NOTA**: È anche possibile creare, compilare, eseguire il training e distribuire il modello tramite l'API REST.

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
    4. Nella parte superiore della pagina fare clic su **Language Studio** per tornare alla pagina iniziale di Language Studio.

1. Nella parte superiore del portale, nel menu **Crea nuovo** selezionare **Riconoscimento entità denominata personalizzata**.

1. Creare un nuovo progetto con le seguenti impostazioni:
    - **Archiviazione Connessione**: *Questo valore è probabilmente già compilato. Modificarlo nell'account di archiviazione, se non lo è già*
    - **Informazioni di base:**
    - **Nome**: `CustomEntityLab`
        - **Lingua principale testo**: Inglese (US)
        - **Il set di dati include documenti che non sono nella stessa lingua?** : *No*
        - **Descrizione**: `Custom entities in classified ads`
    - **Contenitore**:
        - **Contenitore dell'archivio BLOB**: classificazioni
        - **I file sono etichettati con classi?**: no, è necessario assegnare etichette ai file nell'ambito di questo progetto

> **Suggerimento**: se viene visualizzato un errore relativo alla mancata autorizzazione per eseguire questa operazione, è necessario aggiungere un'assegnazione di ruolo. Per risolvere questo problema, per l'utente che esegue il lab, è stato aggiunto il ruolo "Collaboratore ai dati del BLOB di archiviazione" nell'account di archiviazione. Per informazioni dettagliate, vedere la [pagina della documentazione](https://learn.microsoft.com/azure/ai-services/language-service/custom-named-entity-recognition/how-to/create-project?tabs=portal%2Clanguage-studio#enable-identity-management-for-your-resource)

## Assegnare etichette ai dati

Dopo aver creato il progetto, è necessario assegnare etichette ai dati per eseguire il training del modello al fine di identificare le entità.

1. Se la pagina **Etichettatura dati** non è già aperta, nel riquadro a sinistra selezionare **Etichettatura dati**. Viene visualizzato un elenco dei file caricati nell'account di archiviazione.
1. Sul lato destro, nel riquadro **Attività**, selezionare**Aggiungi entità** e aggiungere una nuova entità denominata `ItemForSale`.
1.  Ripetere il passaggio precedente per creare le entità seguenti:
    - `Price`
    - `Location`
1. Dopo aver creato le tre entità, selezionare **Ad 1.txt** in modo da poterla leggere.
1. In *Ad 1.txt*: 
    1. Evidenziare il *pancale di legna da ardere* e selezionare l'entità **ItemForSale**.
    1. Evidenziare il testo *Denver, CO* e selezionare l'entità **Location**.
    1. Evidenziare il testo *$90* e selezionare l'entità **Price**.
1. Nel riquadro **Attività**, notare che questo documento verrà aggiunto al set di dati per il training del modello.
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

## Preparare lo sviluppo di un'app in Cloud Shell

Per testare le funzionalità di estrazione di entità personalizzate del servizio Lingua di Azure AI, si svilupperà una semplice applicazione console in Azure Cloud Shell.

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
    ```

1. After the repo has been cloned, navigate to the folder containing the application code files:  

    ```
    cd mslearn-ai-language/Labfiles/05-custom-entity-recognition/Python/custom-entities
    ```

## Configure your application

1. In the command line pane, run the following command to view the code files in the **custom-entities** folder:

    ```
   ls -a -l
    ```

    The files include a configuration file (**.env**) and a code file (**custom-entities.py**). The text your application will analyze is in the **ads** subfolder.

1. Create a Python virtual environment and install the Azure AI Language Text Analytics SDK package and other required packages by running the following command:

    ```
   python -m venv labenv ./labenv/bin/Activate.ps1 pip install -r requirements.txt azure-ai-textanalytics==5.3.0
    ```

1. Enter the following command to edit the application configuration file:

    ```
   code .env
    ```

    The file is opened in a code editor.

1. Update the configuration values to include the  **endpoint** and a **key** from the Azure Language resource you created (available on the **Keys and Endpoint** page for your Azure AI Language resource in the Azure portal).The file should already contain the project and deployment names for your custom entity extraction model.
1. After you've replaced the placeholders, within the code editor, use the **CTRL+S** command or **Right-click > Save** to save your changes and then use the **CTRL+Q** command or **Right-click > Quit** to close the code editor while keeping the cloud shell command line open.

## Add code to extract entities

1. Enter the following command to edit the application code file:

    ```
    code custom-entities.py
    ```

1. Review the existing code. You will add code to work with the AI Language Text Analytics SDK.

    > **Tip**: As you add code to the code file, be sure to maintain the correct indentation.

1. At the top of the code file, under the existing namespace references, find the comment **Import namespaces** and add the following code to import the namespaces you will need to use the Text Analytics SDK:

    ```python
   # import namespaces
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. Nella funzione **main** notare che il codice per caricare l'endpoint e la chiave del servizio Lingua di Azure AI e i nomi del progetto e della distribuzione indicati nel file di configurazione è già stato fornito. Individuare quindi il commento **Creare un client usando l’endpoint e la chiave** e aggiungere il codice seguente per creare un client di analisi del testo:

    ```Python
   # Create client using endpoint and key
   credential = AzureKeyCredential(ai_key)
   ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

1. Si noti che il codice esistente legge tutti i file nella cartella **ads** e crea un elenco del loro contenuto. Individuare quindi il commento **Estrarre le entità** e aggiungere il codice seguente:

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

1. Salvare le modifiche (CTRL+S), quindi immettere il comando seguente per eseguire il programma (ingrandire il riquadro Cloud Shell e ridimensionare i pannelli per visualizzare altro testo nel riquadro della riga di comando):

    ```
   python custom-entities.py
    ```

1. Osservare l'output. L'applicazione deve elencare i dettagli delle entità presenti in ogni file di testo.

## Eseguire la pulizia

Quando il progetto non è più necessario, è possibile eliminarlo dalla pagina **Progetti** in Language Studio. È anche possibile rimuovere il servizio Lingua di Azure AI e l'account di archiviazione associato nel [portale di Azure](https://portal.azure.com).
