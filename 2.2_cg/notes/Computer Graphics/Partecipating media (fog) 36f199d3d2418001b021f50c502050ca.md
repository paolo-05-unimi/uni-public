# Partecipating media (fog)

## In breve

- cos'è il participating media (fog) e come si simula con rasterizzazione;
- come si calcola il coefficiente di nebbia in spazio vista;
- come si combina il coefficiente con il colore finale del frammento;
- in quale fase della pipeline si calcolano i dati necessari;
- casi d'uso pratici (realismo, mascherare popping, depth cueing);
- implementazione in API low-level e in librerie high-level (three.js).

## 1) Pipeline e idea base

La pipeline di rendering rasterization-based è pensata per decidere:

- quali attributi servono per vertice;
- quali quantità sono calcolate per vertice o per frammento;
- come gli attributi per vertice vengono interpolati dal rasterizer.

L'effetto fog è un esempio classico:

- il colore finale dipende dalla distanza del frammento dalla camera;
- la distanza e' legata alla $z$ in spazio vista;
- quindi il fog può essere implementato con un semplice calcolo per vertice
e una combinazione per frammento.

![image.png](Partecipating%20media%20(fog)/image.png)

## 2) Participating media (fog): significato

Il medium tra oggetto e camera assorbe parte della luce:

- piu' lungo è il percorso, più forte è l'attenuazione;
- il risultato è nebbia, foschia, acqua torbida, ecc.

Nel rendering a rasterizzazione la simulazione è semplice:

- si definisce un coefficiente di nebbia $f_{fog}$ in base alla distanza;
- il colore finale è una interpolazione tra colore originale e colore nebbia.

## 3) Coefficiente di nebbia in spazio vista

La scelta più semplice è una funzione lineare clampata tra $0$ e $1$.
Dato un punto con coordinata $z$ in spazio vista (negativa):

$$

f_{fog} = \operatorname{clamp}\left(\frac{-z - start}{end - start}, 0, 1\right)

$$

Dove:

- $start$ è la distanza sotto cui non si percepisce nebbia;
- $end$ è la distanza oltre cui si ha nebbia al 100%.

Proprietà utili:

- $f_{fog} = 0$ quando $z = -start$;
- $f_{fog} = 1$ quando $z = -end$.

Nota:

- formule non lineari sono possibili, ma la lineare e' facile da controllare.

![image.png](Partecipating%20media%20(fog)/image%201.png)

## 4) Uso del coefficiente: mix dei colori

Dato:

- $C_{orig}$ = colore che il frammento avrebbe senza fog;
- $C_{fog}$ = colore della nebbia (costante o parametro);
- $f_{fog}$ = coefficiente di nebbia.

Il colore finale si ottiene con una interpolazione lineare:

$$

C = (1 - f_{fog}) \, C_{orig} + f_{fog} \, C_{fog}

$$

Equivalente alla funzione `mix` usata negli shader:

$$

C = \operatorname{mix}(C_{orig}, C_{fog}, f_{fog})

$$

![image.png](Partecipating%20media%20(fog)/image%202.png)

## 5) Dove calcolare il fog nella pipeline

Strategia tipica:

- per vertice: si calcola $z$ in spazio vista e si ottiene $f_{fog}$;
- rasterizer: interpola $f_{fog}$ sui frammenti (come ogni attributo);
- per frammento: si applica la formula di mix sul colore finale.

Vantaggio:

- nessun supporto HW speciale è richiesto;
- è solo un effetto shader, come il lighting.

![image.png](Partecipating%20media%20(fog)/image%203.png)

## 6) Usi pratici del fog

1. Simulazione realistica del medium:
    - nebbia, foschia, acqua sporca, oscurità (se $C_{fog}$ è nero).
2. Mascherare popping al far clipping plane:
    - senza fog: gli oggetti spariscono di colpo;
    - con fog: fade-out continuo verso il colore di sfondo.

Per ottenere il secondo effetto:

- usare $C_{fog}$ uguale al clear color dello sfondo;
- usare $end$ coerente con $z_{far}$ della proiezione.

![image.png](Partecipating%20media%20(fog)/image%204.png)

## 7) Depth cueing e visualizzazione

Il fog e' utile per suggerire la profondità anche in assenza di prospettiva:

- rende leggibile la profondità relativa in proiezione ortografica;
- aiuta in scene complesse (es. molecole, modelli scientifici).

Questo uso è chiamato depth cueing.

![image.png](Partecipating%20media%20(fog)/image%205.png)

## 8) Implementazione in API low-level

In API come OpenGL o WebGL:

- il calcolo del fog si implementa in vertex shader e fragment shader;
- non è una funzione HW automatica, al contrario di depth test o culling.

Schema logico:

- vertex shader: calcola $z$ in spazio vista e $f_{fog}$;
- fragment shader: calcola $C$ con la formula di mix.

## 9) Implementazione in librerie high-level (three.js)

Le librerie high-level forniscono l'effetto fog pronto:

```
var colorNebbia = 0x00FFFF; // azzurro cielo
var near = 0.5; // in spazio vista
var far = 4.0; // in spazio vista
var nebbia = new THREE.Fog(colorNebbia, near, far);
scena = new THREE.Scene();
scena.fog = nebbia;
```

Scelta naturale dei parametri:

- usare `colorNebbia` anche come clear color;
- usare `far` coerente con la distanza del far clipping plane.

![image.png](Partecipating%20media%20(fog)/image%206.png)

## 10) Riepilogo

- Il fog simula un medium che attenua la luce lungo la distanza.
- Il coefficiente $f_{fog}$ dipende dalla $z$ in spazio vista ed è clampato.
- Il colore finale è un mix lineare tra colore originale e colore nebbia.
- Il coefficiente è calcolato per vertice e interpolato per frammento.
- E' utile per realismo, depth cueing e mascherare popping al far plane.
- In low-level si implementa negli shader; in high-level basta abilitarlo.

## 11) Domande guida per il ripasso

- Perché si usa la $z$ in spazio vista per il coefficiente di nebbia?
- Cosa rappresentano i parametri `start` e `end` nella formula di fog?
- Che differenza c'è tra calcolo per vertice e per frammento in questo effetto?
- Come si ottiene l'effetto di mascherare il far clipping plane con il fog?
- In che modo il fog aiuta la lettura della profondità in proiezione ortografica?
- Quali differenze di implementazione ci sono tra OpenGL/WebGL e three.js?