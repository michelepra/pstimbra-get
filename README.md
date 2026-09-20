# pstimbra-get

Landing page pubblica e stabile per la distribuzione di **PSTimbra**
(https://github.com/michelepra/pstimbra, sorgente privato).

Serve solo da "ponte": il QR/link stampato sulla card NFC punta sempre qui,
così non va mai ristampato anche se cambia dove/come si scarica l'app.

## Setup (una tantum)

1. Crea su GitHub un repo **pubblico** chiamato `pstimbra-get` (proprietario `michelepra`).
2. `git remote add origin git@github.com:michelepra/pstimbra-get.git`
   `git push -u origin main`
3. Settings → Pages → Source: **Deploy from branch**, branch `main`, cartella `/ (root)`.
   Dopo ~1 minuto è live su:
   **https://michelepra.github.io/pstimbra-get/**

## Come pubblicare una nuova build

1. `flutter build apk --release` nel repo `pstimbra`.
2. In questo repo: **Releases → Draft a new release**, tag (es. `v1.0.0`),
   allega il file `.apk` come asset, pubblica.
3. Fatto: `index.html` rileva da solo l'ultima release via GitHub API e mostra
   il pulsante di download — nessuna modifica manuale alla pagina.

## Quando si pubblica su Play Store / App Store

Aggiornare `index.html` per puntare allo store al posto dell'APK diretto
(un solo punto di modifica, l'URL stampato sulla card resta identico).
