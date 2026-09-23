# Stato — Asta Fantacalcio

Due app da unire in un tool solo per l'asta su **Fantalab** (8 squadre, 500 crediti,
Classic + modificatore difesa, modificatore gol diverso da Rivoluzione Fantacalcio):
- `index.html` — "Asta Assistant", live durante l'asta (budget, prezzo suggerito, registro)
- `scout.html` — cheat sheet pre-asta (sfoglia/filtra/prezzi attesi)

Uso: solo dal telefono, un dispositivo alla volta (Fantalab è un'altra app, usata da PC).

## Fatto

- **23/09/2026 — Supabase sganciato**: il progetto Supabase ("Asta Fantacalcio",
  `swoibefjqkpgijbzrdtd`) aveva due tabelle (`asta_config`, `asta_movimenti`), entrambe
  vuote e con RLS disattivata (mai usate per un'asta vera). Tabelle eliminate. Rimossi
  client, chiavi e polling da entrambe le app: ora usano solo `localStorage`, condiviso
  tra le due pagine sullo stesso origin. Commit `ab32288`, pushato su GitHub.
  Il progetto Supabase è stato poi cancellato del tutto dall'utente a mano (23/09/2026, sera).

## Aperto — piano di unificazione (deciso per proposte il 23/09/2026)

1. ~~Un'unica app, due schede~~ — **fatto** (sopra, index.html+scout.html uniti).
2. ~~Dati live da Jarvis~~ — **fatto in parte** (sotto): `titolari.json` e `infortuni.json`
   collegati live. `statistiche.json` e `modello.json` restano fuori — sono formati
   interni di Jarvis (array numerici senza nomi di campo, pensati solo per i suoi script
   Python), deciso il 23/09/2026 di non provare a decifrarli da fuori.
3. ~~Supabase da decidere~~ — **fatto**, sganciato (sopra).
4. ~~Diversificazione manuale~~ — **fatto** (sotto): lista "giocatori già miei altrove"
   compilata a mano dalle Impostazioni, nessun collegamento con le rose di Rivoluzione
   Fantacalcio (repo diverso, cifrato).
5. ~~Level-up per uso durante un'asta vera~~ — **fatto**: ricerca sempre a fuoco, undo
   sempre a portata (già in index), avviso sfondamento budget di ruolo (colpo d'occhio a
   3 livelli), colpo d'occhio "chi comprare ora" (striscia "Occasioni ora", sopra).
6. ~~Default della nuova asta: 8 partecipanti (nomi provvisori), 500 crediti~~ — **fatto**,
   7 avversari placeholder ("Avversario 1"..."Avversario 7", rinominabili/rimovibili) +
   budget 500 di default (8 squadre totali = te + 7 avversari, coerente con Fantalab).

## Decisioni

- Niente sincronizzazione multi-dispositivo: chiarito il 23/09/2026 che le due app si
  usano solo dal telefono, mai insieme a un altro dispositivo — da qui la scelta di
  sganciare Supabase invece di sistemarne le policy di sicurezza.

- **23/09/2026 — architettura dell'unificazione (punto 1 del piano), decisa prima del
  mockup**: oggi le due app hanno due dataset giocatore separati — Live (`index.html`)
  solo quotazione ufficiale, Prepara (`scout.html`) anche FVM, tag, titolarità/infortuni,
  trend — e Prepara è di sola consultazione (legge lo stato ma non scrive). Deciso:
  1. **Un solo dataset giocatore**, quello ricco di oggi di Prepara (FVM, tag, titolarità,
     infortuni, trend): anche il prezzo consigliato di Live lo userà (es. sconto se
     infortunato, premio se titolare fisso), non solo la quotazione ufficiale.
  2. **Un solo posto per agire**: toccare un giocatore in "Prepara" apre la stessa
     schermata di dettaglio/acquisto di "Live" (oggi Prepara è solo consultazione).
  3. **Impostazioni/Registro/Rivali** raggiungibili da entrambe le schede, non solo da
     Live come oggi.
  Prossimo passo: mockup coerente con queste tre decisioni.

- **23/09/2026 — punto 1 implementato (codice vero, non mockup)**: `index.html` e
  `scout.html` uniti in un solo `index.html` (`scout.html` ora è solo un redirect, per
  chi lo avesse salvato). Verificato in locale con un server statico:
  - **Dataset unico**: 532 giocatori, stesso set di id nelle due liste originali (nessun
    disallineamento), ora con tutti i campi (quotazione ufficiale, FVM, titolarità,
    infortuni, trend) in un solo oggetto.
  - **Prezzo consigliato aggiornato**: non più solo quotazione ufficiale. Peso di
    riferimento = media tra quotazione ufficiale e valore atteso da FVM
    (`fvm * budget/1000`); poi sconto se non titolare fisso (`cambio` ×0,85, `riserva`
    ×0,6) e forte sconto se infortunato (×0,4). **I tre moltiplicatori sono un punto di
    partenza ragionevole, non testato su un'asta vera** — da aggiustare quando si vede
    come si comporta.
  - **Scheda azione unica**: toccare un giocatore in Prepara o in Live apre sempre la
    stessa scheda (prezzo consigliato, quotazione, FVM, titolarità, banner infortunio,
    Preso a X / Prezzo diverso / Preso da altri, alternative simili). Provato: acquisto
    da Prepara aggiorna budget/slot/lista; ricerca in Live su un infortunato mostra il
    banner e il prezzo già scontato.
  - **Registro/Rivali/Impostazioni** in header, raggiungibili da entrambe le schede
    (erano già fuori dal contenuto specifico di tab, ora la barra tab sta sotto).
  Non ancora toccato: la grafica (resta quella scura verde/oro originale, il mockup
  liquid glass provato prima resta da valutare a parte); i punti 2, 4, 5, 6 del piano.

- **23/09/2026 — livello serio, prime 4 priorità funzionali** (l'utente ha guardato
  l'app unita e l'ha trovata scomoda, non solo esteticamente: prima di continuare col
  piano abbiamo fermato e riscritto l'interazione). Fatto e verificato dal vivo in
  locale:
  1. **Ricerca globale unica**, sopra le due schede: cerchi un nome da Prepara o da Live
     senza cambiare scheda, i risultati sostituiscono il contenuto e tornano quando
     cancelli. Cambiare scheda pulisce la ricerca.
  2. **Scheda giocatore come overlay** (foglio dal basso, sfondo scurito): non sposta più
     lo scroll della pagina — si apre sopra, si chiude e resti esattamente dove eri.
  3. **Acquisto in un tocco**: ogni riga/card ha un bottone "✓ prezzo" col prezzo
     consigliato già calcolato — un tocco registra l'acquisto senza aprire la scheda.
     Tutta la card resta comunque cliccabile per aprire il dettaglio completo.
  4. **Impostazioni/Registro/Rivali come overlay** identici alla scheda giocatore
     (stesso meccanismo `openSheet`/`closeSheet`), non spingono più la pagina.
  Non ancora fatto: percentuali con slider, colpo d'occhio visivo (avvisi colorati oltre
  al rosso quando sfori), la grafica vera e propria — questi restano il prossimo giro.
  Commit `d4fbcbd`, pushato su GitHub.

- **23/09/2026 — percentuali con slider**: i 4 campi "% budget" per ruolo in Impostazioni
  (Portieri/Difensori/Centrocampisti/Attaccanti) sono ora slider (`type="range"`) invece
  di campi numerici, con il valore mostrato accanto in tempo reale mentre si trascina.
  Il ribilanciamento automatico delle altre tre percentuali (per restare a 100%) avviene
  già durante il trascinamento, non solo al rilascio — pensato per un tocco rapido durante
  un'asta vera. Provato in locale: trascinamento, ribilanciamento live, "Ripristina
  percentuali di partenza". Logica di calcolo invariata, solo l'input.

- **23/09/2026 — colpo d'occhio visivo**: le card ruolo nella barra in basso (e il totale
  crediti) hanno ora un terzo stato oltre a verde (completo) e rosso (sforato): **arancio
  ("tight")** quando il budget rimasto sul ruolo è positivo ma sotto il costo minimo per
  completarlo (somma delle quotazioni ufficiali più basse tra i giocatori ancora liberi di
  quel ruolo, per gli slot che mancano) — un avviso di rischio prima che sia già tardi.
  Nuova funzione `roleFloorCost(role, neededSlots)`. Provato in locale forzando lo stato
  (via console) nei tre casi: verde/arancio/rosso confermati sia sulle card ruolo che sul
  totale in basso a destra.
  Commit `e52e885`, pushato su GitHub.

- **23/09/2026 — default nuova asta (punto 6 del piano)**: `DEFAULT_STATE.participants`
  ora parte con 7 avversari placeholder ("Avversario 1"..."Avversario 7") invece di lista
  vuota — 8 squadre totali contando te, come su Fantalab. Budget 500 era già di default.
  Si applica solo a un'installazione nuova (localStorage vuoto) — "Azzera asta" non tocca
  i partecipanti, solo acquisti/registro. Nomi restano modificabili/rimovibili dal
  pannello Impostazioni come prima. Provato in locale con localStorage pulito.
  Commit `8ba825d`, pushato su GitHub.

- **23/09/2026 — colpo d'occhio "chi comprare ora" (chiude il punto 5 del piano)**:
  striscia "Occasioni ora" (scelta tra 3 opzioni proposte, quella più leggera) sopra le
  due schede, sempre visibile, non sostituisce le liste esistenti. Mostra i 3 migliori
  titolari/primi cambi liberi (non infortunati) nei ruoli con ancora slot da coprire,
  ordinati per rapporto FVM/quotazione (stessa metrica già usata per "Perle nascoste").
  Bottone acquisto in un tocco come nelle righe compatte di Live/ricerca (`compactRowHtml`/
  `wireCompactRows`, riusate). Si aggiorna da sola a ogni cambio di stato (agganciata a
  `renderBar()`: acquisto, undo, cambio percentuali/slot, reset). Provato in locale:
  striscia visibile su Prepara e Live all'apertura, acquisto rapido da lì aggiorna budget
  e fa sparire/ricalcolare le occasioni.
  Commit `afcc84b`, pushato su GitHub.

- **23/09/2026 — dati live da Jarvis, titolarità e infortuni (punto 2 del piano, in
  parte)**: valutato prima di scrivere codice. `dati/titolari.json` e `dati/infortuni.json`
  del repo JARVIS sono pubblici (verificato `curl` → 200) e in un formato semplice —
  id giocatore (stesso id Fantacalcio.it usato anche qui) → percentuale titolarità o
  motivo/data rientro infortunio. `dati/modello.json` e `dati/statistiche.json` invece
  sono array numerici senza nomi di campo, formato interno degli script Python di Jarvis
  (`modello.py`) — **deciso di non toccarli**, troppo rischioso indovinare cosa
  rappresenta ogni posizione per un dato usato in un'asta vera. All'apertura dell'app,
  fetch di `titolari.json`+`infortuni.json` via URL grezzo GitHub (`raw.githubusercontent
  .com/ManzoChinaglia/JARVIS/main/dati/`), sovrascrivono titolarità/percentuale/infortunio
  di ogni giocatore sopra lo snapshot statico imbottito nell'HTML. Se il fetch fallisce
  (offline, rate limit) resta lo snapshot statico, senza rompere nulla — indicatore
  visibile sotto "Rosa" nel riquadro in alto ("live da Jarvis" in verde / "dati statici"
  in grigio). Nuove funzioni `loadLiveJarvisData()`, `applyLiveJarvisData()`. Provato in
  locale: dati live confermati (es. Svilar passa da 85% statico a 91% live, coerente con
  `titolari.json`), banner infortunio con motivo/data reali su un giocatore infortunato,
  fallback a dati statici forzando `jarvisStatus.ok=false`.
  Commit `9f3224b`, pushato su GitHub.

- **23/09/2026 — diversificazione manuale (chiude il punto 4 del piano, e con questo
  tutti i 6 punti del piano di unificazione)**: nuova sezione "Già miei altrove" in
  Impostazioni, sotto Partecipanti avversari. Campo con autocompletamento (datalist su
  tutti i 532 giocatori, "Nome (Squadra)") per aggiungere un giocatore già posseduto in
  un'altra lega (es. Rivoluzione Fantacalcio) — solo un promemoria scritto a mano,
  **nessun collegamento automatico** con le rose vere (che restano in un repo diverso e
  cifrato, come deciso). I giocatori segnati mostrano un tag "Già tuo altrove" sulla card
  in Prepara e un banner nella scheda di dettaglio; nuovo checkbox "Nascondi i giocatori
  già miei altrove" in Prepara per filtrarli via; esclusi anche dalla striscia "Occasioni
  ora" (non ha senso suggerire di ricomprare chi hai già). Stato in `state.elsewhere`
  (array di `{id, nome, squadra}`), persistito come tutto il resto. Nuove funzioni
  `isElsewhere()`, `populateElsewhereOptions()`, `renderElsewhereList()`. Provato in
  locale: aggiunta Svilar dalle Impostazioni, tag visibile sulla card, filtro nascondi
  funzionante (63→62 giocatori).
  Commit `cbd9960`, pushato su GitHub.

- **23/09/2026 — rework UX "level up" (fase 1, NON ancora committato)**: l'app sembrava
  "da programmatore" (una ventina di controlli visibili prima di vedere un giocatore).
  Decisi con l'utente: uso chiave = **Live** (nome chiamato → prezzo → preso), navigazione
  = **tab bar di vetro in basso** (Live · Prepara · Registro · Rivali · Opzioni). Rifatti
  HTML/CSS e il JS di rendering; logica di prezzi/budget/Jarvis/storage intatta.
  - Stile liquid glass come Jarvis (font Barlow Condensed copiati in `font/`, sfondo a
    macchie di colore, blur, molle).
  - Testata sempre visibile: crediti liberi grandi + 4 chip ruolo (verde/arancio/rosso).
  - Si apre su **Live**: ricerca in alto, "Occasioni ora", poi sfoglia per ruolo.
  - Scheda giocatore: prezzo gigante + un solo pulsante dominante "Preso a X"; dati sotto.
  - Prepara: filtri/ordinamento/toggle spostati in un foglio "Filtri" (badge col numero
    di filtri attivi); tag inline (infortunato, già tuo, preso) invece di riquadri.
  - Dopo un acquisto la ricerca si svuota da sola; secondo tocco su "Live" = cursore nella
    ricerca; trascinare un foglio verso il basso lo chiude; Registro/Rivali/Opzioni sono
    fogli dalla tab bar.
  - Bug corretto: data rientro "non specificata" mostrava "NaN undefined".
  Da fare: provarla sul telefono vero, valutare "Occasioni ora" (oggi premia giocatori da
  1 cr con FVM alto, poco utili), rifinire Opzioni/Registro/Rivali.

- **23/09/2026 — rework UX fase 2 (feedback dopo la prova sul telefono)**: app rinominata
  **FantaQintesi** (la lega nuova su Fantalab). Tolto il banner dei crediti (ridondante con
  Fantalab); la testata ora mostra per ruolo **speso / piano** con barra e sforo in rosso
  (+N), più una riga "Sforo sul piano: +X cr" / "In piano". Registro tolto dalla tab bar
  (si apre da Opzioni). Righe di liste più compatte. "Occasioni ora": ora il migliore
  titolare libero per ruolo ancora da coprire, esclusi i riempitivi (q<3 o FVM atteso<8).
  **Rosa dell'altra lega (Rivoluzione, squadra BURKINA FASO, 25 giocatori)**: NON nel codice
  (repo pubblico, in Jarvis le rose sono cifrate di proposito) ma con link personale
  `index.html#miei=<id,id,...>` che la importa una volta sola in `state.elsewhere` e ripulisce
  l'indirizzo. Fonte: `JARVIS/archivio/rose/rivoluzione-...-1789387078367.xlsx`.
  Analisi asta precedente (Rivoluzione: 10 squadre, 500 cr, 250 acquisti su 532): quota per
  ruolo media P 9% · D 21% · C 31% · A 40% (A varia 24–59%); prezzi molto sbilanciati
  (A: 201, 200, 139, 118…; 4–9 giocatori a ≤2 cr per squadra); prezzo~FVM corr 0.92; stelle
  pagate 2–5× la quotazione, riempitivi ≤ quotazione. Da qui la proposta di ricalibrare il
  prezzo consigliato (curva empirica per rango + inflazione live) — in attesa di scelta.

- **23/09/2026 — fase 3: modello di prezzo empirico + correttivo live + piano a fasce**
  (build `2026-09-23-f`). Analisi sui 250 acquisti veri dell'asta precedente (Rivoluzione,
  10 squadre x 500 cr) incrociati col listone. Cose emerse, da non rifare da zero:
  - Il vecchio FVM/2 era già un buon predittore (errore medio ~8 cr); un modello log-lineare
    con FVM+quotazione+ruolo NON lo batteva (le aste sono rumorose, residuo std 0.8 in log).
    Il "prezzo irrealistico" veniva soprattutto dal calcolo a budget di `computeSuggestion`
    (quota del budget di ruolo), non dal FVM. Fascia bassa/media: o 1 cr o rilancio (bimodale).
  - **(1) Prezzo atteso**: `baseModel(p)` = curva monotona lisciata FVM(crediti)->prezzo
    (`PRICE_KNOTS`, da regressione isotonica sui dati veri); `marketPrice(p)` = atteso +
    fascia realistica larga (0–1.8x sotto i 12 cr, ±30-40% sopra). Sostituisce il prezzo a
    budget in scheda, righe, Prepara, alternative. Sconto 0.9 per cambio/riserva (i dati non
    mostravano sconto per titolarità), 0.6 per infortunato (nessun dato, ipotesi).
  - **(2) Correttivo live**: `liveFactor()` = crediti pagati / crediti previsti sui prezzi
    VERI registrati (peso a priori 60 cr, limitato a 0.6–1.6); sale/scende da solo e corregge
    tutte le stime. Il tocco rapido "✓ prezzo" sulle righe è una stima (`est:true`) e NON
    entra nel calcolo; la scheda ha ora −5/−1/+1/+5 e il suo "Preso a" conta come prezzo vero,
    così come "Preso da altri". I prezzi degli altri vanno registrati per far imparare l'app.
  - **(4) Piano a fasce**: Top >=40 cr, Medi 7–39, Riempitivi <=6, struttura media per
    squadra dall'asta precedente (P 1/1/1, D 0/4/4, C 1/4/3, A 2/2/2 sugli slot di default),
    prezzi di fascia scalati sul budget e sull'inflazione. Foglio "Piano a fasce" (tocco sulla
    riga sotto la testata) con margine ±cr e cosa ti resta da prendere; nota nella scheda
    giocatore se la fascia è già coperta nel piano.
  - Filtri categoria (Tutti/Top/Scomm./Perle/Low) ora su una riga, senza scroll laterale.
  Limiti onesti: una sola asta come campione; la nuova ha 8 squadre invece di 10 (i prezzi
  medi dovrebbero scendere: ci pensa il correttivo live); il piano a fasce e le % per ruolo
  convivono (due piani). Ancora da fare: (3) pressione sulle stelle, (5) avviso tattico esteso.

- **23/09/2026 — fase 4** (build `2026-09-23-h`): tolte le quotazioni ufficiali da tutta la
  vista (righe, scheda -> ora "Fascia di prezzo", descrizioni filtri). **(3) Pressione sulle
  stelle**: per ruolo, stelle libere (prezzo atteso >=40) contro acquirenti attesi (media
  storica x avversari, meno le stelle già prese) x liquidità relativa degli avversari; nel
  foglio Piano e come nota nella scheda di una stella. Si aggiorna solo con "Preso da altri".
  Bias trovato e corretto nel modello: nell'asta precedente portieri ~1.5x e difensori ~1.3x
  il previsto (centrocampisti ~0.9x) -> `ROLE_ADJ` P1.3 D1.2 C0.9 A1.0 (ridotti verso 1).
  Import rosa: ora funziona anche col link aperto ad app già aperta (`hashchange`).
  NOTA: il localStorage di Safari e quello dell'app aggiunta alla Home sono separati su iOS:
  il link #miei= va aperto nel contenitore che si usa davvero.

- **23/09/2026 — fase 5: % per ruolo e piano a fasce unificati** (build `2026-09-23-i`): le %
  in Opzioni sono ora l'unica fonte. `planScale(r)` scala i prezzi delle fasce di ogni ruolo sul
  budget di quel ruolo (budget x %), quindi il costo del piano di un ruolo = il suo budget a
  inflazione 1 (margine iniziale 0) e spostando uno slider i prezzi delle fasce si adeguano.
  Default % portati alle medie reali dell'asta precedente: P9 D21 C30 A40 (prima P8 D20 C27 A45).
  Chi ha già uno stato salvato tiene le vecchie %: "Ripristina percentuali di partenza" applica
  le nuove. Provato in locale (default e P10 D15 C30 A45).
  Aperto: prova sul telefono; moltiplicatori non testati.

- **23/09/2026 — fase 6: avvisi tattici leggeri** (build `2026-09-23-j`, punto (5) chiuso):
  1. **Semaforo sul prezzo** nella scheda giocatore (`updatePriceFlag`, una riga sola, nessuna
     riga se il prezzo è nella norma): verde "Affare" se sotto ~80% dell'atteso; arancio se
     sopra la fascia realistica; rosso se sopra l'atteso E il margine sul piano, simulato
     con quell'acquisto, va sotto zero. Si aggiorna con −5/−1/+1/+5. Al prezzo atteso non
     avvisa mai (le stelle d'attacco partono già sopra il piano al 40%: sarebbero tutte rosse).
  2. **Testata**: "margine fasce" a 3 livelli, verde >=0, arancio fino a −3% del budget (min 10 cr),
     rosso oltre. Nota: il margine parte da 0 per costruzione, non ha senso una soglia "basso".
  Provato in locale su un giocatore per ruolo (P, D, C, A) e forzando i tre stati della testata.

- **23/09/2026 — tab Rivali tolta** (build `2026-09-23-k`): l'utente segue gli avversari su Fantalab;
  qui basta segnare chi ha preso ogni giocatore (selettore "Preso da" invariato), che alimenta
  le misure in background (liquidità avversari, pressione sulle stelle). Tab bar ora Live · Prepara ·
  Opzioni. Il foglio Rivali resta nel codice ma non è più raggiungibile.
  Test prima uscita: l'utente ha provato l'app sul telefono, occasioni e semaforo ok; la rosa
  dell'altra lega non risultava caricata perché il link `#miei=` non era ancora stato aperto.

- **23/09/2026 — pulizia Opzioni e Registro** (build `2026-09-23-l`, commit `3327de5`, non pushato):
  - Tolto il codice morto dei Rivali (foglio, `renderRivals`, `rivalStats`, CSS, chiamate).
  - Opzioni riorganizzate in schede: Budget e rosa · Avversari · Già miei altrove; "Registro
    acquisti" in cima come voce di navigazione col conteggio; "Azzera asta" isolato in fondo e
    ora **chiede conferma** (prima azzerava al primo tocco). Stili inline spostati in classi.
  - Registro: tab con conteggi, badge ruolo, "~" sui prezzi stimati dal tocco rapido (con nota),
    nomi escapati (`escHtml`, anche nei chip di avversari e "già miei altrove").
  Provato in locale (viewport telefono): registro con acquisti veri/stimati/altrui, conferma su
  Azzera, nessun errore in console.

- **23/09/2026 — simulazione d'asta e prova di stress** (build `2026-09-23-m`): asta simulata in
  locale (8 squadre, 500 cr, 200 vendite, io compro seguendo il prezzo consigliato) con prezzi
  "umani" da una curva nascosta diversa dal modello + rumore lognormale, in 6 scenari (storico,
  ruoli neutri, +50%, −40%, stelle pazze, inflazione crescente), 3 semi ciascuno. Il localStorage
  del telefono non è stato toccato. Risultati (errore medio prezzo consigliato vs pagato, cr):
  - MAE 7-18 a seconda dello scenario; sulle stelle (previste ≥40 cr) 33-59: il prezzo è
    un'indicazione, la fascia lo/hi copre solo il 50-60% dei casi.
  - Il correttivo live aiuta davvero solo quando l'asta è più economica del previsto
    (−40%: MAE 7,0 contro 9,7 senza); negli altri scenari è neutro (±0,3). Non peggiora mai.
  - **Difetto**: i prezzi degli ultimi 50 acquisti (attaccanti, chiamati per ultimi) sono
    sovrastimati, MAE 21-33 contro 7-12 dei primi 50: nella simulazione i crediti finiscono e le
    ultime chiamate vanno a meno del previsto. Il margine del piano è pessimista a metà asta
    (fino a −250 cr con prezzi alti) e poi rientra.
  - Idea provata offline, NON implementata: correttivo di "conservazione dei crediti" (crediti
    rimasti di tutti / somma dei prezzi attesi dei giocatori ancora da comprare). Da solo o in
    media geometrica con quello live riduce il MAE di 1-2,5 cr e degli ultimi 50 fino a metà
    (gonfia 33,5→19,6), ma peggiora quando l'asta spende poco (−40%: 8,8 contro 7,0) e richiede
    di registrare TUTTE le vendite degli altri. Decisione aperta.
  - Robustezza: nessun NaN né errore con prezzi 1/999, budget 0, slot sotto il comprato, zero
    avversari, avversario cancellato dopo acquisti, ruolo esaurito, ricerche strane.
  - Corretti due buchi: lo stesso giocatore si poteva registrare due volte (duplicato nel
    registro, falsava il correttivo live) → ora `commitEntry` lo rifiuta; «Annulla» tornava
    indietro di una sola azione → ora a più livelli, sul registro.
  Limiti della simulazione: prezzi generati, non veri; chi compra "vince" se il suo limite supera
  il prezzo (nessuna maledizione del vincitore); gli avversari non si adattano ai prezzi.

- **23/09/2026 — correttivo "crediti in gioco" implementato** (build `2026-09-23-n`; l'utente ha
  scelto di registrare TUTTE le vendite, quindi opzione A). `cashFactor()` = crediti rimasti di
  tutte le squadre / somma dei prezzi attesi dei migliori N liberi per ruolo (N = slot da
  coprire): guarda avanti, dove il live guarda solo i prezzi già pagati. `priceFactor()` =
  media geometrica di live e crediti, con l'eccesso di crediti ridotto a 1/4 (crediti che
  avanzano non garantiscono che vengano spesi; crediti scarsi = prezzi in calo, affidabile).
  Usato da `marketPrice` e `tierPrice`; si attiva dopo 10 vendite registrate (te + "Preso da
  altri", "Non tracciato" compreso); cache per non rallentare le liste. In Live, sotto il riepilogo
  asta, riga "Crediti in gioco: N · da qui i prezzi vanno ±X%".
  Risultati sulla stessa simulazione (errore medio in cr, solo-live → nuovo; fine asta = ultimi 50):
  storico 12,1→11,0 (fine 21→17,6) · neutri 10,9→10,3 · +50% 17,7→14,8 (fine 32,5→20) ·
  −40% 7,3→8,2 (fine 12,4→15) · stelle 17,7→15,5 · crescente 12,6→11,8. Margine del piano a
  metà asta: peggiore osservato −58 cr (prima −254). Peggiora solo se l'asta spende molto meno del
  previsto. Se si salta la registrazione di molte vendite le stime diventano ottimistiche/errate:
  meglio segnare "Preso da altri" (anche "Non tracciato") per ogni vendita.

- **23/09/2026 — sconto infortunati proporzionale** (build `2026-09-23-o`): il moltiplicatore fisso
  0,6 valeva anche per chi rientra tra 12 giorni (Calhanoglu: 57 cr invece di ~89). Ora
  `injFactor(p)` = 1 − 1,5 × (giorni di assenza / 270), minimo 0,4; 0,6 se la data di rientro
  non è specificata. Su 88 infortunati: 35 tra 0,9 e 1, 14 tra 0,75 e 0,9, 33 tra 0,6 e 0,75.
  Usato da `marketPrice` e `baseAdj` (correttivo crediti). **Regola di buon senso, non tarata**:
  non esistono dati d'asta su infortunati. Da rivedere se in una vera asta si vede che gli
  infortunati vanno a prezzi molto diversi.

## Chiusura del 23/09/2026
- Prova sul telefono/Home: fatta dall'utente ("l'app gira bene"), chiusa.
- Supabase: nessuna dipendenza nel codice; progetto `swoibefjqkpgijbzrdtd` cancellato dall'utente
  a mano dalla dashboard. Chiuso.
- Moltiplicatore infortunati: reso proporzionale (sopra); resta un'ipotesi da verificare in asta.
- Da ricordare in asta: registrare OGNI vendita ("Non tracciato" se non si sa chi), altrimenti
  il correttivo sui crediti sovrastima i prezzi.
- **23/09/2026, sera — punti d'uso chiusi dall'utente**: registrazione di ogni vendita in asta e
  controllo dei prezzi degli infortunati considerati chiusi. Nessun punto aperto per questo progetto.
