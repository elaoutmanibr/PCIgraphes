# Création des noeuds antenne à partir de l'API (Limité à 1000 antennes)
```
with "https://data.anfr.fr/api/records/2.0/search/resource_id=88ef0887-6b0f-4d3f-8545-6d64c8f597da&distinct=true&limit=1000&offset=0" as url
call  apoc.load.json(url) yield value
UNWIND value["result"]["records"] as record
CREATE (a:antenne)
SET a = properties(record)
```

# Création d'un lieu par groupement d'antenne sur une même coordonnée
```
MATCH (a:antenne)
WITH DISTINCT a.coordonnees AS unformated_coords
WITH
    split(unformated_coords, ',') AS coords
CREATE (b:lieu {coordonnees: coords, longitude: toFloat(coords[0]), latitude: toFloat(coords[1])})
```

# Création des relations de proximité entre chaque lieux
```
MATCH (a:lieu)
CALL {
    WITH a
    MATCH  (b: lieu)
    WHERE a.coordonnees <> b.coordonnees
    WITH
        b,
        a.coordonnees AS coords_a,
        b.coordonnees AS coords_b
    WITH
        b, coords_a, coords_b,
        point({longitude: toFloat(coords_a[0]), latitude: toFloat(coords_a[1])}) AS pa,
        point({longitude: toFloat(coords_b[0]), latitude: toFloat(coords_b[1])}) AS pb
    RETURN b
    ORDER BY point.distance(pa, pb) ASC
    LIMIT 5
}
MERGE (a)-[:NEAR]-(b)
```

# Connexion des antennes à leur lieu respectif (todo)
```
MATCH (a:antenne)
CALL {
    WITH a
    MATCH (b: lieu)
    WHERE split(a.coordonnees, ",") = b.coordonnees
    RETURN b
}
MERGE (a)-[:AT]->(b)
```

# Recommendation (Brouillon)
```
CREATE (u:utilisateur)

MATCH (u:utilisateur)
MATCH (l:lieu {coordonnees: ["44.916111111111", " -.225"]})
MERGE (u)-[:USES]->(l)

MATCH (u:utilisateur)-[:USES]->(l: lieu)<-[:AT]-(a:antenne)
RETURN a
```
