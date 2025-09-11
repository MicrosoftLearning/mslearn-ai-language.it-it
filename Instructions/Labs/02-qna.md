---
lab:
  title: Creare una soluzione di risposta alle domande
  description: Usare Lingua di Azure AI per creare una soluzione di risposta alle domande personalizzata.
---

# Creare una soluzione di risposta alle domande

Uno degli scenari di conversazione più comuni consiste nel fornire supporto tramite una knowledge base di domande frequenti. Molte organizzazioni pubblicano le domande frequenti come documenti o pagine Web. Questo approccio risulta ottimale per un set ridotto di coppie di domanda e risposta, ma la ricerca nei documenti di grandi dimensioni può risultare difficile e può richiedere molto tempo.

Il servizio **Lingua di Azure AI** include una funzionalità di *risposta alla domanda* che consente di creare una knowledge base di coppie di domande e risposte su cui è possibile eseguire query mediante input in linguaggio naturale e che viene spesso usata come risorsa a cui un bot può fare riferimento per cercare le risposte alle domande inviate dagli utenti. In questo esercizio si userà l'SDK Python di Lingua di Azure AI per l'analisi del testo per implementare una semplice applicazione di risposta alle domande.

Anche se questo esercizio è basato su Python, è possibile sviluppare applicazioni di risposta alle domande usando più SDK specifici del linguaggio, tra cui:

- [Libreria client di risposta alle domande del servizio Lingua di Azure AI per Python](https://pypi.org/project/azure-ai-language-questionanswering/)
- [Libreria client di risposta alle domande del servizio Lingua di Azure AI per .NET](https://www.nuget.org/packages/Azure.AI.Language.QuestionAnswering)

Questo esercizio richiede circa **20** minuti.

## Effettuare il provisioning di una risorsa *Lingua di Azure AI*

Se non ne è già disponibile una nella sottoscrizione, è necessario effettuare il provisioning di una risorsa del **servizio Lingua di Azure AI**. Inoltre, per creare e ospitare una knowledge base per la risposta alla domanda, è necessario abilitare la funzionalità **Risposta alla domanda**.

1. Aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.
1. Selezionare **Crea una risorsa**.
1. Nel campo di ricerca, cercare **Servizio di linguaggio**. Quindi, nei risultati selezionare **Crea** in **Servizio linguistico**.
1. Selezionare il blocco **Risposta personalizzata alle domande**. Selezionare quindi **Continua a creare la risorsa**. Sarà necessario immettere le impostazioni seguenti:

    - **Sottoscrizione**: *la sottoscrizione di Azure usata*
    - **Gruppo di risorse**: *Scegliere o creare un gruppo di risorse*.
    - **Area**: *scegliere qualsiasi località disponibile*
    - **Nome**: *immettere un nome univoco*
    - **Piano tariffario**: Selezionare **F0** (*gratuito*) o **S** (*standard*) se F non è disponibile.
    - **Area di Ricerca di Azure**: * scegliere una posizione nella stessa area globale della risorsa del servizio Lingua*
    - **Piano tariffario di Ricerca di Azure**: Gratuito (F) *Se questo piano non è disponibile, selezionare Basic (B)*
    - **Avviso per intelligenza artificiale responsabile**: *accettare*

1. Selezionare **Crea + rivedi**e quindi selezionare **Crea**.

    > **NOTA** La funzionalità Risposta personalizzata alle domande usa Ricerca di Azure per indicizzare ed eseguire query nella knowledge base di domande e risposte.

1. Attendere il completamento della distribuzione e quindi passare alla risorsa distribuita.
1. Individuare **Endpoint e chiavi** nella sezione **Gestione risorse**. Le informazioni contenute in questa pagina saranno necessarie più avanti nell'esercizio.

## Creare un progetto di risposta alle domande

Per creare una knowledge base per la risposta alla domanda nella risorsa Lingua di Azure AI, è possibile usare il portale Language Studio per creare un progetto di risposta alle domande. In questo caso, si creerà un knowledge base contenente domande e risposte su [Microsoft Learn](https://docs.microsoft.com/learn).

1. In una nuova scheda del browser passare al portale Language Studio all'indirizzo [https://language.cognitive.azure.com/](https://language.cognitive.azure.com/) e accedere usando l'account Microsoft associato alla sottoscrizione di Azure.
1. Se viene richiesto di scegliere una risorsa Lingua, selezionare le impostazioni seguenti:
    - **Directory di Azure**: directory di Azure contenente la sottoscrizione.
    - **Sottoscrizione di Azure**: sottoscrizione di Azure in uso.
    - **Tipo di risorsa**: Linguaggio
    - **Nome risorsa**: risorsa Lingua di Azure AI creata in precedenza.

    Se <u>non</u> viene chiesto di scegliere una risorsa Lingua, è possibile che nella sottoscrizione siano presenti più risorse di questo tipo e in tal caso:

    1. Sulla barra nella parte superiore della pagina selezionare il pulsante **Impostazioni (&#9881;)**.
    2. Nella pagina **Impostazioni** visualizzare la scheda **Risorse**.
    3. Selezionare la risorsa del servizio Lingua appena creata e fare clic su **Cambiare risorsa**.
    4. Nella parte superiore della pagina fare clic su **Language Studio** per tornare alla pagina iniziale di Language Studio.

1. Nella parte superiore del portale scegliere **Risposta personalizzata alle domande** dal menu **Crea nuovo**.
1. Nella procedura guidata ***Crea un progetto**, nella pagina **Scegli l'impostazione della lingua**, selezionare l'opzione **Imposta la lingua per tutti i progetti in questa risorsa** e selezionare la lingua **Inglese**. Quindi seleziona **Avanti**.
1. Nella pagina **Immetti le informazioni di base** immettere i dettagli seguenti:
    - **Nome** `LearnFAQ`
    - **Descrizione**: `FAQ for Microsoft Learn`
    - **Risposta predefinita quando non viene restituita alcuna risposta**: `Sorry, I don't understand the question`
1. Selezionare **Avanti**.
1. Nella pagina **Rivedi e completa** selezionare **Crea progetto**.

## Aggiungere origini alla knowledge base

È possibile creare una knowledge base completamente nuova, ma in genere si inizia importando domande e risposte da una pagina o da un documento di domande frequenti esistente. In questo caso, si importeranno i dati da una pagina Web di domande frequenti esistente per Microsoft Learn e si importeranno anche alcune domande e risposte predefinite per supportare gli scambi colloquiali.

1. Nella pagina **Manage sources** (Gestisci origini) per il progetto di risposta alle domande selezionare **URL** nell'elenco **&#9547; Aggiungi origine**. Nella finestra di dialogo **Aggiungi URL** selezionare **&#9547; Aggiungi URL** e impostare il nome e l'URL seguenti prima di selezionare **Aggiungi tutto** per aggiungerli alla knowledge base:
    - **Nome**: `Learn FAQ Page`
    - **URL**: `https://docs.microsoft.com/en-us/learn/support/faq`
1. Nella pagina **Manage sources** (Gestisci origini) per il progetto di risposta alle domande selezionare **Chitchat** (Chiacchiere) nell'elenco **&#9547; Aggiungi origine**. Nella finestra di dialogo **Aggiungi chiacchiere** selezionare **	Nome descrittivo** e fare clic su **Aggiungi chiacchiere**.

## Modificare la knowledge base

La knowledge base è stata popolata con coppie di domanda e risposta dalle domande frequenti di Microsoft Learn, a cui è stato aggiunto un set di coppie di domanda e risposta di *Chiacchiere* basate su conversazione. È possibile estendere la knowledge base aggiungendo altre coppie di domanda e risposta.

1. Nel progetto **LearnFAQ** in Language Studio selezionare la pagina **Modifica knowledge base** per visualizzare le coppie di domande e risposte esistenti (se vengono visualizzati suggerimenti, leggerli e selezionare **OK** per eliminarli oppure fare clic su **Ignora tutto**)
1. Nella knowledge base, nella scheda **Coppia di domanda e risposta** selezionare **&#65291;** e creare una nuova coppia di domanda e risposta con le impostazioni seguenti:
    - **Origine**:  `https://docs.microsoft.com/en-us/learn/support/faq`
    - **Domanda**: `What are Microsoft credentials?`
    - **Risposta**: `Microsoft credentials enable you to validate and prove your skills with Microsoft technologies.`
1. Selezionare **Fatto**.
1. Nella pagina per la domanda **What are Microsoft credentials?** che viene creata, espandere **Domande alternative**. Aggiungere quindi la domanda alternativa `How can I demonstrate my Microsoft technology skills?`.

    In alcuni casi, è opportuno consentire all'utente di porre un'ulteriore domanda in seguito alla risposta creando una conversazione *a più turni* che consenta all'utente di perfezionare in modo iterativo la domanda fino a ottenere la risposta necessaria.

1. Nella risposta immessa per la domanda sulla certificazione espandere **Richieste di completamento** e aggiungere la richiesta di completamento seguente:
    - **Text displayed in the prompt to the user**: `Learn more about credentials`.
    - Selezionare la scheda **Crea collegamento a nuova coppia** e immettere il testo seguente: `You can learn more about credentials on the [Microsoft credentials page](https://docs.microsoft.com/learn/credentials/).`
    - Selezionare **Mostrare solo nel flusso contestuale**. Questa opzione assicura che la risposta venga restituita solo nel contesto di una domanda di completamento dalla domanda di certificazione originale.
1. Selezionare **Aggiungi richiesta**.

## Eseguire il training e il test della knowledge base

Ora che si ha una knowledge base, è possibile testarla in Language Studio.

1. Salvare le modifiche apportate alla knowledge base selezionando il pulsante **Salva** sotto la scheda **Coppia di domanda e risposta** a sinistra.
1. Dopo aver salvato le modifiche, selezionare il pulsante **Test** per aprire il riquadro di test.
1. Nella parte superiore del riquadro di test deselezionare l'opzione **Includi risposta breve** (se non è già deselezionata). Nella parte inferiore immettere quindi il messaggio `Hello`. Dovrebbe essere restituita una risposta appropriata.
1. Nella parte inferiore del riquadro di test immettere il messaggio `What is Microsoft Learn?`. Dovrebbe essere restituita una risposta appropriata dalle domande frequenti.
1. Immettere il messaggio `Thanks!`. Verrà restituita una risposta colloquiale appropriata.
1. Immetti il messaggio `Tell me about Microsoft credentials`. Verrà restituita la risposta creata insieme a un collegamento per la richiesta di completamento.
1. Selezionare il collegamento di completamento **Altre informazioni sulle credenziali**. Dovrebbe essere restituita una risposta di completamento con un collegamento alla pagina di certificazione.
1. Al termine del test della knowledge base, chiudere il riquadro di test.

## Distribuire la knowledge base

La knowledge base offre un servizio back-end che le applicazioni client possono usare per rispondere alle domande. È ora possibile pubblicare la knowledge base e accedere alla rispettiva interfaccia REST da un client.

1. Nel progetto **LearnFAQ** in Language Studio, selezionare la pagina **Implementa knowledge base** dal menu di spostamento sulla sinistra.
1. Nella parte superiore della pagina selezionare **Distribuisci**. Selezionare quindi **Distribuisci** per confermare la distribuzione della knowledge base.
1. Al termine della distribuzione, fare clic su **Ottieni URL di stima** per visualizzare l'endpoint REST per la knowledge base e copiarlo negli Appunti e notare che la richiesta di esempio include i parametri per:
    - **projectName**: nome del progetto (dovrebbe essere *LearnFAQ*)
    - **deploymentName**: nome della distribuzione (dovrebbe essere *produzione*)
1. Chiudere la finestra di dialogo dell'URL di stima.

## Preparare lo sviluppo di un'app in Cloud Shell

Si svilupperà l'app di risposta alle domande usando Cloud Shell nel portale di Azure. I file di codice per l'app sono stati forniti in un repository GitHub.

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
    cd mslearn-ai-language/Labfiles/02-qna/Python/qna-app
    ```

## Configurare l'applicazione

1. Nel riquadro della riga di comando eseguire il comando seguente per visualizzare i file di codice nella cartella **qna-app**:

    ```
   ls -a -l
    ```

    I file includono un file di configurazione (con estensione **env**) e un file di codice (**qna-app.py**).

1. Creare un ambiente virtuale Python e installare il pacchetto SDK di risposta alle domande di Lingua di Azure AI e altri pacchetti necessari eseguendo il comando seguente:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-language-questionanswering
    ```

1. Immettere il comando seguente per modificare il file di configurazione:

    ```
    code .env
    ```

    Il file viene aperto in un editor di codice.

1. Nel file di codice, aggiorna i valori di configurazione in esso contenuti per riflettere l'**endpoint** e una **chiave** di autenticazione per la risorsa Lingua di Azure creata (disponibile nella pagina **Chiavi ed endpoint** per la risorsa Lingua di Azure AI nel portale di Azure). Anche il nome del progetto e il nome della distribuzione per la knowledge base distribuita devono trovarsi in questo file.
1. Dopo aver sostituito i segnaposto con l'editor di codice, usare il comando **CTRL+S** o **Fare clic con il pulsante destro del mouse > Salva** per salvare le modifiche e quindi usare il comando **CTRL+Q** o **Fare clic con il pulsante destro del mouse > Esci** per chiudere l'editor di codice mantenendo aperta la riga di comando di Cloud Shell.

## Aggiungere il codice per usare la knowledge base

1. Immettere il comando seguente per modificare il file di codice dell'applicazione:

    ```
    code qna-app.py
    ```

1. Esaminare il codice esistente. Si aggiungerà il codice per usare la knowledge base.

    > **Suggerimento**: Quando si aggiunge codice al file di codice, assicurarsi di mantenere il rientro corretto.

1. Nel file di codice trovare il commento **Importa spazi dei nomi**. In questo commento aggiungere quindi il codice seguente specifico del linguaggio per importare gli spazi dei nomi necessari per usare l'SDK di risposta alle domande:

    ```python
   # import namespaces
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.language.questionanswering import QuestionAnsweringClient
    ```

1. Nella funzione **main** il codice per caricare l'endpoint e la chiave del servizio di Lingua di Azure AI dal file di configurazione è già stato fornito. Individuare quindi il commento **Creare un client usando l’endpoint e la chiave** e aggiungere il codice seguente per creare un client di risposta alle domande:

    ```Python
   # Create client using endpoint and key
   credential = AzureKeyCredential(ai_key)
   ai_client = QuestionAnsweringClient(endpoint=ai_endpoint, credential=credential)
    ```

1. Nel file di codice individuare il commento **Inviare una domanda e visualizzare la risposta** e aggiungere il codice seguente per leggere ripetutamente le domande dalla riga di comando, inviarle al servizio e visualizzare i dettagli delle risposte:

    ```Python
   # Submit a question and display the answer
   user_question = ''
   while True:
        user_question = input('\nQuestion:\n')
        if user_question.lower() == "quit":                
            break
        response = ai_client.get_answers(question=user_question,
                                        project_name=ai_project_name,
                                        deployment_name=ai_deployment_name)
        for candidate in response.answers:
            print(candidate.answer)
            print("Confidence: {}".format(candidate.confidence))
            print("Source: {}".format(candidate.source))
    ```

1. Salvare le modifiche (CTRL+S), quindi immettere il comando seguente per eseguire il programma (ingrandire il riquadro Cloud Shell e ridimensionare i pannelli per visualizzare altro testo nel riquadro della riga di comando):

    ```
   python qna-app.py
    ```

1. Quando richiesto, immettere una domanda da inviare al progetto di risposta alle domande, ad esempio `What is a learning path?`.
1. Esaminare la risposta restituita.
1. Porre altre domande. Al termine, immettere `quit`.

## Pulire le risorse

Se si è terminato di esplorare il servizio Lingua di Azure AI, è possibile eliminare le risorse create in questo esercizio. In tal caso, eseguire la procedura seguente:

1. Chiudere il riquadro Azure Cloud Shell.
1. Nel portale di Azure passare alla risorsa Lingua di Azure AI creata in questo lab.
1. Nella pagina della risorsa, selezionare **Elimina** e seguire le istruzioni per eliminare la risorsa.

## Ulteriori informazioni

Per altre informazioni sulla funzionalità di risposta alla domanda in Lingua di Azure AI, vedere la [documentazione di Lingua di Azure AI](https://learn.microsoft.com/azure/ai-services/language-service/question-answering/overview).
