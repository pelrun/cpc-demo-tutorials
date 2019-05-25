UNE DEMO DECORTIQUEE
====================

On a souvent reproché à la rubrique Logon son caractère ésotérique, bien que nous ayons toujours précisé que ces articles étaient réservés aux initiés de la programmation ! Mais, comme c'est Noël, nous vous réservons une petite surprise.

Alors ce mois-ci, pour réconcilier tout le monde, nous vous proposons une vraie démo (cf la rubrique listing), avec sprites, split-rasters, scroll en vague, rupture, scroll hard, etc. Tout le monde pourra taper, et nous nous en tiendrons ici aux explications des parties de code les plus poussées de la démo.

AU DEBUT, LE PIXEL FUT
----------------------

Tout d'abord une petite explication concernant le gros logo Logon de la démo. Il n'a pas été dessiné avec un utilitaire de dessin, mais il est composé de caractères de 8 pixels sur 8, ce qui permet donc de gagner beaucoup de mémoire (un logo dessiné aurait pris 16 fois plus de place !). Ce genre de technique est d'ailleurs employée dans toutes les démos (quelle que soit la machine) qui utilisent des fontes énormes pour des scrolls géants, etc.

UN LOGO VAUT MIEUX QUE DEUX TU L'AURAS
--------------------------------------

Dans ce gros logo, bouge de gauche à droite ce qu'on appelle un split-raster : c'est-à-dire que la même encre change plusieurs fois de couleur sur la même ligne écran ; ici c'est l'encre 1 qui passe du noir au blanc, en passant par les teintes rouge-orange-jaune de notre CPC...

Le mouvement de va-et-vient du raster est fait par une attente plus ou moins longue avant de changer la couleur, ce qui produit donc un effet de décalage. Cette attente se traduit par une série de NOPS d'autant moins longue que le split raster est à gauche.
C'est le « JP roll » qui permet ce roulement, « roll » n'étant jamais une adresse fixe (cf listing 1). Nous avons en fait ici un bel exemple de code automodifié. Le programme change ses instructions en fonction de ce qu'il doit faire. Le code s'automodifie également pour le changement de sens de décalage du raster. Comme il n'y a que 2 états possibles (soit de la gauche vers la droite, soit l'inverse), un simple XOR suffit et fait effet de bascule (voir les labels modif 1 et modif 2 du listing 1).
Et puisque j'en suis venu à vous parler de code automodifié, je vais aussi vous parler de code autogénéré.

IL N'Y A PAS QUE LA MUSIQUE QUI SOIT POP
----------------------------------------

Le scroll en vague de la démo fait 96 octets de large sur 6 lignes de haut. Or si l'on fait une boucle sur chaque octet, on perd à chaque fois un temps précieux (rappelez-vous un article Logon sur l'optimisation, il y a déjà pas mal de temps). On peut donc se permettre de perdre un peu de mémoire au profit de la vitesse d'exécution. Or il est pénible de devoir taper 96 fois la même routine, et de plus, cela est moins compactable qu'une série de zéros. On laisse donc la place nécessaire, et on recopie la routine autant de fois qu'il le faut (cf LDIR du second listing).
La routine de scroll en vague utilise la pile pour obtenir l'adresse écran de l'octet à prendre et l'adresse écran où il faut remettre cet octet. Etant donné que l'adresse d'arrivée d'un octet est l'adresse de départ de l'octet qui le précédait, il suffit de ne prendre qu'une seule adresse à chaque fois qu'on s'occupe d'un octet, c'est pour cela qu'il n'y a qu'un POP par octet. Or donc, comme il faut conserver l'adresse précédente, on fait un coup un POP DE, un coup un POP HL (cf listing 2).
Je vous rappelle qu'il est fortement conseillé de couper les interruptions lors de l'utilisation de la pile d'une façon inhabituelle.

XOR, ES-TU LA?
--------------

La rupture utilisée dans la démo pour le scroll hard est de type simple et a déjà été expliquée par Longshot dans un numéro précédent, de même que le scrolling hard en lui-même, qui est, à peu de choses près, le même que celui que Rubi vous offrit en ces pages, il y a un bout de temps...
Les sprites qui se baladent sur le logo sans l'effacer sont affichés par des XOR. Les sprites étant dessinés à l'encre 2 et le logo à l'encre 1, un XOR du sprite sur le logo fera apparaître l'encre 3, qu'il suffit de mettre à la même couleur que l'encre 2 pour qu'on ne voit pas la supercherie !
De plus, l'effaçage se fait de la même façon que l'affichage, en appelant la même routine ! Deux inconvénients à cette méthode : elle ne permet pas l'emploi de toutes les encres disponibles, et si 2 sprites se chevauchent, ils vont « s'autoxorer » et n'apparaîtront plus en entier !
Ici, le mouvement des sprites fait que cet effet ne se voit pas, c'est pourquoi je l'ai utilisé. L'effet de chevauchement peut en effet être évité en faisant des XOR à l'effacement, et des OR à l'affichage. Il faut donc, cette fois-ci, 2 routines, c'est une autre raison pour m'avoir fait choisir la première.

BIENTOT LES SPRITES
-------------------

Voila, c'est à peu près tout pour aujourd'hui. La prochaine fois, nous nous attaquerons aux sprites, et aux méthodes utilisées pour les afficher en passant des plus simples et usuelles aux plus compliquées et inattendues (ÿ a du pain sur la planche, comme on dit !). Bonnes démos et à bientôt !

Pict, qui espère ne pas rater son dernier métro

```
; listing 1

              HALT
              DI
              DEFS 6,0
              LD BC,#7F54
              XOR A
              INC A
              OUT (C),A
              OUT (C),C
              LD A,101
              LD (CTR+1),A
              LD DE,ROLL
ROLLCT        LD HL,0
              ADD HL,DE
              JP (HL)
ROLL          DEFS 43,0
              LD DE,#5C4C
              LD HL,#4E4A
RASTLOOP      LD A,#4B
              OUT (C),D
              OUT (C),E
              OUT (C),H
              OUT (C),L
              OUT (C),A
              OUT (C),L
              OUT (C),H
              OUT (C),E
              OUT (C),D
              OUT (C),C
CTR           LD A,0
              DEC A
              LD (CTR+1),A
              DEFS 12,0
              JP NZ,RASTLOOP
              LD A,#4B
              OUT (C),A
              EI
              LD A,(ROLLCT+1)
MODIF1        ADD A,1
MODIF2        CP 43
              LD (ROLLCT+1),A
              JP NZ,NORESROLL
              LD A,(MODIF1+1)
              XOR #FE
              LD (MODIF1+1),A
              LD A,(MODIF2+1)
              XOR 43
              LD (MODIF2+1),A
NORESROL
```

```
; listing 2

              LD HL,ORIG
              LD DE,DEST
              LD BC,DEST-ORIG*46
              LDIR

              DI
              LD (SAVSP+1),SP

;PILE SUR TABLE DES COORD ECRANS

              LD SP,SINTAB
              LD B,6
SCR           POP HL

ORIG          POP DE
              LD A,(DE)
              LD (HL),A
              POP HL
              LD A,(HL)
              LD (DE),A

DEST
              DEFS DEST-ORIG*46,0

              POP DE
              LD A,(DE)
              LD (HL),A
              DEC B
              JP NZ,SCR
SAVSP         LD SP,0
              EI
```
