RASTER (suite et fin)
=====================

Tout d'abord je vous présente toutes mes excuses pour l'incident du mois précédent : eh oui ! peut-être que certains d'entre vous (ceux qui tapent les programmes) ont constaté qu'il manquait le programme Basic utile pour créer les tables de mouvement du raster (zut alors ! c'est donc pour cela que le raster était immobile...).

Le listing doit normalement (je dis bien : normalement) se trouver parmi ces pages.
Une petite remarque au sujet du but que le Logon System s'est fixé lorsque nous avons commencé cette rubrique : le but des programmes d'exemple n'est pas de vous fournir des routines utilisables telles quelles dans vos petites démos, mais bel et bien de vous faire comprendre le principe des routines afin que vous puissiez ensuite les récrire (à votre sauce) pour vos propres réalisations. Donc inutile de nous demander des routines prêtes à l'emploi directement intégrables dans un autre programme, nous ne fournissons que des exemples : le lecteur doit lui aussi faire un effort de compréhension (ça sert à quoi que Logon se décarcasse ?).

VA Y AVOIR DU BOULOT
--------------------

Comme convenu, nous allons continuer sur les rasters. Vous allez me dire que cela fait maintenant le troisième article à leur sujet : pour l'instant nous avons seulement vu la partie cachée de l'iceberg, car j'ai uniquement parlé de rasters lignes ou bien de split rasters simples, mais il me reste encore à vous parler des rasters clans plusieurs encres (impression de passage derrière puis devant un plan), rasters en diagonale, scrolling en split rasters, défilement d'un damier avec des rasters (il suffit de regarder les démos allemandes pour voir de nombreuses utili sations de ces rasters). Au niveau de la programmation, les plus simples à réaliser sont les rasters en diagonale, car il s'agit en fait de simples split rasters dont la durée de synchronisation ligne est soit supérieu- re soit inférieure à 64 microsecondes.
Le schéma 1 est la représentation symbolique du déroulement d'une routine de raster en diagonale. Dans notre programme d'exemple (qui se trouve dans ces pages, enfin qui devrait s'y trouver...), comme on désire faire une bande de couleur, il faut qu'elle soit encadrée de deux changement de cou leur en noir. On peut donc déjà prévoir la durée totale des instructions contenues dans la boucle, à chaque changement de couleur on peut compter 4 microsecondes puisque chaque changement de couleur s'opère grâce a une instruction OUT ; il faut en plus comp ter la gestion d'une variable qui servira de compteur de boucle (la variable est sur 8 bits puisqu'on ne dépassera pas 256 lignes de rasters), donc il nous faut comptabiliser un DEC x (décrémentation d'un registre 8 bits quelconque) apres la mise à jour par le DEC de la variable. Il ne faut pas oublier de tester la condition de sortie de boucle et, si cette condition est fausse, de continuer la boucle, ce qui correspond à un saut conditionnel qui utilise 3 microsecondes. Dans l'exemple que j'ai choisi (j'ai encore mon mot à dire dans cet article : je choisis l'exemple que je veux, na !) il y a sept couleurs qui se succèdent dans le raster en diagonale, au nombre desquelles on doit ajouter le changement de couleur pour passer en noir, puis la mise à jour du registre ainsi que le saut conditionnel, donc on obtient :

8 OUT + un DEC + 1 JP c'est-à-dire
8*4+1+3=36 microsecondes.

Attention, car on ne doit pas se hâter pour en déduire la durée de temporisation pour la boucle, car dans notre exemple, et pour des raisons de facilité, je nie suis servi des registres secondaires (j'espère que je n'ai pas à vous rappeler que ces registres sont accessibles en utilisant l'instruction EXX), ce qui nous oblige à ajouter encore deux microsecondes (puisque EXX en prend une seule mais qu'il y a deux EXX dans la boucle) et nous donne pour total 38 microsecondes.
La temporisation de la boucle sera donc de 64 - 38 = 26 microsecondes. C'est le fait d'ajouter ou de soustraire 1 microseconde à cette temporisation qui fera que l'on obtiendra soit un raster en diagonale allant de la droite vers la gauche soit l'inverse. Je n'ai pas à vous répéter qu'il nous faudra mettre autant de fois l'instruction NOP qu'il y a de microsecondes nécessaires dans la boucle.

PASSONS A AUTRE CHOSE
---------------------

Comment réaliser un scrolling en caractères géants sans avoir à afficher d'octets et sans faire de rupture ? Eh bien, c'est très simple Ill suffit de faire un scrolling en split raster. Je sais, c'est facile à dire mais à faire c'est une autre histoire. Je vais essayer de vous expliquer à peu près le principe de ce type de scrolling (j'entends déjà les remarques désobligeantes sortant de la bouche des autres membres du groupe... bouh, les vilains !). Tout d'abord, il faut considérer chaque lettre à faire défiler comme un tableau constitué de lignes et de colonnes ; dans ce tableau, chaque élément est un pixel qui sera soit allumé, soit éteint. Si l'on considère qu'un split raster est lui-même constitué de lignes et de colonnes, on peut facilement dessiner les pixels d'une lettre en un tableau de couleur géré par le split raster (un pixel en largeur correspond à un changement de couleur sur la ligne et en hauteur correspond à un groupe de plusieurs lignes). Pour comprendre la représentation, j'ai fait un petit schéma (il s'agit du schéma 2) qui, je l'espère, est plus explicite que ma prose : parfois un simple dessin aide beaucoup plus à la compréhension que de longues phrases, surtout si le sujet concerne le graphisme et les animations de couleurs. Les deux schémas que j'ai ajoutés à l'article sont seulement une représentation symbolique en fonction du temps, dans le deuxième schéma les instructions OUT 1 et OUT 0 n'existent pas en assembleur mais cela représente en fait le passage d'un pixel du split raster soit en couleur allumée soit en couleur éteinte (la couleur éteinte est la même que celle du fond, c'est logique) ; dans le schéma la disposition des OUT 0 et des OUT 1 est totalement arbitraire.
Maintenant que nous sommes censés savoir représenter une lettre grâce à un split raster, il nous faut la faire bouger.

ATTENTION SVP
-------------

Le tout est de connaître ce qui conditionne la position du split raster par rapport au bord de l'écran ; en fait, cela va dépendre de l'instant ou le split raster va être affiché en fontion de la position du canon à électrons. Plus clairement, on peut dire que cela dépend de la durée entre le HALT de synchronisation du split raster et le début du déroulement de la boucle d'instructions de changement de couleur. Donc, pour déplacer le split raster, il faut installer une temporisation variable avant d'exécuter la boucle composée des OUTs de changement de couleurs; cette temporisation variable va permettre selon sa durée (cette durée est exprimée en microseconde et est réalisée avec des NOPs) de positionner le split raster par rapport au bord du moniteur. Le petit exemple qui sert d'illustration pour cette partie de l'article est une routine qui réalise un quadrillage avec des split rasters, et ce quadrillage donne l'impression de se déplacer en utilisant le principe de temporisation variable cité plus haut. Vous constaterez par vous-même que l'inconvénient des split rasters est qu'ils peuvent au minimum se déplacer à l'écran que de deux octets par deux octets (un changement de temporisation d'un NOP dans le programme donne un déplacement de deux octets à l'écran), ce qui explique pourquoi les scrolls en split raster sont si rapides et si fatigants pour les yeux. En conclusion, les scrolls en split raster c'est bien beau mais c'est à utiliser avec beaucoup de parcimonie et surtout à ne pas mettre en cas de scrolling racontant une longue et fatigante histoire, sinon personne ne regardera votre démo jusqu'a la fin, de peur d'avoir une conjonctivite et une tête comme une citrouille. Bon, je vais clore ici cet article et en promettant de ne plus vous embêter avec les rasters (au moins pas pendant un mois...).

DiGiT

```
; Exemple de split raster par le LOGON SYSTEN pour Amstrad 100 %
; Pour DAMS ne pas oublier de remplacer les DEFS n per DEFS n,0

colonne         equ 10
ligne           equ 100
tab_color1      equ #8000
tab_color2      equ colonne*ligne+tab_color1

                org #9000
                di

; Installe les nouvelles interruptions

                ld hl,#C9FB
                ld (#0038),hl

; Initialise le controleur video

                ld hl,data_crt+13
                ld bc,#BC0D

init_crt        ld a,(hl)
                dec hl
                out (c),c
                inc b
                out (c),a
                dec b
                dec c
                jp p,init_crt

; Initialise la table de de couleurs

fin_crt         ld hl,tab_color1
                ld de,colonne*ligne
init_color      ld (hl),#54           ; Remplissons de noir
                inc hl
                dec de
                ld a,d
                or e                  ;Est-ce que D et E valent zero ?
                jp nz,init_color

; Installe le quadrillage des couleurs

                ld hl,tab_color1
                ld a,#4C              ;Couleur rouge hardware
                ld b,ligne/10
ligne_suiv      push bc
                push hl
                ld b,colonne/2

colonne_suiv    push bc
                push hl
                ld b,ligne/10
                ld de,colonne
pose_rouge      ld (hl),a
                add hl,de
                djnz pose_rouge

                pop hl
                inc hl
                inc hl
                pop bc
                djnz colonne_suiv

                pop hl
                ld de,ligne/10*colonne+1
                add hl,de
                pop bc
                djnz ligne_suiv

; Copie de la deuxieme table decalee

                ld hl,tab_color1
                ld de,tab_color1
                ld bc,colonne*ligne
                ldir
                ei

; Attente de la synchronisation verticale

vbl             ld b,#F5
synchro         in a,(c)
                rra
                jp po,synchro

; Selectionne l'encre zero et la net en noir

                ld bc,#7F00
                ld a,#54
                out (c),c
                out (c),a

; Halt du raster en diagonale

                HALT
                di
                defs 14

                ld bc,#7F54     ; B=Adresse du port, C=Couleur noir.
                ld de,#434A     ; D=couleur jaune pastel, E=Couleur jaune
bril
                ld hl,#4E4C     ; H=couleur orange, L=Couleur rouge.
                exx
                ld b,50
                ld c,b
raster1         exx
                out (c),d
                out (c),e
                out (c),h
                out (c),l
                out (c),h
                out (c),e
                out (c),d
                out (c),c       ; Et on repasse en noir
                exx
                defs 64-38-1    ; Tempo moins grande
                dec c
                jp nz, raster1
raster2         exx             ; On passe directement dans la seconde
                out (c),d       ; boucle sans bavure
                out (c),e
                out (c),h
                out (c),l
                out (c),h
                out (c),e
                out (c),d
                out (c),c       ; La mue chose que plus haut
                exx
                defs 64-38-1    ; Tempo plus grande
                dec b
                jp nz,raster2
                ei
                halt
                defs 64-6       ; Debuter au bon endroit
                di
                ld hl,saut
ecart           ld de,0         ; Valeur de la temporisation
                add hl,de
                jp (hl)
saut            ds 6            ; Temporisation variable
ptr_tab         ld hl,tab_color1
                ld b,#7F
                ld a,ligne
raster3         outi
                inc b
                outi
                inc b
                outi
                inc b
                outi
                inc b
                outi
                inc b
                outi
                inc b
                outi
                inc b
                outi
                inc b
                outi
                inc b
                outi
                inc b
                dec a
                jp nz,raster3
                ld a,#54
                out (c),a       ; Remat le papier en noir
                ei

; Gestion de la temporisation variable

                ld a,(ecart+1)
                add a,l
                ld (ecart+1),a
                cp 6
                jp nz,pas_max
bascule         ld a,0
                xor l
                ld (bascule+1),a
                add a,a
                ld e,a
                ld d,0
                ld hl,list_tab
                add hl,de         ; Alterne entre les deux tables de couleurs
                ld e,(hl)
                inc hl
                ld d,(hl)
                ld (ptr_tab+1),de
                xor a
pas_max         ld (ecart+1),a
                jp vbl

; Lists des tables des couleurs

list_tab        defw tab_color1
                defw tab_color1+1

; Table des valeurs d'initialisation du controleur video
data_crt        defb 63,50,50,8,38,0,64,35,0,07,0,0,48,0
```

```
1 MODE 2:MEMORY &3FFF:LOAD "diag.bin":CALL &9000
```

* Diagrams not transcribed yet