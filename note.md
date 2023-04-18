# costruzione della estensione:

il linguaggio regole è fatto di commenti // o /* */ come nel c++, di comandi, che iniziano con il #, come nel preprocessor c++ e da variabili che hanno la sintassi ${nomevar} o $nomevar. oltre a questo vi sono espressioni matematiche che sono racchiuse tra parentesi graffe:

il linguaggio ha estensione .RG, vorrei formattare questo linguaggio in microsoft code, creando un apposita estensione.  vorrei anche abilitare la colorazione del linguaggio. si può fare? 

- Installa Node.js: Per creare un'estensione per Visual Studio Code, avrai bisogno di Node.js installato sul tuo computer. Puoi scaricarlo da https://nodejs.org/.
-  Installa Yeoman e il generatore di estensioni di Visual Studio Code: Yeoman è uno strumento che ti aiuta a creare rapidamente nuove estensioni per Visual Studio Code. 

```bash
npm install -g yo generator-code
```

Crea una nuova estensione: Esegui il comando `yo code` nel terminale per avviare il generatore di estensioni. Verranno richieste alcune informazioni, come il nome dell'estensione, l'identificativo, la descrizione e il tipo di estensione. Seleziona "New Language" come tipo di estensione.

Apri la cartella dell'estensione in Visual Studio Code: Dopo aver creato l'estensione, apri la cartella dell'estensione in Visual Studio Code. 

Modbashifica il file package.json per aggiungere il tuo linguaggio personalizzato. Aggiungi un oggetto "languages" con le seguenti proprietà: "id" (l'identificativo del linguaggio), "aliases" (nomi alternativi per il linguaggio) e "extensions" (l'estensione del file, in questo caso `.rg`).

Definisci la grammatica del linguaggio: La grammatica del linguaggio viene solitamente definita utilizzando un file TextMate. 


Esempio:
```bash
 "contributes": {
    "languages": [{
      "id": "reg",
      "aliases": ["langreg", "reg"],
      "extensions": [".rg",".var",".regole",".varianti"],
      "configuration": "./language-configuration.json"
    }],
    "grammars": [{
      "language": "reg",
      "scopeName": "source.reg",
      "path": "./syntaxes/reg.tmLanguage.json"
    }]
  },
  "repository": {
    "type": "git",
    "url": "https://github.com/liburno/langreg.git"
  }

```


Crea un nuovo file JSON (ad esempio, myCustomLanguage.tmLanguage.json) nella cartella `syntaxes` dell'estensione e definisci la grammatica del tuo linguaggio. 

```bash
{
  "$schema": "https://raw.githubusercontent.com/martinring/tmlanguage/master/tmlanguage.json",
  "name": "MyCustomLanguage",
  "patterns": [
    {
      "include": "#comments"
    },
    {
      "include": "#commands"
    },
   bash {
      "include": "#variables"
    },
    {
      "include": "#expressions"
    }
  ],
  "repository": {
    "comments": {
      "patterns": [
        {
          "name": "comment.line.double-slash",
          "match": "//.*"
        },
        {
          "name": "comment.block",
          "begin": "/\\*",
          "end": "\\*/"
        }
      ]
    },
    "commands": {
      "patterns": [
        {
          "name": "keyword.control.command",
          "match": "#[a-zA-Z]+"
        }
      ]
    },
    "variables": {
      "patterns": [
        {
          "name": "variable.other.readwrite",
          "match": "\\$\\{?[a-zA-Z_][a-zA-Z0-9_]*\\}?"
        }
      ]
    },
    "expressions": {
      "patterns": [
        {
          "name": "meta.expression",
          "begin": "\\{",
          "end": "\\}",
          "patterns": [
            {
              "include": "#comments"
            },
            {
              "include": "#commands"
            },
            {
              "include": "#variables"
            },
            {
              "include": "#expressions"
            }
          ]
        }
      ]
    }
  },
  "scopeName": "source.myCustomLanguage"
}
```


# formattazione automatica

Per abilitare la formattazione automatica nel tuo linguaggio personalizzato, è necessario implementare un formattatore all'interno della tua estensione per Visual Studio Code. Ecco una panoramica di come farlo:

Installa `vscode-languageclient` e `vscode-languageserver`: 

Nel terminale, naviga nella cartella della tua estensione e installa i pacchetti 
```bash
npm install vscode-languageclient vscode-languageserver
```

- Crea il file del server di linguaggio: Crea un nuovo file, ad esempio languageServer.js, nella cartella della tua estensione. In questo file, implementerai il server di linguaggio che gestirà la formattazione del codice.
- Implementa il server di linguaggio: Importa i moduli necessari e configura il server di linguaggio nel tuo file languageServer.js. Ad esempio:

```
const { createConnection, TextDocuments, ProposedFeatures, TextEdit } = require('vscode-languageserver');
const { TextDocument } = require('vscode-languageserver-textdocument');

const connection = createConnection(ProposedFeatures.all);
const documents = new TextDocuments(TextDocument);

connection.onInitialize(() => {
  return {
    capabilities: {
      textDocumentSync: documents.syncKind,
      documentFormattingProvider: true
    }
  };
});

connection.onDocumentFormatting(async (params) => {
  const document = documents.get(params.textDocument.uri);

  // Implementa qui la logica di formattazione
  // Utilizza il testo del documento e i parametri di formattazione per generare gli edit necessari

  const edits = []; // Array di oggetti TextEdit
  return edits;
});

documents.listen(connection);
connection.listen();
```

- Implementa la logica di formattazione: Nella funzione onDocumentFormatting, implementa la logica di formattazione specifica per il tuo linguaggio. È possibile analizzare il testo del documento e generare un array di oggetti TextEdit che rappresentano le modifiche necessarie per formattare correttamente il codice. Ogni TextEdit ha due proprietà principali: range (l'intervallo di testo da modificare) e newText (il testo che sostituisce l'intervallo).

-  Modifica il file extension.js: Nel file extension.js della tua estensione, aggiungi il codice per avviare il server di linguaggio:
```
const path = require('path');
const { LanguageClient } = require('vscode-languageclient/node');

// ...

function activate(context) {
  const serverModule = context.asAbsolutePath(path.join('server', 'languageServer.js'));
  const debugOptions = { execArgv: ['--nolazy', '--inspect=6009'] };

  const serverOptions = {
    run: { module: serverModule, transport: TransportKind.ipc },
    debug: { module: serverModule, transport: TransportKind.ipc, options: debugOptions }
  };

  const clientOptions = {
    documentSelector: [{ scheme: 'file', language: 'myCustomLanguage' }]
  };

  const client = new LanguageClient('myCustomLanguage', 'My Custom Language Server', serverOptions, clientOptions);

  context.subscriptions.push(client.start());
}
```


# pubblicazione 
se non voglio pubblicare l'estensione in vscode, ma la voglio utilizzare localmente cosa posso fare?

Per utilizzare la tua estensione localmente senza pubblicarla nel Visual Studio Code Marketplace, puoi installare l'estensione direttamente dalla tua cartella di sviluppo seguendo questi passaggi:

- Assicurati di aver eseguito npm install e npm run compile nella cartella della tua estensione per installare le dipendenze e compilare il codice sorgente.

- Fai clic sull'icona delle estensioni nella barra laterale a sinistra o premi Ctrl+Shift+X (Windows/Linux) o Cmd+Shift+X (macOS) per aprire il pannello delle estensioni.
- Fai clic sull'icona a forma di ingranaggio nell'angolo in alto a destra del pannello delle estensioni e seleziona "Install from VSIX..." nel menu a discesa.
- Nella finestra di dialogo "Apri file" che appare, accedi alla cartella del tuo progetto di estensione e seleziona il file .vsix che hai creato con vsce package. Se non hai ancora creato il file .vsix, esegui il comando vsce package nella cartella della tua estensione prima di eseguire questo passaggio.
- Fai clic su "Apri" nella finestra di dialogo "Apri file" per installare la tua estensione localmente.
- Dopo aver completato questi passaggi, la tua estensione dovrebbe essere installata e attiva nel tuo Visual Studio Code. Puoi verificare che sia installata controllando la lista delle estensioni installate nel pannello delle estensioni.

Se apporti ulteriori modifiche alla tua estensione, ricompila il codice sorgente con npm run compile, ricrea il file .vsix con vsce package e reinstalla l'estensione seguendo i passaggi sopra descritti



## vsce non è installato nel computer

Se vsce non è installato nel tuo computer, puoi installarlo facilmente utilizzando npm, il gestore dei pacchetti di Node.js. Ecco come fare:

```
npm install -g vsce

```
Dopo aver installato vsce, dovresti essere in grado di utilizzare il comando vsce nel terminale o nel prompt dei comandi per creare un file .vsix.

Ricorda di eseguire `vsce package`  nella cartella della tua estensione per compilare il codice sorgente prima di creare il file .vsix con vsce package.

devi anche modificare il file `README.md`
