# Agente-Analisi-Brand

Sistema automatico che, dato il nome di un brand e i suoi canali ufficiali, produce in
pochi minuti un documento PDF di analisi pronto per un primo incontro commerciale.

Progettato e realizzato da Fabio Giacomini. Questo repository è una scheda di progetto:
documenta l'architettura e le scelte, **non contiene il workflow eseguibile**.

---

## Il problema

Preparare l'analisi di un brand prima di un incontro commerciale richiede molto lavoro e produce documenti diversi a seconda di chi li scrive. Gli strumenti di mercato che promettono di automatizzarla costano migliaia di euro l'anno e restituiscono cruscotti, non documenti da portare in riunione.

Il vincolo interessante non era però il tempo: era **l'affidabilità**. Un modello
linguistico a cui si chiede «com'è percepito questo brand» risponde sempre, e risponde bene
anche quando non ha alcun dato. In un documento che finisce sul tavolo di un potenziale
cliente, una frase inventata vale più di dieci frasi corrette.

---

## Cosa produce

Un PDF in formato A4 orizzontale:

| | |
|---|---|
| **01** | Sintesi esecutiva — verdetto per area e tre priorità con impatto e sforzo |
| **02** | Percezione del brand — punti di forza e di debolezza, ciascuno con la propria fonte |
| **03** | Tone of voice — come parla il brand, e se regge rispetto a come il mercato lo vede |
| **04** | Presenza social — conteggi del brand e di due competitor, misurati con lo stesso metro |
| **05** | Stato di salute del sito — Core Web Vitals e audit tecnico on-page |
| **06** | Il linguaggio a confronto — quali temi i competitor mettono per iscritto e il brand no |
| **07** | Nota metodologica — cosa non è stato possibile verificare, e perché |

Un esempio completo con dati fittizi: [`esempio-report.pdf'] https://github.com/fabiogiacomini/Agente-Analisi-Brand/blob/main/esempio-report.pdf

---

## La regola che governa il sistema

> **Ogni numero che compare nel documento deve poter essere ricontato a mano da una
> persona che apra la stessa fonte.**

È la scelta architetturale centrale, e ha portato a **togliere** funzionalità che sarebbero
state facili da aggiungere e impossibili da difendere:

| Escluso | Perché |
|---|---|
| Volumi di ricerca e keyword gap | Richiedono un database a pagamento: senza, sarebbero numeri inventati |
| Stima del traffico potenziale | Nasce da un CTR ipotizzato — è un calcolo, non un conteggio |
| Engagement rate | È un rapporto che richiede i follower, dato che le piattaforme non espongono in modo affidabile |

Resta solo ciò che è contato: post pubblicati, date, like, commenti, punteggi misurati da
Google, occorrenze di parole in pagine realmente scaricate.

---

## Architettura

```
Modulo web (7 campi)
      │
      ├─ RACCOLTA — deterministica, nessun modello coinvolto
      │    · lettura e audit tecnico del sito
      │    · Core Web Vitals via PageSpeed Insights
      │    · conteggio dei post pubblici di brand e competitor
      │    · lettura delle home dei competitor
      │
      ├─ RICERCA — modello con accesso al web
      │    · percezione di mercato, forze, debolezze
      │    · individuazione di 2 competitor comparabili
      │    · ogni affermazione porta l'URL da cui proviene
      │
      ├─ REDAZIONE — modello SENZA accesso al web
      │    · può usare soltanto il dossier raccolto sopra
      │
      ├─ CONTROLLO DI TRACCIABILITÀ
      │    · verifica ogni fonte contro gli URL realmente consultati
      │    · rimuove dal documento ciò che non regge, e lo dichiara
      │
      └─ IMPAGINAZIONE → PDF
```

**Tre presidi contro l'invenzione**, in ordine di importanza:

1. **Separazione fra chi cerca e chi scrive.** Due chiamate distinte al modello. La seconda
   non ha accesso a internet: può usare solo il dossier. Un modello che non può cercare non
   può inventare una fonte credibile.
2. **Fonte obbligatoria per ogni affermazione**, verificata da un nodo di controllo che
   cancella ciò che non è tracciabile.
3. **La disciplina è visibile nel documento.** Ogni pagina ha una colonna in monospaziato
   con la provenienza di ogni voce; l'ultima pagina elenca i buchi. Un report che dichiara
   cosa non sa è più utile di uno che finge di sapere tutto.

Una scelta di metodo che vale la pena segnalare: **i competitor non li sceglie l'operatore,
li individua il sistema durante la fase di ricerca, e vengono poi misurati con lo stesso
scraper, lo stesso numero di post e lo stesso conteggio di parole del brand.** È l'unico
modo per rispondere davvero alla domanda «come sta andando rispetto ad altri simili».

---

## Scelte tecniche e loro motivo

**Esecuzione seriale, non parallela.** L'orchestratore non garantisce l'ordine fra rami
paralleli, e il nodo che assembla il dossier legge da tutti i predecessori. In serie si
perdono quaranta secondi e si guadagna un sistema che non fallisce a intermittenza.

**PDF generato da HTML, non PPTX.** Il formato presentazione richiede librerie fragili e
restituisce comunque un layout da sistemare a mano. Da HTML si controlla la griglia al
millimetro e il documento è identico su ogni macchina. La conversione avviene con
Gotenberg, in un container accanto all'orchestratore.

**Impaginazione a pagina fissa con troncamento controllato.** Ogni sezione ha un tetto di
elementi renderizzabili, così un brand molto documentato non manda il layout in overflow.

---

## Costi di esercizio

Circa **0,50 – 1 € per brand analizzato**, senza abbonamenti né depositi minimi.
Cinquanta analisi al mese stanno sotto i venticinque euro.

---

## Limiti, dichiarati

- **LinkedIn non viene misurato**: nessuno scraper affidabile e contrattualmente sereno per
  i post aziendali. L'URL viene comunque usato nell'analisi qualitativa.
- **Il confronto lessicale non è un'analisi SEO**: riguarda le sole home page e non dice
  nulla sulla domanda di ricerca reale.
- **Su brand poco citati in rete la pagina 02 è povera.** È corretto che lo sia: per un
  commerciale quel vuoto è a sua volta un'informazione.

---

## Stack

Orchestrazione n8n autoinstallata su VPS · Claude (Anthropic API) con ricerca web
server-side · Apify per i dati social pubblici · Google PageSpeed Insights · Gotenberg per
la conversione · JavaScript nei nodi di elaborazione · HTML e CSS per il template del
documento.

---

## Licenza e utilizzo

**Tutti i diritti riservati.** Vedi LICENSE (https://github.com/fabiogiacomini/Agente-Analisi-Brand/blob/main/license).

Questo repository ha finalità dimostrative. Il workflow eseguibile, i prompt di sistema e
il template del documento non sono inclusi e non sono concessi in uso, riproduzione o opera
derivata. Per collaborazioni: contattami.
