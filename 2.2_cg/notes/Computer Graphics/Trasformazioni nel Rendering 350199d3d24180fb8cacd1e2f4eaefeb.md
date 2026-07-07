# Trasformazioni nel Rendering

# Pipeline di trasformazioni

## In breve

Riassumere in modo progressivo e operativo:

- spazi di riferimento nella pipeline (oggetto, mondo, vista, clip, screen);
- trasformazioni di modellazione, vista e proiezione;
- coordinate clip e NDC, clipping e culling;
- proiezione ortografica e prospettica;
- ruolo di FoV, distanza focale e view frustum;
- uso delle coordinate omogenee nella proiezione.

## 1) Visione di insieme: la sequenza di trasformazioni

Nel rendering rasterization-based, i vertici attraversano una sequenza di spazi:

- Spazio Oggetto → Spazio Mondo → Spazio Vista → Spazio Clip → Spazio Screen.

Le tre trasformazioni principali sono:

- Modellazione (Model Matrix): da oggetto a mondo.
- Vista (View Matrix): da mondo a vista.
- Proiezione (Projection Matrix): da vista a clip.

Composizione standard:

$$
M_{VP} = M_P M_V M_M
$$

Dove $M_M$ e la matrice di modellazione, $M_V$ quella di vista, $M_P$ quella di proiezione.

![image.png](Trasformazioni%20nel%20Rendering/image.png)

## 2) Spazio Oggetto

È il sistema di riferimento locale in cui è definito ogni modello 3D.
Include:

- posizioni dei vertici della mesh;
- normali e altri vettori locali;
- punti di controllo di patch;
- voxel o campi di altezza.

![image.png](Trasformazioni%20nel%20Rendering/image%201.png)

Ogni oggetto ha il suo spazio oggetto, definito dal modellatore o dal software.

## 3) Spazio Mondo e trasformazione di modellazione

Lo spazio mondo è il riferimento globale della scena, condiviso da tutti gli oggetti.
La trasformazione di modellazione porta un oggetto dal suo spazio locale allo spazio mondo.

Due modi equivalenti per costruire $M_M$:

- come assi e origine dello spazio oggetto espressi in spazio mondo;
- come composizione di trasformazioni affini (traslazioni, rotazioni, scalature).

Multi-instancing:

- una stessa mesh può essere disegnata più volte con matrici $M_M$ diverse;
- si memorizza una sola mesh e si ottengono molte istanze in scena.

![image.png](Trasformazioni%20nel%20Rendering/image%202.png)

## 4) Spazio Vista e trasformazione di vista

La trasformazione di vista porta tutto nello spazio della camera virtuale.
Dipende dai parametri estrinseci (posizione e orientamento della camera).

Caratteristica chiave:

- la matrice di vista è l'inversa della matrice di modellazione dell'oggetto camera.

Quindi, muovere la camera equivale a trasformare tutto il mondo in senso opposto.

![image.png](Trasformazioni%20nel%20Rendering/image%203.png)

## 5) Spazio Clip e NDC

Lo spazio clip e' allineato allo schermo ma e' ancora 3D.

Convenzione NDC:

- solo i punti con $x,y,z$ in $[-1,+1]$ sono visibili;
- gli oggetti fuori dal range vengono "culled";
- gli oggetti che intersecano i limiti vengono "clipped".

Questo e' il motivo storico del nome "clip space".

![image.png](Trasformazioni%20nel%20Rendering/image%204.png)

![image.png](Trasformazioni%20nel%20Rendering/image%205.png)

## 6) Trasformazione di proiezione: obiettivo e parametri

La proiezione porta da spazio vista a spazio clip. Dipende dai parametri intrinseci della camera:

- dimensioni del piano immagine $w, h$;
- distanza focale $d$ o Field of View (FoV).

![image.png](Trasformazioni%20nel%20Rendering/image%206.png)

Relazione tra FoV e $d$:

$$

FoV = 2 \arctan\left(\frac{h}{2d}\right),\quad
FoV_H = 2 \arctan\left(\frac{w}{2d}\right)

$$

Effetto qualitativo:

- FoV grande (d piccola) = grandangolo, forte distorsione;
- FoV piccolo (d grande) = teleobiettivo, prospettiva debole.

![image.png](Trasformazioni%20nel%20Rendering/image%207.png)

## 7) Proiezione ortografica

È la proiezione che ignora la profondità (nessuna prospettiva).

Matrice semplice (spazio vista → clip):

$$

P_{ortho}=
\begin{bmatrix}
1&0&0&0\\
0&1&0&0\\
0&0&0&0\\
0&0&0&1
\end{bmatrix}

$$

Conseguenze:

- linee parallele rimangono parallele;
- dimensione apparente indipendente dalla distanza;
- utile per viste tecniche e scene lontane.

![image.png](Trasformazioni%20nel%20Rendering/image%208.png)

## 8) Proiezione prospettica e coordinate omogenee

Nel modello pin-hole:

$$

(x_p, y_p, z_p)=\left(-d\,\frac{x}{z}, -d\,\frac{y}{z}, -d\right)

$$

Per esprimerla come matrice 4x4 servono coordinate omogenee.

Rappresentazione:

- punto: $(xw, yw, zw, w)$ con $w \neq 0$;
- vettore: $(x, y, z, 0)$.

Matrice prospettica (forma tipica):

$$

P=\begin{bmatrix}
d&0&0&0\\
0&d&0&0\\
0&0&d&-1\\
0&0&0&0
\end{bmatrix}

$$

Dopo la moltiplicazione, si divide per $w$ per tornare alle coordinate cartesiane.

Suggerimento immagine:

- Inserire la slide con la derivazione della proiezione prospettica.
- Inserire la slide sul recap delle coordinate omogenee.

## 9) View frustum e profondità

La proiezione deve mappare il volume visibile (view frustum) nel cubo NDC.

Conseguenze operative:

- i piani di clipping definiscono i limiti di visibilità;
- il range della profondità viene normalizzato in $[-1,+1]$;
- questo prepara il depth test nella rasterizzazione.

![image.png](Trasformazioni%20nel%20Rendering/image%209.png)

![image.png](Trasformazioni%20nel%20Rendering/1d819a54-6288-4f93-8140-f2f61c195325.png)

## 10) Spazio Screen (Viewport)

Dopo il clip space, le coordinate passano allo spazio schermo:

- viewport: rettangolo dello schermo occupato dal rendering;
- coordinate pixel intere: $x$ da 0 a risoluzione orizzontale, $y$ da 0 a risoluzione verticale;
- $z$ diventa depth in $[0,1]$.

![image.png](Trasformazioni%20nel%20Rendering/image%2010.png)

## 11) Dettaglio tecnico: cosa produce il vertex processing

![image.png](Trasformazioni%20nel%20Rendering/image%2011.png)

La fase per-vertice produce coordinate clip omogenee:

$$

(x\cdot w, y\cdot w, z\cdot w, w)

$$

Se, dopo la divisione per $w$, le coordinate sono in $[-1,+1]$, il vertice è inquadrato. I triangoli parzialmente fuori vengono spezzati (clipping).

## Riepilogo essenziale

- La pipeline trasforma i vertici da oggetto a mondo, vista, clip, screen.
- $M_M$ posiziona ogni oggetto, $M_V$ porta tutto nello spazio camera.
- $M_P$ definisce la proiezione e crea NDC con range $[-1,+1]$.
- La proiezione prospettica richiede coordinate omogenee e divisione per $w$.
- Il view frustum è il volume visibile; il clipping gestisce i bordi.

## Domande guida per ripasso

- Qual è la differenza tra spazio oggetto e spazio mondo?
- Perché la matrice di vista è l'inversa della camera?
- Cosa significa che le coordinate in spazio clip sono NDC?
- Quale parametro controlla la distorsione prospettica, e come?
- Perché la proiezione prospettica non e' affine?
- In che punto della pipeline avviene il clipping?

## 12) Approfondimento: trasformazione di vista (Lez107.3)

Ripasso essenziale:

- la trasformazione di vista porta da spazio mondo a spazio vista;
- è determinata dai parametri estrinseci della camera;
- è una trasformazione affine (rotazione + traslazione);
- per la camera vale $V = (M_{camera})^{-1}$.

Una descrizione pratica dei parametri estrinseci (tutti in spazio mondo):

- posizione camera: $p_{eye}$;
- punto target: $p_{target}$ (o direzione di vista);
- vettore up: $v_{up}$.

Costruzione del frame camera (convenzione: asse $z_e$ punta all'indietro):

$$

\mathbf{o}e = p{eye},\quad
\mathbf{z}e = \frac{p{eye}-p_{target}}{\|p_{eye}-p_{target}\|},\quad
\mathbf{x}e = \frac{v{up} \times \mathbf{z}e}{\|v{up} \times \mathbf{z}_e\|},\quad
\mathbf{y}_e = \mathbf{z}_e \times \mathbf{x}_e

$$

La matrice che porta da spazio camera a spazio mondo (`colonne = assi + origine`):

$$

M_{camera}=
\begin{bmatrix}
x_e^x&y_e^x&z_e^x&o_e^x\\
x_e^y&y_e^y&z_e^y&o_e^y\\
x_e^z&y_e^z&z_e^z&o_e^z\\
0&0&0&1
\end{bmatrix},\quad
V = M_{camera}^{-1}

$$

![image.png](Trasformazioni%20nel%20Rendering/image%2012.png)

![image.png](Trasformazioni%20nel%20Rendering/image%2013.png)

![image.png](Trasformazioni%20nel%20Rendering/image%2014.png)

### 12.1) Trackball (orbit control)

Una trackball controlla la camera con tre parametri: $\rho$ (distanza), $\theta$ e $\phi$ (angoli).

Matrice di modellazione della camera (camera → mondo):

$$

M_{camera}=R_y(\phi)\,R_x(\theta)\,T(0,0,rho)

$$

Matrice di vista (inversa, con ordine inverso):

$$

V=T(0,0,-rho)\,R_x(-\theta)\,R_y(-\phi)

$$

Suggerimento immagine:

![Passo 0](Trasformazioni%20nel%20Rendering/image%2015.png)

Passo 0

![Passo 2](Trasformazioni%20nel%20Rendering/image%2016.png)

Passo 2

![Passo 1](Trasformazioni%20nel%20Rendering/image%2017.png)

Passo 1

![Passo 3](Trasformazioni%20nel%20Rendering/image%2018.png)

Passo 3

![Per le matrici inverse, le operazioni avvengono in senso contrario](Trasformazioni%20nel%20Rendering/image%2019.png)

Per le matrici inverse, le operazioni avvengono in senso contrario

## 13) Approfondimento: trasformazione di modellazione nelle scene gerarchiche (Lez108)

In una scena gerarchica, gli oggetti sono nodi di uno scene graph:

- la radice rappresenta lo spazio mondo;
- ogni nodo ha uno spazio oggetto locale;
- ad ogni nodo i è associata una matrice locale $L_i$ che porta al padre.

![image.png](Trasformazioni%20nel%20Rendering/image%2020.png)

La matrice di modellazione globale del nodo i si ottiene cumulando le matrici locali lungo il cammino verso la radice:

$$

M_i = M_{padre} \, L_i

$$

Conseguenza: le trasformazioni del padre si propagano ai figli.

Esempio (auto + ruota):

$$

M_{ruota} = M_{auto}\,M_{ruota|auto}

$$

Vantaggi pratici:

- spostando la macchina, le ruote la seguono automaticamente;
- è naturale definire le trasformazioni locali nel riferimento del padre.
    
    ![image.png](Trasformazioni%20nel%20Rendering/image%2021.png)
    

### Nota operativa (three.js):

- `matrix` = matrice locale del nodo;
- `worldMatrix` = matrice globale dopo la cumulazione;
- la cumulazione viene aggiornata automaticamente prima del rendering.

## Domande guida aggiuntive

- Come si ricavano gli assi della camera a partire da $p_{eye}$, $p_{target}$ e $v_{up}$?
- Perché la view matrix è l'inversa della modellazione della camera?
- In uno scene graph, in che ordine si applicano le trasformazioni locali?
- Quale vantaggio pratico offre la gerarchia per ruote e carlinga?