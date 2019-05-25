MAXIM BARROU
============

Eh oui, chers amis lecteurs, me voici de retour après de longs mois d'absence ! Cette fois-ci, voici le source d'une des parties qui firent le succès de The Demo, la 3DSCROLL démo ! Pour ceux qui n'auraient pas vu cette démo, imaginez un scroll qui s'éloigne en profondeur, un peu à la façon du générique de La Guerre des étoiles. Sympa non ?

Mais comment cela est-il possible ? Avant les explications, tapez d'abord le listing Basic et le source assembleur (ou les datas pour les moins pros) et exécutez-les. Le listing Basic se charge de créer une table de profondeur et le patron de la forme du scroll.
Pour comprendre le principe, nous avons besoin de faire un peu de mathématiques... Non, ne vous enfuyez pas en courant, vous allez voir, c'est très simple (je suis d'ailleurs moimême nul en maths, alors pas de quoi vous affoler...) !
Tout d'abord, il faut savoir que l'oeil humain voit les objets en perspective conique. Cela revient à dire que la profondeur de l'objet dépend d'une fonction conique, et plus précisement d'une fonction hyperbolique de type 1/x. En effet, dans une courbe de ce genre, plus x est grand, plus son image est petite. Il en est de même pour la vision. Plus un objet est éloigné de l'oeil, plus il paraît petit.
Afin de rendre cet effet, le programme Basic crée une table de profondeur composée de 200 valeurs, ce qui est le nombre de lignes de l'écran standard. En mémoire, nous avons un scrolling de 256 lignes. Les 200 valeurs créées par le listing Basic correspondent à certaines de ces 256 lignes, de façon à ce qu'on ait sur l'écran l'effet de profondeur souhaité.
Ensuite, la deuxième partie du scroll sert à créer l'écran pour donner un effet de profondeur à la largeur du scroll. Et là, je dois vous expliquer le principe du scroll qui est le même que celui employé dans le jeu Trail Blazer, si vous connaissez.
On fait passer différentes couleurs dans des encres diverses, grâce à des rasters. Cela dessinera des formes sur l'écran. En l'occurence des lettres, pour ce qui nous concerne. Comme notre scrolling est en mode 1, nous nous servons des 3 autres couleurs (1, 2 et 3) pour l'affichage. Pour cela, nous avons besoin d'une police de caractères spéciale. Etant donné que nous n'avons que 3 encres, il nous faut une fonte qui fasse 3 pixels de large. Le premier pixel est représenté par la première encre, et ainsi de suite. Selon que chaque pixel de la fonte est éteint ou allumé, on fait un raster en couleurs ou en noir dans chacune des 3 encres. On a ainsi la forme de la lettre qui s'affiche à l'écran. Remarquez qu'il était parfaitement possible de faire le scroll en mode 0 et d'avoir ainsi une fonte composée de plus de pixels en largeur (pas trop, car il faut que la durée nécessaire pour changer les couleurs de chaque encre soit inférieure au temps machine d'une ligne raster). Le mode 0 permet aussi d'ajouter des motifs par-dessus le scroll, à condition qu'ils soient d'une autre couleur que les 3 premières. Néanmoins, j'ai choisi le mode 1 afin de privilégier la finesse du tracé. D'ailleurs, rien ne vous empêche de dessiner le fond avec un utilitaire de dessin (vous pourrez obtenir des résultats très marrants tels un scroll en forme de boule, un scroll tordu, etc.).

BON COURAGE
-----------

Les plus habiles pourront essayer de modifier la table de profondeur. Je leur souhaite bon courage, car ce n'est pas si évident que cela. Voilà pour ce qui concerne le principe.
Maintenant, attaquons-nous à la programmation en assembleur proprement dite. Donc au source ci-joint !
Tout d'abord, le CALL &BCOE du début permet de passer en mode 1 et de mettre les pointeurs d'offset des registres 12 et 13 du CRTC à des valeurs qui conviennent au programme.
Puis, on charge les fichiers précédemment créés par le listing Basic, c'est-àdire l'écran et la table qui nous serviront pour le raster. Pour charger ces fichiers directement du programme binaire, nous utilisons tout simplement les bonnes vieilles routines système, car c'est dur de faire autrement.
Ensuite, on remercie le système de nous avoir chargé les fichiers voulus, puis on s'en sépare en le désactivant via un EI-RETI bien envoyé à une adresse que je n'ai plus besoin de citer. Enfin, on passe à l'initialisation de l'écran. On met un fond noir, un border bleu, on initialise le diviseur d'interruption tout en s'assurant d'être en mode 1 et l'on monte l'écran de 8 lignes grâce au registre 7 du CRTC, afin que le moment où le faisceau d'électrons du téléviseur envoie la première ligne d'écran corresponde au moment où se produit le premier HALT (ouf !). Cette manipulation permet de faire notre « scroll-raster » juste après une instruction HALT, car la première interruption interviendra en même temps que l'écran commencera à être affiché sur le moniteur.
Il ne reste plus qu'à débuter la boucle principale du programme.

ÇA ROULE RAOUL
--------------

Tout d'abord, on attend le retour du faisceau d'électrons en haut de l'écran, puisqu'on fait un raster. Aussitôt après, on s'occupe de la gestion du scrolling. Il y a en premier un compteur de lettres, qui permet d'attendre qu'une lettre soit complètement affichée avant de passer à la suivante. Remarquez que ce compteur est autobouciant, étant donné que chaque lettre fait 64 lignes de haut, soit &40 lignes de haut (en fait, en mémoire, chaque lettre fait 8 lignes de haut, mais on recopie 8 fois chaque ligne, d'où la raison d'être d'un second compteur autobouclant plus loin). Il suffit donc de faire un AND &3F sur la variable comptant les lignes pour réinitialiser la variable au bout de 64 fois. Chouette, non ? C'est pratique, net, et c'est plus rapide en temps machine, donc plus court en mémoire, qu'une gestion de pointeur classique.
Ensuite, on affiche la ligne courante de chaque lettre dans un buffer intermédiaire. D'ailleurs, à propos des lettres, je vous signale que vous ne disposez que des caractères de ponctuation, des majuscules et des chiffres, pour que vous n'ayez pas trop de datas à taper.
Voyons maintenant de quoi est composé le buffer intermédiaire dont je vous parlais. Afin de pouvoir afficher la lettre en raster, il faut transformer les pixels de chaque lettre en couleurs. Ainsi, si un pixel de la lettre est éteint, on met du noir à l'endroit correspondant à ce pixel dans le buffer intermédiaire. S'il est allumé, on prend une couleur dans une table pour chaque ligne et on la met dans le buffer au même endroit. La lettre est maintenant prête à être affichée comme raster. On peut donc la mettre dans le buffer final qui servira pour le scroll. Cependant, pour que notre message scrolle, il faut le décaler à chaque balayage en donnant ainsi la sensation de mouvement. Mais, étant donné que nous devons bouger 3 x 256 octets (puisqu'on scrolle 256 lignes de 3 octets [un pour chaque couleur]), des LDI(R)S prendraient trop de temps machine, c'est pourquoi on utilise la technique des décalages de pointeurs dont je vous avais déjà parlé lors du listing sur les ondulations d'écran. Mais cette fois-ci, il y a une nouveauté puisque nous devons réactualiser le buffer.
Cela est en fait très simple à réaliser, puisqu'il suffit de recopier le buffer intermédiaire à l'adresse donnée par un compteur général du scroll au début de la routine. Notez que ce compteur est lui aussi autobouclant, mais on n'a pas besoin de faire un AND, car le buffer est composé de 256 lignes (étant donné que c'est un compteur 8 bits, il revient de lui-même à zéro ! Mais non, je ne vous prends pas pour des imbéciles, mais c'est déjà suffisamment compliqué comme ça...).
Une remarque, n'oubliez pas que l'utilisation des décalages de pointeurs nécessite deux fois plus de mémoire et qu'il faut réactualiser les 2 parties identiques du buffer. C'est pourquoi il y a en tout 6 LDIs dans la gestion du programme (puisqu'il faut recopier 2 fois les 3 octets du buffer intermédiaire, soit 2 x 3 = 6). Une fois la gestion du buffer finie, DE contient l'adresse de départ des données à afficher dans le raster, et il ne reste plus qu'à attendre le HALT pour se synchroniser avec le début de l'écran virtuel et enfin lancer notre raster.

ET LE RASTER FUT
----------------

Pour afficher notre raster, nous allons nous servir d'OUTIs et de la pile. Or pour pouvoir utiliser cette dernière, nous devons envoyer un DI afin que le Z80 n'aille pas poker une adresse due à une interruption fortuite en plein milieu des données. Il faut aussi sauver l'adresse contenue dans SP, puisqu'elle va être modifiée. Le pointeur de pile va adresser la table de profondeur précédemment calculée en Basic. En faisant à chaque raster-line un POP HL et en l'additionnant avec le contenu de DE, on aura l'adresse de la ligne du scroll à afficher, ce qu'on fait avec des OUTIs, en changeant à chaque OUTI de couleur. On n'oubliera pas de réinitialiser le registre B à chaque rasterline, car il est modifié par OUTI. C'est ce qu'on fait avec LD BC,&7F01, ce qui permet de remettre aussi le registre C à l'adresse correspondant à la première couleur. Après les OUTIs, on attend la fin de la raster-line avec des DJNZ de tempo et un NOP (il faut avoir exactement le temps machine d'une rasterline, c'est à dire 64 NOPS). On fait ça 200 fois, puis on teste la touche CONTROL pour changer (enfin non, ce n'est pas pour changer, mais quand j'ai écrit le programme, je ne me rappelais plus le code de la barre d'espace... Hem !). Si elle est enfoncée, alors on quitte le programme et on revient au système (le <« Firmware » comme disent les pros). La suite du listing est composée des données et allocations mémoires (ça y est, je me crois sur Amiga !) nécessaires au programme (fontes, table de couleurs, etc.).

LOGON, LA CLASSE !
------------------

Voilà, j'espère que ce programme vous plaira, et que ceux qui se demandaient quel était le truc en voyant The Demo l'auront compris et sauront l'utiliser à bon escient. Croyez-moi, il permet de faire des trucs faramineux.
A propos, si vous avez fait un fond rigolo ou étonnant pour ce scroll, n'hésitez pas à l'envoyer à la rédac. Bon, je vous quitte, car Poum veut rentrer chez lui, et je n'oserai contredire sa décision.

Call 0! Pict !

```
10 REM ON CREE D'ABORD LA TABLE DE PROFONDEUR DES ORDONEES
20 MODE 2:ORIGIN 0,0:ADR=&4000
30 INITIAL=256:FINAL=200:C=2*INITIAL
40 REM "INITIAL" CORRESPOND AU NOMBRE DE LIGNES QUE L'ON A AU DEPART ET "FINAL" A CELUI QUE L'ON VEUT OBTENIR
50 FOR a=0 TO FINAL-1
60 B=C*(1-FINAL/(A+FINAL)):PLOT 2*B,2*A
70 LOCATE 1,1:PRINT A,INT(B)
80 VL=3*INT(B):POKE ADR+2*A+1,(VL AND &FF00)/256:POKE ADR+2*A,VL AND 255
90 NEXT
100 SAVE "PROFTAB",B,ADR,2*(VL+1)
110 REM MAINTENANT ON CREE L'ECRAN QUI SERVIRA POUR "SIMULER" L'EFFET DE PROFONDEUR SUR LA LARGEUR DU SCROLL.
120 MODE 1:ORIGIN 0,0
130 FOR A=0 TO 2
140 GRAPHICS PEN (A+1):FOR B=2*(107*A) TO 2*(107*(1+A))
150 PLOT B,0:DRAN 320,INITIAL*2
160 NEXT
170 NEXT
180 SAVE"ECRAN",B,&C000,&3FFF
```

```
;
; The 3D-Scroll !
; Logon System pour Amstrad 100%
; (c)Pict/Mai 1992
;
;-------- Adresse de depart du progreme.-------
            ORG #a00
;---Routine de chargement de fichiers.---
load
            LD B,#0C
            PUSH DE
            CALL #BC77
            POP HL
            CALL #BC83
            JP #BC7A
;----Nom des fichiers e charger.-----------
nom1        DEFM 'ECRAN   .BIN'
nom2        DEFM 'PROFTAB .BIN'
;------- Debut du programme.-----------------
go          ENT $
;------- on passe en mode 1------------------
            LD a,1
            CALL #BC0E
;----- on charge les fichiers.---------------
            LD hl,nom1
            LD de,ecran
            CALL load
            LD hl,nom2
            LD de,proftab
            CALL load
;-------On coupe le systeme-------------------
            DI
            LD hl,(#38)
            LD (sys+1),hl
            LD hl,#C9FB
            LD (#38),hl
;------On saute au programme principal.-----
            JP prog
firm
;------- Retourne eu firmware et met en blanc l'encre 1.
            LD bc,#7F01
            OUT (C),C
            LD c,#4B
            OUT (c),c
            DI
sys         LD hl,0
            LD (#38),hl
            EI
            RET

prog
;------- Programme Principal- ------------------
;------- On change les couleurs. ----------------
            XOR a
            LD bc,#7F54
            LD de,#1044
            OUT (c),a
            OUT (c),c;Pond noir.
            OUT (c),d
            OUT (c),e;Border bleu.
;------ Initialise le mode 1 et le diviseur d'interruptions.
            LD a,#9d
            OUT (c),a
;----- "Decale" l'ecran vers le ler Halt. ------
            LD bc,#bc07
            OUT (c),c
            LD bc,#bd00+#20
            OUT (c),c
vbl
;--- Boucle principale. ---------------------
            LD b,#F5
vsync       IN a,(c) ;Attend le Frame FlyBack.
            RRA ;(Retour du faisceau d'electrons)
            JP nc,vsync
;------- Gestion des lettres. -----------------
cptlet      LD a,-1
            INC a
            AND #3F ;Neuteur d'une lettre+61 lignes(=040H).
            LD (cptlet+1),a
            JP nz,colors
;------- Gestion du texte. ----------------------
scrlct      LD hl,text_
getlet      LD a,(hl)
            INC hl
            OR a
            JP nz,aff
;------- si texte fini,on boucle au debut--------
            LD hl,text_
            JP getlet
aff
;------- Trouve le police de le lettre. --------
            SUB 32
            LD (scrlct+1),hl
            LD de,font
            LD l,a
            LD h,0
            ADD hl,hl;*2
            ADD hl,hl;*4
            ADD hl,hl;*8
            ADD hl,de;chaque fonte fait e octets.
            LD (ctl+1),hl
;----- Gestion des couleurs. -------------------
colors      LD a,1
            LD hl,tabcol ;table des couleurs pour lettres.
            INC a
            AND #3F ;la table fait 040H octets de long.
            LD (colors+1),a
            LD C,A
            LD B,0
            ADD HL,BC
;------- Recupere le couleur dons le table. ------
            LD c,(hl)
ctl         LD hl,0
            LD e,(hl)   ;Couleur de la ligne.
            LD d,#54    ;Couleur du fond (Noir).
;------- Affiche 8 foie une ligne de le fonte).----
ligne       LD a,1
            INC a
            AND 7;hauteur d'une ligne dens le buffer=8.
            JP nz,noligne
            INC hl
            LD (ctl+1),hl
noligne
            LD (ligne+1),a
;----- Affiche lee couleur salon lee pixels de le fonte.
            LD hl,buffel
            LD a,e
            LD b,3        ;trois pixels.
write_      LD (hl),c     ;Affiche in ligne
            RLA           ;correspondant au pixel,
            JP c,pret     ;sauf s'il n'est pas allume:
            LD (hl),d     ;dane ce Cas on efface
pret        INC hl        ;avec le couleur du fond.
            DJNZ write_
; Gestion du buffer.
lscr        EQU 256       ;Nombre de lignes du scroll en memoire.
compt       LD a,0
            INC a
            LD (compt+1),a
            LD b,0
            LD c,a
            LD h,b
            LD l,c
            ADD hl,hl     ;*2
            ADD hl,bc     ;*3
            LD bc,buffer
            ADD hl,bc     ;On obtient l'adresse courante
            EX de,hl      ;du scrolling.
            LD hl,buffel
            LDI           ;copie une premiere foie le ligne de la
            LDI           ;tonte danels buffet.
            LDI
            PUSH de       ;On sauve DE car il pointe sur 1a
            EX de,hl      ;bonne adresse.
            LD bc,3*lscr-3
            ADD hl,bc
            EX de,hl

            LD hl,buffel
            LDI           ;Copie une deuxieme fois.
            LDI
            LDI
            POP de        ;Recupere DE.
;----DE pointe sur la partie du buffer a afficher.
;----On attend le ler Halt.
            EI
            HALT
            DI
            LD (stack+1),sp
; ------ La pile pointe aur la gable de profondeur.
            LD sp,proftab
; ----- On affiche on rester de 200 lignes (hauteur de l'ecran)
            LD a,200
loop        POP hl           ;boucle du Raster
            ADD hl,de
            LD bc,#7F01;On change les couleurs
            OUT (c),c;des encres 1 a 3.
            OUTI
            INC c
            OUT (c),c
            OUTI
            INC c
            OUT (c),c
            OUTI
            LD b,5
tempo       DJNZ tempo ;On attend affin que la boucle
            NOP ;fesse 64 NOPs en temps machine.
            DEC a ; C'est a dire le temps d'une
            JP nz,loop ; rasterline,puls on boucle 200 fois.
stack       LD sp,0
            EI
;------- Teste le touche Control. ----------
key
            LD bc,#F40E
            OUT (c),c
            LD b,#F6
            IN a,(c)
            AND #30
            LD c,a
            OR #c0
            OUT (c),a
            OUT (c),c
            LD bc,#F792
            OUT (c),c
            LD bc,#F642
            OUT (c),c
            LD b,#F4
            IN a,(c)
            LD bc,#F782
            OUT (c),c
            LD bc,#F609
            OUT (C),c
            RLA
;------- Si la touche est pressee, fin du programme...
            JP nc, firm
; ------ Sinon on continue--------------
            JP vbl
;-- Fin du programme. ------------
;
; Texte du scroll. -----------
text_       DEFM "LOGON SYSTEM ..."
            DEFM "UNE NOUVELLE DIMENSION"
            DEFM "AUX DEMOS!"
            DEFS 5,32
            DEFB 0

;------- Table des couleurs. -------------
tabcol
            DEFB #54,#54,#54,#54
            DEFB #54,#54,#54,#54,#54,#54
            DEFB #5C,#5C,#4C,#4C,#4C,#4C
            DEFB #4E,#4E,#4A,#4A,#43,#43
            DEFB #43,#43,#4B,#4B,#43,#43
            DEFB #4B,#4B,#4B,#4B,#44,#44
            DEFB #55,#55,#55,#55,#57,#57
            DEFB #53,#53,#53,#53,#5B,#5B
            DEFB #5B,#5B,#5B,#4B,#4B,#4B
            DEFB #4B,#4B,#4B,#4B,#5B,#5B
            DEFB #54,#54,#54,#54,#54,#54

ecran       EQU #c000

; Fonte pour le scroll
; Attention, il n'y a que
; la ponctuation les chiffres
; et les majuscules...

font
            DEFB #00,#00,#00,#00
            DEFB #00,#00,#00,#00
            DEFB #00,#40,#40,#40
            DEFB #40,#00,#40,#00
            DEFB #00,#a0,#a0,#00
            DEFB #00,#00,#00,#00
            DEFB #00,#60,#f0,#60
            DEFB #60,#f0,#60,#00
            DEFB #00,#40,#e0,#c0
            DEFB #e0,#60,#e0,#40
            DEFB #00,#d0,#d0,#20
            DEFB #40,#b0,#b0,#00
            DEFB #00,#40,#a0,#40
            DEFB #50,#a0,#d0,#00
            DEFB #00,#20,#40,#00
            DEFB #00,#00,#00,#00
            DEFB #00,#e0,#80,#80
            DEFB #80,#80,#e0,#00
            DEFB #00,#e0,#20,#20
            DEFB #20,#20,#e0,#00
            DEFB #00,#00,#a0,#40
            DEFB #e0,#40,#a0,#00
            DEFB #00,#00,#40,#40
            DEFB #e0,#40,#40,#00
            DEFB #00,#00,#00,#00
            DEFB #00,#20,#20,#40
            DEFB #00,#00,#00,#00
            DEFB #e0,#00,#00,#00
            DEFB #00,#00,#00,#00
            DEFB #00,#40,#40,#00
            DEFB #00,#20,#20,#40
            DEFB #40,#80,#80,#00
            DEFB #00,#e0,#a0,#a0
            DEFB #a0,#a0,#e0,#00
            DEFB #00,#40,#c0,#40
            DEFB #40,#40,#e0,#00
            DEFB #00,#e0,#a0,#20
            DEFB #e0,#80,#e0,#00
            DEFB #00,#e0,#a0,#60
            DEFB #20,#a0,#e0,#00
            DEFB #00,#80,#a0,#a0
            DEFB #e0,#20,#20,#00
            DEFB #00,#e0,#80,#e0
            DEFB #20,#a0,#e0,#00
            DEFB #00,#e0,#80,#e0
            DEFB #a0,#a0,#e0,#00
            DEFB #00,#e0,#20,#20
            DEFB #40,#40,#40,#00
            DEFB #00,#e0,#a0,#e0
            DEFB #a0,#a0,#e0,#00
            DEFB #00,#e0,#a0,#a0
            DEFB #e0,#20,#e0,#00
            DEFB #00,#00,#40,#00
            DEFB #00,#40,#00,#00
            DEFB #00,#00,#20,#00
            DEFB #00,#20,#20,#40
            DEFB #00,#00,#20,#40
            DEFB #80,#40,#20,#00
            DEFB #00,#00,#00,#e0
            DEFB #00,#e0,#00,#00
            DEFB #00,#00,#80,#40
            DEFB #20,#40,#80,#00
            DEFB #00,#e0,#a0,#20
            DEFB #60,#00,#40,#00
            DEFB #00,#60,#B0,#00
            DEFB #B0,#80,#60,#00
            DEFB #00,#E0,#A0,#A0
            DEFB #e0,#a0,#a0,#00
            DEFB #00,#e0,#a0,#c0
            DEFB #a0,#a0,#e0,#00
            DEFB #00,#e0,#a0,#80
            DEFB #80,#a0,#e0,#00
            DEFB #00,#c0,#a0,#a0
            DEFB #a0,#a0,#c0,#00
            DEFB #00,#e0,#80,#c0
            DEFB #80,#80,#e0,#00
            DEFB #00,#e0,#80,#c0
            DEFB #80,#80,#80,#00
            DEFB #00,#e0,#80,#80
            DEFB #a0,#a0,#e0,#00
            DEFB #00,#a0,#a0,#e0
            DEFB #a0,#a0,#a0,#00
            DEFB #00,#e0,#40,#40
            DEFB #40,#40,#e0,#00
            DEFB #00,#20,#20,#20
            DEFB #a0,#a0,#e0,#00
            DEFB #00,#a0,#a0,#c0
            DEFB #a0,#a0,#a0,#00
            DEFB #00,#80,#80,#80
            DEFB #80,#80,#e0,#00
            DEFB #00,#a0,#e0,#a0
            DEFB #a0,#a0,#a0,#00
            DEFB #00,#e0,#a0,#a0
            DEFB #a0,#a0,#a0,#00
            DEFB #00,#e0,#a0,#a0
            DEFB #a0,#a0,#e0,#00
            DEFB #00,#e0,#a0,#a0
            DEFB #e0,#80,#80,#00
            DEFB #00,#e0,#a0,#a0
            DEFB #e0,#20,#20,#00
            DEFB #00,#e0,#a0,#a0
            DEFB #c0,#a0,#a0,#00
            DEFB #00,#e0,#80,#e0
            DEFB #20,#20,#e0,#00
            DEFB #00,#e0,#40,#40
            DEFB #40,#40,#40,#00
            DEFB #00,#a0,#a0,#a0
            DEFB #a0,#a0,#e0,#00
            DEFB #00,#a0,#a0,#a0
            DEFB #a0,#a0,#40,#00
            DEFB #00,#a0,#a0,#a0
            DEFB #a0,#e0,#a0,#00
            DEFB #00,#a0,#a0,#40
            DEFB #40,#a0,#a0,#00
            DEFB #00,#a0,#a0,#e0
            DEFB #20,#a0,#e0,#00
            DEFB #00,#e0,#20,#40
            DEFB #80,#80,#e0,#00

;-------Buffers pour le scroll. ---------
buffel DEFS 3,00  ;Buffer de le ligne de police courante
buffer DEFS 2*3*lscr,#54;Buffer pour le scroll en memoire
proftab DEFS 201*2,0 ;Buffer pour la table de profondeur
end
```