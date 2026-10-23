---
name: session-resume
description: Ripresa di un lavoro interrotto — ricostruzione dello stato reale, riconciliazione con la documentazione di ripresa e ripartenza senza perdita di contesto. Usa questa skill OGNI VOLTA che una sessione riprende un lavoro già iniziato, per esempio quando l'utente dice "riprendiamo", "continua da dove eravamo", "dove eravamo rimasti", quando apre una sessione su un progetto lasciato a metà, quando esiste un handoff, un compendio, un riassunto di compattazione o delle note di sessione da cui ripartire — e anche quando non esiste nulla e il punto di interruzione va ritrovato nel repo, o quando il codice e la documentazione di ripresa non coincidono. Attivala anche di tua iniziativa quando, su un repo su cui ti viene chiesto di lavorare, trovi un working tree sporco, un branch WIP, un rebase interrotto o un documento di handoff. NON usarla per chiudere una sessione (quella è session-handoff), né per iniziare un task nuovo su un repo di cui hai già il contesto.
---

# Session Resume

Questa skill definisce come riprendere un lavoro interrotto: ricostruire dove si era arrivati, verificarlo, e ripartire senza rifare da capo e senza costruire sopra un'assunzione sbagliata.

Il rischio della ripresa è opposto a quello della chiusura. Chiudendo si perde contesto e lo si sa. Riprendendo si può **credere di averlo**: un documento di handoff scritto bene è convincente, e chi lo legge non ha le evidenze per accorgersi che nel frattempo il repo è cambiato. Il lavoro riparte fluido nella direzione sbagliata, e l'errore si scopre a valle — quando ci sono già tre commit sopra.

**Prerequisiti.** Questa skill si appoggia a quattro skill che governano aspetti che qui non vengono ripetuti:

- `dev-discipline` — come si scrive il codice una volta ripartiti, e la Definition of Done rispetto a cui si misura il residuo ereditato.
- `dev-communication` — quando procedere in autonomia e quando fermarsi. Alla ripresa conta più del solito: le decisioni si prendono con meno contesto del normale.
- `design-docs` — ADR e diagrammi, che alla ripresa servono soprattutto in lettura: sono la fonte dei *perché* che il codice non contiene.
- `session-handoff` — l'altra metà del ciclo. La sua Fase 5 descrive la ripresa in sintesi; questa skill ne è la versione estesa e, quando si riprende, vince.

Leggile se non sono già in contesto. Qui c'è solo ciò che è specifico della ripresa.

## Le tre regole che governano tutto

In caso di tensione tra qualsiasi indicazione di questa skill, vincono queste tre, in quest'ordine:

1. **La realtà del repo vince sul racconto.** Ogni documento di ripresa — handoff, compendio, riassunto di compattazione, note di sessione — è una fotografia con una data, scritta da chi non sapeva cosa sarebbe successo dopo. Si legge come **ipotesi da verificare**, mai come stato accertato. Il costo di verificare è qualche comando; il costo di non verificare è il lavoro costruito sopra prima che l'errore emerga.
2. **Prima ci si orienta, poi si lavora.** Nessuna modifica al codice prima di sapere dove sei e da che baseline parti. Il primo commit fatto sopra uno stato non verificato è precisamente ciò che rende costoso tornare indietro: finché non hai scritto niente, correggere la rotta è gratis.
3. **Contesto minimo sufficiente.** Ricostruire costa, e i token spesi a orientarsi non sono disponibili per lavorare: si paga il minimo che serve a ripartire **in sicurezza**, in ordine di densità informativa. Questa regola è terza perché quando confligge con le prime due perde: rileggere è caro, ripartire da uno stato sbagliato lo è di più.

## Gerarchia delle fonti

Dalla più affidabile alla meno, quando due fonti si contraddicono sul **cosa**:

1. **Lo stato eseguibile** — cosa compila, cosa passa i test, cosa fa il codice adesso.
2. **Git** — commit, diff, reflog: registra fatti, non intenzioni.
3. **Il documento di ripresa** — dichiara intenzioni, decisioni e ragioni, con la data della sua scrittura.
4. **La memoria della conversazione precedente** (transcript, riassunti) — ricostruzione di seconda mano.

L'ordine vale per il *cosa*. Sul **perché** la gerarchia si rovescia: git non dice mai perché una strada è stata abbandonata, il documento sì. Quindi un documento smentito dai fatti non è carta straccia — resta l'unica fonte su decisioni, vicoli ciechi e trade-off, che vanno preservati anche quando lo stato che descrive non esiste più.

## Proporzionalità: quanta ricostruzione serve

Applicare il workflow completo a un residuo da tre righe è esattamente lo spreco che la terza regola vuole evitare. Calibra:

- **Leggera** — stessa giornata, stesso repo, working tree che riconosci, documento coerente: orientamento (Fase 0-1) e via.
- **Standard** — giorni di distanza, o un documento da riconciliare: workflow intero, senza archeologia.
- **Archeologica** — nessun documento, settimane di distanza, lavoro di un altro, o divergenze materiali: serve anche la Fase 3, e il budget va concordato prima di bruciarlo.

Il criterio non è quanto è complesso il task, ma **quanta incertezza c'è sullo stato**.

## Workflow

### Fase 0 — Orientamento a basso costo

Prima di leggere codice, stabilisci se un documento di ripresa esiste e se c'è qualcosa che può fare danno.

- **Cerca il documento** nell'ordine in cui potrebbe esistere: convenzione del repo (`docs/handoff/`, `NOTES.md`, note di sessione, `.claude/`), poi `.handoff/` — che è escluso da git e quindi **invisibile in `git status`**: va guardato esplicitamente — poi il riassunto di compattazione se questa sessione prosegue la stessa conversazione.
- **Se il documento esiste, la prima sezione che leggi è quella dei pericoli** (nel template di `session-handoff` è *⚠️ Da sapere subito*): migration a metà, rebase in corso, servizi lasciati attivi, credenziali temporanee. Non è informazione di contesto, è informazione che può fare danno prima ancora che il lavoro riprenda — si verifica e si risolve per prima.
- **Verifica anche senza documento** se c'è un'operazione git interrotta: `git status` lo dichiara. Un rebase a metà scoperto dopo tre modifiche costa molto più che scoprirlo subito.

I comandi, in ordine di costo, sono in `references/ricognizione.md`. Leggilo quando ti serve la sequenza operativa; non serve impararlo a memoria qui.

### Fase 1 — Stato reale, non dichiarato

Ricostruisci con comandi ciò che il documento afferma (o che nessuno ha scritto):

- **Git**: branch e upstream (avanti/indietro di quanti commit), `git status`, `git diff --stat` (il diff completo solo dopo, e solo dove serve), commit recenti con date, stash, operazioni in corso.
- **Ambiente prima dei test**: dipendenze allineate al lockfile, migration pendenti, servizi necessari attivi. **Ripristina l'ambiente prima di interpretare qualsiasi rosso.** Un fallimento da dipendenza mancante letto come regressione manda l'indagine a sbattere per mezz'ora e brucia il contesto che serviva a lavorare.
- **Baseline, adesso**: gira test e linter **prima** di toccare qualsiasi cosa. Serve a due cose: sapere cosa era già rotto, e avere il termine di paragone di fine sessione. Senza baseline stabilita alla ripresa, a fine sessione non potrai distinguere i tuoi rossi da quelli ereditati — e li consegnerai come ereditati per default, che è la forma più comoda di scorciatoia non dichiarata. Se girarli non è possibile, dichiaralo come "non verificato", mai come "a posto".
- **Attribuzione del working tree sporco**: modifiche non committate non sono automaticamente il tuo lavoro. Capisci di chi sono e di quale task **prima** di toccarle: mescolare il lavoro di un altro task nel tuo commit produce un diff che nessuno saprà rivedere.

### Fase 2 — Riconciliazione documento ↔ realtà

Quando un documento esiste, confrontalo con lo stato reale e classifica la divergenza — la classe determina cosa fare:

- **Documento indietro** (nel repo c'è più lavoro di quanto dichiari): divergenza benigna. Aggiorna il documento e prosegui.
- **Documento avanti** (dichiara fatto ciò che nel repo non c'è): è il caso pericoloso. O il lavoro non è mai stato salvato, o è altrove — altro branch, stash, worktree, commit orfano. **Cercalo prima di riscriverlo**: reflog, stash, branch locali, oggetti dangling. Se è perso davvero, dillo con la stima di cosa va rifatto; non ripartire in silenzio fingendo che il documento non sia mai esistito.
- **Documento parallelo** (altri hanno lavorato a monte: merge, rebase, force push, dipendenze aggiornate): il lavoro descritto può essere già superato o in conflitto. Verifica **prima di continuare** se la feature è già stata risolta altrove o se il branch è ancora riallineabile. Rifare ciò che qualcun altro ha già fatto è il modo più costoso di riprendere con diligenza.
- **Documento di un altro task** (slug, branch o file citati non corrispondono): non adattarlo a forza. Cerca quello giusto, o trattalo come assenza di documento e passa alla Fase 3.
- **Più documenti sovrapposti**: il più recente non è automaticamente quello valido. Vale quello i cui branch e commit citati esistono e coincidono con lo stato attuale. Gli altri, se descrivono task chiusi, andavano cancellati alla chiusura: segnalalo.

La regola che li governa tutti: **la divergenza si segnala e il documento si corregge prima di lavorarci sopra.** Se la divergenza ha conseguenze materiali — lavoro perso, lavoro già fatto da altri, decisione superata dai fatti — è una cosa da portare all'utente subito: la scelta tra recuperare, rifare e abbandonare è sua, non tua.

### Fase 3 — Nessun documento: trovare il punto di interruzione

Ordine di ricerca, dal segnale più economico al più caro:

1. **Data dell'ultimo commit**: il gap temporale ti dice quanto è stantio tutto il resto e quanta fiducia dargli.
2. **Working tree sporco e stash**: le modifiche non committate sono, quasi sempre, esattamente il punto in cui la sessione si è fermata. Il file toccato per ultimo è il candidato più probabile.
3. **Branch non mergiati e PR aperte o draft**: nome del branch, messaggi dei commit e descrizione della PR sono la dichiarazione d'intento più economica che esista.
4. **Diff del branch rispetto alla base, in `--stat`**: l'ambito reale del lavoro senza leggere una riga di codice.
5. **Test rossi** (dopo aver ripristinato l'ambiente): puntano al punto di rottura meglio di qualsiasi ricerca testuale.
6. **Marcatori introdotti di recente**: TODO, FIXME, WIP nei commit recenti — cercati nella storia recente, non con un grep su tutto il repo.
7. **Segnali infrastrutturali**: migration non eseguite, lockfile modificato, `.env` cambiato. Dicono cosa era in corso sotto il codice.

E la regola che vale su tutto: **ricostruisci il "cosa", non inventare il "perché".** Git dimostra cosa è stato fatto; non dice perché una strada è stata scelta e un'altra abbandonata. Dichiara esplicitamente ciò che è andato perso invece di colmarlo con una ricostruzione plausibile: un perché inventato ma credibile blocca l'indagine e sopravvive nel repo molto più a lungo di un "non lo sappiamo".

### Fase 4 — Allineamento con l'utente

Prima di scrivere codice, riporta in poche righe: dove siamo davvero, cosa diverge dal documento, qual è il primo passo. È il momento della **domanda minima** di `dev-communication` — non un questionario, e nemmeno una conferma rituale: se stato e prossimo passo sono chiari e coerenti, procedi.

Fermati e chiedi solo quando: la divergenza ha conseguenze materiali; il documento lasciava una decisione aperta; la ripresa richiede un'operazione strutturale o irreversibile (riallineare un branch molto indietro, recuperare lavoro perso, abbandonare un ramo).

Nota che **il "prossimo passo" del documento è la proposta di una sessione passata, non un ordine.** Chi l'ha scritto non sapeva cosa sarebbe successo dopo. Valutalo contro lo stato di adesso, e se non è più il passo giusto dillo invece di eseguirlo per obbedienza al documento.

### Fase 5 — Ripartenza

Il codice lo governa `dev-discipline`. Qui conta solo ciò che è specifico dell'ereditare lavoro:

- **Chiudi ciò che è aperto prima di aprire altro.** Il residuo di una sessione precedente è debito che ha già una data: sommarci un fronte nuovo moltiplica il costo della prossima consegna. Se l'utente chiede esplicitamente altro, il residuo va ricensito, non dimenticato.
- **Le liste ereditate si ri-derivano, non si eseguono a scatola chiusa.** Una propagazione incompleta dichiarata ("mancano questi chiamanti") si riverifica con un grep: nel frattempo i chiamanti possono essere cambiati, e quella lista è il ricordo di una sessione che ne aveva pochi.
- **Non "correggere" scelte di cui non conosci la ragione.** Codice scritto da un'altra sessione sembra sbagliato finché non ne conosci il motivo. Cerca l'ADR, il commento, il vicolo cieco documentato prima di raddrizzarlo: i vicoli ciechi sono scritti proprio perché nessuno li ripercorra.
- **Il debito ereditato non si eredita in silenzio.** Scorciatoie e TODO dichiarati nel documento restano dichiarati: o li chiudi, o li riporti aggiornati. Un residuo che sparisce dal documento senza essere stato fatto diventa debito invisibile, che è la forma peggiore.
- Le verifiche finali si confrontano con la **baseline stabilita in Fase 1**, non con quella dichiarata nel documento.

### Fase 6 — Non perdere il contesto una seconda volta

La ripresa produce informazione che nessuno aveva scritto. Se quel lavoro resta solo in questa sessione, la ripresa successiva lo ripaga da zero.

- **Aggiorna il documento esistente man mano** — stato, residuo consumato, decisioni prese — invece di riscriverlo alla chiusura. Come dice `session-handoff`, il documento di un task è uno solo e diventa il suo diario.
- **Annota ciò che ti è costato scoprire e non era scritto**: dove vive un modulo, perché un test è flaky, quale comando serve per far girare l'ambiente. È l'informazione con il miglior rapporto tra costo di riscoperta e righe scritte.
- **Se non c'era nessun documento e il task continua**, la ripresa è il momento più economico per crearne uno: il contesto è appena stato ricostruito. Passa a `session-handoff` per formato e collocazione.
- Quando il task finisce, il documento va cancellato o archiviato: un handoff che sopravvive al lavoro descrive uno stato che non esiste più.

## Economia di contesto

L'obiettivo è il minimo sufficiente a ripartire in sicurezza, non la ricognizione più completa possibile.

- **Leggi in ordine di densità**: metadati prima dei contenuti. `git log --oneline`, `git status`, `git diff --stat` prima di qualsiasi diff completo; il documento prima del codice; i file e le righe citati prima di una ricerca a tappeto.
- **Non rifare la ricognizione del repo se non serve.** Ri-mappare un repo che il documento già circoscrive è ripagare un costo già pagato: la ricognizione ampia (`repo-audit`, Fase 1) serve per repo sconosciuti, non per riprendere un task noto.
- **Grep mirato invece di letture integrali**: apri il range di righe citato e allarga solo se il contesto non basta.
- **Non caricare ciò che non toccherai.** Leggere "per farsi un'idea" un'area fuori scope è debito di contesto: si paga subito e non produce nulla.
- **Lo stato, non la cronaca.** Ricostruire la conversazione precedente è caro e poco affidabile: usala solo per i *perché* che nessun'altra fonte contiene, mai per i *cosa*, che git dimostra meglio in una frazione dei token.
- **Se la ricostruzione sta consumando il budget del lavoro, fermati e dichiaralo.** Meglio ripartire con un obiettivo ridotto e il documento aggiornato che bruciare la sessione a orientarsi e chiuderla senza aver né lavorato né scritto nulla — perché la ripresa dopo ripagherebbe lo stesso costo da capo.

## Casi edge

**Operazione git interrotta** (rebase, merge, cherry-pick a metà): va risolta prima di lavorare, ma **continuare o abortire non è una decisione da prendere in autonomia** — entrambe distruggono qualcosa. Esponi lo stato e chiedi.

**Ambiente non ripristinabile** (servizi non avviabili, credenziali scadute, migration non applicabili): dichiara i test come non verificati e dillo. È preferibile a un'indagine su rossi che non sono del codice.

**Lavoro dichiarato ma introvabile**: cercalo in reflog, stash, altri branch e worktree, oggetti dangling prima di darlo per perso. Se è perso, dillo con la stima di cosa va rifatto.

**Branch già mergiato o superato**: verificalo prima di continuare, altrimenti riscrivi lavoro che è già in main.

**Branch molto indietro rispetto alla base** (ripresa dopo settimane): rebase, merge o ripartire è una decisione strutturale → si espone con i trade-off, non si sceglie in autonomia. E il fatto che i test girassero allora non dice nulla su adesso: dipendenze, tooling e CI possono essere derivati.

**Working tree sporco di origine ignota**: attribuiscilo prima di includerlo nel tuo lavoro.

**Ripresa del lavoro di un'altra persona**: se è raggiungibile, una domanda mirata costa meno di un'ora di archeologia. E le sue convenzioni restano le convenzioni: riprendere non è il momento per riscriverne lo stile (`dev-discipline`, principio 3).

**Repo senza git**: timestamp dei file, backup e artefatti di build sono tutto ciò che hai. La ricostruzione è più debole — dichiarane i limiti invece di presentarla con la stessa fiducia.

**Monorepo o più aree toccate**: uno stato per area, ognuna col suo branch e le sue verifiche. Uno stato aggregato nasconde quale parte è pulita.

**Lavoro non su codice** (documenti, analisi, ricerca): cambia cosa puoi verificare, non il metodo — stato, divergenze, residuo, prossimo passo.

## Cosa NON fare

- Non fidarti di un documento senza verificarlo, per quanto sia scritto bene: la fluidità della scrittura non è evidenza.
- Non scrivere codice prima di aver stabilito la baseline: a fine sessione non sapresti più cosa hai rotto tu.
- Non colmare i buchi della ricostruzione con "perché" plausibili ma inventati.
- Non risolvere in autonomia ciò che hai trovato a metà (rebase in corso, lavoro da recuperare, branch da riallineare).
- Non eseguire il "prossimo passo" del documento senza chiederti se è ancora quello giusto.
- Non lasciar sparire dal documento un residuo che non hai fatto.
- Non ri-mappare l'intero repo per ripartire su un task che il documento già circoscrive.
- Non ricostruire la cronaca della sessione precedente: serve lo stato.
- Non riscrivere codice ereditato verso il tuo stile mentre "riprendi".
- Non chiudere la ripresa senza aver scritto da qualche parte ciò che ti è costato scoprire.

## Checklist prima di scrivere la prima riga di codice

1. Se c'era un documento: verificato contro git e test, non solo letto?
2. Pericoli e operazioni a metà (rebase, migration, servizi, credenziali) risolti o esposti?
3. Ambiente ripristinato — o l'esito dei test dichiarato inaffidabile?
4. Baseline test/linter stabilita **adesso**, prima di toccare?
5. Divergenze documento ↔ repo segnalate, e quelle materiali portate all'utente?
6. Working tree sporco attribuito (di chi, di quale task)?
7. Verificato che il lavoro non sia già stato fatto, mergiato o superato altrove?
8. Il prossimo passo è ancora quello giusto alla luce dello stato reale?
9. Residuo e debito ereditati censiti, non solo letti?
10. Ciò che hai ricostruito è finito nel documento, o ci finirà alla chiusura?

Se una risposta è "no", sistemala prima di ripartire — o dichiarala esplicitamente all'utente.
