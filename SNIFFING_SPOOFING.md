## Sniffing e Spoofing

Per effettare uno sniffing si può utilizzare python tramite la libreria `scapy`:

``` python
    from scapy.all import *
    a=IP()
    a.show()
```

Tramite il metodo `sniff` è possibile sniffare i pacchetti proveniente da una determinata interfaccia e si possomno anche specificare dei filtri.

```python
    def print(pkt):
        pkt.show()

    pkt = sniff(iface='br-3bd174d26eed',filter='(icmp) or (tcp and dst host 10.9.0.5 
    and port 23)',prn=print)
```

Tramite questo codice ad esempio si può sniffare dall'interfaccia `br-3bd174d26eed` e filtrare (quindi visualizzare) solo i pacchetti: o di tipo `icmp`, o di tipo `tcp` dalla destinazione `10.9.0.5` e con porta `23`.

Scapy da, inoltre, la possibilità di inviare pachetti che possono essere costruiti ad-hoc.

```python
ip = IP()
ip.dst = '10.0.0.1'
icmp = ip/ICMP()
sendp(icmp)
```

Tramite questo codice, ad esempio, è possibile creare un pacchetto `IP` e impostare una destinzione per poi incapsulare tramite `/` il tipo (`icmp`) per poi inviare il tutto. Oltre a ciò un'altra cosa che `scapy` permette di fare è modificare deliberatamente la source del pacchetto da inviare permettendo così di effettuare numerosi attacchi come lo `smurf attack`. Un'altra possibilità è quella di settare il `time to live` del pacchetto che se adeguatamente settato permette di creare il proprio `traceroute`.
Questo è un esempio di `traceroute` personale creato con scapy:

```python
from scapy.all import *

ttl=1
while True:
    a = IP()
    a.dst = '192.168.178.29'
    a.ttl = ttl
    p=sr1(a/ICMP())
    if p.len==56:
        print("router ip: "+ p.src)
        print("ttl: "+str(ttl))
        ttl+=1
    else:
        print(p.src)
        print("ttl: "+str(ttl))
        break
```

Scapy da la possibilità anche di fare spoofing ed un esempio di come spoofare un icmp inviato tra due ip della stessa lan è:

```python
from scapy.all import *

def print_icmp_type(pkt):
    if ICMP in pkt and pkt[ICMP].type == 8:
        new = IP()
        new.src=pkt[IP].dst
        new.dst=pkt[IP].src
        new = new/ICMP()/"ciao"
        new[ICMP].type=0
        send(new)

pkt = sniff(iface='br-3bd174d26eed',filter='icmp',prn=print_icmp_type)
```