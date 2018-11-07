# Prosit 2 : TCP/UDP

## Equipe :
-**Animateur :** John
-**Secrétaire :** Emilien
-**Scribe :** Flo
-**Gestionnaire :** Vibrophone

## Mots-clés :

-Processus léger: thread

-Thread: processus léger

-File d’execution: liste des processus prêts à être exécutés

-Bloquant non bloquant

-Multithreading

-(A)Synchrone

-Pointeur

-Capsule d’adresse mémoire

-Pointeur de fonction: on code la fonction fun, ```void (*fun_ptr)(int) = &fun;```on l'appelle avec ```(*fun_ptr)(argument);```ou 
`void (*fun_ptr)(int) = fun;
fun_ptr(10);` aka callback

-Quantum de temps: plus petite unité de eps donnée à un processus

-Processus parent/enfant

-Fork, Exec, Exit, Wait: primitives système

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

Comment implémenter du multi-threading dans un programme ?

Comment réduire le temps d’exécution la solution asynchrone ?

Comment rendre une interface non bloquante à l’aide du multi-threading sans altérer les performances ?

## Généralisation :-Gain de temps-Efficacité-Optimisation-UX-Identification

## Hypothèses :

- Un pointeur de fonction permet de transformer en analogue de méthode

- L’augmentation du temps global asynchrone est dû au temps partagé processeur

- Quand le quantum de temps n’est pas assez grand, l’OS peut sauvegarder l’état de la mémoire et peut le restaurer plus tard

- On a moyen de manipuler les processus pour optimiser le temps d’exécution

- Le multi-thread est plus efficace sur un processeur à plusieurs cœurs

## Plan d'action :

## Etudes :
### Primitives ( C )

un pipe est un pseudo fichier qui permet à 2 processus de communiquer ensembles

Les conventions d’appel de fonction API sont prototypées dans des fichiers « header » bien connus : stdio.h, process.h, time.h, etc

Les fonctions système UNIX nécessitent généralement des paramètres, elles rendet leurs résultats sous deux formes:
- les paramètres résultants du service demandé qui sont généralement placés à l'adresse pointée par un des arguments
- un compte-rendu qui est souvent placé dans un registre du processeur. il s'agit en général d'un entier signé au format du registre, il est positif ou nul si la fonction s'est bien déroulée et négatif en cas d'erreur


**gestion de processus (process.h)**

pid = fork()
créé un nouveau processus fils identique au parent

wait(&status)
attend la fin d'un processus et renvoie son status

s = execve(name, argv, envp)
remplace le "core image" d'un processus

exit(status)
termine l'exécution d'un processus et retourne son status

size = brk(addr)
place le segment de donnée à l'adresse addr et renvoie la taille

pid= getpid()
retourne le PID du processus appelant

**Signaux interprocessus (signal.h)**

oldfunc = signal(sig, func)
remplace le traitement stadard du signal sig par une nouvelle fonction

s = kill(pid, sig)
envoie le signal sig au processus désigné par le pid

residu = alarm (secondes)
génère le signal SIGALARM après un certain nombre de secondes

s = pause()
Suspend le processus appelant jusqu'au prochain signal


Dans le cas général, l'appel exec a trois paramètres :
 1. le nom du fichier à exécuter
 2. un pointeur sur une table d'arguments
 3. un pointeur sur une table d'environnement

La plupart des librairies C proposent des sous fonctions de execve permettant de passer les paramètres de différentes manières (execl, exevp ...). Parfois même, une combinaison toute faite de fork et execve (spawnl).

**structure de la core image**
divisé en 3 segments:
- aux adresses les plus basses, on trouve le "text segment" qui contient le codeexécutable, au dessus duquel on trouve le "data seg" qui contient les donées
- au dessus des données se trouve une zone de mémoire libre (le gap)
- et à la fin on a la pile (stack segment)

la pile s'étends vers le bas de la mémoire (dans le gap) et les données peuvent s'étendre vers le haut si besoin

l'appel "brk" permet d'étendre la zone de données vers le haut en déplaçant l'adresse de fin du data segment, l'adresse spécifiée en paramètre peut être plus grande ou plus petite que la valeur courante et il faut donc faire attntion à ne pas donner une adresse appartenant au stack segment (éviter l'overlap)


### Signaux-Gestion des processus

Un processus peut avoir à manipuler des interruptions soft.
Par exemple, si l'opérateur a décidé d'afficher un fichier à l'aide d'un éditeur, et qu'il s'aperçoit que le fichier est très long, il peut arrêter l'édition en appuyant sur la touche DEL. Cela envoie un signal au processus qui exécute alors une procédure de fin d'édition.

Lorsqu'un processus reçoit un signal, l'état du processus est poussé dans la pile et une procédure de gestion des signaux est appelée. Chaque signal est associé à un pointeur vers une fonctionqui est appellée à son activation, on peut modifier ce pointeur sauf pour SIGKILL et SIGSTOP

un signal peut être généré par:
- le noyau, ex: un processus accède à une zone protégé -> SIGSEGV
- kill qui envoie SIGKILL
- le processus lui-même (par exemple avec alarm)
- la frappe d'une touche par l'utilisateur (ctrl+c envoie SIGINT)


- 1 SIGHUP Hangup, envoyé à tous les processus attachés à un terminal lorsque celui-ci est déconnecté du système. Il est courant d'envoyer le signal à certains processus démons afin de leur demander de se réinitialiser. kill 
- 2 SIGINT La touche DEL/CTRL+C a été appuyée, interomp le processus 
- *3 SIGQUIT La touche QUIT/ CTRL+\ a été appuyée. kill+core
- *4 SIGILL  illegal instruction: ne doit jamais arrivé normalement sauf si fichier corrompu. kill + core
- *5 SIGTRAP Trace trap, arrêt en mode trace: envoyé après chaque instruction, utilisé par les progde debug
- *6 SIGIOT Instruction IOT rencontrée (I/OTrap) problème de hardware. Core 
- *7 SIGMENT Instruction EMT rencontrée 
- *8 SIGFPE Floating Point Exception: kill le process et créé le fichier d'image mémoire *core*
- 9 SIGKILL Avortement de processus peut importe l'état
- *10 SIGBUS Erreur bus: erreur d'alignement des adresse sur le bus. kill+ core
- *11 SIGSEGV Violation de segment: adressage émire invalide. kill+ core 
- *12 SIGSYS Appel système avec erreur d'arg. 
- 13 SIGPIPE Ecriture sur un pipe sans lecteur 
- 14 SIGALRM Alarme 
- 15 SIGTERM signal envoyé par kill, kill le process quand il est réélu
- 16 Non utilisé (au choix du programmeur).

execution matérielle:
- SIGFPE
- SIGILL
- SIGSEV
- SIGBUS
- SIGTRAP

terminaison ou interruption de processus:
- SIGHUP
- SIGINT
- SIGQUIT
- SIGIOT
- SIGABRT erreur matérielle, arrêt du processus. Core
- SIGKILL
- SIGTERM

signaux utilisateurs:
- SIGUSR1 : défini par le programmeur, par défaut kil le process
- SIGUSR2 comme le 1

pour un pipe:
- SIGPIPE : erreur d'écriture dans un pipe, kill

signaux liés au contôle d'actvité:
- SIGCLD envoyé au processus parent lorsque les fils terminent. aucne action par défaut
- SIGCONT reprends l'exé 'un processus stoppé. aucune action si il n'était pas stop
- SIGSTOP suspension du processus, non modifiable
- SIGSTP ctrl+z, comme sigstop mais on peut le modif
- SIGTTIN émis par leterminal en direction d'un processus d'arrière plan qui essaie de lire le terminal, par défaut: supsension
- SIGTTOU émis par le terminal en direction d'un processus d'arrière plan qui essaie d'écrire sur le terminal, par défaut: suspension

alarmes:
- SIGALARM : généré lors de la mise en route du système d'alarme avec alarm()


\* Génèrent une copie du « core image » dans un fichier dans l'état exact où il est au moment de l'erreur et le message « core dumped » est affiché. Cette image de la mémoire est destinée au débogage.

**interception des signaux**
les traitements sont effectués par défaut par le noyau UNIX mais le programmeur a la possibilité d'ignorer ou remplacer son traitement. L'appel "signal" permet de diriger à nouveau les interruptions ```signal(SIG, ACTION)``` où SIG est le numéro du signal à intercepter et action un pointeur sur la fonction à exécuter ou une de ces deux cntantes:
- SIG_IGN: igniore l'interruption
- SIG_DFL: rétablit le traitement par défaut

note: après chaque traitement pour une interuption donnée la procédure par défaut est rétablie, on doit donc la re-modifier après chaque traitement 

UNIX offre également des services qui permettent de générer des signaux dans des conditions maitrisables par le développeur : 
- L’appel alarm (secondes) envoie le signal SIGALRM (No 14) au processus appelant après un temps spécifié en secondes. Ceci permet de fixer des délais de réalisation d'une action (time out). 
- L'appel pause() suspend l'exécution du processus jusqu'à réception d'un signal quelconque. 
- L’appel kill ( PID, SIG ) permet d’envoyer les signal SIG au processus PID

**table des processus**
l'os manipule une structure de données qui lui permet de conserver d'autres iformations sur les processus: la tabledes processus. Elle contient toutes les infos indispensables à l'OS pour assurer une gestion cohérente des processus. Elle est stockée dans l'espace mémoire de l'OS 

### Relation OS avec processus
### Fonctionnement de l’ordonnanceur
### Multitâche Linux

le seul moyen de créé des processus sous linux est avec l'appel système fork

au démarrage de linux un processus appelé *init* s'exécute, il lie un fichier indiquant combien de terminaux sont présents, génère un nouveau processus par terminal
si l'ouverture d'une session réussi il lance un shell
les commande du shell peuvent lancer d'autres processus etc
tous les processus de l'OS appartiennent à l'arborescence d'*init*


on peut créer une tâche sur linux avec: initialisation système, exécution d'un appel système par le processus en cours, requête utilisateur demandant la création d'un processus

fork créé un clone du processus l'appelant
après le fork les deux processus ont la même image mémoire et les même fichiers ouverts. L'espace d'adressage initial du fils est une copie de celui du père, il n'y a pas de mémoire partagée, si l'un des deux modifie so espace d'adressage l'autre ne le sais pas.

en général le processs fils excute l'appel système *execve* pour modifier son image mémoire et exécuter un nouveau programme.

un processus se termine si:
- il a terminé sa tâche (*exit* sous unix)
- suite à une erreur 
- tué par un autre processus (kill)


classification des processus:

tributaire des I/O: dépense la plupart du temps a effectuer des requêtes I/O plutôt que des calaculs
tributaire du CPU: génère peut fréquement des requêtes I/O et passe donc la plupart du temps à faire des calculs

rôles du scheduler à long terme (aka scheduler de travaux)
	- selectionne le processus qui doit aller dans la file de processus prêts
	- s'exécute moins fréquement: secondes, minutes
	- contrôle le degré de multiprogrammation (nb de process dans la mémoire)
	- réparti les processus tributaires CPU et tributaire I/O (comme ça on lance un I/O et un CPU peut s'exécuter pendant qu'il lis/écris)

rôles du scheduler à court terme (aka scheduler du CPU)
	- choisit parmi les processus prêt celui a exé
	- doit être TRES rapide
scheduler moyen terme:
- idée clé: il peut être avantageux de suppr des processus de la mémoire et réduire le degré de multi-prog
- plus tard, un processus peut être réintroduit dans la mémoire et son exé peut reprendre là où elle en était
- c'est le swapping (on passe de mémoire principale à auxiliaire et vice-versa)

on peut classer les scheduler en 2 catégories:

non préemptif
- selec le process, le laisse tourner jusqu'à ce qu'il bloque où libère le processeur
- même s'il s'exé pendantdes heures il ne sera pas suspendu de force
-aucune décision d'ordonnancement n'intervient pendant les interruptions d'horloge

préemptif
- sélec le processus à run et le laisser pendant un délai déterminé
- si il est toujours en cours après ce délai on le suspend et on en lance un autre


les schedulers ont des priorités différentes et peuvent favoriser une classe de processus plutôt qu'une autre, il se basent sur:
- utilisation CPU pour le garder autant occupé que possible
- capacité de traitement (throughput), nbde processus terminés par unité de temps
- temps de restitution (turnaround time), temps nécessaire pour l'exécution du processus (temps passé à attendre d"entrer dans la mémoire + temps passé dans la file d'attente + temps d'exé sur le CPU + temps I/O)
- temps d'attente: temps qu'un processus passe à attendre dans la file d'attente (thx captain obvious)
- temps de réponse: delta moment de soumettre la requête et arrivé  de la première réponse

différents algo de scheduling : 

**FCFS**
- impl facile avec une file (FIFO)
- un procesus garde le CPU jusqu'à ce qu'il le libère (fin ou I/O)
- incommode pour le temps partagé puisque les utilisateurs ont besoins du CPU à interval régulier
- pas de temps d'attente minimal et varie si le temps de cycles varient beaucoup
- effet d'accumulation (convoy effect) provoque une utilisation du CPU et des périph plus lente que si on permettait au processus le plus court de passer le premier

**SJF shortest job first**
- associe à chaque processus la longueur de son prochain cycle de CPU, quand le CPU est disp il est assigné au plus court et si 2 ou + ont la même durée FCFS
- cette méthode est optimale: elle obtient le temps moyen minimal pour un ensemble de proessus donné
- difficulté: pouvoir connaître la longueur de la prochaine requête CPU
- existe en préemptif et non-préemptif

**Algo avec des priorités**
- on associe une prio a chaque processus
- on aloue le CPU a celui avec la plus haute priorité (le plus petit chiffre)
- ceux ayant la même prio suivent FCFS
- existe en préemptif ou non préemptif
- problème: famine (starvation) ou blocage indéfinie: les processus avec basses prio risquent d'attendre longtemps et ne jamais s'exécuter
- solution: vieillissement (aging): on augmente la prio quand on processus attends un certains temps 


**Round robin**
- chaque processus a un quantu (en général 10 à 100ms)
- le scheduler parcourt la file d'attente et alloue le temps pour un durée max d'un quantum
- la file d'attente est un FIFO
- avant d'atteindre une durée d'un quantum le processus libérera volontairement le CPU si il a fini
- arrivé à un quantum l'horloge s'arrêtera et provoquera une interuption à l'OS, commutation de contexte (= on save et passe au prochain), le processus est réinséré à la fin de la file
- temps moyen d'attente souvent très long
- (obvious math) pour n processus et un quantum q 
	- chaque processus opbtient 1/n temps de proco par tranche de q temps
	- un processus attents au max q(n-1) temps entre 2 passages
- les perf dépendent de la taille du quantum
	- si il est très grand => FCFS
	- si il est très petit => partage équitable du proco
mais q doit être suffisamment grand par rapport au temps de commutation


**file d'attente à multiniveaux**
séparer les processus ayant des caractéristiques différentes de cycle de CPU en différentes files
- ceux utilisant beaucoup de proco ont une prio inf
- les processus I/O ont une prio supp
- un processus qui atteds trop longtemps dans une file lente peut être passé dans la prio supp

**comment choisir l'algo de scheduling pour un système donné ?**
les différentes méthodes d'valuation:
- modé déterministe: prend en compte une charge de travail prédéterminée et définit la perf de chaque algo pour cette charge de travail
- modèles de file d'attente
- simulations
- implantation


**communication inter-processus**
les processus peuvent avoir besoin de communiquer entre eux (pipeline, shell) l'OS doit fournir aux processus coopératifs les moyens de communiquer entre eux grâce à des fonctions de communication inter-processus

**IPC** interprocess communication:

**treads**
il existe 2 types de thread:
- niveau système
- niveau utilisateur

un thread comporte:
- 1 compteur d'instructions
- 1 ensemble de registres
- 1 pile
- peut save la clock
- file

un thread partge avec les threds ressemblants
- la section de code
- la section de données
- les ressouces de l'OS

pocessus (lourd) = task+ thread



### Librairies actuelles ( C++ )-GIL (Max)

Java: Quartz
C#: quartz.net
Ruby = Rufus-Scheduler, clockwork
c++ Bosma, libcron (utilise cron formatting), libevent
python: schedule, Advanced Python Scheduler (APScheduler), celery
C: Embedded C Scheduler
### Réalisation :
-WORKEU CHOPPE
