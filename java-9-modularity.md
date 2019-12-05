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
## Linking modules
Con Java9 viene introdotta una nuova fase di sviluppo - linking.  
Il tool di riferimento e' `jlink`.  
Linking e' un modulo facoltativo intermedio tra il compilatore e runtime, creato per i moduli in modo da eseguire i collegamenti tra i moduli in modo opportuno prima di eseguire la nostra app a runtime.  
Linking serve a:
* creare custom runtime image (viene eseguita l'analisi di tutti file `module-info` determinando tutti i moduli che servono all'app per essere eseguita in modalita' stand alone), e' l'immagine con tutti i moduli necessari (sia custom che della piattaforma java). 
* l'immagine e' ridotta confronto precedente app+jdk, miglioramento delle performance
* ottimizzazione dell'intero programma, eliminando per esempio "il codice morto" non usato da nessuna parte del programma (quindi non viene caricato a runtime)  

E' stato creato il tool `jlink` usato per creare immagini di runtime custom.  
E' un tool plugin based, e' possibile estenderlo implementando nuove ottimizzazioni.  
es. eseguendo jlink passando il modulo `mymodule.cli`, `jlink` scansiona il suo descrittore, individua il modulo `mymodule.analysis` + il modulo referenziato implicitamente, `java.base`, e crea l'immagine di runtime che contiene solo questi tre moduli - ottimizzazione spazio disco e tempo di startup dell'app (jvm deve caricare solo questi tre moduli e non tutto il JDK come prima).  

## Preparazione progetto x Java9
Il concetto di classpath rimane.  
Utilizzo dei moduli introdotti in Java9 rimane facoltativo, possiamo non creare `module-info.java`, consideriamo pero i benefici che otteniamo usandoli.  
Se non usiamo i moduli, compilazione avviene in questo modo
		`javac -cp $CLASSPATH ...`
		`java -cp $CLASSPATH ...`
Se non usiamo i moduli, Java9 comunque, dietro le quinte, usa il sistema modulare:  
* per poter compilare con versioni nuove di java, senza usare i moduli, lanciamo i comandi standard 
* java crea per la nostra app un modulo di default chiamato `unnamed` - modulo contenente tutto il classpath
* modulo `unnamed` esporta tutto e puo' leggere tutti i moduli di jdk
* se non usiamo i moduli, con Java9 cmq JDK e' stato diviso in moduli e li stiamo usando in modo trasparente per nostro codice, praticamente tutto il nostro classpath finisce nel modulo `unnamed`  

Durante la migrazione a Java9 dobbiamo verificare che non stiamo utilizzando i tipi che sono stati incapsulati e non sono piu' accessibili dall'esterno. Per verificare incongruenze nel nostro codice con nuove versioni java possiamo usare il tool `jdeps` introdotto in Java8.  
Se non siamo in grado di cambiare il codice sorgente (es. usiamo librerie di terzi che non hanno nuove versioni per java9), c'e' il seguente workaround:
* nella fase di compilazione scriviamo `javac --add-exports java.base/sun.security.x509=ALL-UNNAMED Main.java`
* nella fase di esecuzione scriviamo `java --add-exports java.base/sun.security.x509=ALL-UNNAMED Main`

Se usiamo i tipi del modulo JavaSE che non sono quelli di default (es. i tipi che con Java9 non sono piu' disponibili di default, essendoci migrati nei moduli nuovi):
* sono i tipi che sono fuori al modulo java.se (es. i tipi spostati nel modulo java.xml.ws, e altri moduli che sono piu' enterprise edition)
* la soluzione e' usare il flag `--add-modules`, es. `javac --add-modules java.xml.bind Main.java`
* idem per esecuzione `java --add-modules java.xml.bind Main`  

E' sempre meglio usare jdeps per verificare questo tipo di errori
