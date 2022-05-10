# Les pipes
## Les pipes nommé
A la différences des pipes anonymes, les pipes nommés sont des vrais fichiers présent sur la machine. On créé un nouveau pipe nommé de deux manière:
1. Dans le shell avec la commande 
``` 
'$ mkfifo ./monPipe'
```

2. Dans le code C avec la méthode
```c 
int mkfifo(const char *pathname, mode_t mode);
```

Quelque soit la méthode cela vient créé un fichier destiné à être utilisé comme pipe nommé.
### Comment on utilises les pipes nommés et c'est quoi l'avantage?
Alors tout comme pour les pipe anonymes, on va venir read/write, mais cette fois ci sur le fichier créé avec mkfifo. Read et Write on besoins d'un descripteur de fichier et pour ca nous allons utiliser open() sur le fichier qu'on a créé précédement.
L'avantage des tubes nommés c'est qu'on peut les utiliser à travers différents processus sans lien de parenté (pas besoin de fork()). Donc je lance un processus P1 d'un coté, un processus P2 de l'autre, et ils peuvent communiquer à travers le pipe.
#### Exemple ecrivain / lecteur td6
*ecrivain.c*
```c
#define MAX 255

int main(){
	printf("Début écrivain\n");
	char* msg = "Je suis l'ecrivain";
	int fd = open("/tmp/myPipe", O_WRONLY);
	if(fd == -1){
		perror("open");
		exit(-1);
	}

	if(write(fd, msg, MAX) == -1){
		perror("open");
		exit(-1);
	}
	printf("Fin de l'écriture.\n");
	return 0;
}

```

*lecteur.c*
```c
#define MAX 255

int main(){
	int fd = open("/tmp/myPipe", O_RDONLY);
	if(fd == -1){
		perror("open");
		exit(-1);
	}
	char buff[MAX];
	read(fd, buff, MAX);
	printf("Lecture: %s\n", buff);
	return 0;
}

```

On a deux deux programme différent:
- Le premier, l'écrivain va **ouvrir le fichier pipe en mode écriture**, afin d'**écrire** du texte.
- Le deuxième, le lecteur va **ouvrir le fichier pipe en mode lecture**, afin d'en **lire** le contenu

Les deux processus font donc appelle à la méthode open():
```c
int open(const char *pathname, int flags);
```
On a donc en entré, le path vers le fichier que l'on souhaite ouvrir, et un flag qui corespond au mode pour ouvrir le fichier par exemple:
Codage | Signification
 ---: | :---: 
O_RDONLY | Open for reading only.
O_WRONLY | Open for writing only.
O_CREAT | Create the file if he doesn't exist
On peut utiliser plusieurs flags en utilisant des '|' de la manière suivante:
```c
int fd = open("dir/myPipe", O_WRONLY|O_CREAT)
```
