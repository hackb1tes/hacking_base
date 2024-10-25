# Teoria e pratica ethical hacking

## XSS

L'attacco `xss` è un attacco che permette di manipolare un sito web al fine di eseguire codice `javascript` nei client che visitano il sito per rubare informazioni o manifestare un comportamento sgradevole. Esistono 3 tipi di XSS:

* L'XSS stored
* L'XSS reflected
* Il DOM XSS

Nel caso dello `stored XSS` tramite un commento in un blog o in un sito è possibile aggiungere a proprio piacimento tag `HTML` per personalizzare il contenuto e questi tag vengono letti ed eseguiti in un ambiente non controllato, per sincerarsi della vulnerabilità è, ad esempio, possibile lasciare un commento con il tag `<strong> parola </strong>` e ricevere un output del tipo: **parola**. Questo tipo di comportamento identifica una vulnerabilità visto che il tag `<strong>` può essere sostituito con `<script>...</script>`, cioè un tag che permette l'esecuzione automatica ti codice `javascript`. Se questo tag viene messo in un commento pubblico ogni browser che scaricherà il contenuto poi eseguirà il codice nel computer della vittima, dando così la possibilità all'attaccante di ricevere: informazioni private dell'utente, accesso al computer della vittima o cookies di sessione.

Il `reflected XSS` è un tipo di attacco chepuò essere eseguito su l'`url` del sito vittima che non sanifica correttamente le `quary` inviate, questo url deve essere inviato alla vittima tramite una campagna di `phishing`. Un sito vulnerabile a `reflected xss` permette, per esempio, di insireri un tag `HTML` all'interno dell'url, come prima se ciò è permesso allora è anche possibile inserire `script javascript` da eseguire nel browser della vittima. Questa tecnica permette quindi di ricevere informazioni relative alla vittima o sessioni senza, però, passare il complicato processo di `rediretting` e creazioni di siti fake con tool come `Evilginx`, rendendo l'attacco meno palese.

Il `DOM based XSS` permette, come gli altri due, di eseguere codice nel browser dell'utente ma sfruttando vulnerabilità del `DOM` del sito che si vuole attaccare. Per eseguire un attaccco del genere l'attaccante necessita di:

* DOM `source`: cioé un oggetto capace di memorizzare stringhe
* DOM `sink`: cioé delle dom api capaci di eseguire il dom source.

Per esempio immaginiamo un sito che prende l'`hash` dall'url e cerca le corrispondenze con ciò che ha già salvato nel server, come in questo caso:

``` javascript
const hash = document.location.hash;
const founds = [];
const nMatches = findNumberOfMatches(founds, hash);
document.write(nMatches + 'matches found for'+ hash);
```

Possiamo considerare `const hash = document.location.hash;` come il nostro `source` e document.`write(nMatches + 'matches found for'+ hash);` come il nostro `sink`. Se l'attaccante mettesse del codice `javascript` in un tag `HTML` hashandolo, allora il server eseguirebbe quel codice.

## SQL Injection

L'`SQL Injection` permette all'attaccante di manipolare il database di un sito web utilizzando una query non sicura (mescolando dati e codice). Un classico esempio di SQL Injection è `' or 1=1'` inserito nel campo `username` o `password` per superare la fase di autenticazione senza conoscere le credenziali di nessun utente; un'alternativa comune potrebbe essere quella di bypassare il controllo inserendo il carattere di commento dopo il campo desiderato (`#,-,/**/`).
Una contromisura potrebbe essere distinguere il codice da i dati inseriti utilizzando, per esempio, PHP e creare così le `quary parametrizzate` (prepared statement), ovviamente un'altra contromisura è sicuramente sanificare i campi impedendo di inviare caratteri speciali come: ', /, *, # ecc. Ovviamente ciò non è applicabile al campoo delle password.

Per applicare delle quary parametrizzate, quindi, basta passare da:

```PHP
$conn = new mysqli ("localhost", "root", "seedubuntu", "dbtest");
$sql = "SELECT Name, Salary, SSN
    FROM employee
    WHERE eid= '$eid' and password=' $pwd'";
$conn->query($sql);
$result = $conn->query($sql);
```

a

```PHP
$conn = new mysqli ("localhost", "root", "seedubuntu", "dbtest");
$sql = "SELECT Name, Salary, SSN
    FROM employee
    WHERE eid= ? and password=?";
if ($stmt = $conn->prepare ($sql)) {
    $stmt->bind_param ("ss", $eid, $pwd);
    $stmt->execute();
    $stmt->bind_result ($name, $salary, $ssn);
}
```

Una pratica comune è utilizzare i metodi bind che permettono una comunicazione client-DB sicura.

## CSRF

Con `cross site request` si intendono delle richieste fatte da un dominio ad un altro tramite codice javascript. Un chiaro esempio potrebbe essere se sul blog `mioblog.com` è possibile lasciare commenti autenticati con il proprio account `facebook`. Questa possibilità può essere sfruttata da un attaccante con un attacco di tipo `cross-site request forgery` che consiste nel:

1. Utente accede alla sua banca.
1. L'attaccante invia un sito malevolo che l'utente visita dopo aver effettuato l'accesso al sito della banca e quindi dopo aver preso i `cookie di sessione`.
1. Tramite codice `javascript` nascosto dentro un immagine o un bottone, l'attaccante esegue azioni malevole facendo risultare queste azioni provenienti dalla sessione autorizzata dall'utente.

Normalmente questo attacco viene effettuato tramite una `POST request` per nascondere i dati all'utente che a differenza di una `TTP GET` non vengono mostrati nell'url.

Possibili contromisure a questo attacco possono essere:

* **Referer header**. Un sito può richiedere il campo referer head dove viene identificato l'indirizzo della pagina che fa la richiesta per poi scartare tutte le richieste provenienti da domini non autorizzati ecc. (questa soluzione però richiede numerose informazioni sensibili dell'utente stesso come la cronologia e quindi non viene utilizzata molto spesso come soluzione)
* **Secret token**. Questa contromisura è basata sulla policy `same-origin`. Quando un utente fa una richiesta per una pagina il web server crea anche un secret token e un secret timestamp che viene inviato all'utente, il browser dell'utente per richiedere un'azione deve inserire nella richiesta lo stesso secret code che il sito ha mandato precedentemente, le pagini non proveniente dal dominio che invia il secret token non potranno accedere ad esso.
* **Same-site cookies**. Quando viene fatta una richiesta, nella richiesta vanno inseriti i cookie dell'utente, con quetsa contromisura al normale cookie vengono anche inseriti 2 ulteriori cookies:
  * **Strict**: un cookie che non viene mai inviato dal browser se il sito che si cerca di raggiungere non è sullo stesso dominio. (Quindi un cookie che non viene mai inviato nelle cross-site request).
  * **Lax**: Un cookie che può essere inviato nelle cross-site request ma solo se la richiesta proviene da un'azione volontaria dell'utente, ad esempio, il click in un link e non viene inviato se la richiesta proviene da un iframe o un immagine invisibible. Tramite questa mitigazione il `cross-site` è consetito solo se l'utente vede l'url cambiare e quindi ha modo di verificare l'azione eseguita.

(**IDEA:** Ci sono browser `open-source` che possono essere modificati per inviare sempre entrambi i `coockie`, se vengono, tramite una campagnia di `phishing`, fatti installare all'utente possono bypassare queste contromisure).

## Smart Contracts

Con `blockchain` si intende una struttura di memorizzazione di dati replicati decentralizzata (senza un'autorità superiore che possiede e controlla i dati) e mantenuta da più parti non fidate (non certificate). Essenzialmente la blockchain può essere intesa come gruppo di entità che in comune accordo definiscono quali sono i blocchi che la compongono, ogni blocco accettato dal `50% + 1` delle entità viene considerato valido e la blockchain valida è quella valida per il `50% + 1` delle entità, ogni entità ha una copia dell'interà blockchain. Con il concetto di blockchain viene anche inserito il concetto di blocco; un blocco è una lista di transizioni `hashate` ogniuna tra due parti (con relativo pagamento di `fee`) che avviene con un contratto accettato da entrambe le parti. Un contratto può contenera la dichiarazione di scambio di denaro ma anche codice eseguibile o dichiarazione di diritto d'autore, creando così moneta scambiabile ma anche tecnologie come `smart contrats` o `NFT`.
Un blocco, in particolare, contiene: l'hash del blocco precedente, un header e una lista di transazioni.
Gli `Smart contracts` sono contratti tra due parti che possono contenere codice eseguibile e sono scritti in parcolaro linguaggi di programmazione, ad esempio, uno dei più utilizzati soprattutto nella blockchain di `etherium` è `Solidity`. `Solidity` è un linguaggio di programmazione ad alto livello molto nuovo e quindi con molti problemi di sicurezza che possono generare numerosi attacchi.

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

## Buffer Overflow

Il buffer overflow è un attacco tipico ad un programma C eseguibile grazie a degli errori comuni commessi dagli sviluppatori. Per comprendere il buffer overflow bisogna prima comprendere come funzione l'allocamento in memoria da parte di un programma. Per un programma C la memporia può essere divisa in 5 segmenti che qui verranno descritti dall'indirizzo più alto a quello più basso:

* **Stack**: (Ordinato con indirizzi dal più basso al più alto) Contiene variabili locali, epb della funzione, return address e gli argomenti passati.
* **Libraries**: Librerie caricate staticamente e quindi presenti nell'eseguibile.
* **Heap**: Contiene le variabili dinamica, cioè inizializzate con malloc, calloc ecc.
* **Main executable**:
  * **BSS**: Contiene valori statici e globali non inizializzati. Questo spazio viene riempito con tutti 0, quindi le variabili non inizializzate conterranno 0.
  * **Data**: Contiene le variabili statiche e globali inizializzate.
  * **Text**: Contiene il codice eseguibile del programma.

```C
void func(int a, int b)
{
int x, y ;
x =a+ b;
y = a -b;
}
```

Dato questo codice C, quando la funzione viene invocata, vengono caricati:

* In onrdine inverso gli argomenti.
* Return address (L'indirizzo dell'istruzione da eseguire finita la funzione).
* EBP Indirizzo di memoria che punta all'ESP della funzione che ha invocato la funzione in esecuzione.
* Variabili locali in ordine deciso dal compilatore.

Un altro registro utilizzato è l'**esp**, cioe lo stack pointer, un puntatore che punta alla posizione corrente del frame della funzione.

In assembly la funzione, quindi, agisce così:

1. movl 12 (%ebp ) , %eax (Mette nel registro momentaneo eax il valore contenuto in epb + 12 quindi b)
1. movl 8 (%ebp ), %edx (Mette nel registro momentaneo edx il valore contenuto in epb + 8 quindi a)
1. addl %edx, %eax (Somma a + b e inserisce il risultato in eax)
1. movl %eax , -8 (%ebp ) (Mette il risultato della somma contenuto dentro eax in ebp -8 cioè x)
1. movl 12 (%ebp ) , %eax (Mette nel registro momentaneo eax il valore contenuto in epb + 12 quindi b)
1. subl %edx, %eax (Sottrae da edx il valore di eax e lo salva in eax)
1. movl %eax , -4 (%ebp ) (Mette il risultato della somma contenuto dentro eax in ebp -4 cioè y)
Ogni funzione chiamata viene aggiunta in fondo allo stack frame.
