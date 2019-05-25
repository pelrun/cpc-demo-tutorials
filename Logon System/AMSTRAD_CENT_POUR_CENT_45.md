SINUS DOTS
==========

Et hop ! Ce mois-ci, c'est encore moi, Pict, qui m'y colle pour votre rubrique préférée.

Je tiens tout d'abord à dire que je ne m'appelle en aucun cas Maxim Barrou, comme le titre du précédent article le laissait croire (je ne sais toujours pas qui est l'auteur de ce titre étrange, mais ça m'apprendra à rendre mes articles sans titre...). Aujourd'hui, nous allons parler animation, avec des pixels bougeant selon des courbes sinusoïdales, encore appelées Sinus Dots en jargon de demomaker.

PAGE-FLIPPING
-------------

A travers ce programme, qui anime près de 400 pixels en 50 Hertz (record I), nous allons voir comme d'habitude quelques techniques d'optimisation, mais aussi un système d'animation particulier, le Page-Flipping.
Mais encore un dernier mot concernant le programme du mois dernier : des coquilles semblent s'être incrustées lors de l'impression du listing. Il faut en effet taper "ENT $" à la place de "$" au début du programme, et "#54" à la place de &50H vers la fin du listing. Il y a également un problème dont j'assume, cette fois-ci, la lourde responsabilité dans le programme Basic : l'instruction GRAPHICS PEN est mal digérée sur 464 ! Mais cette histoire est en fait très simple à résoudre si vous connaissez votre Basic. II suffit en effet de rajouter, à l'instruction DRAW ou PLOT, un troisième paramètre, ce dernier étant l'encre désirée...
Voilà pour le flash-back, maintenant, attaquons-nous au programme de ce mois!

RUSE OU RIGUEUR?
----------------

Le Page-Flipping, ou encore PageSwitching (connexion d'écran), bien qu'il soit surtout utilisé dans les jeux, est un système qui tient plus de la ruse de demomaker que de la rigueur d'un programmeur professionnel.
Cependant, cette technique n'est pas spécifique à l'Amstrad et est employée sur tous les ordinateurs de jeux. Son principe repose sur le basculement de 2 écrans : une fois qu'un écran est prêt, on l'affiche et, pendant ce temps, on modifie l'autre écran, qui est donc caché. Ce système est en fait nécessaire à partir du moment où l'on veut afficher de gros sprites, faire un gros scrolling, ou n'importe quoi modifiant l'écran qui prend beaucoup de temps machine. Pourquoi ? Parce que sinon l'animation se mange le balayage du faisceau d'électrons qui donne un résultat hideux à l'écran... Etant donné que nous animons plusieurs centaines de points simultanément, cette technique est quasiment obligatoire !
Pour réaliser le Page-Flipping, nous allons utiliser les deux pages vidéo de 16 Ko se trouvant en &4000 et &c000. Pourquoi ces deux pages-là plutôt que d'autres ? Eh bien, car encore une fois, notre bon vieux CPC nous tend une perche: le Gate Array a une commande qui permet de considérer l'écran en #c000 comme étant celui en #4000, cela est possible par un simple OUT &7F00,&C3 en Basic (Longshot et Sined ont d'ailleurs déjà parlé de cette possibilité dans des Cent Pour Cent précédents). Cette astuce nous permet de garder la même adresse absolue quel que soit l'écran que l'on trafique.

DEJA MAL ALATETE?
-----------------

Notre animation étant composée de pixels suivant des courbes sinusoïdales en abscisse et en ordonnée, nous allons gagner un maximum de vitesse en transformant les valeurs de la table de sinus en adresse de ligne d'écran pour les ordonnées, et en masques et octets pour l'abscisse AVANT de faire l'animation. Grâce à ce précalcul, il suffira d'additionner l'adresse de la ligne écran, donnée par la courbe des ordonnées, à l'octet des abscisses pour avoir l'adresse du pixel, et de masquer cette adresse par l'autre octet des abscisses, qui est donc le masque donnant le pixel. Toujours dans ce souci de rapidité, bien que l'on soit en mode 1, on n'utilise que l'encre 3, car le masquage se fait plus rapidement avec un écran où il n'y a qu'une seule encre (il suffit d'un simple OR). L'écran a subi un reformatage féroce et sauvage qui permet d'additionner l'abscisse et l'ordonnée grâce à une opération de 8 bits au lieu de 16, et donc de gagner quelques cycles (qui font en fait beaucoup de temps machine gagné puisqu'il faut multiplier le gain par le nombre de points affichés). En effet, l'écran reformaté fait 256 lignes de haut sur 256 pixels de large, soit 64 octets de large ; dans un tel format, le poids faible de l'adresse du premier octet de chaque ligne de l'écran étant toujours inférieur à 192, et comme 192 + 64 = 256, une addition de 8 bits suffit donc. De plus, si l'écran est moins large, vous bénéficiez par contre d'un overscan vertical sympathique.
Venons en à l'animation proprement dite. Vous avez 4 variables paramétrables en début de programme qui sont la vitesse d'animation et l'écart entre chaque pixel pour l'abscisse et l'ordonnée ; essayez autant de combinaisons et de valeurs que vous voulez (mais ne dépassez pas 1023 ou je ne réponds plus de rien !), il se passe toujours quelque chose de différent sur l'écran !

PAS DE DETAILS...
-----------------

Je ne vais pas détailler le listing davantage, il est déjà commenté en profondeur dans les remarques, mais je tiens tout de même à faire quelques précisions: le RES 3,H que vous pouvez voir plusieurs fois dans le programme est une sorte de compteur autobouclant qui présente 2 contraintes contrastant, hélas ! avec son efficacité et sa rapidité. Tout d'abord, comme les compteurs autobouclants du mois dernier, il faut que la table contienne un nombre d'octets qui soit une puissance de 2 (ici 2" = 2048 octets) et de plus, il faut que la table de données commence à une adresse multiple de 256, ayant un poids faible, nul autrement dit . L'effacement des pixels se fait à la pile grâce à 2 buffers (un pour chaque écran du pageflipping) contenant les adresses des pixels à effacer. Aussitôt après, on affiche les nouveaux points ; regardez bien cette routine, elle est difficile à appréhender (et à comprendre !. Je tiens d'ailleurs à dire que je ne l'ai pas pondue d'un seul coup, mais que j'ai passé plusieurs heures à la cogiter, afin d'avoir une routine rapide sans être pour autant trop gourmande en mémoire.
Eh oui, ne croyez pas que savoir programmer signifie être capable de faire une routine par la seule force de ses doigts en tapotant sur le clavier, il faut aussi (et c'est le plus dur) se servir de sa tete (n'est-ce pas Poum ? Nooon, pas un coup de boule ! Aïe ! Hum, c'est comme ça qu'il se sert de sa tête, lui?). En bref, un papier et un crayon sont souvent les meilleures armes face au hug récalcitrant... Je vous laisse ainsi méditer sur ces bonnes paroles, et je vous souhaite une bonne rentrée et de bonnes routines !

A + ! Pict (Emmanuel, pas Maxim, merci)

```
; Courbes parametrees
; (c) Pict/Logon System
; Pour Amstrad 100%

ORG #1000

; Valeurs parametrables
; Nol; avx Vitesse en X a
; chaque vertical Blanking

avx EQU 2

; No2; decx Ecart en X
; entre chaque pixel

decx EQU 3

; No3; avy Vitesse en Y a
; chaque Vertical Blanking

avy EQU 2

; No4; decy Ecart en Y
; entre chaque pixel

decy EQU 6

; Nombre Total de point affiches.
nbpoint EQU 341

; Veut-on verifier le Temps Machine
; Pris par les differentes routines?
; (0=Non) (1=Oui).

test EQU 0

; Table des ordonnees.
; Attention! sintaby doit toujours
; etre une adresse multiple de 256.


sintaby     DEFS #02*#400,#00

; Table des sinus
; generes en Basic

sintab      DEFS #01*#400,#00

; Table des adresses
; de chaque ligne ecran.

tabline     DEFS #100*02,#00

;Nom du fichier a charger

nom1        DEFM "SINUS   .BIN"

; On change d'origine car
; la table des abscisses,
; sintabx doit egalement
; debuter a une adresse
; multiple de 256.

            ORG #2000
sintabx     DEFS #02*#400,#00

; Valeurs CRTC du programme.

crtprog     DEFB #20,#2A,#0F,#27
            DEFB #00,#20,#22

; Valeurs CRTC du systeme.

crtfirm     DEFB #28,#2E,#0F,#27
            DEFB #00,#19,#1E

; Table pour afficher un pixel
; en mode 1 avec l'encre 3.

pixeltab    DEFB %10001000,%00100010
            DEFB %01000100,%00010001

; Buffers pour l'effecage des points.
buffer1     DEFS nbpoint*#02,#40
buffer2     DEFS nbpoint*#02,#40

; ***DEBUT DU PROGRAMME.***
go          run $

; On charge le fichier
; contenant les sinus.
load        LD hl,nom1
            LD de,sintab
            LD B,#0C
            PUSH DE
            CALL #BC77
            POP HL
            CALL #BC83
            CALL #BC7A

; on coupe le systeme apres
; avoir sauve son adresse
; d'execution.
            LD hl,(#0038)
            LD (sys+1),hl
            LD hl,#c9fb
            LD (#38),hl

; On coupe les interruptions.
            DI

;on sauve les registres secondaires
;carle systeme les utilise.
            EX af,af'
            EXX
            PUSH af
            PUSH bc
            PUSH de
            PUSH hl
            PUSH ix
            PUSH iy

; INITIALISATION DES VARIABLES
; on initialise la table
; des adresses-ecran.
          CALL creeline

; On initialise le CRTC.
          LD hl,crtprog
          CALL setcrtc

; On initialise le Gate-Array
; (Mode,couleurs,banque video).
          LD bc,#7F10
          LD de,#544B
          LD hl,#BDC0
          LD a,#3
          OUT (c),h
          OUT (c),l
          OUT (c),c
          OUT (c),d
          OUT (c),a
          OUT (c),e
          XOR a
          OUT (c),a
          OUT (c),d

; On vide les 2 ecrans utilises.
          LD hl,#4000
          LD de,#4001
          LD (hl),l
          LD bc,#3FFF
          LDIR
          LD hl,#c000
          LD de,#c001
          LD (hl),l
          LD bc,#3FFF
          LDIR

; BOUCLE PRINCIPALE
; On attend le Frame FlyBack
vbl       LD b,#F5
vsync     IN a,(c)
          RRA
          JP nc,vsync

; Permet de voir le temps
; machine (si desire).
          IF test
          LD a,#4C
          CALL border
          ENDIF

; EFFACAGE DE L'ECRAN
; On sauve la pile
          LD (stack+1),sp
          LD sp,(buffadr+1)
          LD bc,nbpoint

; Et on recupere grace a cette
; derniere les coordonnees des
; points a effacer
          XOR a
          LD e,a
clear     POP hl
          LD (hl),e
          DEC bc
          LD a,b
          OR c
          JP nz,clear

; La pile pointe sur la table
; necessaire a l'effacage.
          LD sp,(buffadr+1)
          LD hl,#02*nbpoint
          ADD hl,sp
          LD sp,hl
adrx      LD hl,sintabx
speedx    LD de,avx*#02
          ADD hl,de
          RES 3,h
          LD (adrx+1),hl
addx      LD de,#02*decx-1
          EXX
adry      LD hl,sintaby
speedy    LD de,avy*#02
          ADD hl,de
          RES 3,h
          LD (adry+1),hl
addy      LD de,#02*decy-1

; Nombre de points a afficher
          LD bc,nbpoint

; AFFICHAGE DES POINTS
plot
; Recupere le poids fort
; do l'ordonneo.
          LD a,(hl)
          INC l
          EXX
          LD b,a

; Recupere le poids faible de
; l'ordonnee,passe a l'ordonnes
; suivante...
          EXX
          LD a,(hl)
          ADD hl,de
          RES 3,h
          EXX
; et additionne ce poids faible
; au poids fort de l'abscisse.
          ADD a,(hl)
          LD c,a
          INC l
; Recupere la donnee de l'ecren
; a l'dresse du point
          LD a,(bc)

; et le masque avec la valeur
; du correcte du pixel a afficher
          OR (hl)

; On sauve l'adresse ecran
; pour l'offacage,avec la pile.
          PUSH bc

; On affiche le point.
          LD (bc),a

; Passe a l'abscisse suivante.
          ADD hl,de
          RES 3,h
          EXX

; On tait ca autant de foie
; qu'il y a de points.
          DEC bc
          LD a,b
          OR c
          JR nz,plot

; On recupere la pile.
stack     LD sp,0

; Test temps machine.
          IF test
          LD a,#52
          CALL border
          ENDIF

; PAGE FLIPPING.
; Echange les 2 ecrans
; a chaque balayage pour
; ne pas se prendre le
; faisceau d'elactrons
; dans le tronche.

buffadr   LD hl,buffer1
          LD de,buffer2
          LD (buffadr+1),de
          LD (buffadr+4),hl
switch1   LD hl,#c030
switch2   LD de,#c310
          LD (switch1+1),de
          LD (switch2+1),hl
          EX de,hl
          LD b,#7F
          OUT (c),h
          LD bc,#bc0c
          OUT (c),c
          INC b
          OUT (c),l
          DEC b
          INC c
          XOR a
          OUT (c),c
          INC b
          OUT (c),a

; Test temps machine.
          IF test
          LD a,#44
          CALL border
          ENDIF

; ***TEST DE TOUCHE.***
key       LD bc,#F782
          OUT (c),c
          LD bc,#F40E
          OUT (c),c
          LD bC,#f6C0
          OUT (c),c
          XOR a
          OUT (c),a
          LD bc,#F792
          OUT (c),c
          LD de,#f4f6
          LD c,#45
          LD b,e
          OUT (c),c
          LD b,d
          IN d,(c)
          LD bc,#F782
          OUT (c),c
          DEC c
          OUT (c),a
; Remet le border noir
; (pour test T-M)
          IF test
          CALL black
          ENDIF

; *FIN DE LA BOUCLE.*
; Teste le barre espace.
          RL d
          JP c,vbl

; Si elle est enfonces,
; On arrete le programme.
; On remet les valeurs
; CRTC du systems.
          LD hl,crtfirm
          CALL setcrtc

; On se replace dans la
; banque utilisee.
          LD bc,#7Fc4
          OUT (c),c

; On reconnecte le systeme.
sys       LD hl,#0000
          LD (#0038),hl

; On recupere les registres
; secondaires.
          POP iy
          POP ix
          POP hl
          POP de
          POP bc
          POP af
          EXX
          EX af,af'

; On remet les interruptions.
          EI
; Fini!
          RET

; ROUTINE DE CREATION
; TABLES D'ADRESSES
creeline  LD hl,#c000
          LD de,#C040
          LD bc,#0800
          LD iy,tabline
          XOR a

; XOR A equivaut a LD A,256
; On tree une table contenant
; l'adresse de chaque ligne.
; (l'ecran fait 256 lignes)
creeloop  LD (iy+#01),l
          LD (iy+#00),h
; On se place sur l'ecran en #4000
          RES 7,(iy+#00)
          ADD hl,bc
          JP nc,nocarry
          ADD hl,de
nocarry   INC iy
          INC iy
          DEC a
          JP nz,creeloop

; On cree la table des ordoneos
; en transformant la table de
; sinus faite an Basic en
; adresses-ecran
          LD ix,sintab
          LD iy,sintaby

; en transformant la table de
; sinus faite en Basic en
; adrosses-ecran
          LD ix,sintab
          LD iy,sintaby
          LD de,tabline
          LD bc,#400
loopy     LD h,0
          LD l,(ix+#00)
          ADD hl,hl
          ADD hl,de
          LD a,(hl)
          LD (iy+#00),a
          INC hl
          LD a,(hl)
          LD (iy+1),a
          INC iy
          INC iy
          INC ix
          DEC bc
          LD a,b
          OR c
          JP nz,loopy

; On fait de meure avec les
; abscisses.
          LD ix,sintabx
          LD iy,sintab
          LD de,pixeltab
          LD bc,#400
loopx     LD h,0
          LD l,h
          LD a,(iy+#00)
          SRL a
          RL l
          SRL a
          RL l
          ADD hl,de
          LD (ix+#00),a
          LD a,(hl)
          LD (ix+#01),a
          INC ix
          INC ix
          INC iy
          DEC bc
          LD a,b
          OR c
          JP nz,loopx
          RET

; Routine de changement
; de couleur du border
; pour les mesures T-M.
black     LD a,#54
border    LD bc,#7F10
          OUT (c),c
          OUT (c),a
          RET

; Routine d'installation
; des valeurs du CRTC.
setcrtc   LD a,#01
          LD b,#BC
setloop   LD c,(hl)
          OUT (c),a
          INC b
          OUT (c),c
          INC hl
          DEC b
          INC a
          CP #08
          JP nz,setloop
          RET
```

```
10 REM Programme creant la table de
15 REM sinus necessaire au programme
20 MODE 1:ORIGIN 0,0:DEG
30 MEMORY &2FFF:adr=&3000:c=0
40 FOR a=0 TO 360 STEP 360/1024
50 b=INT(128+127.5*SIN(a))
60 PLOT a,b
70 POKE adr+c,b:c=c+1
80 NEXT
90 SAVE "sinus",b,adr,c
```
