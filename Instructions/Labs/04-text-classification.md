---
lab:
  title: Classificazione personalizzata del testo
  description: Applicare classificazioni personalizzate all'input di testo usando Lingua di Azure AI.
---

# Classificazione personalizzata del testo

Lingua di Azure AI offre diverse funzionalità di elaborazione del linguaggio naturale, tra cui l'identificazione delle frasi chiave, il riepilogo del testo e l'analisi del sentiment. Il servizio Lingua offre anche funzionalità personalizzate, ad esempio la risposta alle domande personalizzate e la classificazione personalizzata del testo.

Per testare la classificazione personalizzata del testo del servizio Lingua di Azure AI, si configurerà il modello usando Language Studio e quindi si userà un'applicazione Python per testarlo.

Anche se questo esercizio è basato su Python, è possibile sviluppare applicazioni di classificazione testo usando più SDK specifici del linguaggio, tra cui:

- [Libreria client di Analisi del testo di Azure AI per Python](https://pypi.org/project/azure-ai-textanalytics/)
- [Libreria client di Analisi del testo di Azure AI per .NET](https://www.nuget.org/packages/Azure.AI.TextAnalytics)
- [Libreria client di Analisi del testo di Azure AI per JavaScript](https://www.npmjs.com/package/@azure/ai-text-analytics)

Questo esercizio richiede circa **35** minuti.

## Effettuare il provisioning di una risorsa *Lingua di Azure AI*

Se non ne è già disponibile una nella sottoscrizione, è necessario effettuare il provisioning di una risorsa del **servizio Lingua di Azure AI**. Inoltre, per usare la classificazione personalizzata del testo, è necessario abilitare la funzionalità **Estrazione e classificazione personalizzata del testo**.

1. Aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.
1. Selezionare **Crea una risorsa**.
1. Nel campo di ricerca, cercare **Servizio di linguaggio**. Quindi, nei risultati selezionare **Crea** in **Servizio linguistico**.
1. Selezionare la casella che include **Classificazione personalizzata del testo**. Selezionare quindi **Continua a creare la risorsa**.
1. Creare una risorsa con le impostazioni seguenti:
    - **Sottoscrizione**: *la sottoscrizione di Azure usata*.
    - **Gruppo di risorse**: *Selezionare o creare un gruppo di risorse*.
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
    - **Nome**: *immettere un nome univoco*.
    - **Piano tariffario**: Selezionare **F0** (*gratuito*) o **S** (*standard*) se F non è disponibile.
    - **Account di archiviazione**: Nuovo account di archiviazione
      - **Nome account di archiviazione**: *immettere un nome univoco*.
      - **Tipo di account di archiviazione: ** Archiviazione con ridondanza locale Standard
    - **Avviso per intelligenza artificiale responsabile**: opzione selezionata

1. Selezionare **Rivedi e crea**, quindi **Crea** per effettuare il provisioning della risorsa.
1. Attendere il completamento della distribuzione e quindi passare al gruppo di risorse.
1. Trovare l'account di archiviazione creato, selezionarlo e verificare che il _Tipo di account_ sia **StorageV2**. Se è v1, aggiornare il tipo di account di archiviazione nella pagina della risorsa.

## Configurare l'accesso basato sui ruoli per l'utente

> **NOTA**: Se si ignora questo passaggio, si riscontrerà un errore 403 quando si tenta di connettersi al progetto personalizzato. È importante che l'utente corrente abbia questo ruolo per accedere ai dati BLOB dell'account di archiviazione, anche se si è il proprietario dell'account di archiviazione.

1. Accedere alla pagina dell’account di archiviazione nel portale di Azure.
2. Selezionare **Controllo di accesso (IAM)** nel menu di spostamento a sinistra.
3. Selezionare **Aggiungi** per aggiungere assegnazioni di ruolo e scegliere il ruolo **Proprietario dei dati dei BLOB di archiviazione** nell'account di archiviazione.
4. In **Assegna accesso a**, selezionare **Utente, gruppo o entità servizio**.
5. Selezionare **Selezionare i membri**.
6. Selezionare l'utente. È possibile cercare nomi utente nel campo **Seleziona**.

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

## Preparare lo sviluppo di un'app in Cloud Shell

Per testare le funzionalità di classificazione personalizzata del testo del servizio Lingua di Azure AI, si svilupperà una semplice applicazione console in Azure Cloud Shell.

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
   cd mslearn-ai-language/Labfiles/04-text-classification/Python/classify-text
    ```

## Configurare l'applicazione

1. Nel riquadro della riga di comando eseguire il comando seguente per visualizzare i file di codice nella cartella **classify-text**:

    ```
   ls -a -l
    ```

    I file includono un file di configurazione (con estensione **env**) e un file di codice (**classify-text.py**). Il testo che verrà analizzato dall'applicazione si trova nella sottocartella **articles**.

1. Creare un ambiente virtuale Python e installare il pacchetto SDK di Analisi del testo di Lingua di Azure AI e altri pacchetti necessari eseguendo il comando seguente:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-textanalytics==5.3.0
    ```

1. Immettere il comando seguente per modificare il file di configurazione dell'applicazione:

    ```
   code .env
    ```

    Il file viene aperto in un editor di codice.

1. Aggiornare i valori di configurazione per includere l'**endpoint** e una **chiave** dalla risorsa Lingua di Azure creata (disponibile nella pagina **Chiavi ed endpoint** per la risorsa Lingua di Azure AI nel portale di Azure). Il file deve contenere già i nomi di progetto e distribuzione per il modello di classificazione testo.
1. Dopo aver sostituito i segnaposto con l'editor di codice, usare il comando **CTRL+S** o **Fare clic con il pulsante destro del mouse > Salva** per salvare le modifiche e quindi usare il comando **CTRL+Q** o **Fare clic con il pulsante destro del mouse > Esci** per chiudere l'editor di codice mantenendo aperta la riga di comando di Cloud Shell.

## Aggiungere codice per classificare i documenti

1. Immettere il comando seguente per modificare il file di codice dell'applicazione:

    ```
    code classify-text.py
    ```

1. Esaminare il codice esistente. Si aggiungerà il codice per usare l'SDK Analisi del testo di Lingua di Azure AI.

    > **Suggerimento**: Quando si aggiunge codice al file di codice, assicurarsi di mantenere il rientro corretto.

1. Nella parte superiore del file di codice, sotto i riferimenti allo spazio dei nomi esistenti, trovare il commento **Importa spazi dei nomi** e aggiungere il codice seguente per importare gli spazi dei nomi necessari per usare l'SDK Analisi del testo:

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

1. Si noti che il codice esistente legge tutti i file nella cartella **articles** e crea un elenco del loro contenuto. Individuare quindi il commento **Get Classifications** e aggiungere il codice seguente:

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

1. Salvare le modifiche (CTRL+S), quindi immettere il comando seguente per eseguire il programma (ingrandire il riquadro Cloud Shell e ridimensionare i pannelli per visualizzare altro testo nel riquadro della riga di comando):

    ```
   python classify-text.py
    ```

1. Osservare l'output. L'applicazione deve elencare un punteggio di classificazione e di attendibilità per ogni file di testo.

## Eseguire la pulizia

Quando il progetto non è più necessario, è possibile eliminarlo dalla pagina **Progetti** in Language Studio. È anche possibile rimuovere il servizio Lingua di Azure AI e l'account di archiviazione associato nel [portale di Azure](https://portal.azure.com).
