# Fragen
- Ist das Iterieren über Arrays wirklich schneller als über Listen? Und wenn ja: unter welchen Rahmenbedingungen (Anzahl der E., Größe der Elemente)
    - kommt auf die Zugrunde liegende Datenstruktur drauf an (genaue Implementierung der Liste)
    - ArrayList verwenden für die Iteration, dann ist es fast so schnell wie ein normales Array

# Tag1 


## Einführung
- Hotspots optimieren, alles andere führt zu nichts

### VM Basics
- es gibt viele VMs
- wenn der -client oder -server Parameter fehlt, dann entscheidet die VM selbst in welchem Modus sie gestartet wird 
    - haben wir das im Webstart?
- -XX:+PrintCompilation wirft aus, was der JIT Compiler optimiert --> ist nur ein Hinweis, daran kann nichts geänder werden
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



