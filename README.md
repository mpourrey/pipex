### Les redirections de fd dans pipex  
*(Attention, c'est juste dans mon pipex, je ne prétends pas savoir comment ça devrait être !)* 

#### C'est quoi un fd ?
Chaque descripteur de fichier est un index qui fait référence à une entrée dans la table interne des descripteurs de fichiers (gérée par le système d'exploitation). Cette entrée pointe vers des informations sur un fichier ouvert ou un autre objet d'entrée/sortie.  
En très gros (et un peu faux), un fd pointe vers un fichier ouvert. 
#

```c
int execve(const char *filename, char *const argv[], char *const envp[])
```
Exécute le programme correspondant au fichier pointé par filename, en utilisant naturellement les descripteurs 0 et 1 que nous appellerons ici **fd0** et **fd1**.   
Il n'est pas possible de modifier ce comportement.
#

```c
int pipe(int pipefd[2])
```
`pipefd[0]` = extrémité de lecture du pipe.  
`pipefd[1]` = extrémité d'écriture du pipe.
#

```c
int dup2(int oldfd, int newfd)
```
Newfd fait désormais référence au fichier pointé par oldfd.  
(C'est de plus en plus imprécis, mais en gros, nous dirons que newfd prend la valeur de oldfd.)
#

#### Exécution : 

Avant le traitement de ma première commande :    
- Je conserve la valeur actuelle de fd0 dans une tmp pour ne pas la perdre.
- Fd0 prend la valeur de mon fichier infile.

Pour chaque commande :  
 - `pipe(fd);` crée un pipe.
 - `pid = fork();` lance un processus enfant dont l'identifiant est stocké dans `pid`.

 - Suite pour tous mes processus : 
   - Si je suis le parent :
     - fd0 prend la valeur de mon pipe (`pipefd[0]` : je lis dans le pipe). *Attention, cette ligne n'a pas d'impact sur les processus enfants déjà en exécution. Ce fd0 sera hérité par le prochain processus enfant issu du prochain fork.*


    - Si je suis un enfant  `if (pid == 0)`:

      >  *Rappel :*
      >  - *Si je traite la première commande : mon fd0 est l'infile.*
      >  - *Sinon : mon fd0 est l'extrémité de lecture du pipe de la commande précédente.*

        - Si je traite la dernière commande : mon fd1 prend mon outfile.
      - Sinon : mon fd1 prend mon pipe (`pipefd[1]` : j'écris dans le pipe).
      - J'exécute.
      - J'exit.
#
