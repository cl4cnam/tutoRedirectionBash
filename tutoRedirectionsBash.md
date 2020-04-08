Vous avez entendu parler des redirections du shell bash. Il s'agit de trucs du genre ***`monProgramme >unFichier 2>&1`***. J'essaie de vous expliquer comment ça marche.

***`Une redirection, c'est quoi ?`***
===========================

En fait :heavy_check_mark:
--------------
C'est une ouverture de fichier (rien de plus !)

Fausse idée :x:
--------------
- Ça prend la sortie (ou bien l'entrée) d'un programme
- et ça la redirige

***`Comment ça se passe ?`***
===========================

En fait :heavy_check_mark:
--------------
1. L'utilisateur lance un terminal et rentre son login
	1. Le shell bash s'exécute dans le terminal
	1. Le shell bash ouvre (en lecture-écriture) un fichier spécial représentant le terminal et l'indique dans **`desc[0]`** (on peut considérer que **`desc`** est une variable Array interne inaccessible)
	1. Le shell bash fait **`desc[1] = desc[0]`**
	1. Le shell bash fait **`desc[2] = desc[0]`**
1. L'utilisateur tape une commande avec (éventuellement) des redirections
	1. Les redirections (c.-à-d. les ouvertures de fichiers) sont faites,
	1. La commande est exécutée
	1. Les fichiers ouverts (dans l'étape 'i.') sont fermés

Fausse idée :x:
--------------
- La commande est exécutée
- mais les redirections interceptent les flux d'entrées et de sorties

***`Les programmes écrivent/lisent où ?`***
===========================
Où les programmes en console écrivent/lisent-ils en général quand ils font une commande genre ***`print`*** ou bien genre ***`input`*** (en python) ?

En fait :heavy_check_mark:
--------------
- Ils écrivent sur le fichier ouvert indiqué dans **`desc[1]`**
- Ils lisent sur le fichier ouvert indiqué dans **`desc[0]`**

Fausse idée :x:
--------------
- Ils écrivent sur le terminal
- Ils lisent sur le clavier

***`Étrange !`***
===========================
C'est pas du bidon tout ça ? C'est contraire à ce qu'on a appris, non ?

Voyons tout cela plus précisément :

***`Plus précisément`***
================
- ***`>cheminFichier`*** : ouvre *cheminFichier* en écriture (mode écrasement ou création) et l'indique dans **`desc[1]`**
- ***`n>cheminFichier`*** : ouvre *cheminFichier* en écriture (mode écrasement ou création) et l'indique dans **`desc[n]`**
- ***`n>>cheminFichier`*** : ouvre *cheminFichier* en écriture (mode ajout) et l'indique dans **`desc[n]`**
- ***`<cheminFichier`*** : ouvre *cheminFichier* en lecture et l'indique dans **`desc[0]`**
- ***`n<cheminFichier`*** : ouvre *cheminFichier* en lecture et l'indique dans **`desc[n]`**
- ***`n<>cheminFichier`*** : ouvre *cheminFichier* en lecture et l'indique dans **`desc[n]`**
- ***`n>&m`*** : **`desc[n] = desc[m]`**
- ***`n<&m`*** : idem (**`desc[n] = desc[m]`**)
- ***`n>&-`*** : ferme le fichier ouvert indiqué dans **`desc[n]`** et fait **`desc[n] = null`**
- ***`n<&-`*** : idem (ferme le fichier ouvert indiqué dans **`desc[n]`** et fait **`desc[n] = null`**)

***`Exemples`***
========

Exemple 1
----------

Voici un fichier '*essai.py*' écrit en python3 :
```python
def ecrireSurFichierDitDansDesc(num, texte):
	f = open(num, 'w')
	f.write(texte)

ecrireSurFichierDitDansDesc(4, 'Bonjour !')
```

Si on l'exécute directement
```shell
python3 essai.py
```
on obtient :
```
OSError: [Errno 9] Bad file descriptor
```
ce qui est normal, puisqu'il n'y a pas de fichier indiqué dans **`desc[4]`**

Maintenant, si on exécute
```shell
python3 essai.py 4>essai.out
```
ça marche, on obtient '**`Bonjour !`**' dans le fichier '*essai.out*'

### Remarque

- On peut aussi écrire les "redirections" en début de commande :
```shell
4>essai.out python3 essai.py
```
(ce qui est plus conforme à l'ordre des opérations)

- Plus étonnant, on peut écrire les redirections au milieu de la commande :
```shell
python3 4>essai.out essai.py'
```

Exemple 2
----------

Voici un fichier '*essai2.py*' écrit en python3 :
```python
def getCheminFichierTerminal():
	import subprocess as subPr
	cheminFichierTerminal = subPr.check_output('tty', universal_newlines=True)
	return cheminFichierTerminal[:-1]

def ecrireSurTerminal(texte):
	f = open(getCheminFichierTerminal(), 'w')
	f.write(texte + '\n')

ecrireSurTerminal('coucou')
```
Ce programme écrit vraiment sur le terminal (ce qui est très inhabituel).<br><br>

Si on l'exécute directement
```shell
python3 essai2.py
```
on obtient (sur le terminal) :
```
coucou
```

Si on "redirige" la sortie
```shell
python3 essai2.py >essai2.out 2>&1
```
on obtient encore (sur le terminal) :
```
coucou
```

Les "redirections" n'ont eu aucun effet, la sortie est encore écrite sur le terminal. C'est la preuve que les "redirections" sont incapables de rediriger les écritures envoyées sur le terminal.

L'exemple classique
-----------------------

```shell
monProgramme >unFichier 2>&1
```

1. Le fichier **`unFichier`** est ouvert en écriture et est indiqué dans **`desc[1]`**
1. L'opération **`desc[2] = desc[1]`** est effectuée (donc **`desc[2]`** indique le fichier **`unFichier`**)
1. Le programme **`monProgramme`** est lancé. Habituellement, il écrit sur le fichier indiqué dans **`desc[1]`** sauf les erreurs qu'il écrit sur le fichier indiqué dans **`desc[2]`**
