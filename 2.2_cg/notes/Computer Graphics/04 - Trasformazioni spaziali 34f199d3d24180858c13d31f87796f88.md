# 04 - Trasformazioni spaziali

# Trasformazioni spaziali affini

## In breve

Questa dispensa unifica e approfondisce la parte matematica delle trasformazioni spaziali affini, con focus su:

- rappresentazione omogenea di punti e vettori;
- trasformazioni elementari (traslazione, scalatura, rotazione, simmetria, shearing);
- composizione, ordine, inversa e proprieta numeriche delle matrici;
- interpretazione geometrica tramite determinante e rango;
- interpretazione equivalente come cambio di sistema di riferimento;
- implicazioni operative per rasterizzazione, mesh e spline.

## 1) Inquadramento: trasformazioni nella pipeline di rasterizzazione

Nel rendering rasterization-based, la fase per vertice applica trasformazioni spaziali ai dati geometrici.

![image.png](04%20-%20Trasformazioni%20spaziali/image.png)

Dato un vertice di coordinate 3D, la pipeline calcola una nuova posizione (in spazi successivi) fino alla proiezione su immagine.

Formalmente, una trasformazione spaziale è una funzione che mappa:

- punti in punti;
- vettori in vettori.

![image.png](04%20-%20Trasformazioni%20spaziali/image%201.png)

Questo dettaglio è cruciale: punti e vettori non sono la stessa cosa, e il formalismo matematico deve rispettare questa differenza.

## 2) Definizione matematica di trasformazione affine

Una trasformazione affine in $\mathbb{R}^3$ ha forma:

$$
f(\mathbf{p}) = \mathbf{A}\mathbf{p} + \mathbf{t}

$$

con:

- $\mathbf{A} \in \mathbb{R}^{3\times 3}$ parte lineare;
- $\mathbf{t} \in \mathbb{R}^3$ termine di traslazione.

Per i vettori (che rappresentano differenze tra punti), il termine traslativo non deve contribuire:

$$
f(\mathbf{v}) = \mathbf{A}\mathbf{v}.

$$

Questa è la distinzione strutturale che poi viene codificata elegantemente nelle coordinate omogenee.

## 3) Coordinate omogenee: distinzione punti vs vettori

Rappresentazione standard:

$$
\mathbf{p}_h = \begin{bmatrix}x\\y\\z\\1\end{bmatrix},
\qquad
\mathbf{v}_h = \begin{bmatrix}x\\y\\z\\0\end{bmatrix}.

$$

Qui la coordinata $w$ (affine) vale:

- $w=1$ per i punti;
- $w=0$ per i vettori.

Con questa convenzione tornano tutte le regole geometriche:

- punto - punto = vettore;
- punto + vettore = punto;
- vettore $\pm$ vettore = vettore.

![image.png](04%20-%20Trasformazioni%20spaziali/image%202.png)

## 4) Matrice affine 4x4: forma generale

In coordinate omogenee, una trasformazione affine è una moltiplicazione:

$$
\mathbf{p}'_h = \mathbf{M}\,\mathbf{p}_h,
\qquad
\mathbf{v}'_h = \mathbf{M}\,\mathbf{v}_h,

$$

con matrice:

$$

\mathbf{M}=
\begin{bmatrix}
\mathbf{A} & \mathbf{t} \\
\mathbf{0}^T & 1
\end{bmatrix}= \begin{bmatrix}
a_{11}&a_{12}&a_{13}&t_x\\
a_{21}&a_{22}&a_{23}&t_y\\
a_{31}&a_{32}&a_{33}&t_z\\
0&0&0&1
\end{bmatrix}.
$$

L'ultima riga $(0,0,0,1)$ garantisce:

- punti mappati in punti;
- vettori mappati in vettori.

Inoltre:

$$
\mathbf{M}\begin{bmatrix}\mathbf{v}\\0\end{bmatrix}=
\begin{bmatrix}\mathbf{A}\mathbf{v}\\0\end{bmatrix},

$$

quindi la traslazione viene automaticamente annullata sui vettori.

## 5) Traslazione

Con $\mathbf{t}=(t_x,t_y,t_z)^T$:

$$
\mathbf{p}'=\mathbf{p}+\mathbf{t},
\qquad
\mathbf{v}'=\mathbf{v}.

$$

Matrice omogenea:

$$
\mathbf{T}(t_x,t_y,t_z)=
\begin{bmatrix}
1&0&0&t_x\\
0&1&0&t_y\\
0&0&1&t_z\\
0&0&0&1
\end{bmatrix}.

$$

Aspetto fondamentale: traslare un oggetto significa traslare tutti i suoi punti, ma non i suoi vettori direzionali (normali, velocità, direzioni).

![image.png](04%20-%20Trasformazioni%20spaziali/67ffc1ef-bd02-449c-bd6f-d2f97af2c2e7.png)

## 6) Scalatura: uniforme e anisotropica

Scalatura generale:

$$
\mathbf{S}(s_x,s_y,s_z)=
\begin{bmatrix}
s_x&0&0&0\\
0&s_y&0&0\\
0&0&s_z&0\\
0&0&0&1
\end{bmatrix}.

$$

Applica:

$$
x'=s_xx,\quad y'=s_yy,\quad z'=s_zz.

$$

Casi:

- uniforme/isotropica: $s_x=s_y=s_z=s$;
- anisotropica: almeno due fattori diversi.

Proprietà geometriche rilevanti:

- se $|s_i|>1$ dilatazione lungo asse $i$;
- se $0<|s_i|<1$ contrazione;
- segno negativo di un fattore implica ribaltamento rispetto al piano coordinato corrispondente.

Determinante della parte lineare:

$$
\det(\mathbf{A})=s_xs_ys_z,

$$

che da il fattore di scala orientato del volume.

![$f\begin{pmatrix}x \\y \\z\end{pmatrix}=\gamma \cdot\begin{pmatrix}x \\y \\z\end{pmatrix}=\begin{pmatrix}\gamma \cdot x \\\gamma \cdot y \\\gamma \cdot z\end{pmatrix}$](04%20-%20Trasformazioni%20spaziali/image%203.png)

$f\begin{pmatrix}x \\y \\z\end{pmatrix}=\gamma \cdot\begin{pmatrix}x \\y \\z\end{pmatrix}=\begin{pmatrix}\gamma \cdot x \\\gamma \cdot y \\\gamma \cdot z\end{pmatrix}$

![$f\begin{pmatrix}
x \\
y \\
z
\end{pmatrix}
=
\begin{pmatrix}
\gamma_x \\
\gamma_y \\
\gamma_z
\end{pmatrix}
\cdot
\begin{pmatrix}
x \\
y \\
z
\end{pmatrix}
=
\begin{pmatrix}
\gamma_x \cdot x \\
\gamma_y \cdot y \\
\gamma_z \cdot z
\end{pmatrix}$ ← prodotto componente per componente“component-wise product” (non un’operazione canonica)](04%20-%20Trasformazioni%20spaziali/image%204.png)

$f\begin{pmatrix}
x \\
y \\
z
\end{pmatrix}
=
\begin{pmatrix}
\gamma_x \\
\gamma_y \\
\gamma_z
\end{pmatrix}
\cdot
\begin{pmatrix}
x \\
y \\
z
\end{pmatrix}
=
\begin{pmatrix}
\gamma_x \cdot x \\
\gamma_y \cdot y \\
\gamma_z \cdot z
\end{pmatrix}$ ← prodotto componente per componente“component-wise product” (non un’operazione canonica)

## 7) Rotazione in 2D: derivazione analitica

![image.png](04%20-%20Trasformazioni%20spaziali/image%205.png)

Per rotazione antioraria di angolo $\beta$ attorno all'origine:

$$
\begin{aligned}
x' &= x\cos\beta - y\sin\beta,\\
y' &= x\sin\beta + y\cos\beta.
\end{aligned}

$$

Derivazione tipica via coordinate polari:

$$
x=\rho\cos\alpha,\quad y=\rho\sin\alpha,

$$

$$
x'=\rho\cos(\alpha+\beta),\quad y'=\rho\sin(\alpha+\beta),

$$

poi identità goniometriche di somma.

Matrice 2x2:

$$
\mathbf{R}_{2D}(\beta)=
\begin{bmatrix}
\cos\beta & -\sin\beta\\
\sin\beta & \cos\beta
\end{bmatrix}

$$

In omogenee 3x3 (2D affine):

$$
\begin{bmatrix}
\cos\beta & -\sin\beta & 0\\
\sin\beta & \cos\beta & 0\\
0&0&1
\end{bmatrix}

$$

## 8) Rotazioni 3D attorno agli assi canonici

### 8.1 Attorno all'asse z

$$
\mathbf{R}_z(\beta)=
\begin{bmatrix}
\cos\beta&-\sin\beta&0&0\\
\sin\beta&\cos\beta&0&0\\
0&0&1&0\\
0&0&0&1
\end{bmatrix}

$$

### 8.2 Attorno all'asse x

$$
\mathbf{R}_x(\beta)=
\begin{bmatrix}
1&0&0&0\\
0&\cos\beta&-\sin\beta&0\\
0&\sin\beta&\cos\beta&0\\
0&0&0&1
\end{bmatrix}

$$

### 8.3 Attorno all'asse y

$$
\mathbf{R}_y(\beta)=
\begin{bmatrix}
\cos\beta&0&\sin\beta&0\\
0&1&0&0\\
-\sin\beta&0&\cos\beta&0\\
0&0&0&1
\end{bmatrix}

$$

Proprietà chiave delle rotazioni pure:

$$
\mathbf{R}^{-1}=\mathbf{R}^T=\mathbf{R}(-\beta),
\qquad
\mathbf{R}^T\mathbf{R}=\mathbf{I},
\qquad
\det(\mathbf{R})=1.
$$

Non commutatività in 3D:

$$
\mathbf{R}_x(\alpha)\mathbf{R}_y(\beta) \ne \mathbf{R}_y(\beta)\mathbf{R}_x(\alpha).

$$

![image.png](04%20-%20Trasformazioni%20spaziali/image%206.png)

## 9) Simmetria planare (mirroring)

Esempio: riflessione rispetto al piano $z=0$:

$$
(x,y,z) \mapsto (x,y,-z),

$$

$$
\mathbf{M}_{z=0}=
\begin{bmatrix}
1&0&0&0\\
0&1&0&0\\
0&0&-1&0\\
0&0&0&1
\end{bmatrix}
$$

Interpretazione:

- inverte l'orientamento (destrorso/sinistrorso);
- determinante negativo ($\det=-1$) indica ribaltamento speculare.

![image.png](04%20-%20Trasformazioni%20spaziali/image%207.png)

## 10) Shearing (deformazione angolare)

Esempio della lezione: shearing di $x$ rispetto a $y$:

$$
x' = x + y\cot\theta,\qquad y'=y,\qquad z'=z.

$$

Matrice:

$$
\mathbf{H}_{xy}(\theta)=
\begin{bmatrix}
1&\cot\theta&0&0\\
0&1&0&0\\
0&0&1&0\\
0&0&0&1
\end{bmatrix}
$$

Proprietà notevoli:

- conserva parallelismo tra rette;
- in generale non conserva angoli e lunghezze;
- con diagonale unitaria e caso base mostrato, $\det=1$ (volume orientato invariato).

![image.png](04%20-%20Trasformazioni%20spaziali/image%208.png)

## 11) Composizione di trasformazioni

Se prima applico $\mathbf{M}_A$ e poi $\mathbf{M}_B$ (vettori colonna):

$$
\mathbf{p}'' = \mathbf{M}_B(\mathbf{M}_A\mathbf{p})=(\mathbf{M}_B\mathbf{M}_A)\mathbf{p}
$$

Quindi:

- ordine applicazione: destra → sinistra;
- ordine matrici: ultima trasformazione a sinistra.

Proprietà:

- associativa: $(\mathbf{A}\mathbf{B})\mathbf{C}=\mathbf{A}(\mathbf{B}\mathbf{C})$;
- non commutativa: $\mathbf{A}\mathbf{B}\ne\mathbf{B}\mathbf{A}$.

Esempio tipico:

- "ruota poi trasla" = $\mathbf{T}\mathbf{R}$;
- "trasla poi ruota" = $\mathbf{R}\mathbf{T}$;
- i risultati geometrici sono diversi.

![image.png](04%20-%20Trasformazioni%20spaziali/image%209.png)

## 12) Inversa di trasformazioni affini

Regole operative centrali:

### 12.1 Inversa di scalatura

$$
\mathbf{S}^{-1}(s_x,s_y,s_z)=\mathbf{S}\left(\frac{1}{s_x},\frac{1}{s_y},\frac{1}{s_z}\right),

$$

valida se $s_x,s_y,s_z\ne 0$.

### 12.2 Inversa di traslazione

$$
\mathbf{T}^{-1}(t_x,t_y,t_z)=\mathbf{T}(-t_x,-t_y,-t_z).

$$

### 12.3 Inversa di rotazione pura

$$
\mathbf{R}^{-1}(\beta)=\mathbf{R}(-\beta)=\mathbf{R}^T.

$$

### 12.4 Inversa di composizione

$$
(\mathbf{M}_B\mathbf{M}_A)^{-1}=\mathbf{M}_A^{-1}\mathbf{M}_B^{-1}.

$$

L'ordine si inverte sempre.

![image.png](04%20-%20Trasformazioni%20spaziali/image%2010.png)

## 13) Determinante, rango, invertibilità: lettura geometrica

Per una matrice affine 4x4 con blocco lineare $\mathbf{A}$:

$$
\det(\mathbf{M})=\det(\mathbf{A}).

$$

Interpretazioni:

- $|\det(\mathbf{A})|$: fattore di scala dei volumi;
- segno di $\det(\mathbf{A})$: preservazione o inversione dell'orientamento;
- $\det(\mathbf{A})=0$: collasso dimensionale e non invertibilità.

Rango (parte lineare):

- $\operatorname{rank}(\mathbf{A})=3$: oggetti 3D restano 3D;
- $\operatorname{rank}(\mathbf{A})=2$: collasso su piano;
- $\operatorname{rank}(\mathbf{A})=1$: collasso su retta;
- $\operatorname{rank}(\mathbf{A})=0$: collasso in punto.

Questa lettura è fondamentale per diagnosticare trasformazioni degeneri.

## 14) Rotazioni attorno ad assi non passanti per l'origine

Idea costruttiva standard:

1. trasla il pivot/asse all'origine;
2. applica la rotazione pura;
3. riporta il pivot nella posizione iniziale.

Se $\mathbf{c}$ è un punto dell'asse e $\mathbf{R}$ la rotazione pura:

$$
\mathbf{M}=\mathbf{T}(\mathbf{c})\,\mathbf{R}\,\mathbf{T}(-\mathbf{c}).

$$

Questa formula rende esplicito il pattern mostrato nelle slide (composizione traslazione-rotazione-traslazione inversa).

## 15) Definizione equivalente: affinità e combinazioni lineari

Una funzione affine preserva le combinazioni affini:

$$
\sum_i k_i = 1
\quad\Rightarrow\quad
f\!\left(\sum_i k_i\mathbf{p}_i\right)=\sum_i k_i f(\mathbf{p}_i).

$$

Dimostrazione rapida, usando $f(\mathbf{p})=\mathbf{A}\mathbf{p}+\mathbf{t}$:

$$
f\!\left(\sum_i k_i\mathbf{p}_i\right)
=\mathbf{A}\sum_i k_i\mathbf{p}_i + \mathbf{t}
=\sum_i k_i\mathbf{A}\mathbf{p}_i + \mathbf{t}
=\sum_i k_i(\mathbf{A}\mathbf{p}_i+\mathbf{t})
=\sum_i k_i f(\mathbf{p}_i).

$$

Implicazione diretta per CG:

- per trasformare un triangolo basta trasformare i vertici;
- per trasformare una spline basta trasformare i punti di controllo.

## 16) Cambio di sistema di riferimento: interpretazione equivalente

Un sistema di riferimento $R$ è definito da:

- base vettoriale $\{\mathbf{a}_x,\mathbf{a}_y,\mathbf{a}_z\}$ (assi linearmente indipendenti);
- origine $\mathbf{p}_o$.

Un punto espresso in coordinate $(x,y,z)$ nel sistema $R$ vale nello spazio canonico:

$$
\mathbf{p}=\mathbf{p}_o + x\mathbf{a}_x + y\mathbf{a}_y + z\mathbf{a}_z.

$$

Forma omogenea:

$$
\mathbf{p}_{\text{canon}}=
\begin{bmatrix}
\mathbf{a}_x & \mathbf{a}_y & \mathbf{a}_z & \mathbf{p}_o\\
0&0&0&1
\end{bmatrix}
\mathbf{p}_R
$$

Interpretazione delle colonne della matrice affine:

- prime 3 colonne: assi del frame sorgente espressi nel frame di arrivo;
- quarta colonna: origine del frame sorgente espressa nel frame di arrivo.

Questo spiega geometricamente perché una matrice affine può essere letta sia come "trasformo oggetto" sia come "cambio di riferimento".

![image.png](04%20-%20Trasformazioni%20spaziali/image%2011.png)

## 17) Esempio sintetico completo (matrice unica)

Supponiamo sequenza:

1. scalatura anisotropica $\mathbf{S}(2,1,1)$;
2. rotazione di $45^\circ$ attorno a $z$: $\mathbf{R}_z(\pi/4)$;
3. traslazione $\mathbf{T}(3,-1,0)$.

Matrice complessiva (ordine corretto):

$$
\mathbf{M}=\mathbf{T}(3,-1,0)\,\mathbf{R}_z(\pi/4)\,\mathbf{S}(2,1,1)
$$

Posto $c=s=\frac{\sqrt2}{2}$, si ottiene:

$$
\mathbf{M}=
\begin{bmatrix}
2c & -s & 0 & 3\\
2s & c & 0 & -1\\
0 & 0 & 1 & 0\\
0 & 0 & 0 & 1
\end{bmatrix}
$$

Questa singola matrice applica l'intera catena in un solo prodotto matrice-vettore per punto.

## 18) Formula sheet operativo (ripasso rapido)

$$
\mathbf{p}_h=[x\ y\ z\ 1]^T,
\qquad
\mathbf{v}_h=[x\ y\ z\ 0]^T

$$

$$
\mathbf{M}=\begin{bmatrix}\mathbf{A}&\mathbf{t}\\0&1\end{bmatrix},
\quad
f(\mathbf{p})=\mathbf{A}\mathbf{p}+\mathbf{t},
\quad
f(\mathbf{v})=\mathbf{A}\mathbf{v}
$$

$$
\mathbf{T}^{-1}(\mathbf{t})=\mathbf{T}(-\mathbf{t}),
\quad
\mathbf{S}^{-1}(s_x,s_y,s_z)=\mathbf{S}(1/s_x,1/s_y,1/s_z),
\quad
\mathbf{R}^{-1}=\mathbf{R}^T
$$

$$

(\mathbf{M}_2\mathbf{M}_1)^{-1}=\mathbf{M}_1^{-1}\mathbf{M}_2^{-1},
\quad
\det(\mathbf{M})=\det(\mathbf{A}),
\quad
\det<0 \Rightarrow \text{ribaltamento orientazione}

$$

$$

\mathbf{M}_{\text{asse non origine}}=\mathbf{T}(\mathbf{c})\,\mathbf{R}\,\mathbf{T}(-\mathbf{c})

$$

## 19) Riassunto essenziale

1. Le trasformazioni affini in 3D si rappresentano con matrici 4x4 in coordinate omogenee.
2. La parte lineare $\mathbf{A}$ agisce su punti e vettori; la traslazione $\mathbf{t}$ solo sui punti.
3. Composizione e ordine contano: il prodotto matriciale è associativo ma non commutativo.
4. Inversa, determinante e rango danno una lettura geometrica immediata (invertibilità, scala volumetrica, collassi dimensionali).
5. Le rotazioni pure sono ortonormali: $\mathbf{R}^{-1}=\mathbf{R}^T$.
6. Ogni trasformazione affine può essere letta anche come cambio di sistema di riferimento.
7. La proprietà di preservare combinazioni affini rende possibile trasformare mesh/spline agendo solo su vertici o punti di controllo.

## 20) Domande guida per il ripasso

1. Perché in coordinate omogenee i punti hanno $w=1$ e i vettori $w=0$?
2. Come si ricava la forma a blocchi $\mathbf{M}=\begin{bmatrix}\mathbf{A}&\mathbf{t}\\0&1\end{bmatrix}$ a partire da $f(\mathbf{p})=\mathbf{A}\mathbf{p}+\mathbf{t}$?
3. Qual è la differenza geometrica tra $\mathbf{T}\mathbf{R}$ e $\mathbf{R}\mathbf{T}$?
4. Come si interpreta geometricamente il determinante (modulo e segno)?
5. Quando una trasformazione affine non è invertibile e che effetto ha su un oggetto 3D?
6. Perché $\mathbf{R}^{-1}=\mathbf{R}^T$ per le rotazioni pure?
7. Come si costruisce una rotazione attorno ad asse non passante per l'origine?
8. In che senso "trasformare prima e interpolare dopo" equivale a "interpolare prima e trasformare dopo" per mappe affini?