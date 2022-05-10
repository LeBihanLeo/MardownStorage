# Les pipes 
## Les pipes anonymes
```c
int main(){
	int a[2];
	char buff[10];
	pipe(a);
	write(a[1], "code", 5);
	read(a[0], buff, 5);
	printf("%s\n", buff);
	return 0;
}
```
Voici le code pour créer un pipeline anonyme, pour cela on créé un tableau d'int de taille 2 et on l'utilise avec la méthode "pipe()", puis 2 options s'offrent à nous:
1. Ecrire dans le pipe en utilisant write sur a[1] 
2. Lire depuis le pipe en utilisant read sur a[0] 

Cette mécanique devient intéressante lorsqu'on la couple avec un fork():
```c
int main(){
	int a[2];
	if(pipe(a) == -1){
		perror("pipe");
		exit(-1);
	}
	int pid = fork();
	if(pid == -1){
		perror("fork");
		exit(-1);
	}	
	else if(pid == 0){//fils
		char* msg="Je suis le fils";
		write(a[1], msg, 512*sizeof(char));
		close(a[1]);
		printf("Fin du fils, msg = %s\n", msg);
		exit(0);
	}
	else{//père
		char buff[512];
		wait(NULL);
		read(a[0], buff, 512);
		printf("Lecture: %s\n", buff);
	}
	return 0;
}

```
#### Que fait-on dans ce code: 
1. On créé notre tableau de taille, on utilise pipe et on vérifie à travers ce que renvoi pipe s'il n'y a pas eu un problème lors de la création du pipe, si pipe retourne -1 il y a eu un souci le code s'interrompt.
2. On réalise notre fork(), même chose que pour le pipe mais avec les spécificité du fork:
	- Si ca retourne -1 le code s'interrompt
	- Si ca renvoit 0 alors c'est le fils, on décide qu'il va écrire dans le pipe
	- Sinon c'est le père, il va lire dans le pipe et print le résultat

Tout comme dans le code d'avant pour écrire on utilise write sur a[1], et pour lire read sur a[0]. De plus à la fin de l'écriture **on rajoute un close(a[1])**, afin de fermer le descripteur de fichier et ca va permettre de **définir une fin de fichier**.
<br>
Et voilà le fils écrit sur a[1], une fois qu'il a fini il close(a[1]) et exit(0) pour se fermer.
Le père attend la fin du fils avec un wait(NULL), puis lis dans a[0] le contenu de ce qu'à écris le fils, et affiche le résultat.
Il est important de retenir qu'après un appel à **pipe()** a[0] et a[1] sont des **descripteurs de fichiers** que l'on peut accéder à travers **read()/write()**, tout comme on peut accéder à un fichier de la même manière avec son descripteur de fichier obtenu avec la commande **open()**

### Une solution proposée pour la question 4 de l'annale de 2018
Écrire le programme C qui créé deux processus P1 et P2 qui communiquent par un tube anonyme. Le processus P1 ouvre un fichier donné en argument au programme et tranfer le contenu du fichier au processus P2 en transformant chaque caractère en majuscules. Le processus P2 écrit ce contenu dans le deuxième fichier passé en paramètre du programme
```c
#include <unistd.h>
#include <stdlib.h>
#include <string.h>
#include <stdio.h>
#include <sys/wait.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <ctype.h>
#define MAX 255


//faire en fork pour créer 2 processus
//créer un tube anonyme pour que les deux processus communique
//P1 ouvre un fichier, met tout le contenue du fichier en uppercase, et transmet à travers le pipe anonyme le CONTENUE du fichier
//P2 lis le contenu à travers le pipe, et écris sur le contenue sur le fichier2 passé en paramètre

//ouvrir f1 et écrit dans p en uppercase
//lis p et écrit dans f2


void openFileWriteContentInUpperCaseIntoPipe(char* file, int a[]){
	printf("Start openFileWriteContentInUpperCaseIntoPipe\n");
	close(a[0]);
	char buff[MAX];
	int n=0;
	int fd = open(file, O_RDONLY);
	if(fd == -1){
		perror("open");
		exit(-1);
	}
	//read renvoit le nbr d'octet lu
	//donc pour write on va utiliser le resultat de read pour indiquer la taille de ce que l'on envoi
	while((n=read(fd, buff, 1))!=0){
		buff[0] = toupper(buff[0]);
		write(a[1], buff, n);
	}
	close(a[1]);
	close(fd);
	printf("End openFileWriteContentInUpperCaseIntoPipe\n");
}

void readPipeAndWriteContentIntoFile(char* file, int a[]){
	printf("Start readPipeAndWriteContentIntoFile\n");
	close(a[1]);
	char buff[MAX];
	int n = 0;
	int fd = open(file, O_WRONLY);
	if(fd == -1){
		perror("open");
		exit(-1);
	}
	while((n = read(a[0], buff, MAX)) !=0){
		write(fd, buff, n);
	}
	printf("End readPipeAndWriteContentIntoFile\n");
}

//commande fichier1 fichier2
int main(int argv, char** argc){
	char* file1 = argc[1];
	char* file2 = argc[2];

	int a[2];
	if(pipe(a) == -1){
		perror("pipe");
		exit(-1);
	}
	
	int pid = fork();
	if(pid == -1){
		perror("fork");
		exit(-1);
	}
	else if(pid==0){//fils P1
		openFileWriteContentInUpperCaseIntoPipe(file1, a);
	}
	else{//père P2
		wait(NULL);
		readPipeAndWriteContentIntoFile(file2, a);
	}
	return 0;
}

```
