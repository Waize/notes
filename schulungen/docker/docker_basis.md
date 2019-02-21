# Tag 1

Fragen: 
- Config-Dateien von außen in die Container reingeben?

## Kubernetes Einführung

### Docker Basics
- Isolation Level niedriger als VM
  - CPU, Memory und FS ist isoliert
  - trotzdem shared resources: kernel und disk buffers
    - Plattenintensive Container sollten nicht zusammen auf einem Cluster laufen, da sie sich sonst gegenseitig den Cache zertrampeln --> beide Dienste werden langsam
- Empfehlung: keine Alpine-base-images, lieber SLES oder Debian stripped verwenden, wenn das eh die Basis ist
  - somit hat man ein System, das einem bekannt ist
- networking
  - default NAT
  - alternativ Host-Based

### Stärken/Schwächen
- definierte Runtime-Umgebungen
- einfache Bereitstellung vieler Umgebungen
- cross-compiling
- erzwingt immutable infrastructure
- Schwächen
  - große Artefakte
  - manche Dinge schwierig/unmöglich
    - während des Buildprozesses auf gesicherte Repos zugreifen
    - 
  - volume-handling schlecht
  
### Docker im Betrieb
- Netz innen vs. Netz außen
  - Host-Based bringt Port-Konflikte
  - Sonst: NAT + Port-Konflikte
  - Service-Discovery problematisch

### Imagebau
- Analyse eines erstellen Dockerimages: https://github.com/wagoodman/dive







