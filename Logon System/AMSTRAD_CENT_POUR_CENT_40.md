GÉREZ VOS BANKS... DE MÉMOIRE
=============================

Avec quoi vais-je bien pouvoir surchauffer vos petits neurones assoiffés ce mois-ci ? La Rupture ligne à ligne ? Les SplitRasters ? La connexion de l'Asic ? La Connexion des Rams sur le CPC... Voilà t'y que c'est une bonne idée, ça! Je sais que ce sujet a déjà été abordé dans des numéros précédents mais je pense que j'apporte aujourd'hui quelques détails supplémentaires...

Comme vous le savez tous (mais si, vous le saviez), le CPC dispose d'un circuit surnommé le Gate Array (qui n'est pas un nom spécifique mais le nom d'un type de circuit spécifique construit spécialement pour l'ordinateur — dédié, quoi... —), ce qui n'est pas le cas pour le reste des circuits de l'Amstrad qu'on retrouve dans de nombreux autres ordinateurs). Le G.A. (ou IC116) donc, permet de gérer pas mal de choses intéressantes comme la palette de couleurs, le mode graphique, le contrôle sur le compteur d'interruption (son timer), les Rom (leur Connexion /Déconnexion), la connexion de la page d'Entrée/Sortie de l'Asic sur le CPC + (Rhaaaa, je l'ai dit...), et la connexion des ram supplémentaires pour les heureux possesseurs de 6128 ou d'extension Dk'Tronic. Il faut noter que les cartes Vortex Ram utilisent le même registre G.A. de sélection Ram mais de façon différente, d'où sa totale incompatibilité avec Dk'Tronic...

LES EXTENSIONS DE MEMOIRE
-------------------------

Sachez cependant que les cartes d'extensions Dk'Tronic sont compatibles totalement avec les connexions mémoires prévues sur le 6128. Ainsi, un 464 (ou 664) avec une extension Dk'Tronic 64 K se comportera de la même façon qu'un 6128 au niveau des Ram et pourra donc supporter le CPM PLUS. C'est cette totale compatibilité de la Dk'Tronic qui l'a privilégiée par rapport à l'extension Vortex, que peu de logiciels reconnaissent !

LE G.A. ET LES RAMS
-------------------

Pour en revenir au G.A., celui-ci dispose de 4 fonctions principales dont la
majorité a déjà dû être décrite par Fred (voir le Gate Array, Cent Pour Cent n°35). Je me bornerais donc à rappeler le fonctionnement du registre contrôlant les Ram. Voici où et comment on donne le numéro de la fonction que l'on désire:
Capito ? OK, on continue...
Nous disposons donc d'une valeur sur 5 bits à envoyer au G.A. Sur un CPC 6128 ou avec une extension Dk'Tronic 64 K, on peut décomposer les 5 bits utiles en 2 parties. Les bits 0 et 1 pour la sélection d'un bloc. Nous verrons les 3 autres bits (2, 3 et 4) plus tard...
Avant toute chose, et pour ceux à qui l'idée n'avait pas traversé l'esprit, le Z80A reste un processeur 8 bits et dispose d'un espace d'adressage de 2 puissance 16 octets, soit 65536 cases bien rangées, ou encore 4 x 16384 ... ce qui est équivalent à 4 x #4000 (Hexadécimal). Mais où veut-il en venir? (NDLR : Oui, où ??) A ceci : Tout comme les Roms qui utilisent le principe de « Swapping », plusieurs plans de mémoire sont mappés (localisés...) aux mêmes adresses et leur commutation dépend... du G.A... bien sûr ! Ah là là Poum, faut tout te dire !

LES PLANS DE MEMOIRE
--------------------

Les plans de mémoire commutés sont généralement de 16 K, ce qui explique qu'on raisonne très souvent par blocs de 16 K sur CPC (même pour les pages VidéoRam). Donc, je reprends... Notre mémoire de 64 K peut donc être divisée en 4 blocs de 16 K. Les bits 0 et 1 permettent d'indiquer le numéro de bloc... La sélection d'un bloc aura pour effet de le placer à partir de l'adresse #4000 jusqu'à #7FFF. A quoi servent les bits 2, 3 et 4? Eh bien tout simplement à indiquer le numéro de la page de 64 K disponible. Sur un 6128 (ou Extension 64 K), on considère que les 64 K supplémentaires représentent la deuxième page de 64 K de la mémoire totale. Pour accéder à ces « extra-pages », il est donc nécessaire de préciser le numéro de page (64 K) dans les bits 2, 3 et 4... Pour accéder à la deuxième page de 64 K, il est donc nécessaire de placer 2-1=1 (on raisonne par rapport à 0) dans les bits 4, 3, 2 (soit 001 en binaire). Si vous êtes l'heureux possesseur d'une extension Dk'Tronic de plus de 64 K, et que celle-ci fonctionne normalement (le rafraîchissement des Ram dynamiques laissant malheureusement à désirer... le contenu de ces Rams est assez « volatile »), alors vous pouvez sélectionner le reste des pages 64 K disponibles grâce aux bits 4, 3, 2. Ce qui nous offre la possibilité d'avoir 8 pages de 64 K en ligne sur le CPC, soit 8 x 64 K = 512 K... un demi-mégaoctet sur CPC...
Il faut noter que même s'il est écrit 256 K ou 512 K sur votre extension, celle-ci contient une page de 64 k en moins que la valeur annoncée, puisque la page 0 est prévue pour la Ram prin- cipale, disponible sur tous les CPC !I Vous payez donc, non pas 512 K, mais 512-64, soit 448 K... Intéressant, non ?

CONNEXION DES BLOCS
-------------------

En parlant de la page 0, que se passe-til lorsqu'on sélectionne un bloc sur celle-ci (donc, ils ne me payent pas !). C'est une bonne manière de découvrir « The Demo », non ?

L'UTILITAIRE M-PACK
-------------------

Et pour finir, je vous livre en pâture un gestionnaire de mémoire que j'ai écrit il y a fort longtemps et qui vous rendra quelques services : Memory-Pack. Ce programme ajoutera quelques RSX au Basic pour permettre une gestion plus aisée de la deuxième page de 64 K de votre CPC.
N'hésitez pas à nous écrire pour nous insulter (NDRobby : genre Gogol System, hé 1 hé ! hé I) ou plus simplement pour nous poser vos questions... Au mois prochain

Longshot
Logon System 91

* Diagrams not replicated yet

```
10 REM LOGON SYSTEM / AMSTRAD 100 % / MEMORY-PACK V1.0
20 REM
30 REM PROGRAMME GENERANT MPACK.BIN
40 REM
50 REM A EXECUTER UNE SEULE FOIS 11
60 REM
70 REM PLACEZ VOUS EN MODE 2 CA IRA MIEUX....
80 REM
100 MODE 2:BORDER O:INK 0,0:INK 1,26
110 PRINT "Generation MPACK.BIN"
120 RESTORE:ADR=&A350
130 FOR I=0 TO 40
135 S=0
140 FOR J=0 TO 15
150 READ A$:A=VAL("&"+A$):S=S+A
160 POKE ADR,A:ADR=ADR+1
170 NEXT J
i80 READ CHK
185 LOCATE 1,3:PRINT HEX$(40-I,2)
190 IF CHK<>S THEN PRINT:PRINT "Erreur Ligne ";2000+(I*10):END
200 NEXT I
210 SAVE "MPACK.BIN",B,&A350,&28A,&1234
220 PRINT:PRINT "MPACK.BIN Sauve !"
230 END
500 P=O:FOR I=&A350 TO &A350+&28A STEP 16:PRINT 2000+(P*10);"DATA        ";:P=P+1:S=0:FOR K=0 TO 15:PRINT HEX$(PEEK(I+K),2);",";:S=S+PEEK(I+K):NEXT K:PRINT S:NEXT I
1997 REM
1998 REM OULA LA LA ! TOUT CA A TAPER ?
1999 REM
2000 DATA 01,BF,A3,21,24,A4,CD,D1,BC,01,CA,A3,21,28,A4,CD, 1998
2010 DATA D1,BC,01,D5,A3,21,2C,A4,CD,D1,BC,01,E0,A3,21,30, 2086
2020 DATA A4,CD,DI,BC,01,EC,A3,21,34,A4,CD,D1,BC,01,F8,A3, 2429
2030 DATA 21,38,A4,CD,D1,BC,01,03,A4,21,3C,A4,CD,D1,BC,01, 1883
2040 DATA OE,A4,21,40,A4,CD,D1,BC,01,19,A4,21,44,A4,CD,D1, 1910
2050 DATA BC,06,04,78,F5,C5,CD,E6,A4,21,00,40,11,01,40,01, 1539
2060 DATA FF,3F,AF,77,ED,B0,C1,F1,3C,10,E9,AF,C3,E6,A4,C4, 2728
2070 DATA A3,C3,48,A4,4D,53,41,56,C5,00,CF,A3,C3,99,A4,4D, 2061
2080 DATA 4C,4F,41,C4,00,DA,A3,C3,BC,A4,4D,4C,44,49,D2,00, 1848
2090 DATA E5,A3,C3,F1,A4,4D,42,53,41,56,C5,00,F1,A3,C3,1B, 2192
2100 DATA A5,4D,42,4C,4F,41,C4,00,FD,A3,C3,42,A5,4D,42,52, 1791
2110 DATA 55,CE,00,08,A4,C3,4B,A5,4D,50,45,45,CB,00,13,A4, 1579
2120 DATA C3,78,A5,4D,50,4F,4B,C5,00,1E,A4,C3,87,A5,4D,44, 1822
2130 DATA 55,4D,D0,00,00,00,00,00,00,00,00,00,00,00,00,00, 370
2140 DATA 00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00, 0
2150 DATA 00,00,00,00,00,00,00,00,CD,86,A4,C5,D5,E5,CD,6B, 1454
2160 DATA A4,E1,AF,CD,E6,A4,7E,F5,78,CD,E6,A4,F1,12,D1,C1, 2914
2170 DATA 23,13,OB,78,B1,20,E4,AF,C3,E6,A4,06,04,3E,3F,BA, 1707
2180 DATA 30,05,C6,40,04,18,F8,DE,3F,EB,57,1E,00,AF,ED,52, 1722
2190 DATA 11,00,40,19,EB,C9,DD,46,01,DD,4E,00,DD,56,03,DD, 1664
2200 DATA 5E,02,DD,66,05,DD,6E,04,C9,CD,86,A4,C5,D5,E5,CD, 2307
2210 DATA 6B,A4,E1,78,CD,E6,A4,1A,F5,AF,CD,E6,A4,F1,77,D1, 2829
2220 DATA C1,13,23,OB,78,B1,20,E4,AF,C3,E6,A4,CD,86,A4,EB, 2317
2230 DATA C5,D5,E5,CD,DD,A4,E1,E5,F5,EB,CD,DD,A4,F1,12,E1, 3237
2240 DATA D1,C1,23,13,OB,78,B1,20,E7,AF,C3,E6,A4,CD,6B,A4, 2267
2250 DATA 78,CD,E6,A4,1A,C9,F6,CO,F3,C5,06,7F,ED,79,C1,FB, 2759
2260 DATA C9,21,5E,AE,11,00,01,D5,01,12,00,ED,B0,21,6B,O1, 1306
2270 DATA 3E,FF,77,2A,66,AE,11,6F,01,AF,ED,52,01,80,00,09, 1515
2280 DATA E5,22,6C,01,Ci,D1,D5,E1,C3,4B,A4,3E,04,CD,E6,A4, 2311
2290 DATA ED,4B,6C,41,3A,6B,41,FE,FF,CO,AF,CD,E6,A4,21,00, 2223
2300 DATA 01,54,5D,CD,9C,A4,21,00,01,11,5E,AE,01,12,00,ED, 1278
2310 DATA B0,C9,CD,1B,A5,CD,00,B9,C3,78,EA,CD,86,A4,C5,D1, 2622
2320 DATA CD,DD,A4,F5,CD,D9,A4,3E,23,CD,5A,BB,F1,F5,CB,2F, 2736
2330 DATA CB,2F,CB,2F,CB,2F,CD,6A,A5,F1,E6,OF,F6,30,FE,3A, 2318
2340 DATA 38,02,CE,07,CD,5A,BB,C9,CD,86,A4,C5,CD,DD,A4,E1, 2469
2350 DATA EB,73,23,72,C3,D9,A4,CD,86,A4,C5,3E,02,CD,OE,BC, 2246
2360 DATA D1,06,10,CD,CD,A5,7A,CD,5D,A5,7B,CD,5D,A5,CD,CD, 2387
2370 DATA A5,C5,D5,CD,DD,A4,D1,FE,20,30,02,3E,2E,CD,5A,BB, 2300
2380 DATA 3E,20,CD,5A,BB,13,C1,10,E8,CD,06,BB,FE,20,C2,D9, 2131
2390 DATA A4,3E,OD,CD,5A,BB,3E,OA,CD,5A,BB,18,C4,C5,06,05, 1703
2400 DATA 3E,20,CD,5A,BB,10,F9,C1,C9,00,00,00,00,00,00,00, 1235
```

```
10 REM MPACK
20 REM
30 REM
40 REM Copyright Serge QUERNE 1986
41 REM
50 REM
51 REM POUR UNE UTILISATION PROGRAMME
52 REM MEMORY &A34F:LOAD "MPACK.BIN":CALL &A350
53 REM APRES VOUS POUVEZ UTILISER LES RSX
54 REM
60 MEMORY &A34F
70 BORDER O:MODE 2:INK 0,0:INK 1,24: LOAD "MPACK.BIN":CALL &A350
80 PRINT SPC(35);"MPACK"
90 PRINT:PRINT"MPACK est un utilitaire developpe pour une supplementaires du 6128 meilleure utilisation des 64 Ko CPC."
100 PRINT:PRINT "En effet, le prog. BANKMAN est, il me semble, tres lourd a mettre en oeuvre!":GOSUB 5000
110 PRINT:PRINT SPC(25);"Nouvelles instructions!":PRINT:PRINT"-|MSAVE,<Adresse RAM>,<Adresse OVERLAY>,<Longueur>"
120 PRINT:PRINT"-|MLOAD,<Adresse RAM>,<Adresse OVERLAY>,<Longueur>":PRINT:PRINT "-|MLDIR,<Adresse OVERLAY 1>,<Adresse OVERLAY 2>,<Longueur>"
130 PRINT:PRINT "<Adresse RAM> represente une adresse entre 0000H habituellement utilisee (RAM et FFFFH concernant la RAM video,prog,ect..)"
140 PRINT:PRINT"<Adresse OVERLAY> represente une adresse entre 0000H et FFFFH concernant la RAM 64 K supplementaire.(Elle est entierement VIDE!)."
150 PRINT:PRINT "Exemple: |MSAVE,&C000,&B000,&4000 sauve la page ECRAN en overlay de B000H a F000H"
160 PRINT "         |MLDIR,&B000,0,&4000 recopie la page ecran sauvee en OVERLAY a l'adresse 0000 toujours en OVERLAY"
170 PRINT "         |MLOAD,&C000,0,&4000 charge la page ecran en OVERLAY de 0000 a 3FFFH dans la RAM active (ici la page ecran!)"
180 GOSUB 5000
190 PRINT:PRINT SPC(30);"Et en plus :"
200 PRINT:PRINT "- |MBSAVE Aucun parametre.Sauve un programme BASIC en overlay.ATTENTION les variables ne sont pas conservees!"
210 PRINT:PRINT "- |MBLOAD Aucun parametre.Recharge un programme BASIC de l'overlay!"
220 PRINT:PRINT "- |MBRUN Aucun parametre.Recharge un prog. BASIC et l'execute!"
230 GOSUB 5000
240 PRINT:PRINT SPC(30);"Mais aussi.."
250 PRINT:PRINT "- |MPEEK,<Adresse OVERLAY> Meme chose que PEEK mais en OVERLAY!"
260 PRINT:PRINT "- |MPOKE,<Adresse OVERLAY>,<Valeur> Place une valeur en OVERLAY.Si celle-ci est plus grande que 255 alors la commande agit comme un DOKE (16 bits)."
270 PRINT:PRINT "- |MDUMP,<Adresse OVERLAY> Dumpe la memoire OVERLAY!
280 GOSUB 5000
290 FOR I=0 TO 799:L=INT(RND(1)*100)+32:PRINT CHR$(L);:NEXT:PRINT:PRINT:PRINT "SAUVEGARDE ECRAN :|MSAVE,&C000,&6000,&4000":|MSAVE,&C000,&6000,&4000
300 PRINT "FINI!":GOSUB 5000
310 PRINT "RAPPEL DE LA PAGE ECRAN PAR |MLOAD,&C000,&6000,&4000":PRINT:PRINT "APPUYEZ SUR UNE TOUCHE POUE LOADER!":CALL &BB06:ÛNLOAD,&C000,&6000,&4000
320 GOSUB 5000
330 PRINT:PRINT "ET POUR TERMINER:SAUVEGARDE DU PROG BASIC EN OVERLAY PAR |MBSAVE:":|MBSAVE:PRINT:PRINT "DESTRUCTION EN RAM PAR NEW ET TAPEZ MBRUN POUR RAPPELER LE PROG DE DEMO!":NEW
4999 END
5000 X$="":WHILE X$="":X$=INKEY$:LOCATE 77,25:PRINT "==>":GOSUB 5100:LOCATE 77,25:PRINT " ":GOSUB 5100:WEND:SOUND 1,500:MODE 2:RETURN
5100 FOR K=0 TO 140:NEXT K:RETURN
```

```
; Envoi d'une valeur au Gate Array sur le port &7F00
  LD BC,#7F00
  LD A,%11000000
  OUT (C),A
```
