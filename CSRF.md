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