# Subdivision Surfaces

# Lez005.1 + Lez005.2

## 1) Inquadramento: dove stanno le subdivision surfaces

Nella categorizzazione dei modelli 3D del corso, le subdivision surfaces sono una rappresentazione **superficiale continua**.

Messaggio chiave della lezione:

- si parte da una rappresentazione discreta (control polyline o control mesh);
- si applica iterativamente uno schema di suddivisione;
- si converge a una **curva/superficie limite** smooth.

![image.png](Subdivision%20Surfaces/image.png)

## 2) Parte A - Curve di suddivisione

### 2.1 Idea base

Una curva può essere approssimata da una linea spezzata (2D o 3D), cioè una sequenza di vertici connessi da segmenti.

Suddividere la spezzata significa:

- creare nuovi vertici dai vecchi;
- sostituire ogni segmento con segmenti più piccoli;
- ripetere il passo con una regola fissa (schema).

Effetto:

- aumenta la risoluzione;
- la spezzata diventa progressivamente più smooth;
- iterando, si approssima la curva limite.

### 2.2 Esempio didattico di schema duale (corner cutting)

Nelle prime slide viene mostrato uno schema in cui:

- per ogni segmento si introducono due nuovi vertici (pesi 1/4, 1/2, 1/4 nel caso mostrato);
- i vecchi vertici vengono scartati;
- la nuova spezzata "taglia gli angoli" (corner cutting).

Esempio preso dalle slide:

- stessa procedura su curva chiusa, ripetuta per più passi, con comparsa visiva della curva limite.

![image.png](Subdivision%20Surfaces/image%201.png)

### 2.3 Classi di schemi per curve

La lezione distingue tre famiglie:

- **Duali**: i vertici originali vengono eliminati.
- **Interpolativi**: i vertici originali restano fermi; la curva passa per essi.
- **Approssimativi**: i vertici originali vengono spostati verso medie dei vicini; la curva passa vicino ai punti di controllo.

Esempio preso dalle slide:

- nello schema approssimativo, prima si inseriscono vertici sui segmenti, poi si "smoothano" i vecchi vertici verso i vicini.

### 2.4 2D vs 3D e propagazione degli attributi

Gli stessi schemi valgono sia per curve in 2D sia per curve in 3D (one-manifold embedded in spazio 2D/3D).

Punto importante:

- se ci sono attributi per vertice (es. colore), si aggiornano con la stessa interpolazione/estrapolazione usata per le posizioni.

## 3) Dalle curve alle superfici: subdivision di mesh

### 3.1 Definizione operativa

Suddividere una mesh significa produrre automaticamente una mesh più densa e più smooth, spezzando ciascun poligono in poligoni più piccoli.

Ogni schema definisce due componenti:

- **Connettività**: quali vertici nuovi creare e come connetterli.
- **Geometria**: come calcolare coordinate (e attributi) dei nuovi vertici e, negli schemi approssimativi, come spostare i vecchi.

### 3.2 Curva/superficie limite

Iterando il processo, si converge a una superficie limite.

Le differenze tra schemi riguardano:

- interpolativo vs approssimativo vs duale;
- tipo di mesh richiesta in input (tri, quad, poligonale generica);
- tipo di mesh prodotta in output;
- smoothness finale (continuità di normale/curvatura).

## 4) Schemi per tri-mesh: Butterfly e Loop

### 4.1 Pattern topologico comune: 1 -> 4

Per gli schemi tri-mesh mostrati, ogni passo:

- aggiunge un vertice su ogni edge;
- connette i nuovi vertici;
- divide ogni triangolo in 4 triangoli.

Incremento risoluzione (stima media ricordata in slide):

- da $V\sim n$, $F\sim 2n$, $E\sim 3n$
- a $V' = V+E \sim 4n$, $F' = 4F \sim 8n$, $E' = 2E+3F \sim 12n$
- quindi risoluzione circa x4 ad ogni passo.

### 4.2 Schema Butterfly (interpolativo)

Caratteristiche:

- tri-mesh → tri-mesh;
- schema 1 → 4;
- interpolativo (la superficie limite passa per i vertici di controllo).

Nelle slide appare la classica "maschera butterfly" con pesi locali (inclusi contributi negativi -1/16) per il nuovo vertice su edge.

Esempio preso dalle slide:

- risultato "molto smooth", interpolante, in esempio quasi sferico dopo passi successivi.

![image.png](Subdivision%20Surfaces/image%202.png)

### 4.3 Schema Loop (approssimativo)

Caratteristiche:

- tri-mesh → tri-mesh;
- stessa connettività 1→ 4 del Butterfly;
- approssimativo.

Nello schema Loop:

- i nuovi vertici sono inseriti su interpolazioni (nel caso mostrato, a metà edge);
- i vecchi vertici vengono riposizionati con pesi sui vicini;
- la superficie limite **non** passa per i vertici di controllo iniziali.

![image.png](Subdivision%20Surfaces/image%203.png)

![image.png](Subdivision%20Surfaces/image%204.png)

![image.png](Subdivision%20Surfaces/image%205.png)

## 5) Schemi per mesh poligonali generiche

### 5.1 Doo-Sabin (duale)

Caratteristiche:

- schema duale per mesh poligonali generiche;
- produce una nuova mesh poligonale;
- costruisce nuovi vertici nelle facce e genera facce associate a vertici, edge e facce originali.

Esempio preso dalle slide:

- caso con vertici A (valenza 3), B (valenza 4) e edge C, mostrando la corrispondenza nella mesh risultante.

![image.png](Subdivision%20Surfaces/image%206.png)

### 5.2 Catmull-Clark (approssimativo)

Caratteristiche:

- pensato per quad mesh (pure-quad o quad-dominant), ma applicabile a mesh poligonali generiche;
- molto usato in pratica;
- dopo il primo passo tende a produrre pure-quad mesh.

Connettività (come mostrato in sequenza slide):

- nuovo vertice su ogni edge;
- nuovo vertice dentro ogni faccia;
- connessioni tra nuovi vertici che spezzano le facce;
- risultato: solo quadrilateri.

Geometria:

- nuovi vertici (edge/face points) da interpolazioni;
- vecchi vertici riposizionati in base ai vicini (e alla valenza).

Esempio preso dalle slide:

- base mesh mista (tri, quad, pentagono) trasformata in pure-quad dopo un passo.

![image.png](Subdivision%20Surfaces/image%207.png)

![image.png](Subdivision%20Surfaces/image%208.png)

![image.png](Subdivision%20Surfaces/image%209.png)

![image.png](Subdivision%20Surfaces/image%2010.png)

Nota: ogni vertice «blu» viene connesso con uno «rosso»

![image.png](Subdivision%20Surfaces/image%2011.png)

![image.png](Subdivision%20Surfaces/image%2012.png)

![image.png](Subdivision%20Surfaces/image%2013.png)

## 6) Proprietà comuni evidenziate in Lez005.2

Per gli schemi trattati in aula:

- il numero di facce cresce tipicamente di fattore 4 per passo;
- i vertici regolari restano regolari, gli irregolari restano irregolari;
- i nuovi vertici interni tendono a essere regolari;
- una mesh suddivisa diventa almeno semi-regolare;
- two-manifoldness, orientabilità e chiusura vengono preservate se presenti in input.

Nota: la lezione specifica che il fattore 4 non vale per *tutti* gli schemi esistenti, ma vale per quelli analizzati.

## 7) Bordi aperti, edge hard e creases

### 7.1 Mesh aperte

Gli schemi possono essere applicati anche a mesh aperte.

Dettaglio importante:

- i vertici/edge di bordo usano formule dedicate (non approfondite nelle slide per brevità).

### 7.2 Edge hard

Il modellatore può taggare edge interni come "hard", trattandoli come bordo.

Conseguenza:

- compaiono creases (feature lines) nella superficie limite;
- aumenta il controllo artistico sulle discontinuità volute.

![image.png](Subdivision%20Surfaces/24d78d70-955c-4140-a4e7-a6af41948e84.png)

## 8) Confronto tra schemi e scelta pratica

Le slide mostrano un confronto diretto tra:

- Doo-Sabin;
- Catmull-Clark;
- Loop;
- Butterfly.

Indicazione metodologica importante:

- evitare accoppiamenti impropri tra schema e tipologia di mesh (es. diagonal split + schema non pensato per quel dominio), segnalati nelle slide come "bad idea".

![image.png](Subdivision%20Surfaces/image%2014.png)

catmull clark e butterfly non sono pensati per triangle meshes ottenute con diagonal splits

## 9) Due usi pratici delle subdivision surfaces

### 9.1 Rappresentazione di superficie smooth

La base mesh vale come control mesh della limit surface.

Workflow:

- l'utente edita una mesh low-res semplice da manipolare;
- il sistema la suddivide (anche solo in rendering) per visualizzare la superficie smooth.

### 9.2 Strategia di modellazione coarse-to-fine

Workflow didattico mostrato:

1. modellare low-poly;
2. suddividere;
3. ritoccare;
4. ripetere.

Questo approccio aiuta a controllare prima la forma globale, poi i dettagli locali.

![image.png](Subdivision%20Surfaces/image%2015.png)

## 10) Collegamento con rendering GPU

Catmull-Clark è particolarmente rilevante anche per motivi di pipeline:

- suddivisione "al volo" fino al livello richiesto;
- base mesh memorizzata, mesh suddivisa visualizzata;
- approccio noto come Dynamic Hardware Tessellation.

Questo spiega perché le subdivision surfaces sono centrali non solo in modellazione, ma anche in visualizzazione efficiente.

## 11) Esempi chiave da ricordare (presi dalle due lezioni)

- Curve: schema duale con pesi 1/4-1/2-1/4 e corner cutting.
- Curve: differenza netta interpolativo vs approssimativo sui vertici originali.
- Tri-mesh: pattern topologico 1->4 condiviso da Butterfly e Loop.
- Butterfly: schema interpolativo con maschera locale e contributi negativi.
- Loop: schema approssimativo con stessi split topologici del Butterfly.
- Doo-Sabin: schema duale per mesh poligonali generiche.
- Catmull-Clark: da mesh mista a pure-quad già dal primo passo.
- Edge hard: controllo esplicito delle creases sulla superficie limite.
- Uso operativo: control mesh editable + raffinamento iterativo coarse-to-fine.

## 12) Conclusione

Lez005 introduce le subdivision surfaces come ponte tra:

- semplicità di editing di una mesh discreta low-res;
- qualità visiva di una superficie curva smooth.

Il cuore della dispensa e distinguere:

- comportamento dei vertici originali (duale/interpolativo/approssimativo);
- dominio topologico dello schema (tri, quad, poligonale);
- proprietà risultanti (regolarità, continuità, preservazione topologica).

In pratica, la scelta dello schema dipende dal tipo di mesh di partenza, dagli obiettivi di smoothness e dal controllo artistico desiderato.

## 13) Domande guida per ripasso

1. Qual e la differenza concettuale tra schema duale, interpolativo e approssimativo?
2. Perché Butterfly e Loop hanno stessa connettività ma risultato geometrico diverso?
3. In che senso Catmull-Clark "quad-rende" una mesh poligonale generica?
4. Quali proprietà topologiche vengono preservate dalla suddivisione?
5. Quando conviene usare edge hard in una pipeline di modellazione?
6. Come cambia il workflow tra uso "in rendering" e uso "coarse-to-fine" in modellazione?