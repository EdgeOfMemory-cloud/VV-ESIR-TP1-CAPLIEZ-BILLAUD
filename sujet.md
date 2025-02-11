# Practical Session #1: Introduction

1. Find in news sources a general public article reporting the discovery of a software bug. Describe the bug. If possible, say whether the bug is local or global and describe the failure that manifested its presence. Explain the repercussions of the bug for clients/consumers and the company or entity behind the faulty program. Speculate whether, in your opinion, testing the right scenario would have helped to discover the fault.

2. Apache Commons projects are known for the quality of their code and development practices. They use dedicated issue tracking systems to discuss and follow the evolution of bugs and new features. The following link https://issues.apache.org/jira/projects/COLLECTIONS/issues/COLLECTIONS-794?filter=doneissues points to the issues considered as solved for the Apache Commons Collections project. Among those issues find one that corresponds to a bug that has been solved. Classify the bug as local or global. Explain the bug and the solution. Did the contributors of the project add new tests to ensure that the bug is detected if it reappears in the future?

3. Netflix is famous, among other things we love, for the popularization of *Chaos Engineering*, a fault-tolerance verification technique. The company has implemented protocols to test their entire system in production by simulating faults such as a server shutdown. During these experiments they evaluate the system's capabilities of delivering content under different conditions. The technique was described in [a paper](https://arxiv.org/ftp/arxiv/papers/1702/1702.05843.pdf) published in 2016. Read the paper and briefly explain what are the concrete experiments they perform, what are the requirements for these experiments, what are the variables they observe and what are the main results they obtained. Is Netflix the only company performing these experiments? Speculate how these experiments could be carried in other organizations in terms of the kind of experiment that could be performed and the system variables to observe during the experiments.

4. [WebAssembly](https://webassembly.org/) has become the fourth official language supported by web browsers. The language was born from a joint effort of the major players in the Web. Its creators presented their design decisions and the formal specification in [a scientific paper](https://people.mpi-sws.org/~rossberg/papers/Haas,%20Rossberg,%20Schuff,%20Titzer,%20Gohman,%20Wagner,%20Zakai,%20Bastien,%20Holman%20-%20Bringing%20the%20Web%20up%20to%20Speed%20with%20WebAssembly.pdf) published in 2018. The goal of the language is to be a low level, safe and portable compilation target for the Web and other embedding environments. The authors say that it is the first industrial strength language designed with formal semantics from the start. This evidences the feasibility of constructive approaches in this area. Read the paper and explain what are the main advantages of having a formal specification for WebAssembly. In your opinion, does this mean that WebAssembly implementations should not be tested? 

5.  Shortly after the appearance of WebAssembly another paper proposed a mechanized specification of the language using Isabelle. The paper can be consulted here: https://www.cl.cam.ac.uk/~caw77/papers/mechanising-and-verifying-the-webassembly-specification.pdf. This mechanized specification complements the first formalization attempt from the paper. According to the author of this second paper, what are the main advantages of the mechanized specification? Did it help improving the original formal specification of the language? What other artifacts were derived from this mechanized specification? How did the author verify the specification? Does this new specification removes the need for testing?

## Answers (Louis-Gabriel CAPLIEZ et Valère BILLAUD, ESIR2 Spé INFO, option SI)

### Pentium FDIV Bug, 1994

Différentes sources consultées: 
1. https://en.wikipedia.org/wiki/Pentium_FDIV_bug
2. https://encyclopedia.pub/entry/32969
3. https://math.mit.edu/~edelman/homepage/papers/pentiumbug.pdf
4. https://en.wikipedia.org/wiki/Division_algorithm#SRT_division
5. https://en.wikipedia.org/wiki/Programmable_logic_array 

En 1994, la société Intel (fabricant de processeurs) a sur le marché des nouveaux CPUs de la génération dite *Pentium*. Le professeur de mathématiques Thomas R. Nicely, de l’université de Lynchburg aux États-Unis, averti publiquement qu’une erreur est contenue dans le nouveau matériel d’Intel la même année. 
Un processeur de cette gamme utilisant une instruction nommée *FDIV*, permettant de retourner le résultat de la division entre deux opérandes flottantes, pouvait retourner un quotient erroné au niveau des arrondis pour certaines opérations.

L’exemple de la page Wikipédia (et celui le plus répendu) est le suivant : 
    • théoriquement, 4 195 835 / 3 145 727 = 1,333**820449136241002**. 
    • en pratique un processeur défectueux retourne 1,333**739068902037589**
Ceci provient d’un bogue que nous pouvons qualifier de « global », venant d’une interaction improbable entre le logiciel et le matériel.

Plus précisément, Intel a choisi de changer l’algorithme procédant à la division, shift-and-substract, avec celui de Sweeney, Robertson et Tocher (SFT). Son implémentation a été réalisée à l’aide de tableaux logiques programmables (outil permettant de définir un circuit logique programmable), au nombre de 2048. 1066 d’entre-elles devaient contenir une valeur parmi [-2,+2]. Cependant, 5 cellules, lors de la compilation du tableau et par la suite lors du chargement des données vers le matériel ayant pour but de graver la puce, ont vu leur valeur devenir 0 au lieu de +2. L’algorithme SFT étant récursif, les divisions réalisées, utilisant ces cellules du processeur, voient leur résultat faussé et extrapolé. 
Toutes personnes ayant donc un ordinateur avec un processeur de cette gamme défectueux peut voir n’importe quel de ses programmes, utilisant une division flottante, avoir un comportement anormal et imprévu. 

Une fois l’affaire rendue publique, Intel reconnaît l’existence de ce bogue mais affirme qu’il n’affecterait en réalité que peu d’utilisateurs, choisissant ainsi de ne rembourser que les clients pouvant prouver l’existence de cette anomalie. Après beaucoup de bruits médiatiques en sa défaveur et une stratégie agressive du concurrent IBM, Intel rappelle, sans conditions pour les clients, tous les processeurs défectueux. L’opération a été estimée à 752 millions de dollar de nos jours, couplée avec une image de marque fragilisée. 

Cette erreur était-elle détectable ? Un scénario naïf aurait été à l’époque de calculer toutes les opérations flottantes possibles du processeur, puis comparer son résultat pratique avec son homologue théorique. Cependant, le coût est tout de suite déraisonnable, au vu du nombre de cas tests à traiter. 
Une solution plus simple pourrait être, vu que 2048 est un nombre fini et petit, vérifier chacune des cellules des tableaux logiques programmables pour s’assurer que la bonne valeur a été inscrite. C’est-à-dire inclure une phase de test supplémentaire entre la compilation ainsi que le chargement des données vers le matériel de gravure et la réalisation de la puce en elle-même. Le coût de production aurait été supérieur pour Intel certes, mais bien moindre que le scandale qui a suivi.

### Taille socket web trop petite

Source consultée: https://incubator.apache.org/projects/wave.html

Nous avons sélectionné le bogue disponible à l’adresse [suivante](https://issues.apache.org/jira/browse/WAVE-352)
Il a été localisé par un utilisateur dans le client web du projet Apache Wave le 28 mai 2012. Lors de la création d’une nouvelle *wave*, la copie d’un texte de taille suffisamment grand dedans résulte en la disparition de tout le contenu. 
L’utilisateur, en copiant la page Wikipédia [suivante](https://en.wikipedia.org/wiki/Google_Wave) obtient un message d’erreur dans un des logs : *INFO: websocket disconnected (1009 - Text message size > 16384 chars): org.waveprotocol.box.server.rpc.WebSocketServerChannel@788219ba*. 
Il correspond à une taille de chaîne de caractères trop grande par rapport à la limite imposée par *WebSocket* implicitement. Il est proposé de configurer la taille du buffer dans *ServerRpcProvider* explicitement. Le téléchargement du patch corrigeant cette erreur révèle l’ajout d’un paramètre correspondant à la limite autorisée pour la connexion, définie à 1024*1024 MB. En testant de nouveau la copie avec le même article Wikipédia, elle n’est pas perdue. Nous n’avons pas vu de nouveaux cas de tests dans le patch fourni. Cependant la limite a été indiquée dans la [documentation](https://reviews.apache.org/r/5256/) (déplier le premier commentaire et regarder la différence pour le fichier *server.config.example*). 
Nous qualifierions ce bogue de local, plus précisément l’omission d’un paramètre correspondant à la taille de message autorisée.

### Netflix

Les sources consultées sont les suivantes : 
1. https://arxiv.org/ftp/arxiv/papers/1702/1702.05843.pdf
2. https://fr.wikipedia.org/wiki/Chaos_Monkey
3. http://principlesofchaos.org/

Afin de tester leur système informatique, les développeurs de Netflix mènent des expérimentations. En popularisant une technique de vérification de type « tolérance aux fautes », nommée l’ingénierie du Chaos, qu’ils décrivent comme « la discipline d’expérimenter sur un système distribué dans le but d’avoir confiance en sa capacité à supporter des conditions turbulentes en production », ils ont déterminé quatre principes qui englobe cette approche :
    • construire une hypothèse sur un comportement stable du système.
    • varier les évènements en temps-réel qui peuvent intervenir.
    • essayer une première fois l’expérimentation en production.
    • puis l’automatiser pour qu’elle soit exécutée plusieurs fois à l’avenir.

Le premier consiste à déterminer une métrique dans le système qui témoigne/soit représentatif de sa santé. Deux exemples sont proposés comme *SPS ((Stream) Starts per second)*, indiquant le nombre de flux vidéos démarrés en une seconde, ainsi que le nombre de connexion au service (avec ses identifiants) en une seconde. 

Le second a pour objectif de trouver des exemples d’évènements réels qui peuvent mettre le système en difficulté. Afin d’éviter le phénomène du *Happy Path*, où les développeurs ne testent pas exhaustivement leurs programme, notamment les cas d’erreurs rares qui sont susceptibles d’arriver avec des utilisateurs lambda, Netflix pioche des évènements parmi ceux survenus pendant des pannes précédentes. Des exemples retenus sont la mort soudaine d’une machine virtuelle, l’injection d’une forte latence entre deux services, voire l’échec d’échanges de requêtes et dans des cas extrêmes l’indisponibilité des serveurs Amazon dans une région. En réalité dans ce dernier cas, une simulation est plutôt réalisée car Netflix ne peut rendre indisponible des serveurs dont ils n’ont pas le contrôle. Autrement, les évènements retenus sont appliqués à un sous-ensemble d’utilisateurs du service, certains subiront les anomalies, d’autres non, afin de réduire les risques et de mieux maîtriser les expérimentations.

Le troisième est d’essayer cette expérimentation en production. Cependant une difficulté se pose car Netflix étant distribué, il est très difficile pour les développeurs de simuler entièrement l’environnement de test avec tous ses micro-services et ses interactions possibles. Des détails une fois le service lancé et déployé peuvent ne pas être prédis, notamment le comportement des clients comme cité précédemment. 
Enfin, il est important d’automatiser cette expérimentation pour qu’elle soit reproduite en temps-réel et régulièrement, afin de confirmer ou d’infirmer la solidité de Netflix. Intervient donc la *Simian Army*, des entités informatiques ayant pour but d’exécuter les expérimentations crées par les développeurs. Chacun a un rôle précis, comme le *Chaos Monkey* choisissant au hasard des machines virtuelles à tuer, ou le *10-18 Monkey* qui a pour but de détecter les problèmes de localisation (langage) sur les différentes instances. Ils ne sont pas forcément actifs en permanence, mais plutôt à des périodes définis, comme une semaine de travail ou une fois par mois, en fonction des besoins.

A la fin de chaque test, les métriques mesurées entre le sous-groupe impacté et celui non-impacté sont comparées. Si elles sont identiques, cela indique que le système a pu maintenir une qualité de service acceptable malgré des aléas (devise importante de Netflix : dégrader la qualité du système en cas de problèmes mais le maintenir disponible), ou au contraire pointent des faiblesses à corriger ultérieurement.
D’autres géants du numérique utilisent des techniques similaires pour tester leurs systèmes, comme Google, Amazon et Facebook/Meta, cependant Netflix espère voir ces derniers communiquer sur leurs adaptations des ces méthodes à leurs contextes. Comme les développeurs de Netflix l'expliquent, ils ont créé des outils trop spécifiques à leur système pour qu’ils soient facilement exportable. Cependant il est aisé de s’imaginer comment Google pourrait agir sur son moteur de recherche, comme mesurer le temps de réponse à une requête utilisateur donnée, en temps ainsi que le nombre de réponses données, puis simuler des évènements de la même manière que Netflix et regarder si le temps de réponse reste constant, dans une marge définie acceptable. S'il augmente de trop, des faiblesses à corriger auront été trouvées. Il pourrait aussi mesurer sur Youtube le nombre de vidéos démarrées, le nombre de pubs vues, ou le nombre de vidéos mises en lignes en une seconde. Pour Facebook, une métrique pourrait être le nombre d’utilisateurs qui quittent l’application ou le site par seconde, Microsoft pourrait avoir une métrique similaire sur ses différents services (cloud, IDE, consoles et plateformes de jeux vidéo).

### WebAssembly

Après plusieurs lectures de [l’article](https://people.mpi-sws.org/~rossberg/papers/Haas,%20Rossberg,%20Schuff,%20Titzer,%20Gohman,%20Wagner,%20Zakai,%20Bastien,%20Holman%20-%20Bringing%20the%20Web%20up%20to%20Speed%20with%20WebAssembly.pdf), plusieurs points importants sont ressortis.
L’approche formelle proposée par WebAssembly a permis d’aboutir à une spécification du langage (sa sémantique) très concise (tenant en moins d’une page), rendant la compilation et la vérification des programmes sources très rapides. Le langage est robuste, en effet les règles de réduction montrées dans l’article couvrent l’intégralité de tous les états d’exécution d’un programme valide, assurant qu’il n’aura jamais de comportement indéfini dans son exécution.

Des évènements tels des appels illégaux, des accès non alignés en mémoire ou même en dehors sont exclus par ce formalisme. La pile d’appel est masquée. Une encapsulation de la mémoire permet de l’abstraire et augmente ainsi sa sécurité et son intégrité, en évitant de propager des informations essentielles sur son état courant.

Les objectifs de WebAssembly sont présentés en première page par leurs auteurs. Il est comme dit précédemment rapide et sûr à compiler et exécuter, s’intègre facilement au Web et reste indépendant du matériel, du langage source ou de la plateforme sur lequel il tourne, argument essentiel au vu de la diversité des navigateurs, des OS, des machines. Il est qualifié de compact, permettant une transmission plus aisée sur le réseau Internet (la figure 6 page 13 l’illustre bien, nous pouvons remarquer une taille de binaire plus faible pour WebAssembly contrairement à du code natif ou du asm.js).

Malgré les différents points précédents, nous pensons qu’il reste nécessaire de tester les implémentations de WebAssembly, car même si la sémantique du langage a été formalisée et prouvée depuis le départ, elle pourrait avoir été biaisée en ayant été rendue facile à formaliser par les auteurs. Il se peut que les implémentations de WebAssembly ne soient pas exemptes de bogues provenant du développeur. Un code écrit dans ce langage n’est pas prouvé. Des explications sont explorées dans la question suivante de ce TP.

### WebAssembly, amélioration de la spécification

Les différentes sources consultées pour cette partie sont les suivantes :
1. https://www.cl.cam.ac.uk/~caw77/papers/mechanising-and-verifying-the-webassembly-specification.pdf
2. https://preshing.com/20120930/weak-vs-strong-memory-models/
3. https://fr.wikipedia.org/wiki/Fuzzing

Les principaux avantages de la spécification mécanisée sont qu’elle a permis d’obtenir un modèle avec des fonctionnalités en plus (boucle avec des arguments non vides non présentes à cause de restriction dans le format du binaire par exemple). De plus elle a permis de mettre en lumière des erreurs dans la spécification de WebAssembly.

Deux outils ont été développés en Isabelle, un *executable type checker* et un *executable interpreter*. Ils ont permis de garantir la robustesse du système de type de WebAssembly et de son exécution sémantique.
L’auteur a pu trouvé des failles dans la spécification initiale du langage (opération *return* erronée, la propagation d’erreur dans la pile se terminait avant d’en atteindre le sommet, des invariants trop légers dans la spécification causant un plantage d’un programme pourtant bien typé avec une *host function*, système de typage fragilisé car le système d’interface entre une implémentation WebAssembly et l’environnement hôte n’a pas été formellement spécifié...). L’équipe a l’origine de la spécification de WebAssembly a été alertée des erreurs pour la corriger.

Les outils nouvellement créés (dont un *parser and linker* permettant d’obtenir un programme exécutable seul) ont été vérifiés en passant des tests de conformité mis à disposition sur le dépôt de WebAssembly, permettant de valider la solidité du modèle créé. L’interpréteur a été confronté à des moteurs WebAssembly pour détecter des erreurs mais aussi à l’utilisation de la technique de *fuzzing* (injection de données aléatoires dans un programme pour voir s’il échoue).

Toutes les nouvelles précautions prises et la correction de la sémantique originelle en WebAssembly en la complétant, permet d’augmenter sa robustesse. Cependant les tests utilisés proviennent en partie du dépôt WebAssembly et ne garantissent pas une absence de bogues mais cherchent plutôt leurs présences. Les nouveaux outils en sont encore dépendants. Il se pourrait que d’autres éléments de la spécification soient modifiés/ajustés, sachant que l’auteur de l'article souhaite implémenter de nouvelles fonctionnalités suite à l’ajout de *weak memory semantics* à WebAssembly, tâche pointilleuse selon lui car la spécification correcte de modèle *weak memory semantics* est un problème ouvert majeur.
