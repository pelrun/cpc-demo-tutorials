LE PPI 8255A
============

Le circuit le plus tarabiscoté de notre cher CPC est bien le M. Devant le manque d'information concernant ce petit bijou indispensable à tout programmeur, Longshot met un terme à notre ignorance. En voiture Simone !

Beaucoup d'entre vous nous ont demandé: « Le PPI, késako ? »
Eh bien chers enfants, c'est un des circuits de votre CPC dont nous n'avions jamais parlé auparavant...
Le PPI 8255A (PPI pour Programmable Peripheral Interface), comme son nom l'indique (Hem !), permet de gérer les entrées-sorties de votre Amstrad.

En gros, sur votre CPC, le PPI s'occupe:
Du AY-3-8912, qui génère les sons.
Du clavier (à travers le AY...).
Du lecteur de cassettes.
De l'imprimante.
Du signal VBL émis parle CRTC.
D'informations diverses câblées.

Le PPI communique avec l'extérieur (et donc avec les circuits périphériques) grâce à 3 ports d'entrées-sorties qui portent respectivement les noms A, B et C. Il peut travailler avec ces 3 ports de plusieurs manières différentes et il est possible de préciser le rôle de chaque port à travers un registre appelé « registre de contrôle ».
Celui-ci permet, entre autres, de déterminer le mode de fonctionnement (Il en existe 3) et le sens de communication des ports A,B,C.

Sur CPC, nous verrons quelles valeurs sont affectées au « registre de contrôle » pour que le PPI fonctionne selon le câblage que les ingénieurs Amstrad ont effectué sur le circuit.
A priori, un port du PPI peut fonctionner en entrée ou en sortie mais il était plus commode de regrouper sur un port toutes les sorties (cf. port C) et sur un autre port toutes les entrées (cf. port B).

De même, le PPI dispose de 3 modes de travail qui sont le Mode 0, 1 et 2. Sur CPC, et à ma connaissance, seul le mode 0 est utilisé.
Celui-ci permet de considérer les 3 ports A,B,C comme un « champ de 24 canaux d'I/O » (8 bits x 3).

Mais détaillons un peu tous les modes du PPI (après tout, c'est ça, la vulgarisation technique...). Ces modes permettent de configurer l'état des 3 ports.

LE MODE 0
---------

Comme je vous l'ai dit plus haut, nous disposons de 3 ports A, B et C. Ce que j'ai oublié de vous dire (ben oui, pas tout à la fois !), c'est que le port C peut être considéré comme 2 registres 4 bits. Cela nous fait au total 2 registres 8 bits (A & B) et 2 registres 4 bits (C1 & C2). Bref, en tout et pour tout, nous disposons en Mode 0 de 24 lignes d'entrées-sorties (8+8+4+4). Les sens de communication de ces 4 registres étant indépendants et définissables via le « registre de contrôle ».

LE MODE 1 (OU ENCORE LES I/O CONTROLEES)
----------------------------------------

Ce mode permet le transfert de données de/vers un port (A ou B) mais avec un contrôle des données reçues via un des 2 registres 4 bits (port C bits 4 à 7 pour A » port C bits 0 à 3 pour B). Jetez un coup d'oeil au schéma n"1.

LE MODE 2 (OU MODE BIDIRECTIONNEL)
----------------------------------

Ce mode est uniquement disponible avec le port A du PPI. Il a la particularité de permettre un fonctionnement bidirectionnel du bus de données, c'est-a-dire que le port A peut émettre ET recevoir des données sans que le sens de communication ait à être défini à chaque fois (comme c'est le cas avec le mode 1). De même que pour le mode 1, la gestion du contrôle de transition des données est effectuée via le port C, et cela sur 5 bits. Louchez donc vers le schéma n°2.

Voilà ! Le tour d'horizon des modes du PPI est fait. Le PPI peut aussi, via le « registre de contrôle » (eh oui, toujours lui !), contrôler l'état de chaque bit du port C individuellement... II est maintenant l'heure de le definir enfin :

LE REGISTRE DE CONTROLE
-----------------------

Il est situé sur CPC à l'adresse d'I/0 #F700. Le bit 7 de ce registre contrôle la fonction des bits restants (bits 0 à 6).

1er cas. Bit 7=0
Dans ce cas, le registre permet de contrôler l'état de chaque bit du port C. Il est donc possible d'affecter le port C bit à bit, ce qui peut s'avérer très pratique lorsqu'il est en sortie. Et si vous regardiez le schéma n°3??

2e cas. Bit 7=1
Dans ce cas, les bits 0 à 6 permettent de définir le mode de travail pour le port A et le port B (notez que le Mode 2 n'est disponible que pour le port A) et le sens de communication pour chacun des 4 ports (A, B, C1 et C2). Le schéma n°4 vous semblera sans doute plus clair...

Ça va ? Vous êtes toujours là ?I? OK, je continue... Je vous ai dit tout à l'heure (si,si !), que le CPC fonctionne en Mode 0 (pour ceux qui pensent que je parle du mode graphique : fermez les yeux, tournez quelques pages, rouvrez les yeux...). Nous allons voir quelles valeurs on trouve couramment dans le registre de contrôle pour un CPC en pleine forme et pas planté du tout... Tout d'abord, le registre de contrôle est très rarement utilisé en contrôle bit à bit du port C... C'est pourquoi le bit 7 de ce registre est toujours à 1. Les ports A et B sont en mode 0, ce qui nous permet de définir les bits 5 et 6 à 0 pour A et le bit 2 à 0 pour B.II reste maintenant à définir les sens de communication des 4 registres PPI IA,B,C1,C2). Ce sens est très lié aux périphériques connectés au PPI...

Sur CPC :
Le port A est utilisé en entrée ET en sortie, ce qui nous donne un bit 4 indéfini (0 ou 1)
Le port B est utilisé en entrée uniquement, ce qui nous donne le bit 1=1.
Le port C1 (Bit 0..3) est utilisé en sortie uniquement, ce qui nous donne le bit 0=0.
Le port C2 (Bit 4..7) est utilisé en sortie uniquement, ce qui nous donne le bit 3=0

En résumé, nous avons deux valeurs possibles pour le registre de contrôle en mode 0 sur CPC, cela en fonction du sens de communication du port A (#82 ou #92) Mais regardez donc le schéma n"5. Je précise que entrée pour un port signifie que le port reçoit des informations d'un périphérique et que sortie correspond à une émission de données vers un périphérique.
Venons-en à la partie la plus motivante de cet article PPI-esque avec le schéma n°6 (mon oeuvre...) qui décrit l'architecture Périphériques/PPI du CPC.
(Poum vient juste de s'évanouir en voyant le schéma...(NDPoum : Si ça peut te faire plaisir mon petit Sergio !)
Donc, après ce schéma intergalactique d'interfaçage entre les circuits du CPC et le PP)... je pense que tout doit vous paraître limpide.... Non ?!?

Bon, je pense que vous avez tous pigé comment lire et écrire dans certains des périphériques présentés. Prenons par exemple une lecture du port B. On configure tout d'abord le registre de contrôle pour placer B en entrée et C en sortie.

```
LD BC,#F782 ; Reg Contrôle
OUT (C),C
```

Cela étant fait, on peut lire le port B situé en #F500

```
LD B,#F5 ; Sélection port B
IN A,(C) ; Lecture dans Accu
```

A contient toutes les informations décrites sur le schéma 6.

Ainsi : On veut attendre que l'imprimante soit prête à recevoir des données:

```
      LD B,#F5
BUSY  IN A,(C)
      AND %01000000 ; Test Bit 6
      JR NZ,BUSY
```

On veut attendre le signal VBL du CRTC:

```
      LD B,#F5
VBL   IN A,(C)
      AND %00000001 ; Test Bit 0
      JR Z,VBL
```

Clairs, mes exemples, non ?

Bref, prenons maintenant le cas d'envoi de données vers un circuit, via le port C situé en #F600. On veut démarrer le lecteur de cassettes ? Il est nécessaire de placer le bit 4 du port C à 1. A moins de connaître l'état des autres bits du port C, il faut le lire avant pour conserver l'état des 7 autres bits. Pour cela, deux méthodes s'offrent à nous.
1) Utiliser la ire fonction du registre de contrôle pour positionner le port C bit à bit.
2) Lire le port C avant d'y écrire... La logique voudrait que l'on soit obligé de placer le port C en entrée pour pouvoir le lire.., cependant, les concepteurs du PPI ont équipé le circuit de buffers, qui retiennent la dernière valeur envoyée ou reçue par les ports PPI. Ces buffers ne sont pas sur tous les ports mais cela nous permet, dans notre exemple, de lire le port C avec un IN Z80 sans que le port C soit placé en lecture.

Aïe... A'ie... Aie... je le sentais venir... Je n'ai parlé que du lecteur de cassette pour l'écriture sur le port C en #F600. A quoi peuvent bien servir les bits 6 et 7 du port C ? Comme vous pouvez le constater sur le schéma 6, ces bits servent à gérer la communication entre le PPI et le AY-3-8912 pour que ce dernier sache transférer correctement les bonnes données au bon endroit (qui a dit : « C'est vague, ça ! »)
Je m'explique (comme dirait Digit): Contrairement au CRTC et à l'imprimante, qui ne communiquent avec l'extérieur que grâce au PPI (en fait, ils ont leur propre adresse d'I/0), le générateur sonore est entièrement dépendant du PPI.
Le circuit sonore (AY-3-8912...AY pour les intimes) dispose, tout comme le CRTC, d'un registre d'index et de plusieurs registres de données 8 bits (15 pour être précis). Le registre d'index permet de sélectionner un des 15 registres de données. La sélection de ces registres ne se fait pas sur des adresses I/O spéciales (comme sur le CRTC en BCOO/BDOO, je le répète): Cela aurait été trop simple (Arf!). C'est pourquoi tout a été câblé sur le PPI (Economie oblige..). Comment ça marche ?

Les données 8 bits de/ou/vers le AY sont placées sur le port A. La signification de ces valeurs pour le générateur sonore dépend des bits 6 & 7 du port C (pour information, ces bits portent respectivement le nom BDIR et BC1 en souvenir des papattes du AY, d'où les lignes proviennent...).
Comment faut-il procéder pour envoyer une valeur vers le AY ?
Il nous faut tout d'abord placer le port A en sortie (#F782).
Puis il faut remplir le registre d'index du AY avec le n° désiré (De 0 a 14).

Pour faire cela, il faut dire au AY que la donnée qui sera mise sur le port A est le reg. d'index. Il faut donc mettre BDIR & BC1 à 1 (cf. Schéma 6). Mise de BDIR & BC1 à 1 sur le port C.

```
LD BC,#F6C0
OUT (C),C
```

Sélection du registre d'index sur port A

```
LD BC,#F401 ; Sélection reg. 1
OUT (C),C
```

Validation de la donnée (BDIR=0 et BC1=0)

```
LD BC,#F600
OUT (C),C
```

Une fois que le registre est sélectionné, il faut écrire la donnée. Pour réaliser ça... mais oui... mais oui... BDIR et BC1, encore eux, prennent les valeurs 1 et 0 (cf. Schéma 6). Mise de BDIR=1 et BC1=0 sur le port C.

```
LD BC,#F680
OUT (C),C
```

Ecriture de la donnée pour le reg. sélectionné (port A)

```
LD BC,#F430 ; Ex Envoi #30 dans Reg 1
OUT (C),C
```

Et enfin validation de la donnée sur port A

```
LD BC,#F600
OUT (C),C
```

Et voila, la donnée est partie vers le générateur sonore. Notez que la validation est nécessaire car elle permet de valider la sélection effectuée (fonction désirée sur port Cet valeur sur port Al. Ainsi l'ordre des 2 premiers OUT n'a-til aucune importance car « rien ne part » sans validation ! Résumons l'envoi d'une valeur vers le AY via le PPI :

```
LD DE,#0130   ; D=Index E=Valeur
LD A,0        ; 0 de validation
LD BC,#F782
OUT (C),C
LD B,#F4      ; Valeur sur A (reg. D)
OUT (C),D
LD BC,#F6CO   ; C'est un index
OUT (C),C
OUT (C),A     ; On valide !
LD B,#F4
OUT (C),E     ; Valeur sur A (reg. E)
LD BC,#F680
OUT (C),C     ; C'est une donnée
OUT (C),A     ; On valide !
RET
```

Dans cet exemple, nous venons de faire : reg.1sonore=#30. Mon Dieu, que tout cela est long à expliquer ! (Aux autres Logon : vous êtes des lâches !)

Et pour terminer en beauté, si nous parlions du clavier. Accrochez vos ceintures. Cela va peut-être vous paraître étrange, mais c'est le générateur sonore qui «« gère » le clavier.
Ce n'est pas par sadomasochisme (parole de connaisseur), mais parce que le circuit AY-3-8912, peut, lui aussi, communiquer avec un périphérique via un port d'entrée-sortie.
Le port A d'I/O du AY (rien à voir avec le port A du PPI) est situé sur le dernier registre du AY (le n" 14). A priori, le port du AY peut fonctionner en entrée ou en sortie. En fonction du Bit 6 du Reg7 de l'AY, 0 pour entrée et 1 pour sortie.
Un clavier étant principalement destiné à fournir des informations (non ?), le reg.14 (j'appelerais le port A du AY comme ça pour éviter les confusions) est placé en entrée. La sortie pouvant servir au téléchargement, par exemple. Le clavier fonctionne selon un système de matrice et chaque touche correspond à un bit d'information. Or le reg.14 est un registre 8 bits et le CPC ne pouvait se contenter de 8 touches Bien que ce soit largement suffisant pour Rubi...

Il fallait donc que le reg.14 puisse recevoir plusieurs octets de la part du clavier. Pour tout dire, le clavier dispose de 10 octets qui correspondent à 10 lignes de la matrice clavier (ce qui nous fait un maximum de 10x8, soit 80 touches différentes gérables). La sélection du n° d'octet envoyé au AY par le clavier est faite par... le port C du PPI. Tout le monde est là ?
OK, donc, pour lire le clavier, il est nécessaire de placer le bit 6 du reg. 7 du AY à 0 pour que le AY reçoive les données du clavier via R14. Ensuite, il faut placer le port A du PPI en entrée pour que le PPI puisse lire R14 du AY...
Arrêtez de pleurer, j'ai pas fini Pratiquement, le positionnement du bit 6 se fait classiquement via une écriture dans le reg. 7 du AY. Attention toutefois car les bits 0 à 5 doivent être à 1 pour qu'il n'y ait pas de son... bref, vous remplacez DE par #073F dans l'exemple précédent !).
Il faut aussi pouvoir récupérer le reg.14 du AY via le port A du PPI. On place donc ce dernier en entrée via le registre de contrôle (on aura donc #F792). Venons-en maintenant à la sélection de la ligne clavier. Le n° de celle-ci est situé dans les bits 0.1.2.3 du port C (qui est positionné, oh ! miracle !, en écriture). Allons-y gaiement avec un exemple commenté !

```
LD D,0       ; Ligne clavier de 0 à 9
LD BC,#F782  ; A Sortie/C Sortie
OUT (C),C
LD BC,#F40E  ;Valeur 14 sur port A
OUT C),C
LD BC,#F6CO  ; C'est un index (R14)
OUT (C),C
LD BC,#F600  ; On valide!
OUT (C),C
LD BC,#F792    ; A Entrée/C Sortie
OUT (C),C
LD A,D         ; A ligne clavier
AND %00001111  ; Conserver Bits 0.3
OR %01000000   ; BDIR=O BC1=1
LD B,#F6
OUT (C),A
LD B,#F4       ; On lit le port A (PP))
IN A,(C)       ; Donc R14 (A du AY)
LD BC,#F782    ; A en Sortie
OUT (C),C
LD BC,#F600    ; Reset PPI Write
OUT (C),C
RET
```

Dans cet exemple, l'accu contient en sortie la valeur de la ligne 0 du clavier (voir schéma 6). Lorsque le bit correspondant à la touche est à 0, c'est qu'elle est enfoncée !
J'ai considéré dans les 2 derniers exemples que personne ne fait mumuse avec le lecteur de cassettes. Si c'est la cas, il vous faudra sauver préalablement les bits 4 et 5 du port C avant d'écrire dessus...

Voilà, ce long délire sur le PPI est terminé. J'espère que ça vous a plu... N'hésitez pas à nous poser vos questions... mais n'oubliez pas, The Demo is good for You 1 Je sais, ça n'a aucun rapport, mais bon ! A dans 2 mois ! Si Dieu le veut.

Longshot, Logon System 92

Schema N°1 : PPI Mode 1.
Port A           Port C
7 6 5 4 3 2 1 0  7 6 5 4

Schéma N°2 : PPI Mode 2.
Port A            Port C
7 6 5 4 3 2 1 0  (3 2 1 0) => Contrôle de Transmission

Schéma N°3 : PPI Registre de Contrôle.(#F700)
Contrôle du Port C bit à bit.

7 6 5 4 3 2 1 0
0 X X X d c b a

bit 0 (a):
1 : Mettre Bit à 1
0 : Mettre Bit à 0

Bits 3-1: Bit number

Schéma N°4 : PPI Registre de Contrôle.(#F700)
Fonction de Contrôle générale.
7 6 5 4 3 2 1 0
1 g f e d c b a

Bit 0 (a):
1:Entrée port C bits 0-3
0:Sortie port C bits 0-3

Bit 3 (d):
1:Entrée port C bits 4-7
0:Sortie port C bits 4-7

Port B
Bit 1 (b):
1: Entrée
0: Sortie
Bit 2 (c):
0: Mode 0
1: Mode 1

Port A
Bit 4 (d)
1: Entrée
0: Sortie
Bits 6-5 (gf):
00: Mode 0
01: Mode 1
10: Mode 2
11: Mode 2

Schéma N°5 : Les valeurs Cpc du registre de controle

7 6 5 4 3 2 1 0
1 0 0 0 0 0 1 0 = #82 sur port #F700
=> Port A en Sortie.

7 6 5 4 3 2 1 0
1 0 0 1 0 0 1 0 = #92 sur port #F700
=> Port A en Entrée.

* Diagram not transcribed yet
