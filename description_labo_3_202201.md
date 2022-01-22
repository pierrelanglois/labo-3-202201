-----------------------------------------------------------------------
<table>
<tr>
<td><img src="figures\Polytechnique_signature-RGB-gauche_FR.png" alt="Logo de Polytechnique Montréal"></td>
<td><h2>INF3500 - Conception et réalisation de systèmes numériques
<br><br>Hiver 2022
<br><br>Laboratoire #3
<br><br>Circuits séquentiels et chemins des données
</h2></td>
</tr>
</table>

----------------------------------------------------------------------------------------------

# Le cadenas à boutons

----------------------------------------------------------------------------------------------


À la fin de ce laboratoire, vous devrez être capable de :

- Concevoir un circuit séquentiel à partir d’une spécification. Donner un diagramme d’état, le code VHDL, le schéma du circuit et son implémentation résultante sur un FPGA. (B5)
    - Utiliser une horloge et un signal de réinitialisation
    - Utiliser des registres et des compteurs
    - Utiliser une machine à états
- Composer un banc d’essai pour stimuler un modèle VHDL d’un circuit séquentiel. Donner le chronogramme résultant de l’exécution d’un banc d’essai. (B4,B5)
    - Générer un signal d'horloge et un signal de réinitialisation
    - Générer des stimuli pour un circuit séquentiel
    - Calculer les sorties correspondantes aux stimuli
    - Réconcilier les problèmes de synchronisation
    - Utiliser des énoncés `assert` ou des conditions pour vérifier le module
- Implémenter un circuit séquentiel sur un FPGA et en vérifier le fonctionnement correct
    - Utiliser des interfaces comme des boutons et des affichages à LED
    - Constater et corriger les phénomènes de rebond des boutons et commutateurs

Ce laboratoire s'appuie sur le matériel suivant :
1. Les concepts couverts dans les laboratoires #1 et #2.
2. La matière des cours de la semaines 4 (Modélisation et vérification de circuits combinatoires) et 5 (Conception de chemins des données - surtout les diapositives 0502).

## Partie 0 : une machine à états de base

### Préparatifs

- Créez un répertoire `inf3500\labo-3` dans lequel vous mettrez tous les fichiers de ce laboratoire.
- Importez tous les fichiers du laboratoire à partir de l'entrepôt Git.
- Si vous utilisez Active-HDL :
    - Créez un espace de travail (workspace) et créez un projet (design).
    - Ajoutez les fichiers dans votre projet.
    - Compilez tous les fichiers.
    
Les fichiers suivants sont  fournis pour aider à contrôler les interfaces de la carte et faire l'implémentation dans le FPGA. Ne les modifiez pas.
- [utilitaires_inf3500_pkg.vhd](sources/utilitaires_inf3500_pkg.vhd) : pour regrouper un ensemble de fonctions utilitaires utiles pour les laboratoires du cours
- [generateur_horloge_precis.vhd](sources/generateur_horloge_precis.vhd) : pour générer une horloge à une fréquence désirée à partir de l'horloge de la carte
- [monopulseur.vhd](sources/monopulseur.vhd) : pour synchroniser les actions des humains avec l'horloge du système
- [top_labo_3.vhd](sources/top_labo_3.vhd) : pour regrouper tous les fichiers lors de l'implémentation
- xdc/basys_3_top.xdc, xdc/nexys_a7_50t_top.xdc et xdc/nexys_a7_100t_top.xdc : pour établir des liens entre des identificateurs et des pattes du FPGA
- [labo_3_synth_impl.tcl](synthese-implementation/labo_3_synth_impl.tcl) : pour regrouper les commandes à utiliser pour faire l'implémentation

### La machine à états et son code VHDL

Une machine à états de base vous est fournie dans le fichier [cadenas_labo_3.vhd](sources/cadenas_labo_3.vhd]). La machine à états de base correspond au diagramme d'états suivant.

![machine à états de base](figures/machine_etats_de_base.svg)

La machine a cinq états, E_00 à E_04 inclusivement. À chaque coup d'horloge, la machine avance d'un état si le bouton(0) a une valeur de '1'. Les sorties sont indiquées dans les états.

### Simulation    

Une ébauche de banc d'essai se trouve dans le fichier [cadenas_labo_3_tb.vhd](sources/cadenas_labo_3_tb.vhd). Faites la simulation de la machine à états de base à l'aide de ce banc d'essai.
- Choisissez le module `cadenas_labo_3_tb` comme `Top-Level`.
- Initialisez la simulation.
- Créez un chronogramme et glissez-y l'unité sous test `UUT`.
- Lancez la simulation, observez le chronogramme et observez les messages dans la console.
- Constatez qu'une valeur de '1' au bouton(0) résulte en une transition d'états lors de la prochaine transistion positive de l'horloge.
- Constatez que les sorties `ouvrir` et `alarme` varient en fonction de l'état actuel de la machaine.

Si vous utilisez un autre environnement, suivez des étapes similaires pour lancer la simulation et observer les résultats.

### Implémentation et programmation de la carte

Suivez les étapes suivantes pour faire la synthèse et l'implémentation du code sur votre carte :

1. Lancez une fenêtre d'invite de commande ("cmd" sous Windows) et naviguez au répertoire "\inf350\labo-3\synthese-implementation\".
2. De ce répertoire, lancez Vivado en mode script avec la commande 
`{repertoire-d-installation-d-Vivado}\bin\vivado -mode tcl` où {repertoire-d-installation-d-Vivado} est probablement C:\Xilinx\Vivado\2021.1 si votre système d'exploitation est Windows.
3. Dans la fenêtre, à l'invite de commande `Vivado%`, entrez les commandes contenues dans le fichier [labo_3_synth_impl.tcl](synthese-implementation/labo_3_synth_impl.tcl). Si votre carte n'est pas une Basys 3, vous devrez commenter certaines lignes et en dé-commenter d'autres qui correspondent à votre carte.

Observez le fonctionnement correct du système sur la carte.

Inspectez le contenu du fichier [top_labo_3.vhd](sources/top_labo_3.vhd) : 
- Le bouton du centre est connecté au port `reset`;
- Le port `boutons(3 downto 0)` est relié dans l'ordre aux boutons du bas, de gauche, d'en haut et de droite.
- Les boutons sont également reliés à des LED, directement et aussi par l'entremise du module de stabilisation et de synchronisation du fichier [monopulseur.vhd](sources/monopulseur.vhd). Observez que quand vous gardez un bouton pressé, observez qu'une LED reste allumée et qu'une autre LED ne reçoit qu'une seule courte impulsion.
- Les LED(1:0) sont reliées aux ports (alarme, ouvrir).
- L'affichage quadruple à 7 segments affiche les caractères placés sur le port `message`.

## Partie 1 : Ajout de fonctionnalités au contrôleur

Ajoutez les fonctionnalités suivantes au contrôleur. Modifiez et remettez le fichier [feux_circulation.vhd](sources/feux_circulation.vhd).

**Ne modifiez pas le nom du fichier, le nom de l'entité, la liste et le nom des ports, la liste et le nom des `generics`, ni le nom de l'architecture.**

### Partie 1a. Ajout d'une transition entre un feu rouge et le feu vert transversal d'une direction à une autre

Dans la version de base du contrôleur, le feu devient vert sur une artère dès qu'il devient rouge sur l'autre. Dans la réalité, il y a une pause pendant laquelle tous les feux sont rouges.

Ajoutez deux états pour inclure cette pause dans la séquence, avec une durée de _DUREE_ROUGE_PARTOUT_ secondes. Il faut un état avant que le feu passe au vert sur le CCSC et un état avant que le feu passe au vert sur l'AVDI. Ajouter les conditions dans le processus de la séquence des états et dans le processus des assignations aux feux.

### Partie 1b. Contrôle de la flèche pour tourner à gauche sur l'AVDI

Dans la version de base du contrôleur, les automobilistes qui circulent en direction ouest sur le CCSC ne peuvent jamais tourner à gauche sur l'AVDI. Modifiez le contrôleur pour ajouter cette fonctionnalité.

Afin que les automobilistes qui circulent en direction ouest sur le CCSC puissent tourner à gauche sur l'AVDI, les feux doivent d'abord être au rouge sur l'AVDI et sur le CCSC en direction est. Après une pause de _DUREE_ROUGE_PARTOUT_ secondes, la flèche-gauche-rouge peut s'éteindre, et la flèche-gauche-verte et la flèche-tout-droit peuvent alors être allumées. Plus tard, la flèche-gauche-verte doit s'éteindre et la flèche-gauche-rouge doit s'allumer. Après une pause, on peut allumer le feu vert sur le CCSC en direction est. Le reste du cycle peut alors se poursuivre.

Le tableau suivant résume la spécification de la séquence des états.

État                | Situation                                         | durée
------------------- | ------------------------------------------------- | ----------------------------------
Vert_CCSC           | Feu vert CCSC est; flèche-tout-droit CCSC ouest   | DUREE_VERT
Jaune_CCSC          | Feux jaunes sur CCSC                              | DUREE_JAUNE
Rouge_partout_1     | Feux rouges dans toutes les directions            | DUREE_ROUGE_PARTOUT
Vert_AVDI           | Feu vert sur l'AVDI                               | DUREE_VERT
Jaune_AVDI          | Feu jaune sur l'AVDI                              | DUREE_JAUNE
Rouge_partout_2     | Feux rouges dans toutes les directions            | DUREE_ROUGE_PARTOUT
Fleche_CCSC_ouest_1 | Feu rouge CCSC est et AVDI; flèche-gauche-verte et flèche-tout-droit sur CCSC ouest    | DUREE_VERT
Fleche_CCSC_ouest_2 | Feu rouge CCSC est et AVDI; flèche-gauche-rouge et flèche-tout-droit sur CCSC ouest    | DUREE_ROUGE_PARTOUT


### Partie 1c. Contrôle des feux pour les piétons pour traverser l'intersection

Modifiez le contrôleur pour permettre aux piétons de traverser l'intersection. 

Quand un piéton appuie sur le bouton d'appel (lors d'une transition positive du signal d'horloge), le système doit mémoriser ce choix et envoyer une rétroaction à tous les piétons en allumant le témoin "appel de piéton" sur les affichages.

Conseil : ajoutez un registre d'un bit, par exemple _appel_pieton_actif_, et donnez-lui une valeur de '1' quand le bouton d'appel de piéton est pesé. Il faut vérifier le bouton dans chacun des états, sauf l'état STOP et l'état où les piétons peuvent traverser. Le registre doit être remis à '0' quand on passe la priorité aux piétons.

Si le bouton a été pesé (tel qu'indiqué par la valeur du registre _appel_pieton_actif_), alors avant que le feu ne passe au vert sur l'AVDI, le système doit passer dans un état où tous les feux sont rouges et où les silhouettes de piétons sont allumées pendant _DUREE_PIETONS_. Pendant les 10 dernières secondes du temps pour les piétons, les silhouettes de piéton doivent être éteintes, la main levée doit clignoter à une fréquence de 1 Hz, et le décompte du nombre de secondes restantes doit être affiché sur les décompte pour piétons. Le système doit fonctionner correctement si _DUREE_PIETONS_ est inférieur à 10 secondes; dans ce cas, on ne doit avoir que la main levée clignotante et le décompte pendant le temps correspondant.


### Simulation, synthèse implémentation

Simulez complètement votre code. Faites en la synthèse et l'implémentation sur votre carte. Vérifiez-en le fonctionnement.

### À remettre pour la partie 1 :
- Une brève explication de vos modifications dans le fichier [rapport.md](rapport.md);
- Un seul fichier [feux_circulation.vhd](sources/feux_circulation.vhd) modifié pour toute la partie 1;
- Un diagramme d'états modifié en format .png ou .svg. Vous pouvez modifier directement [le diagramme fourni avec diagrams.net](https://app.diagrams.net/) ou bien soumettre un dessin fait à la main, au propre, ou par tout autre moyen.
- Votre fichier de configuration final : [labo_3.bit](/synthese-implementation/labo_3.bit).

## Partie 2 : Bonification du banc d'essai

Modifiez le banc d'essai pour qu'il fasse des vérifications de sécurité seulement. Votre banc d'essai n'a pas à vérifier la séquence des états. Il doit seulement vérifier que les sorties du module de contrôle ne créent pas de situations dangereuses.

Voici des exemples de fonctionnalités que vous pourriez ajouter à votre banc d'essai : 
- Vérification qu'on n'a jamais un feu vert simultanément dans plus d'une direction.
- Vérification qu'on n'a jamais un feu vert dans une direction et un feu jaune dans une autre.
- Vérification que le feu est rouge dans une direction si le feu est vert ou jaune dans l'autre.
- Vérification qu'il y a un seul feu allumé à la fois (rouge, jaune, vert ou flèche) pour chacune des direction.
- Vérification que les feux sont conformes aux spécification pour les piétons.
- etc.

### À remettre pour la partie 2 :
- Une brève explication des vérifications de votre banc d'essai dans le fichier [rapport.md](rapport.md);
- Votre fichier [feux_circulation_tb.vhd](sources/feux_circulation_tb.vhd) modifié;

## Partie 3: Bonus

**Mise en garde**. *Compléter correctement les parties 1 et 2 peut donner une note de 17 / 20 (85%), ce qui peut normalement être interprété comme un A. La partie bonus demande du travail supplémentaire qui sort normalement des attentes du cours. Il n'est pas nécessaire de la compléter pour réussir le cours ni pour obtenir une bonne note. Il n'est pas recommandé de s'y attaquer si vous éprouvez des difficultés dans un autre cours. La partie bonus propose un défi supplémentaire pour les personnes qui souhaitent s'investir davantage dans le cours INF3500 en toute connaissance de cause.*


### 3a. Tourner à droite sur l'AVDI

Quand le feu est vert sur l'AVDI, on peut permettre aux automobilistes qui circulent en direction est sur le CCSC de tourner à droite sur l'AVDI au même moment.

Ajoutez une flèche pour tourner à droite sur l'un des segments inutilisés des feux de circulation du CCSC en direction est.

Tenez correctement compte des délais de transition entre les autorisations de circuler dans des directions différentes. Il faut au moins _DUREE_ROUGE_PARTOUT_ secondes après que la flèche pour tourner soit éteinte avant de permettre aux automobilistes qui circulent en sens inverse de passer.

Expliquez tous vos changements dans votre rapport.

### 3b. Tourner à droite sur le CCSC

Quand la flèche-gauche-verte est allumée pour permettre aux automobilistes de tourner à gauche sur l'AVDI, on peut permettre aux automobilistes qui circulent en direction nord sur l'AVDI de tourner à droite sur le CCSC au même moment.

Ajoutez une flèche pour tourner à droite sur l'un des segments inutilisés des feux de circulation de l'AVDI.

Tenez correctement compte des délais de transition entre les autorisations de circuler dans des directions différentes. Il faut au moins _DUREE_ROUGE_PARTOUT_ secondes après que la flèche pour tourner soit éteinte avant de permettre aux automobilistes qui circulent en sens inverse de passer.

Expliquez tous vos changements dans votre rapport.


## Remise

La remise se fait directement sur votre entrepôt Git. Faites un 'push' régulier de vos modifications, et faites un 'push' final avant la date limite de la remise. Respectez l'arborescence de fichiers originale. Consultez le barème de correction pour la liste des fichiers à remettre.

**Directives spéciales :**
- Ne modifiez pas les noms des fichiers, les noms des entités, les listes des `generics`, les listes des ports ni les noms des architectures.
- Remettez du code de très bonne qualité, lisible et bien aligné, bien commenté.
- Indiquez clairement la source de tout code que vous réutilisez ou duquel vous vous êtes inspiré/e.
- Modifiez et complétez le fichier [rapport.md](rapport.md), entre autres pour spécifier quelle carte vous utilisez.


## Barème de correction

Le barème de correction est progressif. Il est relativement facile d'obtenir une note de passage (> 10) au laboratoire et il faut mettre du travail pour obtenir l'équivalent d'un A (17/20). Obtenir une note plus élevée (jusqu'à 20/20) nécessite plus de travail que ce qui est normalement demandé dans le cadre du cours et plus que les 9 heures que vous devez normalement passer par semaine sur ce cours.

Critères | Points
-------- | ------
Partie 1a : Transitions rouge-partout dans la séquence : code et diagramme d'états modifiés | 3
Partie 1b : Contrôle de la flèche pour tourner à gauche sur l'AVDI | 4
Partie 1c :  Contrôle des feux pour les piétons pour traverser l'intersection : code et diagramme d'états modifiés | 4
Partie 2 : Bonification du banc d'essai avec vérifications de sécurité, code du banc d'essai modifié | 4
Qualité, lisibilité et élégance du code : alignement, choix des identificateurs, qualité et pertinence des commentaires, respect des consignes de remise incluant les noms des fichiers, orthographe, etc. | 2
**Pleine réussite du labo** | **17**
Bonus : Tourner à droite sur l'AVDI, code et diagramme d'états modifiés | 1.5
Bonus : Tourner à droite sur le CCSC                                | 1.5
**Maximum possible sur 20 points** | **20**


## Références pour creuser plus loin

Les liens suivants ont été vérifiés en septembre 2021.

- Aldec Active-HDL Manual : accessible en faisant F1 dans l'application, et accessible [à partir du site de Aldec](https://www.aldec.com/en/support/resources/documentation/manuals/).
- Tous les manuels de Xilinx :  <https://www.xilinx.com/products/design-tools/vivado/vivado-ml.html#documentation>
- Vivado Design Suite Tcl Command Reference Guide : <https://www.xilinx.com/content/dam/xilinx/support/documentation/sw_manuals/xilinx2021_1/ug835-vivado-tcl-commands.pdf>
- Vivado Design Suite User Guide - Design Flows Overview : <https://www.xilinx.com/support/documentation/sw_manuals/xilinx2020_2/ug892-vivado-design-flows-overview.pdf>
- Vivado Design Suite User Guide - Synthesis : <https://www.xilinx.com/support/documentation/sw_manuals/xilinx2020_2/ug901-vivado-synthesis.pdf>
- Vivado Design Suite User Guide - Implementation : <https://www.xilinx.com/support/documentation/sw_manuals/xilinx2020_2/ug904-vivado-implementation.pdf>
