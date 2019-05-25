FAITES ONDULER VOS ECRANS
=========================

Eh oui, enfin les vacances! C'est super, et cela va vous permettre de consacrer plus de temps à votre CPC et, plus particuliérement dans notre cas, au Z-80. Je suis sûr qu'entre deux séances de bronzette vous trouverez bien le temps de vous amuser avec le listing Assembleur de ce numéro d'été.

Nous allons nous initier aux ondulations d'écran, appelées également « screen waggle » dans le jargon des démomakers (screen pour écran et waggle pour remuer, tortiller, agiter). Et vous allez voir que cette pratique qui semble être réservée à des machines plus puissantes est tout à fait réalisable sur un CPC.

A LA MANIERE D'UN RASTER
------------------------

Le principe consiste en un changement successif des valeurs contenues dans le registre deux du CRTC, à la manière d'un raster. C'est-à-dire qu'à chaque ligne parcourue par le faisceau d'électrons de l'écran, nous allons modifier le contenu de ce registre. On a alors l'impression que l'écran se tord et ondule, suivant — par exemple — une table sinusoïdale, et on obtient les effets si fréquents sur ST ou Amiga... sur CPC ! Puisqu'un bon exemple vaut mieux qu'un long discours, nous allons commenter un listing source mettant en évidence le « screen waggle » avec un raster de couleur en prime.
Avant tout commentaire, tapez d'abord le listing source puis le programme Basic suivant :

```
10 MEMORY &8FFF:LOAD "TRUC.BIN
20 MODE 0:INK 0,0:BORDER 0
30 LOCATE 6,5:print"VIVE LOGON SYSTEM"
40 LOCATE 6,7:print"VIVE AMSTRAD 100%"
50 CALL &9000:MODE 1:INK 1,26
RUN
```

LES COMMENTAIRES
----------------

Commençons par l'initialisation : il faut d'abord déconnecter le système. Je rappelle pour les néophites que cela permet un gain de temps machine et de précision non négligeable puisque le Z-80 n'est plus interrompu pour la gestion des variables Basic, etc.
Puis, nous mettons le border et le fond d'écran en noir (le numéro correspondant est 854). Nous attendons ensuite le début du balayage de l'écran par le faisceau d'électrons. Nous attendons maintenant trois « Halts » afin que le raster soit bien visible sur l'écran. Un peu de patience, nous attendons encore quelques cycles (étiquette « centre »), afin d'être synchronisé avec l'HBL.

Ensuite, faisons pointer HL sur la table de couleurs et DE sur la table des valeurs d'ondulation du registre deux. Chargeons B avec 40, car nous désirons un raster de 40 lignes. Sauvons alors BC (puisque B est le compteur des rasters), et sélectionnons l'encre 1 du Gate Array dans laquelle nous attribuons la couleur pointée par HL dans la table (cela grâce à l'instruction OUTI). Le contenu du registre 2 est alors changé par un OUT (C),C classique car OUTI fonctionne bizarrement avec le CRTC (je n'ai pas dit que ça ne marchait pas — il y a un moyen — mais c'est assez bizarre). Ensuite la boucle « tempo » temporise (d'où le nom) un peu afin d'atteindre l'HBL. Puis nous recommencons jusqu'à épuisement du registre B.
A la fin du raster, nous ramenons l'encre 1 à sa couleur initiale (&4b est le numéro du blanc). Nous redonnons également sa valeur initiale au registre 2 du CRTC (46), et nous en avons terminé avec le raster et l'ondulation !

A SCROLLE EN PLUS
-----------------

Mais afin de donner l'impression que la vague « scrolle », nous utilisons la ruse du décalage de pointeur. Elle est bien plus rapide qu'un simple scrolling de données, en revanche, elle est deux fois plus gourmande en mémoire. Comme son nom l'indique, cette technique consiste à décaler un pointeur le long d'une table (ici la table des ondulations) et à le réinitialiser en fin de table, C'est pour cette raison que ladite table d'ondulation est doublée.

RETOUR AU SYSTEME
-----------------

En fin de listing, nous faisons un test classique de la barre d'espacement. Nous revenons au système si celle-ci est enfoncée. Vous avez donc un petit programme qui, réuni avec les autres de la série, vous permettra de vous attaquer l'élaboration d'une super démo, Qûestion technique, vous pourrez la montrer sans honte à vos amis qui possèdent un Atari ST ou un Amiga, surtout si ceux-ci ne font qu'admirer les démos tournant sur leurs machines.
Il ne me reste plus qu'à vous souhaiter de bien bronzer et de bien programmer !
Emmanuel, alias Pict

```
; (C)Logon System 1990
; Ecrit par Pict
          ORG #9000

; Sauve Système
          DI
          LD HL,(#38)
          LD (INTER+1),HL
          LD HL,#C9FB
          LD (#38),HL
          EI

; Border noir
          LD BC,#7F10
          OUT (C),C
          LD A,#54
          OUT (C),A
          LD C,0
          OUT (C),C
          OUT (C),A

LOOP
          LD B,#F5
VSYNC     IN A,(C)
          RRA
          JP NC,VSYNC

; on attend que l'écran soit visible

          HALT
          HALT
          HALT

; "Centrage" du raster

          LD B,5
CENTRE
          DJNZ CENTRE
          LD HL,COLOR
VAR       LD DE,ONDUL
          LD B,40           ;Taille Raster
MLOOP
          PUSH BC
          LD BC,#7F01
          OUT (C),C
          OUTI              ;Change la couleur
          LD BC,#BC02
          OUT (C),C
          INC B
          LD A,(DE)
          INC DE
          OUT (C),A         ;Décale le border
          LD B,6

; Synchronisation HBL

TEMPO     DJNZ TEMPO
          POP BC
          DJNZ MLOOP
          LD BC,#7F01       ;Border noir
          LD A,#4B
          OUT (C),C
          OUT (C),A
          LD BC,#BC02       ;Recentre l'ecran
          OUT (C),C
          LD BC,#BD00+46
          OUT (C),C

;Gestion du décalage de la vague

          LD HL,(VAR+1)
          INC HL
VARCOM    LD A,1
          DEC A
          JP NZ,NORESET
          LD A,30
          LD HL,ONDUL

NORESET
          LD (VARCOM+1),A
          LD (VAR+1),HL

; Test de la barre espace

KEY       LD BC,#F40E
          OUT (C),C
          LD BC,#F6C0
          OUT (C),C
          XOR A
          OUT (C),A
          LD BC,#F792
          OUT (C),C
          LD BC,#F645
          OUT (C),C
          LD B,#F4
          IN A,(C)
          LD BC,#F782
          OUT (C),C
          LD BC,#F600
          OUT (C),C
          RLA
          JP C,LOOP
INTER     LD HL,0
          LD (#38),HL
          RET

; Variables Registre 2 crtc

ONDUL
          DB 46,47,47,48,48,48
          DB 49,49,49,49,48,48,48
          DB 47,47,46,45,45,44
          DB 44,44,43,43,43,43
          DB 44,44,44,45,45
          DB 46,47,47,48,48,48
          DB 49,49,49,49,48,48,48
          DB 47,47,46,45,45,44
          DB 44,44,43,43,43,43
          DB 44,44,44,45,45
          DB 46,47,47,48,48,48
          DB 49,49,49,49,48,48,48
          DB 47,47,46,45,45,44
          DB 44,44,43,43,43,43
          DB 44,44,44,45,45

; Variables couleurs ink 0

COLOR
          DB #5C,#4C,#5C,#4C,#4C
          DB #4E,#4C,#4E,#4E,#4A
          DB #4E,#4A,#4A,#43,#43
          DB #43,#43,#4B,#43,#4B
          DB #4B,#43,#4B,#43,#43
          DB #4A,#4E,#4E,#4C,#4E
          DB #4C,#4C,#5C,#4C,#5C
```
