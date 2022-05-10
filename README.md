# MardownStorage
Storage for usefull markdown
<br>
Jni Java and C
<br>
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

<br>

# Echanger des chaînes de caractères en Java et C/C++ et vice versa
### 1) Java String to C++ char*:
```cpp
JNIEXPORT void JNICALL
Java_HelloWorld_printStringToCpp(JNIEnv *env, jclass cl, jstring str)
{
 // Construction d'une chaîne C/C++ à partir d'une chaîne Java
 const char *cStr = env->GetStringUTFChars(str,0);
 // Affichage de la chaîne C++
 printf("%s\n", cStr);
 // Libération de la chaîne C++
 env->ReleaseStringUTFChars(str,cStr);
}
```
**JNIEnv \*env** référence toujours l'environnement par lequel nous obtiendrons les fonctions du support JNI. 
La méthode prend en entrée, l'env, la classe et le string à convertir. On utilise la méthode "GetStringUTFChars()" afin de transformer le **String** en **char**\*:

```java	
 // Construction d'une chaîne C/C++ à partir d'une chaîne Java
 const char *cStr = env->GetStringUTFChars(str,0);
```		
Ensuite on print la chaine de charactère et on libère la variable avec:
```java		
 // Libération de la chaîne C++
 env->ReleaseStringUTFChars(str,cStr);
```	

### 2) C++ char* to Java String
```cpp
JNIEXPORT jstring JNICALL
Java_HelloWorld_stringFromCpp(JNIEnv *env, jobject obj)
{
 // Construction d'une chaîne Java à partir d'une chaîne C/C++
 return env->NewStringUTF("Chaîne en C");
}
```
Cette fois ci on utilise la méthode "NewStringUTF()" qui prend un char* en entrée et renvoi un jstring, c'est à dire un string Java.

<br>

# Invoquer une méthode sur un objet Java depuis C/C++
```cpp
JNIEXPORT void JNICALL
Java_HelloWorld_callJavaMethod(JNIEnv *env, jobject obj) {
	 // Récupération d'un objet de Metaclasse
	 jclass cls = env->GetObjectClass(obj);
	 // Calcule de l'identificateur de "void test(String str)"
	 jmethodID mid = env->GetStaticMethodID(cls,"test","(Ljava/lang/String;)V");
	 if (mid == 0) {
		 // Ca a planté !!!
		 fprintf(stderr, "Ouille, ça a planté !");
	 } else {
		 // Tout va bien, l'appel peut aboutir.
		 jstring str = env->NewStringUTF("Ceci est un paramètre créé en C/C++");
		 env->CallVoidMethod(obj,mid,str);
	 }
	 return;
}
```
### 1) Ce que l'on veut faire
```java	
env->CallVoidMethod(obj,mid,str);
```
On souhaite utiliser une méthode d'un objet java en C/C++, pour ca nous allons utiliser la méthode "CallVoidMethod()". Cette méthode prend en paramètre:
- Un **jobject** qui est l'objet qui possède la méthode que l'on souhaite utiliser
- Un **jmethodID** qui corespond l'identificateur de la méthode que l'on souhaite utiliser
- Un str est le string qui est un paramètre de la méthode.

Il faut donc réusir à obtenir le **jmethodID** car nous ne le possédons pas.

### 2) Obtenir le jmethodID, l'identificateur de la méthode que l'on souhaite utiliser
```java	
jmethodID mid = env->GetStaticMethodID(cls,"test","(Ljava/lang/String;)V");
```
La méthode "GetStaticMethodID()" à besoins de 3 paramètres:
- Un **jclass** qui est un objet de Metaclasse, on l'obtien avec: `jclass cls = env->GetObjectClass(obj);`
- Une chaine de charactère qui corespond au nom de la méthode que l'on souhaite utiliser
- Une chaine de character qui décris le **prototype** complet de la méthode (nous allons voir par le suite comment l'obtenir facilement)

Il y a donc deux choses que l'on doit déterminer:
- Obtenir le **jclass** de notre objet
- Obtenir la description du **prototype** de notre méthode

<br/>


#### 1. Obtenir le jclass de notre objet
Un **jclass** qui est un objet de Metaclasse, on l'obtien avec la méthode "GetObjectClass(obj)" ou objet est notre jobject passé en paramètre: 
```java	
// Récupération d'un objet de Metaclasse
jclass cls = env->GetObjectClass(obj);
```

<br/>

#### 2. Obtenir la description du prototype de notre méthode
C'est très facile, imaginons que notre fichier HelloWorld ressemble à ca:
```java	
    public class HelloWorld {
     public static native void printCpp(); // Déclaration prototype méthode native 
	     static { System.loadLibrary("HelloWorld"); // Chargement de la bibliothèque 
     } 
     public static void main(String args[]) { 
	     System.out.print("Hello "); // Affiche Hello en Java 
	     HelloWorld.printCpp(); // Affiche World en C/C++ 
     }
	//La fameuse méthode test, qui prend en paramètre un String et qui renvoi un void
	 public static void test(String str){}
}
```

Il faut compiler le fichier java pour obtenir le .class donc on va utiliser javac dans le shell:


```SQL
'javac HelloWorld.java'
```
Puis il suffit d'utiliser la commande javap dans le shell:
```SQL
'javap -s HelloWorld'
```

Et on va obtenir les **prototypes** de toutes les méthodes de notre fichier:

```SQL
'D:> javap -s HelloWorld
Compiled from "HelloWorld.java"
public class HelloWorld {
  public HelloWorld();
    descriptor: ()V

  public static native void printCpp();
    descriptor: ()V

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V

  public static void test(java.lang.String);
    descriptor: (Ljava/lang/String;)V

  static {};
    descriptor: ()V
}'
```
On remarque que le prototype de test est "**(Ljava/lang/String;)V**":
```java	
public static void test(java.lang.String);
descriptor: (Ljava/lang/String;)V
```
On a donc notre **prototype** complet et tout ce qui nous faut pour obtenir le **jmethodID**.

### 3) On met tout ensemble

On a donc un **jclass**, et le **prototype** complet de notre méthode, on peut obtenir le **jmethodID**:
```java	
jmethodID mid = env->GetStaticMethodID(cls,"test","(Ljava/lang/String;)V");
```
Maintenant qu'on a le **jmethodID**, on peut appeller la méthode test depuis notre code C/C++:
```java	
env->CallVoidMethod(obj,mid,str);
```
Et voilà on a réussis à appeller une méthode d'un de nos objet Java depuis notre code C/C++.

<br>

# Accéder à un attribut d'un objet Java depuis C/C++

```cpp
JNIEXPORT void JNICALL
Java_HelloWorld_toString(JNIEnv *env, jobject obj) {

// Obtention de la Metaclasse de HelloWorld
 jclass cls = env->GetObjectClass(obj);
 // Calcul de l'identificateur de l'attribut entier de type int
 jfieldID fid = env->GetFieldID(cls,"entier","I");
 
 // Récupération de la valeur entière de l'attribut
 int a = env->GetIntField(obj,fid);
 // Modification de la valeur entière de l'attribut
 env->SetIntField(obj, fid, a + 1);
}
```
Pour récuper un attribut c'est assez similaire que pour une méthode:
1. Récuperer l'objet jclass
2. Récuperer l'objet jfieldID
3. Utiliser les méthodes fournis par env pour utiliser l'objet (GetFieldID(), SetIntField(), ...)

Cette fois on récupère un jfieldID en utilisant la commande "GetFieldID()":
```java	
jfieldID fid = env->GetFieldID(cls,"entier","I");
```
On retrouve ici aussi l'objet cls qui est un jclass, puis une chaine de charactère "entier", puis "I".
Le  chaine de charactère "entier" corespond au nom de l'attribut, notre classe java doit posséder un attribut nommé entier, on aurait pu le remplacer par "roue" si on utilisait la classe voiture.
La chaine "I" indique que l'attribut est un int:

Codage | Signature
 ---: | :---: 
Z | boolean
B | byte
C | char
S | short
I | int
J | long
F | float
D | Double
LclassName indique une classe | ex: Ljava/lang/String; pour la classe String
[type indique un tableau | ex: [I pour un tableau d'entier
(args-types)retType indique un prototype de méthode | ex : (Ljava/lang/String)V

Une fois que l'on a récupéré le jfieldID on peut l'utiliser avec env : 
```cpp
 // Récupération de la valeur entière de l'attribut
 int a = env->GetIntField(obj,fid);
 // Modification de la valeur entière de l'attribut
 env->SetIntField(obj, fid, a + 1);
```
