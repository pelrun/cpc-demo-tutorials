SPRITES !
=========

Salut bande de codeurs en délire ! Voici comment afficher un sprite selon diverses méthodes, en allant de la plus élémentaire et lente à la plus évoluée et rapide. Vous découvrirez aussi la diversité des modes d'affichage qu'offre le Z80.

Le listing se compose donc d'un programme principal et de différentes routines d'affichage de sprites que nous allons expliquer une par une. Qu'entendons-nous par « mode d'affichage »? Eh bien ! c'est la façon dont va se combiner le motif du sprite avec le fond ou le décor affiché à l'écran : le sprite peut se superposer au fond, passer derrière, se mélanger avec, l'effacer, etc.

LE SPRITE EN MODE 1
-------------------

Mais avant de voir les possibilités d'affichage, nous allons d'abord voir comment déterminer l'adresse-écran où il faut afficher le sprite en fonction des coordonnées x et y données. Nous allons étudier le cas d'un sprite en mode 1, car c'est un cas assez global. Un écran mode 1 normal fait 320 pixels de large, soit 80 octets. Un octet contient donc 4 pixels, et leur organisation binaire est si complexe qu'il est très compliqué d'afficher un sprite au pixel près : cela demanderait énormément de temps-machine pour faire tous les décalages, rotations et masquages nécessaires en temps réel. L'astuce consiste, comme d'habitude, à précalculer les positions des sprites au pixel près. Le même sprite est reproduit 4 fois en mémoire, à chaque fois décalé d'un pixel. Inconvénient un sprite ainsi codé en mémoire prend 4 fois plus de place (2 fois plus de place s'il était en mode 0 puisqu'il n'y a que 2 pixels par octet et 8 fois plus de place en mode 2 18 pixels par octet]). Ainsi, le petit listing en Basic met en mémoire un petit sprite de 4 octets sur 13 lignes, décalé 4 fois, ainsi que son masque également décalé (nous en verrons l'utilité plus tard). En résumé, il faut donc décomposer l'abscisse x en nombre d'octets et en nombre de décalages de pixel par rapport au nombre de pixels contenus dans un octet (selon le mode).
Voilà pour l'abscisse, maintenant, occupons-nous de l'ordonnée (y). Comme vous le savez, l'organisation de la mémoire écran n'est pas linéaire c'est-à-dire que la 2" ligne visible à l'écran ne se trouve pas en mémoire juste après la première. Bien qu'il soit relativement facile de descendre d'une ligne à une autre, il est plus difficile de se positionner directement à la ligne voulue par un algorithme, d'où la nécessité encore une fois de précalculer une table des ordonnées contenant l'adresse écran de chaque ligne, les unes en dessous des autres, permettant d'obtenir automatiquement et directement l'adresse de la ligne voulue ! On récupère l'adresse de la ligne voulue en prenant l'ordonnée et en la multipliant par 2 (puisqu'une adresse tient sur 2 octets) et on additionne à cette valeur l'adresse où est stockée la table ; on pointe ainsi sur l'adresse voulue qu'on n'a plus qu'à récupérer. Cette opération peut se faire de plusieurs façons ; dans le listing 1, c'est le registre HL qui pointe sur la table des adresses des lignes écran tandis que dans le listing 2, les adresses sont récupérées grâce à la pile (ce qui peut s'avérer plus rapide, à condition que la pile ne soit pas utilisée en même temps). Voilà donc obtenues l'adresse source (l'adresse du sprite décalé correspondant à l'abscisse) et l'adresse destination (l'adresse écran ou va être affiché le sprite) On peut donc maintenant passer l'affichage proprement dit.

L'AFFICHAGE
-----------

Le listing 1 regroupe plusieurs façons conventionnelles d'afficher un sprite, nous allons donc les commenter une par une, des plus simples aux moins évidentes avec leurs points forts et leurs points faibles :
- n°1 : affichage par LDIR. Vraiment la méthode la plus élémentaire pour afficher un sprite ! Avantage : le code ne prend pas beaucoup de place en mémoire. Inconvénient : un LDIR demande l'équivalent de 5 NOPS en temps-machine, ce qui est quand même pas mal, d'autant plus que le sprite est affiché à l'écran en effaçant le fond sur tout le rectangle qui définit sa surface ;
- n°2 : affichage par LDI. Le principe est à peu près le même que celui de la rounotine n"1, à ceci prés qu'on remplace le LDIR par autant de LDI que le LDIR doit êtrnoe effectué (autrement dit, on met autant de LDI qu'il y a d'octets qui composent le nosprite en largeur). Avantage : sensiblement plus rapide que LDIR (un LDI prend 4 NOPnoS en temps-machine), surtout si le sprite est large. Inconvénient : la routine d'affichage prend d'autant plus de place que le sprite est large, et le fond à l'endroit où le sprite s'affiche disparaît là encore;
- n°3 : affichage avec test d'octet Ce système permet de donner au sprite un semblant de masquage. On teste à l'aide d'un OR si l'octet du sprite est nul. S'il n'est pas nul, on l'affiche, sinon, on passe au suivant Avantage: le fond derrière le sprite est ainsi partiellement sauvé, et la routine occupe peu de mémoire. Inconvénient: la routine est lente et le résultat à l'écran n'est pas toujours parfait car le masquage se fait a l'octet et non au pixel (remarque: le résultat reste fort convenable en mode 0).
- n°4 : affichage en « mélangeant » sprite et fond. Ici, on fait directement un OR entre le sprite et le fond, et on affiche le résultat Ce genre de routine est avantageuse lors d'un écran en « dual-playfield » : grâce à un jeu d'encre, on peut afficher des sprites masqués au pixel par un simple OR, mais ces sprites sont moins colorés que des sprites normaux. Dans le cas d'un dual playfield (cf les sprites de la démo du dernier numéro d'Amstrad Cent Pour Cent, il suffit d'utiliser exactement la même routine avec un XOR à la place du OR pour réafficher le fond. S'il n'y a pas de dual-playfield, le résultat à l'écran peut parfois être bizarre (superpositions de couleurs...). mais cette routine a l'avantage d'être suffisamment rapide et de ne pas prendre trop de mémoire.
- n°5 : affichage par masquage. La routine qui offre le meilleur résultat visuel, mais aussi la plus compliquée ! Elle nécessite, en plus des données du sprite, un masque qui détermine quels pixels du fond doivent disparaître (cela permet, par exemple, de faire un petit contour noir autour des sprites, comme dans le jeu KnightTime). On fait un AND entre le masque et le fond, puis on fait un OR entre le résultat et la donnée de l'octet du sprite, et enfin, on affiche. Avantage : finesse de l'affichage. Inconvénients : le masque prend autant de place que le sprite et l'affi- entre le résultat et la donnée de l'octet du sprite, et enfin, on affiche. Avantage : finesse de l'affichage. Inconvénients : le masque prend autant de place que te sprite et l'affichage est lent.
- n°6 : affichage à la pile. un affichage qui trouve son utilité, surtout pour déplacer de gros sprites sur un fond uni. En effet cette technique d'afficha- ge prend les octets 2 par 2 (d'où la nécessité d'avoir un sprite composé d'un nombre pair d'octets en largeur), ce qui permet d'être plus rapide, mais le fond derrière le sprite est détruit. Voilà pour ce qui est de l'affichage des sprites. Il est possible, par exemple dans le cadre d'une anima- tion, que vous souhaitiez que le fond soit restitué. Dans ce cas, il suffit de le sauver à l'aide de LDIRs ou de LDIs dans un buffer de la taille du sprite, puis que vous affichiez le sprite, et ensuite que vous réaffichiez le fond précédemment sauvegardé grâce à des LDIs, LDIRs, PUSH-POP, etc.

BAS LES MASQUES
---------------

On a donc vu que les routines les plus intéressantes, notamment celles par masquage, sont souvent les plus lentes. Pour avoir un masquage rapide, nous allons faire un généra- teur de codes, qui produit une rou- tine telle que le sprite est incrusté dans cette routine. C'est ce que fait la routine « gener » du listing 2. Cette routine crée donc 4 routines de sprites différentes (une pour chaque décalage de pixel}. Les adresses de ces 4 routines sont contenues dans une table qui sert lors du traitement de l'abscisse, à savoir sur quelle routi- ne d'affichage il faudra sauter plus tard. Comme les masquages entre le fond et le sprite sont immédiats, l'affi- chage est extrêmement plus rapide de cette façon. En revanche, la mémoire demandée est assez consi- dérable puisqu'il n'y a pas de boucle et puisqu'il faut environ 8 octets de code autogénéré pour afficher un octet du sprite. Néanmoins, on peut se permettre cette perte d'espace mémoire pour des petits sprites, et à plus forte raison si on fait une démo.

Le listing 2 vous explique le code autogénéré pour l'affichage par mas- quage, mais de toutes façons, il est également possible de faire du code autogénéré avec les autres types d'affichage. Par exemple, pour l'affi- chage avec test d'octet, il suffit de vérifier si l'octet est vide ou non lors de la génération du code : le code résultant se trouve grandement opti- misé !

Voilà, nous avons vu l'essentiel concernant l'affichage des sprites, il ne me reste plus qu'à vous souhaiter de bien assimiler cette rubrique et à espérer que j'en aurai bientôt fini avec cette ?#°1$@ de grippe !

A bientôt ! Pict

```
; Exemple de routines d'affichage da spritas
; (e) Pict/Logon System 1593 pour A100$
; Assemble avec DANS charge en #4000
;
; Coordonnees du sprite
x             EQU 40
y             EQU 2

; Dimensions du sprite
haut          EQU 13
larg          EQU 4
size          EQU larg*haut
mask          EQU 4*size

; Adresse de la pile
; du programme
stack         ORG #2000

; Adresse des donnees
; du sprite
sprite        EQU #1000
tabspr        DEFW size*0+sprite
              DEFW size*2+sprite
              DEFW size*1+sprite
              DEFW size*3+sprite

; Largeur de l'ecran
; en word.
r1            EQU 40

; Routine de chargement du sprite
;
nom1          DEFM "sprite  .BIN"
load          LD b,#0c
              PUSH DE
              CALL #BC77
              POP HL
              CALL #BC83
              JP #BC7A
; Debut du programme
run_          ENT $
              LD hl,nom1
              LD de,sprite
              CALL load

; Passags en mode 1 et affichage
; d'un fond quelconque
go            LD a,1
              CALL #bc0e
              LD hl,fond
affond        LD a,(hl)
              INC hl
              CP a
              JR z,finaff
              PUSH hl
              CALL #bb5a
              POP hl
              JP affond
finaff

; Bloquage du FirmWare pour
; avoir le jouissance de tous
; les registres du Z-80 qu'on
; sauvegarde.
              DI
              LD hl,(#38)
              LD (sys1+1),hl
              LD hl,#c9fb
              LD (#38),hl
              LD (stasys1+1),sp
              LD sp, stack
              EXX
              EX at,af
              PUSH hl
              PUSH de
              PUSH bc
              PUSH af

; Initialisation des
; couleurs du sprite
              LD bc,#7f00
              LD de,#5458
              LD hl,#4d4b
              OUT (c),c
              OUT (c),d
              INC c
              OUT (c),c
              OUT (c),e
              INC c
              OUT (c},c
              our (c),h
              INC c
              OUT (c),c
              OUT (c),l

; Creation de la table
; des ordonnees
; (hauteur normale=200 lignes)
; (adresse normale/c000

              LD hl,#c000
              LD de,r1*2+#c000
              LD bc,#800
              LD ix,taby
              LD a,200
creey         LD (ix+1),h
              LD (ix+0),l
              INC ix
              INC ix
              ADD hl,bc
              JP nc,nocarry
              ADD hl,de
nocarry       DEC a
              JP  nz,creey

; appel du listing d'Affichage
; (Listingi,Listing2)
              CALL 1isting2

; Attente d'appui sur Espace
key           LD bc,#f782
              OUT (c),c
              LD bc,#f40e
              OUT (c),c
              LD bc,#f6c0
              OUT (c),c
              XOR a
              OUT (c),a
              LD bc,#f792
              OUT (c),c
              LD de,#f4f6
              Lo e,#45
              LD b,e
              OUT (c),c
              LD b,d
              IN d,(c)
              LD bc,#f782
              OUT (c),c
              DEC c
              OUT (c),a
              RL d
              JP c,key

; sets le pen 1 blanc.
              LD bc, #7f01
              LD a,#4b
              OUT (c),c
              OUT (c),a

; Retablissement du Firmware
firm
              DI
              POP af
              POP bc
              POP de
              POP hl
              EXX
              EX af,af
stasys1       LD  sp,0
sys1          LD hl,0
              LD  (#38),hl
              EI
              RET
fond          DEFM "LOGON SYSTEM"
              DEFS 10,13
              DEFM "AMSTRAD 100%"
              DEFB 0
taby          DEFS 2*200,0

; Programmes principal
; On transforme les
; coordannees x et y
; en adresses ecran.

listing1
; On s'occupe d'abord de
; l'abscissse
              LD bc,x
              XOR a
              LD d,a
              SRL b
              RR c
              RLA
              SRL b
              RR c
              RLA
              ADD a,a

; les 2 derniers bits de x
; deteminent le sprite a
; afficher parmi les sprites
; decales au pixel dans la
; memoire
              LD e,a
              LD hl,tabspr
              ADD hl,de
              LD e,(hl)
              INC hl
              LD d,(hl)
              LD a,c

; L'ordonnees permet de pointer
; dans uns table sur l'adresse
; de la ligne ecran correspondan
              LD  hl,y
              ADD hl,hl
              LD bc,taby
              ADD hl,bc
              LD c,(hl)
              INC hl
              LD  b,(hl)

; on additionne a cette adresse
; l'abscisse en octets
              LD h,0
              LD 1,a
              ADD hl,bc
; on a l'adresse finale dans HL
              LD a,haut
; ...Qu'on transfers dans DE
              EX  de,hl

; ROUTINES D'AFFICHAGE OÙ SPRITE

; (jp loop1_1,loop1_2,...,100p1_6)
routine       JP  loop1_5

; 2EME ROUTINE
; AFFICHAGE LE PLUS SIMPLE
; LDIR SEULEMENT
loop1_1       LD bc,larg
              LDIR
              LD bc,#800-larg
              EX de,hl
              ADD hl,bc
              JP nc,nocarry1
              LD bc,r1*2+#c000
              ADD hl,bc
nocarry1      EX dc,hl
              DEC a
              JP nz,loop1_1
              RET

; 2EME ROUTINE:A PEU PRES INDENTIQUE
; A LA PRÉMIERE MAIS ON REMPLACE LDIR
; FAR AUTANT DE LDI QUE LE SPRITE EST
; LARGE EN OCTETS.

loop1_2       LDI
              LDI
              LDI
              LDI
              LD bc, #800-larg
              EX de,hl
              ADD hl,bc
              JP ne,nocar1_2
              LD bc,r1*2+#c000
              ADD hl,bc
nocar1_2      EX de,hl
              DEC a
              JP nz, loop1_2
              RET
;
; 3EME ROUTINE:ON TESTÉ CHAQUE OCTET
; DU SPRITE A AFFICHER:S'IL EST NUL,
; ON NE L'AFPICHE PAS

loop1_3       EX de,hl
loop1_31      PUSH af
              LD b,larg
              loop1_32 LD a,(de)
              INC de
              OR a
              JP z,noaff1_3
              LD (hl),a
noaff1_3      INC hl
              DJNZ 1oop1_32
              LD bc,#800-larg
              ADD hl,bc
              JP nc,nocar1_3
              LD1 bc, #c050
              ADD hl,bc
              nocar1_3 POP af
              DEC a
              JP nz,loop1_31
              RET

; 4EME ROUTINE:LE SPRITE ET LE
; FOND SONT "MELANGES" AVEC UN
; OR;LE RESULTAT N'EST PAS
; TOUJOURS GRACIEUX, MAIS LA
; RAPIDITE D'AFFICHAGE EST
; CONVENABLE...
; ON AURAIT PU UTILISER XOR A
; LA PLACÉ DE XOR CE QUI
; D'ÉMPLOYER LA MEME ROUTINE
; POUR L'AFFICHAGE ET L'EFFACAGE
; MAIS LE RESULTAT À L'ECRAN EST
; SOUVENT TRES LAID...
loop1_4       EX de,hl
loop1_41      PUSH af
              LD b,larg
loop1_42      LD a,(de)
              OR (hl)
              LD (hl),a
              INC hl
              INC de
              DJNZ loop1_42
              LD bc,#800-larg
              ADD hl,bc
              JP nc,nocar1_4
              LD bc,r1*2+#C000
              ADD hl,bc
nocar1_4      POP af
              DEC a
              JP nz,loop1_41
              RET

; SENE ROUTINE:AFFICHAGE MASQUE:
; LE PLUS ÉFYICACE,MATS AUSSI LE
; 4 PLUS LENTI...A MOINS D'UTILISER
; DU CODE AUTOGENERE! (VOIR LISTING 2)
loop1_5
; on recupere la masque
; correspondant su sprite ducale
              LD bc,mask
              PUSH hl
              ADD hl,bc
              LD b,h
              LD c,l
              POP hl
              EX de,hl
              EXX
              DI
              LD (sp1_5+1),sp
              LD sp,#800-larg
loop1_51      EX af,af
              LD b,4
loop1_52      EXX

; DE=sprits
; BCASARÇUE
; HL=adresse ecran
              LD a,(bc)
              AND (hl)
              EX de,hl
              OR (hl)
              EX de,hl
              LD (hl),a
              INC hl
              INC de
              INC bc
              EXX
              DJNZ loop1_52
              EXX
              ADD hl,sp
              JP nc,nocar1_5
              LD sp,r1*2+#c0000
              ADD hl,sp
              LD sp,#800-larg
nocar1_5
              EXX
              EX af,af
              DEC a
              JP nz,loop1_51
sp1_5         LD sp,0
              EI
              RET

; 6EME ROUTINE:
; AFFICHAGE À LA PILE:RAPIDE MAIS
; LE FOND RST DETRUIT ET LE SPRITE
; DOIT ETR£ COMPOSE D'UN NOMBRE
; PAIR D'OCTETS EN LARGEUR
loop1_6       DI
              LD (sp1_6+1),sp
              LD sp,hl
              EX de,hl
              LD b,a
loop1_61      LD c,larg/2
loop1_62
              POP de
              LD (hl),e
              INC hl
              LD  (hl),d
              INC hl
              DEC c
              JP nz,loop1_62
              LD de,#800-larg
              ADD hl,de
              JP  nc,nocar1_6
              LD  de,#c050
              ADD hl,de

nocar1_6      DJNZ loop1_61
sp1_6         LD sp,0
              EI
              RET

; Listing2:Sprite Autogenere:
; ies donness du sprite sont
; incrutees dans la routine.
listing2
              CALL gener
              CALL affiche
              RET

; patine d'affichage en
; code gerere:cette partie
; dsternine laqualis des 4
; routines doit etre
; appales.
; on se sertt da ja pila
; Peur recuperer l'adresse
; da la ligne de l'ecran
; at celle ds la routine
; d'africhage Sont on a
; besoin
;
affiche
              DI
              LD (sp2+1),sp

; on recupere d'abord
; l'adresse de la
; routine d'affichage
; tout en transformant
; l'abscisse.

              LD de,x
              LD a,e
              SRL d
              RR e
              SRL d
              RR e
              LD sp,tabadsp
              AND 3
              ADD a,a
              LD h,d
              LD 1,a
              ADD hl,sp
              LO sp, hl
              POP ix

; on recupere maintenant
; l'adresse de la ligne
; de l'ecran.
              LD sp,taby
              LD hl,y
              ADD hl,hl
              ADD hl,sp
              LD  sp,hl
              POP hl
              ADD hl,de
              LD bc,#800-larg+1
              LD de,r1*2+#c000
sp2           LD  sp,0
              EI

; saut à la routine d'affichage
; selon l'abscisse
              JP (ix)

; Programmes de generation
; des 4 routines de sprites.

gener         LD bc, sprout
              LD hl, sprite
              LD de,sprite+mask
              LD ix,tabadsp
              LD a,4
codebyte
              PUSH af
              PUSH hl
              PUSH iy
              POP hl
              LD (ix+0),1
              LD (ix+1),h
              POP hl
              INC ix
              INC ix
              PUSH ix
              CALL calc
              POP ix

              POP af
              DEC a
              JP nz,codebyte
              RET

calc          LD b,haut
pokeline      PUSH bc
              LD b,larg
poke          LD a,(de)

; on teste d'abord si
; le masque est plein
              CP #ff

; si oui,pas la peine
; d'afficher l'octet,
; on passe au suivant,
              JP z,nxtbyte

; on teste ensuite si
; le masque est nul:
; dans ce Cas,pas besoin
; de masquer,on affiche
; la donnee da l'octet du
; aprite directement

              OR A
              JP nz,maskbyte
              LD a,(hl)
              LD  (iy+0),#36
              INC iy
              LD (iy+0),a
              INC iy
              nxtbyte LD (iy+0),#23
              INC y
              JP coded
maskbyte
              PUSH bc

; On recopie le “bout"
; de con qui affiche ot
; masque un octet du sprite
pkr           LD a,(ix+0)
              LD {iy+0),a
              INC ix
              INC iy
              DJNZ pkr
              POP bc

; on incrusta dans le code
; le masque et la donnes
; da l'octet du sprite
              LD a,(de)
              LD (iy-5),a
              LD a,(hl)
              LD (iy-3),a
coded
              INC de
              INC hl
              DJNZ poke

; en passe à la ligne
; le “INC L" inutile
; en decrensntant IY.
              DEC iy
              LD b,4

; on recopie le “bout”
; de code qui permet de
; descendre d'une ligne.
              LD ix,nxtline
pknx          LD a,(ix+0)
              LD (iy+0),a
              INC ix
              INC iy
              DJNZ pknx
              POP bc
              DJNZ pokeline

; fin de la routine
; d'affichage.
; on n'a pas besoin
; de redescendre d'une
; ligne donc on elinine
; le morceau de code
; precedent en reculant le
; pointeur du code genere
; pour reecrire par dessus.

              LD BC,-4
              ADD IY,BC
              LD )iy+0),#c9
              INC iy
              RET

; “morceau" de programme
; necessaire pour masquer
; et afficher un octet.
;
oper ; bytes
              LD a,(hl)
              AND 0
              OR 0
              LD (hl),a
              INC l

; “morceau" de programme
; necessaire pour descendre
; d'une ligne.
nxtline ; bytes
              ADD hl,bc
              JR nc,noaddde
              LD hl,de
noaddde
end

; table cree par le
; generateur qui
; contiant les adresses
; des 4 routines generees.
tabadsp       DEFS 4*2,0

; les routines sont generees
; dans l'espace de memoire
; debutant ici-meme:
sprout
```

```
10 MEMORY &3FFF
20 FOR a=&4000 TO &4000+4*8*13
30 READ a$:POKE a, VAL("&"+a$)
40 NEXT
50 SAVE"sprite.bin",b,&4000,4*8*13
60 DATA 00,70,00,00,10,80,C0,00,20,00,24,00,40,00,5E,00,40,00,14,00,80,00,00,80,C4,00,00,80,84,00,00,80,42,00,10,00,43,00,12,00,21,6F,2C,00,10,87,C0,00,00,70,00,00,00
70 DATA 30,80,00,00,C0,60,00,10,00,12,00,20,00,27,80,20,00,02,80,40,00,00,40,62,00,00,40,42,00,00,40,21,00,00,80,21,08,01,80,10,3F,1e,00,00,C3,68,00,00,30,80,00,00,E0
80 DATA C0,00,00,60,30,00,00,80,01,80,10,00,13,48,10,00,01,40,20,00,00,20,31,00,00,20,21,00,00,20,10,08,00,40,10,0C,00,48,00,97,8F,80,00,61,3C,00,00,10,C0,00,00,00,E0
90 DATA 00,00,30,10,80,00,40,00,48,00,80,01,AC,00,80,00,28,10,00,00,10,10,88,00,10,10,08,00,10,00,84,00,20,00,86,00,24,00,43,CF,48,00,30,1E,80,00,00,E0,00,FF,88,FF,FF
100 DATA EE,00,33,FF,CC,00,11,FF,88,00,00,FF,88,00,00,FF,00,00,00,77,00,00,00,77,00,00,00,77,88,00,00,FF,88,00,00,FF,CC,00,11,FF,ÈE,00,33,FF,FF,88,FF,FF,FF,CC,77,FF,FF
110 DATA 00,11,FF,EE,00,00,FF,CC,DD,00,77,CC,00,00,77,88,00,00,33,88,00,00,33,88,00,00,33,CC,00,00,77,CC,00,00,77,EE,00,00,FF,FF,00,11,FF,FF,CC,77,FF,FF,EE,33,FF,FF,88
120 DATA 00,FF,FF,00,00,77,EE,00,00,33,EE,00,00,33,CC,00,00,11,CC,00,00,11,CC,00,00,11,EE,00,00,33,EE,00,00,33,FF,00,00,77,FF,88,00,FF,FF,EE,33,FF,FF,FF,11,FF,FF,CC,00
130 DATA 77,FF,88,00,33,FF,00,00,11,FF,00,00,11,EE,00,00,00,EE,00,00,00,EE,00,00,00,FF,00,00,11,FF,00,00,11,FF,88,00,33,FF,CC,00,77,FF,FF,11,FF,00,00,00,00,00,00,00,00
```
