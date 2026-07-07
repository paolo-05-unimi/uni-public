# Approcci al rendering

# Ray tracing e rasterization

## In breve

- differenza di paradigma tra ray-tracing e rasterization;
- definizione di raggio primario e raggi secondari;
- concetto di primitiva di rendering e perché il triangolo domina;
- pipeline rasterization-based e sue fasi;
- complessità, parallelismo implicito e colli di bottiglia;
- dibattito storico e stato attuale (ibridi, real-time ray tracing).

## 1) Inquadramento: due paradigmi di rendering

Due famiglie principali di algoritmi:

- Ray-tracing based: per ogni pixel, lancio un raggio e cerco intersezioni.
- Rasterization based: per ogni primitiva, proietto e rasterizzo in pixel.

Messaggio chiave:

- i due paradigmi differiscono per ordine dei loop e quindi per costi e ottimizzazioni.

![image.png](Approcci%20al%20rendering/image.png)

## 2) Ray casting: idea base (backward)

Concetto principale:

- si seguono a ritroso i fotoni che arrivano al POV (backward ray tracing).

Procedura per pixel:

1. genera un raggio primario dal POV al pixel;
2. trova le intersezioni con le primitive di scena;
3. seleziona l'intersezione più vicina al POV;
4. assegna il colore del punto colpito al pixel.

Pseudo-codice essenziale (dal corso):

```
For each pixel p:
  make a ray r (POV to p)
  for each primitive o in scene:
    find intersect(r, o)
  keep closest intersection
  find color at p
```

![image.png](Approcci%20al%20rendering/image%201.png)

## 3) Primitive di rendering

Definizione:

- una primitiva di rendering è una descrizione che un algoritmo può processare direttamente.

Conseguenze pratiche:

- se la primitiva è il triangolo, una quad-mesh va prima triangolata;
- se l'algoritmo supporta campi di altezza o modelli impliciti, non serve convertirli.

Per il ray casting:

- posso renderizzare qualsiasi cosa per cui so calcolare intersezione raggio-primitiva.

Esempi citati:

- tri-mesh (caso dominante), quad-mesh, height fields, modelli impliciti, superfici parametriche.

## 4) Ray tracing: raggi secondari ed effetti

Estensione rispetto al ray casting:

- dopo il raggio primario, si generano raggi secondari per simulare effetti ottici.

Tipi di raggio secondario:

- shadowing (ombre portate);
- riflessione speculare;
- rifrazione e semitrasparenze.

Equazione standard di un raggio:

$$
r(t)=O+t\,d,\quad t>0
$$

dove $O$ è il POV e $d$ è la direzione associata al pixel.

![image.png](Approcci%20al%20rendering/image%202.png)

![image.png](Approcci%20al%20rendering/image%203.png)

![image.png](Approcci%20al%20rendering/image%204.png)

## 5) Complessità e parallelismo nel ray tracing

Stima semplice (dal corso):

- scena con $M$ primitive, immagine $N\times N$, $K$ raggi per pixel:

$$
N^2\,M\,K\ \text{intersezioni}
$$

- complessità temporale: $O(N^2 M K)$.

Conseguenze:

- costo elevato per immagini grandi e molti effetti;
- rendering tipicamente offline, ma altamente parallelizzabile (pixel indipendenti);
- ottimizzazioni fondamentali per ridurre le primitive testate.

## 6) Varianti della famiglia ray-based

Dalla lezione:

- Ray-casting: un solo raggio primario per pixel.
- Ray-tracing: raggi secondari per fenomeni successivi.
- Path-tracing: molti raggi per pixel, sampling Monte Carlo, media dei risultati.
- Ray-marching: spezza il raggio in passi e testa solo primitive vicine.

## 7) Modelli di riflessione della luce (semplificati)

Tre modelli qualitativi:

- speculare: riflessione come pallina da ping-pong;
- diffusa: riflessione in tutte le direzioni;
- glossy: distribuzione concentrata attorno alla direzione speculare.

![image.png](Approcci%20al%20rendering/image%205.png)

## 8) Ray tracing su modelli già visti

Esempi dal corso:

- ray tracing di un volume di voxel (direct volume rendering);
- ray tracing di una tri-mesh con intersezione raggio-triangolo.

Nota tecnica (tri-mesh):

- si cerca l'intersezione di tipo $\mathbf{p}+k\mathbf{d}$ con ogni triangolo,
- si mantiene il $k$ minimo (punto piu vicino).

Limite pratico:

- generalità elevata, ma costo alto se la mesh è molto densa;
- discretizzazione introduce errori di approssimazione.

![image.png](Approcci%20al%20rendering/image%206.png)

## 9) Rasterization-based: idea base

Procedura per primitiva:

1. proietta la primitiva sullo schermo (3D -> 2D);
2. rasterizza la forma 2D in pixel;
3. calcola il colore dei pixel (lighting).

Supporto hardware:

- la GPU è stata progettata per la rasterizzazione;
- la primitiva quasi unica supportata è il triangolo;
- motivo della predominanza delle tri-mesh in real-time rendering.

![image.png](Approcci%20al%20rendering/image%207.png)

## 10) Rasterizzazione: frammenti e pixel

Concetto chiave:

- rasterizzare un triangolo significa generare frammenti, uno per ogni pixel coperto.

Distinzione utile:

- frammento = pacchetto dati con attributi interpolati;
- pixel = colore RGB finale dopo il lighting.

![image.png](Approcci%20al%20rendering/image%208.png)

## 11) Pipeline rasterization-based

Pipeline tipica (T&L = Transform & Lighting):

1. Fase per vertice: trasformazioni spaziali (3D -> 2D).
2. Fase per triangolo: rasterizzazione in frammenti.
3. Fase per frammento: calcolo del colore RGB finale.

Parallelismo:

- le fasi sono in cascata ma lavorano in parallelo;
- il collo di bottiglia determina la velocità complessiva.

Terminologia:

- transform-limited (geometry-limited): troppa geometria da trasformare;
- fill-limited: troppi pixel/fragment da riempire.

![image.png](Approcci%20al%20rendering/image%209.png)

## 12) Confronto sintetico tra i paradigmi

Sintesi "for each":

- Ray-tracing: for each pixel → for each primitive.
- Rasterization: for each primitive → for each pixel.

Vantaggi rasterization (dal corso):

- complessità lineare con numero di primitive;
- processa solo i pixel effettivamente coperti;
- supporto GPU pieno e standard.

![image.png](Approcci%20al%20rendering/image%2010.png)

## 13) Il dibattito storico e i luoghi comuni

Visione tradizionale:

- Ray-tracing: lento ma di alta qualità, usato per rendering offline.
- Rasterization: veloce ma approssimato, usato per real-time.

Esempi citati:

- Ray-tracing: POV-Ray, Renderman, YafaRay, Mitsuba.
- Rasterization API: OpenGL, WebGL, DirectX, Metal, Vulkan.

## 14) La realtà attuale: convergenza e ottimizzazioni

Punti chiave:

- il ray-tracing è parallelizzabile (render farms);
- primitive più espressive riducono il numero di primitive;
- strutture dati e ottimizzazioni riducono i test per raggio;
- real-time ray tracing sta diventando comune (Unity/Unreal + GPU recenti).

Allo stesso tempo:

- la rasterization può riprodurre effetti complessi con tecniche dedicate (ombre, riflessioni, rifrazioni, global illumination approssimata).

![Pixar/Disney Render Farm](Approcci%20al%20rendering/6f30e2ab-a491-46ed-9937-67b9d80be927.png)

Pixar/Disney Render Farm

## 15) Stato dell'industria (film vs videogame)

Sintesi:

- cinema: ray-tracing per rendering finale, rasterization per preview e asseting;
- videogame: rasterization dominante, con trend verso approcci ibridi e ray-tracing.

![image.png](Approcci%20al%20rendering/image%2011.png)

## 16) Riassunto rapido

1. Ray-tracing: per ogni pixel, lancia un raggio e cerca intersezioni.
2. Rasterization: per ogni primitiva, proietta e rasterizza in pixel.
3. Il triangolo è la primitiva dominante per il real-time.
4. Il ray-tracing gestisce naturalmente ombre, riflessioni e rifrazioni.
5. Il rasterization è supportato in hardware e molto efficiente.
6. Entrambi i paradigmi sono parallelizzabili e oggi spesso ibridati.

## 17) Domande guida per ripasso

1. Qual è la differenza principale tra ray-tracing e rasterization nei loop base?
2. Perché la tri-mesh è la rappresentazione più usata nel rendering real-time?
3. Cosa sono raggio primario e raggio secondario?
4. Come si distingue un frammento da un pixel?
5. Cosa significa che un'applicazione è fill-limited?
6. Quali ottimizzazioni rendono possibile il ray-tracing in real-time?
7. In che senso oggi il dibattito tra i due paradigmi si sta chiudendo?