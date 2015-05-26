# Sinapsi
## Web Service
### Panoramica
Sinapsi è costituito da un Web Service, scritto completamente in java, che permette una comunicazione sicura e stabile tra i clients. 
  
### Comunicazione
Per la comunicazione tra clients e web service, viene usata l’architettura REST (REpresentational State Transfer). REST è un protocollo client-server e stateless che punta sul concetto importante di risorsa, a cui si può accedere tramite un identificatore globale (un URI). Per utilizzare le risorse, le componenti di una rete (componenti client e server) comunicano attraverso una interfaccia standard, HTTP,  e si scambiano rappresentazioni di queste risorse, nel nostro caso è una stringa JSON.

_Esempio di richiesta GET tramite REST:_        
`http://massolit.ns0.it:8181/sinapsi/devices?action=get`    
In questo caso, il client cerca di connetterersi al server “massolit.ns0.it” sulla porta “8181”, e chiede alla servlet “devices” la lista di devices collegati. Tutto questo procedimento viene gestito da una libreria java (RETROFIT) che permette di mandare delle request di tipo GET, POST, et ad un Web Service, usando le annotazioni di java. 
_Esempio di richiesta tramite retrofit:_

```@GET(/devices?action=get)  
   public void getAllDevicesByUser(  
         @Query("email") String email,  
         Callback<List<DeviceInterface>> devices);```  
  
La risposta del server è una stringa JSON del tipo:
`{[email:”email”,name:”Macbook”,model:”Air”],[email:”email”,name:”HTC”,model:”One S”]}`
Tramite la libreria Gson di Google, questa stringa JSON viene poi convertita dal client in un Object java, in particolare nel esempio precedente in un Object di tipo: _List\<DeviceInterface\>_.
 
### Sicurezza
La comunicazione tra clients e server è criptata tramite la libreria BGP. BGP (Bit Good Privacy) offre strumenti per criptare/decriptare utilizzando i benefici della crittografia simmetrica e asimmetrica.

##### Funzionamento BGP
Data la stringa da criptare, questa viene compressa per ridurre lo spazio e ancora più importante per evitare attacchi di tipo Known-plaintext. Successivamente BGP crea una chiave di sessione, di default a 128 bit, che viene utilizzata per criptare il testo in chiaro compresso. Una volta criptato il testo, viene criptata la chiave di sessione usando la chiave pubblica, RSA di default a 1024 bit, del mittente. Infine vengono inviati testo criptato e chiave di sessione criptata, il destinatario decrepita in modo analogo. Decrepita la chiave di sessione usando la propria chiave privata, ottenendo la chiave di sessione originale, che viene usata per decriptare il testo compresso criptato, infine si decomprime il testo per ottenere il testo in chiaro.

#### Autenticazione
Il client al momento della connessione manda una richiesta di login, inviando email e la propria chiave pubblica appena generata. Il server aggiorna la chiave pubblica dell’utente nel db, genera le chiavi, aggiorna le proprie chiavi associate all’utente nel db e invia al client la propria chiave pubblica e la chiave di sessione criptata. Il client a questo punto è pronto per effettuare il login vero e proprio; invia al server la chiave di sessione generata e la password criptata. Il server decrepita la password, salva la chiave di sessione del client nel db, e controlla l’utente se esiste e se la password coincide , infine risponde al client mandando una descrizione dello user con i dispositivi collegati, il tutto come stringa JSON criptata. 
 
#### Sicurezza della comunicazione
Qualsiasi request/response viene criptata. La generazione delle chiavi avviene solo nella fase di login, le quali vengono salvate nel db centrale del server, il quale per ogni utente associa: chiave pubblica, chiave di sessione, chiave privata del server, chiave pubblica del server e chiave di sessione del server, in più, ad ogni request da parte del client, viene generato un HMAC-SHA1 usando come dati input, i parametri della request e come chiave di firma la chiave di sessione decriptata. Usando l’HMAC si fa a meno di usare la sessione, in più si evita attacchi di tipo replay e man in the middle.
 
### Comunicazione full-duplex 
In Sinapsi si fa uso anche dei Web Socket per offrire una comunicazione in tempo reale e full-duplex tra web service e clients.
   
#### Utilizzo dei web socket
I web socket vengono utilizzati in Sinapsi per offrire il servizio di _Macro remote_. Le _Macro remote_ sono normali macro formati da azioni che devono essere eseguite su un device diverso da quello su cui è attivo il trigger. Per offrire questa funzionalità l’engine di Sinapsi, al momento dell’esecuzione della _Macro remota_, avverte il web service mandando i dati necessari all’esecuzione corretta della macro sul device target, il web service si occupa di fare da dispatcher, avvertendo i/il device target usando i web socket, i devices a loro volta, dopo il login, restano in ascolto sul web socket server, in caso di richiesta di esecuzione di azioni/azione da parte di una macro remota.

### Services
Il Web Service viene visto come un device particolare da parte dell’Engine di Sinapsi, essendo un device offre a sua volta componenti di tipo Triggers e Actions.

#### Web Service Actions
Le azioni nel web service vengono eseguite nel momento in cui si verifica un certo trigger. La scelta delle azioni da creare nel web service dipende dalla tecnologia usata per costruire il web service e anche dall’effettiva utilità che potrebbe avere una certa azione nel web service, in particolare non avrebbe senso nel web service impostare azioni che richiedono interazioni con l’utente. Possibili azioni offerte dal web service sono:
- _Send email_: manda un email
- _Log_: salva su file/db un log
- _Tweet_: posta su twitter un determinato tweet
- _Post_: posta su Facebook un determinato post
- _Save on Dropbox_: salva su dropbox un determinato file
- …

#### Web Service Triggers
I Triggers sono particolari eventi che una volta verificati, permettono di avviare una serie di azioni. Nel caso più generale un trigger potrebbe essere la ricezione di un sms o una email, o in generale un qualsiasi cosa che generi un “cambiamento” di stato nel dispositivo su cui era attivo “l’ascolto” del trigger.  Nel web service è possibile settore come Trigger eventi del tipo:
- _Email Recived_: email ricevuta
- _Device connected_: dispositivo connesso
- _GET Request_: richesta GET ricevuta
- _POST Request_: richiesta POST ricevuta
- _User sign in_: login da parte di un utente
- …





	 
	 
