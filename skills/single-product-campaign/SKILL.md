# Single Product Campaign Manager

Gestisci le campagne Meta Ads dove ogni prodotto ha la sua inserzione dedicata. Aggiungi nuovi prodotti, spegni quelli che non performano, redistribuisci le ad tra gli adset — tutto da Claude Code con Pipeboard.

## A chi serve

Hai un e-commerce e sponsorizzi i tuoi prodotti con Meta Ads? Probabilmente hai (o vuoi avere) campagne dove ogni prodotto ha la sua ad dedicata: una foto, il nome, il prezzo, un link diretto alla pagina prodotto. Questo tipo di campagne ti permette di capire esattamente quali prodotti vendono e quali no, spegnere quelli che non convertono e aggiungere nuovi arrivi subito.

Questa skill automatizza tutto il processo: dall'aggiunta di nuovi prodotti alla distribuzione nelle campagne, fino allo spegnimento delle ad che non funzionano.

## Come funzionano le campagne prodotti singoli

Sono campagne Meta Ads (obiettivo Sales) in cui ogni inserzione promuove un singolo prodotto con:
- Una foto del prodotto
- Il prezzo e il nome come titolo (es. "€34,90 - Abito Sophie")
- Un copy generico del brand come testo principale
- Una description breve (es. "Spedizione gratuita 24/48h")
- CTA "Acquista ora" con link diretto alla pagina prodotto

A differenza delle campagne con catalogo dinamico, qui ogni ad è creata manualmente. Il vantaggio è il controllo totale: sai esattamente quale prodotto performa, puoi spegnere i flop e aggiungere nuovi arrivi subito.

Di solito si creano più campagne di questo tipo, ognuna con più adset (gruppi di inserzioni), e in ogni adset si mette un massimo di N ad per non diluire troppo il budget.

## Cosa puoi fare

- **Aggiungere prodotti** — dai le URL del tuo sito, il resto è automatico (immagini, nome, prezzo estratti dalla pagina)
- **Spegnere ad** — manda uno screenshot dal Business Manager con le ad evidenziate, oppure chiedi un recap performance e decidi
- **Vedere come vanno** — recap per campagna, adset o singola ad con CPA, ROAS, spesa
- **Riorganizzare** — redistribuire le ad tra gli adset quando serve

## Come iniziare

1. Assicurati di avere [Pipeboard](https://pipeboard.co) configurato come MCP in Claude Code
2. Copia la cartella `single-product-campaign` sul tuo computer
3. Apri Claude Code nella cartella
4. Scrivi: **"Configuriamo il mio account"**

Claude ti guiderà con un questionario per configurare tutto. Dopo il setup, sei operativo.

## Requisiti

- Account Pipeboard con accesso al tuo account Meta Ads
- Un sito e-commerce con pagine prodotto accessibili (Shopify, WooCommerce, ecc.)
- Almeno una campagna Meta Ads attiva con obiettivo Sales (oppure la creiamo insieme)
