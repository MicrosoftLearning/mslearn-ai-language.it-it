---
lab:
  title: Sviluppare un'app di chat abilitata per gli audio
  description: Informazioni su come usare Fonderia Azure AI per creare un'app di intelligenza artificiale generativa che supporta input audio.
---

# Sviluppare un'app di chat abilitata per gli audio

In questo esercizio, verrà usato il modello di intelligenza artificiale generativa *Phi-4-multimodal-instruct* per generare risposte a richieste che includono file audio. Verrà sviluppata un'applicazione che fornisce assistenza AI per i prodotti freschi in un negozio di alimentari usando Fonderia Azure AI e il servizio dell'inferenza del modello di Azure per intelligenza artificiale.

Questo esercizio richiede circa **30** minuti.

## Creare un progetto Fonderia Azure AI

Per iniziare, creare un progetto Fonderia Azure AI.

1. In un Web browser, aprire il [Portale Fonderia Azure AI](https://ai.azure.com) su `https://ai.azure.com` e accedere usando le credenziali di Azure. Chiudere i suggerimenti o i riquadri di avvio rapido aperti la prima volta che si accede e, se necessario, usare il logo **Fonderia Azure AI** in alto a sinistra per passare alla home page, simile all'immagine seguente:

    ![Screenshot del portale di Azure AI Foundry.](../media/ai-foundry-home.png)

1. Nella home page, selezionare **+ Crea progetto**.
1. Nella procedura guidata **Crea un progetto**, immettere un nome appropriato per il progetto. Se viene suggerito un hub esistente, selezionare l'opzione per crearne uno nuovo. Successivamente, esaminare le risorse Azure che verranno create automaticamente per supportare l'hub e il progetto.
1. Selezionare **Personalizza** e specificare le impostazioni seguenti per l'hub:
    - **Nome hub**: *un nome valido per l'hub*
    - **Sottoscrizione**: *la sottoscrizione di Azure usata*
    - **Gruppo di risorse**: *creare o selezionare un gruppo di risorse*
    - **Posizione**: selezionare una delle aree seguenti\*:
        - Stati Uniti orientali
        - Stati Uniti orientali 2
        - Stati Uniti centro-settentrionali
        - Stati Uniti centro-meridionali
        - Svezia centrale
        - Stati Uniti occidentali
        - Stati Uniti occidentali 3
    - **Connettere Servizi di Azure AI o Azure OpenAI**: *Creare una nuova risorsa di Servizi di AI*
    - **Connettere Azure AI Search**: ignorare la connessione

    > \* Al momento della stesura del presente documento, il modello Microsoft *Phi-4-multimodal-instruct* che verrà usato in questo esercizio è disponibile in queste regioni. È possibile controllare la disponibilità a livello di area più recente per modelli specifici nella documentazione di [Fonderia Azure AI ](https://learn.microsoft.com/azure/ai-foundry/how-to/deploy-models-serverless-availability#region-availability). In caso di raggiungimento di un limite di quota di area più avanti nell'esercizio, potrebbe essere necessario creare un'altra risorsa in un'area diversa.

1. Selezionare **Avanti** per esaminare la configurazione. Quindi selezionare **Crea** e attendere il completamento del processo.
1. Quando viene creato il progetto, chiudere tutti i suggerimenti visualizzati e rivedere la pagina del progetto nel portale Fonderia di Azure AI, che dovrebbe essere simile all'immagine seguente:

    ![Screenshot dei dettagli di un progetto di Azure AI nel portale Fonderia di Azure AI.](../media/ai-foundry-project.png)

## Distribuire un modello multimodale

A questo punto è possibile distribuire un modello multimodale in grado di supportare l'input basato su audio. Esistono diversi modelli tra cui scegliere, compreso il modello OpenAI *gpt-4o*. In questo esercizio, verrà utilizzato un modello *Phi-4-multimodal-instruct*.

1. Nella barra degli strumenti nella parte superiore destra della pagina del progetto Fonderia Azure AI, usare l'icona **Funzionalità di anteprima** (**&#9215;**) per assicurarsi che la funzionalità **Distribuisci modelli nel servizio di inferenza del modello di Azure per intelligenza artificiale** sia abilitata. Questa funzionalità garantisce che la distribuzione del modello sia disponibile per il servizio di inferenza di Azure AI, che verrà usato nel codice dell'applicazione.
1. Nel riquadro a sinistra del progetto, nella sezione **Risorse personali** selezionare la pagina **Modelli + endpoint**.
1. Nella scheda **Distribuzioni del modello** della pagina **Modelli + endpoint**, nel menu **+ Distribuisci modello** selezionare **Distribuisci modello di base**.
1. Cercare il modello **Phi-4-multimodal-instruct** nell'elenco, quindi selezionarlo e confermarlo.
1. Accettare il contratto di licenza se richiesto e quindi distribuire il modello con le impostazioni seguenti selezionando **Personalizza** nei dettagli della distribuzione:
    - **Nome distribuzione**: *nome univoco per la distribuzione del modello*
    - **Tipo di distribuzione**: standard globale
    - **Dettagli della distribuzione**: *usare le impostazioni predefinite*
1. Attendere che lo stato di provisioning della distribuzione sia **completato**.

## Creare un'applicazione client

Dopo aver distribuito il modello, è possibile usare la distribuzione in un'applicazione client.

> **Suggerimento**: è possibile scegliere di sviluppare la soluzione usando Python o Microsoft C#. Seguire le istruzioni nella sezione appropriata per la lingua scelta.

### Preparare la configurazione dell'applicazione

1. Nel Portale Fonderia Azure AI visualizzare la pagina **Panoramica** per il progetto.
1. Nell'area **Dettagli di progetto** prendere nota della **stringa di connessione del progetto**. Questa stringa di connessione verrà usata per connettersi al progetto in un'applicazione client.
1. Aprire una nuova scheda del browser (mantenendo aperto il Portale Fonderia Azure AI nella scheda esistente). In una nuova scheda del browser, passare al [portale di Azure](https://portal.azure.com) su `https://portal.azure.com`, accedendo con le credenziali di Azure se richiesto.

    Chiudere eventuali notifiche di benvenuto per visualizzare la pagina iniziale del portale di Azure.

1. Usare il pulsante **[\>_]** a destra della barra di ricerca, nella parte superiore della pagina, per aprire una nuova sessione di Cloud Shell nel portale di Azure selezionando un ambiente ***PowerShell*** senza archiviazione nell'abbonamento.

    Cloud Shell fornisce un'interfaccia della riga di comando in un riquadro nella parte inferiore del portale di Azure. È possibile ridimensionare o ingrandire questo riquadro per ottimizzare l'esperienza d'uso.

    > **Nota**: se in precedenza è stata creata una sessione Cloud Shell che usa un ambiente *Bash*, passare a ***PowerShell***.

1. Nella barra degli strumenti di Cloud Shell scegliere **Vai alla versione classica** dal menu **Impostazioni**. Questa operazione è necessaria per usare l'editor di codice.

    **<font color="red">Verificare di passare alla versione classica di Cloud Shell prima di continuare.</font>**

1. Nel riquadro Cloud Shell immettere i comandi seguenti per clonare il repository GitHub contenente i file di codice per questo esercizio (digitare il comando o copiarlo negli Appunti e quindi fare clic con il pulsante destro del mouse nella riga di comando e incollarlo come testo normale):


    ```
    rm -r mslearn-ai-audio -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-language mslearn-ai-audio
    ```

    > **Suggerimento**: quando si incollano i comandi in CloudShell, l'ouput può richiedere una grande quantità di buffer dello schermo. È possibile cancellare la schermata immettendo il `cls` comando per rendere più semplice concentrarsi su ogni attività.

1. Dopo aver clonato il repository, passare alla cartella contenente i file di codice dell'applicazione:  

    **Python**

    ```
    cd mslearn-ai-audio/Labfiles/09-audio-chat/python
    ```

    **C#**

    ```
    cd mslearn-ai-audio/Labfiles/09-audio-chat/c-sharp
    ```

1. Nel riquadro della riga di comando di Cloud Shell immettere il comando seguente per installare le librerie che si useranno, ovvero:

    **Python**

    ```
    python -m venv labenv
    ./labenv/bin/Activate.ps1
    pip install python-dotenv azure-identity azure-ai-projects azure-ai-inference
    ```

    **C#**

    ```
    dotnet add package Azure.Identity
    dotnet add package Azure.AI.Inference --version 1.0.0-beta.3
    dotnet add package Azure.AI.Projects --version 1.0.0-beta.3
    ```

1. Immettere il comando seguente per modificare il file di configurazione fornito:

    **Python**

    ```
    code .env
    ```

    **C#**

    ```
    code appsettings.json
    ```

    Il file deve essere aperto in un editor di codice.

1. Nel file di codice, sostituire il segnaposto **your_project_connection_string** con la stringa di connessione per il progetto (copiata dalla pagina **Panoramica** del Portale Fonderia Azure AI) e il segnaposto **your_model_deployment** con il nome assegnato alla distribuzione modello Phi-4-multimodal-instruct.

1. Dopo aver sostituito i segnaposto con l'editor di codice, usare il comando **CTRL+S** o **Fare clic con il pulsante destro del mouse > Salva** per salvare le modifiche e quindi usare il comando **CTRL+Q** o **Fare clic con il pulsante destro del mouse > Esci** per chiudere l'editor di codice mantenendo aperta la riga di comando di Cloud Shell.

### Scrivere il codice per connettersi al progetto e ottenere un client di chat per il modello

> **Suggerimento**: quando si aggiunge codice, assicurarsi di mantenere il rientro corretto.

1. Immettere il comando seguente per modificare il file di codice fornito:

    **Python**

    ```
    code audio-chat.py
    ```

    **C#**

    ```
    code Program.cs
    ```

1. Nel file di codice prendere nota delle istruzioni esistenti aggiunte all'inizio del file per importare i namespaces (spazi dei nomi) SDK necessari. Quindi, trovare il commento **Aggiungi riferimenti** e aggiungere il codice seguente per fare riferimento ai namespaces (spazi dei nomi) nelle librerie installati in precedenza:

    **Python**

    ```python
    # Add references
    from dotenv import load_dotenv
    from azure.identity import DefaultAzureCredential
    from azure.ai.projects import AIProjectClient
    from azure.ai.inference.models import (
        SystemMessage,
        UserMessage,
        TextContentItem,
        AudioContentItem,
        InputAudio,
        AudioContentFormat,
    )
    ```

    **C#**

    ```csharp
    // Add references
    using Azure.Identity;
    using Azure.AI.Projects;
    using Azure.AI.Inference;
    ```

1. Nella funzione **principale**, sotto il commento **Ottieni impostazioni di configurazione**, notare che il codice carica i valori della stringa di connessione del progetto e del nome della distribuzione modello definiti nel file di configurazione.

1. Trovare il commento **Inizializza il client del progetto**, aggiungere il codice seguente per connettersi al progetto Fonderia Azure AI usando le credenziali di Azure con cui si è attualmente eseguito l'accesso:

    **Python**

    ```python
    # Initialize the project client
    project_client = AIProjectClient.from_connection_string(
        conn_str=project_connection,
        credential=DefaultAzureCredential())
    ```

    **C#**

    ```csharp
    // Initialize the project client
    var projectClient = new AIProjectClient(project_connection,
                        new DefaultAzureCredential());
    ```

1. Trovare il commento **Ottieni un client di chat**, aggiungere il seguente codice per creare un oggetto client per chattare con il modello:

    **Python**

    ```python
    # Get a chat client
    chat_client = project_client.inference.get_chat_completions_client(model=model_deployment)
    ```

    **C#**

    ```csharp
    // Get a chat client
    ChatCompletionsClient chat = projectClient.GetChatCompletionsClient();
    ```
    

### Scrivere codice per inviare un prompt basato su audio e sull'URL

1. Nell'editor di codice del file **audio-chat.py**, nella sezione del ciclo, sotto il commento **Ottieni una risposta all'input audio**, aggiungere il seguente codice per inviare un prompt che includa il seguente audio:

    <video controls src="../media/manzanas.mp4" title="Richiesta di mele" width="150"></video>

    **Python**

    ```python
    # Get a response to audio input
    file_path = "https://github.com/microsoftlearning/mslearn-ai-language/raw/refs/heads/main/labfiles/09-audio-chat/data/manzanas.mp3"
    response = chat_client.complete(
        messages=[
            SystemMessage(system_message),
            UserMessage(
                [
                    TextContentItem(text=prompt),
                    {
                        "type": "audio_url",
                        "audio_url": {"url": file_path}
                    }
                ]
            )
        ]
    )
    print(response.choices[0].message.content)
    ```

    **C#**

    ```csharp
    // Get a response to audio input
    string audioUrl = "https://github.com/microsoftlearning/mslearn-ai-language/raw/refs/heads/main/labfiles/09-audio-chat/data/manzanas.mp3";
    var requestOptions = new ChatCompletionsOptions()
    {
        Messages =
        {
            new ChatRequestSystemMessage(system_message),
            new ChatRequestUserMessage(
                new ChatMessageTextContentItem(prompt),
                new ChatMessageAudioContentItem(new Uri(audioUrl))),
        },
        Model = model_deployment
    };
    var response = chat.Complete(requestOptions);
    Console.WriteLine(response.Value.Content);
    ```

1. Usare il comando **CTRL+S** per salvare le modifiche apportate al file di codice. È anche possibile chiudere l'editor di codice (**CTRL+Q**) se si desidera.

1. Nel riquadro della riga di comando di Cloud Shell, sotto l'editor di codice, immettere il seguente comando per eseguire l'applicazione:

    **Python**

    ```
    python audio-chat.py
    ```

    **C#**

    ```
    dotnet run
    ```

1. Quando richiesto, immettere il prompt `What is this customer saying in English?`

1. Rivedere la risposta.

### Usare un prompt diverso

1. Nell'editor di codice del codice dell'app, individuare il codice aggiunto in precedenza sotto il commento **Ottieni una risposta all'input immagine**. Modificare quindi il codice come segue per selezionare un file audio diverso:

    <video controls src="../media/caomei.mp4" title="Richiesta di fragole" width="150"></video>

    **Python**

    ```python
    # Get a response to audio input
    file_path = "https://github.com/microsoftlearning/mslearn-ai-language/raw/refs/heads/main/labfiles/09-audio-chat/data/caomei.mp3"
    response = chat_client.complete(
        messages=[
            SystemMessage(system_message),
            UserMessage(
                [
                    TextContentItem(text=prompt),
                    {
                        "type": "audio_url",
                        "audio_url": {"url": file_path}
                    }
                ]
            )
        ]
    )
    print(response.choices[0].message.content)
    ```

    **C#**

    ```csharp
    // Get a response to audio input
    string audioUrl = "https://github.com/microsoftlearning/mslearn-ai-language/raw/refs/heads/main/labfiles/09-audio-chat/data/caomei.mp3";
    var requestOptions = new ChatCompletionsOptions()
    {
        Messages =
        {
            new ChatRequestSystemMessage(system_message),
            new ChatRequestUserMessage(
                new ChatMessageTextContentItem(prompt),
                new ChatMessageAudioContentItem(new Uri(audioUrl))),
        },
        Model = model_deployment
    };
    var response = chat.Complete(requestOptions);
    Console.WriteLine(response.Value.Content);
    ```

1. Usare il comando **CTRL+S** per salvare le modifiche apportate al file di codice. È anche possibile chiudere l'editor di codice (**CTRL+Q**) se si desidera.

1. Nel riquadro della riga di comando di Cloud Shell, sotto l'editor di codice, inserire il seguente comando per eseguire l'applicazione:

    **Python**

    ```
    python audio-chat.py
    ```

    **C#**

    ```
    dotnet run
    ```

1. Quando richiesto, immettere la richiesta seguente:

    ```
    A customer left this voice message, can you summarize it?
    ```

1. Rivedere la risposta. Immettere quindi `quit` per uscire dal programma.

    > **Nota**: in questa semplice app non è stata implementata la logica per conservare la cronologia delle conversazioni, quindi il modello considererà ogni prompt come una nuova richiesta, senza il contesto del prompt precedente.

1. È possibile continuare a eseguire l'applicazione, selezionando e testando diversi tipi di prompt. Al termine, immettere `quit` per uscire dal programma.

    Se si dispone di tempo, è possibile modificare il codice per usare un prompt di sistema diverso e propri file audio accessibili via Internet.

    > **Nota**: in questa semplice app non è stata implementata la logica per conservare la cronologia delle conversazioni, quindi il modello considererà ogni prompt come una nuova richiesta, senza il contesto del prompt precedente.

## Riepilogo

In questo esercizio sono stati usati SDK Fonderia Azure AI e Inferenza Azure AI per sviluppare un'applicazione client basata su un modello multimodale, in grado di generare risposte ad audio.

## Eseguire la pulizia

Al termine dell'esplorazione di Fonderia Azure AI, è importante eliminare le risorse create durante l'esercizio per evitare costi di Azure non necessari.

1. Tornare alla scheda del browser che contiene il portale di Azure (o riaprire il [portale di Azure](https://portal.azure.com) su `https://portal.azure.com` in una nuova scheda del browser) e visualizzare il contenuto del gruppo di risorse in cui sono state distribuite le risorse usate in questo esercizio.
1. Sulla barra degli strumenti selezionare **Elimina gruppo di risorse**.
1. Immettere il nome del gruppo di risorse e confermarne l'eliminazione.
