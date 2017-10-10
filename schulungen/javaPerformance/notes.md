# Fragen
- Ist das Iterieren über Arrays wirklich schneller als über Listen? Und wenn ja: unter welchen Rahmenbedingungen (Anzahl der E., Größe der Elemente)
    - kommt auf die Zugrunde liegende Datenstruktur drauf an (genaue Implementierung der Liste)
    - ArrayList verwenden für die Iteration, dann ist es fast so schnell wie ein normales Array
    - Eine LinkedList ist bei sehr großen Datenmengen langsamer beim iterieren. Siehe hierzu den Abschnitt Collections
- Wann sollte ich String.intern() verwenden? Und was macht es für einen Unterschied, wenn die JVM zur Laufzeit sowieso fast alle kurzen Strings in den Stringpool packt.
    - Strings die bspw. aus der DB kommen, werden nicht aus dem StringPool gezogen. 
    - mit .intern() sagt man der JVM, dass er doch bitte in dem Pool schauen soll
    - --> somit ist ein `String bla = "Hallo"` gleich einem "Hallo"-Objekt, das aus der DB geladen wird und es werden nicht zwei gleiche Stringobjekte angelegt

# Tag1 


## Einführung
- Hotspots optimieren, alles andere führt zu nichts

### VM Basics
- es gibt viele VMs
- wenn der -client oder -server Parameter fehlt, dann entscheidet die VM selbst in welchem Modus sie gestartet wird 
    - haben wir das im Webstart?
- -XX:+PrintCompilation wirft aus, was der JIT Compiler optimiert --> ist nur ein Hinweis, daran kann nichts geändert werden
- -XX:+UnlockDiagnosticVMOptions -XX:+PrintInlining gibt aus, wann der JIT Compiler etwas inlined 

## Programmiertechniken
- Microbenchmarking (isolierte Test und Messungen) vs Profiling (Messungen im laufenden Betrieb mit Echtdaten) zur Messung von Performance
    - Microbenchmarking gefährlich (siehe Folie Seite 18)
    - Sinnvoll? Meistens nicht, stattdessen sauber Programmieren, JIT Compiler wird guten Code selbst optimieren
    - Falls doch: Warmlaufen lassen (JIT Compiler tut nichts unter 10.000 Aufrufen), Durchschnitt über mehrere Aufrufe der Zielmethode ziehen
  
### Collections
- List
    - ArrayList vorher festlegen wie groß sie werden soll (vergrößern kostet Zeit und Platz!)
        - get(x) ist O(1)
        - Einfügen von Werten an bestimmten Stellen ist teuer! O(n)
    - LinkedList 
        - Liste mit Zeigern auf das nächste Element
        - dadurch sind die einzelnen Elemente größer, da der Zeiger noch unter kommen muss
        - Einfügen von Werten an bestimmten Stellen ist sehr günstig O(1)
        - get(x) hat einen Aufwand von 0(n)
- HashSet
    - drauf achten, dass die HashCode-Methode sauber implementiert ist, da sonst alles in einem HashBucket landet und dann hat man die Performance bspw. einer LinkedList hat
    
### Multithreading
- mit Hilfe von ReentrantLocks ist es möglich sehr flexibel Thread-Locks zu verwenden!

### Immutable Objects
- kein synchronized notwendig, da sie nicht veränderbar sind
- alle Felder und die Klasse selbst sind private und final
- **Verwende LocalDate! Es ist massiv schneller als die alte API.**

## Garbage Collector
- jedes Objekt hat einen ReferenceCounter, solange dieser nicht 0 ist, bleibt das Objekt bestehen. Ist dieser 0, so kann dieses Objekt gelöscht werden
    - Nachteil: Speicherinseln; Objekte die sich gegenseitig referenzieren werden niemals abgeräumt
    - daher wird es nicht gemacht
- Mark&Sweep:
    - von einem "root"-Objekt werden alle referenzen abgegrast
    - Objekte die dadurch nicht markiert werden, kommen weg
    - Nachteil: funktioniert nur als Stop the World
- Copy Algorithmen:
    - zwei identisch große SPeicherbereiche
    - verwendete Objekte werden markiert und in den zweiten Speicherbereich verschoben
    - dadurch automatische Defragmentierung
    - nicht markierte Objekte werden dann aus dem 1. Speicherplatz gelöscht 
    - anschließend der gleiche Algorithmus in die andere Richtung
    - Nachteil: Kopieren kostet Zeit, Referenzen müssen geändert werden, der Speicher muss durch 2 geteilt werden
    - --> lohnt sich nur, wenn viele Objekte sterben
- Generational Collection
    - zwei Bereiche: old & young
        - young verwendet Copy-Algorithmen, da hier viele Objekte schnell sterben
        - old Generation: Mark-Algorithmen, da diese Objekte länger leben
    - oldGeneration am ModeShape-Server sollte recht klein sein

### Optimierungsziele
- Minimale Pausenzeiten, Maximaler Durchsatz und minimaler Memory Footprint; letzten beiden schließen sich aus
- bei vielen Reallocationen zur Startzeit sollte der Xms-Parameter auf das erste "Plateau" gesetzt werden
    - dadurch muss der GC zur Startzeit nicht so viel arbeiten
    - während der Optimierung Ober- und Untergrenze auf den gleichen Wert stellen --> dadurch sind ander Einstellungen deutlich besser zu erkennen
- -XX:+UserAdaptiveSizePolicy
    - sagt der JVM, dass sie die Größen der einzelnen Generationsbereiche selbst einstellen soll
    - man muss ihr lediglich die maximalen Speichergrenzen mitteilen
    - dafür muss mit -XX:MaxGCPauseMillis angegeben, wie selten der GC laufen darf
    - mit XX:GCTimeRatio gibt man einen prozentualen Wert an, an dem der GC laufen darf

### G1 Garbage Collector (Garbage First)
- concurrent Mark&weep Collector
- Ausgelegt für Heap Größen >= 6GB
- ab Java 9 Standard GC
- versucht in einem bestimmten Zeitraum so viel Speicher wie möglich freizuschaufeln
- Umsteigen, wenn ich eine bestehende Anwendung mit CMS/Parallel Old GC und vielen Pausenzeiten habe

#Tag 2

## Memory Leaks in Java
- inner classes
    - haben immer eine Referenz zur äußeren Klasse --> dadurch kann auch die äußere nicht abgeräumt werden, wenn es noch eine Referenz zur inneren Klasse gibt
- durch Collections
    - wenn eine Collection statisch ist, dann werden auch **nie** die Elemente dieser abgeräumt
    - innerhalb von Containern (Beans, @Component). Auch hier werden die Collections niemals geleert. Dabei ist es egal, ob die Collection private ist oder nicht
- können mit Hilfe von VisualVM ermittelt werden
    - Rechtsklick auf dem Prozess --> HeapDump
    - das ganze nochmal ein wenig später und die HeapDumps vergleichen
    - per Doppelklick auf die problematischen Objekten kann sich angezeigt werden, wo diese gehalten werden

## Weak, Soft, Phantom References
- Soft für Caching, Weak für schnelles Aufräumen, Phantom für sicheres Aufräumen
- soft:
    - Referenzen werden freigegeben, wenn VM Speicher benötigt
    - Einsatz:
        - quick-and-dirty caching (besser jedoch fertige Caches verwenden)
        - in Umgebungen mit wenig Speicherressourcen
- weak
    - werden beim nächsten GC-Lauf abgeräumt
    - Einsatz:
        - **nicht für Caches**, da Lebenszyklus viel zu kurz
- phantom
    - ermöglicht die Benachrichtigung, dass ein Objekt freigegeben wurde (Referenz auf ein Objekt, das gar nicht mehr da ist)
    - Einsatz:
        - Native Resourcenverwaltung (IO, Speicher, ...)

## Parallelverarbeitung und Multicore Optimierungen
- parallele Streams verwenden aktuell den Fork Join Pool
    - es kann über einen Parameter definiert werden, wie viele Number of Workers dieser haben darf
-  parallel()
    - best
        - ArrayLists
        - HashMaps
        - plain Arrays
    - worst
        - LinkedList
        - BlockingQueues
        - IO-Zeugs
    - best pracitce
        - When you have a lot of data or very costly per-element calculation
- Probleme:
    - ein normaler Tomcat kann maximal 100 User auf einmal bedienen, da jeder HTTP-Request einen Thread bindet
    - Abhilfe bietet hier Spring 5 mit ihrer Flux/Mono-API, jedoch nur, wenn auch die DB asynchrone Aufrufe unterstützt

## Monitoring und Tools
- Java Melody
    - vor allem für Webanwendungen
    - zeigt auch, wie viele HTTP sessions aktiv waren
    - overhead ist so gering, dass es überall mit laufen kann
        - Java VisualVM braucht ca 5% d. Ressourcen
- JMeter
    - für Lasttests
- typische Lasttestfehler
    - Kaltstarts ohne Aufwärmphase
    - Clients kommen an ihre Grenze
    - unrealistische Usecases
    - Lasttest wird nicht durchgeführt, bis die Anwendung auseinanderfliegt. Somit weiß man nicht was sie eigentlich aushält