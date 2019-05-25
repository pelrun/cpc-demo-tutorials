LA SCAROLE VERTE Y CALE...
==========================

C'est sous ce jeu de mots douteux que je vais vous parler des scrolls hards verticaux, encore appelés Scrolling-Registre 5 en raison de l'aide précieuse que fournit ce registre du CRTC pour décaler vers le haut (ou vers le bas), et avec fluidité, tout un écran.

Cette technique que nous allons vous décrire est employée dans le jeu Mission Genocide (un budget de Firebird que je ne saurais trop vous conseiller) ou dans quelques démos (comme le menu et la Nega-part de The Demo ou la BSC Megademo par exemple).
La méthode est relativement simple (encore fallait-il la trouver), et je m'en va vous l'expliquer de ce pas. Il suffit en fait de séparer l'écran en deux parties au moins, comme pour une rupture toute simple, ainsi que papy Longshot vous l'a expliqué il y a quelque temps dans un précédent numéro. Ce n'est qu'ensuite que le registre 5 intervient : grâce à lui, on décale sans se fatiguer d'une ligne ou plus un des écrans de la rupture, vers le haut ou vers le bas. Comme le registre 5 ne peut varier qu'entre 0 et 31, on ne se sert, en réalité, que des valeurs 0 à 7, et on décale alors l'écran également à l'aide du pointeur Video, autrement dit, les registres 12 et 13 du CRTC. II faut juste faire attention à une chose, ne pas oublier de changer également la valeur du registre 5 des autres écrans de la rupture. Celle-ci doit être égale au complément à 7 de la valeur du registre 5 de l'écran qui scrolle. Cela doit être fait pour stabiliser les écrans et éviter qu'ils ne sautillent.
Voilà pour la partie hard, mais il reste encore la partie soft, autrement dit l'affichage des motifs du scroll. Eh bien, il suffit de descendre, à chaque balayage d'une ligne sur l'écran, si le scrolling est ascendant et réciproquement afin de compenser le décalage généré par le scrolling hard. Sous des aspects compliqués, ce système est en réalité très simple, et il est beaucoup plus rapide qu'un scrolling vertical réalisé en soft Son seul inconvénient est la nécessité d'une rupture, mais ce problème disparaît avec un usage avisé des interruptions, ce qui est en quelque sorte réalisé dans le programme ci-contre puisqu'il se synchronise sur des HALTs.
Voilà justement une parfaite transition pour vous parler du programme de cette fois-ci, mais je trouve qu'il est suffisamment commenté pour ne pas avoir à l'expliquer; disons simplement qu'il fait défiler un message à caractères géants grâce à un scrolling hard vertical. Par contre, je vais vous parler de la façon dont est générée la fonte géante du scrolling.
Étant donné que chaque caractère ne prend que 64 octets en mémoire et fait 256 lignes de haut sur 64 octets (256 pixels en mode 1) sur l'écran, soit 16 Ko, je crois que le mot « géant » n'est pas de trop. Nous avons déjà vaguement parlé de cette technique de compression lors du listing d'une démo, Chaque octet des caractères en mémoire correspond, en fait, à un mor- ceau du caractère géant, un « bloc » par plusieurs caractères. Dans le programme, ces blocs sont définis gràce à des caractères Ascii assemblés pour ne pas prendre trop de place. Pour la même raison, il n'y a dans le programme que les matrices des caractères qui composent le mot « 100% ». Toutefois, il y a bien une gestion de texte, c'est-à-dire que vous pouvez très bien vous amuser à définir les matrices des autres caractères, avec une restriction cependant : ces doivent être dans l'ordre Ascii. Voilà pourquoi, dans le programme, le « % » de « 100%» est r emplacé par un « / » (slash), code qui est juste avant le «0» et le « 1 » dans ordre Ascii. Je rajouterai enfin que ce type de compression par bloc est utilisé, non seulement, dans les démos our faire défiler des caractères, mais également dans les jeux pour les scrollings de décor.
Donc, le programme fait défiler l'écran du haut vers le bas à raison d'une igne par balayage. Je vous laisse le soin de voir comment changer le sens ou la vitesse, ce n'est pas compliqué, mais cela demande rigueur et attention. De toute façon, on ne possède que ce dont on a l'expérience... Sur ces paroles hautement philosophiques (si ! si !), je vous laisse méditer. A bientôt!

Pict/Logon System

```
; SCROLLING HARD VERTICAL
;
; (c)Pict/Logon System Mai 1993
; Assemble avec DAMS
; charge en banque #c4 (#4000)
              ORG #3000
              ;ENT $
;
; largeur en nots da l'ecran:
;
r1            EQU 32
;
; Initialisation mode et texte
;
              LD a,1
              CALL #bc0e
              CALL #bb4e
; Dessin de la matrice
; des blocs
; (6 blocs de 4 caracteres
; de hauteur)
              LD b,6*4
              LD hl,tabchr
lpaff1        PUSH bc
              LD b,4
;
; on affiche une iigne
; de chaque bloc
;
lpaff2
              LD a,(hl)
              INC hl
              CALL #bb5a
              DJNZ lpaff2
              POP bc
;
; on passe au debut de
; la ligne suivante grace
; aux codes de controle
;
              LD a,10
              CALL #bb5a
              LD a,13
              CALL #bb5a
              DJNZ lpaff1
; On sauve les blocs
;

              LD b,24*8
              LD hl,#c000
              LD de,matrix
lcopy         PUSH bc
              LD bc,8
              LDIR
              LD bc,#800-8
              ADD hl,bc
              JP nc,nocarry
              LD bc, #c050
              ADD hl,bc
nocarry       POP bc
              DJNZ lcopy
;
; Affiche le message
; en bas de l'ecran
;
              XOR a
              CALL #bc0e
              LD h,15
              LD l,2
              CALL #bb75
              LD hl,mess
              LD e,2
affmess
              LD a,e
              PUSH hl
              CALL #bb90
              POP hl
              INC e
              LD a,(hl)
              OR a
              JP z,messaff
              INC hl
              CALL #bb5a
              JP affmess
messaff
;
; Configure le format
; de l'ecran
;
              LD  bc,#bc01
              OUT (c),c
              LD BC,#BD00+r1
              OUT (c),c
              LD bc,#bc02
              OUT (c),c
              LD bc,#bd00+43
              OUT (c),c
;
; on interrompt le
; firmware
;
              DI
              LD hl,(#38)
              LD (syst+1),hl
              LD hl,#c9fb
              LD (#38),hl
;
; Passe en Banque #c0
; qui est en fait une
; partie de l'ecran.
;
              LD bc,#7fc0
              OUT (c),c
;
; Éfface l'ecran en #4000
;
              LD hl,#4000
              LD (hl),0
              PUSH hl
              POP de
              PUSH de
              POP bc
              INC de
              DEC bc
              LDIR
;
; Change les couleurs
; des encres
;
              LD hl,mess
              LD bc,#7f00+13
colorlp       OUT  (c),c
              DEC hl
              LD a,(hl)
              OUT (c),a
              DEC c
              JP nz,colorlp
;
; Boucle principale
;
BCLP          DI
;
; on attend la synchro Verticale
;
              LD B,#F5
vs            IN A,(c)
              RRA
              JP NC,vs
;
; Aussitot apres,on configure
; les registres pour la rupture
;
; On affiche 256 lignes
;
              LD BC,#BC06
              OUT (c),c
              LD BC,#BD20
              OUT (c),c
              LD BC,#BC07
              OUT (c),c
              LD BC,#BD7F
              OUT (c),c
;
; On passe en mode 1
; at on initialise le
; diviseur d'interruption
;
              LD BC,#7F9D
              EI
;
; On met en noir l'encre i
; afin de cacher l'effet de
; saccade creer par le decalage
; de l'ecran
;
              LD de,#0154
              OUT (c),c
              OUT (c),d
              OUT (c),e
;
; Gestion Scroll Vertical
; on envoie l'adresse Hard
; de l'ecran aux registres
; du CRTC
;
OFFSET        LD HL,#1000
              LD BC,#BC0C
              OUT (c),c
              INC b
              OUT (c),h
              DEC b
              INC c
              OUT (c),c
              INC B
              OUT (c),l
;
; On fait monter l'ecran
; d'une ligne grace au
; registre 5,et toutes
; les 8 lignes,on change
; l'adresse Hard de l'ecran
;
charup        LD  de,r1
VERTY         LD a,8
speed         SUB 1
              AND 7
              LD  (VERTY+1),A
              JP  nz,ver
              ADD HL,de
ver
;
; On s'assure que l'adresse
; hard de l'ecran scrollant
; ne deborde pas
;
              LD c,a
              LD a,h
              AND %11
              OR #10
              LD h,a
              LD (OFFSET+1),HL
;
; On envoie la valeur de
; decalage au registre 5
; du crtc
              LD a,c
xor1          XOR 0
              LD BC,#BC05
              OUT (c),c
              INC b
              OUT (c),c
;
; on attend quelques Lignes
;
              LD  bc,260
tempo
              DEC bc
              LD a,b
              OR c
              JP  nz,tempo
;
; puis on initialise
; le reg 4 du CRTC
;
              LD BC,#BC04
              OUT  (c),c
              LD BC,#BD1B
              OUT (c),c
;
; ainsi que l'adresse
; du second ecran
;
OFFSET2       LD  HL,#3000
              LD  BC,#BC0C
              OUT (c),c
              INC b
              OUT (c),h
              DEC b
              INC c
              OUT (c),c
              INC b
              OUT (c),l
;
; 11 faut alors
; complementer le
; regiatre 5
;
VERTICA1      LD A, (VERTY+1)
              LD BC,#BC05
              OUT (c),c
              INC B
xor2          XOR 7
              OUT (c),a
;
; puis on affiche
; le 1er raster
;
              LD bc,3
temp1         DEC bc
              LD a,b
              OR c
              JP nz,temp1
              LD hl,raster1
              LD a,10
              LD bc,#7f01
              OUT (c),c
rastlp1       LD c,(hl)
              OUT (c),c
              INC hl
              LD e,12
tempr1
              DEC e
              JP nz,tempr1
              NOP
              NOP
              DEC a
              JP nz,rastlp1
;
; On a plus qu'a attendre
;
N1            HALT
N2            HALT
N3            HALT
N4            HALT
;
; ...-attendre encore...
;
              LD bc,324
temp2         DEC bc
              LD a,b
              OR c
              JP nz,temp2
;
; et afficher le 2nd raster
;
              LD hl,raster2
              LD a,10
              LD bc,#7f01
              OUT (c),c
rastlp2       LD c,(hl)
              OUT (c),c
              INC hl
              LD e,12
tempr2
              DEC e
              JP nz,tempr2
              NOP
              NOP
              DEC a
              JP nz,rastlp2
N5            HALT
;
; Passe en mode 0
;
              LD bc,#7f8c
              OUT (c),c
;
; Reconfigure les
; registres du CRTC
; pour le 2eme ecran
;
              LD BC,#BC04
              OUT (c),c
              LD BC,#BD07
              OUT (c),c
              LD BC,#BC07
              OUT (c),c
              LD BC,#BD05
              OUT (c),c
              LD BC,#BC06
              OUT (c),c
              LD BC,#BD05
              OUT (c),c
;
; Gestion de l'affichage
;
;
; les blocs font 32 lignes
; de hauteur
;
ctmat         LD a,#1f
              INC a
              AND #1f
              LD (ctmat+1),a
              JP nz,cnty
;
; les caracteres font 8 bloca de hauteur
;
ctchr         LD a,7
              INC a
              AND 7
              LD (ctchr+1),a
              JP nz,chline
;
; gestion du texte
;
ctxt          LD hl,text_
              LD a,(hl)
              INC hl
              OR a
              JP nz,charok
              LD hl,text_
              LD a,(hl)
              INC hl

charok
              LD  (ctxt+1),hl
              CP 32
              JP nz,nospace
;
; si c'est un caractere
; d'espacement ,on affiche
; des octets nuls (qui sont
; en fait dans le 1er bloc)
;
              LD hl,matrix
              LD (chline+1),hl
;
; et on s'arrange pour qu'il
; fasse 4 blocs de haut
;
              LD a,4
              LD (ctchr+1),a
              JP chline
;
; sinon gn trouve quelle
; matrice lui correspond
; (on multiplie par 64,
; ce qui est la taille
; de la matrice d'un
; caractere)
;
nospace
              SUB 47
              LD h,0
              LD l,a
              ADD hl,hl
              ADD hl,hl
              ADD hl,hl
              ADD hl,hl
              ADD hl,hl
              ADD hl,hl
              LD de,tabchar
              ADD hl,de
              LD (chline+1),hl
;
; on copie l'adresse de la
; des biocs dans un buffer
;
chline        LD hl,tabchar
              LD bc,matrix
              PUSH iy
              LD iy,buffer
              LD a,8
nline
              LD d,(hl)
              LD e,0
              EX  de,hl
              ADD hl,bc
              EX  de,hl
              INC hl
              LD (iy+1),d
              LD (iy+0),e
              INC iy
              INC iy
              DEC a
              JP nz,nline
              POP iy
              LD (chline+1),hl
;
; On descené sur l'ecran
; d'une ligne pour compenser
; l'effet de scrolling
;
cnty          LD hl,#4000
              LD a,h
              ADD a,8
              LD h,a
              AND #38
              JP nz,noca
              LD a,h
              SUB #40
              LD h,a
              LD a,l
              ADD a,r1*2
              LD l,a
              JP nc,noca
              INC h
              LD a,h
              AND 7
              JP nz,noca
              LD a,h
              SUB 8
              LD h,a
noca
              LD (cnty+1),hl
;
; on affiche une ligne de
; chacun des blocs du
; caractere grace a la pile
; qui pointe sur le buffer
; decrit plus haut
;
              EX de,hl
              DI
              LD (stack+1),sp
              LD sp,buffer
              LD a,8
mlp
              POP hl
              LDI
              LDI
              LDI
              LDI
              LDI
              LDI
              LDI
              LDI
              PUSH hl
              POP hl
              DEC a
              JP  nz,mlp
stack         LD sp,0
;
; Test de la barre espace
;
scan
              LD BC,#F40E
              OUT (c),c
              LD BC,#F6C0
              OUT (c),c
              XOR a
              OUT (c),a
              LD BC,#F792
              OUT (c),c
              DEC b
              LD C,#45
              OUT (c),c
              LD B,#F4
              IN  A,(c)
              LD BC,#F782
              OUT (c),c
              DEC b
              LD C,#00
              OUT (c),c
              EI
              AND #80
              JP NZ,BCLP
;
; Fin du programme:
; on restaure les registres
; du CRTC,les banques de
; memoire et le systene
;
sys
              DI
              LD bc,#7fc4
              LD de,#014b
              OUT (c),c
              OUT (c),d
              OUT (c),e
              LD BC,#BC04
              OUT (c),c
              INC B
              LD c,38
              OUT (c),c
              LD BC,#BC02
              OUT (c),c
              INC B
              LD  C,46
              OUT (c),c
              LD BC,#BC01
              OUT (c),c
              INC B
              LD C,40
              OUT (c),c
              LD BC,#BC07
              OUT (c),c
              INC B
              LD C,30
              OUT (c),c
              LD bc,#BC06
              OUT (c),c
              INC b
              LD c,25
              OUT (c),c
syst          LD HL,#0000
              LD (#38),hl
              EI
              RET
;
; table des couleurs
; du raster du haut
;
raster1
              DEFB #44,#44
              DEFB #55
              DEFB #57
              DEFB #5f,#5f
              DEFB #53,#4b,#5b
              DEFB #4b
;
; table des couleurs
; du raster du bas
;
raster2
              DEFB #43,#4b,#43
              DEFB #4a,#4a
              DEFB #4e
              DEFB #4c
              DEFB #5c,#5c
;
; Table des couleurs
; des encres
;
              DEFB #54,#4b,#44,#55
              DEFB #57,#5f,#53,#54
              DEFB #4b,#43,#4a,#4e
              DEFB #4c,#5c
;
; Message du bas de l'ecran
;
mess
              DEFM "Logon System"
              DEFB 0
;
; Massage scrollant
;
text_          DEFM 100
               DEFB 0
;
; Matrices des blocs
;
tabchr
v             EQU 128
              DEFB v,v,v,v
              DEFB v,v,v,v
              DEFB v,v,v,v
              DEFB v,v,v,v
p             EQU 143
              DEFB p,p,p,p
              DEFB p,p,p,p
              DEFB p,p,p,p
              DEFB p,p,p,p
q             EQU 212
              DEFB p,p,p,q
              DEFB p,p,q,v
              DEFB p,q,v,v
              DEFB q,v,v,v
s             EQU 213
              DEFB s,p,p,p
              DEFB v,s,p,p
              DEFB v,v,s,p
              DEFB v,v,v,s
t             EQU 214
              DEFB v,v,v,t
              DEFB v,v,t,p
              DEFB v,t,p,p
              DEFB t,p,p,p
u             EQU 215
              DEFB u,v,v,v
              DEFB p,u,v,v
              DEFB p,p,u,v
              DEFB p,p,p,u
;
; Matrices des caracteres
;
tabchar
;
; (Attention,on prand en
; fait le code ASCII du
; caractere “/"-slash-)
;%
DEFB 0,4,1,5,0,0,4,0
DEFB 0,1,0,1,0,4,2,0
DEFB 0,3,1,2,4,2,0,0
DEFB 0,0,0,4,2,0,0,0
DEFB 0,0,4,2,4,1,5,0
DEFB 0,4,2,0,1,0,1,0
DEFB 0,2,0,0,3,1,2,0
DEFB 0,0,0,0,0,0,0,0
;0
DEFB 0,4,1,1,1,1,5,0
DEFB 0,1,1,0,0,1,1,0
DEFB 0,1,1,0,4,1,1,0
DEFB 0,1,1,4,2,1,1,0
DEFB 0,1,1,2,0,1,1,0
DEFB 0,1,1,0,0,1,1,0
DEFB 0,3,1,1,1,1,2,0
DEFB 0,0,0,0,0,0,0,0
;1
DEFB 0,0,4,1,1,0,0,0
DEFB 0,0,1,1,1,0,0,0
DEFB 0,0,0,1,1,0,0,0
DEFB 0,0,0,1,1,0,0,0
DEFB 0,0,0,1,1,0,0,0
DEFB 0,0,0,1,1,0,0,0
DEFB 0,1,1,1,1,1,1,0
DEFB 0,0,0,0,0,0,0,0
;espace
;
; Buffer pour les blocs
;
matrix DEFS 64*24
;
; buffer pour l'affichage
; des lignes
;
buffer DEFS 2*8,0
end
```