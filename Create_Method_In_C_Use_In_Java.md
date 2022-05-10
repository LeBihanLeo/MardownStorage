# Créer une méthode C/C++ et l`utiliser en java:
### 1) Créer le fichier java qui fait appelle à la méthode C/C++ "HelloWorld.printCpp()":
```java
public class HelloWorld {
     public static native void printCpp(); // Déclaration prototype méthode native 
	     static { System.loadLibrary("HelloWorld"); // Chargement de la bibliothèque 
     } 
     public static void main(String args[]) { 
	     System.out.print("Hello "); // Affiche Hello en Java 
	     HelloWorld.printCpp(); // Affiche World en C/C++ 
     }
}
```
### 2) Générer le fichier .hb avec la commande suivante:
```sql
'javac -h ./dirOfHelloWorld HelloWorld.java'
```
Dans le fichier .h généré on peut retrouver cette ligne qui définis la signature de la méthode que l'on va devoir implémenter en C++:
`JNIEXPORT void JNICALL Java_HelloWorld_printCpp(JNIEnv *, jclass);`

### 3) On code la méthode HelloWorld_printCpp:
```cpp
#include "HelloWorld.h"
#include <stdio.h>
JNIEXPORT void JNICALL Java_HelloWorld_printCpp(JNIEnv *env, jclass cls) {
	printf("World !\n");
}
```
### 4) Exécution du programme
Une fois la méthode implémentée on génère la bibliothèque dynamique (HelloWorld.dll), on place la dll avec le fichier .class, et on lance notre fichier java:
```sql
'java HelloWorld.java'
```
