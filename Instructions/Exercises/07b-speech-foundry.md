---
lab:
  title: Riconoscimento e sintesi vocale (versione Azure AI Fonderia)
  module: Module 4 - Create speech-enabled apps with Azure AI services
---

<!--
Possibly update to use standalone AI Service instead of Foundry?
-->

# Riconoscimento e sintesi vocale

**Voce di Azure AI** è un servizio che fornisce funzionalità correlate al riconoscimento vocale, tra cui:

- Un'API di *conversione della voce in testo scritto* che consente di implementare il riconoscimento vocale (convertendo parole vocali udibili in testo).
- Un'API *sintesi vocale* che consente di implementare la sintesi vocale (convertendo il testo in voce udibile).

In questo esercizio si useranno entrambe queste API per implementare un'applicazione di orologio vocale.

> **NOTA** Questo esercizio è progettato per essere completato in Azure Cloud Shell, in cui l'accesso diretto all'hardware audio del computer non è supportato. Il lab userà quindi i file audio per i flussi di input e output vocale. Il codice per ottenere gli stessi risultati usando un microfono e un altoparlante viene fornito come riferimento.

## Creare un progetto Fonderia Azure AI

Per iniziare, creare un progetto Fonderia Azure AI.

1. In un Web browser, aprire il [Portale Fonderia Azure AI](https://ai.azure.com) su `https://ai.azure.com` e accedere usando le credenziali di Azure. Chiudere i suggerimenti o i riquadri di avvio rapido aperti la prima volta che si accede e, se necessario, usare il logo **Fonderia Azure AI** in alto a sinistra per passare alla home page, simile all'immagine seguente:

    ![Screenshot del portale di Azure AI Foundry.](./ai-foundry/media/ai-foundry-home.png)

1. Nella home page, selezionare **+ Crea progetto**.
1. Nella procedura guidata **Crea un progetto**, immettere un nome valido per il progetto. Se viene suggerito un hub esistente, selezionare l'opzione per crearne uno nuovo. Successivamente, esaminare le risorse Azure che verranno create automaticamente per supportare l'hub e il progetto.
1. Selezionare **Personalizza** e specificare le impostazioni seguenti per l'hub:
    - **Nome hub**: *un nome valido per l'hub*
    - **Sottoscrizione**: *la sottoscrizione di Azure usata*
    - **Gruppo di risorse**: *creare o selezionare un gruppo di risorse*
    - **Posizione**: scegliere una qualsiasi area disponibile
    - **Connettere Servizi di Azure AI o Azure OpenAI**: *Creare una nuova risorsa di Servizi di AI*
    - **Connettere Azure AI Search**: *creare una nuova risorsa di Azure AI Search con un nome univoco*

1. Selezionare **Avanti** per esaminare la configurazione. Quindi selezionare **Crea** e attendere il completamento del processo.
1. Quando viene creato il progetto, chiudere tutti i suggerimenti visualizzati e rivedere la pagina del progetto nel portale Fonderia di Azure AI, che dovrebbe essere simile all'immagine seguente:

    ![Screenshot dei dettagli di un progetto di Azure AI nel portale Fonderia di Azure AI.](./ai-foundry/media/ai-foundry-project.png)

## Preparare e configurare l'app orologio parlante

1. Nel Portale Fonderia Azure AI visualizzare la pagina **Panoramica** per il progetto.
1. Nell'area **Dettagli progetto** prendere nota della **stringa di connessione del progetto** e della **posizione** del progetto. La stringa di connessione verrà utilizzata per connettersi al progetto in un'applicazione client e sarà necessaria la posizione per connettersi all'endpoint Voce di Servizi di Azure AI.
1. Aprire una nuova scheda del browser (mantenendo aperto il Portale Fonderia Azure AI nella scheda esistente). In una nuova scheda del browser, passare al [portale di Azure](https://portal.azure.com) su `https://portal.azure.com`, accedendo con le credenziali di Azure se richiesto.
1. Usare il pulsante **[\>_]** a destra della barra di ricerca, nella parte superiore della pagina, per aprire una nuova sessione di Cloud Shell nel portale di Azure selezionando un ambiente ***PowerShell***. Cloud Shell fornisce un'interfaccia della riga di comando in un riquadro nella parte inferiore del portale di Azure.

    > **Nota**: se in precedenza è stata creata una sessione Cloud Shell che usa un ambiente *Bash*, passare a ***PowerShell***.

1. Nella barra degli strumenti di Cloud Shell scegliere **Vai alla versione classica** dal menu **Impostazioni**. Questa operazione è necessaria per usare l'editor di codice.

    > **Suggerimento**: quando si incollano i comandi in CloudShell, l'ouput può richiedere una grande quantità di buffer dello schermo. È possibile cancellare la schermata immettendo il `cls` comando per rendere più semplice concentrarsi su ogni attività.

1. Nel riquadro PowerShell immettere i comandi seguenti per clonare il repository GitHub per questo esercizio:

    ```
   rm -r mslearn-ai-language -f
   git clone https://github.com/microsoftlearning/mslearn-ai-language mslearn-ai-language
    ```

    ***Ora seguire i passaggi per il linguaggio di programmazione scelto.***

1. Dopo aver clonato il repository, passare alla cartella contenente i file di codice dell'applicazione orologio parlante:  

    **Python**

    ```
   cd mslearn-ai-language/labfiles/07b-speech/python/speaking-clock
    ```

    **C#**

    ```
   cd mslearn-ai-language/labfiles/07b-speech/c-sharp/speaking-clock
    ```

1. Nel riquadro della riga di comando di Cloud Shell immettere il comando seguente per installare le librerie che si useranno, ovvero:

    **Python**

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects azure-cognitiveservices-speech==1.42.0
    ```

    **C#**

    ```
   dotnet add package Azure.Identity
   dotnet add package Azure.AI.Projects --prerelease
   dotnet add package Microsoft.CognitiveServices.Speech --version 1.42.0
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

    Il file viene aperto in un editor di codice.

1. Nel file di codice sostituire i segnaposti **your_project_endpoint** e **your_location** con la stringa di connessione e la posizione del progetto (copiate dalla pagina **Panoramica** del progetto nel Portale Fonderia Azure AI).
1. Dopo aver sostituito i segnaposto, usare il comando **CTRL+S** per salvare le modifiche e quindi usare il comando **CTRL+Q** per chiudere l'editor di codice mantenendo aperta la riga di comando di Cloud Shell.

## Aggiungere codice per usare l'SDK di Voce di Azure AI

> **Suggerimento**: quando si aggiunge codice, assicurarsi di mantenere il rientro corretto.

1. Immettere il comando seguente per modificare il file di codice fornito:

    **Python**

    ```
   code speaking-clock.py
    ```

    **C#**

    ```
   code Program.cs
    ```

1. Nella parte superiore del file di codice, sotto i riferimenti agli spazi dei nomi esistenti, individuare il commento **Import namespaces**. Sotto questo commento aggiungere quindi il codice seguente specifico della lingua per importare gli spazi dei nomi necessari per usare l'SDK di Voce di Azure AI con la risorsa di servizi Azure AI nel progetto Fonderia Azure AI:

    **Python**

    ```python
   # Import namespaces
   from dotenv import load_dotenv
   from azure.ai.projects.models import ConnectionType
   from azure.identity import DefaultAzureCredential
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.projects import AIProjectClient
   import azure.cognitiveservices.speech as speech_sdk
    ```

    **C#**

    ```csharp
   // Import namespaces
   using Azure.Identity;
   using Azure.AI.Projects;
   using Microsoft.CognitiveServices.Speech;
   using Microsoft.CognitiveServices.Speech.Audio;
    ```

1. Nella funzione **principale**, sotto il commento **Ottieni impostazioni di configurazione**, notare che il codice carica i valori della stringa di connessione del progetto e la posizione definiti nel file di configurazione.

1. Aggiungere il codice seguente sotto il commento **Ottenere l'endpoint Voce AI e la chiave dal progetto**:

    **Python**

    ```python
   # Get AI Services key from the project
   project_client = AIProjectClient.from_connection_string(
        conn_str=project_connection,
        credential=DefaultAzureCredential())

   ai_svc_connection = project_client.connections.get_default(
      connection_type=ConnectionType.AZURE_AI_SERVICES,
      include_credentials=True, 
    )

   ai_svc_key = ai_svc_connection.key

    ```

    **C#**

    ```csharp
   // Get AI Services key from the project
   var projectClient = new AIProjectClient(project_connection,
                        new DefaultAzureCredential());

   ConnectionResponse aiSvcConnection = projectClient.GetConnectionsClient().GetDefaultConnection(ConnectionType.AzureAIServices, true);

   var apiKeyAuthProperties = aiSvcConnection.Properties as ConnectionPropertiesApiKeyAuth;

   var aiSvcKey = apiKeyAuthProperties.Credentials.Key;
    ```

    Questo codice si connette al progetto Fonderia Azure AI, ottiene la risorsa connessa ai servizi di intelligenza artificiale predefinita e recupera la chiave di autenticazione necessaria per usarlo.

1. Sotto al commento **Configurare il servizio Voce**, aggiungere il codice seguente per usare la chiave dei servizi di intelligenza artificiale e l'area del progetto per configurare la connessione all'endpoint Voce di Servizi di Azure AI

   **Python**

    ```python
   # Configure speech service
   speech_config = speech_sdk.SpeechConfig(ai_svc_key, location)
   print('Ready to use speech service in:', speech_config.region)
    ```

    **C#**

    ```csharp
   // Configure speech service
   speechConfig = SpeechConfig.FromSubscription(aiSvcKey, location);
   Console.WriteLine("Ready to use speech service in " + speechConfig.Region);
    ```

1. Salvare le modifiche (*CTRL+S*), ma lasciare aperto l'editor di codice.

## Eseguire l'app

Finora, l'app non fa niente di diverso dalla connessione al progetto Fonderia Azure AI per recuperare i dettagli necessari per usare il servizio Voce, ma è utile eseguirla e verificare che funzioni prima di aggiungere la funzionalità Voce.

1. Nella riga di comando sotto l'editor di codice immettere il seguente comando dell'interfaccia della riga di comando di Azure per determinare l'account Azure che ha eseguito l'accesso per la sessione:

    ```
   az account show
    ```

    L'output JSON risultante deve includere i dettagli dell'account Azure e la sottoscrizione in uso (che deve corrispondere alla stessa sottoscrizione in cui è stato creato il progetto Fonderia Azure AI).

    Per autenticare la connessione al progetto l'app usa le credenziali di Azure per il contesto in cui viene eseguita. In un ambiente di produzione l'app potrebbe essere configurata per l'esecuzione usando un'identità gestita. In questo ambiente di sviluppo userà le credenziali autenticate della sessione Cloud Shell.

    > **Nota**: è possibile accedere ad Azure nell'ambiente di sviluppo usando il comando `az login` dell'interfaccia della riga di comando di Azure. In questo caso, Cloud Shell ha già eseguito l'accesso usando le credenziali di Azure con cui è stato eseguito l'accesso al portale, quindi l'accesso in modo esplicito non è necessario. Per altre informazioni sull'uso dell'interfaccia della riga di comando di Azure per l'autenticazione in Azure, consultare [Eseguire l'autenticazione ad Azure tramite l'interfaccia della riga di comando di Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli).

1. Nella riga di comando immettere il seguente comando specifico della lingua per eseguire l'app orologio parlante:

    **Python**

    ```
   python speaking-clock.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. Se si usa C#, è possibile ignorare eventuali avvisi relativi all'uso dell'operatore **await** nei metodi asincroni. Questo problema verrà risolto in un secondo momento. Il codice dovrebbe visualizzare l'area della risorsa del servizio Voce che verrà utilizzata dall'applicazione. Un'esecuzione riuscita indica che l'app è connessa al progetto Fonderia Azure AI e ha recuperato la chiave necessaria per usare il servizio Voce di Azure AI.

## Aggiungere codice per il riconoscimento vocale

È ora disponibile un elemento **SpeechConfig** per il servizio Voce nella risorsa Servizi di Azure AI nel progetto, è quindi possibile usare l'API di **conversione della voce in testo scritto** per riconoscere il parlato e trascriverlo in testo.

In questa procedura, l'input vocale viene acquisito da un file audio, che è possibile riprodurre qui:

<video controls src="ai-foundry/media/Time.mp4" title="What time is it?" width="150"></video>

1. Nella funzione **Main** si noti che il codice usa la funzione **TranscribeCommand** per accettare l'input vocale. Nella funzione **TranscribeCommand**, sotto il commento **Configure speech recognition**, aggiungere il codice appropriato seguente per creare un client **SpeechRecognizer** che può essere usato per riconoscere e trascrivere il parlato da un file audio:

    **Python**

    ```python
   # Configure speech recognition
   current_dir = os.getcwd()
   audioFile = current_dir + '/time.wav'
   audio_config = speech_sdk.AudioConfig(filename=audioFile)
   speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    ```

    **C#**

    ```csharp
   // Configure speech recognition
   string audioFile = "time.wav";
   using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
   using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    ```

1. Nella funzione **TranscribeCommand**, sotto il commento **Process speech input**, aggiungere il codice seguente per l'ascolto dell'input vocale, prestando attenzione a non sostituire il codice alla fine della funzione che restituisce il comando:

    **Python**

    ```python
   # Process speech input
   print("Listening...")
   speech = speech_recognizer.recognize_once_async().get()
   if speech.reason == speech_sdk.ResultReason.RecognizedSpeech:
       command = speech.text
       print(command)
   else:
       print(speech.reason)
       if speech.reason == speech_sdk.ResultReason.Canceled:
           cancellation = speech.cancellation_details
           print(cancellation.reason)
           print(cancellation.error_details)
    ```

    **C#**

    ```csharp
   // Process speech input
   Console.WriteLine("Listening...");
   SpeechRecognitionResult speech = await speechRecognizer.RecognizeOnceAsync();
   if (speech.Reason == ResultReason.RecognizedSpeech)
   {
       command = speech.Text;
       Console.WriteLine(command);
   }
   else
   {
       Console.WriteLine(speech.Reason);
       if (speech.Reason == ResultReason.Canceled)
       {
           var cancellation = CancellationDetails.FromResult(speech);
           Console.WriteLine(cancellation.Reason);
           Console.WriteLine(cancellation.ErrorDetails);
       }
   }
    ```

1. Salvare le modifiche (*CTRL+S*) e quindi nella riga di comando sotto l'editor di codice immettere il seguente comando per eseguire il programma:

    **Python**

    ```
   python speaking-clock.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. Esaminare l'output dell'applicazione, che dovrebbe "ascoltare" correttamente il parlato nel file audio e restituire una risposta appropriata (si noti che Azure Cloud Shell potrebbe essere in esecuzione in un server che si trova in un fuso orario diverso rispetto all'utente).

    > **Suggerimento**: se SpeechRecognizer rileva un errore, viene generato il risultato "Annullato". Il codice nell'applicazione visualizza quindi il messaggio di errore. La causa più probabile è un valore di area non corretto nel file di configurazione.

## Sintesi vocale

L'applicazione dell'orologio vocale accetta l'input vocale, ma in realtà non parla. Per risolvere questo problema, aggiungere codice per sintetizzare il parlato.

Ancora una volta, a causa delle limitazioni hardware di Cloud Shell, l'output vocale sintetizzato verrà diretto a un file.

1. Nella funzione **Main** per il programma si noti che il codice usa la funzione **TellTime** per indicare all'utente l'ora corrente.
1. Nella funzione **TellTime**, sotto il commento **Configure speech synthesis**, aggiungere il codice seguente per creare un client **SpeechSynthesizer** che può essere usato per generare l'output vocale:

    **Python**

    ```python
   # Configure speech synthesis
   output_file = "output.wav"
   speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
   audio_config = speech_sdk.audio.AudioConfig(filename=output_file)
   speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config,)
    ```

    **C#**

    ```csharp
   // Configure speech synthesis
   var outputFile = "output.wav";
   speechConfig.SpeechSynthesisVoiceName = "en-GB-RyanNeural";
   using var audioConfig = AudioConfig.FromWavFileOutput(outputFile);
   using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig, audioConfig);
    ```

1. Nella funzione **TellTime**, sotto il commento **Synthesize spoken output**, aggiungere il codice seguente per generare l'output vocale, prestando attenzione a non sostituire il codice alla fine della funzione che stampa la risposta:

    **Python**

    ```python
   # Synthesize spoken output
   speak = speech_synthesizer.speak_text_async(response_text).get()
   if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
       print(speak.reason)
   else:
       print("Spoken output saved in " + outputFile)
    ```

    **C#**

    ```csharp
   // Synthesize spoken output
   SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(responseText);
   if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
   {
       Console.WriteLine(speak.Reason);
   }
   else
   {
       Console.WriteLine("Spoken output saved in " + outputFile);
   }
    ```

1. Salvare le modifiche (*CTRL+S*) e quindi nella riga di comando sotto l'editor di codice immettere il seguente comando per eseguire il programma:

   **Python**

    ```
   python speaking-clock.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. Esaminare l'output dell'applicazione, che dovrebbe indicare che l'output parlato è stato salvato in un file.
1. Se si dispone di un lettore multimediale in grado di riprodurre file audio .wav, nella barra degli strumenti del riquadro Cloud Shell usare il pulsante **Carica/Scarica file** per scaricare il file audio dalla cartella dell'app e quindi riprodurlo:

    **Python**

    /home/*user*`/mslearn-ai-language/Labfiles/07b-speech/Python/speaking-clock/output.wav`

    **C#**

    /home/*utente*`/mslearn-ai-language/Labfiles/07b-speech/C-Sharp/speaking-clock/output.wav`

    Il suono del file dovrebbe risultare simile al seguente:

    <video controls src="./ai-foundry/media/Output.mp4" title="Sono le 2:15" width="150"></video>

## Usare Speech Synthesis Markup Language

Speech Synthesis Markup Language (SSML) consente di personalizzare la modalità di sintesi vocale usando un formato basato su XML.

1. Nella funzione **TellTime** sostituire tutto il codice corrente sotto il commento **Synthesize spoken output** con il codice seguente (lasciare il codice sotto il commento **Print the response**):

    **Python**

    ```python
   # Synthesize spoken output
   responseSsml = " \
       <speak version='1.0' xmlns='http://www.w3.org/2001/10/synthesis' xml:lang='en-US'> \
           <voice name='en-GB-LibbyNeural'> \
               {} \
               <break strength='weak'/> \
               Time to end this lab! \
           </voice> \
       </speak>".format(response_text)
   speak = speech_synthesizer.speak_ssml_async(responseSsml).get()
   if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
       print(speak.reason)
   else:
       print("Spoken output saved in " + outputFile)
    ```

   **C#**

    ```csharp
   // Synthesize spoken output
   string responseSsml = $@"
       <speak version='1.0' xmlns='http://www.w3.org/2001/10/synthesis' xml:lang='en-US'>
           <voice name='en-GB-LibbyNeural'>
               {responseText}
               <break strength='weak'/>
               Time to end this lab!
           </voice>
       </speak>";
   SpeechSynthesisResult speak = await speechSynthesizer.SpeakSsmlAsync(responseSsml);
   if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
   {
       Console.WriteLine(speak.Reason);
   }
   else
   {
        Console.WriteLine("Spoken output saved in " + outputFile);
   }
    ```

1. Salvare le modifiche e tornare al terminale integrato per la cartella **speaking-clock**, quindi immettere il comando seguente per eseguire il programma:

    **Python**

    ```
   python speaking-clock.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. Esaminare l'output dell'applicazione, che dovrebbe indicare che l'output parlato è stato salvato in un file.
1. Anche in questo caso, se si dispone di un lettore multimediale in grado di riprodurre file audio .wav, nella barra degli strumenti del riquadro Cloud Shell usare il pulsante **Carica/Scarica file** per scaricare il file audio dalla cartella dell'app e quindi riprodurlo:

    **Python**

    /home/*user*`/mslearn-ai-language/Labfiles/07b-speech/Python/speaking-clock/output.wav`

    **C#**

    /home/*utente*`/mslearn-ai-language/Labfiles/07b-speech/C-Sharp/speaking-clock/output.wav`

    Il suono del file dovrebbe risultare simile al seguente:
    
    <video controls src="./ai-foundry/media/Output2.mp4" title="Sono le 5:30. Ora di finire il lab." width="150"></video>

## Eseguire la pulizia

Al termine dell'esplorazione di Voce di Azure AI, è necessario eliminare le risorse create in questo esercizio per evitare di incorrere in costi di Azure non necessari.

1. Tornare alla scheda del browser che contiene il portale di Azure (o riaprire il [portale di Azure](https://portal.azure.com) su `https://portal.azure.com` in una nuova scheda del browser) e visualizzare il contenuto del gruppo di risorse in cui sono state distribuite le risorse usate in questo esercizio.
1. Sulla barra degli strumenti selezionare **Elimina gruppo di risorse**.
1. Immettere il nome del gruppo di risorse e confermarne l'eliminazione.

## E se si dispone di microfono e altoparlante?

Come input vocale per questo esercizio si sono utilizzati file audio per input e output vocale. Vediamo come modificare il codice per usare l'hardware audio.

### Uso del riconoscimento vocale con un microfono

Se si dispone di un microfono, è possibile usare il codice seguente per acquisire l'input vocale per il riconoscimento vocale:

**Python**

```python
# Configure speech recognition
audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
print('Speak now...')

# Process speech input
speech = speech_recognizer.recognize_once_async().get()
if speech.reason == speech_sdk.ResultReason.RecognizedSpeech:
    command = speech.text
    print(command)
else:
    print(speech.reason)
    if speech.reason == speech_sdk.ResultReason.Canceled:
        cancellation = speech.cancellation_details
        print(cancellation.reason)
        print(cancellation.error_details)

```

**C#**

```csharp
// Configure speech recognition
using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
Console.WriteLine("Speak now...");

SpeechRecognitionResult speech = await speechRecognizer.RecognizeOnceAsync();
if (speech.Reason == ResultReason.RecognizedSpeech)
{
    command = speech.Text;
    Console.WriteLine(command);
}
else
{
    Console.WriteLine(speech.Reason);
    if (speech.Reason == ResultReason.Canceled)
    {
        var cancellation = CancellationDetails.FromResult(speech);
        Console.WriteLine(cancellation.Reason);
        Console.WriteLine(cancellation.ErrorDetails);
    }
}
```

> **Nota**: il microfono predefinito del sistema è l'input audio predefinito, quindi è possibile anche omettere completamente AudioConfig!

### Uso della sintesi vocale con un altoparlante

Se si dispone di un altoparlante, è possibile usare il codice seguente per sintetizzare il parlato.

**Python**

```python
response_text = 'The time is {}:{:02d}'.format(now.hour,now.minute)

# Configure speech synthesis
speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
audio_config = speech_sdk.audio.AudioOutputConfig(use_default_speaker=True)
speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config)

# Synthesize spoken output
speak = speech_synthesizer.speak_text_async(response_text).get()
if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
    print(speak.reason)
```

**C#**

```csharp
var now = DateTime.Now;
string responseText = "The time is " + now.Hour.ToString() + ":" + now.Minute.ToString("D2");

// Configure speech synthesis
speechConfig.SpeechSynthesisVoiceName = "en-GB-RyanNeural";
using var audioConfig = AudioConfig.FromDefaultSpeakerOutput();
using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig, audioConfig);

// Synthesize spoken output
SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(responseText);
if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
{
    Console.WriteLine(speak.Reason);
}
```

> **Nota**: l'altoparlante predefinito del sistema è l'output audio predefinito, quindi è possibile anche omettere completamente AudioConfig!

## Ulteriori informazioni

Per altre informazioni sull'uso delle API di **conversione della voce in testo scritto** e **sintesi vocale**, vedere la [documentazione sulla conversione della voce in testo scritto](https://learn.microsoft.com/azure/ai-services/speech-service/index-speech-to-text) e la [documentazione sulla sintesi vocale](https://learn.microsoft.com/azure/ai-services/speech-service/index-text-to-speech).
