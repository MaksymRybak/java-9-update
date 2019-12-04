# Sistema modulare in Java9+
## Premessa
Nel 2017, dopo aver subito N ritardi partendo dal 2015, viene rilasciato ufficialmente il Java9.
Con questo rilascio il Jdk diventa modulare e Oracle decide di dare una spinta all'evoluzione del linguaggio, 
dichiarando 2 rilasci major nel corso di 1 anno.  
Per scaricare Jdk9, seguire http://jdk.java.net/archive/ o http://jdk.java.net/9.  
Per vedere le demo del materiale riportato sotto, seguire il link [java-9-modularity-demo](https://github.com/MaksymRybak/demo-java-modularity).
## Obiettivi dell'introduzione del sistema modulare
* Uno degli obiettivi principali e' la semplificazione del JDK che nel corso degli anni praticamente e' esploso (conseguenza del requisito di retrocompatibilita' che Java voleva mantenere nel corso degli ultimi 20 anni, quindi, niente eliminazione del codice sorgente legacy/obsoleto). 
* Miglioramento della manutenzione di JDK nel corso di tempo
* Isolamento migliore del codice rafforzando i requisiti di sicurezza
* Le dipendenze tra i moduli sono specificate in modo esplicito e piu' chiaro, miglioramento dei tempi di caricamento in memoria (viene caricato solo quello che ci serve a runtime e non l'intero rt.jar come avveniva con le versioni precedenti)
## Caratteristiche di un modulo 
Un modulo ha il nome ed e' auto contenuto. Viene usato x raggruppare il codice del suo dominio (scope). 
Definisce in modo esplicito quello che vuole rendere pubblico, accessibile dall'esterno (es. API, classi public). 
Per descrivere il modulo viene usato il module-info.java.
Di default i packages creati a livello del modulo non sono esportati, usiamo exports x renderli visibili all'esterno, requires x referenziare altri moduli (nostre dipendenze). Nella sezione requires specifichiamo il nome del Modulo. Nella sezione exports, il nome del package. Per stampare il descrittore del modulo: `java --list-modules`, `java --describe-module [nome]`.  
## Panoramica del sistema modulare
Sistema modulare consente organizzare meglio le relazioni tra vari moduli (packages).  
Al posto di class-path viene usato il termine module-path. 
L'introduzione del sistema modulare ha comportato alla suddivisione del vecchio modulo rt.jar in piu' moduli (es. java.base, java.sql, java.logging etc.)
Sistema modulare si basa su tre pilastri
* incapsulamento 
* definizione di una interfaccia chiara
* esplicita dichiarazione di moduli dai quali dipendiamo e i packages che esportiamo 

Il modulo viene dichiarato nel file module-info.java, definendo due sezioni
1. defiinizione di packages che vogliamo rendere visibili all'esterno (interfaccia del nostro modulo)
2. definizione di moduli dai quali dipendiamo 

Comandi utili:
* compilazione del modulo `javac --module-source-path src -d out .\src\mymodule\my\package\Main.java`
* esecuzione del main: `java --module-path out -m mymodule\my.package.Main`
## Creazione di piu' moduli
Nel momento di avvio del'app viene controllato se tutti i moduli da cui dipendiamo sono presenti.  
Viene analizzato il `module-info.class` (descrittore del modulo) per verificare se abbiamo tutte le dipendenze richieste a runtime (NOTA: il modulo java.base e' referenziato da tutti i moduli in modo implicito).  
Il processo di accesso ai moduli esterni:
* il modulo che importa un'altro modulo ha accesso solo ai tipi public esportati (i tipi public non esportati esplicitamente non possano essere acceduti da altri moduli)
* prima di usare le classi di un'altro modulo dobbiamo porsi tre domande:  
1. stiamo importando il modulo di interesse?
2. Il modulo di interesse esporta i package richiesti?
3. I tipi nei packages esportati hanno la visibilita' public?  

NOTA: i tipi private e protected NON sono visibili al di fuori del modulo  
I moduli non supportano la visibilita' transitiva, cioe', se il modulo X legge il modulo Y che a sua volta legge il modulo Z, il modulo X non puo' accedere al modulo Z, senza una importazione esplicita a livello di module-info.java.  
#### Transitivita'
In certi casi viene comodo avere l'accesso transitivo, per esempio `java.sql` dipende da `java.logging`, e un metodo public di `java.sql` ritorna il tipo di `java.logging`. Se la nostra app usa tale metodo dovrebbe in teoria definire il require anche del modulo `java.logging`. In alternativa, se modulo `java.sql` esegue l'import di `java.logging` come `require transitive`, anche la nostra app che esegue `require java.sql` ha accesso diretto a tutti i tipi public di `java.logging`.  
Possiamo creare anche cosi detti moduli AGGREGATORI, dove viene creato un modulo senza codice con solo il module-info.java contenente `require transitive` di moduli concreti. Questo semplifica organizzazione delle nostre librerie e esposizione all'esterno delle funzionalita'.
## Concetto di servizi
Il concetto di servizi aiuta a esporre i servizi e non singoli package, e' un ragruppamento in modo da non esporre fuori tanti package.
I servizi aiutano a rendere il nostro modulo estendibile piu' facilmente.  
Servizi nel sistema modulare di java:
* viene introdotto un layer in piu' tra il service consumer e service provider - service catalog/registry
* service provider registra l'interfaccia e implementazione nel registry
* service consumer richiede al service registry l'implementazione di una specifica interfaccia  
NOTA: e' il service registry che crea una nuova istanza del servizio richiesto.  

A livello di module descriptor (`module-info.java`) abbiamo tre moduli "in gioco"
1. modulo che contiene l'interfaccia del servizio - interfaccia esportata nel descrittore  
2. modulo che contiene il provider, chi implementa l'interfaccia - importa l'interfaccia, implementa interfaccia, esporta l'implementazione usando la sintassi `provides Impl with Interface` - NOTA: implementazione e' visibile solo al service registry, nessun altro modulo puo' richiedere direttamente l'implementazione  
3. modulo che contiene il consumer, chi usa l'implementazione dell'interfaccia - richiede l'interfaccia, richiede l'imlpementazione dell'interfaccia al service registry usando il comando `uses <interface package>`, il codice del consumatore richiede al service registry l'istanza del servizio, es. `Iterable<MyService> services = ServiceLoader.load(MyService.class)`.  

Il concetto di servizio disacoppia il consumatore e produttore.  
Avendo la dipendenza dall'interfaccia, il consumer non va in errore se per qualche motivo non trova l'implementazione (es. il provider non ha eseguito la registrazione nel service registry).

  
