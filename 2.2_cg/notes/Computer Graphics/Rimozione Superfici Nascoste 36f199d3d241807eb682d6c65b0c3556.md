# Rimozione Superfici Nascoste

## In breve

- problema dell'occlusione e differenze tra ray tracing e rasterizzazione;
- approccio di ordinamento (algoritmo del pittore) e suoi limiti;
- back-face culling: ipotesi, posizione nel pipeline, vantaggi e limiti;
- depth buffer e depth test: algoritmo, vantaggi, costi e implicazioni HW;
- precisione, z-fighting e scelte pratiche di zNear e zFar;
- note pratiche su abilitazione nelle API e librerie.

## 1) Problema: rimozione delle superfici nascoste

Obiettivo: gli oggetti più vicini al punto di vista devono coprire quelli più lontani.

Confronto:

- Ray tracing: per ogni raggio primario si prende l'intersezione con distanza minima.
- Rasterizzazione: si produce una sequenza di frammenti che possono sovrascriversi.

Problema specifico della rasterizzazione:

- se i frammenti di una primitiva sovrascrivono quelli già presenti nel buffer,
il risultato finale dipende dall'ordine di rasterizzazione;
- per essere corretto, l'ordine dovrebbe essere back-to-front.

## 2) Ordinamento back-to-front: algoritmo del pittore

Idea base:

- ordinare le primitive dalla più lontana alla più vicina (back-to-front);
- rasterizzare in sequenza lasciando che ogni frammento sovrascriva il buffer.

Dettaglio sull'ordinamento per Z:

- in spazio vista la Z decresce con la distanza dalla camera;
- in spazio clip o schermo la depth cresce con la distanza (depth in `[0,1]`).

Costo computazionale:

$$

T_{sort} = O(n \log n)

$$

con $n$ numero di primitive.

![image.png](Rimozione%20Superfici%20Nascoste/image.png)

## 3) Limiti dell'ordinamento su Z

Ostacoli principali:

- le coordinate in spazio vista si conoscono solo dopo le trasformazioni;
- una primitiva non ha una singola Z (ogni vertice ha Z diversa);
- non sempre esiste un ordine corretto (intersezioni e cicli di occlusione);
- costo di ordinamento non lineare e dipendenza dall'ordine di rendering.

Conseguenza:

- l'algoritmo del pittore si usa raramente, solo quando l'ordine è semplice
da determinare in preprocessing (es. height field).

![image.png](Rimozione%20Superfici%20Nascoste/image%201.png)

## 4) Back-face culling: caso particolare

Caso valido se:

- mesh chiusa, ben orientata, opaca;
- camera esterna alla superficie.

Idea:

- scartare le facce back-facing prima della rasterizzazione.

Test tipico (orientamento):

$$

\text{cull se } (\mathbf{n} \cdot \mathbf{v}) > 0

$$

Dove $\mathbf{n}$ è la normale della faccia e $\mathbf{v}$ è la direzione di vista.
Il segno dipende dalla convenzione di orientamento.

Dove si applica:

- nella fase di set-up del triangolo, prima della rasterizzazione;
- non generalizzabile a punti o segmenti.

Vantaggi e limiti:

- riduce circa il 50% dei frammenti in scene chiuse e opache;
- non risolve occlusioni in superfici concave.

![image.png](Rimozione%20Superfici%20Nascoste/image%202.png)

## 5) Depth buffer: idea e struttura dati

Approccio order independent:

- per ogni pixel si mantiene la profondità minima vista finora.

Struttura:

- depth buffer e screen buffer hanno la stessa risoluzione;
- per ogni pixel `[x,y]` si memorizza il depth del frammento più vicino.

Algoritmo del depth test:

```
if (depth <= DepthBuffer[x][y]) {
  ScreenBuffer[x][y] = (R,G,B);
  DepthBuffer[x][y] = depth;
} else {
  discard fragment;
}
```

![image.png](Rimozione%20Superfici%20Nascoste/image%203.png)

## 6) Depth: range e mapping da clip a schermo

Definizioni:

- depth è un valore in $[0,1]$;
- $0$ è vicino, $1$ è lontano.

Mapping tipico dalla Z in spazio clip:

$$

\text{depth} = \frac{z_{clip} + 1}{2}

$$

Interpolazione:

- la depth è interpolata sui frammenti tramite coordinate baricentriche,
come gli altri attributi di vertice.

![image.png](Rimozione%20Superfici%20Nascoste/image%204.png)

## 7) Vantaggi e costi del depth test

Vantaggi:

- rendering order independent;
- adatto a implementazione parallela in HW;
- corretto su geometrie complesse e con cicli di occlusione.

Costi e limiti:

- memoria addizionale per depth buffer;
- il buffer va inizializzato (depth massima) a ogni frame;
- il test avviene tardi nel pipeline (parte del lavoro può essere sprecato);
- assume superfici opache (problemi con trasparenze);
- accesso in lettura/scrittura a memoria condivisa.

Ottimizzazione utile:

- il depth sorting resta utile solo come ottimizzazione front-to-back
per scartare prima i frammenti lontani.

## 8) Precisione e z-fighting

Quantizzazione:

- con b bit, la depth è quantizzata in $2^b$ livelli.

Esempio (8 bit):

$$

\text{depth} = \frac{d}{2^8 - 1}, \quad d \in \{0, \dots, 255\}

$$

Problema:

- superfici parallele molto vicine possono risultare con depth identica;
- l'immagine alterna le due superfici: z-fighting.

Soluzione parziale:

- usare almeno 16 bit (o più), ma il problema non scompare del tutto.

![z-fighting](Rimozione%20Superfici%20Nascoste/image%205.png)

z-fighting

## 9) Depth image come effetto collaterale

Ogni rendering produce:

- color buffer (RGB) mostrato a schermo;
- depth buffer (profondità per pixel) normalmente scartato.

Interpretazione:

- la depth image è un bassorilievo della scena;
- equivale a una range scan virtuale e può essere riutilizzata da altri algoritmi.

![image.png](Rimozione%20Superfici%20Nascoste/image%206.png)

## 10) Abilitazione nelle API e in three.js

Note pratiche:

- il depth test è in HW; il programmatore lo abilita o disabilita;
- nelle API low-level è responsabilita del programmatore azzerare
il depth buffer a inizio frame (depth = 1.0).

Esempio in three.js:

- `material.depthTest = true` (default).

## 11) Scelta di zNear e zFar

Considerazioni:

- zNear troppo piccolo o zFar troppo grande riducono la precisione;
- precisione bassa aumenta lo z-fighting;
- zFar > zNear > 0 sempre.

Esempio pratico:

- "inquadro da 0.2 m a 50 m" => zNear = 0.2, zFar = 50.

## Riepilogo essenziale

- L'occlusione è banale nel ray tracing, ma non nella rasterizzazione.
- L'algoritmo del pittore richiede ordinamento back-to-front e ha limiti pratici.
- Il back-face culling è una ottimizzazione valida solo in casi specifici.
- Il depth buffer rende il rendering order independent con un test per frammento.
- La precisione della depth è finita: z-fighting è sempre possibile.
- La scelta di zNear/zFar influisce direttamente sulla precisione.

## Domande guida per ripasso

1. Perché la rasterizzazione non garantisce da sola l'occlusione corretta?
2. Qual e' l'idea dell'algoritmo del pittore e perché fallisce in alcuni casi?
3. In quali condizioni è valido il back-face culling?
4. In che punto del pipeline si esegue il depth test e con quale regola?
5. Come si mappa la Z in clip space nel range di depth $[0,1]$?
6. Perché lo z-fighting compare anche con depth buffer a 16 bit?
7. Perché l'ordine front-to-back resta utile anche con il depth test?
8. Qual è l'effetto di zNear piccolo e zFar grande sulla precisione?