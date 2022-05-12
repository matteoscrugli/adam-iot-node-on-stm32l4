# ADAM - IoT node on STM32L4

E' possibile comunicare con il nodo tramire il modulo BLE, il nodo è visibile con il nome 'STLB100'.

## Scrittura
E' possibile scrivere sulla seguente caratteristica per inviare un comando:
- servizio: 00000000-ffff-11e1-9ab4-0002a5d5c51b
- caratteristica: 00000002-ffff-11e1-ac36-0002a5d5c51b

Il primo byte scritto stabilisce il tipo di comando:
- 0x01: modificare la catena di thread attivi.
- 0x02: modifica la frequenza di campionamento di un sensore.
- 0x04: modifica il periodo di invio dei dati al gateway.


### Comando 0x01:
Con il secondo byte si controllano i diversi operating mode (opm) di tutti i sensori, tradotto in binario abbiamo:
0b aa bb cc dd
- aa: quattro combinazioni per il sensore 3 (respiro)
- bb: quattro combinazioni per il sensore 2 (pressione)
- cc: quattro combinazioni per il sensore 1 (temperatura)
- dd: quattro combinazioni per il sensore 0 (ecg / battito)

Le quattro divese combinazioni:
- 00: catena spenta
- 01: raw data, invio di tutti i sample al gateway
- 10: primo livello di processing abilitato
- 11: secondo livello di processing abilitato
Esempio:
scrivo 0x 01 07 = 0b 00000001  00000111, quindi "11" per il sensore 0 e "01" per il sensore 1, ovvero il sensore 2 manda i dati in raw mentre il sensore 1 abilita il primo livello di processing (calcolo battito cardiaco).

### Comando 0x02:
Con i successivi byte viene specificato il periodo di sampling di uno specifico sensore
se viene scritto 0x 02 AA BB BB BB risulta:
- AA: sensore n
- BB BB BB: numero espresso in ms in formato little-endian
Esempio:
scrivo 0x02 00 F4 01, setto il periodo di invio dei dati del sensore 0 a 500 ms (0d500 = 0x01F4)

### Comando 0x04:
Con i successivi byte viene specificato il periodo di tempo tra un invio al gateway e l'altro di uno specifico sensore
se viene scritto 0x 04 AA BB BB BB risulta:
- AA: sensore n
- BB BB BB: numero espresso in ms in formato little-endian
Esempio:
scrivo 0x04 00 F4 01, setto il periodo di invio dei dati del sensore 0 a 500 ms (0d500 = 0x01F4)



## Lettura
Le caratteristiche in lettura sono:
- servizio: 00000000-0001-11e1-9ab4-0002a5d5c51b
- caratteristica sensore 0: 14000000-0001-11e1-ac36-0002a5d5c51b
- caratteristica sensore 1: 07000001-0001-11e1-ac36-0002a5d5c51b
- caratteristica sensore 2: 07000002-0001-11e1-ac36-0002a5d5c51b
- caratteristica sensore 3: 07000003-0001-11e1-ac36-0002a5d5c51b
Per poter ricevere le notifiche bisogna iscriversi ad una caratteristica, la procedura dipende dal vostro sistema, per poter ricevere dei dati bisogna sia iscriversi alle caratteristiche che abilitare i task come è stato spiegato in precedenza.
Anche qui, i campi dovranno essere letti con l'ordine little-endian.

Per tutte le caratteristiche, i primi 4 byte sono sempre riferiti al timestamp, ovvero un contatore in ms che viene avviato quando si alimenta la scheda.

Esempio caratteristica relativa al sensore 0, tipo di dati interi:
AA AA AA AA BB BB CC CC DD DD EE EE FF FF GG GG HH HH II II
- campo AA: timestamp
- campo BB BB: con il primo livello di processing viene letto il battito cardiaco, senza livello di processing viene letto un sample raw ecg
- da CC CC in poi: con il primo livello di processing viene settato tutto a zero, senza livello di processing vengono letti altri 7 sample ecg.

Esempio caratteristica relativa ai sensor 1 e 2, tipo di dati float:
AA AA AA AA BB BB CC
- campo AA: timestamp
- campo BB BB: parte intera
- campo CC: parte decimale

I dati inviati sono:

### Sensore 0
Viene simulato un tracciato ECG reale con sample a 16 bit e frequenza di campionamento di 330 Hz, il battito cardiaco relativo al tracciato è intorno ai 100 bpm.
- opm raw (0b01): invia i dati grezzi
- opm heartrate (0b10): invia la frequenza cardiaca
- opm qrs (0b11): invia i picchi qrs rilevati

### Sensore 1
Viene rilevata una temperatura fissa di 36.5 gradi celsius.
- opm raw (0b01): invia i dati grezzi

### Sensore 2
Viene rilevata una pressione sanguigna di 69 (minima) e 111 (massima) millimetri di mercurio.
- opm raw (0b01): invia i dati grezzi

### Sensore 3
Viene simulato un tracciato di respiro con sample a 16 bit e frequenza di campionamento a 67 Hz, la frequenza respiratoria è intorno ai 20 atti per minuto.
- opm raw (0b01): invia i dati grezzi
- opm breathrate (0b10): invia la frequenza respiratoria
