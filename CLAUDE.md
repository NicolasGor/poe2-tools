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
- **Le icone si scaricano, non si linkano.** Gli asset stanno in `<tool>/img/`,
  committati, con il nome dell'item in kebab-case (`omen-of-light.png`). Un PNG
  da 8 KB non merita un'eccezione alla regola delle dipendenze: una pagina che
  offline mostra quindici quadrati rotti è una pagina rotta.
  - **Dove si prendono, in ordine.** La `image` di poe.ninja
    (`.../economy/exchange/current/overview?type=<tipo>` → `items[].image`), che
    è un percorso relativo da anteporre a `https://web.poecdn.com`. Per gli item
    che **non passano dal Currency Exchange** — le ossa della desecrazione, per
    dire — non c'è: lì il campo `icon` di un risultato di **trade2** ce l'ha, e
    una sola ricerca larga con `term` ne raccoglie diversi in un colpo.
  - ⚠️ **Il CDN tronca**: scaricando quindici icone di fila, sette sono arrivate
    a metà (`IncompleteRead`, `Remote end closed connection`). Si riprova, non è
    un URL sbagliato — e **si verifica che il file cominci per `\x89PNG`**, perché
    un troncamento lascia un file che esiste e non si apre.
  - L'`alt` è **vuoto**: il nome dell'item sta scritto accanto, e uno screen
    reader che lo legge due volte è peggio di uno che lo legge una volta.
  - 🔴 **Ma non tutto merita un'icona, e la linea è netta**: la si mette agli
    item con **un'identità fissa** — valute, omen, essenze, ossa, e al limite le
    uniche. **Mai a basi bianche, magiche o rare** (boots, cintura, amuleto,
    anello): lì l'icona mostrerebbe un contenitore, mentre ciò di cui si parla
    sono i **mod** che ci finiscono sopra. Detto da Nicolas il 1 settembre 2026,
    respingendo la proposta di metterle nella tabella dei tier.
- Un strumento = una cartella con il suo `index.html`, più una scheda nel
  catalogo in `index.html` alla radice.
- 🔴 **Su una pagina di strategie l'unità è la scheda, e tutto quello che riguarda
  una strategia sta dentro la sua scheda, in righe apribili.** Non si aggiunge una
  sezione al livello della pagina per le rune, i rumor, i prezzi o i limiti di una
  strategia: quelle sono parti di quella strategia, e fuori dalla scheda la pagina
  diventa uno scorrimento lungo in cui non si capisce più cosa appartiene a cosa.
  Detto da Nicolas l'11 settembre 2026 — *«non si capisce nulla»* — davanti a
  `/strategie/`, che aveva quattro sezioni di pagina per una sola strategia.
  - **A riposo la pagina deve essere un indice**: titolo, i numeri di sintesi in
    fascia, e le righe chiuse. Si apre solo quella che serve.
  - **Una riga che non è un passo della sequenza si marca diversamente** dalle
    fasi numerate: qui le fasi hanno `1 2 3` e gli allegati `§`.
  - 🔴 **E l'indice in cima deve aprire la riga a cui punta.** Un'ancora che
    atterra su un accordion chiuso sembra non aver fatto niente: è il difetto che
    rende inutile un indice sopra un accordion. Lo fa lo script della pagina, su
    `a[href^="#"]` e su `hashchange`, aprendo anche gli antenati.
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
