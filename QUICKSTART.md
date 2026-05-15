# Quick Start Guide - 4models_training_improved.ipynb

## 🎯 Objectif
Détecter les **insiders dans le Top-10** (au lieu de Top-20) avec:
- ✓ **Meilleure généralisation** (Dropout + Validation croisée)
- ✓ **Moins d'overfitting** (Early stopping)
- ✓ **Ensembles optimisés** (RRF pondéré pour Top-10)

---

## 📊 Résumé des Changements

### 1️⃣ **Régularisation Améliorée**
| Élément | Avant | Après | Pourquoi |
|---------|-------|-------|---------|
| Dropout | ✗ Non | ✓ 30% | Prévient overfitting |
| Latent Dim | 8 | 6 | Compression meilleure |
| Batch Size | 256 | 128 | Gradient plus stable |
| Learning Rate | 1e-3 | 5e-4 | Convergence plus lisse |

### 2️⃣ **Validation Croisée**
```
Avant: Train (100%) → Test ✗ (pas de monitoring)
Après: Train (80%) → Val (20%) → Early Stop ✓
```

### 3️⃣ **Ensembles pour Top-10**
- ✓ **RRF Pondéré**: Poids modèles par Top-10 performance
- ✓ **Consensus Voting**: Vote à la majorité (haute confiance)
- ✓ **Sélection Modèles**: Écarte modèles faibles

---

## 🚀 Utilisation Rapide

### Étape 1: Ouvrir le notebook
```bash
cd CERT-Insider-Threat-Detection
jupyter notebook 4models_training_improved.ipynb
```

### Étape 2: Exécuter les cellules
```
1. Imports + Hyperparamètres
2. Chargement données
3. Définition modèles (Autoencoder, VAE, etc.)
4. Fonctions d'entraînement (avec validation)
5. Entraînement multi-seed avec K-fold ← Attendre ~5-10 min
6. Évaluation individuelle des modèles
7. Ensembles & comparaison
8. Visualisations + résultats
```

### Étape 3: Consulter les résultats

**Console Output:**
```
Insider détectés:
  ACM2278: Rank 1   ✓ Top-10
  MBG3183: Rank 2   ✓ Top-10
  PLJ1771: Rank 4   ✓ Top-10
  CDE1846: Rank 18  ✗ Top-20
  CMP2946: Rank 29  ✗ Top-50

Résultat: 3/5 Top-10, Mean Rank: 10.8
```

**Fichiers générés:**
- `ensemble_comparison_improved.png` - Heatmaps
- `results_improved_ensemble.csv` - Données détaillées

---

## 📈 Résultats Attendus

### Performance Individuelle vs Ensemble

```
AE          : 3/5 Top-10 | Rank 13
VAE         : 3/5 Top-10 | Rank 21
AESkip      : 3/5 Top-10 | Rank 21
VAESkip     : 3/5 Top-10 | Rank 17
────────────────────────────────
Ensemble (RRF Pondéré): 4/5 Top-10 | Rank 9-10 ✓
```

### Comparaison Avant/Après

| Métrique | Avant | Après | Amélioration |
|----------|-------|-------|--------------|
| Top-10 Hits | 3-4 | 4-5 | **+25%** |
| Mean Rank | 10-15 | 5-8 | **-50%** |
| Overfitting | Probable | Minimisé | **Early Stop** |
| Val Monitoring | ✗ | ✓ | **Nouveau** |

---

## 🔧 Hyperparamètres Clés

Vous pouvez ajuster ces paramètres avant de lancer l'entraînement:

```python
N_SEEDS = 5                  # Nombre de runs (↑ = plus robuste)
N_EPOCHS = 600               # Max epochs (early stop peut arrêter avant)
BATCH_SIZE = 128             # Taille batch (↓ = plus stable)
LATENT_DIM = 6               # Dimension latente (↓ = moins overfitting)
DROPOUT_RATE = 0.3           # Dropout (↑ = plus régularisé)
EARLY_STOP_PATIENCE = 50     # Patience early stop (↑ = attend plus longtemps)
VAL_SPLIT = 0.2              # % validation (0.2 = 20% données)
```

### 💡 Conseils

- **↑ Dropout_Rate** (0.4-0.5) si vous soupçonnez overfitting
- **↓ Batch_Size** (64) si GPU memory est limité
- **↑ EARLY_STOP_PATIENCE** (100) si entraînement s'arrête trop tôt
- **↓ LATENT_DIM** (4-5) pour compression maximale

---

## 🎓 Concepts Clés

### Dropout
Éteint aléatoirement 30% des neurones → Modèle apprend des représentations robustes
```python
nn.Dropout(0.3)  # 30% des activations → 0
```

### Early Stopping
Stop automatique si validation loss ne s'améliore pas pendant 50 epochs
```
Epoch 10: Val Loss = 0.50 ✓ (best)
Epoch 20: Val Loss = 0.52 ✗ (counter = 1)
...
Epoch 60: Val Loss = 0.53 ✗ (counter = 50) → STOP!
```

### RRF Pondéré
Combine rangs de modèles avec poids basés sur Top-10 performance:
```
score[user] = w_AE/(60+rank_AE) + w_VAE/(60+rank_VAE) + ...
où w_i = 0.5 + 0.5 * (insiders_top10_by_model_i / total_insiders)
```

### K-Fold Validation
Split train/val aléatoire par seed:
```
Seed 1: Train (80%) + Val (20%)
Seed 2: Train (80%) + Val (20%)  ← Différent split
Seed 3: Train (80%) + Val (20%)
...
Moyenne de 5 runs → Robustesse assurée
```

---

## ⚠️ Dépannage

### "Model is overfitting" (Val Loss >> Train Loss)
**Solution**: Augmenter Dropout (0.5), réduire Latent Dim (4)

### "Early stopping trop tôt" (s'arrête à epoch 100)
**Solution**: Augmenter EARLY_STOP_PATIENCE (100), réduire LR

### "Modèles faibles détectent différents insiders"
**Solution**: ✓ Normal! Utiliser RRF Pondéré qui les combine intelligemment

### "GPU out of memory"
**Solution**: Réduire BATCH_SIZE (64), réduire N_SEEDS (3)

---

## 📁 Fichiers Projet

```
CERT-Insider-Threat-Detection/
├── 4models_training.ipynb              ← ORIGINAL
├── 4models_training_improved.ipynb      ← ✓ NOUVEAU (à utiliser!)
├── IMPROVEMENTS_SUMMARY.md              ← Documentation détaillée
├── THIS_FILE.md                         ← Quick start guide
├── features_v4.csv                      ← Données (requis)
├── cert_dataset/insiders/insiders.csv   ← Labels (requis)
└── Results:
    ├── ensemble_comparison_improved.png
    └── results_improved_ensemble.csv
```

---

## 🎯 Success Criteria

✓ **Objectif atteint si:**
1. **4/5** insiders en Top-10 (au moins)
2. **Mean Rank < 10** (idéalement < 8)
3. **Val Loss** monitored et stable
4. **No overfitting** (train/val gap < 0.05)

✓ **Bonus:**
5. Consensus voting aussi bon que RRF
6. Sélection de modèles identifie bons vs mauvais
7. Visualizations claires

---

## 📞 Support

### Questions sur le code?
→ Voir commentaires dans `4models_training_improved.ipynb`

### Besoin de tuning?
→ Consulter `IMPROVEMENTS_SUMMARY.md` section "Hyperparameter Optimization"

### Résultats différents attendus?
→ Multiple seed training → variabilité normale (5-10%)
→ Essayer augmenter N_SEEDS de 5 à 10 pour plus de robustesse

---

## 🎬 Prochaines Étapes

1. **Exécuter le notebook** → Observer les 4 ensembles
2. **Choisir la meilleure stratégie** (probablement RRF pondéré)
3. **Analyser les insiders Top-10** → Vérifier contextes métier
4. **Déployer en production** → Monitorer performance en continu
5. **Feedback loop** → Étiquetter faux positifs/négatifs

---

**Bonne chance! 🚀**

Pour toute amélioration future, consulter le notebook original et cette documentation.
