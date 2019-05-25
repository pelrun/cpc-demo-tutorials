RUPTURE FACILE
==============

Pour ce numéro d'avril, nous retrouvons le père des Logon System, pour revoir le problème de la rupture d'écran sur CPC. Pour une meilleure compréhension de cet article, relisez notre numéro de décembre où Longshot vous expliquait déjà comment repousser les limites du CRT, le processeur vidéo des CPC.
Place à longshot !

Comme je vous l'avais expliqué, la rupture est la méthode consistant à changer l'adresse de l'écran en cours de balayage. Pour effectuer cela, plusieurs registres CRTC sont à notre disposition, notamment le 4, 5 et 9 (revoir le numéro de décembre). Un léger problème subsistait cependant : comment stabiliser correctement l'écran ? Lorsqu'un VBL (Vertical Blanking) commence pour un écran, une zone de synchronisation (présente dans l'overscan vertical) est générée. Le CRT doit se baser sur cette zone pour synchroniser l'image avec le début du balayage. Comment peut-il le faire ? Tout simplement grâce à un nouveau registre : le numéro 7!

SYNCHRONISONS
-------------

La synchronisation de l'écran doit être basée par rapport au VBL de la dernière rupture générée (lorsqu'on en a plus de 2, je parle des ruptures). Le registre 7 permet d'indiquer, en partant du bas de l'écran, le nombre de lignes caractères où trouver le signal de synchro (qui est, je le répète, dans le VBL de la dernière rupture) pour que l'image puisse être stabilisée.

Ce registre doit donc prendre sa véritable valeur durant la période d'affichage de la dernière rupture. Donc, pour être un peu plus concret, je reprends la rupture simple déjà évoquée, c'est-à-dire : le registre 4 = 19, le registre 5 = 0 et le registre 9 = 7.

Bon, comme je vous l'avais expliqué, chaque rupture génère une VBL.. ce qui implique donc que, pour chaque VBL, le registre 7 de synchronisation doit être pris en compte. Cette prise en compte du registre 7 est traduite sur l'écran, à l'endroit de la VBL, par une « bande noire » qui correspond à la zone de synchronisation. Or, si cette zone nous intéresse pour stabiliser notre écran par rapport à la dernière rupture, elle ne nous intéresse guère pour les ruptures précédentes. Que faire pour empêcher son apparition ? Non Robby, on ne met pas un sparadrap sur l'écran!

DUPONS LE CRT
-------------

Oui, tout simplement, nous allons feinter le CRT. Pourquoi ne pas mettre le registre 7 à sa valeur maximale (qui est 127) ? Nous lui indiquerions ainsi que la zone de synchro (je rappelle que seule la dernière compte), pour toutes les ruptures avant la dernière, sera très très très bas. Donc en mettant registre 9 = 127, nous obtenons l'effet désiré : la bande noire disparaît !
Oui, je sais, tout cela est très théorique Eh bien voyons l'application de la chose grâce à un petit exemple prédigéré : les démos de Fefesse à la portée de tous ! Voici donc pour terminer ce chapitre sur la rupture, qui, je l'espère, vous a plu ! Vous pouvez ranger votre double-hache car vous devriez, grâce à ma petite routine commentée, vous offrir des ruptures sympa même sous Basic. Il reste beaucoup de notions à développer, mais comme je le disais hier à Luke Skywalker, Demain, tu seras prêt !!

Longshot

```
;
; Rupture Simple sous Basic
; Longshot. Logon System pour Amstrad 100 %
; LES RECOMMANDATIONS DE PAPY LONGSHOT
;
; Sauvegarder le code et utiliser sans DAMS (Perturbations)
; CALL #A000 installe le mode rupte
; CALL #A003 désinstalle

          ORG #A000

          JP INSTALL      ; Rupture installée
          JP DESINST      ; Rupture désinstallée

INSTALL
          DI              ; Hip Hop Je suis seul
          LD HL,(#39)    ; Vecteur Std Inter
          LD DE,BUF       ; zone sauvegarde
          PUSH HL
          LDI
          LDI
          LDI             ; Transfère 3 bytes
          LD C,(HL)         ; Lire Byte Dept du JR
          LD B,0
          INC HL            ; Instruc. Suivante
          LD  (MRET2+1),HL  ; Ptr Ret Cond. Fausse
          ADD HL,BC         ; Adresse Arrivee is CPC
          LD  (MRET1+1),HL  ; Ptr Ret Cond Vraie
          POP DE            ; Modif Rout Système
          LD  HL,VECTEUR    ; transfert vecteur pirate
          LDI
          LDI
          LDI
          EI                ; En avant marche !
          RET
VECTEUR   JP INTER
BUF       DS 3,0

DESINST
          DI
          LD DE,(#39)
          LD HL,BUF         ; restitution du buffer
          LDI
          LDI
          LDI

          LD BC,#BC04      ; reprogrammation Std du CRT
          OUT (C),C
          LD BC,#BD00+38
          OUT (C),C

          LD BC,#BC07
          OUT (C),C
          LD BC,#BD00+30
          OUT (C),C

          LD BC,#BC06
          OUT (C),C
          LD BC,#BD00+25
          OUT (C),C

          EI
          RET

; Routine Interruption

INTER
          PUSH AF           ; sauvegarde de quelques registres
          PUSH DE
          PUSH HL
          PUSH BC
          LD HL,COUNTI+1    ; compteur Inter
          LD B,#F5         ; gestion période compteur
          IN A,(C)
          RRA
          JP NC,NEXTI
          LD (HL),#FF      ; lere Inter
NEXTI
          INC (HL)          ; Num Inter+1 (1/300 x 6)
COUNTI    LD A,0
          CP 6              ; vérif pas de pb (Cas
          JR C,OK           ; de CRTC bizarroïdes)
          XOR A

OK
          SLA A             ; x 2 + TabVectInt
          LD C,A
          LD B,0
          LD HL,TABINT
          ADD HL,BC

          LD A,(HL)         ; donne Ptr VecNum
          INC HL
          LD H,(HL)
          LD L,A
          LD BC,RETOUR      ; adresse retour
          PUSH BC
          JP (HL)           ; saut VecNum

RETOUR
          POP BC
          POP HL
          POP DE
          POP AF
          EX AF,AF
MRET1     JP C,0000
MRET2     JP 0000
;
TABINT    DW INT1
          DW INT2
          DW INT3
          DW INT4
          DW INT5
          DW INT6
          DW INT2

; INT1 début de rafraichissement écran. Fin de VBL passée.

INT1
          LD BC,#7F9E
          OUT (C),C
          LD BC,#BC0C        ; changement adresse Reg0C Reg0D
          OUT (C),C
          LD BC,#BD30        ; en #C000
          OUT (C),C
          LD BC,#BC0D
          OUT (C),C
          LD BC,#BD00
          OUT (C),C

          LD BC,#BC04        ; Nb ligne moitié écran Reg04
          OUT (C),C
          LD BC,#BD00+18
          OUT (C),C

          LD BC,#BC07        ; Overflow du Reg 7
          OUT (C),C
          LD BC,#BD00+255
          OUT (C),C

          LD BC,#BC06        ; Nb de caractères affichès
          OUT (C),C
          LD BC,#BD00+19
          OUT (C),C
          RET

; INT2 1/300éme de sec est passé depuis INT1

INT2
          RET

; INT3 Ca fait 2/300ème...

INT3
          LD BC,#BC0C      ; Bufferisation Offset pour écran suiv
          OUT (C),C
          LD BC,#BD20      ; en #8000
          OUT (C),C
          LD BC,#BC00
          OUT (C),C
          LD BC,#BD00
          OUT (C),C
          RET


; INT4 nous arrivons à 3/300ème soit 1/100. ce qui est la moitié
;       de 1/50ème donc au milieu de l'écran

INT4
        RET

; INT5 4/300ème

INT5
        RET

; INT6 5/300émé sont passés depuis INT1..Il reste 1/300émé de sec

INT6
          LD BC,#BC07
          OUT (C),C
          LD BC,#BD00+18     ; resynchronisation écran
          OUT (C),C
          RET
```
