# Ricognizione — sequenza di comandi

Sequenza operativa per le Fasi 0-3. **Tutti i comandi elencati sono di sola lettura**, tranne quelli nella sezione *Recupero*, esplicitamente marcati: quelli modificano lo stato e non si eseguono in autonomia.

L'ordine è per costo crescente di contesto: fermati appena hai abbastanza per ripartire. Scendere fino in fondo serve solo nella ripresa archeologica.

## 1. Documento di ripresa — esiste?

```bash
ls -la .handoff/ docs/handoff/ .claude/ 2>/dev/null   # .handoff/ è invisibile in git status
ls NOTES.md HANDOFF.md TODO.md 2>/dev/null
git log --oneline -5 -- '*handoff*' '*NOTES*'          # se è versionato, la storia dice quanto è vecchio
```

Se esiste, leggi **prima la sezione dei pericoli**, poi *Stato del repo* e *Prossimo passo*. Il resto dopo aver verificato: leggere un documento intero prima di sapere se descrive la realtà è contesto speso su una fonte non ancora attendibile.

## 2. Pericoli e operazioni interrotte

```bash
git status                       # dichiara rebase/merge/cherry-pick in corso e i conflitti aperti
ls .git/REBASE_HEAD .git/MERGE_HEAD .git/CHERRY_PICK_HEAD 2>/dev/null
git stash list
```

Ambiente lasciato in uno stato attivo o sporco (adatta allo stack del progetto):

```bash
docker ps                        # container ancora attivi dalla sessione precedente
git diff --stat -- '*.env*' '*lock*'   # credenziali temporanee, dipendenze cambiate
```

Migration pendenti o applicate a metà: usa il comando di stato del framework in uso (es. `php artisan migrate:status`, `npx prisma migrate status`, `alembic current`). Una migration a metà è il pericolo che va risolto per primo, e la sua risoluzione non è autonoma.

## 3. Stato git

```bash
git branch --show-current
git status -sb                   # include avanti/indietro rispetto all'upstream
git log --oneline -15
git log --oneline -5 --format='%h %ad %s' --date=short   # le date dicono quanto è stantio il contesto
git diff --stat                  # non committato, in forma compatta
git diff --staged --stat
```

Il diff completo **solo dei file che ti servono davvero**, dopo aver visto lo `--stat`:

```bash
git diff -- percorso/file.ext
```

Ambito del lavoro del branch senza leggere codice:

```bash
git merge-base --fork-point origin/main HEAD    # o il branch base del repo
git diff --stat origin/main...HEAD
git log --oneline origin/main..HEAD
```

## 4. Ambiente — prima dei test

Verifica che le dipendenze installate corrispondano al lockfile e che i servizi necessari siano attivi **prima** di girare la suite. Un rosso da ambiente interpretato come regressione manda l'indagine nella direzione sbagliata.

```bash
git diff --stat -- package-lock.json composer.lock poetry.lock yarn.lock
```

Poi il comando di install/sync del progetto, se il lockfile è più recente dell'ultima installazione.

## 5. Baseline

Gira test e linter **prima di modificare qualsiasi cosa** e registra l'esito: è il termine di paragone di fine sessione. Usa i comandi del repo (script in `package.json`, `composer.json`, `Makefile`, CI config) — non inventarne di nuovi.

Se la suite è troppo lenta per il budget, gira il sottoinsieme dell'area toccata e **dichiara** che il resto non è verificato.

## 6. Archeologia — nessun documento

```bash
git log --oneline -20 --format='%h %ad %an %s' --date=short   # chi, quando, cosa
git log --all --oneline -15                                    # anche i branch non correnti
git branch -a --sort=-committerdate --format='%(refname:short) %(committerdate:short)'
git log --diff-filter=M --name-only -5                         # file toccati più di recente
```

Marcatori introdotti dal lavoro recente (non un grep su tutto il repo):

```bash
git diff origin/main...HEAD | grep -n 'TODO\|FIXME\|WIP\|XXX'
git log -S 'TODO' --oneline -5
```

PR aperte o draft, se il repo ha un remote su una forge: la descrizione della PR è spesso la dichiarazione d'intento più completa che esista, e costa una richiesta sola.

## 7. Recupero di lavoro dichiarato ma introvabile

Ispezione (sola lettura):

```bash
git reflog --date=iso -20              # commit non più raggiungibili da un branch
git fsck --dangling                    # oggetti orfani: commit persi, stash droppati
git show <sha>                         # ispeziona un candidato prima di toccare qualsiasi cosa
git worktree list                      # il lavoro potrebbe essere in un altro worktree
```

**Da qui in poi si modifica lo stato: esponi cosa hai trovato e aspetta conferma.** Ripristinare un commit orfano, applicare uno stash o creare un branch di recupero sono operazioni che possono sovrascrivere lavoro presente. La scelta tra recuperare, rifare e abbandonare è dell'utente.
