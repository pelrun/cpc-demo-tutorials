LOGON SYSTEM
============

Ouf, on se remet à peine de l'Amstrad Expo. Si vous vous rappelez bien, nous parlions le mois dernier du CRTC où j'ai décrit comment étendre le compteur vidéo sur 32 Ko pour pouvoir afficher une image overscan. Cette gestion étant contrôlée par les bits 2 et 3 du registre 12.

Nous allons cependant passer à des choses plus sérieuses que l'overscan, en l'occurrence, de la Rupture. Comme je l'expliquais la dernière fois, un écran à 50 Hz donne, pour une fréquence de balayage de 15625 Hz. envi ron 312 lignes à parcourir (c'est à dire 15625/50). Ce nombre de lignes est programmé dans le CRT à l'aide de 3 registres qui sont (on ne souffle pas, au rond !) les registres 4,5 et 9. La formule magique pour calculer ce nombre de lignes est :

```((Reg4+1)*(Reg9+1))+Reg5```

Cela nous donne, avec les valeurs de la Rom:

```((38+1)*(7+1))+0 = 313```

On ne va pas chipoter pour une ligne.
Comme vous l'avez compris, le Reg 5 permet d'etlectuer un réglage « fin » du nombre de lignes total et, par conséquent, sa valeur maximale est identique à la valeur maximale du Reg 9.
Le Reg 9 détermine pour le CRT le nombre de lignes caractères, ou si vous préférez le nombre de blocs de 2 Ko utilisés pour l'affichage.
Je m'explique : le nombre dc lignes caractères maximum sur Amstrad est 8 (le CRT peut en gérer plus mais la mémoire vidéo ne fait que 16 Ko).
Lorsque le Reg 9 a une valeur (standard) de 7, les 8 blocs seront utilisés et les « caractères » (ou tramage des lignes vidéo) occuperont 8 lignes vidéo.
Quelqu'un a des questions? Non ! OK je continue.
Pas mal de gens ont un jour essayé de modifier, durant le balayage vidéo, l'adresse où le CRT va pêcher ses données. Pas mal de gens se sont ainsi exclamés : ça maaarche paaaaas !!! Pourquoi ? Tout simplement parce que le CRT suit des règles de programmation internes au circuit, très strictes et qu'il refuse obstinément de changer l'offset tant qu'il n'a pas fini d'afficher son nombre de lignes total. En fait, et pour être plus clair (hem, hem). l'adresse de l'écran ne peut être changée que durant la VBL.

VBL, HBL ?
----------

La VBL (Vertical BLanking) représente le laps de temps durant lequel le canon électron, qui vient péniblement de rafraîchir la dernière ligne (et se trouve donc en bas de l'écran), effectue une trajectoire (diagonalement parlant) pour revenir tout en haut. A ce temps, est ajoutée une autre période dont je parlerai après, faut pas tout mélanger (je précise tout de même que cette période servira à nous synchroniser).

Bon, que disais-je ? Ah oui, que devient l'écran vidéo durant la VBL? Eh bien tout simplement rien n'est affiché, le néant absolu ! C'est ce qu'on appelle l'overscan ! Durant ce laps de temps, rien ne peut être afliché, d'où la zone noire que l'on nomme communément Border. Attention cependant, le véritable overscan est une zone très petite. Pourquoi véritable ? Parce que les bords que vous voyez habituellement ne sont pas le vrai overscan, puisque 312 lignes sont affichées par le CRT.
Seulement 200 lignes de données sont affichées verticalement, mais ça c'est une autre histoire (cf. Reg 6 du CRT). Tant que j'y suis, je, décris aussi la HBL (Horizontal BLanking), qui est le laps de temps que met le canon à électron, lorsqu'il a fini d'afficher une ligne, pour revenir (toujours diagonalement parlant) au début de la ligne suivante.
Les zones noires générées par la HBL et la VBL ne sont pas « visibles » puisque les moniteurs Amstrad sont trop petits. Bon, de toute façon j'aurais l'occasion de revenir sur HBL et VBL en temps voulu si cette rubrique connaît le succès , mais on a déjà parlé de me brûler en place publique...

LA RUPTURE
----------

Je disais donc qu'il est possible de changer l'adresse de l'écran durant la VBL. Aussi, lumineuse idée, nous allohs feinter le CRT en lui faisant générer une VBL un peu plus tôt ; nous savons déjà que celle-ci survient une fois que le nombre total de lignes a été affiché. Pour cela, il est nécessaire de reprogrammer le nombre de lignes total affiché par le CRT à l'aide de la formule donnée plus haut.
Cependant, la modification de ce nombre de ligne entraîne un problème de taille : l'écran n'est prévu que pour gérer des images à 50 Hz (plus ou moins proche). Aussi, il est impératif de reprogrammer les registres 4,5,9 autant de fois que nécessaire pour que le nombre de ligne total affiché pendant un cinquantième de seconde soit égal à 312.
Hem, y'en a encore qui suivent? Bon, prenons un exemple très simple. Une rupture communément utilisée par de nombreux démomakers. Nous avons décidé de couper l'écran en deux. Premièrement, munissez-vous d'une scie bien affûtée. Nous allons donc afficher la moitié des lignes en reprogrammant le CRT : 38/2=19 d'où ((19+1)x(7+1))+0=160 lignes (donc R4=19, R9=7 et R5=0).
Tant que nous y sommes. pourquoi ne pas en profiter pour aussi définir l'adresse de départ du premier bloc vidéo ? Donc, saisissez le listing Assembleur pour reprogrammer consciencieusement le CRT.
Une fois que cela est fait, le CRT exécute les ordres du commandant de bord et va afficher le nombre de lignes demandées. Lorsque toutes les lignes desirées ont été affichées, il est nécessaire de reprogrammer le CRT pour compléter le nombre de lignes total à 312. Il faut donc modifier les registres 4, 5, 9. 12 et 13 derechef.

BUFFERISER LES REGISTRES
------------------------

Un truc intéressant : il est possible de « bufferiser » certains registres, c'est-àdire de les pré-charger pour ne pas être obligé de les modifier alors que 160 lignes ont été affichées. Cette « bufférisation » est possible pour les registres 12 et 13 ; vous pouvez ainsi indiquer l'adresse de la seconde zone, alors que la première n'a pas fini d'être affichée, elle ne sera prise en compte que lors de la prochaine (fausse) VBL.
Généralement, il est nécessaire de reprogrammer le registre 4 pour avoir un nombre total de 312 ; cependant nous avons ici un exemple très simple puisqu'il faut normalement reprogrammer 19 dans le registre 4. Or c'est la valeur que nous avions fixée la première fois (l'écran est coupé au centre) et il faut savoir que lorsque les registres CRT ont fini de décompter, ils sont rechargés avec leur valeur initiale.
Il est donc inutile dans ce cas de reprogrammer le Reg 4 du CRT. Et le registre 6 alors? Bien que le rôle de ce registre n'ait rien à voir avec la fréquence (le balayage (Reg 4, 5 et 9), il détermine le nombre de « caractères » données affichées.
En effet, je vous ai parlé tout à l'heure des 200 lignes de données affichées sur les 312 réellement gérées. Cela est obtenu par la formule numéro 2, Abracadabra : (Reg6x(Reg9+1)) ce qui nous donne en standard, 25x(7+1)=200).
Lorsque vous redéfinissez votre écran comme disposant de moins de lignes vidéo dans certaines parties, il est aussi préférable (mais pas nécessaire) de reprogrammer ce registre. Attention, la rupture étant une technique consistant à construire en temps réel son écran en reprogrammant sans arrêt le CRT, la nouvelle configuration de l'écran existe seulement dans le cas où le Contrôleur Vidéo est reprogrammé tous les 50e de seconde.

De plus, la gestion de l'écran étant prise en main par le programmeur, l'affichage vertical peut commencer à la ligne désirée, en l'occurence, à partir du moment où le programmeur décide d'effectuer sa rupture.
Mais que se passe-t-il si la valeur du registre 4 est changée alors que le compteur de lignes n'a pas atteint 0?
Eh bien logiquement, la nouvelle valeur est rechargée et le compteur repart de plus belle : rappellez-vous cependant que c'est le nombre de lignes réellement affichées qui compte.
Ainsi, un mauvais calcul pourra entraîner la génération d'un écran dont la fréquence sera différente de 50 Hz dans la limite de tolérance du moniteur. Donc la période d'apparition du flag de synchro verticale (Port F5) apparaîtra plus ou moins vite en tonction de la fréquence du nouvel écran généré.
C'est pourquoi les musiques écoutées dans certaines démo sur CPC vont plus ou moins vite par rapport à la version originale qui était jouée toutes les 0,02 secondes. Reste un problème : l'écran n'est plus stable. Cela a un rapport étroit, vous vous en doutez, avec les registres CRT de synchronisation mais nous les passerons en revue le mois prochain...
Quoi qu'il en soit, j'attends toutes vos réactions et suggestions par courrier. Je rappelle évidemment que cette rubrique est destinée aux personnes connaissant l'assembleur. Après tout, il est toujours temps de vous y mettre...
Longshot

```
PRG	LD B,#F5	; Selection du PPI port B
SYNC	IN A,(C)	; Retirer Byte
	RRA		; Bit 0=1 pour indiquer la fin de la VBL
			; (debut du balayage vertical tous les 1/50eme)
	JP NC, SYNC	; Attente de la Synchro Verticale
	LD BC,#BC04	; Selection du registre 4
	OUT (C),C
	LD BC,#BD00+19	; Envol de la valeur 19
	OUT (C),C
	LD BC,#BC09	; Selection du registre 9
	OUT (C),C
	LD BC,#BD00+7	; Envol de la valeur 7
	OUT (C),C
	LD BC,#BC05	; Selection du registre 5
	OUT (C),C
	LD BC,#BD00	; Envoi de la valeur 0
	OUT (C),C
	LD BC,#BC0C	; Selection du registre 12
	OUT (C),C
	LD BC,#BD30	; Envoi de la valeur binaire 00110000
	OUT (C),C	; Ecran en #C000
	LD BC,#BC0D	; Selection du registre 13
	OUT (C),C
	LD BC,#BD00	; Envoi de la valeur 0
	OUT (C),C	; (Reste de l'adresse video)
```
