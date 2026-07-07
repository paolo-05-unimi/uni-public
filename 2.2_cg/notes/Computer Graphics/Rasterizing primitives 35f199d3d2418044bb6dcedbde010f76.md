# Rasterizing primitives

## In breve

- ruolo del rasterizer nella pipeline rasterization-based;
- conversione da clip space a screen space (viewport);
- algoritmo base di rasterizzazione dei triangoli e test di appartenenza;
- interpolazione degli attributi tramite coordinate baricentriche;
- clipping, culling e regole sui bordi;
- primitive supportate (triangoli, linee, punti) e note storiche;
- aliasing e antialiasing.

## 1) Rasterizer nella pipeline

La rasterizzazione è la fase che trasforma primitive 2D in frammenti.

Flusso essenziale:

- input: vertici già trasformati in clip coords (poi in screen coords);
- output: frammenti (candidati pixel) con coordinate intere e attributi interpolati;
- step successivo: fragment processing per calcolare il colore RGB finale.

Punti chiave:

- è un procedimento 2D;
- è altamente parallelizzabile;
- sulle GPU è implementato in hardware, quindi molto efficiente ma poco flessibile.

![image.png](Rasterizing%20primitives/image.png)

## 2) Da Clip Space a Screen Space (Viewport)

I vertici arrivano al rasterizer in clip space omogeneo, poi si passa a screen space 2D.

Spazi coinvolti:

- clip space: $[-1,+1] \times [-1,+1]$ (si ignora $z$ in questa fase);
- screen space: $[0, resX-1] \times [0, resY-1]$.

Mapping tipico (con viewport di origine $(x_0,y_0)$):

$$

\begin{aligned}
X_{screen} &= x_0 + \frac{resX}{2}(x_{clip}+1) \\
Y_{screen} &= y_0 + \frac{resY}{2}(y_{clip}+1)
\end{aligned}

$$

Nota:

- le coordinate screen dei vertici non sono necessariamente intere;
- i frammenti prodotti hanno invece coordinate intere (pixel grid).

![image.png](Rasterizing%20primitives/image%201.png)

## 3) Rasterizzazione dei triangoli: idea base

Obiettivo:

- produrre tutti e soli i frammenti i cui centri di pixel cadono dentro il triangolo 2D.

Input/Output:

- input: tre vertici 2D in screen space (`x,y` non intere);
- output: frammenti interni al triangolo (uno per pixel).

Algoritmo base (parallelizzabile):

1. Calcola il bounding box intero che contiene il triangolo.
2. Scansiona tutte le posizioni intere nel bounding box.
3. Per ogni posizione, testa se è interna; se sì, produce un frammento.

![image.png](Rasterizing%20primitives/c1dae46b-f711-446a-aa0a-c196b8c16204.png)

## 4) Clipping e culling

Casi:

- triangolo completamente fuori: culling, niente frammenti;
- triangolo completamente dentro: rasterizzazione diretta;
- triangolo parzialmente fuori: clipping (si rasterizzano solo i frammenti nel viewport).

Nota storica:

- il clipping geometrico "classico" (intersezioni, poligono, triangolazione) è' complesso;
- hardware moderno preferisce: bounding box + intersezione con viewport.

![image.png](Rasterizing%20primitives/image%202.png)

## 5) Test di appartenenza: semipiani ed edge function

Un triangolo 2D è l'intersezione di 3 semipiani.
Per ogni lato $(v_0, v_1)$:

$$

\begin{aligned}
\mathbf{d} &= \mathbf{v}_1 - \mathbf{v}_0, \quad
\mathbf{d}' = (-d_y, d_x) \\
E(\mathbf{p}) &= (\mathbf{p}-\mathbf{v}_0) \cdot \mathbf{d}'
\end{aligned}

$$

Il punto $\mathbf{p}$ è interno se il segno di $E(\mathbf{p})$ è coerente per tutti e tre i lati.

Orientamento:

- se il triangolo è orario vs antiorario, il segno atteso cambia;
- l'implementazione deve trattare correttamente entrambi i casi.

![image.png](Rasterizing%20primitives/image%203.png)

## 6) Interpolazione degli attributi (coordinate baricentriche)

Il rasterizer non produce solo frammenti, ma anche attributi interpolati.

Coordinate baricentriche:

$$

\mathbf{p} = \alpha \mathbf{v}_0 + \beta \mathbf{v}_1 + \gamma \mathbf{v}_2, \quad
\alpha + \beta + \gamma = 1

$$

Per ogni attributo (colore, normale, texcoord, ecc.):

$$

A(\mathbf{p}) = \alpha A_0 + \beta A_1 + \gamma A_2

$$

Suggerimento immagine:

- Inserire la slide che mostra l'interpolazione tramite coordinate baricentriche.

## 7) Regole sui bordi: evitare overdraw e gap

Problema:

- un frammento esattamente sulla linea tra due triangoli non deve essere prodotto due volte,
ma nemmeno scartato da entrambi.

Regola pratica:

- se un edge scarta un frammento, l'edge opposto non deve scartarlo;
- la GPU impone una regola coerente per tutto il frame (es. "top-left rule").

Suggerimento immagine:

- Inserire la slide con il frammento sulla linea e il richiamo al problema dei gap.

## 8) Back-face culling

Ottimizzazione utile quando la mesh è chiusa, ben orientata e opaca, con POV esterno.

Idea:

- le facce viste "da dietro" saranno comunque occluse da facce front-facing;
- quindi si possono scartare prima della rasterizzazione.

Test tipico (orientamento):

- si usa il verso del triangolo (o la normale) rispetto alla direzione di vista.

Suggerimento immagine:

- Inserire la slide con triangolo front-facing vs back-facing.

## 9) Efficienza e casi sfavorevoli

Osservazioni dal corso:

- triangoli lunghi e stretti possono avere bounding box enorme e testare molti pixel inutili;
- triangoli simili ma traslati possono generare insiemi di frammenti diversi;
- un triangolo può generare 0, pochi o moltissimi frammenti.

Suggerimento immagine:

- Inserire la slide con i triangoli lunghi e stretti "worst case".

## 10) Altre primitive: linee e punti

Le GPU moderne rasterizzano nativamente:

- triangoli (3 vertici),
- segmenti/linee (2 vertici),
- punti (1 vertice, point splat).

Note:

- il vertex processing e il fragment processing sono gli stessi per tutte le primitive;
- cambia solo la fase di rasterizzazione.

Linea: algoritmo storico di Bresenham (1962)

- usa solo aritmetica intera, molto efficiente;
- ma è intrinsecamente sequenziale, quindi poco parallelizzabile.

Suggerimento immagine:

- Inserire la slide su Bresenham e quella su wireframe.

## 11) Antialiasing

Problema:

- il rasterizer classico è binario (tutto o niente), generando aliasing a gradini.

Antialiasing (AA):

- tecniche che producono frammenti parziali (semi-trasparenti) vicino ai bordi;
- riducono l'effetto scalettatura su triangoli e linee.

![image.png](Rasterizing%20primitives/image%204.png)

## Riepilogo essenziale

- Il rasterizer trasforma primitive 2D in frammenti (pixel candidati).
- Si passa da clip space a screen space, poi si rasterizza per pixel interi.
- I triangoli si rasterizzano con bounding box e test di appartenenza.
- Gli attributi si interpolano con coordinate baricentriche.
- Culling e clipping evitano lavoro inutile e frammenti fuori viewport.
- Linee e punti sono supportati, ma il triangolo resta la primitiva dominante.
- L'antialiasing riduce i bordi a gradini.

## Domande guida per ripasso

1. Perché la rasterizzazione è considerata una fase 2D?
2. Qual è il mapping standard da clip space a screen space?
3. Come funziona il test di appartenenza con i semipiani?
4. A cosa servono le coordinate baricentriche?
5. Qual è la regola per gestire i frammenti sui bordi?
6. Quando ha senso il back-face culling?
7. Perché i triangoli lunghi e stretti sono un caso sfavorevole?
8. Qual è il limite principale dell'algoritmo di Bresenham nelle GPU moderne?
9. Cosa cambia tra rasterizzazione binaria e antialiasing?