---
jupytext:
  cell_metadata_json: true
  encoding: '# -*- coding: utf-8 -*-'
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.19.5
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
---

# TP on the moon

+++

**Notions intervenant dans ce TP**

* suppression de colonnes avec `drop` sur une `DataFrame`
* suppression de colonne entièrement vide avec `dropna` sur une `DataFrame`
* accès aux informations sur la dataframe avec `info`
* valeur contenues dans une `Series` avec `unique` et `value_counts` 
* conversion d'une colonne en type numérique avec `to_numeric` et `astype` 
* accès et modification des chaînes de caractères contenues dans une colonne avec l'accesseur `str` des `Series`
* génération de la liste Python des valeurs d'une série avec `tolist`

```{admonition} pensez à l'aide en ligne (rappels)
:class: dropdown tip

vous avez plein de moyens pour obtenir de l'aide; notamment dans ipython ou jupyter:

- pensez à utiliser la complétion avec Tab
- utiliser 'Shift-Tab' pour voir l'aide de la fonction que vous venez de taper - genre pour voir les paramètres attendus
- on peut aussi faire par exemple `pd.DataFrame.drop?` pour obtenir de l’aide, mais il faut valider la cellule, c'est souvent moins pratique que `Shift-Tab`

il y a aussi la méthode *old-school* qui consiste à appeler `help(une_fonction)`, qui fonctionne dans Python de base - mais en pratique c'est sous-optimal
```

+++

## 1. import

1. importez les librairies `pandas` et `numpy`

```{code-cell} ipython3
# votre code
import pandas as pd
import numpy as np
```

## 2. read

1. lisez le fichier de données `data/objects-on-the-moon.csv`
2.  affichez sa taille et regardez quelques premières lignes

```{code-cell} ipython3
# votre code
df = pd.read_csv("data/objects-on-the-moon.csv")
print(df.size)
df.head()
```

## 3. drop

1. vous remarquez une première colonne franchement inutile  
   utiliser la méthode `drop` des dataframes pour supprimer cette colonne de votre dataframe

```{code-cell} ipython3
# votre code
df = df.drop(columns=df.columns[0])
df
```

## 4. info

1. appelez la méthode `info` des dataframes (`non-null` signifie `non-nan` i.e. non manquant)
2. remarquez une colonne entièrement vide

```{code-cell} ipython3
# votre code
df.info()
```

La colonne 6 est entièrement vide.

+++

## 5. dropna

1. utilisez la méthode `dropna` des dataframes pour supprimer *en place* les colonnes qui ont toutes leurs valeurs manquantes  
   (ici on s'interdit un code qui ferait explicitement référence à la colonne `'Size'`)
2. vérifiez que vous avez bien enlevé la colonne `'Size'`

```{code-cell} ipython3
# votre code
df.dropna(axis = 1, thresh=1, inplace = True)
df
```

## 6. dropna (2)

1. affichez la ligne d'`index` $88$, que remarquez-vous ?
2. utilisez la méthode `dropna` des dataframes pour supprimer
   *en place* les lignes qui ont toutes leurs valeurs manquantes
   (et de nouveau sans faire référence à une ligne en particulier)

```{code-cell} ipython3
# votre code
print(df.iloc[88])
```

La ligne d'index 88 est entièrement vide.

```{code-cell} ipython3
df.dropna(thresh = 1, inplace = True)
df
```

## 7. dtypes

1. utilisez l'attribut `dtypes` des dataframes pour voir le type de vos colonnes
2. que remarquez vous sur la colonne des masses ?

```{code-cell} ipython3
# votre code
df.dtypes
```

La colonne des masses contient des chaînes de caractères.

+++

## 8. unique

1. utilisez la méthode `unique` des `Series`pour en regarder le contenu de la colonne des masses
2. que remarquez vous ?

```{code-cell} ipython3
# votre code
s = df['Mass (lb)']
s.unique()
```

La colonne des masses contient des nombres exprimés sous forme de chaînes de caractères.

+++

## 9. to_numeric

1. conservez la colonne `'Mass (lb)'` d'origine  
   (par exemple dans une colonne de nom `'Mass (lb) orig'`)  
1. utilisez la fonction `pd.to_numeric` pour convertir  la colonne `'Mass (lb)'` en numérique  
   en remplaçant les valeurs invalides par la valeur manquante (NaN)
1. naturellement vous vérifiez votre travail en affichant le type de la série `df['Mass (lb)']`
1. combien y a-t-il de données manquantes dans cette colonne ?

```{code-cell} ipython3
# votre code
df['Mass (lb) orig'] = df['Mass (lb)'].copy()
df['Mass (lb)'] = pd.to_numeric(df['Mass (lb)'],errors='coerce')
```

```{code-cell} ipython3
print(df['Mass (lb)'])
```

```{code-cell} ipython3
print(df['Mass (lb)'].isna().sum())
```

Il y a 4 données manquantes dans cette colonne.

+++

## 10. replace

1. cette solution ne vous satisfait pas, vous ne voulez perdre aucune valeur  
   (même au prix de valeurs approchées)  
2. vous décidez vaillamment de modifier les `str` en leur enlevant les caractères `<` et `>`  
   afin de pouvoir en faire des entiers  
   remplacez les `<` et les `>` par des '' (chaîne vide)
   ````{admonition} *hint*
   :class: dropdown tip

   les `pandas.Series` formées de chaînes de caractères sont du type `pandas` `object`  
   mais elle possèdent un accesseur `str` qui permet de leur appliquer les méthodes python des `str`  
   (comme par exemple `replace`)
    ```python
    df['Mass (lb) orig'].str
    ```
    ````
 3. utilisez la méthode `astype` des `Series` pour la convertir finalement en `int`
 4. (optionnel) pour les avancés: sauriez-vous convertir la série en entiers tout en conservant les `nan` ?
    ````{admonition} *hint*
    :class: tip dropdown

    cherchez `pandas.Int64Dtype`
    ````

```{code-cell} ipython3
# votre code
df['Mass (lb) orig']=df['Mass (lb) orig'].str.replace('<','')
df['Mass (lb) orig']=df['Mass (lb) orig'].str.replace('>','')
df['Mass (lb)'] = df['Mass (lb) orig'].astype(int)
```

```{code-cell} ipython3
print(df['Mass (lb)'].isna().sum())
```

## 11. convert

1. sachant que `1 kg = 2.205 lb`  
   créez une nouvelle colonne `'Mass (kg)'` en convertissant les lb en kg  
   arrondissez les flottants en entiers en utilisant `astype`

```{code-cell} ipython3
# votre code
df['Mass (kg)'] = df['Mass (lb)']/2.205
df['Mass (kg)'] = df['Mass (kg)'].astype(int)
```

## 12. countries

1. Quels sont les pays qui ont laissé des objets sur la lune ?
2. Combien en ont-ils laissé en pourcentage (pas en nombre) ?
   ```{admonition} *hint*
   :class: dropdown tip
   
   regardez les paramètres de `value_counts`
   ```

```{code-cell} ipython3
# votre code
s = df['Country']
print(s.unique())
```

```{code-cell} ipython3
s.value_counts()/len(s)
```

## 13. total

1. quel est le poids total des objets sur la lune en kg ?
2. quel est le poids total des objets laissés par les `United States`  ?

```{code-cell} ipython3
# votre code
s = df['Mass (kg)']
poids_total = s.sum()
print(f"poids total = {poids_total} kg")
```

```{code-cell} ipython3
mask = (df['Country'] == 'United States')
poids_us = df.loc[mask,'Mass (kg)'].sum()
print(f"poids total des objets laissés par les US = {poids_us} kg")
```

## 14. blame

1. quel pays a laissé l'objet le plus léger ?  
   ````{admonition} *hint*
   :class: dropdown tip
   
   voyez les méthodes `Series.idxmin()` et `Series.argmin()`
   ````

```{code-cell} ipython3
s = df['Mass (kg)']
min_index = s.idxmin()
print(df.loc[min_index, 'Country'])
```

## 15. memorial

1. y-a-t-il un Memorial sur la lune ?  
   ````{admonition} *hint*
   :class: dropdown tip
   en utilisant l'accesseur `str` de la colonne `'Artificial object'`  
   regardez si une des descriptions contient le terme `'Memorial'`
   ````
2. quel est le pays qui a mis ce mémorial ?

```{code-cell} ipython3
# votre code
s = df['Artificial object']
c = s.str
mask = c.find('Memorial') != -1
df.loc[mask]
```

Il y a un Memorial sur la Lune.

```{code-cell} ipython3
# votre code
mask = c.find('Memorial') != -1
df.loc[mask, 'Country']
```

## 16.  tolist

1. faites la liste Python des objets sur la lune  
   ````{admonition} *hint*
   :class: dropdown tip
   voyez la méthode `tolist()` des séries
   ```

```{code-cell} ipython3
# votre code
s = df['Artificial object']
s.to_list()
```

***
