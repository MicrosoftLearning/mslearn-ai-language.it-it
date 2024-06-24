---
lab:
  title: Tradurre il testo
  module: Module 3 - Getting Started with Natural Language Processing
---

# Tradurre il testo

**Traduttore per Azure AI** è un servizio che consente di tradurre testo tra lingue. In questo esercizio verrà usato per creare una semplice app che traduce l’input in una qualsiasi lingua supportata nella lingua di destinazione desiderata.

## Effettuare il provisioning di una risorsa di *Traduttore per Azure AI*

Se non è già disponibile una risorsa di **Traduttore per Azure AI** nella sottoscrizione, è necessario effettuare il provisioning di una.

1. Aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.
1. Nel campo di ricerca nella parte superiore, cercare **Servizi di Azure AI** e premere **INVIO**, quindi selezionare **Crea** in **Traduttore** nei risultati.
1. Creare una risorsa con le impostazioni seguenti:
    - **Sottoscrizione**: *la sottoscrizione di Azure usata*
    - **Gruppo di risorse**: *Scegliere o creare un gruppo di risorse*
    - **Area**: *scegliere una qualsiasi area disponibile*
    - **Nome**: *immettere un nome univoco*
    - **Piano tariffario**: Selezionare **F0**, *gratuito*, o **S**, *Standard*, se F non è disponibile.
    - **Avviso sull’intelligenza artificiale responsabile**: Accettare.
1. Selezionare **Rivedi e crea**, quindi **Crea** per effettuare il provisioning della risorsa.
1. Attendere il completamento della distribuzione e quindi passare alla risorsa distribuita.
1. Visualizzare la pagina **Chiavi ed endpoint**. Le informazioni contenute in questa pagina saranno necessarie più avanti nell’esercizio.

## Prepararsi a sviluppare un’app in Visual Studio Code

Si svilupperà l’app di traduzione testo usando Visual Studio Code. I file di codice per l’app sono stati forniti in un repository GitHub.

> **Suggerimento**: Se è già stato clonato il repository **mslearn-ai-language**, aprirlo in Visual Studio Code. In caso contrario, seguire questa procedura per clonarlo nell’ambiente di sviluppo.

1. Avviare Visual Studio Code.
2. Aprire il riquadro comandi (MAIUSC+CTRL+P) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/mslearn-ai-language` in una cartella locale. Non è importante usare una cartella specifica.
3. Dopo la clonazione del repository, aprire la cartella in Visual Studio Code.

    > **Nota**: Se Visual Studio Code visualizza un messaggio popup per chiedere di considerare attendibile il codice che si sta aprendo, fare clic sull’opzione **Sì, si considerano attendibili gli autori** nel popup.

4. Attendere il completamento dell'installazione di file aggiuntivi per supportare i progetti in codice C# nel repository.

    > **Nota**: se viene richiesto di aggiungere gli asset necessari per la compilazione e il debug, selezionare **Non adesso**.

## Configurare l'applicazione

Sono state fornite applicazioni sia per C# che per Python. Entrambe le app presentano la stessa funzionalità. Prima di tutto, verranno completate alcune parti chiave dell’applicazione per abilitarla all'uso della risorsa di Traduttore per Azure AI.

1. Nel riquadro **Explorer** in Visual Studio Code, passare alla cartella **Labfiles/06b-translator-sdk** ed espandere la cartella **CSharp** o **Python** in base alle preferenze di lingua e alla cartella **translate-text** in essa contenuta. Ogni cartella contiene i file di codice specifici della lingua per un’app in cui si intende integrare la funzionalità di Traduttore per Azure AI.
2. Fare clic con il pulsante destro del mouse sulla cartella **translate-text** contenente i file di codice e aprire un terminale integrato. Installare quindi il pacchetto dell’SDK di Traduttore per Azure AI eseguendo il comando appropriato per le preferenze linguistiche:

    **C#:**

    ```
    dotnet add package Azure.AI.Translation.Text --version 1.0.0-beta.1
    ```

    **Python**:

    ```
    pip install azure-ai-translation-text==1.0.0b1
    ```

3. Nel riquadro **Explorer** nella cartella **translate-text**, aprire il file di configurazione per la lingua preferita

    - **C#**: appsettings.json
    - **Python**: .env
    
4. Aggiornare i valori di configurazione per includere l’**area** e una **chiave** dalla risorsa di Traduttore per Azure AI creata, disponibile nella pagina **Chiavi ed endpoint** per la risorsa di Traduttore per Azure AI nel portale di Azure.

    > **NOTA**: Assicurarsi di aggiungere l’*area* per la risorsa, <u>non</u> l’endpoint.

5. Salvare il file di configurazione.

## Aggiungere codice per tradurre testo

A questo punto è possibile usare Traduttore per Azure AI per tradurre testo.

1. La cartella **translate-text** contiene un file di codice per l’applicazione client:

    - **C#**: Program.cs
    - **Python**: translate.py

    Aprire il file di codice e nella parte superiore, sotto i riferimenti agli spazi dei nomi esistenti, individuare il commento **Import namespaces**. In questo commento aggiungere quindi il codice seguente specifico per la lingua per importare gli spazi dei nomi necessari per usare Text Analytics SDK:

    **C#**: Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.Translation.Text;
    ```

    **Python**: translate.py

    ```python
    # import namespaces
    from azure.ai.translation.text import *
    from azure.ai.translation.text.models import InputTextItem
    ```

1. Nella funzione **Main** il codice esistente legge le impostazioni di configurazione.
1. Trovare il commento **Creare client usando l’endpoint e la chiave** e aggiungere il codice seguente:

    **C#**: Programs.cs

    ```csharp
    // Create client using endpoint and key
    AzureKeyCredential credential = new(translatorKey);
    TextTranslationClient client = new(credential, translatorRegion);
    ```

    **Python**: translate.py

    ```python
    # Create client using endpoint and key
    credential = TranslatorCredential(translatorKey, translatorRegion)
    client = TextTranslationClient(credential)
    ```

1. Trovare il commento **Scegliere la lingua di destinazione** e aggiungere il codice seguente che usa il servizio di traduzione testuale per restituire l’elenco delle lingue supportate per la traduzione e richiede all’utente di selezionare un codice di lingua per la lingua di destinazione.

    **C#**: Programs.cs

    ```csharp
    // Choose target language
    Response<GetLanguagesResult> languagesResponse = await client.GetLanguagesAsync(scope:"translation").ConfigureAwait(false);
    GetLanguagesResult languages = languagesResponse.Value;
    Console.WriteLine($"{languages.Translation.Count} languages available.\n(See https://learn.microsoft.com/azure/ai-services/translator/language-support#translation)");
    Console.WriteLine("Enter a target language code for translation (for example, 'en'):");
    string targetLanguage = "xx";
    bool languageSupported = false;
    while (!languageSupported)
    {
        targetLanguage = Console.ReadLine();
        if (languages.Translation.ContainsKey(targetLanguage))
        {
            languageSupported = true;
        }
        else
        {
            Console.WriteLine($"{targetLanguage} is not a supported language.");
        }

    }
    ```

    **Python**: translate.py

    ```python
    # Choose target language
    languagesResponse = client.get_languages(scope="translation")
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

    **C#**: Programs.cs

    ```csharp
    // Translate text
    string inputText = "";
    while (inputText.ToLower() != "quit")
    {
        Console.WriteLine("Enter text to translate ('quit' to exit)");
        inputText = Console.ReadLine();
        if (inputText.ToLower() != "quit")
        {
            Response<IReadOnlyList<TranslatedTextItem>> translationResponse = await client.TranslateAsync(targetLanguage, inputText).ConfigureAwait(false);
            IReadOnlyList<TranslatedTextItem> translations = translationResponse.Value;
            TranslatedTextItem translation = translations[0];
            string sourceLanguage = translation?.DetectedLanguage?.Language;
            Console.WriteLine($"'{inputText}' translated from {sourceLanguage} to {translation?.Translations[0].To} as '{translation?.Translations?[0]?.Text}'.");
        }
    } 
    ```

    **Python**: translate.py

    ```python
    # Translate text
    inputText = ""
    while inputText.lower() != "quit":
        inputText = input("Enter text to translate ('quit' to exit):")
        if inputText != "quit":
            input_text_elements = [InputTextItem(text=inputText)]
            translationResponse = client.translate(content=input_text_elements, to=[targetLanguage])
            translation = translationResponse[0] if translationResponse else None
            if translation:
                sourceLanguage = translation.detected_language
                for translated_text in translation.translations:
                    print(f"'{inputText}' was translated from {sourceLanguage.language} to {translated_text.to} as '{translated_text.text}'.")
    ```

1. Salvare le modifiche apportate al file di codice.

## Testare l'applicazione

A questo punto l’applicazione è pronta per il test.

1. Nel terminale integrato per la cartella **Tradurre testo**, immettere il comando seguente ed eseguire il programma:

    - **C#**: `dotnet run`
    - **Python**: `python translate.py`

    > **Suggerimento**: È possibile usare l’icona **Ingrandire le dimensioni del pannello** (**^**) nella barra degli strumenti del terminale per visualizzare altro testo della console.

1. Quando richiesto, immettere una lingua di destinazione valida dall’elenco visualizzato.
1. Immettere una frase da tradurre, ad esempio `This is a test` o `C'est un test`, e visualizzare i risultati, che devono rilevare la lingua di origine e tradurre il testo nella lingua di destinazione.
1. Al termine, immettere `quit`. È possibile eseguire di nuovo l’applicazione e scegliere una lingua di destinazione diversa.

## Eseguire la pulizia

Quando il progetto non è più necessario, è possibile eliminare la risorsa di Traduttore per Azure AI nel [portale di Azure](https://portal.azure.com).
