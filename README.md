# Note molto sintetiche che riguardano le novita' introdotte in Java9
## Un po' di storia prima di cominciare
* 	2006: Java6, Scripting API, ultimo rilascio di SUN Microsystem  
* 	2011: Java7, primo rilascio di Oracle, bytecode x il linguaggio dinamico, piccole modifiche al linguaggio  
* 	2014: Java8, lambda e stream  
* 	2017: Java9, dopo aver subito N ritardi partendo dal 2015 e' un JDK modulare, dipendenze esplicite. Dopo questo rilascio Oracle decide di fare 2 rilasci major in 1 anno, senza i legami alle implementazioni di funzionalita' specifiche x scaricare jdk9, seguire http://jdk.java.net/archive/ o http://jdk.java.net/9.
* 	OpenJDK - è una implementazione libera della piattaforma Java, edizione standard, è l'implementazione di riferimento ufficiale di Java SE dalla versione 7.  
## JDK modulare  
impatti riguardano:  
- linguaggio  
- compilatore  
- jvm (carica dati dei moduli salvati nel formato binario)  
- tools (IDEs, libs)  

una delle modifiche piu' grosse e' l'introduzione del sistema modulare  
il motivo perche' e' stato introdotto:  

*  JDK originale (rt.jar) aveva gia' 20 anni con dipendenze tecnicamente molto profonde  
*  x migliorare la manutenzione di JDK (nel corso di tempo e' esploso, nessuna rimozione di codice x mantenere la retrocompatibilita')  
*  isolamento migliore del codice, miglioramento di sicurezza  
Java8 e prima: 
*  JVM carica tutti file JAR in una singolo e piatto classpath
*  classpath molto fragile, non abbiamo la possibilita' di negare ad una classe di accedere ad un'altra a runtime  
*  file JAR raggruppano i file, ma a runtime, una classe di un JAR puo' accedere a qualsiasi classe di altri JAR presenti nel classpath  

con Java9 e sistema modulare vengono rilasciati > 90 moduli (es. java.base, java.logging, java.xml, jdk.httpserver, java.sql, java.prefs, etc...) vedi lo schemino in basso

* i moduli java.*, fanno parte della specifica JSE  
* i moduli jdk.httpserver, jdk.unsupported sono specifici x JDK  
* il modulo jdk.incubator contiene nuove funzionalita' rilasciate in beta, possiamo testarle prima del rilascio ufficiale  
* niente piu' dipendenze circolari  
* referenziamo solo i moduli strettamente necessari, questo diminuisce drasticamente il tempo di scansione delle classi quando devono essere caricate in memoria  

caratteristiche di un modulo  
* ha il nome  
* e' auto contenuto  
* raggruppa il codice relativo  
* ha una sezione esposta all'esterno ed una interna  
* file module-info.java definisce il descrittore del modulo  
* 	di default i packages usati nel modulo non sono esportati  
* 	usiamo requires x referenziare altri moduli  
* 	NOTA: requires specifica il nome del Modulo, invece exports, il nome del package  
* 	cmd x stampare il descrittore del modulo: java --list-modules, java --describe-module [nome]  

Per un approfondimento piu' dettagliato fate riferimento a [java-9-modularity](https://github.com/MaksymRybak/java-9-update/blob/master/java-9-modularity.md)

## Migrazione di una app classpath based  
la compilazione non cambia se:  
1. la nostra app NON usa i tipi di JDK che sono stati incapsulati in Java9  
1. sono usati i tipi delle librerie "non default di Java SE modules"  

1) quando il nostro classpath viene caricato, con Java9, viene cmq costruito il modulo 'unnamed' (anche se noi non abbiamo definito nessun modulo e relativo descrittore)  
unnamed module e' un modulo speciale in JVM, acquisisce automaticamente le dipendenze dalla JDK modulare, per far funzionare codice vecchio in Java9, hanno applicato 2 compromessi:  
*  il vecchio codice puo' continuare a referenziare i tipi incapsulati in Java9, ricevendo un warning nel momento di compilazione con Java8. Da aggiungere questo punto al debito tecnico, che deve essere risolto nel tempo, per non avere i problemi futuri nell'aggiornamento di Java, JDK, librerie terzi.  
Recap: il codice compilato con Java8 puo' girare con Java9  
*  Invece se compiliamo il nostro codice con Java9, l'errore e' bloccante, qui non possiamo continuare e siamo obbligati ad adattarsi alla regola di moduli  
NOTA: x forzare la negazione di eseguire il codice compilato con Java8 (e che usa i tipi incapsulati) in Java9 , possiamo passare il parametro --illegal-access=deny al comando java  
(viene sollevata l'eccezione se si verifica questa casistica)  
Se questo problema ha una nostra libreria esterna, dobbiamo notificare il fornitore richiedendo la modifica / scarica nuova versione.  
Possiamo cmq aggiungere i parametri nel compilatore (javac) e runtime java, in modo di poter continuare ad usare i tipi incapsulati in Java9. (NON E' CONSIGLIATO)  
Jdeps - e' un nuovo tool introdotto per poter scansionare il nostro codice e determinare se da qualche parte usiamo i tipi incapsulati, in piu' abbiamo anche dei suggerimenti sui tipi che possiamo usare.  
2.  esempio, se prima usavamo java.xml.bind che era' in JSE, ma all'interno del package relativo a qualche estensione EE portata in JSE, con Java9 questo modulo non e' piu' all'interno di java.se ma e' stato spostato sotto java.se.ee -> questo fa fallire la compilazione con Java9, ma possiamo aggiungere il modulo mancante usando i parametri del compilatore  
es. javac --add-modules java.xml.bind Main.java, stessa cosa dobbiamo fare anche quando avviamo l'app, aggiungendo stesso modulo al runtime java  
Jdeps puo' essere usato anche in questo caso, per trovare i moduli spostati con Java9 e usati nella nostra app.  
Recap: con Java9 abbiamo 2 possibilita', continuare ad usare classpath, o adottare il sistema a moduli info sul sistema modulare: javamodularity.com  
## Piccole migliorie nel linguaggio e librerie SE in Java9
#### Collection factory methods
possiamo definire subito gli elementi della lista, prima con Java8 spesso usavamo Array.asList():  
* List.of(N elementi), ritorna la lista immutabile (Immutable Collection)  
* Set.of(N elementi), non possiamo aggiungere valori null, e valori duplicati  
NOTA: implementazione del metodo statico of() nel tipo List prevede >10 override, x poter passare entro 10 elementi nel momento di creazione della lista, senza allocazioni di memoria aggiuntiva (es. se API usa varargs es. E... elements, varargs sono passati al metodo creando un array, idem per la classe Set)  
* Map.of(), es. Map.of(key1, value1, key2, value2)  
* Map.ofEntries(Map.entry(key, value), Map.entry(key, value)... )  
NOTA: quando iteriamo elementi di una Map o Set, l'ordine con il quale ci vengono ritornati i valori NON e' garantito. Invece per List, e' garantito!  
#### Streams, miglioramenti  
recap as is: stream mette a disposizione vari metodi di elaborazione elementi, es. map(), filter(), forEach().  
con Java9 sono stati aggiunti i nuovi metodi:  
* takeWhile(predicate)
* dropWhile(predicate)
* static ofNullable()
* static iterate()
#### Collectors
recap as is: potevamo eseguire la creazione di una lista partendo da uno stream (es. Stream.of(1,2,3).collect(Collectors.toList()))  
avevamo Collectors.groupingBy(), x ragruppare elementi per una regola definita con lambda es. costruzione di Map<Integer, List<Integer>>  
in Java9 sono stati aggiunti 2 nuovi Collectors  
*  filtering (usato di solito con altri collector, es. groupingBy, x escludere degli elementi )  
*  flatMapping, x mappare uno stream in un'altro, specificando il collettore da usare (es. Set)  
NOTA: Collectors.toSet() rimuove automaticamente i dupplicati, Set puo' contenere solo valori univoci  
#### Optional API  
nato in Java8, migliorato in Java9  
Java8: Optional.of(), Optional.empty() - evitiamo null check e NullPointerException, Optional.orElse(), Optional.get() - da NON usare, puo' sollevare l'eccezione se il valore e' assente  
Java9 aggiunti
*  ifPresentOrElse(metodoSeAbbiamoValore, metodoSeNonAbbiamoValore)  
*  or() - gestisce il chaining/catena di chiamate,  
*  stream() - per convertire N stream in un singolo stream (es. usando flatMap())  
#### Piccole modifiche al linguaggio  
*  _: singolo _ non e' piu' possibile usare come identificatore della variabile (x non avere conflitti con lambda nelle versioni successive di java)  
*  try-with-resources: puo' accettare direttamente la variabile che referenzia la risorsa (senza creare un nuovo riferimento come avveniva in Java8)  
NOTA: tale variabile deve essere final / effettivamente final, nel senso che si puo' referenziare un parametro non final passato al metodo, e compilatore deduce automaticamente che e' una variabile final - NON possiamo cambiare il riferimento di tale variabile nel corso di try-with-resources 
*  miglioramento inferenza dei tipi in Generics: in java7/8 potevamo gia' usare diamond operator <>, es. ArrayList<String> arr = new ArrayList<>(); ma veniva sollevato un errore se usavamo una classe innestata anonima, e. new ArrayList<>() { @Override add(String str) { ... } }, in Java9 e' stata gestita questa casistica
*  metodi private nell'interfacce (nota: i metodi definiti in una interfaccia sono public di default): in Java8 e' stato aggiunto default, x implementare una logica di default in una interfaccia in Java9, si puo' usare private davanti a default, x rendere l'implementazione di default nascosta all'interno di interfaccia
## Altri miglioramenti
#### Javadoc
* supporta HTML5
* supporta ricerca
* supporta i Moduli
* es. https://docs.oracle.com/javase/9/docs/api
#### Localizzazione: 
*  supporto a unicode e' stato aggiornato dalla versione 6.2 alla 8.0, ora sono supportati > 10000 nuovi caratteri file di proprieta' (una lista di key=value, usato per es. per la traduzione di testi,label etc.) - prima la codifica era' ISO-8859-1, con Java9 e' stato fatto il passaggio a UTF-8
*  Common Local Date Repository (CLDR): i formati in base alla localizzazione, x mappare impostazioni locali
*  nuova API x DateTime (pachage java.time): nuovi metodi x la divisione della durata per durata standard(es. giorni, ore)
```
long duration(Duretion divisor)
trucatedTo(TemporalUnit unit)
Clock systemUTC() - ritorna il clock preciso!
LocalDate.datesUntil - crea stream di date
```
NOTA: non usare piu' java.util.date!
## HTTP/2 e Process new APIs in Java9
#### Process 
java.lang.Process - gestione processi, possiamo esplorare i processi non solo della JVM  
```
ProcessHandler.of(ID) - accesso a parent/descendants/destroy/info
ProcessHandler.Info - info di dettaglio di un processo (command, arguments, user, startInstant, totaleCpuDuration)
```
NOTA: possiamo avere accesso solo al processo creato all'interno di JVM, partendo dal processo chiamante (es. nostro codice) 
es. ProcessHandler.current().pid()
*  ESEMPIO1: possiamo recuperare i processi in esecuzione sulla macchina ProcessHandler.allProcesses() - ritorno lo stream che possiamo elaborare successivamente
*  ESEMPIO2: otteniamo il riferimento ad un processo, metodo onExit() ritorna CompletableFuture (introdotto in Java8), che puo' essere completata in modo assincrono nel futuro lambda x eseguire la logica nel momento di distruzione del processo 
NOTA: se vogliamo che il nostro main() aspetta l'esecuzione della future onExit, dobbiamo eseguire onExit().join() sul processo referenziato 
#### HttpClient
supporto a HTTP/2, Web Socket (in Java9, HttpClient e' nel modulo incubatore)  
ora possiamo usare questa versione evitando di referenziare altre librerie x eseguire chiamate HTTP  
```
send - invio sincrono
sendAsync - abbiamo completable future x gestire le resp in modo assincrono 
Builder - newHttpClient()
HttpRequest - lettura di uri, headers, method, newBuilder
HttpResponse - lettura di uri, statusCode, body
```
## Reactive Streams
standardizzato da http://www.reactive-streams.org/  
definizione interfacce e comportamento da adottare) nuovo concetto supportato - Backpressure, client che processa elementi di uno stream puo' notificare il fornitore di elementi di fermarsi finche' non invia il comando di procedere in JDK sono state aggiunte solo le interfacce (Flow API), rimane necessita' di usare le implementazioni di terzi  
NOTA: avendo le interfacce in JDK, abbiamo l'interoperabilita' tra diversi implementazioni  
#### Flow API 
interfacce x Reactive Streams in JDK
NOTA: non implementare queste interfacce in casa, NON e' banale, usare implementazioni di terzi
```
Publisher
Subscriber 
Subscriber - effettua subscribe su Publisher
Publisher - chiama onSubscribe sul Subscriber x ritornare la Subscription 
Subscriber - invia una request alla Subscription (possiamo specificare num di elementi di cui abbiamo bisogno) -> Publisher chiama onNext() su Subscriber -> Publisher chiama onComplete() su Subscriber x notificare che ha inviato tutti elementi 
```
Publisher puo' chiamare onError() x notificare Subscriber che si e' verificato un errore e non verra' piu' inviato nessun elemento  
Subscriber puo' chiamare cancel() sulla Subscription x terminare il flusso  
NOTA: abbiamo meccanismo event-driven per gestire lo scambio dei dati  
es. HttpClient implementa il Subscriber quando dobbiamo leggere il body della response
Chi ha annunciato lo supporto a Flow API (top implementation of Reactive Streams)  
* RxJava 2
* Spring 5
* Akka Streams
#### StackWalker
una nuova API aggiunta in Java9 x ottenere Stacktraces in modo piu' affidabile e performante 
StackWalker ha aggiunto
```
walk - ritorna lo stream di StackFrame (NOTA: qui possiamo eseguie i filtri che vogliamo)
forEach - per iterare streamd di StackFrame
```
ogni StackFrame rappresenta la chiamata di un metodo all'interno di una classe  
## Miglioramenti alle Prestazioni e Sicurezza in Java9
deprecato CMS (Concurrent Mark Sweap) collector, sostituito da G1  
#### new G1
garbage first introdotto in JDK6, con Java9 e' impostato come default  
Review di Generational Garbage Collection (usato in tutti GC compreso G1)  
Heap -> dove abbiamo i nostri oggetti, viene diviso in 3 regioni: 1) Eden (oggetti appena istanziati) -> 2) Survivor (oggetti istanziato da un periodo di tempo piu' lungo) -> 3) Tenured (oggetti di lunga durata) - questa divisione permette a GC di scansionare diverse regioni con frequenza differente (es. Eden viene scansionato piu' spesso, contiene oggetti non stabili, e di solito gli oggetti nascono e muoiano piu' spesso -> il problema di prima era legato alla pausa lunga in cui GC e' in esecuzione (JVM viene messa in "stand by")  
In caso di G1 abbiamo sempre Heap che contiene i riferimento ai nostri oggetti -> al posto di definire 3 regioni (come descritto prima), G1 definisce N regioni (ognuna grande circa 32MB) -> es. abbiamo 5 Eden, 2 Survivor, 2 Old -> le regioni come Survivor e Old aumentano con l'incremento di oggetti di lunga durata -> in questo caso abbiamo Incremental GC -> in questo modo GC puo' evitare di scansionare delle region che non servono -> GC puo' scansionare regoni differenti in parallelo -> G1 disegnano per grandi heap -> le pause piu' piccole, possiamo specificare i tempi di queste pause migliorando le performance -> piu' computazioni CPU , threads paralleli sono impostati in base al CPU -> la dimensione di una regione viene scelta in base a CPU -> impostazioni di GC per migliorare le performance (es. pause timeout, -XX:MaxGCPauseMillis=200 (200milisecs), -XX:+G1enableStringDeduplication - per allegerire il carico sulla memoria quando abbiamo molte elaborazioni sulle stringhe)  
NOTA: teniamo sempre sotto il controllo le prestazioni del GC
#### String performance
###### Compact Strings
x diminuire memoria utilizzata dalle stringhe, usate di default  
Prima di Java9: tipo String viene salvato in memoria come un array di caratteria char[], dove ogni carattere e' codificato in UTF-16 (2bytes, spesso 2bytes non ci servono) -> UTF-16 viene usato x supportare UNICODE -> Compact String risolvono questo problema - una Stringa in Java9 puo' essere salvata in byte[] array -> come fa classe String a capire che il suo contenuto deve essere interpretato come una stringa ASCII e non UNICODE -> abbiamo il flag byte a livello della classe, che puo' essere impostato a 0 - ISO-8859-1/Latin1 (ASCII) / 1 - Full Unicode (UTF-16) -> tutto questo e' gestito automaticamente, x devs e' trasparente 
###### Filter incoming serialization data
recap serializzazione dell'oggetto: Java Object -> ObjectOutputStream -> Raw Bytes  
recap deserializzazione: Raw bytes -> ObjectInputStream -> Java Object (Problemi di sicurezza!)  
In Java9 e' stata introdotta una nuova interfaccia x filtrare i dati prima di deserializzazione:
```
interface ObjectInputFilter { checkInput(FilterInput filterInput) }

```
x agganciare il filtro usiamo ObjectInputStream::setInputFilter (x oggetto)
x impostare il filtro senza modificare il codice usiamo il parametro jdk.serialFilter nella nostra JVM, proprieta' configurabili sono: maxbytes=n; maxarray=n; maxdepth=n;  
consentono limitare la dimensione di oggetti, per evitare attacchi di tipo Denial Of Service (quando oggetto deserializzato occupa piu' memoria disponibile) possiamo impostare anche i tipi che possano essere deserializzati / oppure NO.  
NOTA: questa funzionalita' e' stata portata anche in versioni 6,7,8 di Java!
#### Miglioramenti a TLS (Transport Layer Security)
usato in HTTPS  
aggiunta la feature ALPN - Application Layer Protocol Negotiation  
ci permette di selezionare il protocollo a livello applicativo durante la negoziazione TLS  
praticamente il client invia al server la richiesta con protocolli supportati, server risponde al client con protocollo scelto e contemporaneamente viene stabilita la connessione TLS.  
ALPN e' stata implementate x supportare HTTP/2 che e' un protocollo piu' sicuro e performante (dati binari scambiati tra client e server) un'altro miglioramento e' DTLS 1.0/1.2 - Datagrame Transport Layer Security  
DTLS e' applicato al networking senza una connessione TCP affidabile (aka usiamo UDP con un layer di sicurezza TLS)  
###### OCSP stapling 
stapling - graffettatrice
OCSP - Online Certificate Status Protocol  
NOTA: la sicurezza in TLS si basa sui certificati, il server presenta al client il certificato, il client verifica il certificato x capire se puo' dare la fiducia al server MA in piu' il client deve verificare se il certificato e' stato revocato o no (il ceritificato puo' essere revocato dal server stesso) OCSP supporta il check di X.509 certificate revocation, praticamente il client, durante l'avvio di una connessione TLS, richiede al server di fornire il proprio certificato con info relativa allo stato del certificato -> server gestisce questa richiesta facendo a sua volta un controllo dello stato del proprio certificato e invia entrambe le cose al client (stapled - graffettato) -> il client verifica la risposta in modo sicuro  
Server mantiene in cache i dati di OCSP, aggiornati periodicamente da un server OCSP  
In questo modo client diventa piu' leggero senza effettuare per ogni connessione TLS una verifica di OCSP  
Certificati SHA-1 sono stati disabilitati i certificati firmati con SHA-1 sono automaticamente rifiutati in Java9  
E' stato aggiunto supporto a SHA-3  
## Altre info
JEPs x esplorare le proposte fatte per migliorare Java  
JEP - Java Enhancement Proposal  


