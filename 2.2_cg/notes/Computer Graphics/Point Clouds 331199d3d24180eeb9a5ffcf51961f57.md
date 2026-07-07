# Point Clouds

## Point Clouds in breve

- cosa sono le point cloud e come si collocano tra i modelli 3D;
- come si memorizzano e si renderizzano;
- come si acquisiscono (con focus su fotogrammetria);
- quali problemi pratici presentano;
- quali task di geometry processing sono più comuni.

## 1) Inquadramento: dove stanno le point cloud nei modelli 3D

Le point cloud sono una rappresentazione **superficiale** del dato 3D: descrivono il boundary dell'oggetto tramite campioni discreti.

Nel panorama dei modelli 3D, si collocano tra le rappresentazioni meno strutturate:

- piu semplici da acquisire e da memorizzare;
- meno comode da usare direttamente per editing, simulazione e rendering avanzato;
- spesso punto di partenza per conversione verso modelli piu strutturati (es. mesh poligonali).

## 2) Point cloud: definizione e proprietà

Una point cloud (o point-set) e un insieme di campioni indipendenti della superficie.

Per ogni campione si memorizza tipicamente:

- posizione 3D del punto $\mathbf{p}$;
- normale unitaria locale $\mathbf{n}$ (quando disponibile);
- eventuali attributi (colore, proprietà di materiale, misure fisiche, ecc.).

Proprieta chiave:

- non esistono relazioni esplicite di adiacenza tra campioni;
- l'ordine dei punti e irrilevante;
- può contenere da migliaia a milioni/miliardi di campioni;
- i dati acquisiti reali sono spesso rumorosi e possono contenere outlier.

## 3) Vettori normali: ruolo geometrico

La normale descrive l'orientamento locale della superficie in un punto.

- una superficie piana ha normale costante;
- una superficie curva ha normale variabile punto per punto.

Interpretazione utile: una point cloud e un campionamento di coppie posizione-orientamento, quindi può essere vista come collezione di piccoli "surfel" (surface elements orientati), non solo punti isolati.

## 4) Memorizzazione (storage)

Formato esempio: file `.xyz`.

Ogni riga puo contenere:

- coordinate del punto: $(x,y,z)$;
- opzionalmente la normale: $(n_x,n_y,n_z)$;
- in altri formati anche attributi extra.

Vantaggio: estrema semplicità.
Limite: nessuna connettività esplicita (a differenza delle mesh).

## 5) Rendering: point splatting

Per visualizzare una point cloud non basta disegnare punti infinitesimi, perché il risultato sarebbe spesso discontinuo e poco leggibile. La lezione introduce quindi il **point splatting**:

- ogni punto viene renderizzato tramite uno **splat**, cioè una piccola regione di pixel sullo schermo;
- la dimensione dello splat dipende dalla densità dei punti e quindi dalla loro distanza media;
- gli splat possono essere rappresentati anche come piccoli dischi, interpretabili come frammenti locali di superficie.

Questo approccio consente di ottenere una visualizzazione più continua e più vicina all'idea di superficie, pur partendo da dati non connessi.

## 6) Multi-risoluzione

Dato che i punti sono indipendenti, la multi-risoluzione è naturale:

- si prende un sottoinsieme dei campioni;
- se l'ordine è progettato bene, i primi $M$ punti restano ben distribuiti per ogni $M < N$;
- una permutazione casuale è spesso una buona approssimazione pratica;
- in rendering si usano solo i primi $M$ punti, regolando anche la dimensione degli splat.

Effetto: controllo semplice del compromesso qualità-prestazioni.

## 7) Acquisizione 3D delle point cloud

Tecniche citate:

- laser scanning;
- time-of-flight;
- fotogrammetria (Shape From Motion), trattata in dettaglio.

### Fotogrammetria: idea di base

Da molte fotografie dell'oggetto/scena si ricostruiscono:

- posizione delle camere;
- posizioni 3D dei punti osservati.

Vantaggi:

- costi bassi;
- hardware semplice (anche camera standard);
- ampia scalabilità (dal macro alla microscopia).

Limiti:

- sensibilità a rumore/incompletezza;
- difficolta con superfici trasparenti, lucide o riflettenti;
- produce nativamente point cloud, spesso da convertire poi in mesh.

### Pipeline fotogrammetrica

La pipeline si articola nei seguenti passaggi:

1. **Cattura delle immagini**: si scattano molte fotografie dell'oggetto, preferibilmente con illuminazione buona e costante.
2. **Copertura multi-vista**: ogni punto della superficie deve essere visibile da più punti di vista, altrimenti non può essere ricostruito.
3. **Identificazione dei feature point**: in ogni immagine si cercano dettagli locali facilmente riconoscibili anche nelle altre immagini.
4. **Matching dei feature point**: si determinano le corrispondenze tra feature osservate in immagini diverse.
5. **Ricostruzione globale**: si stimano simultaneamente:
    - le pose delle camere;
    - le posizioni 3D dei punti della superficie.
6. **Verifica di consistenza**: le proiezioni dei punti 3D sulle immagini devono essere coerenti con i match osservati.

Una pipeline più completa da fotografie a modello 3D pronto per il web:

- campagna fotografica;
- fotogrammetria;
- point cloud acquisita;
- geometry processing;
- mesh 3D pulita con tessiture;
- applicazione web per il rendering del modello.

## 8) Software e strumenti citati

Tra gli strumenti menzionati compaiono:

- **Meshroom**, **Regard3D**, **Metashape** per la fotogrammetria;
- **Sketchfab** e **Real-World Textured Things** come repository o fonti di modelli acquisiti;
- **Point Cloud Library (PCL)** come libreria C++;
- **MeshLab** e **CloudCompare** come software per il geometry processing.

MeshLab è orientato soprattutto alle mesh, ma include filtri per point set; CloudCompare è invece specializzato maggiormente sulle point cloud.

## 9) Limiti tipici delle point cloud acquisite

Problemi ricorrenti:

- campionamento inadeguato (troppo fitto, troppo rado, non uniforme);
- rumore su posizioni e/o normali;
- outlier (punti spurii fuori superficie);
- incompletezza (occlusioni, zone non viste, materiali difficili).

## 10) Geometry processing su point cloud: panoramica

Task comuni:

- denoising (riduzione rumore);
- outlier removal;
- completamento parti mancanti;
- ri-campionamento/cambio risoluzione;
- stima delle normali quando assenti;
- registrazione/allineamento di più nuvole;
- surface reconstruction (conversione in mesh, argomento successivo).

Prima di molti di questi task emerge un sotto-problema fondamentale: definire il vicinato di ciascun punto.

## 11) Sotto-problema fondamentale: definire il vicinato

<aside>

[Norma di un vettore](01%20-%20Punti%20e%20Vettori%20331199d3d2418003877ccee018dd3777.md) 

</aside>

Molti algoritmi richiedono di conoscere i vicini di un punto.

### Definizione con raggio

Dato un punto $\mathbf{p}_i$, i vicini sono i punti $\mathbf{p}_j$ tali che:

$$
\lVert \mathbf{p}_j - \mathbf{p}_i \rVert < d

$$

Criticità: la soglia $d$ dipende da scala e densità del campionamento.

### Definizione k-NN (preferita)

Dato un punto, si scelgono i suoi $k$ nearest neighbors.

Perché è più robusta:

- meno sensibile alla scala assoluta;
- più stabile con densità non uniforme.

Costo computazionale:

- brute force: proibitivo su dataset grandi (fino a complessità quadratica sul totale);
- in pratica servono strutture dati/algoritmi efficienti (non dettagliati in lezione).

## 12) Stima delle normali

Quando le normali non sono disponibili:

1. per ogni punto, trova il suo vicinato k-NN;
2. stima il piano best-fitting dei vicini;
3. usa la normale del piano come normale del punto.

Note importanti:

- con $N>3$ punti non esiste in generale un piano che passi per tutti;
- si risolve un problema di best fitting;
- il verso della normale e ambiguo (due direzioni opposte), quindi va reso consistente globalmente.

## 13) Denoising e outlier removal

### Denoising

Obiettivo: ridurre gli errori di misura preservando il segnale geometrico.

Idea tipica: spostare i punti verso configurazioni localmente più lisce, coerenti con il piano locale stimato dal vicinato.

### Outlier removal

Problema diverso dal denoising:

- nel denoising i punti sono validi ma perturbati;
- negli outlier alcuni punti sono proprio errati e vanno identificati/rimossi.

## 14) Registrazione di più point cloud

Quando si acquisiscono porzioni diverse dello stesso oggetto, si ottengono spesso più nuvole, ciascuna nel proprio sistema di riferimento. Per unirle in una rappresentazione unica è necessario allinearle.

La registrazione viene distinta in due fasi:

1. **Coarse registration**: produce un allineamento iniziale approssimato.
2. **Fine registration**: raffina l'allineamento una volta che le superfici comuni sono già abbastanza vicine.

Per l'allineamento fine la lezione introduce l'algoritmo **ICP (Iterative Closest Points)**. Lo schema di base è:

1. scegliere un sottoinsieme di punti di una delle due nuvole;
2. trovare per ciascuno il punto più vicino sull'altra nuvola;
3. scartare le corrispondenze troppo lontane;
4. calcolare la rotazione e traslazione che avvicinano al meglio i punti ai rispettivi corrispondenti;
5. iterare fino a convergenza.

L'ICP funziona bene se si parte già vicino alla soluzione corretta; altrimenti può convergere a risultati sbagliati o non convergere affatto. Per questo la fase coarse e essenziale.

## 15) Conversione verso modelli più strutturati

Un messaggio importante della lezione è che le point cloud sono molto diffuse soprattutto per la facilità di acquisizione, non perché siano la rappresentazione più comoda da usare direttamente. Un task frequente consiste quindi nel convertirle in un'altra struttura dati più adatta a processing e rendering.

L'esempio tipico anticipato è la conversione **da point cloud a mesh poligonale**, tema delle lezioni successive.

## 16) Domande guida per ripasso

1. Perché una point cloud e detta rappresentazione superficiale ma non strutturata?
2. Perché il criterio k-NN e spesso preferito alla soglia metrica fissa $d$?
3. In cosa differiscono denoising e outlier removal?
4. Perché ICP richiede una buona inizializzazione (coarse registration)?
5. Perché la fotogrammetria e economica ma fragile su certi materiali?
6. Qual è il motivo principale per convertire point cloud in mesh?