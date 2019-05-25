LOGON SYSTEM
============

Serge (alias Longshot) a créé le groupe des Démo-makers le plus coté en France sur CPC : les Logon System. Dorénavant, vous le retrouverez chaque mois dans cette rubrique High-tech, il vous dévoilera les secrets des écrans overscan, des scrollings hardware, des rasters et autres astuces techniques, le tout avec pour seul outil un bon Assembleur Z80.

Roulement de tambours, voici Longshot :
"Longshot from Logon System presents... (Poum : Du frannnçaiiis !!!)...
OK ! OK ! T'énerve pas ! Bon, par quoi je commence ? Pour tout dire. je vais essayer d'animer (je dis bien essayer) cette rubrique concernant la programmation technique de notre machine préférée. J'ai nommé le CPC (tout en majuscules, s'il vous plaît !)".

PROGRAMMATION TECHNIQUE
-----------------------

Autant prévenir tout de suite ceux à qui l'Assembleur apparaît encore comme une langue barbare que ces lignes vont regorger de codes mnémoniques et de termes étranges...
Donc pour être plus clair, si le Basic est votre seul adage, allez bouffer ailleurs votre fromage, il y a une rubrique "Basic Perfectionnement" fort intéressante un peu plus loin...
Quel va donc être mon but ?
Eh bien. pour commencer, remplir vos chères têtes blondes de techniques vidéo comme la Rupture (c'est comme ça que je l'ai appelée) qui consiste, comme tout le monde le sait (non?), à modifier l'adresse du compteur vidéo en cours de balayage... Ça y est, y'en a déjà dix au fond qui savent pas ce qu'est un compteur vidéo et quarante autres qui se demandent ce que balayer a à voir avec leur CPC ! Je vais donc de ce pas, et pour pouvoir m'exprimer librement, ouvrir quelques chapitres.

LE BALAYAGE ET LE CRTC 6845
---------------------------

Eh bien, chers enfants, le balayage peut être défini comme le parcours utilisé par le canon à électrons (clans votre moniteur, si, si !) lorsqu'il genère une image...
Mais quand ce balayage députe-t-il ?
Excellente question ! Merci de la poser. Lee balayage se reproduit à une fréquence de 50 Hz, ce qui nous donne 50 rasters par seconde, soit un toutes les 0.02 secondes.
Mais qui s'occupe de la vidéo et du raster? Tout simplement un composant nommé CRTC 6845....

LE CRTC 6845, KESAKO ?
----------------------

Pourquoi CRTC ? Pour "Cathode Ray Tube Controller", qui est, comme tout le monde l'a traduit, le contrôleur de voire cher canon à électrons. Le 6845 est un circuit qui a t'ait ses preuves puisqu'il a été utilisé dans ale nombreux micro-ordinateurs et équipe notamment tous les compatible PC (actuellement, même les CRTC des cartes graphiques EGA et VGA émulent le mode 6845, ce qui n'est pas peu dire de son utilisation).

Ce composant est tellement réputé qu'il a été copié, recopié et encore copié ! De plus, Motorola, qui en est le constructeur d'origine, s'est l'ait un malin plaisir d'éditer plusieurs versions de son 6845. C'est pourquoi tous les CPC n'ont pas le même CRT et ne peuvent donc pas. malheureusement, être entièrement compatibles. Qui aurait cm que les CPC seraient touchés avec des problèmes de compatibilité vidéo ?

Pour ceux qui ont un Atari ST et qui ricanent bêtement en lisant ces lignes, ils doivent savoir qu'Atari a exactement lait la même c...e avec le Shifter (Circuit Vidéo du ST). Je reprends donc : tous les CRT ne sont pas identiques et seulement quelques personnes s'en sont aperçu dans les jeunes années du CPC, comme Rémy Herbulot, auteur aie Crafton & Xunk, qui avait réalisé un Scrolling hard Horizontal vers la gauche pour effectuer les changements de tableaux.

Quelle ne dut pas être sa surprise quand, lançant sa routine de scrolling sur un autre CPC, il vit son jeu se planter. Celui-ci attendait inexorablement le début d'un balayage vidéo que le CRTC ne générait plus !

TESTEZ VOTRE CRTC
-----------------

Je vais tout d'abord vous permettre ale tester le type de votre CRTC. Mais expliquons avant tout due ce circuit dispose de 19 registres internes (quelques-uns ne sont pas utilisés sur CPC comme la gestion du stylo optique). Sur ces 19 registres, l'un d'eux est utilisé comme registre d'index pour sélectionner les 18 autres.

Je ne décrirai pas tous les registres du CRTC ici, de nombreux ouvrages le font très bien. Et j'ai promis à Pict (un autre membre du Logon System) de lui laisser cette tâche. Je ne décrirai donc que les registres que je vais utiliser. Sachons toutefois que la sélection du registre d'index du CRTC se l'ait via le port situé en &BC00 et que l'affectation de la valeur pour le registre choisi se l'ait en &BD00.

Deux autres ports existent aussi pour accéder au CRTC en lecture : un en &BE00 et un autre en &BFOO. La fonction de ce dernier devrait être de Pouvoir lire le contenu d'un registre sélectionné à l'aide du port I3C00. mais ça ne marche que sur certains CRTC. Cela nous permet d'ores et déjà de disposer d'une méthode de test pour les différencier. Ce port, &BFO0, n'est opérationnel que sur le CRTC ale type H6845SP, qui n'est pas un circuit Motorola, niais un circuit japonais émulant un vrai 6845.
C'est d'ailleurs ce type de CRTC qui a été écoulé dans le nouveau circuit vidéo des 464 et 6128 Plus, nais c'est une autre histoire...
Je disais donc qu'il est possible, sur un H6845SP, de lire certains registres (lui CRTC, niais seulement les registres 12,14,15,16,17.

Aussi..Test du CRTC :

```
LD BC,#BC00+12  ; Sélection du registre 12
OUT (C),C       ; Envoi I/O
LD BC,#BF00     ; Port Input
IN A,(C)        ; Retirer valeur
```

Certains CPC rendront dans A la valeur (lu registre 12, qui est. en l'occurrence, le poids fort de l'offset vidéo. D'autres CPC, par contre. répondront 0. no comment ! Je vous ai parlé des registres d'ollset 12 et 13 du CRTC. Ces registres permettent de gérer l'adresse de début d'affichage, la taille du buffer vidéo et le Blok vidéoram utilisé.

Les Bits 0..9
-------------

Valeur ale 0 a 1023 qui indiquent le Mot (Word, donc deux octets, t'as compris, Robby ?) à partir duquel la vidéoram commence. C'e qui nous amène à dire que l'offset ne Peut être changé clue sur les 2048 premiers octets ale la Ram vidéo... Et, comme par hasard, ces 2048 premiers octets sont TOUJOURS visualisés comme la première ligne caractère quel que soit le format de l'écran ou le nombre de lignes par caractères (oui, ça aussi, ça se règle). Je le dis donc tout haut : il est impossible de Paire commencer l'écran autrement que sur la première ligne d'un caractère !
C'est d'ailleurs pour cette raison que décaler l'offset de la largeur d'une ligne (c'est-à-dire : nombre octets par ligne / 2 (pour avoir le nombre de mots par ligne) provoque un scrolling vertical de la hauteur d'un caractère.
Qui peut. je le redis, être modifiée. De plus ces 2048 octets bouclent, c'est-àdire que les octets 2046 et 2047 sont immédiatement suivi par les octets 0000 et 0001.

Les bits 10..11: Taille du Buffer Vidéo
0 0 16 Kb Compteur d'adresse modulo 16 Kb
0 1 Idem
1 0 Idem
1 1 32 Kb Le Compteur d'adresse est étendu pour la joie des petits et des grands...

Les bits 12.13 : Base de la vidéoram parmi un des 4 blocs de la Ram principale.
0 0 0000..3FFF
0 1 4000..7FFF
1 0 8000..BFFF
1 1 C000..FFFF . Valeur par défaut pour le CrocoSysteme.

NB : Eh oui, c'est triste, mais il est impossible d'utiliser la Ram supplémentaire du 6128 (ou les Rams d'une Extension mémoire) comme Ram Vidéo. C'est toujours la Ram principale dui est décodée par le contrôleur vidéo. D'ailleurs je parlerais des
commutations Ram la prochaine fois, ce qui nous permettra de mettre à jour une technique aies "Reset Crackers".
Sur ce, je termine ici cette rubrique et je vous donne rendez-vous le mois prochain, si vous arrivez encore à me supporter.

CTRL SHIFT ESC!
Longshot

PS : Après ce relatif démarrage en douceur, nous espèrons que cette rubrique suscitera de nombreuses réactions de votre hart. Aussi, n'hésitez pas a nous faire parvenir vos questions et propositions.