RASTER SUITE...
===============

En épluchant le sympathique (enfin dans la plupart des cas) courrier que vous nous faites parvenir, nous en avons conclu que certains d'entre vous souhaiteraient encore plus de choses sur les Rasters (déplacement sinusoïdale, devant et derrière un logo, split raster equaliser, etc.).

Ayant fait le premier article sur les rasters, je me doit de continuer. Je suppose que vous savez déjà faire un raster tout bête au milieu de l'ecran, jusque-là OK, pas de problèmes, mais pour le faire bouger : mystère... Il faut d'abord comprendre le principe des tableaux, un tableau est une zone mémoire dans laquelle on stocke des éléments de même type (type : soit un octet, une adresse, une chaîne de caractères, etc.), on peut accéder à n'importe quel élément du tableau en utilisant un indice qui indique où est rangé l'élément souhaité.

ELEVE FRED ALLEZ AU TABLEAU !
-----------------------------

Dans notre cas, le tableau est la zone mémoire dans laquelle se trouvent les couleurs du raster, le nombre d'éléments correspond au nombre de lignes que fait le raster, donc un raster de 10 lignes n'est ni plus ni moins qu'un tableau de couleurs constitué de 10 éléments.

RASTER: BOUGE DE LA!
--------------------

Pour faire bouger notre raster, il faut donc déplacer les éléments du tableau qu'est notre raster. Une des premières choses simples à faire consiste à faire soit monter ou descendre un raster, ce qui, en terme de programmation, revient à déplacer les éléments d'un tableau d'un rang vers le début du tableau ou d'un rang vers la fin du tableau. Pour faire défiler le tableau vers le haut, il faut remplacer chaque élément d'indice « i » (j'ai pris « i » c'est arbitraire) par son élément suivant d'indice « i+1 » on fait ainsi du début jusqu'à la fin sans avoir oublié de conserver le premier élément qui va se retrouver le dernier à la fin de l'opération. En consultant le petit programme servant d'exemple à l'article, vous constaterez que pour réaliser ce petit prodige, les instructions LDIR et LDDR nous seront utiles (pour l'instant nous parlons de compréhension non pas d'optimisation, donc je ne veux pas de remarque du genre « Hé, y'a plus rapide que LDIR H, n'est-ce pas Longshot ?

VOUS BOUGIEZ, EH BIEN DANSEZ MAINTENANT
---------------------------------------

Toujours plus fort (mais non Fred, je n'ai pas dit de taper plus fort sur PICT) comment faire si on désire faire suivre une trajectoire précise à notre raster (par exemple, effet de rebond ou bien décélération et changement de sens, tout cela de façon propre). Si vous voulez mon avis, rien ne vaut un bon cosinus pour réaliser cette petite merveille (quia dit « Ah bon ! » ?).
Le principe de fonctionnement est le suivant, nous disposons de trois tableaux, le premier (tableau 1) contenant notre beau raster (on suppose qu'il est rouge et qu'il fait 8 lignes de haut) qui va se déplacer ; le second (tableau 2) contenant le raster qui sera réellement visualisé (au départ il doit être rempli de couleur noire, on suppose qu'il fait 50 lignes de haut), et enfin le plus important, un tableau (tableau 3) contenant des valeurs qui vont correspondre aux numéros de lignes que va emprunter le raster rouge lorsqu'il va se déplacer à l'écran (il va en réalité se déplacer dans le tableau 2).
Maintenant, voyons ce que nous allons faire de ces trois tableaux. Tout d'abord, il faut remplir le tableau 2 avec la couleur du fond, ensuite il faut prendre une valeur, que nous noterons « x », dans le tableau 3 et aller copier le tableau 1 (le raster rouge) dans le tableau 2 (le raster écran) à partir de l'indice de « x » (qui correspond à la ligne d'affichage). Oui je sais ce n'est pas évident au début, donc j'en profite pour de nouveau vous dire de vous inspirer du programme d'exemple dans lequel tout les cas sont traités. Vous allez me dire quel rapport avec les cosinus ? Eh bien, je vous répondrai qu'il faut bien remplir le tableau 3 de notre exemple (le tableau contenant le mouvement du raster) avec quelque chose. Avant de faire nos calculs, il faut déterminer la taille de la zone dans laquelle notre raster va se déplacer ; nous avions dit que le raster écran faisait 50 lignes, il ne faut pas oublier de retirer de ces 50 lignes la taille du raster rouge qui est de lignes, et il nous reste donc 42 lignes qui vont correspondre à notre zone de déplacement. L'autre chose à savoir est que les valeurs extrêmes prises par la fonction cosinus (petit rappel mathématique) sont 1 et -1. Avec tout cela, il vous suffit de consulter le programme Basic qui génère le tableau de déplacement du raster, ce programme se chargera de poker le tableau en mémoire, ensuite de charger le programme binaire que vous aurez pris soin de sauvegarder au préalable sous le nom de RASTER.BIN, et de lancer le programme binaire.
Me voilà arrivé à la fin de cet article, mais dites-vous bien que le sujet des RASTER n'est pas clos et que l'on risque de vous en reparler plus tard.

Digit

```
10 'Programme genrant les tables de sin et cos
20 MEMORY &4000-1:DEG:MODE 1:PRINT"Calculs en cours."
21 adr=0:FOR i=0 TO 359 STEP 360/256
22 POKE &5000+adr,42/2+COS(i)*42/2:POKE &5100+adr,42-ABS(COS(i)*42)
23 adr=adr+1:NEXT
30 MODE 1:LOAD"prog2.bin":CALL &4000
```

```
; PROGRAMME D'EXEMPLE DE RASTER PAR DIGIT
; DE LOGON SYSTEM POUR AMSTRAD 100%
; pour DAMS les DS n deviennent DEFS n,0

              org #4000
mouvement     equ #5000       ; PREMIER TABLEAU
mouvement2    equ #5100       ; DEUXIEME TABLEAU
              di

              ld hl,#C9FB     ; DETOURNE INTERRUPTION
                              ; AVEC EI, RET
              ld (#0038),hl
              im 1            ; NO EST JAMAIS TROP SUR
              ei
synchro       halt            ; ON ATTEND LE PREMIER HALT
              ld b,#F5
              in a,(c)        ;A=PORT(F5XX)
              rra
              jp nc,synchro
              ld hl,raster
              ld de,raster+1
              ld bc,49
              ld (hl),#54
              ldir
pointe_mov    ld hl,mouvement
              ld e,(hl)
              inc l
              ld (pointe_mov+1),hl
              ld d,0
              ld hl,raster
              add hl,de
              ex de,hl
              ld hl,raster_rouge
              ld bc,8
              ldir
              ld hl,raster2
              ld de,raster2+1
              ld bc,49
              ld (hl),#54
              ldir
pointe_mov2   ld hl,mouvement2
              ld e,(hl)
              inc l
              ld (pointe_mov2+1),hl
              ld d,0
              ld hl,raster2
              add hl,de
              ex de,hl
              ld hl,raster_rouge
              ld bc,8
              ldir

              halt            ;ON EST DANS LA PARTIE VISIBLE
                              ;DE L'ECRAN
              DS 25           ;ON ATTEND POUR QUE LE RASTER
                              ;COMMENCE
                              ;EN DEBUT DE LIGNE
              ld bc,#7F00     ;SELECTIONNE PAPIER POUR TOUTE
                              ;LA DUREE DU
              out (c),c       ;RASTER
pointeur      ld hl,raster    ;HL POINTE LA TABLE DE COULEURS
              ld de,#0010     ;E=#10, BORDER D=0, PAPIER
                              ;ON VA ALTERNER BORDER/PAPIER
              ld a,50
aff_raster2   ld c,(hl)

              inc l
              out (c),e       ;SELECTIONNE BORDER
              out (c),c       ;MET COULEUR BORDER
              out (c),d       ;SELECTIONNE PAPIER
              out (c),c       ;MET COULEUR PAPIER
              DS 41
              dec a
              jp nz,aff_raster2
              ld a,#54        ;COULEUR NOIR
              out (c),a       ;MET PAPIER EN NOIR
              out (c),e       ;SELECTIONNE BORDER
              out (c),a       ;MET BORDER EN NOIR
              halt            ;ON EST DANS LA PARTIE VISIBLE
              DS 25           ;ATTENDRE DEBUT RASTER
                              ;EN DEBUT DE LIGNE
              ld bc,#7F00     ;SELECTIONNE PAPIER POUR TOUTE
              out (c),c       ;LA DUREE DU RASTER

pointeur2     ld hl,raster2   ;HL POINTE SUE LA TABLE COULEURS
              ld de,#0010     ;E=#10, BORDER D=0, PAPIER
                              ;ON ALTERNEE BORDER/PAPIER
              ld a,50
aff_raster1   ld c,(hl)
              inc l
              out (c),e       ;SELECTIONNE BORDER
              out (c),c       ;MET COULEUR BORDER
              out (c),d       ;SELECTIONNE PAPIER
              out (c),c       ;MET COULEUR PAPIER
              DS 41
              dec a
              jp nz,aff_raster1
              ld a,#54        ;COULEUR NOIR
              out (c),a       ;MET PAPIER EN NOIR
              out (c),e       ;SELECTIONNE BORDER
              out (c),a       ;MET BORDER EN NOIR
              jp synchro      ;ON RECOMMENCE
raster        ds 50
raster2       ds 50
raster_rouge  db 76,78,74,67,67,74,78,76
```

LE BUG
======

Eomme vous avez été nombreux à le remarquer, notre article du n"40 sur les connexions banques du CPC a été honteusement saboté !

Je vous livre donc la partie de l'article qui manquait, ainsi que les lignes 2080 à 2270 de l'utilitaire Mpack qui s'étaient perdues entre les pages 41 et 42 ! Mille excuses pour ce fâcheux contretemps !

CONNEXION DES BLOCS
-------------------

En parlant de la page 0, que se passe-til lorsqu'on sélectionne un bloc sur celle-ci (donc lorsque les bits 4,3 et 2 sont à 0)... Contrairement à ce qu'on pourrait penser, les blocs 0, 1, 2 et 3 ne sont pas placés à l'adresse #4000.. # 7FFF... ceci pour la bonne raison qu'une telle connexion aurait été absurde puisque la Ram centrale est déjà disponible... C'est pourquoi les concepteurs du CPC ont opté pour un « set » de combinaisons de pages habilement pensé.

Sélectionner le bloc 0 de la page 0 permet de mettre la Ram dans son état standard...
La connexion des blocs 1 et 3 de la page 0 offre des combinaisons très intéressantes, surtout pour les concepteurs de jeux où de démos (bien que personne n'ait semblé l'utiliser à ce jour... A vos claviers !!). La connexion du bloc 2 a été conçue dans l'optique CP/M +, où il était nécessaire de disposer d'une zone TPA très large... Cette Rani devait être très grande et surtout linéaire, d'où un grave problème puisque le CP/M + occupe plus de 30 K en Ram centrale. C'est pourquoi vous disposez de 61 K de Ram avec le CP/M +. Pourquoi pas 64 K, me direz-vous ?

Tout simplement parce que la connexion du bloc 2 de la page 0 (Fction 11x00010) ne se fait pas sans précaution puisque l'ensemble de la Ram est bascule (64 K). Ce qui peut être très fâcheux pour le programme qui effectue la connexion car cette dernière l'efface... à moins que... on ait prévu dans la Ram qui va se connecter un programme situé juste après le « out de connexion » et c'est ce qui est fait pour CP/M+...
Le schéma longshotien (Commutation des Banks-Rams. Cent pour Cent N°40 page 40), qui, je l'espère, sera découpé et collé dans les références book des plus fanatiques de la programmation hard du CPC, représente l'ensemble des combinaisons Ram possibles avec un 6128... c'est-à-dire la page 0 et la page 1 (chacune ayant 4 blocs).
Concernant le schéma, quelques précisions... Les blocs 0, 1, 2, 3 correspondent aux 4 premiers blocs de 16 K de la Ram principale.

A noter : La Ram Vidéo n'est prise en compte que sur la Ram principale, même si une banque est connectée à la place de la VidéoRam.
Une banque connectée à la place de la VidéoRam aura pour seule conséquence de vous empêcher d'avoir des accès en Lecture/Ecriture sur la Ram Vidéo. Le problème est un peu similaire lors de la connexion des Rom avec la seule différence que les accès en écriture auront lieu dans la Ram connectée (phénomène de transparence des Rom). Les accès en lecture fourniront bien évidemment le contenu de la Rom connectée. Et pour finir sur les priorités Ram/Rom, précisons aussi que la page I/O Asic (pour les possesseurs de CPC Plus) est prioritaire sur toutes les Ram lorsqu'elle est connectée.

Les blocs 4, 5, 6 et 7 correspondent aux blocs supplémentaires disponibles d'origine sur 6128 ou fournis par l'adjonction d'une extension DKTronic. Ils sont en fait les blocs 0, 1, 2 et 3 de la page 1.
d'acquérir une extension Dk'Tronic

Voilà, je pense qu'à présent les Ram de 64 K (mais non, ils ne me payent pas !) votre CPC n'ont plus de secrets pour C'est une bonne manière de découvrir vous. Si vous possédez un 464 ou 664 sans extension, il est toujours temps « The Demo », non ?

Longshot
