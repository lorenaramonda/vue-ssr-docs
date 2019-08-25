# Iniziamo

## Installazione

```bash
npm install vue vue-server-renderer --save
```

Useremo NPM durante tutta la guida, ma sentiti libero di usare [Yarn](https://yarnpkg.com/en/) se lo preferisci.

#### Note

- Si raccomanda di usare Node.js versione 6+.
- Le versioni di `vue-server-renderer` e `vue` devono corrispondere.
- `vue-server-renderer` si appoggia su alcuni moduli Node.js nativi pertanto può essere usato solo con Node.js. È possibile che forniremo una distribuzione più semplice in futuro che possa girare su altri runtime JavaScript.

## Eseguire un'istanza Vue

```js
// Passo 1: Creare un'istanza Vue
const Vue = require('vue')
const app = new Vue({
  template: `<div>Hello World</div>`
})

// Passo 2: Creare un renderer
const renderer = require('vue-server-renderer').createRenderer()

// Passo 3: Convertire l'istanza Vue in HTML
renderer.renderToString(app, (err, html) => {
  if (err) throw err
  console.log(html)
  // => <div data-server-rendered="true">Hello World</div>
})

// nella versione 2.5.0+, ritorna una Promise se nessuna callback viene passata:
renderer.renderToString(app).then(html => {
  console.log(html)
}).catch(err => {
  console.error(err)
})
```

## Integrazione col Server

È piuttosto semplice se utilizzato all'interno di un server Node.js, ad esempio [Express](https://expressjs.com/):

```bash
npm install express --save
```

---

```js
const Vue = require('vue')
const server = require('express')()
const renderer = require('vue-server-renderer').createRenderer()

server.get('*', (req, res) => {
  const app = new Vue({
    data: {
      url: req.url
    },
    template: `<div>The visited URL is: {{ url }}</div>`
  })

  renderer.renderToString(app, (err, html) => {
    if (err) {
      res.status(500).end('Internal Server Error')
      return
    }
    res.end(`
      <!DOCTYPE html>
      <html lang="en">
        <head><title>Hello</title></head>
        <body>${html}</body>
      </html>
    `)
  })
})

server.listen(8080)
```

## Utilizzare un Template di Pagina

Quando viene eseguita un'applicazione Vue, il renderer genera soltanto il markup dell'applicazione. Nell'esempio abbiamo dovuto racchiudere il risultato all'interno di una pagina HTML aggiuntiva.

Per semplificare, puoi fornire direttamente un template di pagina durante la creazione del renderer. La maggior parte delle volte inseriremo il template di pagina all'interno del suo proprio file, ad esempio `index.template.html`:

```html
<!DOCTYPE html>
<html lang="en">
  <head><title>Hello</title></head>
  <body>
    <!--vue-ssr-outlet-->
  </body>
</html>
```

Nota il commento `<!--vue-ssr-outlet-->` -- qui è dove il codice della tua applicazione verrà iniettato.

Possiamo dunque leggere il file e passarlo al renderer Vue:

```js
const renderer = createRenderer({
  template: require('fs').readFileSync('./index.template.html', 'utf-8')
})

renderer.renderToString(app, (err, html) => {
  console.log(html) // è l'intera pagina con il contenuto iniettato
})
```

### Interpolazione del Template

Il template supporta anche della semplice interpolazione. Dato il seguente template:

```html
<html>
  <head>
    <!-- usa doppie parentesi graffe per interpolare HTML con escape dei caratteri speciali -->
    <title>{{ title }}</title>

    <!-- usa triple parentesi graffe per interpolare HTML senza escape dei caratteri speciali -->
    {{{ meta }}}
  </head>
  <body>
    <!--vue-ssr-outlet-->
  </body>
</html>
```

Possiamo fornire dati da interpolare passando un "render context object" come secondo argomento a `renderToString`:

```js
const context = {
  title: 'hello',
  meta: `
    <meta ...>
    <meta ...>
  `
}

renderer.renderToString(app, context, (err, html) => {
  // Il titolo della pagina sarà "Hello"
  // con meta tag iniettati
})
```

L'oggetto `context` può essere condiviso anche con l'istanza dell'applicazione Vue, permettendo ai componenti di registrare dinamicamente dati per l'interpolazione del template.

In aggiunta, il template supporta alcune funzionalità avanzate quali:

- Auto iniezione di CSS critici durante l'utilizzo di componenti `*.vue`;
- Auto iniezione di collegamenti agli assets e suggerimenti sulle risorse durante l'utilizzo di `clientManifest`;
- Auto iniezione e prevenzione di XSS quando si incorpora lo stato di Vuex per l'idratazione lato client. 

Ne discuteremo più avanti quando introdurremo i concetti annessi nella guida.
