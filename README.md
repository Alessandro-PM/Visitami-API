# Visitami-API
Visitami API versione 0.9.2.6

Questo progetto contiene l'elenco delle funzionalità per l'integrazione con le API Visitami e un codice di esempio da usare come guida per le chiamate in PHP.

# Prerequisiti

a) Concordare per l'acquisizione di un <b>Token Applicazione</b>;
b) Comunicare la URL certificata (https) da cui far provenire le richieste in modo che questa venga aggiunta alle URL per cui esiste un grant per le API;
c) Definire una propria pagina in cui ospitare il <b>Widget Dinamico</b> di prenotazione Visitami;
d) Conoscere ed eventualmente mappare le Categorie di Prestazioni presenti su Visitami;

# Url API

Esistono 2 url, uno di sviluppo e l'altro di produzione:

<i> Sviluppo </i>
https://demo.visitamiapp.com/app_api_web/genApi.asmx (possibile visionare WSDL)

<i> Produzione </i>
https://www.visitamiapp.com/app_api_web/genApi.asmx (WSDL bloccato)

# Chiamate API

Tutte le chiamate API prevedono una chiamata SOAP/Ajax con il passaggio di un token (alcune prevedono il passaggio del <b>Token Applicazione</b>, altre del <b>Token Utilizzo</b>) come parte dell'Header SOAP. Tutte le chiamate restituiscono JSON.
<i>vedere esempio nel file <b>token.php</b></i>

# Ricavare il Token Utilizzo (necessario possedere Token Applicazione)

Il <b>Token Applicazione</b> identifica univocamente l'applicazione che invoca le API e viene consegnato da Visitami al partner.
Attraverso il <b>Token Applicazione</b> è possibile ottenere e rinnovare il <b>Token Utilizzo</b>, tramite il quale chiamare le API.
Il <b>Token Utilizzo</b> ha una scadenza di 30 giorni, dopo i quali va rinnovato.

    Public Function APIGEN_TokenRenew(ByRef lErr As String) As String

* lErr (out) = parametro in uscita valorizzato con l'eventuale errore se il metodo ritorna emtpystring

Questa chiamata produrrà un JSON di questo tipo:

    {"token":"<TOKEN UTILIZZO>","issued":1497,"validuntil":1527,"UTCdatetime":"20190308"}

* Token      = Token Utilizzo per le successive chiamate API
* issued     = id del giorno di richiesta
* validuntil = id del giorno di fine validità
* UTCdatetime= data ultimo giorno di validità nel formato yyyyMMdd


<i>TUTTE LE CHIAMATE DA QUESTO PUNTO IN POI NECESSITANO DEL TOKEN DI UTILIZZO DA PASSARE NELL'HEADER</i>


# Ricavare Città

E' previsto un metodo per ricavare l'elenco delle Città di copertura del servizio Visitami abilitate alle API:

    Public Function APIGEN_GetCitta(ByRef lErr As String) As String

* lErr (out) = parametro in uscita valorizzato con l'eventuale errore se il metodo ritorna emtpystring

Questa chiamata produrrà un JSON di questo tipo:

    [{"nome":"Milano","lat":"45.4642035","lng":"9.189982","codice":"MI-1","raggio":15}]

* nome     = nome della Città da mostrare
* lat      = latitudine
* lng      = longitudine
* codice   = Codice Univoco che identifica la città
* raggio   = raggio in Km di copertura a partire dalle coordinate gps

# Ricavare Elenco delle Categorie di Prestazioni

E' previsto un metodo per ricavare l'elenco delle Categorie di Prestazioni presenti su Visitami:

    Public Function APIGEN_GetCategorie(ByRef lErr As String) As String

* lErr (out) = parametro in uscita valorizzato con l'eventuale errore se il metodo ritorna emtpystring

Questa chiamata produrrà un JSON di questo tipo:

    [{"IdCategoria":56,"Nome":"Agopuntore"},{"IdCategoria":10,"Nome":"Allergologo-Immunologo"},{"IdCategoria":53,"Nome":"Andrologo"},{"IdCategoria":11,"Nome":"Angiologo-Chirurgo Vascolare"},....]
    
* IdCategoria   = Id univoco Categoria Prestazioni
* Nome          = nome della categoria da mostrare

# Ricerca dei Professionisti (Semplice)

Questo metodo restituisce l'elenco dei professionisti disponibili per una data categoria <b>in studio</b>. In questa versione di API non è prevista la ricerca a domicilio.
L'elenco del professionisti restituiti mostra solo la prima disponibilità per la <i>prestazione base</i> della categoria prestazioni richiesta (ad es. se categoria = "Medico Generico", prestazione di base = "Visita medica generica" etc). 
L'elenco restituito è ordinato per prima disponibilità.

    Public Function APIGEN_DoSearchSimple(ByVal idcate As Integer, ByVal codCitta As String, ByRef lErr As String) As String

* idcate     = Id univoco Categoria Prestazioni
* codCitta   = Codice Univoco che identifica la città
* lErr (out) = parametro in uscita valorizzato con l'eventuale errore se il metodo ritorna emtpystring

Questa chiamata produrrà un JSON di questo tipo:

    [{"Nome":"Francesco Bianchi","Tit":"Dr.","Specializzazione":"medico generico","Sesso":"M","UrlImmagine":"https://demo.visitamiapp.com/getpic?MaxWidth=80\u0026PicID=!PROF@1117:med\u0026Token=TXRCdHoycklSUVl3V1ZrTDhCcFp3UVVWaDg1c0szcGR2b2YyQjFVd1M5ND0=\u0026MyToken=TXRCdHoycklSUVl3V1ZrTDhCcFp3UVVWaDg1c0szcGR2b2YyQjFVd1M5ND0=","Valutazione":3.1,"TokenProf":"TXRCdHoycklSUVl3V1ZrTDhCcFp3UVVWaDg1c0szcGR2b2YyQjFVd1M5ND0=","IDVisitami":1117,"IdPrestazione":184,"PrimaDisponibilita":{"IdDay":1497,"IdHour":77,"Data":"OGGI","Ora":"19:00","Luogo":"Studio a Milano zona Barona","Indirizzo":"Viale Famagosta, 50, Milano, 20142 MI","IdLuogo":4938,"Tariffa":"200-250 €","TariffaScontata":"140-175 €","Distanza":3.6,"Sconto":"-30%","ScontoD":0.3}},....]
    
* Nome               = Nome e Cognome del professionista o Nome della Struttura
* Tit                = Titolo (Dr./Prof. o vuoto)
* Specializzazione   = Nome della specializzazione principale
* Sesso              = M/F/S ( S=struttura)
* UrlImmagine        = URL dell'immagine del professionista/struttura
* Valutazione        = Rating (da 1 a 5). Valore -1 = nessuna valutazione espressa
* TokenProf          = Token Professioinsta da utilizzare nel <b>Widget dinamico</b>
* IDVisitami         = Id univoco Professionista in Visitami
* IdPrestazione      = Id univoco della Prestazione in Visitami
* PrimaDisponibilita = classe che esprime la prima disponiblità del professionista:
  * IdIday           = Id del giorno di prima disponibilità
  * IdHour           = Id dell'ora di prima disponibilità
  * Data             = Data della prima disponibilità
  * Ora              = Ora della prima disponibilità
  * Luogo            = Descrizione del luogo dell'appuntamento
  * Indirizzo        = Indirizzo per esteso del luogo dell'appuntamento
  * IdLuogo          = Id univoco del luogo
  * Tariffa          = Tariffa prevista per la prestazione. Le tariffe sono espresse in € e possono essere variabili (per es. 50-70€)
  * TariffaScontata  = Se presente uno sconto, rappresenta il prezzo che pagherà il paziente
  * Sconto           = Label che rappresenta lo Sconto % eventualmente applicato
  * ScontoD          = Valore decimale dello sconto applicato. 0=nessuno sconto
  * Distanza         = Distanza del luogo dal centro di ricerca individuato da Lat e Lng
 
# Ricerca dei Professionisti

E' possibile invocare questa API specificando le coordinate GPS anzichè la città

    Public Function APIGEN_DoSearch(ByVal idcate As Integer, ByVal lat As String, ByVal lng As String, ByRef lErr As String) As String
    
* idcate     = Id univoco Categoria Prestazioni
* lat        = stringa della latitudine (usare ".", es. "45.12312312")
* lng        = stringa della longitudine (usare ".", es. "9.12312312")
* lErr (out) = parametro in uscita valorizzato con l'eventuale errore se il metodo ritorna emtpystring

La chiamata produrrà il JSON della chiamata <b>Ricercare Professionisti (Semplice)</b>

# Creazione Utente (Paziente)

Questo metodo serve per creare una corrispondenza tra utente dell'applicazione chiamante ed utente (paziente) di Visitami. Questo metodo restituisce il <b>Token User</b> univoco del paziente su Visitami e dovrà essere salvato nell'applicazione richiedente (<b>legame tra applicazione richiedente e Visitami</b>) e usato ove richiesto nei successivi metodi.

Se viene riconosciuta una corrispondenza tra dati passati e dati già presenti su Visitami, non verrà creata una nuova posizione, ma verrà restituita la posizione trovata.

NB: il passaggio di dati da applicazione richiedente a Visitami prevede una gestione della titolarità dei dati passati.
In particolare: un Utente non presente in Visitami verrà marcato su Visitami come proveniente da applicazione richiedente a cui spetta la titolarità dei dati. Una richiesta di cancellazione su applicazione richiedente implica una cancellazione dei dati su Visitami secondo quanto regolato dalla Privacy Policy indicata all'indirizzo https://www.visitamiapp.com/note-legali. Per un utente in queste condizioni che acceda indipendentemente a Visitami, accettettando esplicitamente la suddetta privacy policy, verrà predisposto un aggiornamento della marcatura a "Cogestito", ossia presente in entrambe le piattaforme; un Utente già presente in Visitami verrà marcato come "Cogestito" in termini di titolarità. Una cancellazione su applicazione richiedente implicherà la cancellazione del <b>legame tra applicazione richiedente e Visitami</b> e il relativo aggiornamento della marcatura a "Solo Visitami", ma non la cancellazione su Visitami.

     Public Function APIGEN_NewUser(ByVal nome As String, ByVal cognome As String, ByVal email As String, ByVal phone As String, ByRef lErr As String) As String
    
* nome         = Nome del Paziente
* cognome      = Cognome del Paziente
* email        = Email del Paziente (campo chiave su cui viene ricercata la presenza in Visitami)
* phone        = Telefono del Paziente (opzionale, verrà comunque richiesto prima della prenotazione su Widget Dinamico). Può essere usato per ricercare la presenza in Visitami.
* lErr (out)   = parametro in uscita valorizzato con l'eventuale errore se il metodo ritorna emtpystring

La chiamata produrrà un JSON contenente il Token di accesso:

     {"usertoken":"<TOKEN USER>"}

# Modifica Utente (Paziente)

Questo metodo aggiorna un utente su Visitami.

    Public Function APIGEN_UpdUser(ByVal usertoken As String, ByVal nome As String, ByVal cognome As String, ByVal phone As String, ByRef lErr As String) As String
   
* usertoken     = Token Paziente
* nome          = Nome del Paziente
* cognome       = Cognome del Paziente
* phone         = Telefono del Paziente
* lErr (out)    = parametro in uscita valorizzato con l'eventuale errore se il metodo ritorna emtpystring

La chiamata produrrà un JSON che indica il successo dell'operazione:
     
     {"status":"success"}

* status     = success, o la chiamata non ha restituito nulla 

# Eliminazione Utente (Paziente) o eliminazione Legame

Questo metodo serve per eliminare il paziente su Visitami o il relativo <b>Legame</b> con l'applicazione richiedente

    Public Function APIGEN_DeleteUser(ByVal usertoken As String, ByRef lErr As String) As String

* usertoken     = Token Paziente
* lErr (out)    = parametro in uscita valorizzato con l'eventuale errore se il metodo ritorna emtpystring

La chiamata produrrà un JSON che indica il successo dell'operazione:

     {"status":"success","type":"<TIPO ELIMINAZIONE>"}

* status     = success, o la chiamata non ha restituito nulla 
* type       = può assumere: TOTALE/PARZIALE asseconda della marcatura dell'utente su Visitami.


