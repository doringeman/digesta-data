# Instrucțiuni de adnotare

Aceleași instrucțiuni pentru orice adnotator (model sau om). Adnotatorii lucrează
independent: nu văd propunerile altui adnotator.

## Scop

Pentru fiecare variantă a unei grile de examen (INM, INPPA) identifici articolele din
Codul civil care **decid** dacă afirmația este adevărată sau falsă. Setul rezultat va
evalua un agent care, pentru o speță, spune ce articole se aplică, pro și contra.
Propunerile tale sunt verificate apoi de avocați.

## Ce primești

Grile în format JSONL: `id`, `enunt`, `variante` (`A`, `B`, `C`), `adevarate`
(`true`/`false` pentru fiecare literă), `enunt_negativ`.

- Afirmația evaluată este **enunțul împreună cu varianta** (ex. „Nu poate fi declarată
  nedemnă de a moșteni:” + „persoana care…”).
- **Excepție**: când `enunt_negativ` este `true` („Care dintre enunțuri **nu** este
  corect”), afirmația evaluată este **varianta singură**, iar `adevarate` spune dacă
  varianta e corectă din punct de vedere juridic. Nu inversezi a doua oară.
- Valoarea de adevăr vine din baremul oficial. Nu o schimbi; dacă textul actual al
  Codului pare să ducă la alt verdict, folosești semnalul `barem_vs_text_actual`.

## Reguli care nu se încalcă

1. Propui doar articole pe care le-ai **găsit și citit integral în corpus**, cu unealta
   `cauta_corpus.py`. Nu propui un număr de articol din memorie fără să-l fi citit.
2. Pentru fiecare articol propus copiezi **exact, caracter cu caracter**, pasajul decisiv
   (o propoziție sau un alineat) din textul afișat de `cauta_corpus.py art`. Nu
   reformulezi, nu prescurtezi, nu corectezi. Nu incluzi numărul alineatului („(2)”).
   Un citat scurt (până la 25 de caractere) e acceptat doar dacă e un
   alineat, o literă sau un punct întreg (ex. „răscumpărarea;” din art. 770 alin. (1)
   lit. d), „dreptul de superficie;” din art. 551 pct. 2).
3. Nu scrii text de lege nicăieri altundeva. Motivarea e explicația ta, nu un citat.

## Ce articole propui (`articole`)

- **Variantă adevărată**: articolele care o susțin (temeiul).
- **Variantă falsă**: articolele care arată de ce e falsă: adesea o excepție, o
  condiție, un termen sau o altă regulă decât cea sugerată de variantă.
- **Setul minim suficient**: toate articolele fără de care verdictul nu se poate
  stabili, dar nu cele doar înrudite cu tema. De obicei 1–3.
- **Doar partea care decide**: la o variantă falsă construită pe „regula generală în
  loc de excepție”, propui excepția; regula generală o numești doar în `motivare`.
- Același articol poate apărea de mai multe ori, cu alineate diferite, dacă fiecare
  alineat contribuie la verdict.
- O definiție intră în `articole` (nu în `suport`) când varianta pune în discuție chiar
  noțiunea definită.
- Capcane frecvente, de verificat de fiecare dată:
  - **articolul vecin**: verifică articolele din jur (`vecini`);
  - **regula generală în loc de excepție**: dacă o regulă are excepții, caută-le;
  - **dispoziții modificate**: textul din corpus este forma în vigoare în 2026.
- Nu adaugi articolele la care textul trimite explicit („potrivit art. X”): acestea se
  calculează automat.

## `suport` (opțional)

Articole fără de care un articol din `articole` nu poate fi aplicat, dar la care textul
**nu** trimite explicit, de regulă o definiție (ex. „bun mobil”, „profesionist”). Tot
cu citat exact. Lista rămâne goală dacă nu e cazul.

## `incredere`

- `sigur`: temeiul e clar și l-ai verificat în text.
- `probabil`: temeiul e cel mai plauzibil, dar poate lipsi un articol sau
  interpretarea e discutabilă.
- `nesigur`: nu ai găsit un temei convingător.

## `semnale` (listă, poate fi goală)

- `barem_vs_text_actual`: textul actual al Codului pare să ducă la alt verdict decât
  baremul (posibilă modificare a legii după examen). Propui totuși articolele relevante.
- `in_afara_codului`: temeiul e în altă lege (ex. Legea nr. 71/2011, Codul de procedură
  civilă, o lege specială). Propui articole din Codul civil doar dacă sunt cu adevărat
  relevante; altfel `articole` rămâne goală.
- `enunt_ambiguu`: enunțul sau varianta se pot citi în mai multe feluri.
- `problema_sursa`: textul grilei pare deteriorat (trunchiat, litere greșite) sau
  baremul nu se potrivește cu formularea întrebării, independent de lege.

## Format de ieșire

Un rând JSON pentru fiecare variantă (JSONL, UTF-8), cu id-ul grilei urmat de literă:

```json
{"id": "exemplu-0001-A",
 "articole": [{"numar": "1", "alineat": "1", "citat": "Sunt izvoare ale dreptului civil legea, uzanțele și principiile generale ale dreptului."}],
 "suport": [],
 "motivare": "De ce aceste articole decid verdictul, în 1–3 propoziții.",
 "incredere": "sigur",
 "semnale": []}
```

- `numar` ca în corpus: `"1730"`, `"2029^1"`.
- `alineat`: alineatul din care e citatul (`"3"`, `"1^1"`) sau `null` dacă articolul
  nu are alineate numerotate.
- Pentru o literă sau un punct: citezi textul fără eticheta („b)”, „2.”), pui
  alineatul părinte în `alineat` și, opțional, eticheta în `litera` (`"b"`, `"2"`).

## Unelte

Din directorul proiectului:

```
.venv/bin/python cauta_corpus.py cuprins [filtru]     structura Codului, cu intervalele de articole
.venv/bin/python cauta_corpus.py lista "<cale>"       articolele unei unități (calea din cuprins)
.venv/bin/python cauta_corpus.py cauta <termen>...    articolele care conțin toți termenii
.venv/bin/python cauta_corpus.py art <numar>...       textul integral
.venv/bin/python cauta_corpus.py vecini <numar> [k]   articolele din jur
.venv/bin/python cauta_corpus.py verifica <fișier> [<lot>]   verificare mecanică a propunerilor
```

Căutarea ignoră diacriticele și majusculele. Înainte de a termina, rulezi `verifica`
pe fișierul tău, cu lotul primit ca al doilea argument (verifică și că ai toate
variantele), și corectezi tot ce e semnalat, copiind din nou citatele din corpus.
Lucrezi doar cu corpusul și cu cunoștințele tale juridice, fără căutări pe internet.

## Mod de lucru recomandat

1. Citește enunțul și toate variantele; identifică instituția juridică.
2. Găsește unitatea în `cuprins` și parcurge articolele ei (`lista`).
3. Caută după termenii din enunț și din variante (`cauta`), inclusiv sinonime.
4. Citește integral articolele candidate și vecinii lor.
5. Pentru fiecare variantă, separat: stabilește setul minim, copiază citatele, scrie
   motivarea, alege încrederea și semnalele.

## Istoric

- **v1** (2026-09-27): folosită la pilotul pe 20 de grile (`data/eval/pilot/`).
- **v2** (2026-09-27), după observațiile adnotatorilor din pilot: excepția pentru
  întrebările negative (în v1, „enunț + variantă” producea o dublă negație), semnalul
  `problema_sursa`, reguli pentru litere, alineate multiple, partea care decide și
  definiții; `verifica` controlează și lotul complet și alineatul declarat.
