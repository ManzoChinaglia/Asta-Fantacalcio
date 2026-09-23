# FantaQintesi (Asta-Fantacalcio) — contesto per Claude Code

Tool d'asta per la lega Fantalab: 8 squadre, 500 crediti, Classic + modificatore difesa.
Un solo `index.html` (`scout.html` è solo un redirect). Stato e cronologia: `STATO.md`.

- Si usa solo dal telefono, un dispositivo alla volta: niente sincronizzazione (Supabase scartato).
- Repo pubblico: mai chiavi né rose vere nel codice. La rosa dell'altra lega entra solo col
  link personale `index.html#miei=<id,...>`. Su iOS il localStorage di Safari e quello dell'app
  sulla Home sono separati.
- I dati live di titolarità e infortuni arrivano da JARVIS (raw GitHub), con fallback statico.
  Non usare `modello.json` né `statistiche.json` di JARVIS.
- Provare in locale con un server statico prima di consegnare.
- Grafica: stile liquid glass come JARVIS, con mockup prima del codice.
- Niente push: i commit li controlla l'utente prima di pubblicare.
