# Vision artificielle — TD 1 & TP 1 (Master 2 GNIPIA)

Corrigé du **TD 1** et réalisation du **TP 1** du chapitre 1 de *Vision artificielle*
(Master 2 GNIPIA — ENS Rabat, Université Mohammed V).

## Contenu

| Fichier | Description |
|---|---|
| [`TD1_corrige.md`](TD1_corrige.md) | Corrigé détaillé des 7 exercices du TD (tâches visuelles, matrice image, échantillonnage/quantification, mémoire et débit vidéo, acquisition et mouvement, histogrammes et couleur, évaluation) |
| [`TP1_vision_artificielle.ipynb`](TP1_vision_artificielle.ipynb) | Notebook des 6 activités du TP, avec code, sorties et réponses aux questions |
| `data/` | Images d'entrée (`image1.jpg`, `image2.jpg`) |
| `sorties/` | Figures et fichiers générés (histogrammes, masques, exports PNG/JPEG) |

## Activités du TP

1. **Inspection du tableau image** : forme, type, `size`, `nbytes`, pixel central, ordre BGR
2. **Couleurs et ROI** : conversions RGB / gris / HSV, canaux séparés, `copy()`, réduction `INTER_AREA`
3. **Intensités et histogrammes** : éclaircissement, contraste réduit, écrêtage, histogrammes à 32 classes
4. **Quantification sur 4 niveaux** : effet de paliers, `nbytes`, stockage compact 2 bits (4 pixels/octet)
5. **Masque de couleur rouge** : double intervalle HSV, taux de pixels, variation des seuils, limites
6. **Stockage et fichiers** : PNG vs JPEG (qualités 90 et 30), taille fichier vs tableau décodé, `np.array_equal`

## Installation et exécution

```bash
git clone <url-du-depot>
cd vision-artificielle-gnipia
python -m venv .venv && source .venv/bin/activate   # Windows : .venv\Scripts\activate
pip install -r requirements.txt
jupyter notebook TP1_vision_artificielle.ipynb
```


> Les images fournies sont des scènes synthétiques (voiture rouge, panneau rouge) afin que le TP soit reproductible.
> Les valeurs numériques commentées correspondent à ces images ; les raisonnements restent valables pour toute image.

## Résultats clés (image fournie, 480 × 640)

- 307 200 pixels, 921 600 valeurs, 921 600 octets (`uint8`)
- ROI centrale `(240, 320, 3)` ; image réduite au quart `(120, 160, 3)` (rapport 16)
- Quantification à 4 niveaux : `np.unique → [0, 85, 170, 255]` ; `nbytes` inchangé (8 bits/élément), 76 800 octets en compact 2 bits
- Masque rouge (H ∈ [0,10] ∪ [170,179], S ≥ 80, V ≥ 50) : 6,90 % des pixels
- PNG : pixels identiques après relecture ; JPEG : pertes (erreur moyenne 1,62 à qualité 90 ; 2,82 à qualité 30)


