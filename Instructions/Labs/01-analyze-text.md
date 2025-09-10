---
lab:
  title: Analizzare il testo
  description: 'Usare Lingua di Azure AI per analizzare il testo, tra cui il rilevamento della lingua, l''analisi del sentiment, l''estrazione di frasi chiave e il riconoscimento di entità.'
---

# Analizzare il testo

**Lingua di Azure AI** supporta l'analisi del testo, tra cui il rilevamento della lingua, l'analisi del sentiment, l'estrazione di frasi chiave e il riconoscimento di entità.

Si supponga, ad esempio, che un'agenzia di viaggi voglia elaborare recensioni di alberghi inviate al sito Web della società. Il servizio Lingua di Azure AI consente di determinare la lingua in cui è scritta ogni recensione, la valutazione (positiva, neutra o negativa) delle recensioni, le frasi chiave che possono indicare gli argomenti principali discussi nella recensione e le entità denominate, ad esempio località, luoghi di interesse o persone citate nelle recensioni. In questo esercizio si userà l'SDK Python di Lingua di Azure AI per l'analisi del testo per implementare una semplice applicazione di recensioni di hotel basata su questo esempio.

Anche se questo esercizio è basato su Python, è possibile sviluppare applicazioni di analisi del testo usando più SDK specifici del linguaggio, tra cui:

- [Libreria client di Analisi del testo di Azure AI per Python](https://pypi.org/project/azure-ai-textanalytics/)
- [Libreria client di Analisi del testo di Azure AI per .NET](https://www.nuget.org/packages/Azure.AI.TextAnalytics)
- [Libreria client di Analisi del testo di Azure AI per JavaScript](https://www.npmjs.com/package/@azure/ai-text-analytics)

Questo esercizio richiede circa **30** minuti.

## Effettuare il provisioning di una risorsa *Lingua di Azure AI*

Se non ne è già disponibile una nella sottoscrizione, è necessario effettuare il provisioning di una risorsa del servizio **Lingua di Azure AI** nella sottoscrizione di Azure.

1. Aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.
1. Selezionare **Crea una risorsa**.
1. Nel campo di ricerca, cercare **Servizio di linguaggio**. Quindi, nei risultati selezionare **Crea** in **Servizio linguistico**.
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
1. Individuare **Endpoint e chiavi** nella sezione **Gestione risorse**. Le informazioni contenute in questa pagina saranno necessarie più avanti nell'esercizio.

## Clonare il repository per questo corso

Verrà sviluppato il codice usando Cloud Shell dal portale di Azure. I file di codice per l'app sono stati forniti in un repository GitHub.

1. Nel portale di Azure, usare il pulsante **[\>_]** a destra della barra di ricerca nella parte superiore della pagina per creare una nuova Cloud Shell nel portale di Azure, selezionando un ambiente ***PowerShell***. Cloud Shell fornisce un'interfaccia della riga di comando in un riquadro nella parte inferiore del portale di Azure.

    > **Nota**: se in precedenza è stata creata una sessione Cloud Shell che usa un ambiente *Bash*, passare a ***PowerShell***.

1. Nella barra degli strumenti di Cloud Shell scegliere **Vai alla versione classica** dal menu **Impostazioni**. Questa operazione è necessaria per usare l'editor di codice.

    **<font color="red">Verificare di passare alla versione classica di Cloud Shell prima di continuare.</font>**

1. Nel riquadro PowerShell immettere i comandi seguenti per clonare il repository GitHub per questo esercizio:

    ```
    rm -r mslearn-ai-language -f
    git clone https://github.com/microsoftlearning/mslearn-ai-language
    ```

    > **Suggerimento**: Quando si immettono i comandi in Cloud Shell, l'ouput può richiedere una grande quantità di buffer dello schermo. È possibile cancellare la schermata immettendo il `cls` comando per rendere più semplice concentrarsi su ogni attività.

1. Dopo aver clonato il repository, passare alla cartella contenente i file di codice dell'applicazione:  

    ```
    cd mslearn-ai-language/Labfiles/01-analyze-text/Python/text-analysis
    ```

## Configurare l'applicazione

1. Nel riquadro della riga di comando eseguire il comando seguente per visualizzare i file di codice nella cartella **text-analysis**:

    ```
   ls -a -l
    ```

    I file includono un file di configurazione (con estensione **env**) e un file di codice (**text-analysis.py**). Il testo che verrà analizzato dall'applicazione si trova nella sottocartella **reviews**.

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

1. Aggiornare i valori di configurazione per includere l'**endpoint** e una **chiave** dalla risorsa Lingua di Azure creata (disponibile nella pagina **Chiavi ed endpoint** per la risorsa Lingua di Azure AI)
1. Dopo aver sostituito i segnaposto con l'editor di codice, usare il comando **CTRL+S** o **Fare clic con il pulsante destro del mouse > Salva** per salvare le modifiche e quindi usare il comando **CTRL+Q** o **Fare clic con il pulsante destro del mouse > Esci** per chiudere l'editor di codice mantenendo aperta la riga di comando di Cloud Shell.

## Aggiungere il codice per connettersi alla risorsa Lingua di Azure AI

1. Immettere il comando seguente per modificare il file di codice dell'applicazione:

    ```
    code text-analysis.py
    ```

1. Esaminare il codice esistente. Si aggiungerà il codice per usare l'SDK Analisi del testo di Lingua di Azure AI.

    > **Suggerimento**: Quando si aggiunge codice al file di codice, assicurarsi di mantenere il rientro corretto.

1. Nella parte superiore del file di codice, sotto i riferimenti allo spazio dei nomi esistenti, trovare il commento **Importa spazi dei nomi** e aggiungere il codice seguente per importare gli spazi dei nomi necessari per usare l'SDK Analisi del testo:

    ```python
   # import namespaces
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. Nella funzione **main** il codice per caricare l'endpoint e la chiave del servizio di Lingua di Azure AI dal file di configurazione è già stato fornito. Individuare quindi il commento **Create client using endpoint and key** e aggiungere il codice seguente per creare un client per l'API Analisi del testo:

    ```Python
   # Create client using endpoint and key
   credential = AzureKeyCredential(ai_key)
   ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

1. Salvare le modifiche (CTRL+S), quindi immettere il comando seguente per eseguire il programma (ingrandire il riquadro Cloud Shell e ridimensionare i pannelli per visualizzare altro testo nel riquadro della riga di comando):

    ```
   python text-analysis.py
    ```

1. Osservare l'output perché il codice deve essere eseguito senza errori, visualizzando i contenuti di ogni file di testo di recensione nella cartella **reviews**. L'applicazione crea correttamente un client per l'API Analisi del testo, ma non la usa. Questo problema verrà corretto nella prossima sezione.

## Aggiungere codice per rilevare la lingua

Ora che è stato creato un client per l'API, è possibile usarlo per rilevare la lingua in cui viene scritta ogni recensione.

1. Nell'editor di codice trovare il commento **Ottieni lingua**. Aggiungere quindi il codice necessario per rilevare la lingua in ogni recensione:

    ```python
   # Get language
   detectedLanguage = ai_client.detect_language(documents=[text])[0]
   print('\nLanguage: {}'.format(detectedLanguage.primary_language.name))
    ```

     > **Nota**: *in questo esempio ogni recensione viene analizzata singolarmente e di conseguenza viene effettuata una chiamata separata al servizio per ogni file. Un approccio alternativo consiste nel creare una raccolta di documenti e passarli al servizio in un'unica chiamata. In entrambi gli approcci, la risposta restituita dal servizio è costituita da una raccolta di documenti. È per questo motivo che nel codice Python precedente viene specificato l'indice del primo (e unico) documento incluso nella risposta ([0]).*

1. Salva le modifiche. Rieseguire quindi il programma.
1. Osservare l'output, notando che questa volta viene identificata la lingua per ogni recensione.

## Aggiungere codice per valutare il sentiment

L'*analisi valutazione* è una tecnica usata in genere per classificare il testo come *positivo* o *negativo* (oppure possibile *neutro* o *misto*). Viene usato in genere per analizzare post di social media, recensioni di prodotti e altri elementi in cui la valutazione del testo può fornire utili informazioni dettagliate.

1. Nell'editor di codice trovare il commento **Ottieni valutazione**. Aggiungere quindi il codice necessario per rilevare la valutazione di ogni recensione:

    ```python
   # Get sentiment
   sentimentAnalysis = ai_client.analyze_sentiment(documents=[text])[0]
   print("\nSentiment: {}".format(sentimentAnalysis.sentiment))
    ```

1. Salva le modifiche. Chiudere quindi l'editor di codice ed eseguire di nuovo il programma.
1. Osservare l'output, notando che viene rilevata la valutazione delle recensioni.

## Aggiungere codice per identificare le frasi chiave

Può essere utile identificare le frasi chiave nel corpo di un testo per determinare i principali argomenti trattati.

1. Nell'editor di codice trovare il commento **Ottieni frasi chiave**. Aggiungere quindi il codice necessario per rilevare le frasi chiave in ogni recensione:

    ```python
   # Get key phrases
   phrases = ai_client.extract_key_phrases(documents=[text])[0].key_phrases
   if len(phrases) > 0:
        print("\nKey Phrases:")
        for phrase in phrases:
            print('\t{}'.format(phrase))
    ```

1. Salvare le modifiche ed eseguire di nuovo il programma.
1. Osservare l'output, notando che ogni documento contiene frasi chiave che forniscono alcune informazioni dettagliate sul contenuto della recensione.

## Aggiungere codice per estrarre le entità

Spesso nei documenti o in altri corpi del testo vengono citati persone, luoghi, periodi di tempo o altre entità. L'API Analisi del testo è in grado di rilevare più categorie (e sottocategorie) dell'entità nel testo.

1. Nell'editor di codice trovare il commento **Ottieni entità**. Aggiungere quindi il codice necessario per identificare le entità citate in ogni recensione:

    ```python
   # Get entities
   entities = ai_client.recognize_entities(documents=[text])[0].entities
   if len(entities) > 0:
        print("\nEntities")
        for entity in entities:
            print('\t{} ({})'.format(entity.text, entity.category))
    ```

1. Salvare le modifiche ed eseguire di nuovo il programma.
1. Osservare l'output, notando le entità che sono state rilevate nel testo.

## Aggiungere codice per estrarre le entità collegate

Oltre alle entità classificate, l'API Analisi del testo è in grado di rilevare le entità per cui sono presenti collegamenti noti alle origini dati, ad esempio Wikipedia.

1. Nell'editor di codice trovare il commento **Ottieni entità collegate**. Aggiungere quindi il codice necessario per identificare le entità collegate citate in ogni recensione:

    ```python
   # Get linked entities
   entities = ai_client.recognize_linked_entities(documents=[text])[0].entities
   if len(entities) > 0:
        print("\nLinks")
        for linked_entity in entities:
            print('\t{} ({})'.format(linked_entity.name, linked_entity.url))
    ```

1. Salvare le modifiche ed eseguire di nuovo il programma.
1. Osservare l'output, notando le entità collegate che sono state identificate.

## Pulire le risorse

Se si è terminato di esplorare il servizio Lingua di Azure AI, è possibile eliminare le risorse create in questo esercizio. In tal caso, eseguire la procedura seguente:

1. Chiudere il riquadro Azure Cloud Shell.
1. Nel portale di Azure passare alla risorsa Lingua di Azure AI creata in questo lab.
1. Nella pagina della risorsa, selezionare **Elimina** e seguire le istruzioni per eliminare la risorsa.

## Ulteriori informazioni

Per altre informazioni sull'uso di **Lingua di Azure AI**, vedere la [documentazione](https://learn.microsoft.com/azure/ai-services/language-service/).
