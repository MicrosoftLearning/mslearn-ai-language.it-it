# Requisiti

## Esecuzione in Cloud Shell

* Sottoscrizione di Azure con accesso a OpenAI
* In caso di in esecuzione in Azure Cloud Shell, scegliere la shell Bash. L'interfaccia della riga di comando di Azure e Azure Developer CLI sono inclusi in Cloud Shell.

## Eseguire localmente

* È possibile eseguire l'app Web in locale dopo aver eseguito lo script di distribuzione:
    * [Azure Developer CLI (azd)](https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/install-azd)
    * [Interfaccia della riga di comando di Azure](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)
    * Sottoscrizione di Azure con accesso a OpenAI


## Variabili di ambiente

Il file `.env` viene creato dallo script *azdeploy.sh*. L'endpoint del modello di intelligenza artificiale, la chiave API e il nome del modello vengono aggiunti durante la distribuzione delle risorse.

## Distribuzione delle risorse di Azure

L'elemento fornito `azdeploy.sh` crea le risorse necessarie in Azure:

* Modificare le due variabili nella parte superiore dello script in base alle proprie esigenze. Non modificare altro.
* Lo script:
    * Distribuisce il modello *gpt-4o* usando AZD.
    * Crea il servizio Registro Azure Container
    * Usa attività del Registro Azure Container per compilare e distribuire l'immagine del Dockerfile in Registro Azure Container
    * Crea il piano di servizio app
    * Crea l'app Web del servizio app
    * Configura l'app Web per l'immagine del contenitore in Registro Azure Container
    * Configura le variabili di ambiente dell'app Web
    * Lo script fornirà l'endpoint del Servizio app

Lo script offre due opzioni di distribuzione: 1. Distribuzione completa e 2. Ridistribuzione della sola immagine. L'opzione 2 è destinata solo alla fase post-distribuzione, quando si vuole provare ad apportare modifiche nell'applicazione. 

> Nota: È possibile eseguire lo script in PowerShell o Bash, usando il comando `bash azdeploy.sh`. Questo comando consente anche di eseguire lo script in Bash senza doverlo rendere eseguibile.

## Sviluppo locale

### Effettuare il provisioning del modello di intelligenza artificiale in Azure

È possibile eseguire il progetto in locale ed effettuare il provisioning solo del modello di intelligenza artificiale seguendo questa procedura:

1. **Inizializzare l'ambiente** (scegliere un nome descrittivo):

   ```bash
   azd env new gpt-realtime-lab --confirm
   # or: azd env new your-name-gpt-experiment --confirm
   ```
   
   **Importante**: Questo nome diventa parte dei nomi delle risorse di Azure.  
   Il flag `--confirm` imposta questa opzione come ambiente predefinito senza richiedere conferma.

1. **Impostare il gruppo di risorse**:

   ```bash
   azd env set AZURE_RESOURCE_GROUP "rg-your-name-gpt"
   ```

1. **Accedere ed effettuare il provisioning delle risorse di intelligenza artificiale**:

   ```bash
   az login
   azd provision
   ```

    > **Importante**: NON eseguire `azd deploy`: l'app non è configurata nei modelli AZD.

Se è stato effettuato il provisioning solo del modello usando il metodo `azd provision`, È NECESSARIO creare un file `.env` nella radice della directory con le voci seguenti:

```
AZURE_VOICE_LIVE_ENDPOINT=""
AZURE_VOICE_LIVE_API_KEY=""
VOICE_LIVE_MODEL=""
VOICE_LIVE_VOICE="en-US-JennyNeural"
VOICE_LIVE_INSTRUCTIONS="You are a helpful AI assistant with a focus on world history. Respond naturally and conversationally. Keep your responses concise but engaging."
VOICE_LIVE_VERBOSE="" #Suppresses excessive logging to the terminal if running locally
```

Note:

1. L'endpoint è l'endpoint per il modello e deve includere solo `https://<proj-name>.cognitiveservices.azure.com`.
1. La chiave API è la chiave per il modello.
1. Il modello è il nome del modello usato durante la distribuzione.
1. È possibile recuperare questi valori dal portale di Fonderia AI.

### Esecuzione del progetto in locale

Il progetto è stato creato e gestito usando **uv**, ma non è necessario per l'esecuzione. 

Se **uv** è installato:

* Eseguire `uv venv` per creare l'ambiente
* Eseguire `uv sync` per aggiungere pacchetti
* Alias creato per l'app Web: `uv run web` per avviare lo script `flask_app.py`.
* File requirements.txt creato con `uv pip compile pyproject.toml -o requirements.txt`

Se **uv** non è installato:

* Creare un ambiente: `python -m venv .venv`
* Attivare l'ambiente: `.\.venv\Scripts\Activate.ps1`
* Installare le dipendenze: `pip install -r requirements.txt`
* Eseguire l'applicazione (dalla radice del progetto): `python .\src\real_time_voice\flask_app.py`
