# S33 - Stompbox

Pedaliera di effetti audio per il browser, pensata per **iPhone in verticale** e installabile come **PWA**.
Quattro pedali in catena: **Chorus**, **Flanger**, **Delay** e **Riverbero**. Il suono è elaborato in tempo reale con la Web Audio API e quattro `AudioWorklet` scritti a mano, senza librerie esterne.

## Caratteristiche

- **Quattro effetti** con interfaccia a pedali: manopole, LED e footswitch per attivare o escludere ogni effetto.
- **Tre sorgenti d'ingresso**: microfono, file audio (riprodotto in loop) e un generatore di note interno per provare gli effetti.
- **Tap tempo** sul delay, con display a tre cifre a sette segmenti che mostra i BPM corrispondenti al tempo impostato.
- **Indicatori di livello ad ago** (IN e OUT) con spia di clipping.
- **Pedali riordinabili** trascinando la maniglia ≡: l'ordine cambia anche nel segnale audio.
- **Funziona offline** una volta installata (service worker).
- Un solo file `index.html`, senza build e senza dipendenze.

## Come si usa

| Gesto | Effetto |
| --- | --- |
| Trascinare una manopola in verticale | Cambia il valore (su = di più) |
| Doppio tocco su una manopola | Riporta il valore iniziale |
| Toccare il footswitch del pedale | Attiva o esclude l'effetto (il LED si accende) |
| Toccare più volte **TAP** | Imposta il tempo del delay sulla media degli ultimi tocchi (massimo cinque) |
| Trascinare la maniglia ≡ di un pedale | Sposta il pedale nella catena |

All'avvio il delay è acceso, gli altri effetti sono spenti. La catena predefinita è:

```
Ingresso → Chorus → Flanger → Delay → Riverbero → Uscita
```

> Con il **microfono** usa le cuffie, altrimenti il segnale in uscita rientra nel microfono e si genera feedback acustico. L'elaborazione del browser (cancellazione dell'eco, riduzione del rumore, controllo automatico del guadagno) è disattivata di proposito.

## Parametri

| Pedale | Parametro | Intervallo |
| --- | --- | --- |
| Chorus | Velocità | 0,1 – 5 Hz |
| | Profondità, Mix | 0 – 100% |
| Flanger | Velocità | 0,05 – 3 Hz |
| | Profondità, Mix | 0 – 100% |
| | Feedback | da −90% a +90% |
| Delay | Tempo | 10 – 1500 ms |
| | Feedback | 0 – 95% |
| | Tono (filtro nel feedback) | 200 Hz – 16 kHz |
| | Mix | 0 – 100% |
| Riverbero | Decadimento | 0 – 100% |
| | Smorzamento (alte frequenze) | 800 Hz – 16 kHz |
| | Pre-delay | 0 – 150 ms |
| | Mix | 0 – 100% |

Le manopole di tempo, velocità e frequenza hanno una risposta logaritmica, più precisa sui valori bassi.

## Come funzionano gli effetti

- **Delay**: buffer circolare con interpolazione cubica (Hermite), filtro passa-basso nel percorso di feedback e saturazione morbida (`tanh`) per evitare che il feedback esploda. I parametri sono smussati per evitare i click.
- **Chorus**: linee di ritardo modulate da un LFO (circa 14 ms di ritardo centrale), due voci per canale con fasi diverse, per un leggero allargamento stereo anche con ingresso mono.
- **Flanger**: ritardo molto breve (circa 0,5 – 4,5 ms) modulato da un LFO, con feedback che può essere positivo o negativo.
- **Riverbero**: algoritmo a piastra nello stile di Dattorro. Quattro filtri allpass in ingresso, poi due rami incrociati con allpass modulati per evitare il suono metallico, filtro di smorzamento e uscite stereo prelevate da più punti dei rami.

Il riverbero segue lo schema descritto in J. Dattorro, *Effect Design, Part 1: Reverberator and Other Filters*, Journal of the Audio Engineering Society, 1997.

## Installazione

La PWA richiede **HTTPS**: il service worker non parte da un file aperto in locale.

### Pubblicare con GitHub Pages

1. Crea un repository e carica tutti i file di questa cartella nella radice.
2. Vai in **Settings → Pages**.
3. In *Build and deployment* scegli **Deploy from a branch**, seleziona il ramo `main` e la cartella `/ (root)`, poi salva.
4. Dopo un minuto circa la pagina è disponibile all'indirizzo `https://<utente>.github.io/<repository>/`.

I percorsi dei file sono relativi, quindi l'app funziona anche in una sottocartella come questa.

### Aggiungerla alla Home dell'iPhone

1. Apri l'indirizzo con **Safari**.
2. Tocca **Condividi** e poi **Aggiungi alla schermata Home**.
3. Apri l'app dall'icona sulla Home: solo così si comporta come app a schermo intero.

### Aggiornare l'app

Dopo ogni modifica a `index.html`, aumenta il numero di versione della cache in `sw.js`:

```js
const CACHE = 'stompbox-v1'; // → 'stompbox-v2', 'stompbox-v3', ...
```

Se non lo fai, i telefoni che l'hanno già installata continuano a mostrare la vecchia copia.

## Struttura del progetto

```
.
├── index.html            # interfaccia, logica e codice audio (AudioWorklet)
├── manifest.webmanifest  # nome, icone e modalità di visualizzazione della PWA
├── sw.js                 # service worker: cache e funzionamento offline
├── icon-192.png
├── icon-512.png
├── apple-touch-icon.png
└── README.md
```

## Limiti noti

- Due PWA diverse non possono scambiarsi audio: il browser le tiene isolate. Per processare il suono di un'altra app serve collegare l'uscita del telefono a un ingresso (per esempio con un'interfaccia audio) e usare il microfono come sorgente, oppure integrare il codice degli effetti nell'altra app.
- Quando sposti un pedale mentre l'audio è in riproduzione, può sentirsi un piccolo scatto nel momento in cui la catena viene ricollegata.
- Il riverbero è un algoritmo a piastra: non simula le prime riflessioni di una stanza.
- Il delay ha lo stesso tempo sui due canali, quindi non c'è un vero effetto ping-pong.
- Sull'iPhone il tasto del silenzioso può azzerare l'audio del browser. Il codice imposta la sessione audio di iOS per evitarlo, ma se non senti nulla controlla il tasto.

## Idee per il futuro

- Preset per delay e riverbero.
- Ping-pong stereo per il delay.
- Salvataggio dell'ultima configurazione.
