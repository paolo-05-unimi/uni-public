# Volumetrics Models

# Modelli volumetrici in breve

Nella categorizzazione mostrata in aula, i modelli volumetrici 3D stanno nel ramo 3-manifold e comprendono due famiglie principali:

1. discreti e irregolari: mesh poliedrali (tetra/hexa mesh);
2. discreti e regolari: griglie voxelizzate.

Messaggio chiave:

- le mesh poliedrali sono l'analogo volumetrico delle mesh poligonali;
- i voxel sono l'analogo 3D di una immagine raster.

![image.png](Volumetrics%20Models/image.png)

# Mesh poliedrali

## 1.1 Definizione

Una mesh poliedrale è composta da poliedri adiacenti faccia-a-faccia.

Tipi di celle citati:

- hexahedra (cuboidi);
- tetrahedra (piramidi a base triangolare);
- poliedri generici (rari in pratica).

Terminologia pratica:

- hexahedron mesh (hexa-mesh);
- tetrahedron mesh (tetra-mesh).

Esempio preso dalle slide:

- visualizzazione didattica indicata: [https://www.hexalab.net](https://www.hexalab.net/).

## 1.2 Struttura dati

Come per le mesh poligonali, la struttura combina:

- geometria: vertici con coordinate $(x,y,z)$;
- connettività: celle (3D), facce (2D), edge (1D);
- attributi su vertici, interpolati nel volume interno delle celle.

La lezione sottolinea anche che esistono varianti dati derivate dal paradigma half-edge per gestire mesh poliedrali.

![image.png](Volumetrics%20Models/image%201.png)

## 1.3 Uso principale: FEM/FEA

Uso tipico mostrato:

- simulazioni fisiche (Finite Element Method / Finite Element Analysis).

Esempi citati esplicitamente:

- simulazione di carico strutturale (palazzi, ponti);
- simulazioni termiche (diffusione del calore nel volume).

Punto importante:

- la qualità della mesh volumetrica condiziona accuratezza e stabilita della simulazione.

![image.png](Volumetrics%20Models/image%202.png)

## 1.4 Problema aperto: hexa-meshing / tetra-meshing

Task difficile evidenziato in aula:

- input: mesh superficiale (spesso tri-mesh) M;
- output: mesh volumetrica (hexa o tetra) il cui bordo approssima M.

Esempio preso dalle slide:

- schema diretto INPUT (surface mesh) -> OUTPUT (hexa-mesh).

## 1.5 Tetra-mesh come complesso simpliciale

Analogia didattica:

- triangolo = simplesso di superficie;
- tetraedro = simplesso di volume.

Ogni punto interno/di bordo di un tetraedro si esprime in modo unico con coordinate baricentriche sui 4 vertici.

Formula (coerente con la notazione di lezione):

$$

\mathbf{p}=\sum_i t_i\,\mathbf{p}_i,
\quad \sum_i t_i=1,
\quad 0\le t_i\le 1

$$

Interpolazione di un attributo scalare $a$:

$$

a(\mathbf{p})=\sum_i t_i\,a_i

$$

## 1.6 Tetrahedralization

Concetto chiave mostrato:

- ogni poliedro può essere scomposto in tetraedri, come ogni poligono in triangoli;
- la scomposizione non è unica.

Esempio preso dalle slide:

- un esaedro con almeno due decomposizioni alternative: 12 tetra oppure 5 tetra.

![image.png](Volumetrics%20Models/image%203.png)

## 1.7 Qualità, risoluzione, regolarità

Indicazioni della lezione:

- risoluzione di una mesh poliedrale = numero di celle (o vertici);
- più risoluzione -> simulazioni più accurate ma più lente;
- costo in elementi cresce in modo cubico con la scala lineare;
- utile avere risoluzione adattiva nelle zone rilevanti.

Classificazioni richiamate:

- pure hexa-mesh;
- hexa-dominant mesh.

Regolarità locale (analoga al caso mesh superficiali):

- edge regolare in hexa-mesh: condiviso da 4 hexa;
- edge regolare in tetra-mesh: condiviso da 6 tetra.

## 1.8 Tetra vs Hexa (trade-off)

Dalle slide:

- hexa-meshing è tipicamente più difficile da costruire;
- a parità di risoluzione, spesso le simulazioni su hexa sono considerate più accurate;
- ma risultati recenti citati a lezione mettono in discussione parte di questo vantaggio.

Vincolo pratico per la mesh di partenza (superficiale):

- two-manifold,
- chiusa,
- ben orientata.

# Modelli voxelizzati

## 2.1 Definizione di voxel model

Un modello voxelizzato è una griglia regolare 3D (lattice):

$$

\texttt{array[RES\_X][RES\_Y][RES\_Z] of Voxels}

$$

Analogie terminologiche dalle slide:

- voxel = volume element;
- pixel = picture element;
- texel = texture element.

Esempio (Java, mostrato in aula):

```java
Voxel[][][] volume = new Voxel[resX][resY][resZ];
```

## 2.2 Curse of dimensionality

Punto centrale della parte a voxel:

- il costo memoria cresce con legge cubica rispetto alla risoluzione lineare.

Esempio esplicito nelle slide:

- $1024^3$ voxel = 1 gigavoxel.

Evoluzione mostrata a schermo:

- $10\times4\times3$ → $20\times8\times6$ → $40\times16\times12$ → $80\times32\times24$ → $160\times64\times48$.

Conclusione operativa:

- anche con 1 bit/voxel il costo diventa presto oneroso;
- con 1 byte, float, double o colore per voxel il costo peggiora drasticamente.

![image.png](Volumetrics%20Models/bb837a18-06b1-44b5-b524-74ee27bce1ec.png)

![image.png](Volumetrics%20Models/68352edd-f909-4e15-88a7-5428ce0fea00.png)

## 2.3 Livelli di dettaglio e stampa 3D

Lezione:

- possibile costruire piramidi LoD (LoD0, LoD1, LoD2) anche su voxel;
- caso booleano 1 bit/voxel: pieno (1) / vuoto (0);
- tale rappresentazione è input naturale per alcuni dispositivi di rapid prototyping.

Parentesi pratica mostrata:

- pipeline mesh chiusa e ben orientata → slicer → G-Code → stampante 3D.

## 2.4 Cosa può contenere un voxel

La lezione mostra più semantiche possibili:

1. booleano pieno/vuoto;
2. indice di materiale/tipo terreno (esempio esplicito: terreni stile Minecraft);
3. colore RGB (texture volumetriche / solid textures);
4. scalare float (densità, intensità, temperatura, ecc.).

## 3) Volumetric textures (solid textures)

Quando 1 voxel = colore (es. RGB/RGBA):

- si modella il segnale nel volume interno;
- utile per oggetti che possono rompersi o essere sezionati;
- utile per pattern materici volumetrici (es. legno, marmo);
- non richiede parametrizzazione della superficie.

Aspetti hardware ricordati:

- storage in RAM GPU;
- accesso accelerato;
- interpolazione tri-lineare durante il campionamento.

Limite invariato:

- occupazione memoria elevata (slide con esercizi su texture 3D molto grandi).

![image.png](Volumetrics%20Models/image%204.png)

![image.png](Volumetrics%20Models/image%205.png)

## 4) Voxel scalari e dati medici

Quando 1 voxel = float (tipicamente in $[0,1]$):

- il dataset rappresenta un campo scalare 3D.

Applicazioni citate:

- CT/TAC: coefficiente di attenuazione ai raggi X;
- MRI/Risonanza magnetica: risposta del tessuto al campo magnetico;
- PET: attività del tracciante radioattivo.

Output visualizzabili richiamati in lezione:

- slice arbitraria;
- direct volume rendering;
- estrazione di una isosuperficie.

Suggerimento immagine:

- Inserire una tavola a 3 viste: slice arbitraria, direct volume rendering, mesh isosuperficie.

## 5) Isolinee, isosuperfici e marching

### 5.1 Definizioni

In 2D:

- isolinea di valore $\sigma$ = bordo della regione con valori > $\sigma$.

In 3D:

- isosuperficie di valore $\sigma$ = bordo della regione volumetrica con valori > $\sigma$.

### 5.2 Marching squares (2D, propedeutico)

Passi mostrati:

1. sogliare i voxel/pixel in dentro-fuori rispetto a $\sigma$;
2. trovare le intersezioni sugli edge che attraversano la soglia;
3. unire le intersezioni con segmenti secondo lookup table.

Per un quadrato ci sono $2^4=16$ configurazioni.

Interpolazione lineare su edge (formula esplicita):

$$
a(1-t)+bt=\sigma
\quad\Longrightarrow\quad
t=\frac{\sigma-a}{b-a}

$$

Suggerimento immagine:

- Inserire la sequenza di slide 64-69 (soglia -> intersezione -> segmenti -> risultato).

### 5.3 Marching cubes (3D)

Generalizzazione 3D:

- su ciascun cubetto 2x2x2, ogni vertice e dentro/fuori;
- numero casi: $2^8=256$;
- per simmetria i casi base sono 15;
- lookup table decide quali triangoli connettono le intersezioni.

Esempi diretti mostrati:

- caso con 4 sotto soglia / 4 sopra soglia -> 2 triangoli;
- caso con 7 sotto / 1 sopra -> 1 triangolo;
- caso con 5 sotto / 3 sopra -> 3 triangoli.

![image.png](Volumetrics%20Models/image%206.png)

### 5.4 Efficienza e qualità dell'output

La lezione discute due implementazioni:

- marching vicino all'isosuperficie (evitando molti cubi vuoti);
- brute force su tutte le celle (praticabile con hardware moderno).

Output tipico di marching cubes:

- mesh chiusa, two-manifold, ben orientata;
- ma spesso con triangoli sottili, piccoli, talvolta quasi degeneri (meshing di qualità non ideale).

## 6) Poisson Reconstruction: point cloud → volume → mesh

![image.png](Volumetrics%20Models/image%207.png)

### 6.1 Idea generale

Pipeline indiretta mostrata in lezione:

1. stendere una griglia voxel nel volume;
2. stimare una Signed Distance Function (SDF);
3. estrarre isosuperficie con marching cubes (soglia 0).

Vincoli locali imposti da ciascun punto di input $(\mathbf{p},\mathbf{n})$:

$$
f(\mathbf{p})=0,
\qquad
\nabla f(\mathbf{p})=\mathbf{n}

$$

### 6.2 Signed Distance Field (SDF)

Definizione operativa:

- valore assoluto = distanza dalla superficie chiusa;
- segno = interno (negativo) o esterno (positivo).

Lo SDF è quindi un volume scalare dispendioso ma molto utile come rappresentazione intermedia.

### 6.3 Confronto con metodi diretti

Metodi diretti (ball-pivoting/front advancing, Delaunay):

- mantengono i punti originali come vertici mesh;
- soffrono di rumore, outlier, disallineamenti, densità non uniformi, buchi.

Poisson Reconstruction:

- produce nuovi vertici mediando il dato;
- tende a ridurre rumore/difetti;
- garantisce mesh chiusa, two-manifold e ben orientata;
- ma può essere irregolare, perdere adattività locale e costare molto in memoria/calcolo.

![image.png](Volumetrics%20Models/image%208.png)

# Griglie adattive, quad-tree e octree

## 7.1 Perché passare a strutture adattive

Lez007.3 riparte dal punto finale della lezione precedente:

- il limite centrale dei voxel regolari è che la risoluzione e uniforme ovunque;
- questo amplifica la curse of dimensionality anche in regioni vuote o poco dettagliate;
- servono strutture gerarchiche capaci di raffinare solo dove necessario.

![image.png](Volumetrics%20Models/image%209.png)

Transizione didattica usata nelle slide:

- prima il caso 2D (quad-tree),
- poi la generalizzazione 3D (octree).

## 7.2 Quad-tree in 2D: idea operativa

Un quad-tree è una struttura ricorsiva ad albero per dati su griglia 2D:

- la radice rappresenta l'intero dominio;
- ogni nodo interno si divide in 4 figli (quattro quadranti);
- le foglie codificano blocchi omogenei (es. pieno/vuoto nel caso binario).

Nell'esempio delle slide, il dato di partenza e una "voxellizzazione" 2D binaria:

- 1 = pieno;
- 0 = vuoto.

## 7.3 Esempi diretti mostrati in Lez007.3

Sequenza esplicita 2/4 → 3/4 → 4/4:

1. griglia 2D binaria di partenza;
2. scomposizione ricorsiva in quadranti;
3. albero finale risultante.

Esempio numerico importante riportato in slide:

- stesso contenuto rappresentato come quad-tree: 28 foglie + 9 nodi interni;
- come griglia piena equivalente 8x8: 64 voxel/pixel.

Esempio strutturale mostrato (nota "l'ordine" nelle slide):

$$

A \rightarrow [B, C, D, E]

$$

cioè ogni nodo interno espande in 4 figli con ordine fissato.

Altro esempio in profondita:

- visualizzazione livelli da LVL 0 a LVL 7;
- solo alcuni nodi vengono ulteriormente suddivisi;
- nel caso mostrato, la struttura codifica un disegno composto da 4 linee, il resto e vuoto.

<aside>
💡

riferimento slide 101-103 e poi 105-106 (compressione e livelli).

</aside>

## 7.4 Generalizzazione 3D: octree

Octree = estensione 3D del quad-tree.

Dalle slide (sommario):

- struttura ricorsiva ad albero;
- ogni nodo rappresenta un cubo di volume;
- la radice copre tutto il volume;
- ogni nodo non foglia ha 8 figli (otto ottanti);
- ogni foglia memorizza un voxel di dimensione dipendente dal livello.

Contenuto del voxel-foglia possibile:

- 1 bit (pieno/vuoto), oppure
- un valore scalare (es. Signed Distance), quando l'octree rappresenta un SDF.

## 7.5 Costo memoria: voxel pieni vs octree

Per profondità $n$:

- griglia piena equivalente: risoluzione lineare $N=2^n$;
- voxel totali densi: $N^3=2^{3n}$.

Esempio delle slide:

- $n=10 \Rarr 1024^3 \approx 10^9$ voxel.

Con octree, il numero di nodi effettivamente memorizzati è in pratica molto minore nei casi tipici di superfici sparse; la lezione lo riassume come andamento tendenzialmente quadratico con la risoluzione lineare del modello:

$$
\mathcal{O}(N^2)=\mathcal{O}(2^{2n})

$$

invece del cubico della griglia piena.

## 7.6 Poisson Surface Reconstruction con octree

Lez007.3 mostra la variante con struttura adattiva:

1. Point cloud in input;
2. stima di valori scalari su octree (SDF campionata adattivamente);
3. marching cubes adattato al caso octree;
4. output tri-mesh.

Messaggio operativo:

- stessa logica point cloud → campo scalare → isosuperficie,
- ma con supporto dati più compatto grazie alla discretizzazione adattiva.

![image.png](Volumetrics%20Models/image%2010.png)

## 7.7 Profondità octree e risoluzione della mesh

Esempio quantitativo riportato:

- profondità $n=6$ → equivalente a $64^3$;
- profondità $n=8$ → equivalente a $256^3$;
- profondità $n=10$ → equivalente a $1024^3$.

Interpretazione pratica:

- aumentare la profondità incrementa il dettaglio recuperabile,
- ma anche costo computazionale e uso memoria.

![image.png](Volumetrics%20Models/9e10c5b2-708d-4d1d-804b-1f7c48b1ba7b.png)

## 7.8 Nota importante su Poisson: mesh sempre chiusa

Osservazione finale della lezione:

- tecnicamente l'isosuperficie estratta da Poisson è sempre chiusa;
- se la point cloud iniziale descrive solo un lato dell'oggetto (caso tipico range scan),
- la mesh risultante chiude comunque il volume, introducendo una parte "ricostruita" non osservata.

![image.png](Volumetrics%20Models/image%2011.png)

## 8) Esempi chiave da ricordare (presi dalle slide)

- Hexa-meshing: da mesh superficiale a hexa-mesh di volume.
- Tetraedro: coordinate baricentriche per posizione e attributi nel volume.
- Tetrahedralization non unica: stesso esaedro, decomposizioni 12 tetra e 5 tetra.
- Voxel memory blow-up: $1024^3$ = 1 gigavoxel.
- Marching squares: 16 configurazioni su un quadrato.
- Marching cubes: 256 configurazioni (15 canoniche per simmetria).
- Intersezione su edge: $t=(\sigma-a)/(b-a)$.
- Poisson classico: vincoli $f(\mathbf{p})=0$ e $\nabla f(\mathbf{p})=\mathbf{n}$.
- Quad-tree 2D: solo i blocchi non omogenei vengono suddivisi.
- Esempio compressione: 28 foglie + 9 nodi interni invece di 64 celle piene (8x8).
- Octree 3D: ogni nodo interno ha 8 figli (ottanti).
- Profondita octree: $n=6,8,10$ corrisponde a $64^3,256^3,1024^3$ equivalenti.
- Poisson su octree: mesh chiusa anche con nuvole parziali.

## 9) Conclusione

Lez007.1-2-3 costruiscono un percorso unico in tre passi:

- mesh poliedrali: modellazione volumetrica per simulazioni numeriche (FEM/FEA);
- voxel regolari: rappresentazione volumetrica semplice e generale, ma costosa in memoria;
- strutture adattive (quad-tree/octree): stessa logica volumetrica con risoluzione locale, molto più efficiente nei casi sparsi.

Il ponte concettuale complessivo resta:

- trasformare dati acquisiti (point cloud) in campo scalare volumetrico (SDF),
- estrarre una superficie triangolata con marching cubes,
- scegliere una discretizzazione densa o adattiva in base a memoria, dettaglio e robustezza.

La terza parte aggiunge anche una cautela pratica importante: Poisson tende a chiudere sempre la superficie, quindi con input incompleti può generare chiusure geometriche non osservate.

## 10) Domande guida per ripasso

1. Quali differenze operative ci sono tra tetra-mesh e hexa-mesh in simulazione?
2. Perché la qualità degli elementi e centrale in FEA?
3. In cosa consiste la curse of dimensionality nei voxel regolari?
4. Qual è la differenza tra slice, direct volume rendering e isosuperficie?
5. Come si passa da marching squares (2D) a marching cubes (3D)?
6. Quali vantaggi e svantaggi porta Poisson Reconstruction rispetto ai metodi diretti su point cloud?
7. In che senso un quad-tree comprime una griglia 2D binaria?
8. Come si generalizza il quad-tree in octree e cosa cambia nel branching factor?
9. Come si interpreta la profondità $n$ dell'octree rispetto alla risoluzione equivalente $2^n$ per lato?
10. Perché Poisson pò restituire mesh chiuse anche quando la nuvola di punti copre solo una parte dell'oggetto?