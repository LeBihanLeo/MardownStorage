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
