---
lab:
  title: Riconoscimento e sintesi vocale
  module: Module 4 - Create speech-enabled apps with Azure AI services
---

# Riconoscimento e sintesi vocale

**Voce di Azure AI** è un servizio che fornisce funzionalità correlate al riconoscimento vocale, tra cui:

- Un'API di *conversione della voce in testo scritto* che consente di implementare il riconoscimento vocale (convertendo parole vocali udibili in testo).
- Un'API *sintesi vocale* che consente di implementare la sintesi vocale (convertendo il testo in voce udibile).

In questo esercizio si useranno entrambe queste API per implementare un'applicazione di orologio vocale.

> **NOTA**: per questo esercizio è necessario usare un computer con altoparlanti/cuffie. Per un'esperienza ottimale è necessario anche un microfono. Alcuni ambienti virtuali ospitati potrebbero essere in grado di acquisire audio dal microfono locale, ma se questo approccio non funziona o non è disponibile alcun microfono, è possibile usare un file audio fornito per l'input vocale. Seguire con attenzione le istruzioni, poiché sarà necessario scegliere opzioni diverse in base alla scelta di usare un microfono oppure il file audio.

## Effettuare il provisioning di una risorsa *Voce di Azure AI*

Se non è già disponibile nella sottoscrizione, sarà necessario effettuare il provisioning di una risorsa di **Voce di Azure AI**.

1. Aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.
1. Nel campo di ricerca nella parte superiore, cercare **Servizi di Azure AI** e premere **INVIO**, quindi selezionare **Crea** in **Servizio Voce** nei risultati.
1. Creare una risorsa con le impostazioni seguenti:
    - **Sottoscrizione**: *la sottoscrizione di Azure usata*
    - **Gruppo di risorse**: *Scegliere o creare un gruppo di risorse*
    - **Area**: *scegliere una qualsiasi area disponibile*
    - **Nome**: *immettere un nome univoco*
    - **Piano tariffario**: Selezionare **F0** (*gratuito*) o **S** (*standard*) se F non è disponibile.
    - **Informativa sull'Intelligenza artificiale responsabile**: D'accordo.
1. Selezionare **Rivedi e crea**, quindi **Crea** per effettuare il provisioning della risorsa.
1. Attendere il completamento della distribuzione e quindi passare alla risorsa distribuita.
1. Visualizzare la pagina **Chiavi ed endpoint**. Le informazioni contenute in questa pagina saranno necessarie più avanti nell'esercizio.

## Preparare lo sviluppo di un'app in Visual Studio Code

Si svilupperà l'app voce usando Visual Studio Code. I file di codice per l'app sono stati forniti in un repository GitHub.

> **Suggerimento**: Se il repository **mslearn-ai-language** è già stato clonato, aprirlo in Visual Studio Code. In caso contrario, seguire questa procedura per clonarla nell'ambiente di sviluppo.

1. Avviare Visual Studio Code.
1. Aprire il riquadro comandi (MAIUSC+CTRL+P) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/mslearn-ai-language` in una cartella locale. Non è importante usare una cartella specifica.
1. Dopo la clonazione del repository, aprire la cartella in Visual Studio Code.

    > **Nota**: Se Visual Studio Code visualizza un messaggio popup per chiedere se si considera attendibile il codice che si apre, fare clic sull'opzione **Sì, considero attendibili gli autori** nel popup.

1. Attendere il completamento dell'installazione di file aggiuntivi per supportare i progetti in codice C# nel repository.

    > **Nota**: se viene richiesto di aggiungere gli asset necessari per la compilazione e il debug, selezionare **Non adesso**.

## Configurare l'applicazione

Sono state fornite applicazioni sia per C# che per Python. Entrambe le app presentano la stessa funzionalità. Prima di tutto, verranno completate alcune parti chiave dell'applicazione per abilitarla all'uso della risorsa Voce di Azure AI.

1. Nel riquadro **Esplora risorse** di Visual Studio Code passare alla cartella **Labfiles/07-speech** ed espandere la cartella **CSharp** o **Python**, in base alle preferenze della lingua e alla cartella **speaking-clock** che contiene. Ogni cartella contiene i file di codice specifici del linguaggio per un'app in cui si intende integrare la funzionalità Voce di Azure AI.
1. Fare clic con il pulsante destro del mouse sulla cartella **speaking-clock** contenente i file di codice e aprire un terminale integrato. Installare quindi il pacchetto Speech SDK di Voce di Azure AI eseguendo il comando appropriato per la lingua scelta:

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.30.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-speech==1.30.0
    ```

1. Nel riquadro **Esplora risorse**, aprire il file di configurazione per la lingua preferita nella cartella **speaking-clock**

    - **C#**: appsettings.json
    - **Python**: .env

1. Aggiornare i valori di configurazione per includere l'**area** e una **chiave** dalla risorsa Voce di Azure AI creata (disponibile nella pagina **Chiavi ed endpoint** per la risorsa Voce di Azure AI nel portale di Azure).

    > **NOTA**: Assicurarsi di aggiungere l'*area* per la risorsa, <u>non</u> l'endpoint.

1. Salvare il file di configurazione.

## Aggiungere codice per usare l'SDK di Voce di Azure AI

1. Si noti che la cartella **speaking-clock** contiene un file di codice per l'applicazione client:

    - **C#**: Program.cs
    - **Python**: speaking-clock.py

    Aprire il file di codice e nella parte superiore, sotto i riferimenti agli spazi dei nomi esistenti, individuare il commento **Import namespaces**. In questo commento aggiungere quindi il codice seguente specifico del linguaggio per importare gli spazi dei nomi necessari per usare l'SDK di Voce di Azure AI:

    **C#**: Program.cs

    ```csharp
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    ```

    **Python**: speaking-clock.py

    ```python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

1. Nella funzione **Principale** si noti che il codice per caricare la chiave e l'area di del servizio dal file di configurazione è già stato fornito. È necessario usare queste variabili per creare un **SpeechConfig** per la risorsa Voce di Azure AI. Aggiungere il codice seguente sotto il commento **Configure speech service**:

    **C#**: Program.cs

    ```csharp
    // Configure speech service
    speechConfig = SpeechConfig.FromSubscription(aiSvcKey, aiSvcRegion);
    Console.WriteLine("Ready to use speech service in " + speechConfig.Region);
    
    // Configure voice
    speechConfig.SpeechSynthesisVoiceName = "en-US-AriaNeural";
    ```

    **Python**: speaking-clock.py

    ```python
    # Configure speech service
    speech_config = speech_sdk.SpeechConfig(ai_key, ai_region)
    print('Ready to use speech service in:', speech_config.region)
    ```

1. Salvare le modifiche e tornare al terminale integrato per la cartella **speaking-clock**, quindi immettere il comando seguente per eseguire il programma:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. Se si usa C#, è possibile ignorare eventuali avvisi relativi all'uso dell'operatore **await** nei metodi asincroni. Questo problema verrà risolto in un secondo momento. Il codice dovrebbe visualizzare l'area della risorsa del servizio Voce che verrà utilizzata dall'applicazione.

## Aggiungere codice per il riconoscimento vocale

È ora disponibile un elemento **SpeechConfig** per il servizio Voce nella risorsa Voce di Azure AI ed è quindi possibile usare l'API di **conversione della voce in testo scritto** per riconoscere il parlato e trascriverlo in testo.

> **IMPORTANTE**: Questa sezione include istruzioni per due procedure alternative. Seguire la prima procedura se si dispone di un microfono funzionante. Seguire la seconda procedura se si vuole simulare l'input parlato usando un file audio.

### Se è disponibile un microfono funzionante

1. Nella funzione **Main** per il programma si noti che il codice usa la funzione **TranscribeCommand** per accettare l'input vocale.
1. Nella funzione **TranscribeCommand**, sotto il commento **Configure speech recognition**, aggiungere il codice appropriato seguente per creare un client **SpeechRecognizer** che può essere usato per riconoscere e trascrivere il parlato usando il microfono di sistema predefinito:

    **C#**

    ```csharp
    // Configure speech recognition
    using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    Console.WriteLine("Speak now...");
    ```

    **Python**

    ```python
    # Configure speech recognition
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    print('Speak now...')
    ```

1. Passare ora alla sezione **Aggiungere il codice per elaborare il comando trascritto** di seguito.

---

### In alternativa, usare l'input audio da un file.

1. Nella finestra del terminale immettere il comando seguente per installare una libreria che può essere usata per riprodurre il file audio:

    **C#**

    ```
    dotnet add package System.Windows.Extensions --version 4.6.0 
    ```

    **Python**

    ```
    pip install playsound==1.2.2
    ```

1. Nel file di codice per il programma aggiungere il codice seguente sotto le importazioni dello spazio dei nomi esistenti per importare la libreria appena installata:

    **C#**: Program.cs

    ```csharp
    using System.Media;
    ```

    **Python**: speaking-clock.py

    ```python
    from playsound import playsound
    ```

1. Nella funzione **Main** si noti che il codice usa la funzione **TranscribeCommand** per accettare l'input vocale. Nella funzione **TranscribeCommand**, sotto il commento **Configure speech recognition**, aggiungere il codice appropriato seguente per creare un client **SpeechRecognizer** che può essere usato per riconoscere e trascrivere il parlato da un file audio:

    **C#**: Program.cs

    ```csharp
    // Configure speech recognition
    string audioFile = "time.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    ```

    **Python**: speaking-clock.py

    ```python
    # Configure speech recognition
    current_dir = os.getcwd()
    audioFile = current_dir + '\\time.wav'
    playsound(audioFile)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    ```

---

### Aggiungere il codice per elaborare il comando trascritto

1. Nella funzione **TranscribeCommand**, sotto il commento **Process speech input**, aggiungere il codice seguente per l'ascolto dell'input vocale, prestando attenzione a non sostituire il codice alla fine della funzione che restituisce il comando:

    **C#**: Program.cs

    ```csharp
    // Process speech input
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

    **Python**: speaking-clock.py

    ```python
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

1. Salvare le modifiche e tornare al terminale integrato per la cartella **speaking-clock**, quindi immettere il comando seguente per eseguire il programma:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. Se si usa un microfono, parlare chiaramente e pronunciare "che ora è?". Il programma trascriverà l'input vocale e visualizzerà l'ora in base all'ora locale del computer in cui è in esecuzione il codice, che potrebbe non essere l'ora corretta in cui ci si trova.

    SpeechRecognizer offre circa 5 secondi per parlare. Se non rileva alcun input vocale, produce un risultato di tipo "Nessuna corrispondenza".

    Se SpeechRecognizer rileva un errore, viene generato il risultato "Cancelled". Il codice nell'applicazione visualizza quindi il messaggio di errore. La causa più probabile è una chiave o un'area non corretta nel file di configurazione.

## Sintesi vocale

L'applicazione dell'orologio vocale accetta l'input vocale, ma in realtà non parla. Per risolvere questo problema, aggiungere codice per sintetizzare il parlato.

1. Nella funzione **Main** per il programma si noti che il codice usa la funzione **TellTime** per indicare all'utente l'ora corrente.
1. Nella funzione **TellTime**, sotto il commento **Configure speech synthesis**, aggiungere il codice seguente per creare un client **SpeechSynthesizer** che può essere usato per generare l'output vocale:

    **C#**: Program.cs

    ```csharp
    // Configure speech synthesis
    speechConfig.SpeechSynthesisVoiceName = "en-GB-RyanNeural";
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    ```

    **Python**: speaking-clock.py

    ```python
    # Configure speech synthesis
    speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    ```

    > **NOTA** La configurazione audio predefinita usa il dispositivo audio di sistema predefinito per l'output, quindi non è necessario fornire in modo esplicito un **AudioConfig**. Se devi reindirizzare l'output audio a un file, puoi usare **audioConfig** con un percorso file per farlo.

1. Nella funzione **TellTime**, sotto il commento **Synthesize spoken output**, aggiungere il codice seguente per generare l'output vocale, prestando attenzione a non sostituire il codice alla fine della funzione che stampa la risposta:

    **C#**: Program.cs

    ```csharp
    // Synthesize spoken output
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(responseText);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```

    **Python**: speaking-clock.py

    ```python
    # Synthesize spoken output
    speak = speech_synthesizer.speak_text_async(response_text).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

1. Salvare le modifiche e tornare al terminale integrato per la cartella **speaking-clock**, quindi immettere il comando seguente per eseguire il programma:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. Quando richiesto, parlare chiaramente nel microfono e dire "che ora è?". Il programma parlerà, indicando l'ora.

## Usare una voce diversa

L'applicazione dell'orologio vocale usa una voce predefinita, che è possibile modificare. Il servizio Voce supporta una gamma di voci *standard* e voci *neurali* più simili a quelle umane. È anche possibile creare voci *personalizzate*.

> **Nota**: Per un elenco delle voci neurali e standard, vedere [Raccolta voci](https://speech.microsoft.com/portal/voicegallery) in Speech Studio.

1. Nella funzione **TellTime**, sotto il commento **Configure speech synthesis**, modificare il codice come segue per specificare una voce alternativa prima di creare il client **SpeechSynthesizer**:

   **C#**: Program.cs

    ```csharp
    // Configure speech synthesis
    speechConfig.SpeechSynthesisVoiceName = "en-GB-LibbyNeural"; // change this
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    ```

    **Python**: speaking-clock.py

    ```python
    # Configure speech synthesis
    speech_config.speech_synthesis_voice_name = 'en-GB-LibbyNeural' # change this
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    ```

1. Salvare le modifiche e tornare al terminale integrato per la cartella **speaking-clock**, quindi immettere il comando seguente per eseguire il programma:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. Quando richiesto, parlare chiaramente nel microfono e dire "che ora è?". Il programma parlerà con la voce specificata, indicando l'ora.

## Usare Speech Synthesis Markup Language

Speech Synthesis Markup Language (SSML) consente di personalizzare la modalità di sintesi vocale usando un formato basato su XML.

1. Nella funzione **TellTime** sostituire tutto il codice corrente sotto il commento **Synthesize spoken output** con il codice seguente (lasciare il codice sotto il commento **Print the response**):

   **C#**: Program.cs

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
    ```

    **Python**: speaking-clock.py

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
    ```

1. Salvare le modifiche e tornare al terminale integrato per la cartella **speaking-clock**, quindi immettere il comando seguente per eseguire il programma:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. Quando richiesto, parlare chiaramente nel microfono e dire "che ora è?". Il programma parlerà nella voce specificata in SSML (eseguendo l'override della voce specificata in SpeechConfig), indicando l'ora e quindi, dopo una pausa, indicando che è il momento di terminare il lab, effettivamente.

## Ulteriori informazioni

Per altre informazioni sull'uso delle API di **conversione della voce in testo scritto** e **sintesi vocale**, vedere la [documentazione sulla conversione della voce in testo scritto](https://learn.microsoft.com/azure/ai-services/speech-service/index-speech-to-text) e la [documentazione sulla sintesi vocale](https://learn.microsoft.com/azure/ai-services/speech-service/index-text-to-speech).
