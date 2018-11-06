

# Prosit 2 : Multiprogrammation

  

## Equipe :

-   Animateur : John
    
-   Secrétaire : Emilien
    
-   Scribe : Flo
    
-   Gestionnaire : Vibrophone
    

  

## Mots-clés :

  

- Processus léger = thread

- Thread = morceau d'un processus

- File d’execution = file pour traitement processeur

- Bloquant non bloquant = préemption = mis en attente

- Multithreading = plusieurs tâches en simultanée (illusion)

- (A)Synchrone = deux processus qui ne se déroulent pas de manière synchronisés

- Pointeur = *

- Capsule d’adresse mémoire = range d'adresse mémoire

- Pointeur de fonction = pointeur dans une fonction

- Quantum de temps = petit laps de temps

- Processus parent/enfant = père (PID), fils (PID) + (PPID = père) ⇒ Crées avec fork()

- Fork, Exec, Exit, Wait : Primitives

 
## Contexte :

**Qui ?**

Osef

**Quoi ?**

Réaliser un programme pour calculer des factorielles

Empêcher que le programme freeze

Réaliser une interface non bloquante

**Comment ?**

En implémentant le multi-thread avec des signaux

**Pourquoi ?**

Interface non bloquante

## Contraintes :

Ø

## Problématique :

 
~~Comment implémenter du multi-threading dans un programme ?~~

Comment réduire le temps d’exécution la solution asynchrone ?

~~Comment rendre une interface non bloquante à l’aide du multi-threading sans altérer les performances ?~~

## Généralisation :

~~-   Gain de temps~~
    
~~-   Efficacité~~
    
~~-   Optimisation~~
    
~~-   UX~~
    
-   Identification
    

  
  

## Hypothèses :

  

-   Un pointeur de fonction permet de transformer en analogue de méthode
    
-   L’augmentation du temps global asynchrone est dû au temps partagé processeur
    
-   Quand le quantum de temps n’est pas assez grand, l’OS peut sauvegarder l’état de la mémoire et peut le restaurer plus tard
    
-   On a moyen de manipuler les processus pour optimiser le temps d’exécution
    
-   Le multi-thread est plus efficace sur un processeur à plusieurs cœurs
    

  
  
 
## Plan d'action :

Etudes :

-   Primitives ( C )
    
-   Signaux
    
-   Gestion des processus
    
-   Relation OS avec processus
    
-   Fonctionnement de l’ordonnanceur
    
-   Multitâche Linux
    
-   Librairies actuelles ( C(pp) )
    
-   GIL (Max)


### Réalisation :

-   WORKEU CHOPPE


## Primitives C & signaux

**Identification de processus**
- getpid()
- getppid()

**Primitive fork**
fork() permet la création dynamique d'un nouveau processus fils, qui s'exécute de façon concurrente avec son père. Il hérite de plusieurs de ses attributs.
- Même code
- Copie de la zone de données du père
- Environnement
- Priorité
- Descripteur de fichiers ouverts
- Du traitement des signaux

**Primitive exit**
exit(n) met fin au process avec le code de retour n. (Si OK : 0)

**Primitive wait**
wait() provoque la suspension du processus appelant jusqu'à que l'un de ses processus fils se termine, ou reçoit un signal.

**La famille "exec"**
C'est une famille de primitive permettant le lancement de l'exécution d'un nouveau programme par un processus.

**Primitives de signaux**
```C
#define SIGHUP  1 /* Hangup (POSIX).  */
#define SIGINT  2 /* Interrupt (ANSI).  */
#define SIGQUIT  3 /* Quit (POSIX).  */
#define SIGILL  4 /* Illegal instruction (ANSI).  */
#define SIGTRAP  5 /* Trace trap (POSIX).  */
#define SIGABRT  6 /* Abort (ANSI).  */
#define SIGIOT  6 /* IOT trap (4.2 BSD).  */
#define SIGBUS  7 /* BUS error (4.2 BSD).  */
#define SIGFPE  8 /* Floating-point exception (ANSI).  */
#define SIGKILL  9 /* Kill, unblockable (POSIX).  */
#define SIGUSR1  10 /* User-defined signal 1 (POSIX).  */
#define SIGSEGV  11 /* Segmentation violation (ANSI).  */
#define SIGUSR2  12 /* User-defined signal 2 (POSIX).  */
#define SIGPIPE  13 /* Broken pipe (POSIX).  */
#define SIGALRM  14 /* Alarm clock (POSIX).  */
#define SIGTERM  15 /* Termination (ANSI).  */
#define SIGSTKFLT 16 /* Stack fault.  */
#define SIGCLD  SIGCHLD /* Same as SIGCHLD (System V).  */
#define SIGCHLD  17 /* Child status has changed (POSIX).  */
#define SIGCONT  18 /* Continue (POSIX).  */
#define SIGSTOP  19 /* Stop, unblockable (POSIX).  */
#define SIGTSTP  20 /* Keyboard stop (POSIX).  */
#define SIGTTIN  21 /* Background read from tty (POSIX).  */
#define SIGTTOU  22 /* Background write to tty (POSIX).  */
#define SIGURG  23 /* Urgent condition on socket (4.2 BSD).  */
#define SIGXCPU  24 /* CPU limit exceeded (4.2 BSD).  */
#define SIGXFSZ  25 /* File size limit exceeded (4.2 BSD).  */
#define SIGVTALRM 26 /* Virtual alarm clock (4.2 BSD).  */
#define SIGPROF  27 /* Profiling alarm clock (4.2 BSD).  */
#define SIGWINCH 28 /* Window size change (4.3 BSD, Sun).  */
#define SIGPOLL  SIGIO /* Pollable event occurred (System V).  */
#define SIGIO  29 /* I/O now possible (4.2 BSD).  */
#define SIGPWR  30 /* Power failure restart (System V).  */
#define SIGSYS  31 /* Bad system call.  */
```

**Primitives de manipulation du masque des signaux**
- sigprocmask : Consulter et affecter le masque des signaux du processus appelant, définissant l'ensemble des signaux masqués pour un processus :
- sigsetops : fournit la description des fonctions manipulant les masques
- sigpending : indique en retour les signaux bloqué et non délivrés

**alarm**
alarm() entraîne l'envoi du signal SIGALARM au processus.

**horloges**
- setitimer() / getitimer() : manipuler trois horloges associées à un processus.

**points de reprises**
setjmp() / longjmp() : utile pour la mise en place et l'utilisation de points de reprise.

## Gestion des processus

/!\ Process != programme : 
- Programme ⇒ statique = Recette du gâteau = Séquence d'instructions stockés en RAM
- Processus ==> dynamique = Suite d'actions effective = Exécution du programme
	- Instance d'exécution d'un programme
	- Table qui décrit l'espace mémoire réservé à l'OS
	- État de la machine à un instant t

![](https://perso.liris.cnrs.fr/pchampin/enseignement/se/_images/ps_states.png)
- Eligible = prêt = en attente d'un processeur pour s'exécuter ⇒ Création par l'OS
- Elu = Actif = en train de s'exécuter sur un processeur ==> Manipulation par l'ordonnanceur
- Bloqué = En attente d'un évènement.
- préemption = désactivation

A chaque processus, nous avons un contexte :
- Matériel : photographie de l'état des registres à un instant 't' 
- Logiciel : Infos sur droits d'accès aux ressources de la machine (priorité, quotas, espace disponible..)

Nous avons également des infos mémorisés (UNIX) : 
- PID (identifiant)
- Configuration des registres (pile, compteur)
- Répertoire courant

Lors de l'exécution d'un processus, ce contexte va changer, changeant les configuration de ses registres. Ce contexte est disponible dans le PCB :

**PCB (Process Control Block) :**
- Structure de données du SE contenant les informations sur un processus
- Ces infos sont dans une zone mémoire accéssible uniquement par le noyau du SE
- Il faut utiliser des appels systèmes pour y accéder (getpid, getppid, getuid...)

Tous les processus sont associés à une entrée dans la table des processus (interne au noyau) :
- Code : Code exécutable (pointeur vers le programme, ce qui permet à la fois de partager le code par plusieurs processus, mais aussi de le déplacer par le système)
- Data : Données  (manipulable par le noyau)
- Stack: Pile

## Le rôle de l'OS 

![](https://i.ytimg.com/vi/d-vhxg_j1kM/maxresdefault.jpg)

Les rôles de l'OS sont :
- Gestion du processeur (allocation du processeur entre les différents programmes) grâce à un algorithme d'ordonnancement. Il va rendre les processus "éligibles" parmi ceux qui sélectionnées.
- Gestion de la mémoire (RAM et mémoire virtuelle) pour que tous les ps aient une allocation suffisante + empêcher famine
- Gestion entrée/sorties : unifier et contrôler l'accès aux ressources via des pilotes
- Gestion de l'exécution des applications : Chargé de la bonne exécution en leur affectant des ressources. Il peut donc kill si nécessaire.
- Gestion des droits 
- Gestion des fichiers : gère la lecture et l'écriture
- Gestion des informations : Fournit des indicateurs pour diagnostiquer une machine

### Ordonnanceur :

Composant du noyau (kernel) du système d'exploitation :
- Choisis l'ordre d'exécution des processus sur les processeur d'un ordinateur
- Répartit selon les critères de priorité, le temps machine entre chaque processus
- Rend "élu" ou "bloqué" les différents processus
Un système est dit **préemption** lorsqu'il possède un ordonnanceur (planificateur) :

Un système est dit à **temps partagé** lorsqu'un quota de temps est alloué à chaque ps par l'ordonnanceur (comme systèmes multi-utilisateurs)

![](http://www.noelshack.com/2018-45-1-1541415016-capture.png)


## Multithread :

- Multi-tâche : Plusieurs tâches peuvent-être exécutées simultanément.
- Applications composés en séquence d'instructions que l'on appelle "processus léger" **(thread)**

Plusieurs processus en mémoire et partager l'unité centrale :
- rendement (throughput) : nombre de processus exécutés par unité de temps
- temps de service : temps mémoire + attente processus éligible + temps exécution + attente + exécution périphériques e/s)
- temps d'attente : temps passé dans la file des processus éligibles
- temps de réponse : temps qui s'écoule entre la soumission d'une requête et la première réponse obtenue

**Algorithmes processus #ordonnanceur :**
- FCFS (First-come, first served)
	- file d'attente FIFO
	- Incommode pour le temps partagé
- SJF (Shortest job first) 
	- Associe à chaque processus la longueur de son prochain cycle CPU, le cycle le plus petit passe en premier.
	- Optimal, mais difficile de connaître la longueur d'un CPU
- Priorités 
	- Priorités sur chaque PS
	- Problème de famine à cause de ps avec basses priorités
	- Solution : Viellissement (aging)
- RR (Round Rodin)
	- Chaque processus a une petite unité de temps, appelée tranche de temps ou quantum 
	- Temps moyen d'attente souvent très long (dépend de la taille du quantum)
- File d'attente à multiniveaux
	- Systèmes de files à priorités
	- Séparer les ps ayant des caractéristiques différentes de cycle de CPU en différentes files, permettant aux PS de se déplacer entre les files (temps processeur, E/S, attend trop longtemps...)

**Concurrence :**
- Problème de synchronisation si deux processus accèdent à une ressource en même temps **en modification**
- La partie de programme dans laquelle se font des accès à une ressource partagée s'appelle une "section critique"
⇒ On met donc des verrous (sémaphores / fonction)

## Librairie actuelles C/C++:

- \<pthread> (c)
- \<Boost> (c++)
- TBB (thread building blocks)
- PPL (microsoft)

Others
- MPI
- POCO
- ICE
- ACE
- TBB (déconseillée pour débuter)
- OPENMP (déconseillée pour débuter)
