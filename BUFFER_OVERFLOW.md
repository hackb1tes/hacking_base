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