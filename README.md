
# Prosit 2 : Multiprogrammation

  

## Equipe :

-   Animateur : John
    
-   Secrétaire : Emilien
    
-   Scribe : Flo
    
-   Gestionnaire : Vibrophone
    

  

## Mots-clés :

  

- Processus léger

- Thread

- File d’execution

- Bloquant non bloquant

- Multithreading

- (A)Synchrone

- Pointeur

- Capsule d’adresse mémoire

- Pointeur de fonction

- Quantum de temps

- Processus parent/enfant

- Fork, Exec, Exit, Wait

 
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
