---
lab:
  title: Sviluppare un'app di chat abilitata per gli audio
  description: Informazioni su come usare Fonderia Azure AI per creare un'app di intelligenza artificiale generativa che supporta input audio.
---

# Sviluppare un'app di chat abilitata per gli audio

In questo esercizio, verrà usato il modello di intelligenza artificiale generativa *Phi-4-multimodal-instruct* per generare risposte a richieste che includono file audio. Verrà sviluppata un'app con l'assistenza dell'intelligenza artificiale per una società di produzione usando Fonderia Azure AI e l'SDK OpenAI di Python per riepilogare i messaggi vocali lasciati dai clienti.

Anche se questo esercizio è basato su Python, è possibile sviluppare applicazioni simili usando più SDK specifici del linguaggio, tra cui:

- [Progetti di Azure AI per Python](https://pypi.org/project/azure-ai-projects)
- [Libreria OpenAI per Python](https://pypi.org/project/openai/)
- [Progetti di Azure AI per Microsoft .NET](https://www.nuget.org/packages/Azure.AI.Projects)
- [Libreria client di Azure OpenAI per Microsoft .NET](https://www.nuget.org/packages/Azure.AI.OpenAI)
- [Progetti di Azure AI per JavaScript](https://www.npmjs.com/package/@azure/ai-projects)
- [Libreria Azure OpenAI per TypeScript](https://www.npmjs.com/package/@azure/openai)

Questo esercizio richiede circa **30** minuti.

## Creare un progetto Fonderia Azure AI

Per iniziare, distribuire un modello in un progetto Fonderia Azure AI.

1. In un Web browser, aprire il [Portale Fonderia Azure AI](https://ai.azure.com) su `https://ai.azure.com` e accedere usando le credenziali di Azure. Chiudere i suggerimenti o i riquadri di avvio rapido aperti la prima volta che si accede e, se necessario, usare il logo **Fonderia Azure AI** in alto a sinistra per passare alla home page, simile all'immagine seguente:

    ![Screenshot del portale di Azure AI Foundry.](../media/ai-foundry-home.png)

1. Nella home page, nella sezione **Esplora modelli e funzionalità** cercare il modello `Phi-4-multimodal-instruct`, che verrà usato nel progetto.
1. Nei risultati della ricerca selezionare il modello **Phi-4-multimodal-instruct** per visualizzarne i dettagli e quindi nella parte superiore della pagina per il modello selezionare **Usa questo modello**.
1. Quando viene richiesto di creare un progetto, immettere un nome valido per il progetto ed espandere **Opzioni avanzate**.
1. Selezionare **Personalizza** e specificare le impostazioni seguenti per l'hub:
    - **Risorsa di Fonderia Azure AI**: *nome valido per la risorsa di Fonderia Azure AI*
    - **Sottoscrizione**: *la sottoscrizione di Azure usata*
    - **Gruppo di risorse**: *creare o selezionare un gruppo di risorse*
    - **Area**: *selezionare una delle **opzioni consigliate di Fonderia AI***\*

    > \* Alcune risorse Azure AI sono limitate da quote di modelli regionali. In caso di superamento di un limite di quota più avanti nell'esercizio, potrebbe essere necessario creare un'altra risorsa in un'area diversa. È possibile controllare la disponibilità a livello di area più recente per modelli specifici nella [documentazione di Fonderia Azure AI](https://learn.microsoft.com/azure/ai-foundry/how-to/deploy-models-serverless-availability#region-availability).

1. Selezionare **Crea** e attendere che venga creato il progetto.

    Il completamento dell'operazione potrebbe richiedere alcuni minuti.

1. Selezionare **Accetta e continua** per accettare le condizioni del modello, quindi selezionare **Distribuisci** per completare la distribuzione del modello Phi.

1. Una volta creato il progetto, i dettagli del modello verranno aperti automaticamente. Prendere nota del nome della distribuzione del modello, che dovrebbe essere **Phi-4-multimodal-instruct**

1. Nel riquadro di spostamento a sinistra, selezionare **Panoramica** per visualizzare la pagina principale del progetto, che avrà questo aspetto:

    > **Nota**: se viene visualizzato un errore *Autorizzazioni insufficienti**, usare il pulsante **Correggi** per risolverlo.

    ![Screenshot della pagina di panoramica del progetto Fonderia Azure AI.](../media/ai-foundry-project.png)

## Creare un'applicazione client

Ora che è stato distribuito un modello, è possibile usare gli SDK Fonderia Azure AI e Inferenza del modello Azure AI per sviluppare un'applicazione che chatti con esso.

> **Suggerimento**: è possibile scegliere di sviluppare la soluzione usando Python o Microsoft C#. Seguire le istruzioni nella sezione appropriata per la lingua scelta.

### Preparare la configurazione dell'applicazione

1. Nel Portale Fonderia Azure AI visualizzare la pagina **Panoramica** per il progetto.
1. Nell'area **Dettagli di progetto** prendere nota dell'**endpoint del progetto Fonderia Azure AI**. Questo endpoint verrà usato per connettersi al progetto in un'applicazione client.
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
   git clone https://github.com/MicrosoftLearning/mslearn-ai-language
    ```

    > **Suggerimento**: quando si incollano i comandi in CloudShell, l'ouput può richiedere una grande quantità di buffer dello schermo. È possibile cancellare la schermata immettendo il `cls` comando per rendere più semplice concentrarsi su ogni attività.

1. Dopo aver clonato il repository, passare alla cartella contenente i file di codice dell'applicazione:  

    ```
   cd mslearn-ai-language/Labfiles/09-audio-chat/Python
    ````

1. Nel riquadro della riga di comando di Cloud Shell immettere il comando seguente per installare le librerie che si useranno, ovvero:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-identity azure-ai-projects openai
    ```

1. Immettere il comando seguente per modificare il file di configurazione fornito:

    ```
   code .env
    ```

    Il file deve essere aperto in un editor di codice.

1. Nel file di codice sostituire il segnaposto **your_project_endpoint** con l'endpoint del progetto (copiato dalla pagina **Panoramica** del progetto nel Portale Fonderia Azure AI) e il segnaposto **your_model_deployment** con il nome assegnato alla distribuzione del modello Phi-4-multimodal-instruct.

1. Dopo aver sostituito i segnaposto con l'editor di codice, usare il comando **CTRL+S** o **Fare clic con il pulsante destro del mouse > Salva** per salvare le modifiche e quindi usare il comando **CTRL+Q** o **Fare clic con il pulsante destro del mouse > Esci** per chiudere l'editor di codice mantenendo aperta la riga di comando di Cloud Shell.

### Scrivere il codice per connettersi al progetto e ottenere un client di chat per il modello

> **Suggerimento**: quando si aggiunge codice, assicurarsi di mantenere il rientro corretto.

1. Immettere il comando seguente per modificare il file di codice:

    ```
   code audio-chat.py
    ```

1. Nel file di codice prendere nota delle istruzioni esistenti aggiunte all'inizio del file per importare i namespaces (spazi dei nomi) SDK necessari. Quindi, trovare il commento **Aggiungi riferimenti** e aggiungere il codice seguente per fare riferimento ai namespaces (spazi dei nomi) nelle librerie installati in precedenza:

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
    ```

1. Nella funzione **principale**, sotto il commento **Ottieni impostazioni di configurazione**, notare che il codice carica i valori della stringa di connessione del progetto e del nome della distribuzione modello definiti nel file di configurazione.

1. Trovare il commento **Inizializza il client del progetto** e aggiungere il codice seguente per connettersi al progetto di Fonderia Azure AI:

    > **Suggerimento**: prestare attenzione a mantenere il livello di rientro corretto per il codice.

    ```python
   # Initialize the project client
   project_client = AIProjectClient(            
       credential=DefaultAzureCredential(
           exclude_environment_credential=True,
           exclude_managed_identity_credential=True
       ),
       endpoint=project_endpoint,
   )
    ```

1. Trovare il commento **Ottieni un client di chat**, aggiungere il seguente codice per creare un oggetto client per chattare con il modello:

    ```python
   # Get a chat client
   openai_client = project_client.get_openai_client(api_version="2024-10-21")
    ```

### Scrivere il codice per inviare un prompt basato su audio

Prima di inviare il prompt, è necessario codificare il file audio per la richiesta. È quindi possibile allegare i dati audio al messaggio dell'utente con un prompt per l'LLM. Si noti che il codice include un ciclo per consentire all'utente di immettere un prompt fino a quando non immette "esci". 

1. Sotto il commento **Codifica file audio**, immettere il codice seguente per preparare il file audio seguente:

    <video controls src="https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Instructions/media/avocados.mp4" title="Una richiesta di avocado" width="150"></video>

    ```python
   # Encode the audio file
   file_path = "https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Labfiles/09-audio-chat/data/avocados.mp3"
   response = requests.get(file_path)
   response.raise_for_status()
   audio_data = base64.b64encode(response.content).decode('utf-8')
    ```

1. Sotto il commento **Ottieni una risposta all'input audio** aggiungere il codice seguente per inviare un prompt:

    ```python
   # Get a response to audio input
   response = openai_client.chat.completions.create(
       model=model_deployment,
       messages=[
           {"role": "system", "content": system_message},
           { "role": "user",
               "content": [
               { 
                   "type": "text",
                   "text": prompt
               },
               {
                   "type": "input_audio",
                   "input_audio": {
                       "data": audio_data,
                       "format": "mp3"
                   }
               }
           ] }
       ]
   )
   print(response.choices[0].message.content)
    ```

1. Usare il comando **CTRL+S** per salvare le modifiche apportate al file di codice. È anche possibile chiudere l'editor di codice (**CTRL+Q**) se si desidera.

### Accedere in Azure ed eseguire l'app

1. Nel riquadro della riga di comando di Cloud Shell immettere il comando seguente per accedere ad Azure.

    ```
   az login
    ```

    **<font color="red">È necessario accedere ad Azure, anche se la sessione di Cloud Shell è già autenticata.</font>**

    > **Nota**: nella maggior parte degli scenari, il semplice uso di *az login* sarà sufficiente. Tuttavia, in caso di sottoscrizioni in più tenant, potrebbe essere necessario specificare il tenant usando il parametro *--tenant*. Per dettagli, visualizzare [Accedere ad Azure in modo interattivo usando l'interfaccia della riga di comando di Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively).
    
1. Quando richiesto, seguire le istruzioni per aprire la pagina di accesso in una nuova scheda e immettere il codice di autenticazione fornito e le credenziali di Azure. Completare quindi il processo di accesso nella riga di comando, selezionando la sottoscrizione contenente l'hub di Fonderia Azure AI, se richiesto.

1. Nel riquadro della riga di comando di Cloud Shell immettere il comando seguente per eseguire l'app:

    ```
   python audio-chat.py
    ```

1. Quando richiesto, immettere il prompt 

    ```
   Can you summarize this customer's voice message?
    ```

1. Esaminare la risposta.

### Usare un file audio diverso

1. Nell'editor di codice del codice dell'app, individuare il codice aggiunto in precedenza sotto il commento **Codifica file audio**. Modificare quindi l'URL del percorso del file come segue per usare un file audio diverso per la richiesta (lasciando il codice esistente dopo il percorso del file):

    ```python
   # Encode the audio file
   file_path = "https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Labfiles/09-audio-chat/data/fresas.mp3"
    ```

    Il nuovo file è simile al seguente:

    <video controls src="https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Instructions/media/fresas.mp4" title="Richiesta di fragole" width="150"></video>

 1. Usare il comando **CTRL+S** per salvare le modifiche apportate al file di codice. È anche possibile chiudere l'editor di codice (**CTRL+Q**) se si desidera.

1. Nel riquadro della riga di comando di Cloud Shell, sotto l'editor di codice, inserire il seguente comando per eseguire l'applicazione:

    ```
   python audio-chat.py
    ```

1. Quando richiesto, immettere la richiesta seguente: 
    
    ```
   Can you summarize this customer's voice message? Is it time-sensitive?
    ```

1. Esaminare la risposta. Immettere quindi `quit` per uscire dal programma.

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
