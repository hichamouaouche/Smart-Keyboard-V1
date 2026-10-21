# Smart Keyboard V1 - Mini rapport

Auteur : Hicham Ouaouche

Lien de déploiement Streamlit : https://smart-keyboard-v1git-mbygy2xwjucaqrtmygynq9.streamlit.app/

## 1. Objectif du projet

Ce projet a pour but de construire un moteur de suggestions de mots pour un clavier intelligent. L’idée est simple : à partir du texte déjà saisi, le modèle prédit le mot suivant avec une probabilité calculée à partir d’un modèle de langage.

Le système utilise des modèles n-grammes :
- unigramme
- bigramme
- trigramme
- interpolation 3/2/1-gram
- back-off

L’objectif final est double :
1. évaluer la qualité des prédictions,
2. proposer une interface utilisateur simple avec Streamlit.

---

## 2. Ce que le projet fait

Le projet lit un corpus synthétique de phrases, apprend les statistiques de cooccurrence des mots, puis prédit le mot le plus probable après un contexte donné.

Exemple :

```text
Entrée : the horses eat the
Suggestions : bread, apples, soup
```

L’algorithme ne s’appuie pas sur le sens du texte, mais sur les fréquences observées dans le corpus. Il est donc fondé sur des statistiques de langage, ce qui est la base historique des modèles de langage.

---

## 3. Données utilisées

Les données sont générées automatiquement dans le fichier `data/make_data.py`.

Caractéristiques :
- 1 200 phrases synthétiques
- 60 documents
- environ 10 000 tokens
- vocabulaire de taille limitée
- graine fixe pour reproduire les mêmes résultats

Le corpus est généré par une grammaire simple, donc il ne reflète pas le langage naturel réel, mais il est parfait pour un laboratoire pédagogique.

---

## 4. Séparation entraînement / test

Le projet sépare les données par document, et non par phrase.

Pourquoi ?
- Les phrases d’un même document partagent souvent un thème ou un vocabulaire.
- Si on mélangeait les phrases au hasard, le modèle verrait des mots du test dans l’entraînement.
- Cela fausserait les résultats (fuite de données).

La séparation utilisée est :
- entraînement : 960 phrases
- test : 240 phrases

---

## 5. Modèles utilisés

### 5.1 Unigramme
Le modèle ignore le contexte et ne dépend que de la fréquence globale des mots.

### 5.2 Bigramme
Le modèle prend en compte le mot précédent.

### 5.3 Trigramme
Le modèle prend en compte les deux mots précédents.

### 5.4 Lissage add-k
Sans lissage, une séquence jamais vue dans l’entraînement reçoit une probabilité nulle. Cela crée un problème majeur :
- la probabilité devient 0,
- le log devient infini,
- la perplexité devient infinie.

Pour éviter cela, le projet utilise le lissage add-k :

```text
P(w | contexte) = (count(contexte, w) + k) / (count(contexte) + k * V)
```

où :
- `k` est le paramètre de lissage,
- `V` est la taille du vocabulaire.

### 5.5 Interpolation et back-off
Le projet compare aussi :
- le modèle trigramme seul,
- le modèle interpolé 3/2/1,
- le back-off vers un ordre inférieur si le contexte est absent.

---

## 6. Métriques d’évaluation

### 6.1 Perplexité
La perplexité mesure à quel point le modèle est “confus” par le vrai mot suivant.

Plus la perplexité est faible, mieux c’est.

### 6.2 Top-3 accuracy
Le modèle propose 3 mots. On vérifie si le bon mot est dans ces 3 suggestions.

Exemple de résultat observé :
- unigramme : ~48 %
- bigramme : ~61 %
- trigramme : ~64 %
- interpolated 3/2/1 : ~64.7 %

---

## 7. Résultats observés

Les résultats montrent clairement que le trigramme lissé est bien meilleur que le simple unigramme. Le modèle bigramme est déjà beaucoup plus fort, mais le trigramme capture mieux le contexte local.

Le lissage est indispensable : sans lui, les séquences inconnues produisent des probabilités nulles et la perplexité explose.

---

## 8. Analyse des erreurs

Le modèle n-gramme est simple et utile, mais il a des limites :
- il ne comprend pas vraiment le sens,
- il peut se tromper sur des mots très plausibles,
- il a du mal quand le contexte n’a jamais été vu dans l’entraînement,
- il ne propose que 3 candidats, même si plusieurs mots pourraient convenir.

Exemples de contextes difficiles :
- the → foxes
- the foxes eat the → cakes
- brave foxes
- happy bread

Ce sont des cas où plusieurs choix sont possibles ou où le contexte n’a pas été vu pendant l’apprentissage.

---

## 9. Interface utilisateur

Le projet inclut une interface Streamlit dans `app.py`.

Fonctionnement :
- l’utilisateur tape une phrase ou une partie de phrase,
- le système affiche les mots les plus probables,
- le modèle utilise l’ordre le plus fiable disponible,
- les mots inconnus sont traités comme `<unk>`.

L’interface a été déployée et fonctionne bien en ligne.

---

## 10. Structure du projet

```text
02_smart_keyboard_v1/
├── app.py
├── requirements.txt
├── data/
│   ├── corpus.csv
│   └── make_data.py
├── src/
│   ├── evaluate.py
│   └── pipeline.py
├── results/
│   ├── failures.txt
│   ├── metrics.json
│   └── ngram_results.png
└── README.md
```

---

## 11. Comment exécuter le projet

### Installation

```bash
cd 02_smart_keyboard_v1
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### Génération du corpus

```bash
python3 data/make_data.py
```

### Évaluation

```bash
python3 -m src.evaluate
```

### Interface Streamlit

```bash
streamlit run app.py
```

---

## 12. Déploiement

Le projet a été déployé sur Streamlit Community Cloud avec succès.

Le déploiement est adapté à cette application car :
- elle est écrite en Python,
- elle a une interface web simple,
- elle est facile à lancer avec Streamlit,
- elle ne nécessite pas de backend complexe.

---

## 13. Conclusion

Ce projet montre la base de la modélisation de langage : compter les mots, extraire les contextes, calculer des probabilités, lisser les valeurs et évaluer la qualité des prédictions.

Même si le modèle n-gramme est simple, il permet de comprendre les principes fondamentaux des systèmes de suggestion de texte, qui sont utilisés plus tard dans des modèles beaucoup plus avancés.

En résumé, ce laboratoire permet de passer de la théorie des probabilités à une application concrète de prédiction de mot, et c’est exactement la base du fonctionnement des claviers intelligents modernes.
