# Sistema modulare in Java9+
## Premessa
Nel 2017, dopo aver subito N ritardi partendo dal 2015, viene rilasciato ufficialmente il Java9.
Con questo rilascio il Jdk diventa modulare e Oracle decide di dare una spinta all'evoluzione del linguaggio, 
dichiarando 2 rilasci major nel corso di 1 anno.
Per scaricare Jdk9, seguire http://jdk.java.net/archive/ o http://jdk.java.net/9.
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
WIP
