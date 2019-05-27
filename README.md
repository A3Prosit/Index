
## Unit 3 Prosit 4 – Index / Fragmentation
### Mots Clés
- Fragmentation*
- Optimisation
- Tuning* John
- Index
- Mémoire (Alloué au traitement de bases)
- SGBD
- Base transactionnelles*
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


**Les index**
un index est un glossaire (sommaire, table des matière, touça touça ... tu parles fr)
un index est basé sur une ou plusieurs valeurs on parle alors d'index composé ou composite

il existe différents types d'index en oracle:
- B-tree: type par défaut, c'est évidement un arbre binaire. les racines sont constituées par les valeurs de gauche. 
```
create index monindex on individu(nom, prenom) 
```
créé un index en arbre avec les gens clasés par nom puis par prénom
Note : les valeurs null ne sont pasindexées  si elles sont seules
- bitmap: un mot binaire est créé composé d'autant de bits que de possiblités de laeurs de l'index. Il est particulièrement efficace pour un petit nombre de valeur. Lors de la recherche oracle effectue un AND sur la valeur de l'index pour comparer. Les index bitmaps provoquent lors des opérations d'écriture des verrous importants qui font qu'ils sont très adaptés aux bases en lecture mais deviennent contre-indiqués dès lors que la base devra subir un nombre important d'insert
 ```
CREATE BITMAP INDEX monIndex ON individu(nom)
```
- index à clé inversées: on inverse juste la clé. ça vient du fait que les index peuvent devenir un goulot d'étrangement quand on fait un grand nombre d'insertions puisqu'ils sont ordonés, on casse donc la continuité. c'est utilise si par exemple on modif un bloc de valzurs qui seraient normalement en fin. L'inconvéninent est qu'on ne peut pas faire de range scan. on peut faire un where champ = valeur mais pas de where champ \<valeur>
- index basés sur des fonctions: on n'index plus un champ mais le résultat d'une opération sur un champ par exemple sur UPPER(monChamp)

synthaxe générale

CREATE [UNIQUE] INDEX index_name
  ON table_name (column1, column2, ... column_n)
  [ COMPUTE STATISTICS ];

compute statistics dit à oracle de collecter des stats à la création pour opti le plan d'exécution

**balayage d'index**
il existe différents types de balayage d'index:
- balayage d'interval d'index (index range scan): parcouru pour trouver les valeurs. les clauses d'égalités, supériorité et infériorité déclenchent généralement cegenre d'opérations. on peut se servir de l'index pour faire un tri car il est déjà trié
- balayage unique (unique index scan): lorsqu'une index repose sur une colonne avec un CI unique
- balayage à contre-sens (descending index scan): fait généralement pour un tri
- balayage par saut (index skip scan): on saute des zones où la clé ne pourra pas se trouver. par exemple si on n'utilise pas la 1ere colone d'un index composé

inconvénients et limites des index: 
limite: les fonctions:
on ne peut utiliser un index que si larecherche est aite ur la vleur indexée mais par exemple UPPER(champ) et champ sont 2 choses différentes pour oracle (à juste titre)
on peut avoir un exemple où on veut faire un recherche qui n'est pas sensible à la casse mal faite et l'index est complètement inutile
si le champ est uniquement en maj il faut faire
where champ = upper(value)
et pas 
where upper(champ) = value
ily a des cas où on ne peut pas résoudre la problème et il vaut donc mieux créer un index sur upper(champ)

il vaut également mieux tojours mettre des valeurs par défaut plutôt que des null si possible car si la requête contient un "is not null" ou "is null" on est parti pour un full table scan


coût des index d'après la doc oracle insert dans un champ indexé prends 3 foisplus de temps donc si on met par exemple 4 index sur une table: 4*3=12 fois plus lent


- on peut créer une table ayant une structure d'index, utile pour les tables qui changent peu

```
CREATE TABLE MATABLE ( nom	VARCHAR2(30), prenom  VARCHAR2(20) ) ORGANISATION INDEX; 
```

- il faut faire attention aux conversion de type
WHERE number = '1' case l'index car oracle fait un to_char() en théorie depuis la 10g oracle ne tombe plus dans le piège mais dans le doute

il ne faut pas oublier de relancer l'analyse après de grosses modifpour avoir les stats à jour

attention aux requêtes sur les vues qui sont des doubles requêtes



**l'opti oracle**


l'optimiseur oracle trouve le meilleur chemin tout seul mais on peut l'influencer en insérant un hint dans l'ordre sql avec /*+ MONHINT */. Oracle le suivra si possible et l'igniorera sinon. Le problème c'est qu'ils sont fixes et peuvent donc finir par être dépassés voir contre-performants

on peut voir le chemin que prends oracle avec l'auto-trace

```
set autotrace on;
```

on peut avoir des explications avec 
```
explain plan for (commande);
select plan_table_output from table(dbms_xplan.display());
```

si erreur faire : grant plustrace to votreUser;



les différents chemins d'accès:
- full table scan: parcourt tt la table. Arrive si il n'y a pas d'index. il est évidement très couteux sur les grosses tables
- parcours (balayage) d'index: on parcours un index, cette lecture donne généralement lieu à des accès par rowid aux lignés concernées dans la table, mais pas toujours
- table accès by ROWID: un row id permet d'accéder plus rapidement à un ligne, oracle accède directement à la ligne avec le rowid dans la table  mais il faut l'avoir indexé auparavant généralement avec index range scan


- Fragmentation

**Réalisation**
- Corbeille
exo1:
cb d i/o sur cette requête avec un parcours séquentiel de la relation hotel 5cat, 1000pages,10tuples par page)
SELECT adresse, tel, nb_chambres FROM hotel WHERE noms=’pesey’ AND categorie=3 ;

on prends adresse, tel, nb_chambres on test noms et categorie et un index b-tree sur (noms,nomh) de 26p et catgorie de 33p

du coup:

1 sans index: check 2 champs et lis 3
(2*1000p*10)*3 = 60k i/o

2 avec index sur catagorie:
(check 26p categorie*10+1000*10 pour noms )*3 champ = (260+10k)*3 = 30780 i/o

3
double index
33p*10 pour categorie=3, 26p*10 pour noms (c'est le 1er de l'arbre) et on recup 3 entrées pour chaque = (33* 10+26* 10)*3 = (330+260)*3 = 590 *3 = 1770 i/o

exo 2 :
 select acteur, realisateur from film,vu where film.titre=vu.titre
 
---------------------------------------------------------------------------
| Id  | Operation          | Name | Rows  | Bytes | Cost (%CPU)| Time     |

---------------------------------------------------------------------------
|   0 | SELECT STATEMENT   |      |     1 |    72 |     5  (20)| 00:00:01 |
|*  1 |  HASH JOIN         |      |     1 |    72 |     5  (20)| 00:00:01 |
|   2 |   TABLE ACCESS FULL| FILM |     1 |    54 |     2   (0)| 00:00:01 |
|   3 |   TABLE ACCESS FULL| VU   |     1 |    18 |     2   (0)| 00:00:01 |

---------------------------------------------------------------------------

2

--------------------------------------------------------------------------------
-------

| Id  | Operation                    | Name   | Rows  | Bytes | Cost (%CPU)| Time     |

--------------------------------------------------------------------------------
-------

|   0 | SELECT STATEMENT             |        |     1 |    72 |     2   (0)| 00:00:01 |

|   1 |  NESTED LOOPS                |        |       |       |            |      |

|   2 |   NESTED LOOPS               |        |     1 |    72 |     2   (0)| 00:00:01 |

|   3 |    TABLE ACCESS FULL         | VU     |     1 |    18 |     2   (0)| 00:00:01 |

|*  4 |    INDEX RANGE SCAN          | INDEX1 |     1 |       |     0   (0)| 00:00:01 |

|   5 |   TABLE ACCESS BY INDEX ROWID| FILM   |     1 |    54 |     0   (0)| 00:00:01 |

--------------------------------------------------------------------------------
-------

3


| Id  | Operation                    | Name   | Rows  | Bytes | Cost (%CPU)| Time     |

--------------------------------------------------------------------------------
-------

|   0 | SELECT STATEMENT             |        |     1 |    72 |     1   (0)| 00:00:01 |

|   1 |  NESTED LOOPS                |        |       |       |            |      |

|   2 |   NESTED LOOPS               |        |     1 |    72 |     1   (0)| 00:00:01 |

|   3 |    INDEX FULL SCAN           | INDEX2 |     1 |    18 |     1   (0)| 00:00:01 |

|*  4 |    INDEX RANGE SCAN          | INDEX1 |     1 |       |     0   (0)| 00:00:01 |

|   5 |   TABLE ACCESS BY INDEX ROWID| FILM   |     1 |    54 |     0   (0)| 00:00:01 |


exo 3: later

exo 4: same

exo 5 :
1)  select Nom, Titre from Artiste,Joue,Film;
2)




**En termes de Savoir :**

1.  Comprendre les mécanismes internes au SGBD Oracle et leurs interactions et tirer profit des automatismes de tuning afin d’optimiser au plus juste les rendements et les performances des bases de données Oracle.
2.  Savoir décrire l’organisation du dictionnaire de données dans une base Oracle

**En termes de Savoir-faire :**

-   Chargement des données dans une base Oracle
-   Gestion des index basés sur des contraintes
-   Développement de procédures, fonctions et packages
-   Résolution des problèmes PL/SQL critiques
-   Génération des statistiques avec DBMS_STATS
-   Utilisation des outils de diagnostic de performances
-   Optimisation des applications Oracle
-   Identification des goulots d’étranglement de SQL avec DBMS_PROFILER
-   Le dictionary cache.
-   Retrouver une information dans le dictionnaire de données
