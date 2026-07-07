# Parametric Curves & Surfaces

## In breve

Questa dispensa unifica in un solo percorso:

- definizione e uso delle curve parametriche;
- curve di Bezier (De Casteljau + forma polinomiale);
- continuità C0/C1 nelle curve composte;
- estensione alle superfici parametriche;
- Bezier patch quadratici e bi-cubici;
- benefici pratici nel design/CAD;
- conversione da superficie parametrica a mesh triangolare per il rendering.

## 1) Inquadramento nel panorama dei modelli 3D

Nella mappa generale del corso, le superfici parametriche stanno tra i modelli:

- superficiali continui;
- alternativi alle mesh lineari a tratti;
- molto usati in CAD/CAM e design industriale.

Messaggio chiave iniziale delle slide:

- una superficie parametrica rappresenta geometria davvero curva;
- non è vincolata a soli poligoni piani;
- si esprime con funzioni matematiche, non solo con triangoli.

![image.png](Parametric%20Curves%20&%20Surfaces/image.png)

## 2) Curve parametriche: definizione e terminologia (Lez006.1)

### 2.1 Ripasso funzione, dominio, codominio, immagine

Una curva parametrica è impostata come immagine di una funzione:

- funzione: $f: A \to B$;
- dominio parametrico: $A$;
- codominio: $B$;
- curva: $\mathrm{immagine}(f)$.

Esempio dalle slide (ripasso funzioni):

- $f: \mathbb{N} \to \mathbb{N}$, $f(x)=2x$;
- immagine = naturali pari.

### 2.2 Definizione di curva parametrica

Per le curve:

- $A \subseteq \mathbb{R}$ (una coordinata parametrica $t$);
- $B=\mathbb{R}^2$ (curve sul piano) oppure $B=\mathbb{R}^3$ (curve nello spazio);
- il punto sulla curva e $f(t)$.

Terminologia usata in aula:

- $t$ = parametro (o coordinata parametrica);
- $A$ = dominio parametrico;
- $f(A)$ = luogo dei punti della curva.

### 2.3 Usi richiamati nelle slide

- 2D: grafica vettoriale e CAD 2D.
- 3D: traiettorie in animazione (spesso $t$ rappresenta il tempo).

![image.png](Parametric%20Curves%20&%20Surfaces/image%201.png)

## 3) Esempio giocattolo: il segmento parametrico

Esempio preso direttamente dalle slide:

- dati due punti di controllo $\mathbf{p}_0, \mathbf{p}_1$,
- il segmento è descritto da

$$

f(t)=\mathbf{p}_0 + (\mathbf{p}_1-\mathbf{p}_0)t
= \mathbf{p}_0(1-t)+\mathbf{p}_1t,
\quad t\in[0,1].

$$

Osservazioni importanti (esplicite in lezione):

- $f(0)$ è l’inizio curva, $f(1)$ è la fine curva;
- i punti di controllo sono costanti nella formula e governano la forma;
- in questo caso speciale la curva è dritta e i punti di controllo appartengono alla curva.

![image.png](Parametric%20Curves%20&%20Surfaces/image%202.png)

## 4) Da archi a spline: famiglie industriali

La lezione ricorda che esistono varie famiglie di curve parametriche:

- Hermite splines;
- Bezier splines (focus del corso);
- B-splines;
- NURBS.

Idea strutturale chiave:

- curve complesse = concatenazione di archi;
- ogni arco è definito da una funzione polinomiale in $t\in[0,1]$.

Forma generale mostrata:

$$

f(t)=P_0(t)\,\mathbf{p}_0 + P_1(t)\,\mathbf{p}_1 + P_2(t)\,\mathbf{p}_2 + \dots

$$

dove:

- $P_i(t)$ sono polinomi base;
- $\mathbf{p}_i$ sono punti di controllo;
- aumentando il grado aumentano i punti di controllo.

![image.png](Parametric%20Curves%20&%20Surfaces/1a78b48a-c312-48fb-8ede-817fe02152e1.png)

## 5) Curve di Bezier in Lez006.1

### 5.1 Grado 1 come primo caso

Il segmento parametrico è già un arco di Bezier di grado 1:

- polinomi base: $P_0(t)=1-t$, $P_1(t)=t$;
- 2 punti di controllo;
- relazione generale: grado $n\to n+1$ punti di controllo.

### 5.2 Arco di Bezier di grado 2 (algoritmo di De Casteljau)

Esempio dalle slide (3 punti di controllo):

1. Con parametro $t$, si interpolano i primi segmenti:
$\mathbf{q}_0 = \mathrm{lerp}(\mathbf{p}_0,\mathbf{p}_1,t)$,
$\mathbf{q}_1 = \mathrm{lerp}(\mathbf{p}_1,\mathbf{p}_2,t)$.
2. Si interpola ancora:
$f(t)=\mathrm{lerp}(\mathbf{q}_0,\mathbf{q}_1,t)$.

Le slide propongono anche due esercizi:

- costruzione grafica a mano per vari valori di $t$;
- sviluppo algebrico per verificare che il grado è 2.

Suggerimento immagine:

- Inserire la slide con i passi di De Casteljau per il grado 2 e i punti intermedi $\mathbf{q}_0,\mathbf{q}_1$.

## 6) Lez006.2 - Tangente di una curva parametrica

Data una curva e un parametro $t$:

- il punto e $f(t)$;
- incremento piccolo $\varepsilon$ sposta il punto in $f(t+\varepsilon)$;
- vettore di spostamento: $f(t+\varepsilon)-f(t)$.

Passando al limite:

$$

\mathbf{T}(t) = f'(t)

$$

e direzione tangente unitaria:

$$

\hat{\mathbf{T}}(t)=\frac{f'(t)}{\|f'(t)\|}.

$$

Esempio richiesto nelle slide:

- applicare la regola all'esempio giocattolo del segmento per verificare la direzione attesa.

![image.png](Parametric%20Curves%20&%20Surfaces/99a3c1bc-0b34-477f-83a0-acf382cdbcb2.png)

## 7) Curve di Bezier: due formulazioni equivalenti

### 7.1 Formulazione geometrica (De Casteljau)

Per grado 3, dalle slide:

- prima riga: $\mathbf{q}_0,\mathbf{q}_1,\mathbf{q}_2$ da interpolazioni sui 4 punti controllo;
- seconda riga: $\mathbf{r}_0,\mathbf{r}_1$;
- punto finale: $f(t)=\mathrm{lerp}(\mathbf{r}_0,\mathbf{r}_1,t)$.

Esempio diretto di codice mostrato in aula:

```cpp
vec3 bezier(float t, vec3 p0, vec3 p1, vec3 p2, vec3 p3) {
  vec3 q0 = mix(p0,p1,t);
  vec3 q1 = mix(p1,p2,t);
  vec3 q2 = mix(p2,p3,t);
  vec3 r0 = mix(q0,q1,t);
  vec3 r1 = mix(q1,q2,t);
  return mix(r0,r1,t);
}
```

### 7.2 Formulazione polinomiale equivalente

Per grado 2 (come derivato nelle slide):

$$

f(t)=\bar t^2\,\mathbf{p}_0 + 2\bar t t\,\mathbf{p}_1 + t^2\,\mathbf{p}_2,
\quad \bar t = 1-t.

$$

Per grado 3:

$$

f(t)=\bar t^3\,\mathbf{p}_0 + 3\bar t^2 t\,\mathbf{p}_1 + 3\bar t t^2\,\mathbf{p}_2 + t^3\,\mathbf{p}_3.

$$

Versione ottimizzata (dalle slide):

```cpp
vec3 bezier(float t, vec3 p0, vec3 p1, vec3 p2, vec3 p3) {
  float k = 1.0 - t;
  return (k*k*k)*p0 +
         (3.0*k*k*t)*p1 +
         (3.0*k*t*t)*p2 +
         (t*t*t)*p3;
}
```

Nota storica presente in aula:

- le curve di Bezier nascono in ambito automotive (fusoliere/carrozzerie).

![image.png](Parametric%20Curves%20&%20Surfaces/image%203.png)

![image.png](Parametric%20Curves%20&%20Surfaces/image%204.png)

## 8) Endpoint, tangenti agli estremi e scelta del grado cubico

Proprietà evidenziate:

- gli endpoint dell'arco coincidono con primo e ultimo punto controllo;
- tangente iniziale orientata come $\mathbf{p}_1-\mathbf{p}_0$;
- tangente finale orientata come $\mathbf{p}n-\mathbf{p}{n-1}$.

Perché il grado 3 è il piu usato:

- controlla in modo intuitivo posizione e orientamento in inizio/fine;
- con grado $< 3$ non si controllano indipendentemente entrambe le tangenti;
- grado $>3$ spesso ridondante: meglio concatenare piu archi cubici.

## 9) Concatenare archi: Bezier curve/path, C0 e C1

![image.png](Parametric%20Curves%20&%20Surfaces/image%205.png)

Dalle slide sulla concatenazione:

- continuità C0: endpoint consecutivi coincidenti;
- continuità C1: oltre alla coincidenza, allineamento dei segmenti di controllo ai lati del giunto.

Interpretazione pratica:

- C0 evita gap geometrici;
- C1 evita spigoli visivi nella direzione tangente (curva smooth).

## 10) Curve di Bezier nel mondo reale

Applicazioni citate esplicitamente:

- disegno vettoriale 2D;
- formato SVG (standard Web);
- descrizione del profilo dei font;
- path di animazione (specifica di posizione/velocità in punti chiave).

Esempio pratico proposto a lezione (Inkscape):

1. Importare una bitmap a due colori.
2. Usare Path → Trace bitmap.
3. Ottenere un path Bezier cubico editabile.
4. Salvare SVG e osservare il file testuale.

![image.png](Parametric%20Curves%20&%20Surfaces/image%206.png)

## 11) Dalle curve alle superfici parametriche

Generalizzazione principale:

- curva: $f: A\subseteq\mathbb{R} \to B\subseteq\mathbb{R}^3$;
- superficie: $f: A\subseteq\mathbb{R}^2 \to B\subseteq\mathbb{R}^3$.

Per superfici, il parametro diventa coppia $(s,t)$:

$$

f(s,t)=\begin{bmatrix}x(s,t)\\y(s,t)\\z(s,t)\end{bmatrix}.

$$

Convenzioni del corso:

- dominio parametrico spesso $A=[0,1]\times[0,1]$;
- coordinate parametriche chiamate $s,t$ (oppure $u,v$);
- analogia diretta con coordinate texture.

![image.png](Parametric%20Curves%20&%20Surfaces/image%207.png)

## 12) Bezier patch: grado 2 e grado 3

### 12.1 Patch quadratico (grado 2)

Esempio dalle slide con griglia $3\times3$ di punti controllo:

- per un $(s,t)$ fissato, si valutano tre curve Bezier in una direzione (ottenendo $a,b,c$);
- poi si valuta una quarta curva con quei tre punti e parametro nell'altra direzione;
- si ottiene il punto finale del patch.

### 12.2 Bi-cubic Bezier patch (grado 3)

Configurazione standard:

- griglia $4\times4$ di punti controllo $\mathbf{p}_{i,j}$;
- algoritmo equivalente in due ordini:
    - prima lungo $t$, poi lungo $s$;
    - oppure prima lungo $s$, poi lungo $t$.

Proprietà dichiarate nelle slide:

- patch interpolante sui 4 angoli ($\mathbf{p}{0,0}$*, $\mathbf{p}{0,3}$*, $\mathbf{p}{3,0}$*, $\mathbf{p}{3,3}$f*);
- approssimante sugli altri punti controllo;
- bi-cubico perché cubico in entrambe le coordinate parametriche.

Esempio diretto dalle slide:

- demo interattiva indicata per modificare i 16 punti controllo e osservare la deformazione del patch.

## 13) Piano tangente locale e controllo di forma

Le slide mostrano che:

- il patch passa per alcuni punti chiave (angoli);
- il piano tangente locale e influenzato dai punti controllo vicini (handle locali).

Conseguenza pratica:

- piccoli spostamenti dei control points possono cambiare in modo intuitivo forma e orientamento locale della superficie.

Suggerimento immagine:

- Inserire la slide con punti verdi/blu che evidenzia passaggio per angoli e controllo del tangente.

## 14) Superfici di Bezier multi-patch e continuità

Come per le curve composte:

- una superficie di Bezier completa e composta da piu patch connessi;
- matching dei control points sul bordo tra patch adiacenti -> continuita geometrica senza gap;
- vincoli aggiuntivi (colinearita opportuna dei punti controllo interni al bordo) -> transizioni smooth senza creases.

Suggerimento immagine:

- Inserire una configurazione a 2 o 4 patch adiacenti con evidenza dei control points condivisi sul bordo.

## 15) Lo zoo delle superfici parametriche e ruolo delle NURBS

La lezione estende il quadro alle famiglie:

- Hermite;
- Bezier;
- B-spline;
- NURBS.

Punto enfatizzato:

- NURBS molto usate in CAD 3D.

Suggerimento immagine:

- Inserire una tabella/slide comparativa tra famiglie di spline surface.

## 16) Benefici delle superfici parametriche (dal design al CAD)

Benefici esplicitati nelle slide:

- controllo intuitivo della forma (utile in CAID);
- rappresentazione compatta (pochi punti controllo);
- geometria realmente curva;
- risoluzione adattiva legata a numero/distribuzione patch;
- uv-mapping intrinseco alla definizione;
- texturing semplice rispetto al caso mesh non parametrizzata.

Caso storico riportato:

- Utah Teapot (Martin Newell, 1975), costruita con patch di Bezier.

Software citati in aula:

- Rhinoceros3D, Autodesk Alias, moi3D, Blender, Solidworks.

![image.png](Parametric%20Curves%20&%20Surfaces/image%208.png)

## 17) Geometry processing e conversione in mesh

Le superfici parametriche possono essere input/output di processing:

- semplificazione (riduzione numero patch);
- ricostruzione automatica da altre rappresentazioni (task difficile).

Per il rendering, spesso serve mesh poligonale:

1. Campionare il dominio $A$ su griglia regolare (es. $10\times 10$).
2. Per ogni campione $(s,t)$ creare vertice in $f(s,t)$.
3. Calcolare normale per vertice da derivate di $f$.
4. Fare diagonal split di ogni quad campionato.
5. Ottenere una tri-mesh regolare.

Messaggio pratico della lezione:

- conversione semplice e fattibile anche on-demand in fase di rendering;
- mantenere la rappresentazione parametrica per storage/authoring resta vantaggioso.

![image.png](Parametric%20Curves%20&%20Surfaces/image%209.png)

## 18) Riassunto operativo per studio/esame

1. Curva parametrica: immagine di $f(t)$ con $t$ nel dominio parametrico.
2. Tangente locale: derivata $f'(t)$ (normalizzata per la direzione).
3. Bezier: definizione equivalente De Casteljau/polinomi di Bernstein.
4. Cubica: compromesso ideale per controllo intuitivo e concatenazione.
5. Continuità tra archi: C0 (posizione), C1 (tangente).
6. Superficie parametrica: immagine di $f(s,t)$ da $\mathbb{R}^2$ a $\mathbb{R}^3$.
7. Bi-cubic patch: griglia $4\times4$, interpolazione agli angoli, controllo locale della forma.
8. Multi-patch: vincoli sui control points per continuità e smoothness.
9. Vantaggi: compattezza, design intuitivo, uv naturale, uso CAD.
10. Rendering: tessellazione/campionamento in tri-mesh.

## 19) Domande guida per ripasso

1. Qual è la differenza formale tra dominio, codominio e immagine in una curva parametrica?
2. Perché $f'(t)$ descrive la direzione tangente della curva in $f(t)$?
3. In che senso De Casteljau e forma polinomiale sono equivalenti?
4. Perché la cubica è spesso preferita ad altri gradi in pratica?
5. Quali vincoli garantiscono C0 e C1 nella concatenazione di archi Bezier?
6. Come cambia la definizione passando da curve parametriche a superfici parametriche?
7. Qual è la procedura di valutazione di un bi-cubic Bezier patch?
8. Quali vantaggi offrono le superfici parametriche rispetto alle mesh lineari a tratti?
9. Perché il mapping UV è naturale nel caso parametrico?
10. Come si ottiene una tri-mesh regolare da una superficie parametrica?