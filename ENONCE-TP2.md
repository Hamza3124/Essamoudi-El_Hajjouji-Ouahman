# TP2 : entraîner, évaluer et accepter un modèle de sentiment

**Cours :** Développement et déploiement d'applications intelligentes · **Semaine 2** · **Travail :** en binôme · **Dépôt :** `sentiment-app` (celui du TP1)

## Ce que vous allez faire

Au TP1, vous avez cadré trois cas et préparé un dépôt. Vous allez maintenant produire le composant appris de l'application fil rouge : un classifieur de sentiment, entraîné, mesuré, puis accepté ou refusé selon un seuil décidé à l'avance. Le modèle que vous enregistrerez aujourd'hui est celui que l'API de la semaine 3 chargera.

Le point central du TP n'est pas d'obtenir un bon score. C'est de savoir ce que le score veut dire, sur quelles données il a été obtenu, et ce qu'il ne garantit pas.

## Prérequis

- Le TP1 terminé : `pytest -q` affiche `38 passed`.
- L'environnement virtuel actif (l'invite commence par `(.venv)`).
- Le réseau en séance pour l'étape D (environ 60 Mo par binôme). À défaut, l'enseignant fournit `data/allocine.csv` sur clé USB.

## Objectifs

À la fin du TP, votre dépôt contient :

1. `src/make_demo.py`, `src/train.py`, `src/evaluate.py` et `src/predict.py`, exécutés au moins une fois chacun ;
2. `models/metrics.json`, versionné dans Git, avec vos chiffres ;
3. `models/sentiment.joblib`, présent sur vos postes mais **jamais commité** ;
4. `docs/semaine2.md` rempli : vos mesures, votre lecture des erreurs, votre décision d'accepter ou non le modèle ;
5. `pytest -q` au vert, 42 tests (38 du TP1 et 4 du TP2).

## Ce que le kit fournit

| Fichier | Rôle |
| --- | --- |
| `appliquer_tp2.py` | Dépose le kit sur votre dépôt du TP1 sans rien écraser ; options `--simulation` et `--force` |
| `src/make_demo.py` | 290 avis français déterministes : 100 positifs, 100 négatifs, 60 mitigés construits par paires, 30 courts |
| `src/train.py` | Pipeline TF-IDF (1 et 2 grammes) et régression logistique ; option `--allocine` ; refuse le modèle sous F1 macro 0,85 |
| `src/evaluate.py` | Rapport par classe, matrice de confusion, trois erreurs affichées |
| `src/predict.py` | La fonction `predict()` que l'API de la semaine 3 appellera |
| `tests/test_model.py` | 4 tests : 2 sur `metrics.json`, 2 sur le modèle (ignorés si le modèle est absent) |
| `docs/semaine2.md` | Gabarit de vos notes, quatre rubriques obligatoires |
| `notebooks/TP2-colab.ipynb` | Le même TP sur Colab, sans installation |

## Déroulement

### Étape A : installer le kit

Depuis le dossier du kit décompressé (à côté de votre dépôt, pas dedans), en indiquant le chemin de votre dépôt. Faites d'abord une simulation, qui n'écrit rien :

```bash
python appliquer_tp2.py <chemin-de-votre-depot> --simulation
python appliquer_tp2.py <chemin-de-votre-depot>
```

Attendu : `7 fichier(s) copie(s), 0 laisse(s) en place.` Le script complète aussi `requirements.txt` (`joblib==1.4.2`, `pyarrow==17.0.0`) et `.gitignore` (le cache Allociné est exclu, `models/metrics.json` est versionné alors que le modèle reste hors de Git).

Puis, dans votre dépôt, environnement virtuel activé :

```bash
pip install -r requirements.txt
pip freeze > requirements.lock
```

Windows : si `requirements.txt` est en lecture seule après la décompression du TP1, lancez d'abord `attrib -R requirements.txt`.

### Étape B : fabriquer les données, puis entraîner

```bash
python src/make_demo.py      # contrôle T8
python src/train.py          # contrôle T9
```

Le jeu de démonstration est volontairement petit et hors ligne, pour que tout le monde démarre avec les mêmes chiffres. Ses 60 avis mitigés sont construits par paires : les deux avis d'une paire emploient exactement les mêmes mots, seul l'ordre des deux propositions change. Ce sont eux qui feront échouer le modèle, et c'est voulu.

Notez le F1 macro et le rappel sur la classe négative. Vérifiez que `models/metrics.json` existe. Si le F1 macro passait sous 0,85, le script afficherait « Modele refuse » et sortirait avec le code 1, sans rien enregistrer.

### Étape C : lire les erreurs

```bash
python src/evaluate.py       # contrôle T10
```

Le script affiche la matrice de confusion et trois avis mal classés. Recopiez-les dans `docs/semaine2.md` et répondez : qu'ont-ils en commun, et pourquoi un modèle de ce type s'y trompe-t-il ?

### Étape D : passer aux vraies données

```bash
python src/train.py --allocine       # contrôle T13
python src/evaluate.py --allocine
```

Le script télécharge 20 000 critiques de films en français (une seule fois, mise en cache dans `data/allocine.csv`, exclu de Git) et réentraîne. Comparez avec la démonstration : le jeu change, les scores changent, le code ne change pas.

Si le réseau est coupé, gardez la démonstration et notez-le dans `docs/semaine2.md`.

### Étape E : prédire, tester, rendre

```bash
python src/predict.py "Livraison rapide, produit conforme, tres satisfait"     # contrôle T11
python src/predict.py "Produit casse et service injoignable, a fuir"
python -m pytest -q                                                            # contrôle T12
git status
```

Essayez ensuite le premier avis avec le modèle entraîné sur Allociné : c'est un avis de livraison, jugé par un modèle qui n'a lu que des critiques de films. Notez ce que vous observez.

`pytest -q` doit afficher `42 passed`, et `git status` ne doit lister aucun fichier `.joblib` ni `allocine.csv`.

Pour revenir au modèle de démonstration avant de rendre (plus rapide et reproductible) : `python src/train.py`.

## Contrôles T8 à T13 : résultats attendus

| Contrôle | Commande | Attendu |
| --- | --- | --- |
| T8 données | `python src/make_demo.py` | `290 avis ecrits dans data/demo_reviews.csv (145 positifs, 145 negatifs)` |
| T9 entraînement | `python src/train.py` | 58 avis en test, F1 macro 0.8961, rappel négatif 0.9655 |
| T10 évaluation | `python src/evaluate.py` | matrice 28 / 1 / 5 / 24, 6 erreurs sur 58, dont 4 avis mitigés |
| T11 prédiction | `python src/predict.py "..."` (deux avis) | `positive (confiance 83%)` puis `negative (confiance 70%)` |
| T12 tests et Git | `python -m pytest -q` puis `git status` | `42 passed` ; `metrics.json` suivi, aucun `.joblib` |
| T13 vraies données | `python src/train.py --allocine` | 20 000 avis, 4 000 en test, F1 macro 0.9020, rappel négatif 0.8905 (sous le seuil métier, voir plus bas) ; 392 erreurs sur 4 000 |

La marche à suivre détaillée, avec les sorties complètes, est dans `MARCHE-A-SUIVRE-TP2.md`.

## Chiffres de référence

Ces valeurs ont été obtenues en exécutant le kit. Elles sont déterministes : vous devez retrouver les mêmes.

| Mesure | Démonstration (290 avis) | Allociné (20 000 avis) |
| --- | --- | --- |
| Avis en test | 58 | 4 000 |
| F1 macro | 0,8961 | 0,9020 |
| Rappel sur la classe négative | 0,9655 | 0,8905 |
| Erreurs sur le jeu de test | 6 | 392 |

Un écart peut venir d'une version différente de scikit-learn : signalez-le plutôt que de modifier les seuils.

## À remarquer, et à discuter dans vos notes

**Deux seuils, deux réponses.** Le seuil de `train.py` porte sur le F1 macro et vaut 0,85 : les deux modèles le passent. Mais la fiche de cadrage du cas 3 (priorisation des avis négatifs) exigeait un **rappel d'au moins 0,90 sur la classe négative**. Le modèle de démonstration l'atteint (0,9655), celui entraîné sur Allociné ne l'atteint pas (0,8905) : sur 100 avis négatifs réels, environ onze passeraient sans alerte. Le modèle le mieux noté des deux est celui qui échoue au besoin métier. C'est exactement ce que la fiche de cadrage sert à détecter.

**Le décalage de domaine.** Avec le modèle Allociné, « Livraison rapide, produit conforme, très satisfait » ressort `negative` avec 54 % de confiance, alors qu'un avis de film est classé correctement avec une confiance élevée. Le vocabulaire du commerce est absent du corpus d'entraînement, et la confiance proche de 50 % le signale. Une métrique ne vaut que sur des données de même nature que celles d'entraînement.

## Livrables

- `models/metrics.json` à jour et commité ;
- `docs/semaine2.md` rempli, avec les quatre rubriques obligatoires :
  1. vos chiffres, pour les deux jeux de données ;
  2. la lecture des trois erreurs affichées par `evaluate.py` ;
  3. la décision : le modèle est-il accepté, au regard des deux seuils, et pour quel usage ?
  4. les limites constatées, dont l'essai hors domaine ;
- un commit par membre du binôme ;
- CI au vert (sur la machine de CI, aucun modèle n'est entraîné : `40 passed, 2 skipped` est le résultat normal).

```bash
git add -A
git status                   # aucun .joblib, aucun allocine.csv
git commit -m "TP2 : modele TF-IDF, F1 macro 0,896 (demo), 0,902 (Allocine)"
git push
```

## Barème indicatif (sur 20)

| Critère | Points |
| --- | --- |
| Kit installé, `pytest -q` au vert (42 tests) | 4 |
| `metrics.json` commité, aucun `.joblib` dans Git | 3 |
| Lecture des erreurs argumentée dans `docs/semaine2.md` | 5 |
| Comparaison démonstration et Allociné | 3 |
| Décision d'accepter ou de refuser le modèle, justifiée par les seuils | 3 |
| Travail du binôme visible dans l'historique Git | 2 |

## Pièges fréquents

| Symptôme | Cause | Correction |
| --- | --- | --- |
| `data/demo_reviews.csv absent` | `train.py` lancé avant `make_demo.py` | `python src/make_demo.py`, puis relancer |
| `40 passed, 2 skipped` sur votre poste | le modèle n'a pas été entraîné sur ce poste | refaire l'étape B |
| `models/sentiment.joblib` apparaît dans `git status` | `.gitignore` non complété ou forcé | relancer `appliquer_tp2.py`, ne jamais utiliser `git add -f` |
| `appliquer_tp2.py` : « ne ressemble pas au dépôt du TP1 » | mauvais chemin | donner le dossier qui contient `requirements.txt` |
| Échec du téléchargement Allociné | réseau coupé ou filtré | garder la démonstration, ou copier `data/allocine.csv` fourni |
| Confusion entre les seuils | 0,85 (F1 macro, `train.py`) et 0,90 (rappel négatif, cas 3) | ce sont deux questions différentes : technique et métier |
