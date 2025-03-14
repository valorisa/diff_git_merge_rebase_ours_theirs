# Comprendre Git : Merge vs. Rebase et --ours vs. --theirs par l'exemple

Ce document présente un exemple concret pour illustrer les différences fondamentales entre les commandes '```merge```' et '```rebase```' de Git, ainsi que la signification contextuelle des termes '```--ours```' et '```--theirs```' lors de la résolution de conflits.

## Mise en place du scénario

Pour bien comprendre ces concepts, nous allons créer un scénario simple mais complet. Voici les étapes que nous suivrons :

### Création du dépôt et de la structure initiale

Commençons par créer un nouveau dépôt et y ajouter quelques fichiers :

```
mkdir exemple-git
cd exemple-git
git init
echo "# Projet démo" > README.md
echo "ligne 1" > fichier.txt
echo "ligne 2" >> fichier.txt
echo "ligne 3" >> fichier.txt
git add .
git commit -m "Premier commit - structure initiale"
```

Nous avons maintenant un dépôt avec un commit initial contenant deux fichiers : README.md et fichier.txt. Le fichier.txt contient trois lignes simples.

### Création d'une branche feature

Maintenant, créons une branche feature pour développer une nouvelle fonctionnalité :

```
git checkout -b feature
```

Sur cette branche, modifions fichier.txt :

```
echo "ligne 1 - modifiée par feature" > fichier.txt
echo "ligne 2" >> fichier.txt
echo "ligne 3" >> fichier.txt
echo "ligne 4 - ajoutée par feature" >> fichier.txt
git commit -am "Modification sur la branche feature"
```

### Modification de la branche principale

Revenons maintenant sur la branche principale (main) et faisons-y également des modifications :

```
git checkout main
echo "ligne 1" > fichier.txt
echo "ligne 2 - modifiée par main" >> fichier.txt
echo "ligne 3" >> fichier.txt
git commit -am "Modification sur la branche main"
```

À ce stade, nous avons deux branches qui ont divergé, chacune avec ses propres modifications sur le même fichier. C'est exactement le type de situation qui peut mener à des conflits lors d'une intégration.

## Scénario 1 : Utilisation de Merge

Dans ce premier scénario, nous allons utiliser la commande  pour intégrer les changements de la branche feature dans la branche main.

```
git checkout main
git merge feature
```

Cette commande tentera de fusionner automatiquement les modifications, mais dans notre cas, elle génèrera un conflit puisque les deux branches ont modifié la même partie du fichier.txt :

```
Auto-merging fichier.txt
CONFLICT (content): Merge conflict in fichier.txt
Automatic merge failed; fix conflicts and then commit the result.
```

### Résolution de conflit avec merge

En ouvrant fichier.txt, nous verrons quelque chose comme :

```
<<<<<<< HEAD
ligne 1
ligne 2 - modifiée par main
=======
ligne 1 - modifiée par feature
ligne 2
>>>>>>> feature
ligne 3
```

C'est ici que les concepts de '```--ours```' et '```--theirs```'  entrent en jeu :
- **ours** (le nôtre) : fait référence à la version de la branche actuelle (main)
- **theirs** (le leur) : fait référence à la version de la branche que nous intégrons (feature)

Pour résoudre ce conflit en utilisant la version de notre branche actuelle (main) :

```
git checkout --ours fichier.txt
git add fichier.txt
git commit -m "Résolution de conflit en faveur de main"
```

Si nous avions voulu garder la version de la branche feature :

```
git checkout --theirs fichier.txt
git add fichier.txt
git commit -m "Résolution de conflit en faveur de feature"
```

## Scénario 2 : Utilisation de Rebase

Pour illustrer le rebase, revenons d'abord à l'état initial (avant le merge) :

```
git reset --hard HEAD~1  # Annule le dernier commit (celui du merge)
```

Maintenant, au lieu de merger, nous allons utiliser rebase depuis la branche feature :

```
git checkout feature
git rebase main
```

Comme avec le merge, Git signalera un conflit :

```
Auto-merging fichier.txt
CONFLICT (content): Merge conflict in fichier.txt
error: could not apply abcd123... Modification sur la branche feature
```

### Résolution de conflit avec rebase

C'est ici que la différence cruciale apparaît. Dans le contexte d'un rebase :
- **ours** : fait référence à la branche CIBLE du rebase (main)
- **theirs** : fait référence à VOTRE branche en cours de rebasage (feature)

Cette inversion par rapport au merge peut être source de confusion. Si nous voulons garder les modifications de notre branche feature :

```
git checkout --theirs fichier.txt
git add fichier.txt
git rebase --continue
```

Si nous voulions plutôt conserver les modifications de la branche main :

```
git checkout --ours fichier.txt
git add fichier.txt
git rebase --continue
```

## Différences fondamentales entre Merge et Rebase

### Comportement de Merge

Lors d'un merge, Git crée un nouveau commit de fusion qui combine les historiques des deux branches. L'historique ressemblera à ceci :
```
A---B---C---E (main)
     \     /
      D---F (feature)
```

Les deux branches restent intactes et leur historique est préservé. Le commit E est un commit de fusion qui incorpore les changements des deux branches.

### Comportement de Rebase

Avec un rebase, l'historique de votre branche est réécrit. Le rebase prend les commits de votre branche et les "rejoue" sur la pointe de la branche cible :
```
A---B---C (main)
         \
          D'---F' (feature)
```

Les commits D et F ont été "rejoués" (recréés) sur la pointe de main, donnant les nouveaux commits D' et F'. L'historique devient linéaire, comme si vous aviez créé la branche feature à partir du dernier commit de main.

## Conclusion

La distinction entre merge et rebase reflète deux philosophies différentes d'intégration de code :
- Le merge préserve l'historique exact et crée un commit de fusion.
- Le rebase réécrit l'historique pour créer une ligne de développement plus propre et linéaire.

Quant aux termes **ours** et **theirs**, leur signification contextuelle change selon l'opération en cours :
- Dans un merge, **ours** représente la branche actuelle et **theirs** la branche mergée.
- Dans un rebase, **ours** représente la branche cible et **theirs** votre branche en cours de rebasage.

Cette différence est cruciale pour résoudre efficacement les conflits, et sa méconnaissance peut conduire à des erreurs lors de l'intégration du code.
