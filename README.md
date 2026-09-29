# lenovo-yogabook-linux-setup
# Installazione Linux su Lenovo Yoga Book (YB1-X91L/F)

Questo repository documenta l'approccio ottimale per installare distribuzioni Linux (es. Ubuntu) su Lenovo Yoga Book, configurando correttamente i driver per la tastiera aptica Halo e aggirando le limitazioni hardware legate all'alimentazione delle porte USB OTG in fase di boot.

## 1. Il Problema Hardware e il BIOS
La porta micro-USB dello Yoga Book eroga una quantità di corrente estremamente limitata, insufficiente per alimentare hub non alimentati a cui sono connesse tastiere cablate standard. Poiché la tastiera Halo integrata richiede driver in *userspace* non presenti nei Live CD standard, è impossibile interagire con i menu di boot senza aggirare questo limite elettrico. 

> **Avvertenza di sicurezza:** Non utilizzare mai alimentatori da 12V su hub USB con ingresso DC 5V nel tentativo di alimentare le periferiche. L'eccesso di tensione brucerebbe l'hub e, tramite backfeeding, danneggerebbe irreversibilmente la scheda madre del tablet.

---

## 2. Metodi di Installazione

A seconda dell'hardware a disposizione, è possibile procedere in due modi differenti.

### Metodo A: Con Hub alimentato a 5V o Tastiera Wireless (Dongle 2.4GHz)
Questo è il metodo standard che permette l'uso di software multiboot come Ventoy.

1. **Preparazione:** Creare un'unità Ventoy e copiare la ISO di Ubuntu (versione desktop standard con GNOME, richiede una chiavetta da almeno 8GB).
2. **Setup Hardware:** 
   - Utilizzare un hub USB alimentato esternamente (rigorosamente a 5V, ad esempio tramite powerbank) **OPPURE**
   - Collegare direttamente un dongle USB da 2.4GHz di una tastiera wireless (l'assorbimento è minimo e supportato dal tablet).
3. **Avvio:** Accendere il tablet premendo `Volume Su + Power`, selezionare il Boot Menu e avviare Ventoy.
4. **Installazione:** Procedere con la normale installazione usando la tastiera fisica per la navigazione.

### Metodo B: Installazione "Touch Only" (Senza periferiche esterne compatibili)
Se non si dispone di hub alimentati o tastiere a basso assorbimento, è possibile bypassare la necessità di input fisico nel BIOS sfruttando il touchscreen in ambiente Live.

1. **Preparazione ISO Diretta:** Utilizzare un tool come Rufus o BalenaEtcher per flashare la ISO di Ubuntu (desktop con GNOME) direttamente sull'unità USB da almeno 8GB (evitando menu intermedi come Ventoy).
2. **Avvio:** Accendere tenendo premuto `Volume Su + Power`. Navigare il menu del BIOS utilizzando i tasti fisici del volume e confermare con il tasto di accensione.
3. **Accessibilità in ambiente Live:** Una volta avviato Ubuntu in Live, il touchscreen capacitivo funzionerà nativamente (grazie all'ambiente GNOME). Dal menu in alto a destra, attivare la **Tastiera a schermo (Screen Keyboard)** nelle opzioni di Accessibilità.
4. **Installazione:** Completare il partizionamento e il setup utente digitando a schermo.

---

## 3. Post-Installazione: Driver per Tastiera Halo e Audio
Una volta completata l'installazione, la tastiera Halo risulterà ancora inattiva e mancherà l'audio. È necessario installare i pacchetti e il kernel patchato sviluppati dalla community.

### 3.1 Scaricare i pacchetti necessari
Dall'ambiente Ubuntu appena installato, aprire il browser e recarsi alla pagina Releases del repository jekhor/yogabook-linux (https://github.com/jekhor/yogabook-linux/releases).
Scaricare i seguenti file `.deb` dell'ultima release disponibile:
- `linux-image-...-yogabook_..._amd64.deb` (Il kernel patchato)
- `touch-keyboard_..._amd64.deb` (Il demone userspace per la tastiera)
- `yogabook-support_..._all.deb` (Script di supporto)
- `alsa-ucm-conf-yogabook_..._all.deb` (Configurazioni audio)

### 3.2 Installazione tramite terminale
Aprire il terminale (tramite touchscreen e tastiera a schermo o tramite tastiera esterna), navigare nella cartella in cui sono stati salvati i file e procedere con l'installazione:

    cd Scaricati
    sudo dpkg -i *.deb

*(Nota: se il sistema è stato installato in lingua inglese, la cartella sarà `cd Downloads`)*

### 3.3 Risoluzione dipendenze
È molto probabile che il comando precedente segnali delle dipendenze mancanti (errori di pacchetti non installati). Per risolvere automaticamente il problema e completare l'installazione, eseguire:

    sudo apt-get install -f

### 3.4 Riavvio e Verifica
Applicate le configurazioni, riavviare il sistema:

    sudo reboot

Al riavvio, il sistema caricherà automaticamente il nuovo kernel personalizzato (verificabile aprendo il terminale e digitando `uname -a`, che restituirà una stringa contenente "yogabook"). La tastiera Halo si illuminerà, restituirà il feedback aptico e funzionerà regolarmente come dispositivo di input, così come la scheda audio integrata.
