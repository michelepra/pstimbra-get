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
- **Canale beta** (opzionale): stesso tag con suffisso `-beta.N` (es.
  `v1.3.0-beta.1`), marcato **pre-release** su GitHub — `GET
  /releases/latest` lo esclude nativamente, quindi resta invisibile al
  canale stable (`index.html` di default, app senza "Canale beta" attivato
  dal menu). Dettagli: sezione "Come pubblicare una beta" sotto.
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

## Come pubblicare una beta (canale beta)

Le beta usano lo stesso meccanismo delle release normali, con due differenze:
tag con suffisso `-beta.N` e flag **pre-release** attivo. `GET
/releases/latest` (usato sia da `index.html` in modalità normale sia
dall'app sul canale stable) esclude nativamente le pre-release: chi non ha
attivato il canale beta non le vede mai, zero rischio per lo stable.

1. `flutter build apk --release` nel repo `pstimbra`, con `version:` in
   `pubspec.yaml` nella forma `X.Y.Z-beta.N+B` (es. `1.3.0-beta.1+8`).
2. **Releases → Draft a new release**, tag `vX.Y.Z-beta.N` (stesso `X.Y.Z`
   della prossima stable prevista; `N` incrementale per build successive
   sullo stesso `X.Y.Z`), corpo = changelog, allega l'APK, **spunta "Set as
   a pre-release"**, pubblica. Con `gh` CLI:
   `gh release create vX.Y.Z-beta.N app-release.apk --title vX.Y.Z-beta.N
   --notes-file release-notes.md --prerelease`.
3. Distribuzione:
   - **In-app**: chi ha attivato "Canale beta" dal menu dell'app (vedi
     `pstimbra/lib/config_store.dart`, `UpdateChannel`) riceve la notifica
     di aggiornamento al prossimo controllo, automatico o da "Verifica
     aggiornamenti".
   - **Landing page**: `https://michelepra.github.io/pstimbra-get/?channel=beta`
     mostra il pulsante di download per l'ultima release del canale beta
     (stable + prerelease, vince la più recente per precedenza semver) — il
     link/QR stampato sulle card NFC **non** cambia e continua a mostrare
     solo lo stable.

Precedenza tra versioni (rilevante pubblicando più beta di fila sullo stesso
`X.Y.Z`, o passando da beta a stable): a parità di `major.minor.patch` una
release finale supera sempre una prerelease, e tra due beta vince il numero
più alto (`1.3.0-beta.2` > `1.3.0-beta.1`) — stessa regola semver
implementata sia in `pstimbra/lib/update_service.dart` sia in questo
`index.html`: se una cambia va aggiornata anche l'altra.

## Quando si pubblica su Play Store / App Store

Aggiornare `index.html` per puntare allo store al posto dell'APK diretto
(un solo punto di modifica, l'URL stampato sulla card resta identico).
