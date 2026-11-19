# TD 1 — Vision artificielle : corrigé

**Master 2 GNIPIA — ENS Rabat (Université Mohammed V)** · Chapitre 1

> Conventions : 1 MB = 10⁶ octets · 1 MiB = 2²⁰ octets · 1 octet = 8 bits.

---

## Exercice 1 — Tâches visuelles et chaîne de traitement

| Situation | Tâche principale | Résultat attendu | Risque lié aux données / à l'acquisition |
|---|---|---|---|
| **A** Catégorie d'une photo de document | **Classification d'image** | Une étiquette de classe (+ score de confiance) pour l'image entière | Variations d'éclairage, de cadrage, de perspective ; flou ; classes déséquilibrées |
| **B** Localiser chaque véhicule | **Détection d'objets** | Une liste de boîtes englobantes (x, y, l, h) + classe + score | Occultations, objets petits ou partiellement visibles, annotations incohérentes |
| **C** Associer la même voiture entre images successives | **Suivi (tracking) / ré-identification** | Un identifiant persistant par véhicule le long de la séquence | Occultations, cadence trop faible, flou de bougé → échanges d'identités |
| **D** Séparer chaque feuille au niveau des pixels | **Segmentation (d'instances)** | Un masque pixel par feuille | Feuilles qui se chevauchent, ombres, annotation pixel coûteuse et subjective |
| **E** Extraire le texte d'un formulaire numérisé | **Reconnaissance de caractères (OCR)** | Une chaîne de texte (avec position des champs) | Faible résolution, document incliné, bruit de numérisation, polices variées |

### Chaîne complète pour B (détection de véhicules) — 5 étapes
1. **Acquisition** : caméra, choix de l'exposition, de la résolution et de l'éclairage.
2. **Prétraitement** : redimensionnement, normalisation des intensités, éventuel débruitage.
3. **Détection** : extraction de caractéristiques et prédiction des boîtes (classe + score) par un modèle (ex. CNN de type YOLO).
4. **Post-traitement** : seuil de confiance, suppression des doublons (NMS).
5. **Sortie et évaluation** : liste des véhicules détectés, comparée à une annotation (IoU, précision, rappel).

### Image améliorée vs liste de véhicules détectés
- Une **image améliorée** est une transformation *image → image* : même nature de donnée (un tableau de pixels), seule la qualité visuelle change (contraste, bruit…). Aucune interprétation n'est produite.
- Une **liste de véhicules détectés** est une *description structurée de la scène* (classes, positions, scores) : on passe des pixels à une information sémantique exploitable par une décision.

---

## Exercice 2 — Matrice image et région d'intérêt

Matrice de l'énoncé (4 lignes × 5 colonnes, lignes et colonnes indexées à partir de 0) :

| | c0 | c1 | c2 | c3 | c4 |
|---|---|---|---|---|---|
| **0** | 0 | 64 | 128 | 192 | 255 |
| **1** | 10 | 70 | 130 | 200 | 250 |
| **2** | 20 | 80 | 140 | 210 | 240 |
| **3** | 30 | 90 | 150 | 220 | 230 |

1. **H = 4, W = 5**, nombre de pixels = 4 × 5 = **20**. Type `uint8` (1 octet/valeur) → **20 octets**.
2. `I[1, 3]` = **200** (ligne 1, colonne 3). Le point (x = 2, y = 3) s'indexe `I[y, x]` = `I[3, 2]` = **150**. (NumPy : `[ligne, colonne]` ; x = colonne, y = ligne.)
3. `I[1:3, 2:5]` : lignes 1 et 2, colonnes 2, 3, 4 (bornes hautes exclues) :
   ```
   [[130 200 250]
    [140 210 240]]
   ```
   Forme : **(2, 3)**.
4. Image couleur (480, 640, 3) : pixels = 480 × 640 = **307 200** ; valeurs = 307 200 × 3 = **921 600**. Ils diffèrent car chaque pixel porte **3 valeurs** (un canal par composante de couleur).

---

## Exercice 3 — Échantillonnage et quantification

1. 1024 → 256 : facteur de réduction **4 sur chaque axe** (1024 / 256). Rapport des nombres de pixels : 4 × 4 = **16** (1 048 576 → 65 536 pixels).
2. Un agrandissement **interpole** : il calcule de nouvelles valeurs à partir des pixels restants mais ne peut pas retrouver les détails haute fréquence supprimés (information définitivement perdue). Sans **préfiltrage passe-bas** adapté avant le sous-échantillonnage, les fréquences trop élevées se replient en basses fréquences : c'est l'**aliasing** (repliement de spectre : moiré, contours en escalier, motifs parasites).
3. Nombre de niveaux = 2^b : **8 bits → 256**, **4 bits → 16**, **2 bits → 4**.
   Quantification sur [0, 85, 170, 255] au plus proche :
   - 20 → **0** (|20−0| = 20 < |20−85| = 65)
   - 90 → **85**
   - 170 → **170**
   - 240 → **255** (|240−255| = 15 < |240−170| = 70)

   Résultat : **[0, 85, 170, 255]**.
4. Image 1024 × 1024 = 1 048 576 pixels :
   - 8 bits/pixel → 1 048 576 octets = **1 MiB** ;
   - 2 bits/pixel compacts → 2 097 152 bits = 262 144 octets = **256 KiB** (facteur **4**).

   Un tableau NumPy `uint8` garde **1 octet par élément** : après quantification, les valeurs ne prennent que 4 états mais restent stockées dans un type 8 bits, donc la taille mémoire ne change pas. Il faudrait **empaqueter 4 pixels par octet** (manipulation de bits).

---

## Exercice 4 — Mémoire et débit vidéo

Image 1920 × 1080, RGB `uint8`, 30 images/s, données denses.

1. Taille d'une image : 1920 × 1080 × 3 = **6 220 800 octets** = **6,2208 MB** = 6 220 800 / 1 048 576 ≈ **5,933 MiB**.
2. Débit : 6,2208 MB × 30 = **186,624 MB/s** ; en bits : 186,624 × 8 = 1 492,992 Mbit/s ≈ **1,493 Gbit/s**.
3. 60 s : 186,624 MB/s × 60 = 11 197,44 MB ≈ **11,197 GB** (décimaux).
4. En `float32` (4 octets par composante) : 6 220 800 × 4 = **24 883 200 octets ≈ 24,88 MB** (×4 par rapport à `uint8`).
5. **YCbCr 4:2:0**, 8 bits par échantillon :
   - Y : 1920 × 1080 = 2 073 600 octets ;
   - Cb : 960 × 540 = 518 400 octets ; Cr : 518 400 octets ;
   - Total : **3 110 400 octets ≈ 3,11 MB**.

   Gain par rapport au 4:4:4 (6 220 800 octets) : facteur **2** (réduction de 50 %).
6. **Non.** La taille d'un fichier PNG ou JPEG dépend du **contenu** (redondance, textures, bruit), du **niveau de compression/qualité**, des **métadonnées** et de l'encodeur. Les dimensions donnent seulement la taille du tableau décodé, pas celle du fichier compressé : deux images de même taille peuvent donner des fichiers très différents (une image unie compresse bien mieux qu'une image bruitée).

---

## Exercice 5 — Acquisition et mouvement

Vitesse : 600 pixels/s, 30 images/s.

1. Période entre deux images : T = 1/30 s ≈ **33,3 ms**.
2. Déplacement pendant l'exposition (d = v · t) :
   - t = 1/30 s : 600 × 1/30 = **20 pixels** (flou de bougé marqué) ;
   - t = 1/600 s : 600 × 1/600 = **1 pixel**.
3. Pour un déplacement ≤ 2 pixels : t ≤ 2 / 600 = **1/300 s ≈ 3,33 ms**.
4. **Non** : la cadence fixe seulement la période entre deux images ; le temps d'exposition peut être **n'importe quelle valeur ≤ 1/30 s** (l'obturateur reste ouvert moins longtemps que la période). Compromis d'une exposition courte : **moins de lumière** captée → image plus sombre et plus **bruitée**, qu'il faut compenser par un éclairage plus fort, une ouverture plus grande ou un gain plus élevé (qui amplifie aussi le bruit).

---

## Exercice 6 — Histogrammes et sélection de couleur

Image A : 4 lignes identiques `[0, 0, 255, 255]`. Image B : lignes `[0,255,0,255]` et `[255,0,255,0]` en alternance, deux fois (4 × 4 pixels).

1. **A** : 8 pixels à 0 et 8 pixels à 255. **B** : chaque ligne contient 2 zéros et 2 valeurs 255 → 8 pixels à 0 et 8 à 255. Les histogrammes sont **identiques** : ils ne permettent **pas** de distinguer la structure (bandes verticales pour A, damier pour B). L'histogramme perd toute l'information **spatiale**.
2. **Non.** Une distribution concentrée vers les faibles intensités peut correspondre à une scène réellement sombre et correctement exposée (par exemple un chat noir sur fond sombre, une scène de nuit, un tableau noir). Inversement, une image sous-exposée peut présenter d'autres formes d'histogramme. Il faut connaître le contenu de la scène.
3. Gris du pixel RGB [255, 0, 0] : 0,299 × 255 + 0,587 × 0 + 0,114 × 0 = **76,245 ≈ 76**.
4. Avec `cv.COLOR_BGR2HSV` sur `uint8` : **H ∈ [0, 179]** (la teinte, 0–360°, est divisée par 2), **S ∈ [0, 255]**, **V ∈ [0, 255]**. Le rouge se trouve **aux deux extrémités du cercle des teintes** (autour de H = 0 et de H = 179) : un seul intervalle continu ne peut pas le couvrir sans inclure toutes les autres teintes ; on utilise donc **deux intervalles** (ex. [0, 10] et [170, 179]) combinés par un OU logique.
5. **Non.** Limites : (i) d'autres objets rouges (panneau, vêtement, enseigne) sont aussi sélectionnés : la couleur n'identifie pas la **catégorie** ; (ii) la **sensibilité à l'éclairage** (ombres, reflets, surexposition modifient S et V) : certaines parties de la voiture sont manquées et des zones de fond sont sélectionnées. (Autres limites : une voiture d'une autre couleur n'est pas détectée ; le masque ne sépare pas deux voitures rouges voisines.)

---

## Exercice 7 — Évaluation et indépendance des données

TP = 18, FP = 6, FN = 2.

1. **Précision** = TP / (TP + FP) = 18 / (18 + 6) = 18/24 = **0,75** (dénominateur : tous les pixels **prédits** positifs).
   **Rappel** = TP / (TP + FN) = 18 / (18 + 2) = 18/20 = **0,90** (dénominateur : tous les pixels **réellement** positifs).
2. **Faux positif** : un pixel sélectionné comme rouge alors qu'il n'appartient pas à la voiture annotée (ex. pixel d'un panneau rouge ou d'un reflet). **Faux négatif** : un pixel de la voiture rouge non sélectionné (ex. zone très sombre ou surexposée dont S ou V sort des seuils).
3. L'étudiant **règle les seuils sur le jeu de test** puis évalue sur ce même jeu : le test n'est plus indépendant (**fuite de données / surapprentissage sur le test**), la performance annoncée est **optimiste** et ne mesure pas la généralisation. **Correction** : séparer en jeux d'**apprentissage/réglage**, de **validation** et de **test** ; régler les seuils sur validation, figer les paramètres, puis évaluer **une seule fois** sur le test.
4. Les images successives d'une même vidéo sont **quasi identiques** (mêmes objets, fond, éclairage). Un partage aléatoire place des images presque jumelles dans l'apprentissage et dans le test : le modèle « reconnaît » la scène plutôt qu'il ne généralise, d'où une performance **surestimée**. Il faut séparer **par vidéo / par scène** (voire par jour ou par site).
5. **Protocole simple** : (i) photographier la même scène/objet sous plusieurs éclairages (plein soleil, ombre, nuit éclairée, lampe artificielle) ; (ii) annoter une vérité terrain au niveau pixel ; (iii) fixer les seuils HSV sur un sous-ensemble de réglage dans un seul éclairage ; (iv) mesurer **précision et rappel** de façon séparée pour chaque condition d'éclairage ; (v) comparer et reporter la dégradation, éventuellement en simulant des variations de V (luminosité ×0,5 ; ×1,5).
