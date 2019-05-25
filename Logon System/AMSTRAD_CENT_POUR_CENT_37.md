HISTOIRES DE RASTERS
====================

Hello ! amis du CPC, j'espère que le mois d'attente entre deux parutions de votre rubrique préférée n'a pas été trop long. Et je sais vous, c'est avec une joie d'eu que pour certains d'a non dissimulée que vous vous jetez sur les pages Logon pour les dévorer.

Mais qu'a-t-on encore inventé pour torturer votre cerveau malade d'un trop plein d'Assembleur ? Une petite info avant d'ouvrir les hostilités: une nouvelle version de The Demo, sans chargeur musical (donc plus rapide) et avec un formatage classique (c'est-à-dire copiable sans problèmes) est terminée au moment où vous lisez ces lignes.

GATE ARRAYTE TU ME CHATOUILLES!
-------------------------------

D'accord mon intertitre n'est pas « gégène » mais je n'ai pas la classe de Sined ou de Poum. Pour cette fois, nous allons faire un tour du côté du Gate Array (je sais, Fred en a déjà parlé), oui ce composant d'Amstrad s'occupant de la connexion Rom et Ram bank, ainsi que la sélection du mode graphique et des couleurs. Je vais faire un petit récapitulatif sur les différentes façons de faire des rasters, je suppose que ce mot ne vous est plus inconnu, il semble que ses origines viennent du Commodore C64 (dixit Longshot). Donc, un raster consiste à modifier les couleurs des encres du Gate Array, afin d'obtenir différentes couleurs simultanément sur une même encre, et cela pouvant se faire du haut jusqu'au bas de l'écran (évidemment dans la limite des 27 couleurs disponibles de la palette).

DIS...MOI CE QU'IL FAUT FAIRE...
--------------------------------

Ce qu'il faut tout d'abord savoir c'est qu'avant de faire des rasters, il faut éviter que votre programme soit interrompu intempestivement par les interruptions du système. Pour cela, on peut les empêcher avec le DI classique mais la meilleure solution consiste à les détourner (grâce aux non moins classiques EI et RET en &38) pour pouvoir se synchroniser sur les HALT. Une fois le problème des interruptions résolu, occupons-nous de savoir comment faire notre raster.

LES GOUTS ET LES COULEURS...
----------------------------

Il faut premièrement dire au Gate Array dans quel numéro d'encre on désire faire le raster (exemple : &7F00 pour le papier) et ensuite quelle couleur on désire avoir sur cette encre (exemple : &7F40+12 pour le rouge, pour les couleurs Gate Array référez-vous à l'article de Fred, Cent Pour Cent n° 35). Une petite remarque toutefois : si le raster se trouve tout le temps dans la même encre il n'est pas nécessaire de sélectionner à chaque fois le numéro, car une fois le OUT de sélection d'encre fait, le numéro d'encre est devenu le registre dit « courant » (oui, le Gate Array, tout comme le Z80, est équipé des registres) il n'y a plus qu'à changer la couleur pour créer de beaux rasters.

PRIEZ SYNCHRO...
----------------

Maintenant, nous allons parler de temps machine, car si l'on désire faire de beaux rasters il doivent être synchronisés sur la durée d'une ligne écran (cette durée est de 64 microsecondes d'origine, modifiable via le registre 0 du CRTC). C'est-a-dire qu'il faut que l'intervalle de temps entre deux changements de couleurs soit égal à 64 microsecondes (en instruction cela donne 64 NOP), et de plus que le changement s'effectue le plus tôt en début de ligne pour qu'il n'y ait pas de cassure en plein milieu du raster.
Si vous ne connaissez pas les cycles exacts pour chaque instruction (il doit y avoir un petit tableau avec quelques exemples de durée qui traîne dans ces pages), vous pouvez essayer de synchroniser les rasters de visu. Si les rasters sont en diagonales en montant de la gauche vers la droite, cela signifie que la durée de synchronisation est trop courte, il faut donc ajouter des NOP. Si les rasters sont en diagonales en descendant de la gauche vers la droite, cela signifie que la durée de synchronisation est trop longue : dans ce cas il supprime des NOP, et si cela n'est pas possible, optimisez le programme.
Sur le plan de la programmation proprement dite, il faut faire une boucle autant de fois que l'on désire de lignes et ensuite envoyer des données au Gate Array (ces données sont les couleurs, bien sùr). Ces données sont stockées dans un tableau en mémoire auquel on accède par un registre 16 bits quelconque (j'explique le principe en gros, ensuite c'est à vous de faire le programme). Pour faire se déplacer le raster, il suffit soit de déplacer les données contenues dans le tableau, soit de changer la valeur du pointeur de tableau.

QUE DE COULEURS!
----------------

Afin d'ajouter des couleurs dans vos écrans, vous pouvez aussi faire des rasters dans plusieurs encres en même temps (exemple : un raster en encre 0 et un raster dans le BORDER donnant l'impression que l'écran est en OVERSCAN). Mais le problème du temps machine survient de nouveau car il vous faut changer plusieurs fois de numéro d'encre et ensuite attribuer une couleur particulière à chaque encre, ce qui fait un nombre de OUT assez important et cela nécessite souvent l'utilisation des registres secondaires (HL', DE', BC', AF') ; mais il faut surtout éviter d'utiliser IX et IY, sinon il vous restera très peu de temps machine.

SPLIT RASTER KESAKO !?
----------------------

Peut-être n'avez vous jamais entendu ce mot. « Split » vient de l'anglais et signifie déchirure, scission. En fait, des split rasters sont des rasters qui changent de couleurs mais plusieurs fois sur une même ligne (jusqu'à aujourd'hui le maximum que j'ai réussi à mettre est 13 split rasters sur une même ligne). Donc, imaginez un instant que la palette de couleurs contienne plus de couleurs que les 27 d'origine (qu'est-ce qu'elles sont tristes ces 27 malheureuses couleurs...), vous pourriez faire un split raster de 200 lignes et que la couleur change 10 fois par ligne, cela donnerait 10 x 200, c'est-à-dire 2 000 couleurs à l'écran ! (Ce genre de chose est réalisable sur le CPC+, malheureusement les 4096 couleurs sont protégées par copyright...)

Il y a plusieurs solutions pour les split rasters. Ils peuvent être composés de couleurs qui ne changent pas ; dans ce cas, il suffit de charger différents registres avec les couleurs désirées et ensui- te d'alterner OUT après OUT. Ou bien les split rasters sont faits de couleurs qui changent (tubes qui montent, tube qui tourne selon des tables de sinus ou autre effets amusants...) : dans ce cas, il vaut mieux utiliser l'instruction OUTI. Cette instruction envoie un octet pointé par HL sur le port indiqué par le registre B, incrémente HL et décrémente B ; mais cependant, il y a un problème, qui vient du fait que le registre B est décrémenté par le OUTI; il est donc nécessai- re de le remettre à jour en l'incrémentant de nouveau après le OUTI.

QUEL OUT CHOISIR ?
------------------

Pour les non-initiés, cette instruction peut paraître bien bizarre : en effet « OUT (C),A » met la valeur de l'accumulateur sur le port indiqué par le registre B, alors que B n'est pas spécifié dans le OUT (étrange, non ?). Ce qu'il faut savoir, c'est que le Z80 peut adresser 256 périphériques (c'est-à-dire qu'il a 256 adresses d'entrée/sortie). Le problème ne vient pas du Z80 mais du CPC, car les périphériques (dont font partie le CRTC, le PSG et le Gate Array) sur Amstrad sont câblés sur les fils d'adresse de poids fort (A8a Al5) et donc au lieu de passer par C lors d'un OUT sur CPC, on est obligé de passer par B (qui correspond au poids fort du registre BC).

Dans le genre buggé, il y a aussi le « OUT (0),A » qui normalement met la valeur de l'accumulateur sur le port 0 : mais nouveau problème, car la valeur specifiée entre parenthèses est mise sur les fils d'adresse de poids faible, ce qui n'affecte en rien les périphériques. Ce qu'il faut juste savoir, c'est que le registre B est mis sur les fils de poids forts lors d'un a OUT (0),A ». Attention toutefois car il semble que certaines couleurs ne soient pas utilisables avec ce OUT et affectent certains registres du FDC (ah I bug , quand tu nous tiens l.,,). Tout a une fin, et nous voilà arrivé au bout de cet article, duquel j'espère vous retiendrez l'essentiel. Si toutefois certains points ne vous paraissent pas clairs faites-nous-en part afin que nous puissions vous venir en aide.

Digit

Registres de sélection d'encre du Gate Array :

```
&7F00 -> encre 0 (papier)
&7F01 -> encre 1
&7F02 -> encre 2
&7F03 -> encre 3
&7F04 -> encre 4
&7F05 -> encre 5
&7F06 -> encre 6
&7F07 -> encre 7
&7F08 -> encre 8
&7F09 -> encre 9
&7F0A -> encre 10
&7F0B -> encre 11
&7F0C -> encre 12
&7F0D -> encre 13
&7F0E -> encre 14
&7F0F -> encre 15
&7F10 -> bord
```

```
;Progresse d'examplo de RASTER per DIGIT de LOGON SYSTEM pour Amatred 100%
:pour DANS lea DS n devt«nnent DEFS n.0

TO BE TYPED IN - OCR is unusable
```
