LE GATE ARRAY
=============

Vous connaissez Longshot, Digit et Pict, c'est au tour de Fred Crazy de vous dévoiler son savoir sur les méandres des CPC d'Amstrad. Fred, le hardos, vous propose un récapitulatif des fonctions connues et moins connues du Gate Array de votre CPC.

Le Gate Array est le composant qui permet la connexion de Rom, de Rams supplémentaires (appelées aussi Banks), mais aussi le changement de mode graphique et, le plus intéressant, le changement de couleur (les fameux Rasters !). Il s'agit donc d'un petit circuit musclé qui est souvent sollicité sur notre beau CPC. Il faut savoir que le Gate Array est accessible par l'envoi d'une donnée 8 bits sur le port #7F00.

8 BITS POUR QUOI FAIRE?
-----------------------

Procédons par élimination : le bit 5
n'était pas utilisé jusqu'à ce jour, mais
l'arrivée du CPC+ a changé bien des
choses. Afin de ne pas développer deux
pages sur les capacités que possède le
CPC+, nous considérons que ce bit 5
doit être à 0 (de plus amples informations vous seront communiquées par la
suite). Les bits 7 et 6 déterminent
l'action que l'on veut effectuer avec le
Gate Array.

Correspondance des couleurs
BASIC et Gate Array :

Bits 7 et 6 à 0 : chargement d'un registre de palette.
Les bits 3, 2, 1 et 0 contiennent le numéro du registre, (correspond à un
choix de stylo en Basic, par la commande INK), le bit 4 est à 1 si le border est
selectionné, sinon il est à 0.
Bits 7 à 0 et 6 a 1: choix de la couleur à envoyer sur le registre précédemment sélectionné.
Les bits 4, 3, 2, 1, 0 contiennent la couleur, mais attention, le Gate Array possède sa propre palette : regardez le tableau de correspondance Basic
Voilà donc sur quel principe sont basés
les rasters : il faut savoir que l'on peut
changer la couleur d'une même encre
plusieurs fois par ligne, et avoir ainsi à
l'écran un plus grand nombre de couleurs (à ce propos, le record est détenu
par Digit, qui a réussi à afficher 12 couleurs par ligne, et elles sont toutes
visibles !).

LES BONNES RESOLUTIONS
----------------------

Et si le bit 7 est à et le 6 à 0, que se passe-t-il ? Eh bien, nous sommes tout simplement dans le cas de la commutation des Roms et du contrôle vidéo... Eclaircissons le problème. Le bit 4, s'il est à 1, remettra à 0 le diviseur qui génère les interruptions. Il faut l'activer après chaque synchro ! Si le bit 3 est à 1, la Rom supérieure (de #C000 à #FFFF) sera déconnectée ; dans le cas contraire, elle sera connectée. Si le bit 2 est à 1, la Rom inférieure (de #0000 à #3FFF) sera déconnectée, dans le cas contraire connectée.
Les bits 1 et 0 servent pour contrôler la résolution de l'écran.
- B1=0, B0=0 : MODE 0, 160 pixels par 200 en 16 couleurs.
- B1=0, B0=1 : MODE 1, 320 pixels par 200 en 4 couleurs.
- B1=1, B0=0 : MODE 2, 640 pixels par 200 en 2 couleurs.
- B1=1, 80=1 : MODE 3, 160 pixels par 200 en 4 couleurs.
Il ne faudrait pas penser que l'on puisse changer de mode plusieurs fois par ligne. En effet, c'est impossible! (jusqu'à preuve du contraire, le mode s'enclenche à chaque synchro horizontale (HBL !)). De plus, pour sélectionner le numéro de la Rom supérieure que l'on veut connecter, il suffit de passer par le port #DF00. Par exemple, pour sélectionner la Rom 7 (qui est la Rom Disc) il faudrait faire: LD BC, #DF07;OUT (C),C Ce puis indiquer à la Rom haute de se connecter.

PLACEZ VOTRE ARGENT DANS LES BANKS!
-----------------------------------

Nous voilà arrivés au dernier cas, lorsque les bits 7 et 6 sont à 1, nous allons pouvoir sélectionner les 64 Ko supplémentaires du CPC 6128 (ou 464 avec une extension du type DKTRONICS), c'est-à-dire les banks de mémoire. Les bits 4 et 3 ne sont utilisés que pour les extensions 256 Ko. Les 64 Ko supplémentaires sont divisés en 4 blocks de 16 Ko chacun. Le bit 2, lorsqu'il est à 1, permet de connecter le block dont le numéro est contenu par les bits 1 et 0, de #4000 à #7FFF. Pour déconnecter ces blocks, il suffit de mettre 0 sur les bits 2, 1,0 et la Ram redevient normalement linéaire. Mais il existe un cas, des plus intéressants, lorsque le B2=0, B1=1, B0=0, il se trouve tout simplement (III) que les 4 blocks de 16 Ko, se connectent les uns après les autres de #0000 à #FFFF, pour former une nouvelle Ram de 64 Ko ; étonnant, non ? Hélas, vu que les Ram Plus ne sont pas connectées en vidéo (merci Amstrad !), tout ce qui sera chargé de #C000 à #FFFF ne sera pas visible, seule l'ancienne page écran sera affichée (il est entièrement possible de travailler dans cette nouvelle Ram !). Réfléchissez bien au petit problème que ce basculement de Ram implique. II faut un point de chute à votre programme dans les nouveaux 64 Ko, sinon c'est le plantage assuré (vous ne voyez toujours pas ??? Bon, c'est pas grave, laissez tomber 1).

A VOS ASSEMBLEURS
-----------------

Eh bien voilà, vous avez tous les éléments pour comprendre mon petit programme ci-contre, qui est à mon avis très commenté. Amusez-vous à faire des modifications pour en saisir toutes les astuces...
En passant, profitez du dossier Assembleur de Rum vous découvrir ou redécouvrir l'Assembleuur Z80... Bon courage à tous, et à la prochaine...

Fred Crazy


```
; Fred Crazy from Logon System - 03/91 -

            ORG #9000
            DI
            LD HL, #C9FB
            LD (#0038), HL    ; A MORT LES INTERUPTS
            EI
WSYNCHRO    LD B, #F5         ; ATTENDRE SYNCHRO
WSYNCH      IN A, (C)
            RRA
            JR NC, WSYNCH

            LD BC, #7F9D        ; DIVISEUR U'INTERRUPT A !
            OUT (C), C          ; NOUVELLE SEQUENCE D'INTERRUPTS

            LD HL, #008C        ; H=#00 ET L=#8C
            LD DE, #8D8E        ; D=#8D ET E=#8E
            LD C, #10           ; C=#10
            LD B, #7F           ; ADRESSAGE DU GATE ARRAY
            OUT (C), C          ; SELECTION BORDER (BC=#7F10)
            LD A, 12+64         ; ROUGE VIF (VOIR TABLEAU)
            OUT (C), A          ; 64-BIT 6 A 1 DONC CHARGEMENT
            HALT                ; HALT !!! INTERUPTION !!!

            OUT (C), C          ; BC=#7110 SELECTION BORDER
            LD A, 14+64         ; SELECTION DE LA COULEUR ORANGE
            OUT (C), A          ; ENVOIE COULEUR
            OUT (C), L          ; SELECTION MODE 0 (L=#BC=%10001100)
            HALT                ; IDEM.
            LD A, 10+64         ; BORDER EN JAUNE VIF
            OUT (C), C          ; DEVINEZ ?
            OUT (C), A          ; ET LA ?
            OUT (C), D          ; MODE 1 ?
            HALT                ; HEIN !?!

            LD A, 15+64         ; MIR COULEUR
            OUT (C), C          ; BORDER ?
            OUT (C), A          ; ENVOIE DE LA COULEUR
            OUT (C), H          ; SELECTION PAPER (H=0)
            LD A, 31+64         ; DASH 3 ?
            OUT (C). A          ; FACILE 1
            OUT (C), E          ; NOUVEAU MODE 1!! (E=#BE=%10001110)
            HALT                ; HUMMMM !?!
            LD A, 13+64
            OUT (C), C
            OUT (C), A
            LD A, 20+64
            OUT (C), H
            OUT (C), A
            OUT (C), D
            HALT

; A VOUS DE JOUER !
; JE SUIS FATIGUE
; CF PLUS HAUT
; J'AI PEUR DU NOIR !!!
; ET BIEN

            LD A, 24+64         ; C'EST TROP FACILE CETTE FOIS...
            OUT (C), C
            OUT (C), A
            HALT                ; DERNIER HALT ( » 6 HALT PAR SYNCHRO)
            JP WSYNCHRO         ; ET BIEN ALORS BOUCLE LA HAUT !!!
```
