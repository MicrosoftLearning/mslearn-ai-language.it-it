---
lab:
  title: Tradurre il parlato
  description: Tradurre la voce in lingua e implementare nella propria app.
---

# Tradurre il parlato

Voce di Azure AI include un'API di traduzione vocale che consente di tradurre la lingua parlato. Si supponga ad esempio di voler sviluppare un'applicazione di traduzione che possa essere usata dagli utenti quando viaggiano in luoghi di cui non conoscono la lingua locale. Gli utenti potrebbero pronunciare frasi quali "Dove si trova la stazione?" o "Ho bisogno di trovare una farmacia" nella propria lingua e ottenere la traduzione nella lingua locale. In questo esercizio si userà l'SDK Voce di Azure AI per Python per creare una semplice applicazione basata su questo esempio.

Anche se questo esercizio è basato su Python, è possibile sviluppare applicazioni di traduzione vocale usando più SDK specifici del linguaggio, tra cui:

- [SDK di Voce di Azure AI per Python](https://pypi.org/project/azure-cognitiveservices-speech/)
- [SDK di Voce di Azure AI per .NET](https://www.nuget.org/packages/Microsoft.CognitiveServices.Speech)
- [SDK di Voce di Azure AI per JavaScript](https://www.npmjs.com/package/microsoft-cognitiveservices-speech-sdk)

Questo esercizio richiede circa **30** minuti.

> **NOTA** Questo esercizio è progettato per essere completato in Azure Cloud Shell, in cui l'accesso diretto all'hardware audio del computer non è supportato. Il lab userà quindi i file audio per i flussi di input e output vocale. Il codice per ottenere gli stessi risultati usando un microfono e un altoparlante viene fornito come riferimento.

## Creare una risorsa per Voce di Azure AI

Per iniziare, creare una risorsa Voce di Azure AI.

1. Aprire il [portale di Azure](https://portal.azure.com) all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.
1. Nel campo di ricerca, cercare **Servizio Voce**. Selezionarlo dall'elenco, quindi selezionare **Crea**.
1. Effettuare il provisioning della risorsa usando le impostazioni seguenti:
    - **Sottoscrizione**: *la sottoscrizione di Azure usata*.
    - **Gruppo di risorse**: *Selezionare o creare un gruppo di risorse*.
    - **Area**: *scegliere una qualsiasi area disponibile*
    - **Nome**: *immettere un nome univoco*.
    - **Piano tariffario**: Selezionare **F0** (*gratuito*) o **S** (*standard*) se F non è disponibile.
1. Selezionare **Rivedi e crea**, quindi **Crea** per effettuare il provisioning della risorsa.
1. Attendere il completamento della distribuzione e quindi passare alla risorsa distribuita.
1. Individuare **Endpoint e chiavi** nella sezione **Gestione risorse**. Le informazioni contenute in questa pagina saranno necessarie più avanti nell'esercizio.

## Preparare lo sviluppo di un'app in Cloud Shell

1. Lasciando aperta la pagina **Chiavi ed endpoint**, usare il pulsante **[\>_]** a destra della barra di ricerca nella parte superiore della pagina per creare una nuova Cloud Shell nel portale di Azure, selezionando un ambiente ***PowerShell***. Cloud Shell fornisce un'interfaccia della riga di comando in un riquadro nella parte inferiore del portale di Azure.

    > **Nota**: se in precedenza è stata creata una sessione Cloud Shell che usa un ambiente *Bash*, passare a ***PowerShell***.

1. Nella barra degli strumenti di Cloud Shell scegliere **Vai alla versione classica** dal menu **Impostazioni**. Questa operazione è necessaria per usare l'editor di codice.

    **<font color="red">Verificare di passare alla versione classica di Cloud Shell prima di continuare.</font>**

1. Nel riquadro PowerShell immettere i comandi seguenti per clonare il repository GitHub per questo esercizio:

    ```
   rm -r mslearn-ai-language -f
   git clone https://github.com/microsoftlearning/mslearn-ai-language
    ```

    > **Suggerimento**: Quando si immettono i comandi in Cloud Shell, l'ouput può richiedere una grande quantità di buffer dello schermo. È possibile cancellare la schermata immettendo il `cls` comando per rendere più semplice concentrarsi su ogni attività.

1. Dopo aver clonato il repository, passare alla cartella contenente i file di codice:

    ```
   cd mslearn-ai-language/Labfiles/08-speech-translation/Python/translator
    ```

1. Nel riquadro della riga di comando eseguire il comando seguente per visualizzare i file di codice nella cartella **translator**:

    ```
   ls -a -l
    ```

    I file includono un file di configurazione (con estensione **env**) e un file di codice (**translator.py**).

1. Creare un ambiente virtuale Python e installare il pacchetto SDK di Voce di Azure AI e altri pacchetti necessari eseguendo il comando seguente:

    ```
    python -m venv labenv
    ./labenv/bin/Activate.ps1
    pip install -r requirements.txt azure-cognitiveservices-speech==1.42.0
    ```

1. Immettere il comando seguente per modificare il file di configurazione fornito:

    ```
   code .env
    ```

    Il file viene aperto in un editor di codice.

1. Aggiornare i valori di configurazione per includere l’**area** e una **chiave** dalla risorsa Voce di Azure AI creata, disponibile nella pagina **Chiavi ed endpoint** per la risorsa Traduttore per Azure AI nel portale di Azure.
1. Dopo aver sostituito i segnaposto, usare il comando **CTRL+S** per salvare le modifiche e quindi usare il comando **CTRL+Q** per chiudere l'editor di codice mantenendo aperta la riga di comando di Cloud Shell.

## Aggiungere codice per usare l'SDK di Voce di Azure AI

> **Suggerimento**: quando si aggiunge codice, assicurarsi di mantenere il rientro corretto.

1. Immettere il comando seguente per modificare il file di codice fornito:

    ```
   code translator.py
    ```

1. Nella parte superiore del file di codice, sotto i riferimenti agli spazi dei nomi esistenti, individuare il commento **Import namespaces**. In questo commento aggiungere quindi il codice seguente specifico della lingua per importare gli spazi dei nomi necessari per usare l'SDK di Voce di Azure AI:

    ```python
   # Import namespaces
   from azure.core.credentials import AzureKeyCredential
   import azure.cognitiveservices.speech as speech_sdk
    ```

1. Nella funzione **main**, sotto il commento **Ottieni impostazioni di configurazione**, notare che il codice carica la chiave e l'area definite nel file di configurazione.

1. Trovare il codice seguente sotto il commento **Configurare la traduzione** e aggiungere il codice seguente per configurare la connessione all'endpoint Voce di Servizi di Azure AI:

    ```python
   # Configure translation
   translation_config = speech_sdk.translation.SpeechTranslationConfig(speech_key, speech_region)
   translation_config.speech_recognition_language = 'en-US'
   translation_config.add_target_language('fr')
   translation_config.add_target_language('es')
   translation_config.add_target_language('hi')
   print('Ready to translate from',translation_config.speech_recognition_language)
    ```

1. **SpeechTranslationConfig** verrà usato per tradurre il parlato in testo, ma verrà usato anche **SpeechConfig** per sintetizzare le traduzioni in parlato. Aggiungere il codice seguente sotto il commento **Configure translation**:

    ```python
   # Configure speech
   speech_config = speech_sdk.SpeechConfig(speech_key, speech_region)
   print('Ready to use speech service in:', speech_config.region)
    ```

1. Salvare le modifiche (*CTRL+S*), ma lasciare aperto l'editor di codice.

## Eseguire l'app

Finora, l'app non fa niente di diverso dalla connessione alla risorsa Voce di Azure AI, ma è utile eseguirla e verificare che funzioni prima di aggiungere la funzionalità Voce.

1. Nella riga di comando immettere il seguente comando per eseguire l'app Traduttore:

    ```
   python translator.py
    ```

    Il codice visualizzerà l'area della risorsa del servizio Voce che verrà usata dall'applicazione, un messaggio che indica che il servizio è pronto per la traduzione da en-US e richiedere una lingua di destinazione. Un'esecuzione riuscita indica che l'app è connessa al servizio Voce di Azure AI. Premere INVIO per terminare il programma.

## Implementare la traduzione vocale

Ora che è disponibile un'istanza **SpeechTranslationConfig** per il servizio Voce di Azure AI, è quindi possibile usare l'API di traduzione di Voce di Azure AI per riconoscere e tradurre il parlato.

1. Nel file di codice si noti che il codice usa la funzione **Translate** per tradurre l'input parlato. Nella funzione **Translate** sotto il commento **Translate speech** aggiungere quindi il codice seguente per creare un client **TranslationRecognizer** che può essere usato per riconoscere e tradurre il parlato da un file.

    ```python
   # Translate speech
   current_dir = os.getcwd()
   audioFile = current_dir + '/station.wav'
   audio_config_in = speech_sdk.AudioConfig(filename=audioFile)
   translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config = audio_config_in)
   print("Getting speech from file...")
   result = translator.recognize_once_async().get()
   print('Translating "{}"'.format(result.text))
   translation = result.translations[targetLanguage]
   print(translation)
    ```

1. Salvare le modifiche (*CTRL+S*) ed eseguire di nuovo il programma:

    ```
   python translator.py
    ```

1. Quando richiesto, immettere un codice di lingua valido (*fr*, *es* o *hi*). Il programma trascriverà il file di input e tradurlo nella lingua specificata (francese, spagnolo o hindi). Ripetere questo processo, provando ogni lingua supportata dall'applicazione.

    > **NOTA**: È possibile che la traduzione in hindi non venga sempre visualizzata correttamente nella finestra della console a causa di problemi di codifica dei caratteri.

1. Al termine, premere INVIO per terminare il programma.

> **NOTA**: Il codice nell'applicazione traduce l'input in tutte e tre le lingue in un'unica chiamata. Viene visualizzata solo la traduzione per la lingua specifica, ma è possibile recuperare qualsiasi traduzione specificando il codice della lingua di destinazione nella raccolta **translations** del risultato.

## Sintetizzare la traduzione in parlato

Fino a questo punto, l'applicazione traduce l'input vocale in testo, che potrebbe risultare sufficiente se è necessario richiedere assistenza a qualcuno durante un viaggio. Sarebbe tuttavia meglio se la traduzione fosse pronunciata a voce alta con una voce appropriata.

> **Nota**: A causa delle limitazioni hardware di Cloud Shell, l'output vocale sintetizzato verrà indirizzato a un file.

1. Nella funzione **Translate** trovare il commento **Sintetizzare la traduzione** e aggiungere il codice seguente per usare un client **SpeechSynthesizer** per sintetizzare la traduzione in parlato e salvarla come file con estensione wav:

    ```python
   # Synthesize translation
   output_file = "output.wav"
   voices = {
            "fr": "fr-FR-HenriNeural",
            "es": "es-ES-ElviraNeural",
            "hi": "hi-IN-MadhurNeural"
   }
   speech_config.speech_synthesis_voice_name = voices.get(targetLanguage)
   audio_config_out = speech_sdk.audio.AudioConfig(filename=output_file)
   speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config_out)
   speak = speech_synthesizer.speak_text_async(translation).get()
   if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
   else:
        print("Spoken output saved in " + output_file)
    ```

1. Salvare le modifiche (*CTRL+S*) ed eseguire di nuovo il programma:

    ```
   python translator.py
    ```

1. Esaminare l'output dell'applicazione, che indicherà che la traduzione dell'output parlato è stata salvata in un file. Al termine, premere **INVIO** per terminare il programma.
1. Se si dispone di un lettore multimediale in grado di riprodurre file audio con estensione wav, scaricare il file generato immettendo il comando seguente:

    ```
   download ./output.wav
    ```

    Il comando download crea un collegamento popup in basso a destra del browser, che è possibile selezionare per scaricare e aprire il file.

> **NOTA**
> *In questo esempio è stata usata un'istanza **SpeechTranslationConfig** per tradurre il parlato in testo e quindi è stato usato un **SpeechConfig** per sintetizzare la traduzione come voce. È infatti possibile usare **SpeechTranslationConfig** per sintetizzare direttamente la traduzione, ma questo approccio funziona solo quando si traduce in una singola lingua e restituisce un flusso audio che viene in genere salvato come file.*

## Pulire le risorse

Se si è terminato di esplorare il servizio Voce di Azure AI, è possibile eliminare le risorse create in questo esercizio. Ecco come:

1. Chiudere il riquadro Azure Cloud Shell.
1. Nel portale di Azure passare alla risorsa Voce di Azure AI creata in questo lab.
1. Nella pagina della risorsa, selezionare **Elimina** e seguire le istruzioni per eliminare la risorsa.

## E se si dispone di microfono e altoparlante?

Come input vocale per questo esercizio si sono utilizzati file audio per input e output vocale. Vediamo come modificare il codice per usare l'hardware audio.

### Uso della traduzione vocale con un microfono

1. Se si dispone di un microfono, è possibile usare il codice seguente per acquisire l'input vocale per la traduzione vocale:

    ```python
   # Translate speech
   audio_config_in = speech_sdk.AudioConfig(use_default_microphone=True)
   translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config = audio_config_in)
   print("Speak now...")
   result = translator.recognize_once_async().get()
   print('Translating "{}"'.format(result.text))
   translation = result.translations[targetLanguage]
   print(translation)
    ```

> **Nota**: il microfono predefinito del sistema è l'input audio predefinito, quindi è possibile anche omettere completamente AudioConfig!

### Uso della sintesi vocale con un altoparlante

1. Se si dispone di un altoparlante, è possibile usare il codice seguente per sintetizzare il parlato.
    
    ```python
   # Synthesize translation
   voices = {
            "fr": "fr-FR-HenriNeural",
            "es": "es-ES-ElviraNeural",
            "hi": "hi-IN-MadhurNeural"
   }
   speech_config.speech_synthesis_voice_name = voices.get(targetLanguage)
   audio_config_out = speech_sdk.audio.AudioConfig(use_default_speaker=True)
   speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config_out)
   speak = speech_synthesizer.speak_text_async(translation).get()
   if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

> **Nota**: l'altoparlante predefinito del sistema è l'output audio predefinito, quindi è possibile anche omettere completamente AudioConfig!

## Ulteriori informazioni

Per altre informazioni sull'uso dell'API di Voce di Azure AI, vedere la [documentazione di Traduzione vocale](https://learn.microsoft.com/azure/ai-services/speech-service/speech-translation).
