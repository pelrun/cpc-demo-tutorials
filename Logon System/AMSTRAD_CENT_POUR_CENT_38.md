OVERSCAN FACILE
===============

Ce terme étrange n'est pas une nouvelle marque de lessive mais plus simplement l'utilisation de la totalité de récran pour afficher une image. En effet, il ne vous était peut-être encore pas venu à l'idée d'utiliser les zones bordant votre écran graphique (border) pour y loger des données. Cet article va vous expliquer le plus simplement possible la méthode à employer pour formater vous-même un écran aux dimensions désirées.

Créer des pages graphiques en Overscan c'est donner une tout autre dimension à vos oeuvres. Et si les possesseurs de CPC+ se mettent à les utiliser conjointement avec les nouvelles commandes que nous avons créées pour vous, nous devrions bientôt recevoir de superbes réalisations à la rédaction. Mais voyons sans plus attendre la création d'un écran Overscan. Celle-ci se fait en plusieurs étapes.

PLAÇONS L'ECRAN
---------------

Il convient tout d'abord de placer votre écran en haut à gauche.
Ceci est fait très simplement grâce aux registres CRTC de positionnement. Respectivement les resgistres 2 et 7 de la bébête.
Registre 2 : Position X de l'écran.
Registre 7 : Pisftion Y de l'écran.
Amusez-vous à modifier les valeurs de ces deux registres pour déplacer votre écran dans tous les sens (la valeur par défaut des ces 2 registres étant Registre 2 = 46 et Registre 7 = 30).
Je rappelle rapidement comment envoyer une valeur sur un registre CRTC (une fois n'est pas coutume, il y a du Basic dans cette rubrique) :

```
OUT &BC00,numéro de registre
OUT &BD00,valeur
```

Reprenons, nous voulons positionner notre écran en haut à gauche. Visualisons d'abord le Border en changeant sa couleur grâce à la subtile commande Basic permettant de modifier la couleur de bord d'écran : BORDER 0, et hop 1 Modifions ensuite la valeur des registres 2 et 7 du CRT. Tout d'abord en X :

```
OUT &BC00,2:0UT &BD00,49
```

Puis en Y:

```
OUT &BC00,7:OUT &BD00,36
```

Ca y est, j'en étais stir, le petit blond au fond se marre parce que sur sa télé Schneider Artron coins très carrés, l'écran n'est pas vraiment tout à fait à gauche. Qu'à cela ne tienne, il suffit de mettre la valeur 50 dans le Registre 2. Arrrgghgh !!! Ceci est le cri d'un CRTC...
Effectivement, si cette valeur est correcte pour votre CPC, elle ne l'est pas pour tous les CRTC. Nous abordons alors le douloureux problème de la compatibilité vidéo entre les CPC. Sur certains CRTC, placer une valeur supérieure à 49 dans le registre 2 provoque une perte de synchronisation verticale, ce qui est traduit à l'écran par un défilement ininterrompu de l'image. Alors, devrons-nous être toujours limité à 49 pour rester 100 % compatible ?
Non! Et voici la solution au problème.

SOLUTIONNONS LE CRTC
--------------------

II est possible de mettre une valeur supérieure à 49 dans le registre 2 des CRTC qui ne le supportent pas. Il faudra alors vérifier que la somme des valeurs contenues dans le registre 2 et le registre 3 soit inférieure à celle du registre 0.

* Diagram not transcribed yet

Par exemple, notre registre 0 est égal à 63 et nous voulons que notre registre 2 soit à 50.
Nous allons donc placer le registre 3 à 8, ce qui sera largement suffisant pour remédier au problème de compatibilité. Donc, nous pouvons placer notre écran un peu plus à gauche :
En X:

```
OUT &BC00,3:OUT &BD00,8
OUT &BC00,2:OUT &BD00,50
```

Maintenant que notre écran est placé, passons à l'étape suivante.

AGRANDISSONS L'ECRAN
--------------------

Donc, notre écran est maintenant positionné en haut à gauche.
Et nous allons l'agrandir grâce aux registres CRTC de taille:
Reg 1 : Taille en X
Reg 6 : Taille en Y
Commencons par le deuxième registre, qui concerne la taille verticale. Ce registre contient le nombre de lignes caractères affichées verticalement. Habituellement, sa valeur est 25 car vous disposez de 25 lignes de caractères. En changeant sa valeur, vous modifierez donc la taille de l'écran. Pour que l'écran arrive jusqu'en bas : OUT &BCOO,6:OUT &BD00,34. Ce qui fait donc 34x8=272 lignes verticales...
C'est mieux que 200, non ?!?

Ça y est, première réflexion.., l'écran se répète !!! C'est-à-dire que ce qui est affiché en haut reprend en bas...
Et pourquoi donc ? Tout simplement parce que la mémoire vidéo du CPC ne fait que 16384 octets, et que nous avons affiché 272 lignes x 80 octets/ lignes, ce qui fait 21760 octets. C'est pourquoi le CRTC, lorsqu'il a fini de compter jusqu'a 16384, repart de 0...et l'image recommence...
Facheux, non ??? Pas vraiment, parce qu'il existe heureusement une feinte... Les OverscanBits.
En effet, il existe un moyen de dire au CRT: Ne t'arrête pas, Fils, Continue!
Pour faire cela, étudions la structure du registre 12 et 13 du CRTC :

Nous voyons sur le schéma que le registre 12 permet de placer l'écran dans la mémoire du CPC par tranches de 16 Ko et aussi de mettre le compteur vidéo sur 16 Ko ou 32 Ko (précisément ce qui nous intéresse).
En bref, et pour être encore plus clair, quelques exemples :

```
Ecran en &0000 : OUT &BC00,12:OUT &BD00,&X00000000
Ecran en &4000 : OUT &BC00,12:OUT &BD00,&X00010000
Ecran en &8000 : OUT &BC00,12:OUT &BD00,&X00100000
Ecran en &C000 : OUT &BC00,12:OUT &BD00,&XO0110000 (standard du système)
```

Puis les exemples pour un compteur supérieur à 16 Ko.

```
Ecran en 80000 : OUT &BC00,12:OUT &BD00,&X00001100
Ecran en &4000 : OUT &BC00,12:OUT &BD00,&X00011100
Ecran en &8000 : OUT &BC00,12:OUT &BD00,&X00101100
Ecran en &C000 : OUT &BC00,12:OUT &BD00,&X00111100 (standard du système)
```

En bref, pour reprendre l'exemple de tout à l'heure et que l'écran continue, il suffit (l'écran étant en COCO) de placer le reg 12

```
OUT &BC00,12 : OUT &BD00,&X00111100
```

A partir de ce moment-là, la zone qui apparaissait en modulo disparaît pour faire apparaître une autre zone mémoire...

Notre page se trouvant en &C000, la page qui apparaît est donc en &0000. Du coup, les zones des restarts et du Basic sont visualisées à l'écran.
L'utilisation d'une page Overscan est critique pour la mémoire du CPC puisque deux pages servent à l'affichage, soit 32 Ko. Ce qui représente la moitié de la Ram centrale du CPC.

Il est donc impossible de conserver le système lors de l'affichage d'une page vidéo overscan, car celle-ci doit obligatoirement écraser une page en 8000 ou en 0000, qui sont toutes deux occupées par le système.
C'est pourquoi la manipulation d'une page overscan doit être faite sans le système et donc en Assembleur...
Le problème pouvant se poser en overscan étant le passage d'un écran à l'autre (certaines personnes peuvent avoir du mal à calculer les passages d'une page à l'autre), il est pratique de séparer nettement les deux pages vidéo en fixant une longueur horizontale de 128 octets (donc en plaçant le regi à 64)... Attention toutefois à placer le reg0 à une valeur supérieure ou égale au regl Eh oui ! Encore une règle) et donc à 64 (sa valeur par défaut étant de 63).
Chaque page vidéo fait alors 128 lignes de haut et il n'y a pas de reste, donc de zone gênante pour les passages d'une page à l'autre...
Voilà, si l'Overscan vous tient encore tête... vendez votre CPC !
(Mais non Poum, c'était une blague...).
Et rendez-vous dans ûn mois pour une rubrique technique toujours aussi intéressante et avec les Logon System.
NB : la rédaction du magazine tient à décliner toute responsabilité quant aux jeux de mots vaseux errant parfois dans les articles des Logon System.
Au fait ?! Pourquoi un chat complètement maboule pour illustrer les articles des Logon ? Simplement parce que nous essayons de trouver une mascotte aux Logon.

LES LOGON SUR MINITEL
---------------------

A partir de ce mois de juin, vous pourrez laisser des messages à l'intention des Logon System sur le serveur du magazine: le 3615 ACPC. II sera même possible de télécharger très prochainement les meilleures démos du groupé.

Longshot
