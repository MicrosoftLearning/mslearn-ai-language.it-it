---
lab:
  title: Creare una soluzione di risposta alle domande
  module: Module 6 - Create question answering solutions with Azure AI Language
---

# Creare una soluzione di risposta alle domande

Uno degli scenari di conversazione più comuni consiste nel fornire supporto tramite una knowledge base di domande frequenti. Molte organizzazioni pubblicano le domande frequenti come documenti o pagine Web. Questo approccio risulta ottimale per un set ridotto di coppie di domanda e risposta, ma la ricerca nei documenti di grandi dimensioni può risultare difficile e può richiedere molto tempo.

Il servizio **Lingua di Azure AI** include una funzionalità di *risposta alla domanda* che consente di creare una knowledge base di coppie di domande e risposte su cui è possibile eseguire query mediante input in linguaggio naturale e che viene spesso usata come risorsa a cui un bot può fare riferimento per cercare le risposte alle domande inviate dagli utenti.

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
1. Visualizzare la pagina **Chiavi ed endpoint**. Le informazioni contenute in questa pagina saranno necessarie più avanti nell'esercizio.

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

## Prepararsi a sviluppare un'app in Visual Studio Code

Si svilupperà l'app di risposta alla domanda usando Visual Studio Code. I file di codice per l'app sono stati forniti in un repository GitHub.

> **Suggerimento**: se il repository **mslearn-ai-language** è già stato clonato, aprirlo in Visual Studio Code. In caso contrario, eseguire questi passaggi per clonarlo nell'ambiente di sviluppo.

1. Avviare Visual Studio Code.
2. Aprire il riquadro comandi (MAIUSC+CTRL+P) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/mslearn-ai-language` in una cartella locale. Non è importante usare una cartella specifica.
3. Dopo la clonazione del repository, aprire la cartella in Visual Studio Code.

    > **Nota**: Se Visual Studio Code visualizza un messaggio popup per chiedere se si considera attendibile il codice che si apre, fare clic sull'opzione **Sì, considero attendibili gli autori** nel popup.

4. Attendere il completamento dell'installazione di file aggiuntivi per supportare i progetti in codice C# nel repository.

    > **Nota**: se viene richiesto di aggiungere gli asset necessari per la compilazione e il debug, selezionare **Non adesso**.

## Configurare l'applicazione

Sono state fornite applicazioni sia per C# che per Python, oltre a un file di testo di esempio che verrà usato per testare il riepilogo. Entrambe le app presentano la stessa funzionalità. Prima di tutto, verranno completate alcune parti chiave dell'applicazione per consentirle di usare la risorsa Lingua di Azure AI.

1. Nel riquadro **Esplora risorse** di Visual Studio Code passare alla cartella **Labfiles/02-qna**, espandere la cartella **CSharp** o **Python** in base al linguaggio scelto ed espandere la cartella **qna-app** che contiene. Ogni cartella contiene i file specifici del linguaggio per un'app in cui si intende integrare la funzionalità di risposta alla domanda di Lingua di Azure AI.
2. Fare clic con il pulsante destro del mouse sulla cartella **qna-app** contenente i file di codice e aprire un terminale integrato. Installare quindi il pacchetto SDK di risposta alla domanda di Lingua di Azure AI eseguendo il comando appropriato per il linguaggio scelto:

    **C#:**

    ```
    dotnet add package Azure.AI.Language.QuestionAnswering
    ```

    **Python**:

    ```
    pip install azure-ai-language-questionanswering
    ```

3. Nel riquadro **Esplora risorse**, nella cartella **qna-app** aprire il file di configurazione per il linguaggio preferito

    - **C#**: appsettings.json
    - **Python**: .env
    
4. Aggiornare i valori di configurazione per includere l'**endpoint** e una **chiave** dalla risorsa Lingua di Azure creata (disponibile nella pagina **Chiavi ed endpoint** per la risorsa Lingua di Azure AI nel portale di Azure). Anche il nome del progetto e il nome della distribuzione per la knowledge base distribuita devono trovarsi in questo file.
5. Salvare il file di configurazione.

## Aggiungere codice all'applicazione

È ora possibile aggiungere il codice necessario per importare le librerie SDK necessarie, stabilire una connessione autenticata al progetto distribuito e inviare domande.

1. Notare che la cartella **qna-app** contiene un file di codice per l'applicazione client:

    - **C#**: Program.cs
    - **Python**: qna-app.py

    Aprire il file di codice e nella parte superiore, sotto i riferimenti agli spazi dei nomi esistenti, individuare il commento **Import namespaces**. In questo commento aggiungere quindi il codice seguente specifico per la lingua per importare gli spazi dei nomi necessari per usare Text Analytics SDK:

    **C#**: Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.Language.QuestionAnswering;
    ```

    **Python**: qna-app.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.language.questionanswering import QuestionAnsweringClient
    ```

1. Nella funzione **Main** si noti che il codice per caricare l'endpoint e la chiave di servizi Lingua di Azure AI dal file di configurazione è già stato fornito. Individuare quindi il commento **Create client using endpoint and key** e aggiungere il codice seguente per creare un client per l'API Analisi del testo:

    **C#**: Programs.cs

    ```C#
    // Create client using endpoint and key
    AzureKeyCredential credentials = new AzureKeyCredential(aiSvcKey);
    Uri endpoint = new Uri(aiSvcEndpoint);
    QuestionAnsweringClient aiClient = new QuestionAnsweringClient(endpoint, credentials);
    ```

    **Python**: qna-app.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = QuestionAnsweringClient(endpoint=ai_endpoint, credential=credential)
    ```

1. Nella funzione **Main** individuare il commento **Submit a question and display the answer** e aggiungere il codice seguente per leggere ripetutamente le domande dalla riga di comando, inviarle al servizio e visualizzare i dettagli delle risposte:

    **C#**: Programs.cs

    ```C#
    // Submit a question and display the answer
    string user_question = "";
    while (user_question.ToLower() != "quit")
        {
            Console.Write("Question: ");
            user_question = Console.ReadLine();
            QuestionAnsweringProject project = new QuestionAnsweringProject(projectName, deploymentName);
            Response<AnswersResult> response = aiClient.GetAnswers(user_question, project);
            foreach (KnowledgeBaseAnswer answer in response.Value.Answers)
            {
                Console.WriteLine(answer.Answer);
                Console.WriteLine($"Confidence: {answer.Confidence:P2}");
                Console.WriteLine($"Source: {answer.Source}");
                Console.WriteLine();
            }
        }
    ```

    **Python**: qna-app.py

    ```Python
    # Submit a question and display the answer
    user_question = ''
    while user_question.lower() != 'quit':
        user_question = input('\nQuestion:\n')
        response = ai_client.get_answers(question=user_question,
                                        project_name=ai_project_name,
                                        deployment_name=ai_deployment_name)
        for candidate in response.answers:
            print(candidate.answer)
            print("Confidence: {}".format(candidate.confidence))
            print("Source: {}".format(candidate.source))
    ```

1. Salvare le modifiche e tornare nel terminale integrato per la cartella **qna-app**, quindi immettere il comando seguente per eseguire il programma:

    - **C#**: `dotnet run`
    - **Python**: `python qna-app.py`

    > **Suggerimento**: è possibile usare l'icona **Ingrandisci dimensioni pannello** (**^**) nella barra degli strumenti del terminale per visualizzare altro testo della console.

1. Quando richiesto, immettere una domanda da inviare al progetto di risposta alle domande, ad esempio `What is a learning path?`.
1. Esaminare la risposta restituita.
1. Porre altre domande. Al termine, immettere `quit`.

## Pulire le risorse

Se si è terminato di esplorare il servizio Lingua di Azure AI, è possibile eliminare le risorse create in questo esercizio. In tal caso, eseguire la procedura seguente:

1. Aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.
2. Passare alla risorsa Lingua di Azure AI creata in questo lab.
3. Nella pagina della risorsa selezionare **Elimina** e seguire le istruzioni per eliminare la risorsa.

## Ulteriori informazioni

Per altre informazioni sulla funzionalità di risposta alla domanda in Lingua di Azure AI, vedere la [documentazione di Lingua di Azure AI](https://learn.microsoft.com/azure/ai-services/language-service/question-answering/overview).
