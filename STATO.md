# Stato — Asta Fantacalcio (FantaQintesi)

**Al 26/09/2026: lega passata a 10 squadre** (build `2026-09-26-a`, da provare sul telefono). Default 9 avversari; la
vecchia lista di 7 «Avversario N» mai toccata passa da sola a 9, una lista con nomi veri resta com'è (aggiungere i 2 in
Opzioni). Le % P7/D20/C28/A45 restano: il mercato a 10 le sposta di un punto al massimo (P8 D20 C32 A41 contro
P8 D19 C31 A42 a 8). La curva dei prezzi era tarata su 10×500: a 10 la spesa attesa (4.993) torna coi crediti (5.000).
Tattica: portiere buono entro metà asta (i titolari decenti sono ~10), due attaccanti solidi presto (la fascia
54-84 cr sarà contesa), riempitivi di fine asta più poveri. Il cronologico dettagliato del 23-24/09 è nella
storia di git (commit fino a `d18e94d`).

Tool unico per l'asta su **Fantalab** (10 squadre, 500 crediti, Classic + modificatore difesa,
modificatore gol diverso da Rivoluzione Fantacalcio). Tutto in `index.html`: schede **Live**
(budget, prezzo atteso, registro) e **Prepara** (sfoglia/filtra), più **Opzioni**.
`scout.html` è solo un redirect. Uso: solo dal telefono, un dispositivo alla volta.

## Com'è fatto
- **Dati**: 532 giocatori in un solo dataset; titolarità e infortuni live da Jarvis
  (`titolari.json`, `infortuni.json` dal repo JARVIS), con fallback allo snapshot statico.
  `modello.json` e `statistiche.json` di Jarvis restano fuori (formato interno).
- **Prezzo atteso**: curva empirica FVM→prezzo (`baseModel`/`marketPrice`) tarata su 250 acquisti
  veri dell'asta Rivoluzione, con correttivi `ROLE_ADJ`, `liveFactor` (prezzi pagati) e
  `cashFactor` (crediti in gioco), uniti in `priceFactor`. Sconto infortunati proporzionale ai
  giorni di assenza (`injFactor`, regola di buon senso non tarata).
- **Piano**: solo la ripartizione % per ruolo, di partenza **P7 / D20 / C28 / A45**, adattiva
  (`effPct`: un ruolo completo si congela a quanto speso). Nessun piano a fasce, nessun tetto
  per acquisto: tolti di proposito il 24/09; se servissero, ricostruirli da git.
- **Rosa dell'altra lega** ("Già miei altrove"): non nel codice (repo pubblico); si importa con
  link personale `index.html#miei=<id,id,...>`. Su iOS Safari e app in Home hanno localStorage
  separati: aprire il link dove si usa davvero.
- **Persistenza**: solo `localStorage`. Supabase rimosso e progetto cancellato (23/09).
- **Rivali**: tab tolta; "Preso da" resta e alimenta i correttivi.

## Da ricordare in asta
- Registrare **ogni** vendita ("Non tracciato" se non si sa chi), altrimenti `cashFactor`
  sovrastima i prezzi.
- Il prezzo è un'indicazione: sulle stelle l'errore medio nelle simulazioni è 33-59 cr.
- Moltiplicatori (infortunati, sconto cambio/riserva) non verificati su un'asta vera.
