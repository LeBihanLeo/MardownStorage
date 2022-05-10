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
