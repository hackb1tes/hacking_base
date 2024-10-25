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