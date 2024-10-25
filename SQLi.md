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