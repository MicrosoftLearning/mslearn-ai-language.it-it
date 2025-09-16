---
lab:
  title: Riconoscimento e sintesi vocale
  description: Implementare un orologio parlante che converte la voce in testo e il testo in voce.
---

# Riconoscimento e sintesi vocale

**Voce di Azure AI** è un servizio che fornisce funzionalità correlate al riconoscimento vocale, tra cui:

- Un'API di *conversione della voce in testo scritto* che consente di implementare il riconoscimento vocale (convertendo parole vocali udibili in testo).
- Un'API *sintesi vocale* che consente di implementare la sintesi vocale (convertendo il testo in voce udibile).

In questo esercizio si useranno entrambe queste API per implementare un'applicazione di orologio vocale.

Anche se questo esercizio è basato su Python, è possibile sviluppare applicazioni vocali usando più SDK specifici del linguaggio, tra cui:

- [SDK di Voce di Azure AI per Python](https://pypi.org/project/azure-cognitiveservices-speech/)
- [SDK di Voce di Azure AI per .NET](https://www.nuget.org/packages/Microsoft.CognitiveServices.Speech)
- [SDK di Voce di Azure AI per JavaScript](https://www.npmjs.com/package/microsoft-cognitiveservices-speech-sdk)

Questo esercizio richiede circa **30** minuti.

> **NOTA** Questo esercizio è progettato per essere completato in Azure Cloud Shell, in cui l'accesso diretto all'hardware audio del computer non è supportato. Il lab userà quindi i file audio per i flussi di input e output vocale. Il codice per ottenere gli stessi risultati usando un microfono e un altoparlante viene fornito come riferimento.

## Creare una risorsa per Voce di Azure AI

Per iniziare, creare una risorsa di Voce di Azure AI.

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

## Preparare e configurare l'app orologio parlante

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

1. Dopo aver clonato il repository, passare alla cartella contenente i file di codice dell'applicazione orologio parlante:  

    ```
   cd mslearn-ai-language/Labfiles/07-speech/Python/speaking-clock
    ```

1. Nel riquadro della riga di comando eseguire il comando seguente per visualizzare i file di codice nella cartella **speaking-clock**:

    ```
   ls -a -l
    ```

    I file includono un file di configurazione (con estensione **env**) e un file di codice (**speaking-clock.py**). I file audio usati dall'applicazione si trovano nella sottocartella **audio**.

1. Creare un ambiente virtuale Python e installare il pacchetto SDK di Voce di Azure AI e altri pacchetti necessari eseguendo il comando seguente:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-cognitiveservices-speech==1.42.0
    ```

1. Immettere il comando seguente per modificare il file di configurazione:

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
   code speaking-clock.py
    ```

1. Nella parte superiore del file di codice, sotto i riferimenti agli spazi dei nomi esistenti, individuare il commento **Import namespaces**. In questo commento aggiungere quindi il codice seguente specifico della lingua per importare gli spazi dei nomi necessari per usare l'SDK di Voce di Azure AI:

    ```python
   # Import namespaces
   from azure.core.credentials import AzureKeyCredential
   import azure.cognitiveservices.speech as speech_sdk
    ```

1. Nella funzione **main**, sotto il commento **Ottieni impostazioni di configurazione**, notare che il codice carica la chiave e l'area definite nel file di configurazione.

1. Trovare il commento **Configurare il servizio Voce** e aggiungere il codice seguente per usare la chiave di Servizi di Azure AI e l'area per configurare la connessione all'endpoint Voce di Servizi di Azure AI

    ```python
   # Configure speech service
   speech_config = speech_sdk.SpeechConfig(speech_key, speech_region)
   print('Ready to use speech service in:', speech_config.region)
    ```

1. Salvare le modifiche (*CTRL+S*), ma lasciare aperto l'editor di codice.

## Eseguire l'app

Finora, l'app non fa niente di diverso dalla connessione al servizio Voce di Azure AI, ma è utile eseguirla e verificare che funzioni prima di aggiungere la funzionalità Voce.

1. Nella riga di comando immettere il seguente comando per eseguire l'app orologio parlante:

    ```
   python speaking-clock.py
    ```

    Il codice dovrebbe visualizzare l'area della risorsa del servizio Voce che verrà utilizzata dall'applicazione. Un'esecuzione riuscita indica che l'app è connessa alla risorsa Voce di Azure AI.

## Aggiungere codice per il riconoscimento vocale

È ora disponibile un elemento **SpeechConfig** per il servizio Voce nella risorsa Servizi di Azure AI nel progetto, è quindi possibile usare l'API di **conversione della voce in testo scritto** per riconoscere il parlato e trascriverlo in testo.

In questa procedura, l'input vocale viene acquisito da un file audio, che è possibile riprodurre qui:

<video controls src="https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Instructions/media/Time.mp4" title="What time is it?" width="150"></video>

1. Nel file di codice si noti che il codice usa la funzione **TranscribeCommand** per accettare l'input vocale. Nella funzione **TranscribeCommand**, trovare il commento **Configurare il riconoscimento vocale** e aggiungere il codice appropriato seguente per creare un client **SpeechRecognizer** che può essere usato per riconoscere e trascrivere il parlato da un file audio:

    ```python
   # Configure speech recognition
   current_dir = os.getcwd()
   audioFile = current_dir + '/time.wav'
   audio_config = speech_sdk.AudioConfig(filename=audioFile)
   speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    ```

1. Nella funzione **TranscribeCommand**, sotto il commento **Process speech input**, aggiungere il codice seguente per l'ascolto dell'input vocale, prestando attenzione a non sostituire il codice alla fine della funzione che restituisce il comando:

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

1. Salvare le modifiche (*CTRL+S*) e quindi nella riga di comando sotto l'editor di codice rieseguire il programma:
1. Esaminare l'output che dovrebbe "ascoltare" correttamente il parlato nel file audio e restituire una risposta appropriata (si noti che Azure Cloud Shell potrebbe essere in esecuzione in un server che si trova in un fuso orario diverso rispetto all'utente).

    > **Suggerimento**: se SpeechRecognizer rileva un errore, viene generato il risultato "Annullato". Il codice nell'applicazione visualizza quindi il messaggio di errore. La causa più probabile è un valore di area non corretto nel file di configurazione.

## Sintesi vocale

L'applicazione dell'orologio vocale accetta l'input vocale, ma in realtà non parla. Per risolvere questo problema, aggiungere codice per sintetizzare il parlato.

Ancora una volta, a causa delle limitazioni hardware di Cloud Shell, l'output vocale sintetizzato verrà diretto a un file.

1. Nel file di codice si noti che il codice usa la funzione **TellTime** per indicare all'utente l'ora corrente.
1. Nella funzione **TellTime**, sotto il commento **Configure speech synthesis**, aggiungere il codice seguente per creare un client **SpeechSynthesizer** che può essere usato per generare l'output vocale:

    ```python
   # Configure speech synthesis
   output_file = "output.wav"
   speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
   audio_config = speech_sdk.audio.AudioConfig(filename=output_file)
   speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config,)
    ```

1. Nella funzione **TellTime**, sotto il commento **Synthesize spoken output**, aggiungere il codice seguente per generare l'output vocale, prestando attenzione a non sostituire il codice alla fine della funzione che stampa la risposta:

    ```python
   # Synthesize spoken output
   speak = speech_synthesizer.speak_text_async(response_text).get()
   if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
   else:
        print("Spoken output saved in " + output_file)
    ```

1. Salvare le modifiche (*CTRL+S*) ed eseguire nuovamente il programma, che dovrebbe indicare che l'output parlato è stato salvato in un file.

1. Se si dispone di un lettore multimediale in grado di riprodurre file audio con estensione wav, scaricare il file generato immettendo il comando seguente:

    ```
   download ./output.wav
    ```

    Il comando download crea un collegamento popup in basso a destra del browser, che è possibile selezionare per scaricare e aprire il file.

    Il suono del file dovrebbe risultare simile al seguente:

    <video controls src="https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Instructions/media/Output.mp4" title="Sono le 2:15" width="150"></video>

## Usare Speech Synthesis Markup Language

Speech Synthesis Markup Language (SSML) consente di personalizzare la modalità di sintesi vocale usando un formato basato su XML.

1. Nella funzione **TellTime** sostituire tutto il codice corrente sotto il commento **Synthesize spoken output** con il codice seguente (lasciare il codice sotto il commento **Print the response**):

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
       print("Spoken output saved in " + output_file)
    ```

1. Salvare le modifiche ed eseguire nuovamente il programma, che dovrebbe indicare di nuovo che l'output parlato è stato salvato in un file.
1. Scaricare e riprodurre il file generato, che dovrebbe essere simile al seguente:
    
    <video controls src="https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Instructions/media/Output2.mp4" title="Sono le 5:30. Ora di finire il lab." width="150"></video>

## (FACOLTATIVO) E se si dispone di microfono e altoparlante?

Come input vocale per questo esercizio si sono utilizzati file audio per input e output vocale. Vediamo come modificare il codice per usare l'hardware audio.

### Uso del riconoscimento vocale con un microfono

Se si dispone di un microfono, è possibile usare il codice seguente per acquisire l'input vocale per il riconoscimento vocale:

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

> **Nota**: il microfono predefinito del sistema è l'input audio predefinito, quindi è possibile anche omettere completamente AudioConfig!

### Uso della sintesi vocale con un altoparlante

Se si dispone di un altoparlante, è possibile usare il codice seguente per sintetizzare il parlato.

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

> **Nota**: l'altoparlante predefinito del sistema è l'output audio predefinito, quindi è possibile anche omettere completamente AudioConfig!

## Eseguire la pulizia

Al termine dell'esplorazione di Voce di Azure AI, è necessario eliminare le risorse create in questo esercizio per evitare di incorrere in costi di Azure non necessari.

1. Chiudere il riquadro Azure Cloud Shell.
1. Nel portale di Azure passare alla risorsa Voce di Azure AI creata in questo lab.
1. Nella pagina della risorsa, selezionare **Elimina** e seguire le istruzioni per eliminare la risorsa.

## Ulteriori informazioni

Per altre informazioni sull'uso delle API di **conversione della voce in testo scritto** e **sintesi vocale**, vedere la [documentazione sulla conversione della voce in testo scritto](https://learn.microsoft.com/azure/ai-services/speech-service/index-speech-to-text) e la [documentazione sulla sintesi vocale](https://learn.microsoft.com/azure/ai-services/speech-service/index-text-to-speech).
