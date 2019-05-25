LOGON SYSTEM
============

Après vous avoir éclairé sur le CRTC, la rupture et les modes d'interruption du Z80, les Logon System ont décidé ce mois-ci de vous dévoiler quelques usages détournés de la pile. Bien sûr, nous parlerons ici d'assembleur et seuls les initiés du langage du microprocesseur de leur machine pourront bénéficier des enseignements de cette rubrique. Autre membre du célèbre groupe de démo-makers, vous allez maintenant faire connaissance avec Emmanuel alias Pict.

Je ne vais certainement pas apprendre aux lecteurs assidus de cette rubrique l'usage classique que l'on fait de la pile en assembleur. Vous pouvez d'ailleurs relire avec intérêt quelques initiations à l'assembleur de Sined le Barbare à ce sujet. Reprenez à cette effet votre collection d'Amstrad Cent Pour Cent ou bien commandez-les à l'adresse du magazine.
Il s'agit ici de découvrir quelques bidouilles intéressantes pour optimiser vos propres routines.

LE POINTEUR DE PILE
-------------------

Tout d'abord, sachez que thus les trois centièmes de seconde, le pointeur de pile SP (Stack Pointer pour ceux qui auraient oublié) est modifié par les interruptions pour stocker l'adresse contenue dans le PC (Program Counter). Il faut donc utiliser un DI pour que SP ne soit pas changé par les interruptions mais seulement par votre routine. Vous vous rappellerez peut-être que vous avez déjà vu une utilisation particulière de la pile dans Amstrad Cent Pour Cent grâce aux compères Poum et Franck Einstein (mais si, souvenezvous, le programme "CLS.CLS", florilège d'astuces de programmation absolument délirantes). Dans ce cas précis, on se servait de la pile pour afficher plusieurs fois les valeurs des registres 16 bits du Z-80 à la suite dans la mémoire-écran.

Exemple :

```
        ORG #9000
        ENT $
        LD (SAUVPIL+1),SP ; on sauve la pile car le systme s'en sert
        DI                ;on vire le systme pour qu'il ne modifie pas SP
        LD SP,#C000+80    ; Cf. Epl.
        LD HL,#FFFF
        LD B,40
PUSHL   PUSH HL
        DJNZ PUSHL
SAUVPIL LD SP,O           ; on rcupre la pile
        EI
        RET
```

Comme vous avez pu le voir, si vous avez exécuté ce petit programme, la première ligne de l'écran a été remplie d'octets à #FF. La pile stockant des registres 16 bits, le registre SP est donc décrémenté de 2 octets après un PUSH et incrémenté de 2 octets après un POP (on peut trafiquer ça avec INC SP et DEC SP mais c'est généralement peu intéressant).
De ce fait, on place la pile (après l'avoir sauvée, car le système en a besoin), en #C000+80 octets si on prend la largeur totale de l'écran, et on PUSH 40 fois HL chargé avec #FFFF. Si vous faites le calcul #C000+80-(40*2), on arrive bien sûr en #C000.
Quel est l'intérêt de cette routine ?
Comme tout autre routine avancée, elle sert à optimiser au maximum vos programmes. Cette routine peut vous servir lors de l'effaçage de sprites sur fond uni, par exemple. En effet, elle est environ 2 fois plus rapide qu'un programme ayant la même fonction sans utiliser la pile.

Exemple :

```
        LD HL,#C000
        LD C,#FF
        LD 0,80
POKFF   LD (HL),C
        INC HL
        DJNZ POKFF
        RET
```

La pile sert donc à stocker des données de 16 bits à une vitesse assez impressionnante, mais elle a une autre fonction plus intéressante : elle permet de prendre des données 16 bits ou 8 bits 2 par 2 dans une table. Supposons que vous ayez créé une table d'adresses-écran pour faire bouger un sprite selon un parcours précis. Une des manières classiques de prendre l'adresse est :

```
LD H,(IX+1) ;IX contient l'adresse de la table
LD L,(IX+0) ;On met l'adresse cran dans HL
INC IX
INC IX ;IX pointe sur l'adresse suivante
... ;votre routine d'affichage sprite.
```

Les interruptions bloquées et la pile sauvée, vous pouvez remplacer cette routine tout simplement par un POP HL (SP pointe sur la table d'adresses) et votre routine d'affichage.
Vous avez deux avantages incontestables avec la pile :
- POP HL prend huit fois moins de place que la routine précédente.
- POP HL est surtout 5 fois plus rapide

Remarque : cet exemple n'est pas le meilleur ; la saisie des adresses-écran prenant relativement moins de temps que l'affichage du sprite quelle que soit sa taille, il serait plus intéressant de se servir de la pile pour prendre les données du sprite plutôt que son adresse (voir la routine de sprites de Thibault dans la rubrique « Arcade » du mois de décembre - n 32, p. 72 et 73 -, qui peut d'ailleurs être encore optimisée !

PETIT CONCOURS
--------------

Je vous signale que les routines de scrolling en vague les plus rapides que j'ai créées utilisent la pile. Ainsi, ma routine de scrolling en vague fixe se sert de la pile pour prendre les adresses-écran tandis que ma routine de scrolling en vague mouvante l'utilise pour prendre les données du scroll. A ce propos, je vous propose un petit concours où vous ne gagnerez rien que notre estime et la présentation de votre routine dans cette rubrique.

En partant de tout ce que je vous ai dit sur la rapidité de lecture et d'écriture de la pile, comment faire un scrolling (en vague, horizontal, bref comme vous le voulez) en se servant de la pile, bien sûr, mais aussi des autres registres (même les registres secondaires accessibles par EXX si vous êtes courageux !). Les meilleures routines seront donc publiées et commentées.

LE TEMPS MACHINE
----------------

Bien, maintenant nous allons aborder la notion de temps machine, ou comment savoir le temps d'exécution de votre routine.
Donc, chaque instruction du Z-80 met un certain temps à être exécutée. Ce temps est exprimé en cycles machine. Il y a 2 sortes de cycles machine : les T states et les M-states, mais cela n'est guère important pour nous, car nous allons exprimer en NOP le temps machine. Le NOP est l'une des instructions les plus courtes du Z-80 avec les instructions du genre « LD registre simple, registre simple » ou encore « XOR registre simple »... Par contre, les instructions utilisant les registres d'index IX et IY sont d'une lenteur effroyable (ce type d'instructions n'est que rarement utilisé dans nos démos car il consomme beaucoup trop de NOP en temps machine). Je ne saurais que trop vous conseiller l'excellent livre de Rodnay Zaks sur le Z80 où tous les temps machine de chaque instruction du microprocesseur sont répertoriés.

MESURER LE TEMPS MACHINE
------------------------

Il existe une autre facon de mesurer le temps machine en comptant le nombre de NOP pris par instruction. On peut aussi, si l'on est sûr que sa routine prend moins d'un cinquantième de seconde, changer la couleur d'une encre au début de celle-ci puis la changer à nouveau à la fin de la routine. Par exemple, je désire savoir combien de temps le Z-80 mettra à faire une boucle vide :

Vous obtenez ainsi le temps mis par le Z-80 à effectuer la routine exprimée en ligne d'écran (l'espace qui est en rouge). Enfin, si vous avez beaucoup de mémoire disponible, évitez les DJNZ ou instructions dans ce genre et recopiez votre routine le nombre de fois que vous voulez qu'elle soit effectuée. Inutile de dire que toutes les routines de cet article sont alors optimisables : remplacez BOUCLE DEC a ;JP NZ,BOUCLE par DEC a..DEC a..DEC a..., 255 fois de suite (faites DEFS 255,#3D sur votre assembleur) et constatez la différence... Il y aussi la solution mitigée : on laisse quand même une boucle mais qui sera exécutée moins de fois car les instructions seront répétées à l'intérieur de cette boucle ; cette dernière méthode gaspille moins d'octets que l'autre mais prend évidemment plus de temps machine...
Sur ce, je vous laisse en vous souhaitant « une bonne prise de téte sur plein de routines bétons ». CALL 0!
Je vois rappelle que toutes les démos réalisées par vous, lecteurs, sont les bienvenues. Vous pouvez nous les faire parvenir a l'addresse du magazine.
Pict

```
          DI
          LD HL,#C9FB
          LD (#38),HL ;Classique
          EI
START     LD B,#F5    ;On attend la Synchro VBL
VSYNC     IN a,(c)
          RRA
          JP NC,VSYNC
          EI
          HALT
          HALT        ;On attend un peu...
          LD bc,#7F10 ;Gate Array pointe sur Borde.
          OUT (c),c   ;Valide
          LD c,#4C    ;Couleur Rouge
          OUT (c),c
          LD a,#FF    ;Routine tester
BOUCLE    DEC a
          JP NZ,BOUCLE
          LD bc,#7F10
          OUT (c),c
          LD c,#54    ;Couleur Noire
          OUT (c),c
          JP START
```
