---
lab:
  title: Tradurre il parlato
  module: Module 8 - Translate speech with Azure AI Speech
---

# Tradurre il parlato

Voce di Azure AI include un'API di traduzione vocale che consente di tradurre la lingua parlato. Si supponga ad esempio di voler sviluppare un'applicazione di traduzione che possa essere usata dagli utenti quando viaggiano in luoghi di cui non conoscono la lingua locale. Gli utenti potrebbero pronunciare frasi quali "Dove si trova la stazione?" o "Ho bisogno di trovare una farmacia" nella propria lingua e ottenere la traduzione nella lingua locale.

> **NOTA**: in questo esercizio è necessario usare un computer con altoparlanti/cuffie. Per un'esperienza ottimale è necessario anche un microfono. Alcuni ambienti virtuali ospitati potrebbero essere in grado di acquisire audio dal microfono locale, ma se questo approccio non funziona o non è disponibile alcun microfono, è possibile usare un file audio fornito per l'input vocale. Seguire con attenzione le istruzioni, poiché sarà necessario scegliere opzioni diverse in base alla scelta di usare un microfono oppure il file audio.

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

> **Suggerimento**: se il repository **mslearn-ai-language** è già stato clonato, aprirlo in Visual Studio Code. In caso contrario, eseguire questi passaggi per clonarlo nell'ambiente di sviluppo.

1. Avviare Visual Studio Code.
1. Aprire il riquadro comandi (MAIUSC+CTRL+P) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/mslearn-ai-language` in una cartella locale. Non è importante usare una cartella specifica.
1. Dopo la clonazione del repository, aprire la cartella in Visual Studio Code.

    > **Nota**: se in Visual Studio Code viene visualizzato un messaggio popup che chiede se si considera attendibile il codice da aprire, fare clic sull'opzione **Sì, mi fido degli autori** nel popup.

1. Attendere il completamento dell'installazione di file aggiuntivi per supportare i progetti in codice C# nel repository.

    > **Nota**: se viene richiesto di aggiungere gli asset necessari per la compilazione e il debug, selezionare **Non adesso**.

## Configurare l'applicazione

Sono state fornite applicazioni per C# e per Python. Entrambe le app presentano la stessa funzionalità. Prima di tutto, verranno completate alcune parti chiave dell'applicazione per abilitarla all'uso della risorsa di Voce di Azure AI.

1. Nel riquadro **Esplora risorse** in Visual Studio Code, esplorare la cartella **Labfiles/08-speech-translation** ed espandere la cartella **CSharp** o **Python** in base alla scelta della lingua e alla cartella **translator** in essa contenuta. Ogni cartella contiene i file di codice specifici della lingua per l'app in cui si vuole integrare la funzionalità Voce di Azure AI.
1. Fare clic con il pulsante destro del mouse sulla cartella **translator** contenente i file di codice e aprire un terminale integrato. Installare quindi il pacchetto SDK di Voce di Azure AI eseguendo il comando appropriato per la lingua scelta:

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.30.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-speech==1.30.0
    ```

1. Nel riquadro **Esplora risorse** nella cartella **translate-text**, aprire il file di configurazione per la lingua preferita

    - **C#**: appsettings.json
    - **Python**: .env

1. Aggiornare i valori di configurazione per includere l'**area** e una **chiave** dalla risorsa di Voce di Azure AI creata (disponibile nella pagina **Chiavi ed endpoint** per la risorsa di Voce di Azure AI nel portale di Azure).

    > **NOTA**: Assicurarsi di aggiungere l'*area* per la risorsa, <u>non</u> l'endpoint.

1. Salvare il file di configurazione.

## Aggiungere codice per usare l'SDK di Voce

1. Si noti che la cartella **translator** contiene un file di codice per l'applicazione client:

    - **C#**: Program.cs
    - **Python**: translator.py

    Aprire il file di codice e nella parte superiore, sotto i riferimenti agli spazi dei nomi esistenti, individuare il commento **Import namespaces**. In questo commento aggiungere quindi il codice seguente specifico della lingua per importare gli spazi dei nomi necessari per usare l'SDK di Voce di Azure AI:

    **C#**: Program.cs

    ```csharp
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    using Microsoft.CognitiveServices.Speech.Translation;
    ```

    **Python**: translator.py

    ```python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

1. Nella funzione **Principale**, si noti che il codice per caricare la chiave e l'area di del servizio Voce di Azure AI dal file di configurazione è già stato fornito. È necessario usare queste variabili per creare un'istanza **SpeechTranslationConfig** per la risorsa di Voce di Azure AI, che verrà usato per tradurre l'input vocale. Aggiungere il codice seguente sotto il commento **Configure translation**:

    **C#**: Program.cs

    ```csharp
    // Configure translation
    translationConfig = SpeechTranslationConfig.FromSubscription(aiSvcKey, aiSvcRegion);
    translationConfig.SpeechRecognitionLanguage = "en-US";
    translationConfig.AddTargetLanguage("fr");
    translationConfig.AddTargetLanguage("es");
    translationConfig.AddTargetLanguage("hi");
    Console.WriteLine("Ready to translate from " + translationConfig.SpeechRecognitionLanguage);
    ```

    **Python**: translator.py

    ```python
    # Configure translation
    translation_config = speech_sdk.translation.SpeechTranslationConfig(ai_key, ai_region)
    translation_config.speech_recognition_language = 'en-US'
    translation_config.add_target_language('fr')
    translation_config.add_target_language('es')
    translation_config.add_target_language('hi')
    print('Ready to translate from',translation_config.speech_recognition_language)
    ```

1. **SpeechTranslationConfig** verrà usato per tradurre il parlato in testo, ma verrà usato anche **SpeechConfig** per sintetizzare le traduzioni in parlato. Aggiungere il codice seguente sotto il commento **Configure translation**:

    **C#**: Program.cs

    ```csharp
    // Configure speech
    speechConfig = SpeechConfig.FromSubscription(aiSvcKey, aiSvcRegion);
    ```

    **Python**: translator.py

    ```python
    # Configure speech
    speech_config = speech_sdk.SpeechConfig(ai_key, ai_region)
    ```

1. Salvare le modifiche e tornare al terminale integrato per la cartella **translator**, quindi immettere il comando seguente per eseguire il programma:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python translator.py
    ```

1. Se si usa C#, è possibile ignorare eventuali avvisi relativi all'uso dell'operatore **await** nei metodi asincroni. Questo problema verrà risolto in un secondo momento. Il codice dovrebbe mostrare un messaggio che indica che è tutto pronto per tradurre da en-US e richiedere una lingua di destinazione. Premere INVIO per terminare il programma.

## Implementare la traduzione vocale

Ora che è disponibile un'istanza **SpeechTranslationConfig** per il servizio Voce di Azure AI, è quindi possibile usare l'API di traduzione di Voce di Azure AI per riconoscere e tradurre il parlato.

> **IMPORTANTE**: Questa sezione include istruzioni per due procedure alternative. Seguire la prima procedura se si dispone di un microfono funzionante. Seguire la seconda procedura se si vuole simulare l'input parlato usando un file audio.

### Se è disponibile un microfono funzionante

1. Nella funzione **Main** per il programma si noti che il codice usa la funzione **Translate** per tradurre l'input vocale.
1. Nella funzione **Translate** sotto il commento **Translate speech** aggiungere il codice seguente per creare un client **TranslationRecognizer** che può essere usato per riconoscere e tradurre il parlato usando il microfono di sistema predefinito per l'input.

    **C#**: Program.cs

    ```csharp
    // Translate speech
    using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    using TranslationRecognizer translator = new TranslationRecognizer(translationConfig, audioConfig);
    Console.WriteLine("Speak now...");
    TranslationRecognitionResult result = await translator.RecognizeOnceAsync();
    Console.WriteLine($"Translating '{result.Text}'");
    translation = result.Translations[targetLanguage];
    Console.OutputEncoding = Encoding.UTF8;
    Console.WriteLine(translation);
    ```

    **Python**: translator.py

    ```python
    # Translate speech
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config = audio_config)
    print("Speak now...")
    result = translator.recognize_once_async().get()
    print('Translating "{}"'.format(result.text))
    translation = result.translations[targetLanguage]
    print(translation)
    ```

    > **NOTA**: il codice nell'applicazione converte l'input in tutte e tre le lingue in un'unica chiamata. Viene visualizzata solo la traduzione per la lingua specifica, ma è possibile recuperare qualsiasi traduzione specificando il codice della lingua di destinazione nella raccolta **translations** del risultato.

1. Passare ora alla sezione **Eseguire il programma** disponibile più avanti.

---

### In alternativa, usare l'input audio da un file.

1. Nella finestra del terminale immettere il comando seguente per installare una libreria che può essere usata per riprodurre il file audio:

    **C#**: Program.cs

    ```csharp
    dotnet add package System.Windows.Extensions --version 4.6.0 
    ```

    **Python**: translator.py

    ```python
    pip install playsound==1.3.0
    ```

1. Nel file di codice per il programma aggiungere il codice seguente sotto le importazioni dello spazio dei nomi esistenti per importare la libreria appena installata:

    **C#**: Program.cs

    ```csharp
    using System.Media;
    ```

    **Python**: translator.py

    ```python
    from playsound import playsound
    ```

1. Nella funzione **Main** per il programma si noti che il codice usa la funzione **Translate** per tradurre l'input vocale. Nella funzione **Translate** sotto il commento **Translate speech** aggiungere quindi il codice seguente per creare un client **TranslationRecognizer** che può essere usato per riconoscere e tradurre il parlato da un file.

    **C#**: Program.cs

    ```csharp
    // Translate speech
    string audioFile = "station.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    using TranslationRecognizer translator = new TranslationRecognizer(translationConfig, audioConfig);
    Console.WriteLine("Getting speech from file...");
    TranslationRecognitionResult result = await translator.RecognizeOnceAsync();
    Console.WriteLine($"Translating '{result.Text}'");
    translation = result.Translations[targetLanguage];
    Console.OutputEncoding = Encoding.UTF8;
    Console.WriteLine(translation);
    ```

    **Python**: translator.py

    ```python
    # Translate speech
    audioFile = 'station.wav'
    playsound(audioFile)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config = audio_config)
    print("Getting speech from file...")
    result = translator.recognize_once_async().get()
    print('Translating "{}"'.format(result.text))
    translation = result.translations[targetLanguage]
    print(translation)
    ```

---

### Eseguire il programma

1. Salvare le modifiche e tornare al terminale integrato per la cartella **translator**, quindi immettere il comando seguente per eseguire il programma:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python translator.py
    ```

1. Quando richiesto, immettere un codice della lingua valido (*fr*, *es* o *hi*) e quindi, se si usa un microfono, parlare in modo chiaro e pronunciare la frase "Dove si trova la stazione?" o un'altra frase che potrebbe risultare utile quando si viaggia all'estero. Il programma dovrebbe trascrivere l'input vocale e tradurlo nella lingua specificata (francese, spagnolo o hindi). Ripetere questo processo, provando ogni lingua supportata dall'applicazione. Al termine, premere INVIO per terminare il programma.

    TranslationRecognizer offre circa 5 secondi per parlare. Se non rileva alcun input vocale, produce un risultato di tipo "Nessuna corrispondenza". È possibile che la traduzione in hindi non venga sempre visualizzata correttamente nella finestra della console a causa di problemi di codifica dei caratteri.

> **NOTA**: Il codice nell'applicazione traduce l'input in tutte e tre le lingue in un'unica chiamata. Viene visualizzata solo la traduzione per la lingua specifica, ma è possibile recuperare qualsiasi traduzione specificando il codice della lingua di destinazione nella raccolta **translations** del risultato.

## Sintetizzare la traduzione in parlato

Fino a questo punto, l'applicazione traduce l'input vocale in testo, che potrebbe risultare sufficiente se è necessario richiedere assistenza a qualcuno durante un viaggio. Sarebbe tuttavia meglio se la traduzione fosse pronunciata a voce alta con una voce appropriata.

1. Nella funzione **Translate** sotto il commento **Synthesize translation** aggiungere il codice seguente per usare un client **SpeechSynthesizer** per sintetizzare la traduzione come parlato tramite l'altoparlante predefinito.

    **C#**: Program.cs

    ```csharp
    // Synthesize translation
    var voices = new Dictionary<string, string>
                    {
                        ["fr"] = "fr-FR-HenriNeural",
                        ["es"] = "es-ES-ElviraNeural",
                        ["hi"] = "hi-IN-MadhurNeural"
                    };
    speechConfig.SpeechSynthesisVoiceName = voices[targetLanguage];
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(translation);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```

    **Python**: translator.py

    ```python
    # Synthesize translation
    voices = {
            "fr": "fr-FR-HenriNeural",
            "es": "es-ES-ElviraNeural",
            "hi": "hi-IN-MadhurNeural"
    }
    speech_config.speech_synthesis_voice_name = voices.get(targetLanguage)
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    speak = speech_synthesizer.speak_text_async(translation).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

1. Salvare le modifiche e tornare al terminale integrato per la cartella **translator**, quindi immettere il comando seguente per eseguire il programma:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python translator.py
    ```

1. Quanto richiesto, immettere un codice della lingua valido (*fr*, *es* o *hi*) e quindi parlare in modo chiaro nel microfono e pronunciare una frase che potrebbe risultare utile quando si viaggia all'estero. Il programma dovrebbe trascrivere l'input vocale e rispondere con una traduzione vocale. Ripetere questo processo, provando ogni lingua supportata dall'applicazione. Al termine, premere **INVIO** per terminare il programma.

> **NOTA**
> *In questo esempio è stata usata un'istanza **SpeechTranslationConfig** per tradurre il parlato in testo e quindi è stato usato un **SpeechConfig** per sintetizzare la traduzione come voce. È infatti possibile usare **SpeechTranslationConfig** per sintetizzare direttamente la traduzione, ma questo approccio funziona solo quando si traduce in una singola lingua e restituisce un flusso audio che viene in genere salvato come file invece che inviato direttamente a un parlante.*

## Ulteriori informazioni

Per altre informazioni sull'uso dell'API di Voce di Azure AI, vedere la [documentazione di Traduzione vocale](https://learn.microsoft.com/azure/ai-services/speech-service/speech-translation).
