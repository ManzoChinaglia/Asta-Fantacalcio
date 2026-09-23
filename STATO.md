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
  Il progetto Supabase resta attivo (senza tabelle) — cancellarlo del tutto va fatto a
  mano dalla dashboard, nessuno strumento automatico lo fa.

## Aperto — piano di unificazione (deciso per proposte il 23/09/2026)

1. Un'unica app, due schede: "Prepara" (oggi scout) + "Live" (oggi index), stesso
   dataset e stesso stato.
2. Dati live da Jarvis: leggere via URL grezzo da GitHub `dati/statistiche.json`,
   `dati/titolari.json`, `dati/infortuni.json`, `dati/modello.json` del repo JARVIS
   (pubblici, fuori dal lucchetto) invece di uno scraping proprio.
3. ~~Supabase da decidere~~ — **fatto**, sganciato (sopra).
4. Diversificazione manuale: lista "giocatori già miei altrove" compilata a mano (niente
   collegamento con le rose di Rivoluzione Fantacalcio, che sono in un repo diverso e
   cifrate).
5. Level-up per uso durante un'asta vera: ricerca sempre a fuoco, colpo d'occhio "chi
   comprare ora" all'apertura, undo sempre a portata (già in index), avviso sfondamento
   budget di ruolo.
6. Default della nuova asta: 8 partecipanti (nomi provvisori), 500 crediti.

Nessuno dei punti 1, 2, 4, 5, 6 è ancora iniziato.

## Decisioni

- Niente sincronizzazione multi-dispositivo: chiarito il 23/09/2026 che le due app si
  usano solo dal telefono, mai insieme a un altro dispositivo — da qui la scelta di
  sganciare Supabase invece di sistemarne le policy di sicurezza.
