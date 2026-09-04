# PoE 2 Tools

Sito statico con la UI degli strumenti per Path of Exile 2 (patch 0.5.5, lega
*Forbidden Rites*). Servito da GitHub Pages, nessuna build: HTML e CSS scritti
a mano.

## Struttura

```
poe2-tools/
├── index.html    catalogo degli strumenti
└── <tool>/       una cartella per strumento, con il suo index.html
```

Ogni strumento è autoconcluso: niente dipendenze esterne, niente CDN, tutto
inline. Una pagina che smette di funzionare offline è una pagina rotta.

## Sviluppo

Apri `index.html` nel browser. Non serve altro.

`node controlla-barre.mjs` verifica che la barra di navigazione sia identica in
tutte le pagine; una GitHub Action lo rifà a ogni push.
