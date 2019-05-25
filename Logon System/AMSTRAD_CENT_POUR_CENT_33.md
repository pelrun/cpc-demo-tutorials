LOGON SYSTEM
============

Pour ce mois de janvier, c'est Digit, l'un des neuf acolytes des Logon System (mais aussi le rédac' chef de MétroBoulot-Micro) qui vous éclaire sur les interruptions du Z80.

Ainsi, il m'a été imparti la dure tâche de vous expliquer les interruptions du Z80. Imaginez que vous soyez en train de jouer à l'un de vos ludiciels favoris... Durant le jeu, le Z80 doit effèctuer plusieurs choses différentes (musique, scanning clavier, etc.) et cela à intervalles réguliers. C'est là qu'intervient la notion d'interruption car le microprocesseur doit être interrompu du programme qu'il exécute afin de pouvoir se charger (les sousprogrammes d'interruptions. Sur le CPC, les interruptions sont déclenchées par un TIMER (système à compteur qui divise la fréquence de l'horloge afin d'obtenir une fréquence moins élevée) qui active la broche INT du Z80 tous les 300e de seconde.

IM 0, lM 1 et IM 2 KESAKO ?
---------------------------

Comment se comporte le Z80 lorsque se produit une interruption? Plusieurs cas de figure se présentent : tout d'abord, pour qu'une interruption soit prise en compte lorsqu'elle apparaît, il faut qu'elle ait été préalablement autorisée. Cela se fait grâce à l'instruction EI. Ensuite le comportement du Z80 dépendra du mode dans lequel il travaille (ATTENTION !! il ne s'agit pas là du mode graphique). Il existe trois modes que l'on spécifie grâce à l'instruction IM (comme Interrupt Mode), qui sont IM 0, IM 1 et IM 2.

LE MODE IM 0
------------

Commençons par le mode IM 0: ce mode d'interruption est identique dans son fonctionnement à celui du microprocesseur 8080 (prédécesseur du Z80 et fabriqué par Intel). Ce mode ne sert que si l'on désire utiliser des périphériques compatibles avec le 8080. Ce qu'il faut savoir, c'est que chaque périphérique a un Restart qui lui est alloué et qu'à chaque interntption, il place son numéro de Restart sur le bus de données afin que le Z80 exécute l'instruction RST correspondante (cela va de RST OOH jusqu'à RST 38H). Ce mode n'est pas utilisable sur CPC.

LE MODE IM1 (DU DEJA VU)
------------------------

Passons au mode IM 1. Ce mode est toujours utilisé sur CPC. Lorsqu'une interruption se produit dans ce mode, le Z80 exécute un RST 38H. Donc, à l'adresse 38H, se trouve le sous-programme d'interruption (ce qui est valable pour tous les modèles de CPC ).

En farfouillant dans les sources de nombreuses démos, on constate qu'à cette adresse se trouve l'instruction RET. Elle interdit les interruptions (un DI en début de programme est plus simple). Mais une autre solution est préférable si l'on désire avoir uniquement le déclenchement d'interruption afin de se synchroniser sur le TIMER.
Elle consiste à placer les instructions EI, RETI (note de Longshot : « ou EI, RET pour aller plus vite ») à partir de l'adresse 38H. En utilisant cette méthode, il sera ensuite possible de se synchroniser avec un HALT n'importe où dans son programme (pour avoir, par exemple, des rasters bien synchronisés ou pour faire de la rupture).
Car si on se contente de détourner les interruptions avec un RET, cela empêche de valider le retour d'interruption et interdit donc ainsi les interniptions suivantes. Alors que le fait de mettre un El valide les flags internes de masquage d'interruption du Z80 (qui sont IFFI et IFF2). Ce qui signifie en clair que les interruptions sont à nouveau autorisées en retour du sousprogramme d'interruption.

LE MODE IM 2 (ENFIN DU NOUVEAU)
-------------------------------

Nous allons terminer par le mode IM 2 (appelé aussi mode vectorisé) qui est. à mon avis, le plus intéressant. Dans ce mode, l'adresse du sous-programme d'interruption n'est pas connue d'avance par le Z80. Il faut alors utiliser le registre I (eh oui, il sert à quelque chose !). Ce registre va déterminer le poids fort de l'adresse à partir de laquelle se trouvé la table des vecteurs d'interruptions : quant au poids faible de la table des vecteurs, il est fourni par le périphérique qui génère l'interruption (oui. je sais, ce n'est pas très clair).
Donc, supposons que l'on soit en mode IM 2 et qu'une interruption se produise. Le Z80 va prendre la valeur du registre I et va lire son bus de données (ce qui fait 8 bits avec le registre I et 8 hits lus sur le bus). Avec ces deux valeurs, le microprocesseur connaît maintenant l'adresse de la table des vecteurs (supposons que ce soit en 400OH). Il va lire 16 bits à partir de 4000H, ces 16 bits formant une aciresse à laquelle il va sauter.
Je vais essayer de vous donner un exemple concret. On suppose qu'à l'adresse 5000H se trouve notre sousprogramme d'interruption, et qu'à l'adresse 4000H se trouve notre table des vecteurs :
Table vecteurs sous-programme d'interruption

```
#4000 : #50 #5000 : #FB : EI
#4001 : #00 #5001 : #ED ; RETI
#4002 : #50 #5002 : #4D
#40FF : #00
#4100 : #50
```

Le. registre I doit être égal à 40H (poids fort de 4000H). Sur CPC, le mode IM 2 étant mal câblé, on ne connaît pas le poids faible ; nous sommes donc obligé de remplir la zone mémoire de 4000H à 4OFFH avec la valeur 5000H.

Comme tout ne va pas sans mal sur CPC. non content d être mal câblé, le mode IM 2 est de surcroît buggé. car il faut aller jusqu'en 4100H écrire l'adresse 5000H. L'avantage de ce mode est qu'on peut mettre les interruptions dans les banks d'extensions et avoir ainsi 64 Ko de mémoire vidéo...

ET AVEC CA, CE SERA tOUT ?
--------------------------

Un petit programme d'exemple figure dans les environs, il illustre la façon d'utiliser le mode IM 2 sur son petit CPC à soi, malgré ce « %#&!'?$ » de bug inhérent au câblage.
Digit

Cet exemple sert a illustrer l'installation des interruptions en mode IM 2. On fait changer la couleur du BORDER afin de verifier que le HALT se produit correctement (on aurait pu egalement s'amuser avec des rasters)

```
	org #4000
table_vecteur	equ #5000
	di
	ld hl,table_vecteur
	ld bc,inter
	ld (hl),b
	inc hl
	ld (hl),c
	ld e,l
	ld d,h
	inc de
	dec hl
	ld bc,#FF
	ldir
	ld a,#50
	ld i,a
	im 2
	ei

synchro	ld b,#F5
	in a,(c)
	rra
	jp nc,synchro
	halt
	halt
	ld bc,#7F10
	ld a,04CH
	out (c),c
	out (c),a
	halt
	ld a,054H
	out (c),a
	jp synchro
	;
inter	ei
	reti
```