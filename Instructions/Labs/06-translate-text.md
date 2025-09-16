---
lab:
  title: Tradurre il testo
  description: Tradurre il testo fornito tra le lingue supportate con Traduttore per Azure AI.
---

# Tradurre il testo

**Traduttore per Azure AI** è un servizio che consente di tradurre testo tra lingue. In questo esercizio verrà usato per creare una semplice app che traduce l’input in una qualsiasi lingua supportata nella lingua di destinazione desiderata.

Anche se questo esercizio è basato su Python, è possibile sviluppare applicazioni di traduzione testo usando più SDK specifici del linguaggio, tra cui:

- [Libreria client di traduzione di Azure AI per Python](https://pypi.org/project/azure-ai-translation-text/)
- [Libreria client di traduzione di Azure AI per .NET](https://www.nuget.org/packages/Azure.AI.Translation.Text)
- [Libreria client di traduzione di Azure AI per JavaScript](https://www.npmjs.com/package/@azure-rest/ai-translation-text)

Questo esercizio richiede circa **30** minuti.

## Effettuare il provisioning di una risorsa di *Traduttore per Azure AI*

Se non è già disponibile una risorsa di **Traduttore per Azure AI** nella sottoscrizione, è necessario effettuare il provisioning di una.

1. Aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.
1. Nel campo di ricerca nella parte superiore cercare **Translator** e quindi selezionare **Translator** nei risultati.
1. Creare una risorsa con le impostazioni seguenti:
    - **Sottoscrizione**: *la sottoscrizione di Azure usata*
    - **Gruppo di risorse**: *Scegliere o creare un gruppo di risorse*
    - **Area**: *scegliere una qualsiasi area disponibile*
    - **Nome**: *immettere un nome univoco*
    - **Piano tariffario**: Selezionare **F0** (*gratuito*) o **S** (*standard*) se F non è disponibile.
1. Selezionare **Rivedi e crea**, quindi **Crea** per effettuare il provisioning della risorsa.
1. Attendere il completamento della distribuzione e quindi passare alla risorsa distribuita.
1. Visualizzare la pagina **Chiavi ed endpoint**. Le informazioni contenute in questa pagina saranno necessarie più avanti nell'esercizio.

## Preparare lo sviluppo di un'app in Cloud Shell

Per testare le funzionalità di traduzione testo di Traduttore per Azure AI, si svilupperà una semplice applicazione console in Azure Cloud Shell.

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
   cd mslearn-ai-language/Labfiles/06-translator-sdk/Python/translate-text
    ```

## Configurare l'applicazione

1. Nel riquadro della riga di comando eseguire il comando seguente per visualizzare i file di codice nella cartella **translate-text**:

    ```
   ls -a -l
    ```

    I file includono un file di configurazione (con estensione **env**) e un file di codice (**translate.py**).

1. Creare un ambiente virtuale Python e installare il pacchetto SDK di traduzione di Azure AI e altri pacchetti necessari eseguendo il comando seguente:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-translation-text==1.0.1
    ```

1. Immettere il comando seguente per modificare il file di configurazione dell'applicazione:

    ```
   code .env
    ```

    Il file viene aperto in un editor di codice.

1. Aggiornare i valori di configurazione per includere l’**area** e una **chiave** dalla risorsa di Traduttore per Azure AI creata, disponibile nella pagina **Chiavi ed endpoint** per la risorsa di Traduttore per Azure AI nel portale di Azure.

    > **NOTA**: Assicurarsi di aggiungere l’*area* per la risorsa, <u>non</u> l’endpoint.

1. Dopo aver sostituito i segnaposto con l'editor di codice, usare il comando **CTRL+S** o **Fare clic con il pulsante destro del mouse > Salva** per salvare le modifiche e quindi usare il comando **CTRL+Q** o **Fare clic con il pulsante destro del mouse > Esci** per chiudere l'editor di codice mantenendo aperta la riga di comando di Cloud Shell.

## Aggiungere codice per tradurre testo

1. Immettere il comando seguente per modificare il file di codice dell'applicazione:

    ```
   code translate.py
    ```

1. Esaminare il codice esistente. Si aggiungerà il codice da usare con l'SDK di traduzione di Azure AI.

    > **Suggerimento**: Quando si aggiunge codice al file di codice, assicurarsi di mantenere il rientro corretto.

1. Nella parte superiore del file di codice, sotto i riferimenti allo spazio dei nomi esistenti, trovare il commento **Importa spazi dei nomi** e aggiungere il codice seguente per importare gli spazi dei nomi necessari per usare l'SDK di traduzione:

    ```python
   # import namespaces
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.translation.text import *
   from azure.ai.translation.text.models import InputTextItem
    ```

1. Nella funzione **main** il codice esistente legge le impostazioni di configurazione.
1. Trovare il commento **Creare client usando l’endpoint e la chiave** e aggiungere il codice seguente:

    ```python
   # Create client using endpoint and key
   credential = AzureKeyCredential(translatorKey)
   client = TextTranslationClient(credential=credential, region=translatorRegion)
    ```

1. Trovare il commento **Scegliere la lingua di destinazione** e aggiungere il codice seguente che usa il servizio di traduzione testuale per restituire l’elenco delle lingue supportate per la traduzione e richiede all’utente di selezionare un codice di lingua per la lingua di destinazione.

    ```python
   # Choose target language
   languagesResponse = client.get_supported_languages(scope="translation")
   print("{} languages supported.".format(len(languagesResponse.translation)))
   print("(See https://learn.microsoft.com/azure/ai-services/translator/language-support#translation)")
   print("Enter a target language code for translation (for example, 'en'):")
   targetLanguage = "xx"
   supportedLanguage = False
   while supportedLanguage == False:
        targetLanguage = input()
        if  targetLanguage in languagesResponse.translation.keys():
            supportedLanguage = True
        else:
            print("{} is not a supported language.".format(targetLanguage))
    ```

1. Trovare il commento **Tradurre testo** e aggiungere il codice seguente che chiede ripetutamente testo da tradurre, usa il servizio Traduttore per Azure AI per tradurre nella lingua di destinazione, rilevando la lingua di origine automaticamente, e mostra i risultati fino a quando non si *esce*.

    ```python
   # Translate text
   inputText = ""
   while inputText.lower() != "quit":
        inputText = input("Enter text to translate ('quit' to exit):")
        if inputText != "quit":
            input_text_elements = [InputTextItem(text=inputText)]
            translationResponse = client.translate(body=input_text_elements, to_language=[targetLanguage])
            translation = translationResponse[0] if translationResponse else None
            if translation:
                sourceLanguage = translation.detected_language
                for translated_text in translation.translations:
                    print(f"'{inputText}' was translated from {sourceLanguage.language} to {translated_text.to} as '{translated_text.text}'.")
    ```

1. Salvare le modifiche (CTRL+S), quindi immettere il comando seguente per eseguire il programma (ingrandire il riquadro Cloud Shell e ridimensionare i pannelli per visualizzare altro testo nel riquadro della riga di comando):

    ```
   python translate.py
    ```

1. Quando richiesto, immettere una lingua di destinazione valida dall’elenco visualizzato.
1. Immettere una frase da tradurre, ad esempio `This is a test` o `C'est un test`, e visualizzare i risultati, che devono rilevare la lingua di origine e tradurre il testo nella lingua di destinazione.
1. Al termine, immettere `quit`. È possibile eseguire di nuovo l’applicazione e scegliere una lingua di destinazione diversa.

## Pulire le risorse

Se si è terminato di esplorare il servizio Traduttore per Azure AI, è possibile eliminare le risorse create in questo esercizio. Ecco come:

1. Chiudere il riquadro Azure Cloud Shell.
1. Nel portale di Azure passare alla risorsa Traduttore per Azure AI creata in questo lab.
1. Nella pagina della risorsa, selezionare **Elimina** e seguire le istruzioni per eliminare la risorsa.

## Ulteriori informazioni

Per altre informazioni sull'uso di **Traduttore per Azure AI**, vedere la [documentazione di Traduttore per Azure AI](https://learn.microsoft.com/azure/ai-services/translator/).
