# VireoXlibrary

VireoXlibrary è una libreria di componenti Vue3 per la realizzazione di applicazioni web.
Utilizza Tailwind CSS.

## Requisiti

- Tailwind CSS

Per installare Tailwind CSS fare riferimento alla seguente guida:
https://tailwindcss.com/docs/guides/vite

## Installazione

Per scaricare e installare la libreria eseguire il comando:

```
npm i @sirius-rd/vireoxlibrary
```

oppure

```
yarn add @sirius-rd/vireoxlibrary
```

Installato e configurato Tailwind CSS, andare nel file `tailwind.config.js` e modificarlo come segue:

```js
module.exports = {
    ...,
    content: [
        ...
        './node_modules/@sirius-rd/vireoxlibrary/src/**/*.{ts,js,vue}'
    ]
}
```

## Utilizzo

Tutti i componenti iniziano con il prefisso "Vrx" per evitare conflitti con altri componenti.

# Sito Documentazione

https://sirius-ea.github.io/VireoXlibrary/
