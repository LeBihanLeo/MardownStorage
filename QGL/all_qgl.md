# QGL Révision Exam Final

## Principe SOLID:
**S -> Single responsability**
Une classe sert à une chose 

**O-> Open/Closed**
Ouverte aux extensions mais fermé aux modifications

**L -> Liskov substitution**
Si dans une méthode on remplace une classe par une de ses soeur ou sa mère, la méthode doit fonctionner

**I -> Interface segregation**
Un client ne devrait jamais être forcé d'implémenter d'une interface qu'il n'utilise pas ou de dépendre d'une méthode qu'il n'utilise pas

**D -> Dependency inversion**
Les modules de haut niveau doivent dépendre d'abstraction et non d'objet concret


## Les Anagrames importants:
KISS --> Keep It Simple Stupid
DRY --> Don't Repeat Yourself
YAGNI --> Your aren't Gonna Need It

## Les Métriques:
Il en existe beaucoup:
- **CYCLO**: compte le nombre de chemins indépendant à travers le code d'une fonction 
	- Aide à déterminer le nbr minimum de test
- **WMC**: Calcul la compléxité des méthodes d'une classe (utilise souvent CYCLO) 
	- Configurable donc adaptable à nos besoins
- **DIT**: Calcul la profondeur maximum d'une classe dans un hierarchie de class 
	-  Mesure la qualité de l'héritage du projet
- **CBO**: Montre pour une classe A, le nombre de d'autre classe qui acèdent aux méthodes/attributs de la classe A
	- Met en lumière les vrais dépendances pas seulement celles déclarées
- **TTC**: Compte le nombre de méthodes qui accèdent à des attributs communs 
	- Permet de vérifier la cohérence de la classe, une classe cohérente a une seule responsabilité donc les méthode vont accéder aux mêmes attributs, score de cohésion fort. Une classe peu cohéente a plusieurs responsabilités, les méthodes vont accéder à peu d'attributs en commun, score de cohésion faible.

### Le problèmes des métriques:
#### 1) La granularité des métriques
Capturer les symptomes, non les causes des problèmes, n'amène vers une solution d'amélioration

#### 2) Mapping implicite
On en pense pas en terme de métrique mais en terme de conception. Casser certaines convention déteriore le code et amène à des problèmes de conception

L'utilisation de seuil dans les métriques les rendent difficile à interpreter.

Detection strategies
C'est utiliser plusieurs métriques à la fois pour trouver des problèmes

### Exemple de mauvaise pratique détectable avec des métriques, les God Classes:
Les **God Classes** centralisent l'intelligence du système, pour tout faire et utiliser les données de plus petites classes
- Complexe
- Non cohésive (cohérente, elle sert à plusieurs choses)
- Accèdent à des données externes


## SONAR
### Objectif:
- Fournir une analyse complète de la qualité d'un projet de développement à prtir de métrique
- Evaluer la qualité du code
- En connaître l'évolution au cours du développement

### Structure:
Un runner qui lance des annalyse complètes et compile les résultats
Une BDD qui stocke les analyses
Un serveur web pour consulter les données

### Métrique que SONAR mesure:
- Nombre de ligne
- Nombre de Bugs
- Nombre de vulnérabilités (problèmes de sécurité)
- Le nombre d'heure/ jours de dette technique 
- Le nombre de Code smells (niveau de code smells: blocker, critical, major, minor, info)
	- Nous permet de supprimer les mauvaises pratique et ainsi garder un code de qualité
- Test Coverage
	- permet de nous donné une idée de si nos tests couvre bien l'entierté du code
- Code duplication 
	- Lorque l'on duplique du code cela peut souligner le besoins d'une nouvelle classe

#### Pourquoi est-il pertinent d'utiliser des représentation visuel du code comme la ville de Sonar pour évaluer notre projet ?
On utilise beaucoup des représentations visuels de notre code afin de détecter des patterns qui soulignent des problèmes de conception.

### Les PiTest, les mutations
L'avantage de tester les mutation c'est ce que cela nous fait prendre conscience que nos tests ne sont souvent pas suffisant et peuvent laisser passer bcp de beug. Il nous permet donc d'identifier là ou il manque des tests ou même là ou nos tests sont mal concus.

#### Avantages:
- Identifier les parties du code pas assez testé
- Identifier les tests mal concus

## Git:
### Les commandes de bases:
```sql
'- $> git clone https://....
- $> git pull
- $> git add .
- $> git commit -m "msg de mon commit"
- $> git push
- $> git tag -d WEEK8 (Puis git push origin WEEK8)'
```
### Les branches:
Dans git on a des branches, chaque branches contient des commits, on se déplace à travers ces commit avec un pointeur appelé **HEAD**.

#### Commande git en rapport avec les branches:

```sql
'- $> git checkout branch1 (se déplacer sur la branch1)
- $> git checkout -b  branch2 (créer la branch2 et se déplace dessus)
- $> git merge branch2 (On suppose que l\'on est sur master et cela va merge branch2 sur master)
- $> git rebase master (On suppose être sur branch2, cela va faire en sorte de prendre branch2 et la faire commencer au bout de master avec tout les changement de master)
-$> git cherry-pick 14597CECBO (On est sur branch2 et on va récuper le commit 14597CECBO et récuper les changement de ce commit, une feature dont on a besoin ou un bug fixé)'

```
#### Les pull requests:
C'est lorsqu'un developpeur a réalisé une feature du projet et demande aux autres developpeurs l'autorisation d'ajouter la feature sur une branche. Cela implique que les autres membres regardent le contenue de la feature avant de valider l'ajout

##### Avantage:
- Cela permet à l'ensemble des membres du groupe d'avoir un visuel sur les nouvelles feature ajoutée
- Dans une équipe qui a son code open-source la pull request peut permettre à d'autre developper de proposer de nouvelles améliorations du code, il fera donc une pull request que l'équipe devra valider.

#### Pourquoi utiliser des branches:
Garder un projet stable en isolant les nouvelles implémentations

### Branching stratégie:
La branching stratégie corespond à une convention autour de laquelle l'équipe s'est mise d'accord afin d'organiser la manière dont ils vont collaborer sur git. On peut définis des branches, qui seront plus ou moins à jour et plus ou moins stable. De plus le nom des branches peut être codifié (exemple feature/ajout_des_marins#12, ici notre branche est une feature qui va corespondre à l'issue 12 d'ajout des marins).

#### Les avantages de la branching stratégie:
- Avoir la même convention de nommage des branches à travers l'équipes et tout au long du projet -> si le projet est repris par une autre équipes ce sera facile de comprendre la signification des branches
- Cela permet de garder toujours une version stable du code et isoler les les nouveaux changements.

### Tag:
Un tag corespond à étiquette que l'on place sur un commit afin de pouvoir accéder à cette version du code facilement



## Automatisation et gitAction:
#### Les grandes étapes admises entre écriture du code et utilisation du programme en production (la production c'est le programme stable que l'on fournis aux utilisteurs):
- Compilation
- Lancement des tests
- Empaquetage (Création du jar)
- Le déploiement

#### C'est quoi un automate ?
Un automate réalise une tâche que l'on a automatisé:
-> une tâche réalisé par un humain
-> cher
-> récurrente

## Maven:
Maven utilise un pom.xml pour définir la structure d'un projet mais aussi toutes ses dépendance. 
#### Dans un projet Maven on définis:
- Les informations concernant notre projet (nom du projet, artifactId, version, packaging)
- Les propriété du projet (version de java, version de maven, version des autres dépendances)
- Les dépendances (Jackson)
- Les plugins(Sonar, PiTest) 

#### Maven sans connexion internet ?
Maven utilise des dépendance, et pour soit elle va les chercher dans un dépot en ligne soit en local, si tout se dont Maven à besoin se trouve dans le dépot local alors oui Maven pourra marcher sans Internet

#### Différence entre une dépendance et un plugin dans Maven ?
Dépendance corespond à une librairie qui va être utiliser directement dans le code -> Jackson pour les json
Un plugin est un outil utilisé par Maven -> Sonar et PiTest

#### Lancer les test et construire le jar:
```sql
'$> mvn clean package'
```

#### Lancer un commande suivant un profil particulier:
```sql
'$> mvn clean/test/ceketuve -PmonProfil'
```
Exemple, je veux lancer un clean package sur le profil testRapide:
```sql
'$> mvn clean package -PtestRapide'
```
