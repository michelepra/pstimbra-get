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

## Come pubblicare una release iOS (IPA) — gratuita, via AltStore (default)

Niente account Apple a pagamento, niente manifest OTA: l'IPA prodotto da
Codemagic (vedi `pstimbra/codemagic.yaml` e `pstimbra/docs/ios-build-senza-mac.md`,
"Strada B") non è firmato — `index.html` lo rileva da solo (un asset `.ipa` **senza**
un `.plist` accanto) e mostra un pulsante che passa per AltStore
(`altstore://install?url=...`), che scarica/firma/installa in locale con l'Apple ID
gratuito di chi lo usa. Nessun passaggio manuale di firma qui.

1. Lancia il workflow `ios-unsigned` su Codemagic (o l'equivalente
   `.github/workflows/ios-build.yml` se si usa GitHub Actions) sul repo `pstimbra`:
   produce `pstimbra-unsigned.ipa`.
2. In questo repo: **Releases → Draft a new release** (o la stessa release Android se
   escono insieme), tag `vX.Y.Z`, corpo = changelog, allega **solo**
   `pstimbra-unsigned.ipa` come asset (nessun `manifest.plist`), pubblica.
3. Fatto: alla prossima visita `index.html` mostra "Installa per iOS via AltStore
   (vX.Y.Z)". Chi non ha ancora AltStore vede, dopo il tap, le istruzioni per
   installarlo (una tantum); chi lo ha già lo vede aprirsi e installare/aggiornare
   PSTimbra.

Limite noto, non aggirabile (vincolo Apple su un Apple ID gratuito, non di questo
repo): il certificato di firma usato da AltStore scade ogni 7 giorni. Il rinnovo è
automatico se AltServer resta raggiungibile da AltStore sulla stessa rete Wi-Fi
**oppure** se si configura una tantum "Remote AltServer" nelle impostazioni di
AltStore (pairing col PC + profilo VPN "LocalDevVPN", vedi
[faq.altstore.io/altstore-classic/remote-altservers](https://faq.altstore.io/altstore-classic/remote-altservers)):
da lì in poi il rinnovo avviene tramite AltServer remoti gestiti dalla community,
**senza bisogno del proprio PC acceso** (stesso vantaggio di SideStore, vedi sotto,
ma dipende dalla disponibilità di quei server remoti). Senza nessuna delle due,
il rinnovo va fatto a mano riaprendo AltStore — dettagli in
`pstimbra/docs/ios-build-senza-mac.md`.

### Alternative ad AltStore per l'installazione gratuita (SideStore, Sideloadly)

Il pulsante su `index.html` punta sempre ad **AltStore** (default). Chi preferisce
un'altra modalità di sideload gratuita scarica comunque `pstimbra-unsigned.ipa` dalla
release su questo repo (link "Vedi la release") e lo usa con uno degli strumenti
sotto. Vincoli comuni a **tutte** le modalità (limite di un Apple ID gratuito, non di
questo repo né dello strumento scelto): certificato valido **7 giorni**, **max 3 app**
firmate contemporaneamente e **max 3 dispositivi** per Apple ID.

| | **AltStore** (default) | SideStore | Sideloadly |
| --- | --- | --- | --- |
| Setup iniziale | AltServer su PC (Windows/Mac) + abbinamento del dispositivo (USB o Wi-Fi), una tantum | Un tool di pairing (es. [iDevice Pair](https://github.com/jkcoxson/idevice_pair)) su PC genera un "pairing file" una tantum; **oppure**, più semplice se si ha già AltStore configurato, si installa SideStore come app da una fonte in AltStore e poi non serve più AltStore | App desktop (solo Windows/macOS) che firma e installa via USB a ogni release, nessun abbinamento persistente |
| Rinnovo del certificato (ogni 7 giorni) | Automatico **se** AltServer resta raggiungibile sulla stessa rete Wi-Fi del telefono, **oppure**, senza PC, abilitando una tantum "Remote AltServer" nelle impostazioni (pairing + profilo VPN "LocalDevVPN" — [guida](https://faq.altstore.io/altstore-classic/remote-altservers), rinnovo tramite server remoti della community); altrimenti si riapre AltStore e si tocca "Refresh All" | Automatico **senza bisogno del PC** dopo il setup iniziale: SideStore si rinnova da sola in background (nessuna rete/server remoto da raggiungere) | Automatico **solo se** si tiene attivo il "Sideloadly Daemon" sul PC con il device sulla stessa Wi-Fi o via USB e l'auto-refresh abilitato nell'app; altrimenti va rifatto a mano (riconnettere il telefono via USB e rilanciare la firma) a ogni scadenza |
| Aggiornamento a una nuova release PSTimbra | Tap sul link `altstore://install?url=...` su questa pagina | Va aggiunta una fonte (source) in SideStore che punti all'IPA, oppure si reimporta manualmente il nuovo `.ipa` scaricato da questa pagina | Si riscarica `pstimbra-unsigned.ipa` da questa pagina e lo si trascina di nuovo nell'app desktop via USB |
| Più indicato per | Uso quotidiano — con PC raggiungibile in Wi-Fi, o con "Remote AltServer" configurato per non dipenderne più (default consigliato) | Chi non vuole dipendere né da un PC né da server remoti di terzi dopo il setup iniziale | Test rapidi una tantum o chi preferisce un controllo manuale esplicito a ogni rinnovo |

Nessuno dei tre richiede un account Apple a pagamento: cambiano solo dove/come avviene
la firma e quanto lavoro manuale resta a carico dell'utente dopo il setup iniziale. Con
"Remote AltServer" configurato, AltStore raggiunge sostanzialmente la stessa comodità
di SideStore restando comunque lo strumento di default — differenza residua: SideStore
passa da server/infrastruttura propria del progetto via notifiche push, AltStore passa
da AltServer remoti gestiti da volontari della community tramite una VPN dedicata.

## Come pubblicare una release iOS (IPA) — a pagamento, via OTA (alternativa)

Da usare solo se un giorno si decide di sostenere l'Apple Developer Program (99
$/anno) per avere installazione OTA a un tap senza AltStore, o per usare Core
NFC/Associated Domains (vedi `pstimbra/docs/ios-build-senza-mac.md`, "Strada A").

1. `flutter build ipa --release` nel repo `pstimbra` (richiede un certificato di
   distribuzione + provisioning profile ad-hoc, vedi `pstimbra/docs/ios-build-senza-mac.md`).
2. **Releases → Draft a new release**, stesso tag `vX.Y.Z` della release Android (o
   una release iOS-only se le build non escono insieme), allega l'`.ipa`, **salva come
   bozza** (non pubblicare ancora).
3. Copia l'URL `browser_download_url` dell'asset `.ipa` appena caricato (visibile
   nella pagina della bozza, o `gh release view <tag> --json assets`).
4. Copia `manifest.plist.template`, sostituisci `REPLACE_WITH_IPA_ASSET_URL` e
   `REPLACE_WITH_VERSION`, salva come `manifest.plist`.
5. Carica `manifest.plist` come secondo asset della stessa release, pubblica.

Un `.ipa` scaricato non si installa da solo su iOS (Safari non lo sa aprire): serve
questo manifest OTA, che `index.html` usa per costruire il link `itms-services://`
corretto — se è presente anche un manifest, `index.html` preferisce questo percorso
a quello AltStore. Limite non aggirabile con questo meccanismo (non tecnico di
questo repo, ma della distribuzione ad-hoc Apple): funziona solo sui dispositivi
già registrati nel provisioning profile usato per firmare l'IPA.

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
