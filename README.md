# Installazione Linux su Lenovo Yoga Book (YB1-X91L/F)

Questo repository documenta l'approccio ottimale per installare distribuzioni Linux (es. Ubuntu) su Lenovo Yoga Book, configurando correttamente i driver per la tastiera aptica Halo e aggirando le limitazioni hardware legate all'alimentazione delle porte USB OTG in fase di boot e ai freeze grafici dell'architettura Intel Atom.

**Versione Raccomandata: Ubuntu 24.04 LTS**
L'esperienza ha dimostrato che è la versione ottimale da utilizzare. Con il kernel nativo di Ubuntu 24.04, il pannello touch (Goodix) è calibrato correttamente di default a installazione conclusa, eliminando definitivamente la necessità e il rischio di installare kernel patchati di terze parti per risolvere il problema degli assi specchiati.

---

## 1. Il Problema Hardware e il BIOS
La porta micro-USB dello Yoga Book eroga una quantità di corrente estremamente limitata, insufficiente per alimentare hub non alimentati a cui sono connesse tastiere cablate standard. Poiché la tastiera Halo integrata richiede driver in *userspace* non presenti nei Live CD standard, è impossibile interagire con i menu di boot senza aggirare questo limite elettrico. 

> **Avvertenza di sicurezza:** Non utilizzare mai alimentatori da 12V su hub USB con ingresso DC 5V nel tentativo di alimentare le periferiche. L'eccesso di tensione brucerebbe l'hub e, tramite backfeeding, danneggerebbe irreversibilmente la scheda madre del tablet.

---

## 2. Metodi di Installazione

A seconda dell'hardware a disposizione, è possibile procedere in due modi differenti.

### Metodo A: Con Hub alimentato a 5V o Tastiera/Mouse Wireless (Dongle 2.4GHz)
Questo è il metodo standard e più sicuro per evitare crash dovuti a cali di tensione sulla porta micro-USB.

1. **Preparazione:** Creare un'unità Live USB o Ventoy con la ISO di Ubuntu 24.04 LTS (richiede una chiavetta da almeno 8GB).
2. **Setup Hardware:** 
   - Utilizzare un hub USB alimentato esternamente (rigorosamente a 5V) a cui collegare chiavetta e mouse **OPPURE**
   - Collegare a un piccolo hub non alimentato la chiavetta USB e il dongle da 2.4GHz di un mouse wireless (l'assorbimento è minimo).
3. **Avvio:** Accendere il tablet premendo `Volume Su + Power`, selezionare il Boot Menu e avviare l'unità USB.
4. **Installazione:** Procedere con la normale installazione usando il mouse. Selezionare l'**Installazione Predefinita (Minimale)** per ridurre il tempo di scrittura sulla lenta memoria eMMC ed evitare crash.

### Metodo B: Installazione "Touch Only" (Senza mouse o hub)
1. **Preparazione ISO Diretta:** Utilizzare Rufus per flashare la ISO in modalità **GPT** e **UEFI (non CSM)**.
2. **Forzare Avvio Automatico e Safe Graphics:** 
   Nel file `boot/grub/grub.cfg` della chiavetta, impostare `set timeout=5` e aggiungere `nomodeset` alla fine della riga che inizia con `linux` per evitare blocchi video.
3. **Avvio:** Premere `Volume Su + Power` per selezionare la chiavetta dal BIOS. Attendere 5 secondi per l'avvio automatico di GRUB.

### 2.1 Sopravvivere all'ambiente Live (Esperienze e Workaround)
Durante l'installazione in ambiente Live, prima che i driver vengano installati, si verificheranno questi comportamenti:
*   **Touchscreen con Assi Invertiti:** Se non si dispone di un mouse, il pannello touch funzionerà ma risulterà capovolto di 180 gradi (muovendo il dito in basso a destra, il cursore va in alto a sinistra). Per cliccare un pulsante, **toccare il punto diametralmente opposto** sullo schermo.
*   **La tastiera Halo interferisce:** Se sfiorata, invia coordinate touch sballate. Evitare di toccarla durante l'installazione.
*   **Impossibile digitare Nome e Password:** Se la tastiera a schermo di GNOME non compare (o scompare a causa dell'input errato della base Halo), è possibile aggirare il problema del testo **aprendo Firefox (anche offline)**, evidenziando col mouse o col touch una qualsiasi lettera o parola preesistente, per poi usare il **Copia e Incolla** per riempire i campi obbligatori di Nome Utente e Password.

---

## 3. Post-Installazione: Driver per Tastiera Halo e Audio
Completata l'installazione e riavviato il PC rimuovendo l'unità USB, il touchscreen funzionerà in modo nativo e con l'orientamento corretto. Ora bisogna sbloccare la tastiera Halo e l'audio.

### 3.1 Scaricare i pacchetti necessari
Connettere il tablet al Wi-Fi. Aprire il browser e recarsi alla pagina Releases del repository jekhor: `https://github.com/jekhor/yogabook-linux/releases` (cercare la release `debs-20251024` o la più recente).

> **CRITICO - COSA NON SCARICARE:** Poiché Ubuntu 24.04 gestisce già il kernel e il touch correttamente, **NON** scaricare o installare pacchetti chiamati `linux-image` o `linux-headers`. L'installazione di questi pacchetti sostituirebbe il kernel moderno, rompendo nuovamente il touchscreen.

Scaricare **esclusivamente** questi tre pacchetti `.deb`:
*   `touch-keyboard_..._amd64.deb` (Motore della tastiera aptica)
*   `yoga-book-support-1.5-ubuntu..._all.deb` (Script di configurazione specifico per Ubuntu)
*   `alsa-ucm-conf-yoga-book_..._all.deb` (Configurazioni scheda audio Intel SST / Realtek)

### 3.2 Installazione tramite terminale
Aprire il terminale (usando la tastiera a schermo dal menu Accessibilità o un mouse) ed eseguire questi tre comandi in sequenza:

    cd Scaricati
    sudo dpkg -i *.deb
    sudo apt -f install -y

### 3.3 Riavvio e Verifica
Applicate le configurazioni, riavviare il sistema. Al riavvio, la tastiera Halo si illuminerà, vibrerà al tocco e funzionerà come un normale dispositivo di input testuale. Anche l'audio integrato risulterà funzionante.

---

## 4. Bug Hardware Noto: Ricarica della batteria
Sull'architettura Intel Cherry Trail con Linux, il driver di gestione energetica (`bq24190`) soffre di un bug legato all'hot-plugging.

**Il Problema:** Se si collega il cavo di ricarica alla porta micro-USB mentre il tablet è acceso e in uso, il sistema operativo non rileva l'ingresso di corrente e la batteria non si ricarica (il led laterale non lampeggia).

**Il Workaround Infallibile (Ciclo di Sospensione):**
Non è necessario riavviare il tablet o perdere il lavoro in corso per farlo caricare.
1. Mettere il tablet in **Standby/Sospensione** (chiudendo lo schermo a libro o premendo brevemente il tasto Power). Lo schermo diventerà nero.
2. Mentre è in standby, **collegare il cavo di ricarica**. Il controllo dell'energia sarà gestito dal BIOS, che riconoscerà l'alimentazione.
3. Attendere un paio di secondi e **risvegliare il tablet** (riaprendo lo schermo o premendo Power).
4. Linux si risveglierà ereditando lo stato corretto dell'alimentazione e continuerà a ricaricare normalmente la batteria durante l'utilizzo.
