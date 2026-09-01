# Istruzioni per Claude — PoE 2 Tools (sito)

Questa cartella è un **repo git separato e pubblico**, annidato dentro una wiki
locale che non va pubblicata.

## Regola prima di tutte

**Le operazioni git si eseguono solo con la working directory dentro
`poe2-tools/`.** Mai `git add` da un livello superiore: la cartella genitore
contiene appunti privati e file di credenziali. La radice del repo è qui, e deve
restarci.

Prima di ogni commit, verifica cosa stai includendo:

```bash
git -C . status --short
```

Se compare qualcosa che sta fuori da `poe2-tools/`, fermati.

## 🔴 L'identità git di questo repo non è quella globale della macchina

**Ogni commit qui è firmato `NicolasGor`**, con l'email di noreply di GitHub.
Non è una preferenza estetica: il `user.email` globale della macchina appartiene
a un altro account GitHub, e usarlo qui **lega pubblicamente questo repo a
quell'account** — chi apre la pagina dei commit vede l'altro nome, l'altro
avatar, e l'email vera in chiaro dentro il commit. I due mondi non devono avere
nessuna correlazione visibile.

La configurazione è **locale al repo** (`.git/config`, non `--global`): non tocca
nient'altro sulla macchina, ma **non si eredita**. Un repo nuovo nasce con
l'identità globale sbagliata, quindi la si imposta **prima del primo commit**:

```bash
git config user.name  "NicolasGor"
git config user.email "82063374+NicolasGor@users.noreply.github.com"
```

⚠️ **Se il commit è già stato fatto, `git config` da solo non basta**: la firma è
dentro l'oggetto commit e va riscritta — la firma sta nell'oggetto, non nella
configurazione.

## Cosa ci va

Solo il sito: HTML, CSS, JS della UI degli strumenti. **Nessun contenuto della
wiki** — gli appunti di gioco, i personaggi, gli obiettivi restano locali. Se un
dato della wiki serve a uno strumento, si copia il singolo valore, non la pagina.

## 🔴 Questo è Path of Exile **2**, e i due giochi si somigliano abbastanza da far sbagliare

Nomi identici, numeri diversi; meccaniche omonime, regole diverse. Un fatto
dell'altro gioco **non attraversa**: si rimisura, oppure non entra nel sito.
Le divergenze già misurate, che sono anche il motivo per cui la regola esiste:

| | l'altro gioco | **qui** |
|---|---|---|
| Unità di conto dei prezzi | Chaos Orb | 🔴 **Divine Orb** — `primaryValue` di poe.ninja è in divine |
| Ruolo dell'Exalted | valuta pregiata | 🔴 **spicciolo**: 1 divine = 414 exalted |
| Endpoint di trade | `/api/trade/` | **`/api/trade2/`** |
| Endpoint di poe.ninja | `/poe1/api/economy/…` | **`/poe2/api/economy/…`** |
| Scarab, frammenti, carte divinatorie | esistono | **non esistono** (ci sono Rune, Soul Core, Omen, Tablet) |
| Atlante | albero di passive | mappa continua con torri e tablet |

⚠️ **Cercando fuori, metà dei risultati è dell'altro gioco.** Guide, video e post
su «Path of Exile» senza il 2 sono dell'altro gioco nel dubbio: si controlla
*quale gioco* prima di estrarre il fatto, non dopo.

## I link di trade si costruiscono così

**Endpoint:** `POST https://www.pathofexile.com/api/trade2/search/<lega>` — le
leghe valide vengono da `GET /api/trade2/data/leagues`.

Ogni ricerca filtra per Instant Buyout con `query.status.option = "securable"`:
il campo **status**, non `sale_type`. I valori accettati sono `any`,
**`securable`**, `online`, `onlineleague`; `instant` risponde **400 Unknown
status type**.

- **Un link si collauda prima di metterlo in pagina.** Se il POST non torna
  `HTTP 200` con un `total` sensato, il link non si pubblica.
- ⚠️ **Il rate limit è per IP**: molte ricerche ravvicinate bloccano anche il
  browser di chi guarda. Una ricerca larga e molte fetch, non molte ricerche.
- ⚠️ **Cento risultati sono un troncamento**, non un mercato.

## Convenzioni

- **Zero dipendenze esterne a runtime**: niente CDN, niente font remoti, niente
  fetch verso altri host all'apertura. La pagina deve funzionare offline.
- **I link in uscita sono ammessi** — un `<a href>` verso
  `pathofexile.com/trade2` è navigazione, non una dipendenza: la pagina si
  carica lo stesso se il sito remoto è giù.
- **Il design sta in [stile.css](stile.css), condiviso da tutte le pagine**:
  token (colori, scala tipografica, spaziatura), barra di navigazione, e i
  componenti ricorrenti — `.card`, `.badge`, `.avviso`, `.tabella`, `.kpi`,
  `.sec`. È un file locale, non un CDN. Nel `<style>` di una pagina ci va **solo
  ciò che è davvero suo**. Se una regola servirebbe a due pagine, il suo posto è
  `stile.css`.
- **La larghezza della colonna è la variabile `--colonna`**, condivisa fra barra
  e contenuto perché restino allineati. Una pagina che ha bisogno di più spazio
  mette `class="largo"` su `<body>` — non un max-width solo sul contenitore.
- **Le icone si scaricano, non si linkano.** Gli asset stanno in `<tool>/img/` e
  sono committati.
- Un strumento = una cartella con il suo `index.html`, più una scheda nel
  catalogo in `index.html` alla radice.
- 🔴 **E la voce va aggiunta alla barra di *tutte* le pagine**, non solo a quella
  su cui stai lavorando: la barra è copiata dentro ogni `index.html`.
  `node controlla-barre.mjs` lo verifica, e una GitHub Action lo rifà a ogni push
  — perché non è un errore che si nota guardando una pagina: bisogna
  confrontarle fra loro.
- **Una voce di barra può essere un gruppo invece di una pagina**
  (`<button class="nav-gruppo">` più un pannello `.nav-giu`). Quattro cose da
  sapere prima di aggiungerne uno:
  - il pannello sta **fuori da `.nav-link`**, che ha `overflow-x:auto` e dentro
    lo ritaglierebbe su schermo stretto;
  - il suo `left` lo scrive `menu.js` all'apertura, perché il pulsante si sposta
    quando la barra scorre — nel CSS sarebbe una costante sbagliata;
  - nel markup il pannello nasce **aperto** (`aria-expanded="true"`, niente
    `hidden`) e lo chiude lo script: senza JS resterebbe altrimenti muto, e le
    sue voci irraggiungibili;
  - 🔴 aprire un gruppo **chiude gli altri**, e va fatto a mano in `menu.js`: il
    click sul pulsante ferma la propagazione, quindi il gestore che chiude «sul
    click fuori» non scatta e i pannelli resterebbero aperti sovrapposti.
- Il sito dichiara la patch e la lega a cui si riferisce. Quando cambiano, si
  aggiornano.
- Niente numeri di gioco inventati: valgono le stesse regole della wiki. Ogni
  numero in pagina viene da una misura datata, e ogni pagina dichiara cosa il
  numero **non** dice.
