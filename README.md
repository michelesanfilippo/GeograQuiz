
## Introduzione

Il progetto ha come obiettivo lo sviluppo di un bot Telegram in Python che elabora un dataset da noi ricavato, per creare in automatico dei quiz geografici ai quali partecipare in compagnia dei propri amici, dentro le chat di gruppo di Telegram. 

Le basi di conoscenza coinvolte per realizzare il nostro dataset sono state Wikidata e DBpedia. Nello specifico Wikidata è stata la fonte principale, dalla quale abbiamo estrapolato informazioni ed immagini, mentre DBpedia è stata importante per ottenere l'insieme delle nazioni affini, ossia delle alternative che potessero essere utilizzate come possibili risposte al quiz.


## Bot Telegram

### Introduzione

Una volta estratte le informazioni di nostro interesse, la cosa da fare è stata predisporre un metodo agevole per generare delle domande, tenere traccia delle risposte date, e competere insieme ai propri amici. A tal proposito la soluzione più rapida ed efficace è stata quella di sviluppare un bot Telegram. L'idea è stata quella di permettere al bot di inviare dei messaggi di tipo quiz all'interno dei gruppi, tenendo il conto delle risposte corrette date dai vari partecipanti. 


### Libreria

L'API di Telegram può essere utilizzata in un qualsiasi linguaggio di programmazione che supporti le richieste HTTP, in questo caso si è preferito utilizzare il linguaggio Python più la libreria [python-telegram-bot](https://github.com/python-telegram-bot/python-telegram-bot). Questa libreria si installa molto facilmente tramite il gestore dei pacchetti Pip di Python, e fornisce un wrapper completo per tutte le funzionalità dell'API Telegram.

Per l'installazione basta eseguire da riga di comando: `pip install python-telegram-bot --upgrade`


### Funzionamento

Il bot di fatto viene aggiunto al gruppo come fosse un normale utente, e l'interazione con esso avviene tramite i cosiddetti *comandi*. Quest'ultimi altro non sono che semplici parole chiave antecedute da uno *slash*, che vengono intercettate dal bot non appena si invia un messaggio che le contiene. Inoltre questi comandi vengono resi accessibili tramite un pannello predisposto che rende l'interazione ancora più pratica. 

Il bot può mantenere una sessione di gioco per ogni gruppo e per ogni sessione ha una lista dei partecipanti con i relativi punteggi, questa lista è memorizzata dentro il dizionario `session`. All'avvio del bot esso stampa un messaggio di aiuto contenente una descrizione di tutti i comandi e la sua modalità di funzionamento.

![aiuto](./img/aiuto.png)

Il primo passo per creare un quiz è quello di dare il comando `/nuovo`, così facendo si innesca l'handler `new_handler`, esso nel caso in cui il comando sia stato lanciato in una chat adeguata, aggiunge la chat alla sessione, inizializzando il conteggio dei round residui a `rounds_left: rounds_count`, lo stato del quiz come `is_started: False` e la possibilità di richiedere le domande come `can_request: False`. Questo perché a questo punto il bot è in attesa che gli utenti diano la loro patecipazione.

![nuovo](./img/nuovo.png)

Per prendere parte al gioco, gli utenti del gruppo devono dare il comando `/partecipo`. Si fa notare che si è preferito definire esplicitamente i partecipanti piuttosto che rendere il quiz aperto direttamente a tutto il gruppo per poter bloccare l'invio di domande successive fino a che tutti gli effettivi interessati al quiz avranno risposto. Una volta che l'utente sarà stato aggiunto alla partita, il suo contatore di risposte corrette verrà posto a zero `sessions[chat_id]['participants'][user_id] = 0`.

![partecipo](./img/partecipo.png)

Una volta stabilita la lista dei partecipanti si può procedere al quiz tramite il comando `/avvia` . Questo di fatto imposta per la partita sessions `[chat_id]['is_started'] = True` e `sessions[chat_id]['can_request'] = True` mettendo il bot in attesa del comando `/avanti`. Affinché si possa avviare il gioco sono richiesti almeno due partecipanti, se così non fosse il bot invierà un messaggio informativo.

Per richiedere la nuova domanda, come già detto, è necessario che tutti i partecipanti al quiz abbiano comunicato la loro risposta, o che comunque lo stato sia in `sessions[chat_id]['can_request'] = True`. A questo punto lo stato passa in `sessions[chat_id]['can_request'] = False`, e viene preparata ed inviata la domanda. 

Di volta in volta il contenuto viene ricavato partendo da una tra le quattro funzioni per la generazione di domande, scelta a caso. I valori restituiti dalle le funzioni sono coerenti tra di loro, e prevedono un testo, delle risposte, l'indice della risposta corretta ed, eventualmente, un'immagine. La produzione delle domande verrà approfondita in seguito.


È possibile annullare la partita in corso in un suo punto qualsiasi digitando `/termina`. Questo comando rimuove direttamente la sessione per la chat all'interno della quale è chiamato.

Una volta terminati i round viene stampata la classifica dei partecipanti, con i relativi punteggi.
![classifica](./img/classifica.png)

### Generazione delle domande

Come già anticipato, sono stati implementati quattro possibili tipi di domanda:

+ Popolazione: data una nazione, a quanto ammonta la sua popolazione?
+ Bandiera: data l'immagine di una bandiera, a quale nazione appartiene?
+ Capitale: dato il nome di una capitale, a quale nazione appartiene?
+ Mappa: data l'immagine di una mappa con un territorio evidenziato, di che nazione si tratta?


Quiz sulle mappe geografiche:
![esempio1](./img/avanti1.png)
Quiz sulle bandiere:
![esempio2](./img/avanti2.png)
Quiz sulla popolazione:
![esempio3](./img/popolazione.png)

Telegram permette di impostare una *spiegazione* per ogni domanda del quiz, in un primo momento avevamo pensato di aggiungere la descrizione dell'about di DBpedia per la nazione della risposta corretta, purtroppo il limite nel numero dei caratteri della spiegazione ci ha costretti ad una soluzione differente, anche più funzionale, cioè il link a Wikipedia.

![tip](./img/tip.png)
