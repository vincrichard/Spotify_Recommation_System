## Présentation

Dans ce projet on cherche via un <a href="https://www.kaggle.com/zaheenhamidani/ultimate-spotify-tracks-db/">dataSet kaggle </a> à créer un système de recommendation prenant une musique en input et proposant un certain nombre de titre similaire.

On se base sur un algorithme des K Nearest Neighbors et le projet est en scala. 

On pourra retrouver ce notebook sur <a href="https://github.com/vincrichard/Spotify_Recommation_System">github </a>

Étapes:

* [Importation des données](#import)

* [Préprocessing des données](#preproces)<br>
Dans cette partie on supprime des abbérations du à des titres de musique trop longue qui amènenent le décalage des informations de notre DataFrame. On remarque aussi qu'il y a des duplicats de musique qu'il faudra traiter plus tard. Ces duplicats sont du à la possibilité pour une musique d'appartenir à plusieur genre différent. Ces musiques auront alors plusieurs lignes associées et même si leurs ID reste les mêmes elles peuvent avoir des mesures différentes.

* [Etude des différentes métriques](#metric)<br>
Cette partie permet d'expliquer le sens de chaque métrique disponible dans le dataset. Pour chaque métrique un graphique associé permet de se rendre compte de la répartition des données dans le dataset.
Les différentes métriques sont: genre, artist_name, track_name, track_id, popularity, acousticness, danceability, duration_ms, energy, instrumentalness, key, liveness, loudness, mode, speechiness, tempo, time_signature, valence.

* [Typage des données](#typage)<br>
Typage du dataset en fonction de leur valeurs.

* [Etude de la correlation de nos features](#corr)<br>
Etude de la matrice de correlation de nos features pour s'assurer que nos features ne sont pas trop corrélées.

* [Selection de nos features](#selection)<br>
On décide de ne pas se servir de *duration_ms* et *popularity* dans la suite de notre modèle car ces valeurs ne semblent pas pertinente pour un système de recommendation.

* [Preparation de nos données](#prep)<br>
Dans cette partie on va modifier les champs de nos données pour pouvoir les utiliser dans notre modèle.
  + [Encodage de nos variable de texte avec StringIndexer](#stringindexer)   <br>
  On modifie quatre de nos features contenant du texte, genre, key, mode et time_signature afin d'avoir un input numérique.
  
  + [Utilisation d'un One Hot Encoder](#onehot)<br>
  Sur les mêmes features que le StringIndexer pour éviter à notre modèle de penser à une hiéarchie entre nos variables nous allons les passer dans un `OneHotEncoder` qui les transformera en vecteur de 0 et de 1.
  
  + [Aggregation de nos données](#agg)<br>
  Il faut régler le problème de la duplication des musiques dans notre dataset. De plus le genre de chaque musique nous intéresse il faut donc pouvoir supprimer les duplicats tout en gardant ces informations. Pour cela on aggrège chaque ligne ensemble en calculant la moyenne de chaque valeur numérique pour les mesures étant des entiers. Pour nos vecteurs créé avec le OneHotEncoder, on utilise la fonction max pour être sûr de ne pas perdre l'information des différents genres. 
  
  + [Vector Assembler](#vectorAssembler)<br>
  Afin d'avoir une seule colonne regroupant toutes nos features comme input de notre modèle.
  
  + [Normalizer](#normalizer)<br>
  Afin que chaque feature ait le même poids on normalize nos données.
 
* [Algorithme des plus proches voisins](#knn)<br>
On utilise la librairie de spark MLlib plus spécifiquement la classe `BuckereRandomProjectionLSH` qui est une approximation de la méthode des plus proches voisins dans le cas du traitement de grosse donnée.

* [Resultats](#result)<br>

* [Utilisation du model une fois entrainé](#udf)<br>
Définition d'une fonction permettant de réutiliser le modèle entrainé en prenant un *track_id* en input.


Défaut de l'étude:

Il n'y a pas de moyen de vérifier la précision du modèle, chacun doit essayer et écouter les titres proposés pour se rendre compte si le modèle marche ou non. Personnellement je trouve les résultats pertinents.

Amélioration:

Création d'une fonction pour la partie d'aggregation des musiques possédant de multiple lignes afin de pouvoir compacter la préparation des données en une seule pipeline.