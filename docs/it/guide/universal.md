# Scrivere codice Universale

Prima di andare oltre, parliamo un attimo dei vincoli che ci possono essere scrivendo codice "universale", ovvero codice che viene eseguito sia sul server che sul client. A causa delle differenze tra i casi d'uso e le API della piattaforma, il comportamento del codice non sarà esattamente lo stesso durante l'esecuzione in ambienti diversi. Esamineremo qui i concetti chiave di cui essere consapevoli.

## Reattività dei dati sul server

In un'app solo client, ogni utente utilizzerà una nuova istanza dell'applicazione nel proprio browser. Vogliamo ottenere la stessa cosa anche per l'esecuzione lato server: ogni richiesta deve avere un'istanza dell'applicazione nuova e isolata in modo che non vi sia inquinamento dello stato tra le richieste.

Poiché il processo di rendering effettivo deve essere deterministico, provvederemo anche ad un "pre-fetch" dei dati sul server - questo significa che lo stato della nostra applicazione sarà già risolto una volta iniziato il rendering. Ciò significa che la reattività dei dati non è necessaria sul server, quindi è disabilitata per impostazione predefinita. La disabilitazione della reattività dei dati evita anche il dispendio di prestazione per la conversione dei dati in oggetti reattivi.

## Lifecycle Hooks del componente

Dato che non ci sono aggiornamenti dinamici, di tutti i lifecycle hooks, solo `beforeCreate` e `created` saranno chiamati durante il SSR. Questo significa che qualsiasi codice dentro altri lifecycle hooks quali `beforeMount` o `mounted` saranno eseguiti soltanto sul client.

Un'altra cosa da notare è che si dovrebbe evitare codice che produca effetti collaterali globali dentro `beforeCreate` e `created`, per esempio impostando dei timer con `setInterval`. Nel codice solo lato client possiamo impostare un timer e poi distruggerlo in `beforeDestroy` o `destroyed`. Tuttavia, siccome l'hook destroy non verrà chiamato dal SSR, i timer rimarranno attivi per sempre. Per evitarlo, sposta invece il codice con effetti collaterali in `beforeMount` o `mounted`.

## Accesso alle API specifiche della piattaforma

Il codice universale non può desumere l'accesso a API specifiche di piattaforma, perciò se il tuo codice usa direttamente variabili globali che funzionano solo nel browser come `window` o `document`, queste restituiranno errore quando eseguite in  Node.js, e viceversa.

Per attività condivise tra server e client, ma che usano API di piattaforma differenti, si consiglia di racchiudere le implementazioni specifiche di piattaforma dentro una API universale, o usare librerie che lo facciano per te. Per esempio, [axios](https://github.com/axios/axios) è un client HTTP che espone le stesse API sia per server sia per client.

Per API solo browser, l'approccio comune è quello di accedervi in modo lazy all'interno di un lifecycle hook utilizzabile solo dal client.

Tieni conto che se una libreria di terze parti non è stata scritta con considerando la modalità universale, potrebbe essere complicato intgrarlo in un'applicazione eseguita sul server. *Potresti* essere in grado di farlo funzionare imitando alcune delle variabili globali, ma potrebbe creare confusione e interferire con il codice di rilevamento dell'ambiente di altre librerie.

## Directives personalizzate

La maggior parte delle direttive personalizzate manipolano direttamente il DOM e pertanto causerà errori durante il SSR. Esistono due modi per aggirarlo:

1. Preferisci l'utilizzo di componenti come meccanismo di astrazione e lavora invece a livello di DOM virtuale (ad esempio, usando le funzioni di rendering).

2. Se disponi di una direttiva personalizzata che non può essere facilmente sostituita da componenti, puoi fornirne una "versione lato server" della stessa usando l'opzione [`directives`](../api/#directives) durante la creazione del renderer del server.
