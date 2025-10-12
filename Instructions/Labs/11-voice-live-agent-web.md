---
lab:
  title: Sviluppare un agente vocale di Azure AI Voice Live
  description: Informazioni su come creare un'app Web per abilitare le interazioni vocali in tempo reale con un agente di Azure AI Voice Live.
---

# Sviluppare un agente vocale di Azure AI Voice Live

In questo esercizio si completa un'app Web Python basata su Flask che consente interazioni vocali in tempo reale con un agente. Si aggiunge il codice per inizializzare la sessione e gestire gli eventi di sessione. Si usa uno script di distribuzione che: distribuisce il modello di intelligenza artificiale, crea un'immagine dell'app in Registro Azure Container usando le attività del Registro Azure Container e quindi crea un'istanza del Servizio app di Azure che esegue il pull dell'immagine. Per testare l'app è necessario un dispositivo audio con funzionalità di microfono e altoparlante.

Anche se questo esercizio è basato su Python, è possibile sviluppare applicazioni simili usando altri SDK specifici del linguaggio, tra cui:

- [Libreria client di Azure VoiceLive per .NET](https://www.nuget.org/packages/Azure.AI.VoiceLive/)

Attività eseguite in questo esercizio:

* Scaricare i file di base per l'app
* Aggiungere codice per completare l'app Web
* Esaminare la codebase complessiva
* Aggiornare ed eseguire lo script di distribuzione
* Visualizzare e testare l'applicazione

Questo esercizio richiede circa **30** minuti.

## Avviare Azure Cloud Shell e scaricare i file

In questa sezione dell'esercizio si scarica un file compresso contenente i file di base per l'app.

1. Nel browser passare al portale di Azure [https://portal.azure.com](https://portal.azure.com). Accedere con le credenziali di Azure, se richiesto.

1. Usare il pulsante **[\>_]** a destra della barra di ricerca, nella parte superiore della pagina, per aprire una nuova sessione di Cloud Shell nel portale di Azure selezionando un ambiente ***Bash***. Cloud Shell fornisce un'interfaccia della riga di comando in un riquadro nella parte inferiore del portale di Azure.

    > **Nota**: se in precedenza è stata creata una sessione Cloud Shell che usa un ambiente *PowerShell*, passare a ***Bash***.

1. Nella barra degli strumenti di Cloud Shell scegliere **Vai alla versione classica** dal menu **Impostazioni**. Questa operazione è necessaria per usare l'editor di codice.

1. Eseguire il comando seguente nella shell **Bash** per scaricare e decomprimere i file dell'esercizio. Il secondo comando passerà anche alla directory per i file dell'esercizio.

    ```bash
    wget https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/downloads/python/voice-live-web.zip
    ```

    ```
    unzip voice-live-web.zip && cd voice-live-web
    ```

## Aggiungere codice per completare l'app Web

Quando i file dell'esercizio sono stati scaricati, il passaggio successivo consiste nell'aggiungere codice per completare l'applicazione. I passaggi seguenti vengono eseguiti in Cloud Shell. 

>**Suggerimento:** Ridimensionare Cloud Shell per visualizzare altre informazioni e il codice trascinando il bordo superiore. È anche possibile usare i pulsanti Riduci a icona e Ingrandisci per spostarsi tra Cloud Shell e l'interfaccia principale del portale.

Eseguire il comando seguente per passare alla directory *src* prima di continuare con l'esercizio.

```bash
cd src
```

### Aggiungere il codice per implementare l'assistente di Voice Live

In questa sezione si aggiunge codice per implementare l'assistente di Voice Live. Il metodo **\_\_init\_\_** inizializza l'assistente vocale archiviando i parametri di connessione di Azure VoiceLive (endpoint, credenziali, modello, voce e istruzioni di sistema) e configurando le variabili di stato del runtime per gestire il ciclo di vita della connessione e gestire le interruzioni utente durante le conversazioni. Il metodo **start** importa i componenti necessari di Azure VoiceLive SDK che verranno usati per stabilire la connessione WebSocket e configurare la sessione vocale in tempo reale.

1. Eseguire il comando seguente per aprire il file *flask_app.py* per la modifica.

    ```bash
    code flask_app.py
    ```

1. Cercare il commento **# BEGIN VOICE LIVE ASSISTANT IMPLEMENTATION - ALIGN CODE WITH COMMENT** nel codice. Copiare il codice seguente e immetterlo subito sotto il commento. Assicurarsi di controllare il rientro.

    ```python
    def __init__(
        self,
        endpoint: str,
        credential,
        model: str,
        voice: str,
        instructions: str,
        state_callback=None,
    ):
        # Store Azure Voice Live connection and configuration parameters
        self.endpoint = endpoint
        self.credential = credential
        self.model = model
        self.voice = voice
        self.instructions = instructions
        
        # Initialize runtime state - connection established in start()
        self.connection = None
        self._response_cancelled = False  # Used to handle user interruptions
        self._stopping = False  # Signals graceful shutdown
        self.state_callback = state_callback or (lambda *_: None)

    async def start(self):
        # Import Voice Live SDK components needed for establishing connection and configuring session
        from azure.ai.voicelive.aio import connect  # type: ignore
        from azure.ai.voicelive.models import (
            RequestSession,
            ServerVad,
            AzureStandardVoice,
            Modality,
            InputAudioFormat,
            OutputAudioFormat,
        )  # type: ignore
    ```

1. Immettere **CTRL+S** per salvare le modifiche e mantenere aperto l'editor per la sezione successiva.

### Aggiungere il codice per implementare l'assistente di Voice Live

In questa sezione si aggiunge codice per configurare la sessione di Voice Live. Specifica le modalità (l'opzione solo audio non è supportata dall'API), le istruzioni di sistema che definiscono il comportamento dell'assistente, la voce di Sintesi vocale di Azure per le risposte, il formato audio sia per i flussi di input che per i flussi di output e il rilevamento attività vocale lato server che specifica la modalità con cui il modello rileva quando gli utenti iniziano e smettono di parlare.

1. Cercare il commento **# BEGIN CONFIGURE VOICE LIVE SESSION - ALIGN CODE WITH COMMENT** nel codice. Copiare il codice seguente e immetterlo subito sotto il commento. Assicurarsi di controllare il rientro.

    ```python
    # Configure VoiceLive session with audio/text modalities and voice activity detection
    session_config = RequestSession(
        modalities=[Modality.TEXT, Modality.AUDIO],
        instructions=self.instructions,
        voice=voice_cfg,
        input_audio_format=InputAudioFormat.PCM16,
        output_audio_format=OutputAudioFormat.PCM16,
        turn_detection=ServerVad(threshold=0.5, prefix_padding_ms=300, silence_duration_ms=500),
    )
    await conn.session.update(session=session_config)
    ```

1. Immettere **CTRL+S** per salvare le modifiche e mantenere aperto l'editor per la sezione successiva.

### Aggiungere codice per gestire gli eventi di sessione

In questa sezione si aggiunge codice per aggiungere gestori eventi per la sessione di Voice Live. I gestori eventi rispondono agli eventi chiave della sessione di VoiceLive durante il ciclo di vita della conversazione: **_handle_session_updated** segnala quando la sessione è pronta per l'input dell'utente,**_handle_speech_started** rileva quando l'utente inizia a parlare e implementa la logica di interruzione arrestando qualsiasi riproduzione audio dell'assistente in corso e annullando le risposte in corso per consentire il flusso naturale della conversazione e **_handle_speech_stopped** gestisce il momento in cui l'utente ha smesso di parlare e l'assistente inizia a elaborare l'input.

1. Cercare il commento **# BEGIN HANDLE SESSION EVENTS - ALIGN CODE WITH COMMENT** nel codice. Copiare il codice seguente e immetterlo subito sotto il commento. Assicurarsi di controllare il rientro.

    ```python
    async def _handle_event(self, event, conn, verbose=False):
        """Handle Voice Live events with clear separation by event type."""
        # Import event types for processing different Voice Live server events
        from azure.ai.voicelive.models import ServerEventType
        
        event_type = event.type
        if verbose:
            _broadcast({"type": "log", "level": "debug", "event_type": str(event_type)})
        
        # Route Voice Live server events to appropriate handlers
        if event_type == ServerEventType.SESSION_UPDATED:
            await self._handle_session_updated()
        elif event_type == ServerEventType.INPUT_AUDIO_BUFFER_SPEECH_STARTED:
            await self._handle_speech_started(conn)
        elif event_type == ServerEventType.INPUT_AUDIO_BUFFER_SPEECH_STOPPED:
            await self._handle_speech_stopped()
        elif event_type == ServerEventType.RESPONSE_AUDIO_DELTA:
            await self._handle_audio_delta(event)
        elif event_type == ServerEventType.RESPONSE_AUDIO_DONE:
            await self._handle_audio_done()
        elif event_type == ServerEventType.RESPONSE_DONE:
            # Reset cancellation flag but don't change state - _handle_audio_done already did
            self._response_cancelled = False
        elif event_type == ServerEventType.ERROR:
            await self._handle_error(event)

    async def _handle_session_updated(self):
        """Session is ready for conversation."""
        self.state_callback("ready", "Session ready. You can start speaking now.")

    async def _handle_speech_started(self, conn):
        """User started speaking - handle interruption if needed."""
        self.state_callback("listening", "Listening… speak now")
        
        try:
            # Stop any ongoing audio playback on the client side
            _broadcast({"type": "control", "action": "stop_playback"})
            
            # If assistant is currently speaking or processing, cancel the response to allow interruption
            current_state = assistant_state.get("state")
            if current_state in {"assistant_speaking", "processing"}:
                self._response_cancelled = True
                await conn.response.cancel()
                _broadcast({"type": "log", "level": "debug", 
                          "msg": f"Interrupted assistant during {current_state}"})
            else:
                _broadcast({"type": "log", "level": "debug", 
                          "msg": f"User speaking during {current_state} - no cancellation needed"})
        except Exception as e:
            _broadcast({"type": "log", "level": "debug", 
                      "msg": f"Exception in speech handler: {e}"})

    async def _handle_speech_stopped(self):
        """User stopped speaking - processing input."""
        self.state_callback("processing", "Processing your input…")

    async def _handle_audio_delta(self, event):
        """Stream assistant audio to clients."""
        if self._response_cancelled:
            return  # Skip cancelled responses
            
        # Update state when assistant starts speaking
        if assistant_state.get("state") != "assistant_speaking":
            self.state_callback("assistant_speaking", "Assistant speaking…")
        
        # Extract and broadcast Voice Live audio delta as base64 to WebSocket clients
        audio_data = getattr(event, "delta", None)
        if audio_data:
            audio_b64 = base64.b64encode(audio_data).decode("utf-8")
            _broadcast({"type": "audio", "audio": audio_b64})

    async def _handle_audio_done(self):
        """Assistant finished speaking."""
        self._response_cancelled = False
        self.state_callback("ready", "Assistant finished. You can speak again.")

    async def _handle_error(self, event):
        """Handle Voice Live errors."""
        error = getattr(event, "error", None)
        message = getattr(error, "message", "Unknown error") if error else "Unknown error"
        self.state_callback("error", f"Error: {message}")

    def request_stop(self):
        self._stopping = True
    ```

1. Immettere **CTRL+S** per salvare le modifiche e mantenere aperto l'editor per la sezione successiva.

### Esaminare il codice nell'app

Finora è stato aggiunto codice all'app per implementare l'agente e gestire gli eventi dell'agente. Esaminare il codice completo e i commenti per comprendere meglio il modo in cui l'app gestisce lo stato e le operazioni del client.

1. Al termine, immettere **CTRL+Q** per uscire dall'editor. 

## Aggiornare ed eseguire lo script di distribuzione

In questa sezione si apporta una piccola modifica allo script di distribuzione **azdeploy.sh** e quindi si esegue la distribuzione. 

### Aggiornare lo script di distribuzione

Nella parte superiore dello script di distribuzione **azdeploy.sh** è necessario modificare solo due valori. 

* Il valore **rg** specifica il gruppo di risorse che deve contenere la distribuzione. È possibile accettare il valore predefinito oppure immettere un valore personalizzato, se è necessario eseguire la distribuzione in un gruppo di risorse specifico.

* Il valore **location** imposta l'area per la distribuzione. Il modello *gpt-4o* usato nell'esercizio può essere distribuito in altre aree, ma possono essere previsti limiti in un'area specifica. Se la distribuzione ha esito negativo nell'area scelta, provare **eastus2** o **sveziacentral**. 

    ```
    rg="rg-voicelive" # Replace with your resource group
    location="eastus2" # Or a location near you
    ```

1. Eseguire i comandi seguenti in Cloud Shell per iniziare a modificare lo script di distribuzione.

    ```bash
    cd ~/voice-live-web
    ```
    
    ```bash
    code azdeploy.sh
    ```

1. Aggiornare i valori per **rg** e **location** in base alle proprie esigenze e quindi premere **CTRL+S** per salvare le modifiche e **CTRL+Q** per uscire dall'editor.

### Eseguire lo script di distribuzione

Lo script di distribuzione distribuisce il modello di intelligenza artificiale e crea le risorse necessarie in Azure per eseguire un'app in contenitori nel Servizio app.

1. Eseguire il comando seguente in Cloud Shell per iniziare a distribuire le risorse di Azure e l'applicazione.

    ```bash
    bash azdeploy.sh
    ```

1. Selezionare l'**opzione 1** per la distribuzione iniziale.

    Il completamento della distribuzione dovrebbe richiedere 5-10 minuti. Durante la distribuzione potrebbero essere richieste le informazioni o le azioni seguenti:
    
    * Se viene richiesto di eseguire l'autenticazione in Azure, seguire le istruzioni visualizzate.
    * Se viene richiesto di selezionare una sottoscrizione, usare i tasti di direzione per evidenziare la sottoscrizione e premere **INVIO**. 
    * È probabile che vengano visualizzati alcuni avvisi durante la distribuzione. Questi avvisi possono essere ignorati.
    * Se la distribuzione ha esito negativo durante la distribuzione del modello di intelligenza artificiale, modificare l'area nello script di distribuzione e riprovare. 
    * A volte le aree in Azure possono risultare occupate e interferire con la tempistica delle distribuzioni. Se la distribuzione ha esito negativo dopo la distribuzione del modello, eseguire nuovamente lo script di distribuzione.

## Visualizzare e testare l'app

Al termine della distribuzione, un messaggio di tipo "Distribuzione completata" viene mostrato nella shell, insieme a un collegamento all'app Web. È possibile selezionare il collegamento oppure passare alla risorsa del Servizio app e avviare l'app da questa posizione. Il caricamento dell'applicazione può richiedere alcuni minuti. 

1. Selezionare il pulsante **Avvia sessione** per connettersi al modello.
1. Verrà richiesto di concedere all'applicazione l'accesso ai dispositivi audio.
1. Iniziare a parlare con il modello quando l'app chiede di iniziare a parlare.

Risoluzione dei problemi:

* Se l'app segnala variabili di ambiente mancanti, riavviare l'applicazione nel Servizio app.
* Se il log mostrato nell'applicazione include messaggi relativi a un numero eccessivo di *blocchi di audio*, selezionare **Arresta sessione** e quindi riavviare la sessione. 
* Se l'app non funziona, verificare che sia stato aggiunto tutto il codice e che il rientro corretto. Se è necessario apportare di nuovo modifiche, eseguire di nuovo la distribuzione e selezionare l'**opzione 2** per aggiornare solo l'immagine.

## Pulire le risorse

Eseguire il comando seguente in Cloud Shell per rimuovere tutte le risorse distribuite per questo esercizio. Verrà richiesto di confermare l'eliminazione della risorsa.

```
azd down --purge
```