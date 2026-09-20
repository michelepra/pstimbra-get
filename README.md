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

## Convenzione di versione e changelog

- Il **tag** della release è la versione, sempre `vX.Y.Z` (es. `v1.3.0`),
  identico al campo `version:` di `pstimbra/pubspec.yaml` (senza il build
  number `+N`, non significativo). L'app di PSTimbra legge questo stesso
  tag (via l'API "latest release" di questo repo) per sapersi dire "c'è
  una versione più recente" — vedi `pstimbra/lib/update_service.dart`.
- Il **corpo della release** è il changelog mostrato sia agli utenti sulla
  pagina di download sia nel dialog di aggiornamento in-app: sintetico,
  scritto a mano (non un dump di `git log`), due sole sezioni:

  ```markdown
  ### Novità
  - Timbratura rapida in background anche su iOS

  ### Correzioni
  - Corretto calcolo del verso proposto dopo un'omessa timbratura
  ```

  Sezione assente se non pertinente (es. solo bugfix: niente "### Novità").
  `pstimbra/scripts/draft-changelog.sh v1.2.0` genera una bozza raggruppando
  i commit `feat:`/`fix:` dal tag precedente a `HEAD`, da rifinire a mano
  (i messaggi di commit sono per chi programma, il changelog per chi usa
  l'app: va riscritto in prosa comprensibile, non incollato).

## Come pubblicare una nuova release Android (APK)

1. `flutter build apk --release` nel repo `pstimbra`.
2. In questo repo: **Releases → Draft a new release**, tag `vX.Y.Z` (vedi
   convenzione sopra), corpo = changelog, allega il file `.apk` come asset,
   pubblica.
3. Fatto: `index.html` rileva da solo l'ultima release via GitHub API e mostra
   il pulsante di download — nessuna modifica manuale alla pagina.

## Come pubblicare una release iOS (IPA)

Un `.ipa` scaricato non si installa da solo su iOS (Safari non lo sa aprire):
serve un manifest OTA (`manifest.plist`, vedi `manifest.plist.template` in
questo repo) caricato come **secondo asset della stessa release**, che
`index.html` usa per costruire il link `itms-services://` corretto.

1. `flutter build ipa --release` nel repo `pstimbra` (richiede macOS/Xcode,
   vedi il README di `pstimbra`).
2. **Releases → Draft a new release**, stesso tag `vX.Y.Z` della release
   Android (o una release iOS-only se le build non escono insieme), allega
   l'`.ipa`, **salva come bozza** (non pubblicare ancora).
3. Copia l'URL `browser_download_url` dell'asset `.ipa` appena caricato
   (visibile nella pagina della bozza, o `gh release view <tag> --json assets`).
4. Copia `manifest.plist.template`, sostituisci `REPLACE_WITH_IPA_ASSET_URL`
   e `REPLACE_WITH_VERSION`, salva come `manifest.plist`.
5. Carica `manifest.plist` come secondo asset della stessa release, pubblica.

Limite non aggirabile con questo meccanismo (non tecnico di questo repo,
ma della distribuzione ad-hoc Apple): funziona solo sui dispositivi già
registrati nel provisioning profile usato per firmare l'IPA.

## Quando si pubblica su Play Store / App Store

Aggiornare `index.html` per puntare allo store al posto dell'APK diretto
(un solo punto di modifica, l'URL stampato sulla card resta identico).
