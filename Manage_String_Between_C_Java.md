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
**JNIEnv *env** référence toujours l'environnement par lequel nous obtiendrons les fonctions du support JNI. 
La méthode prend en entrée, l'env, la classe et le string à convertir. On utilise la méthode "GetStringUTFChars()" afin de transformer le **String** en **char***:

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
