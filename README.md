## Unit 3 Prosit 4 – Index / Fragmentation
### Mots Clés
- Fragmentation* : Art de décomposer ses données (blocs/extends) entre les différents tablespace. On peut optimiser un peu les performances avec.
- Optimisation
-Tuning* John
- Index
- Mémoire (Alloué au traitement de bases)
- SGBD
- Base transactionnelles* : Base qui vérifie tout avant d'éffectuer son opération.
- Cout d’exploitation
- Performance de traitement
- Maintenance
- Dictionnaire de données
- Oracle
## Contexte
### Quoi ?
Optimiser la base
### Comment ?
En faisant des mises a jours régulière
Better sql code
En gérant lmieux ses indexes + les données en cache
Evitant la fragmentation des tables
### Pourquoi ?
Parce que.
## Contraintes
404 Not Found
## Problématique
Comment optimiser le temps de réponse de sa base de données Oracle ?
## Généralisation
Optimisation
## Hypothèses
- Une mise a jour permet d’optimiser la base de données.
- Si on a assez de RAM pour mettre toute la base, ça va plus vite !
- Si l’on peint sa base de données en rouge, ça va plus vite !
- Indexer les champs va grandement augmenter les perfs.
- Oracle intègre des outils permettant de monitorer les tables.
## Plan d’Action
**Etudes**
- Index
- Fragmentation

**Réalisation**
- Corbeille




## Optimisation de requêtes SQL sous oracle - Les INDEXS

- Trouver le chemin le plus cours (ne pas parcourir inutilement)
	- Parcours de la tale complet (Full Table Scan) ==> Absence d'index, très couteux.
	- Parcours (balayage) d'index : Parcouru à la recherche des valeurs,via un ROWID ou une ligne concerné dans la table.
		- Intervalle index (Index range scan) : Index parcouru pour trouver toutes les valeurs, clauses d’égalité, supériorité ou infériorité se déclenchent/
		- Balayage unique (Unique index scan) : S'applique lorsqu'il repose sur une colonne avec une Ci unique
		- Balayage à contre-sens : Pour faire un tri
		- Balayage par saut : Saute les zones où les clefs ne pourraient pas se trouver.
	- Accès par ROWID ou TableAccess by ROWID : On accès DIRECTEMENT au ROW ID en supposant que Oracle le connaît.
- **Plan d'exécution** : regroupe l'ensemble des chemins d'accès utilisés pour une requête. C'est donc l'itinéraire (ex : tournez à droite).
- **Optimiser le chemin** : On demande à oracle d'utiliser le chemin qu'il estime avoir le meilleur "coût" (cost-based), prenant en compte l'utilisation des ressources CPU, disques, mémoire...
- **statistiques** = métadonnées permettant à l'optimiseur d'estimer le coût d'accès aux données. Elles sont calculés automatiquement et périodiquement. 
```SQL
EXEC DBMS_STATS.gather_database_stats; // pour toute la base ... argl !
EXEC DBMS_STATS.gather_database_stats(estimate_percent => 15); // pour la base, avec un échantillon de 15%

EXEC DBMS_STATS.gather_schema_stats('SCOTT'); // collecte pour le schéma SCOTT
EXEC DBMS_STATS.gather_schema_stats('SCOTT', estimate_percent => 15); // idem avec 15% d'achantillon

EXEC DBMS_STATS.gather_table_stats('SCOTT', 'EMPLOYEES'); // pour une table
EXEC DBMS_STATS.gather_table_stats('SCOTT', 'EMPLOYEES', estimate_percent => 15); // idem avec 15% ...

EXEC DBMS_STATS.gather_index_stats('SCOTT', 'EMPLOYEES_PK'); // pour un index
EXEC DBMS_STATS.gather_index_stats('SCOTT', 'EMPLOYEES_PK', estimate_percent => 15);
```
- Les "hints" ou suggestions, sont des instructions que l'on peut insérer dans les ordres SQL pour influencer l'optimisateur. ``/* + MONHINT */```. A ne pas utiliser tant qu'il n'y a pas de soucis. C'est écrit en dur et donc figé, on peut avoir une contre-performance.
- On peut connaître le plan que suit l'optimisateur. Pour se faire, on peut utiliser le mode "Autotrace sous SQLPlus".
``SQL> set autotrace on;``
	 -	 En exécutant une requête,on voit ainsi la consommation de la requête en coût CPU.
	 -	On peutégalement utiliser EXPLAIN PLAN FOR (<requête>) pour retrouver des informations importantes
```SQL
SQL> explain plan for (select * from employees where last_name like 'T%');
ExplicitÚ.
SQL> select plan_table_output from table(dbms_xplan.display());
```
- Les colonnes les plus importantes sont le cost et le time

### Les indexs : 

- Les index sont comme un sommaire/glossaire. Il permet rapidement d’accéder à la page du livre.
- Ex : Si on crée un index sur une date de naissance, on peut retrouver rapidement tous les individus à une date de donnée.
- Il en existe plusieurs types :
	- B-tree : index par défaut, organisé en arbre. (Ordre du de création des indexs). NB : On peut indexer une valeur nulle si elle est associée à une valeur non nulle.
	- Bitmap :  Mot binaire composéa d'autant de bits que de possibilités de valeur de l'index. Efficace quand peux de possibilité (Ex: NULL/Mr/Mme/Mademoiselle ⇒ quatre possibilités sur une même colonne)
	- Clé inverse : On casse la continuité au niveau de la numérotation des index, on commence à l'envers à partir d'une haute séquence, utile quand beaucoup d'enregistrements et sur une clause where x = ID.
	- Index basé sur les fonctions.: On indexe sur le résultat d'une opération. On pré calcule donc la valeur.


**Indexs composés** :
- On place les colonnes les plus interrogées en premier (cf B-tree)

** Inconvénients / Limites index** : 
- Attention aux fonctions sur les index (Ex : UPPER(<index>) prendre plus de temps que la clé directement). Il faut prévoir en amont pour ne pas utiliser de fonctions sur les index.
- Ne pas mettre trop d'index. ça ralentirai d'autres requêtes comme les INSERT.Il faut donc trouver un compromis


## Les tables organisées en index :
- Structure est celle d'une index
-  Les valeurs sont directement enregistrés dans la structure
- Table de paramètres qui changent peu

## Clés étrangères et index
- Oracle n'index pas AUTOMATIQUEMENT  les clés étrangères !
- Il faut le faire si d'un côté, c'est un clé primaire (en parcourant la seconde table, on va accéder à la valeur de référence plus rapidement pour vérifier son intégrité)

## Tables en cache
- On peut demander à Oracle de conserver des tablesen cache, pour des petites tables de paramètres c'est utile. (comme jointures)

## Vues matérialisées
- Utile dans la prise de décision ???
-  Pas indispensables

## FRAGMENTS

Différents moyens derépartir les différents extents d'une table. 
**"EFFET GRUILLERE"**
- L'ASSM s'en occuppe, donc pas de soucis pour le nettoyage. ==> L'espace libre ne pose pas de problème particulier.
- Possibilité de choisir la taille des blocs / extends
- Cependant il faut bien structurer ses tablespace pour ne pas exécuter trop de "DELETE" sur toutes les tablespace. 
  -  Il existe des pattern de suppression de données qui peuvent résoudre des situations de sous-blocs dispersés, et/ou qui permettent de réutiliser l'espace libre.
- Au niveau des index, il existe deux problèmes :
  - On ne met pas à jour une entrée d'idex, on supprime l'ancienne, et on ajoute une nouvelle valeur qui prendra la valeur de l'ancienne ==> Problèmes lors de la régénération si c'était une date à un instant T, on régénère à la suite au lieu de remplacer, prenant de la place pour rien sur les anciennes lignes. (cf indexation qui ne suit pas en SQL)
  - Ajout d'un index, alors que c'est plein dans un bloc de données, il va le placer à la fin. On doit odnc faire beaucoup de recherches sur les blocs avant de trouver la solution. 
  ==> Il vaut donc mieux tout détruire au niveau des indexs pour les replacer ensemble.
  
  

