# 02 - Coordinate Baricentriche

## Obiettivo della lezione

La lezione introduce il legame tra:

- interpolazione lineare;
- coordinate baricentriche;
- interpolazione di attributi su segmenti, triangoli e mesh triangolari.

Il punto centrale e operativo: in Computer Graphics, le coordinate baricentriche sono lo strumento standard per calcolare valori continui all'interno delle facce triangolari.

## 1) Combinazione lineare e interpolazione

### Combinazione lineare

Data una famiglia di vettori (o punti) $\mathbf{v}_i$, una combinazione lineare ha forma:

$$

\mathbf{q} = \sum_i k_i \mathbf{v}_i

$$

dove i $k_i$ sono pesi scalari.

### Interpolazione lineare

La combinazione lineare e una vera interpolazione quando i pesi sono una partizione dell'unita:

$$
\sum_i k_i = 1, \qquad k_i \ge 0

$$

Se vale solo la somma unitaria ma almeno un peso e negativo, si parla di estrapolazione.

## 2) Caso a due elementi: mix / lerp

Per due elementi $\mathbf{a},\mathbf{b}$:

$$

\operatorname{mix}(\mathbf{a},\mathbf{b},t) = (1-t)\mathbf{a} + t\mathbf{b}

$$

Interpretazione pratica del parametro $t$:

- $t=0$: primo estremo;
- $t=1$: secondo estremo;
- $t\in(0,1)$: interpolazione;
- $t<0$ o $t>1$: estrapolazione.

Terminologia equivalente nelle librerie: mix, blend, interpolate, lerp.

## 3) Segmento come luogo di punti

Un segmento e l'insieme dei punti ottenibili interpolando i suoi due estremi.

Viceversa, ogni punto del segmento ha una e una sola coppia di coordinate baricentriche.

Dato un segmento con estremi $\mathbf{p}_0,\mathbf{p}_1$ e un punto interno $\mathbf{q}$:

$$
\mathbf{q} = k_0\mathbf{p}_0 + k_1\mathbf{p}_1,
\qquad k_0+k_1=1,
\qquad k_0,k_1\in[0,1]

$$

### Calcolo geometrico nel segmento

Se $\mathbf{q}$ divide il segmento in due parti, le coordinate baricentriche sono proporzionali alle lunghezze opposte:

- peso di $\mathbf{p}_0$ proporzionale al tratto verso $\mathbf{p}_1$;
- peso di $\mathbf{p}_1$ proporzionale al tratto verso $\mathbf{p}_0$.

In forma normalizzata:

$$
k_0 = \frac{d_1}{d_{tot}},
\qquad
k_1 = \frac{d_0}{d_{tot}}

$$

dove $d_{tot}$ e la lunghezza totale del segmento e $d_0,d_1$ sono i due sotto-segmenti.

## 4) Triangolo come luogo di punti

Un triangolo e l'insieme dei punti ottenibili interpolando i suoi tre vertici.

Dato un triangolo con vertici $\mathbf{p}_0,\mathbf{p}_1,\mathbf{p}_2$ e un punto interno $\mathbf{q}$:

$$
\mathbf{q} = k_0\mathbf{p}_0 + k_1\mathbf{p}_1 + k_2\mathbf{p}_2,
\qquad
k_0+k_1+k_2=1,
\qquad
k_i\in[0,1]

$$

I tre coefficienti $k_0,k_1,k_2$ sono le coordinate baricentriche di $\mathbf{q}$ nel triangolo.

### Calcolo tramite aree

Il punto $\mathbf{q}$ divide il triangolo in tre sotto-triangoli.

Se $A_{tot}$ e l'area del triangolo totale, e $A_0,A_1,A_2$ le aree dei sotto-triangoli opposti ai vertici corrispondenti:

$$
k_0 = \frac{A_0}{A_{tot}},
\qquad
k_1 = \frac{A_1}{A_{tot}},
\qquad
k_2 = \frac{A_2}{A_{tot}}

$$

con:

$$

A_{tot} = A_0 + A_1 + A_2.

$$

## 5) Coordinate baricentriche e attributi

Applicazione chiave in CG:

- conosciute le coordinate baricentriche di un punto $\mathbf{q}$ dentro un triangolo;
- noti gli attributi ai vertici $a_0,a_1,a_2$ (scalari o vettoriali);
- l'attributo in $\mathbf{q}$ si ottiene con la stessa interpolazione:

$$
a(\mathbf{q}) = k_0 a_0 + k_1 a_1 + k_2 a_2.

$$

Esempi di attributi interpolabili:

- colore;
- temperatura o altri scalari;
- coordinate texture (UV);
- normali (con cautela, vedi sotto).

## 6) Continuità su mesh triangolari

In una mesh di triangoli, se gli attributi sono definiti ai vertici e interpolati baricentricamente dentro ogni faccia, allora i valori variano in modo continuo anche attraverso gli edge condivisi.

Questo è il motivo per cui la rasterizzazione triangolare usa sistematicamente coordinate baricentriche per il calcolo per-frammento.

## 7) Attenzione: interpolazione di vettori unitari

Interpolare linearmente vettori unitari non preserva in generale la norma unitaria.

Quindi, se il risultato deve essere ancora unitario (es. normale di shading), dopo l'interpolazione occorre rinormalizzare:

$$
\mathbf{v}_{\text{unit}} = \frac{\mathbf{v}}{\|\mathbf{v}\|}.

$$

## 8) Estensione concettuale oltre il triangolo

La lezione richiama la generalizzazione:

- 2 punti → segmento;
- 3 punti → triangolo;
- 4 punti (in 3D) → tetraedro.

Questa famiglia porta ai complessi simpliciali (linee spezzate, mesh triangolari, mesh tetraedriche).

## 9) Riassunto essenziale (da ricordare)

1. Interpolazione lineare = combinazione lineare con pesi non negativi a somma $1$.
2. Le coordinate baricentriche sono i pesi di interpolazione di un punto in segmento/triangolo.
3. Nel triangolo, i pesi si calcolano in modo semplice tramite rapporti di area.
4. Gli stessi pesi interpolano qualsiasi attributo definito ai vertici.
5. Nelle mesh triangolari questo garantisce continuità degli attributi tra facce adiacenti.
6. Se si interpolano vettori unitari, rinormalizzare quando necessario.

## 10) Domande guida per il ripasso

1. Qual è la differenza formale tra combinazione lineare, interpolazione ed estrapolazione?
2. Perché le coordinate baricentriche di un punto interno a un triangolo sono uniche?
3. Come si ricavano i tre pesi tramite aree dei sotto-triangoli?
4. Perché lo stesso set di pesi può interpolare qualunque attributo (scalare o vettoriale)?
5. Perché l'interpolazione di normali richiede spesso una rinormalizzazione finale?