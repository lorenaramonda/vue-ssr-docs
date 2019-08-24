# Guida all'esecuzione lato server di Vue.js

::: tip NOTE
La guida richiede le seguenti versioni minime di Vue e delle librerie di supporto:

- vue & vue-server-renderer 2.3.0+
- vue-router 2.5.0+
- vue-loader 12.0.0+ & vue-style-loader 3.0.0+

Se hai precedentemente usato Vue 2.2 con SSR, ti accorgerai che la struttura di codice consigliata è [leggermente diversa](./guide/structure.md) ora (con una nuova opzione [runInNewContext](./api/README.md#runinnewcontext) impostata a `false`). La tua applicazione esistente dovrebbe continuare a funzionare, ma è consigliato migrare in base alle nuove raccomandazioni.
:::

## Che cos'è l'esecuzione lato server, o Server-Side Rendering (SSR)?

Vue.js è un framework utilizzato per creare applicazioni lato client. Per impostazione predefinita, i componenti Vue producono e manipolano il DOM nel browser come risultato. Tuttavia, è altresì possibile convertire gli stessi componenti in stringhe HTML sul server, spedirle direttamente al browser, e infine "idratare" il markup statico all'interno di un'applicazione totalmente interattiva sul client.

Un'applicazione Vue.js eseguita sul server può anche essere considerata "isomorfica" o "universale", nel senso che la maggior parte del codice gira sia sul server **che** sul client.

## Perché SSR?

Confrontato con una tradizionale SPA (Applicazione Single-Page), il vantaggio del SSR risiede principalmente in:

- una SEO migliore, dato che i crawlers dei motori di ricerca potranno vedere una pagina interamente renderizzata.

    Tieni conto che adesso, Google e Bing possono indicizzare applicazioni JavaScript sincrone piuttosto bene. La sincronizzazione è la chiave qui. Se la tua applicazione parte con un indicatore di caricamento, che recupera il contenuto via Ajax, il crawler non aspetterà che finisca. Questo significa che se hai del contenuto recuperato in modo asincrono in pagine in cui la SEO è importante, allora il SSR potrebbe essere necessario.

- un tempo di caricamento del contenuto più rapido, in particolar modo con una rete Internet lenta o su dispositivi lenti. Il markup già eseguito sul server non ha bisogno di aspettare che tutto il codice JavaScript sia scaricato ed eseguito per essere mostrato, pertanto il tuo utente vedrà molto prima una pagina totalmente renderizzata.

Ci sono anche alcuni compromessi da considerare quando si usa il SSR:

- Vincoli di sviluppo. Il codice specificatamente relativo al browser può essere usato solo all'interno di determinati {em0}lifecycle hooks{/em0}; alcune librerie esterne potrebbero aver bisogno di trattamenti speciali affinché possano essere in grado di girare in un'applicazione eseguita sul server.

- Più requisiti coinvolti nell'installazione e nel distribuzione. Diversamente da un'applicazione statica SPA che può essere distribuita su qualsiasi server, un'applicazione eseguita sul server ha bisogno di un ambiente in cui un server Node.js possa girare.

- Più carico lato server. È chiaro che eseguire un'intera applicazione in Node.js sarà più dispendioso in termini di CPU rispetto alla semplice pubblicazione di file statici, pertanto se ti aspetti un traffico elevato, devi essere preparato al carico del server che ne deriverà e ad utilizzare strategie di caching in modo saggio.

Prima di utilizzare il SSR nella tua applicazione, la prima domanda da porti è se davvero ne hai bisogno. Principalmente dipende da quanto è importante il tempo di caricamento del contenuto per la tua applicazione. Per esempio, se stai creando una dashboard interna dove un centinaio di millisecondi in più sul caricamento iniziale non contano così tanto, allora il SSR potrebbe essere un'esagerazione. Ma, in casi in cui  il tempo di caricamento del contenuto è assolutamente critico, allora il SSR può aiutarti ad ottenere la migliore prestazione di caricamento iniziale possibile.

## Il SSR contro la pre-esecuzione (o Prerendering)

Se stai approfondendo il SSR soltanto per migliorare la SEO di una manciati di pagine marketing (quali `/`, `/chi-siamo`, `/contatti`, ecc...), allora probabilmente è preferibile usare il **prerendering** al posto. Piuttosto che utilizzare un server web per compilare HTML al volo, la pre-esecuzione genera semplicemente i file HTML statici per rotte specifiche al momento della creazione. Il vantaggio di impostare il prerendering rende tutto molto più semplice e ti consente di mantenere il tuo frontend come sito interamente statico.

Se usi webpack, puoi facilmente aggiungere il prerendering grazie al [prerender-spa-plugin](https://github.com/chrisvfritz/prerender-spa-plugin). È stato ampiamente testato con le applicazioni Vue apps - tant'è vero che, [il suo creatore](https://github.com/chrisvfritz), è un membro del Vue core team.

## A proposito di questa Guida

La guida si focalizza sulle applicazioni Single-Page eseguite su server che utilizzano Node.js. La combinazione di Vue SSR con altre impostazioni backend è un argomento a parte brevemente discusso in un'[apposita sezione](./guide/non-node.md).

Questa guida andrà molto in profondità e presuppone che tu abbia già familiarità con Vue.js stesso nonché una discreta conoscenza operativa di Node.js e di webpack. Se preferisci una soluzione di livello superiore che offra un'esperienza immediata, dovresti probabilmente provare [Nuxt.js](https://nuxtjs.org/). È costruito sullo stesso stack di Vue, ma ne estrapola il boilerplate in gran parte e offre alcune funzionalità extra come la generazione di siti statici. Tuttavia, potrebbe non essere adatto al tuo caso d'uso se hai bisogno di un controllo più diretto sulla struttura della tua applicazione. Indipendentemente da questo, è comunque utile leggere questa guida per capire meglio come funzionano insieme le cose.

Man mano che leggi, potrebbe essere utile fare riferimento alla [demo ufficiale di HackerNews](https://github.com/vuejs/vue-hackernews-2.0/), che utilizza la maggior parte delle tecniche descritte in questa guida.

Infine, tieni conto che le soluzioni fornite in questa guida non sono definitive - abbiamo trovato che funzionano bene per noi, ma questo non significa che non possano essere migliorate. Potrebbero essere riviste in futuro - e sentiti libero anche tu di contribuire sottoponendo pull requests!
