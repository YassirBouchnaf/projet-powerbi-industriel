# 📐 Mesures DAX - Tableau de Bord Industriel

---

## INDICATEURS MAINTENANCE

```dax
-- Coût total de maintenance
Coût Maintenance = SUM('fact_industriel'[cout_maintenance])

-- Coût total global
Coût total = SUM('fact_industriel'[cout_total])

-- Part du coût maintenance sur le coût total
% Coût maintenance = DIVIDE([Coût Maintenance], [Coût total], 0)

-- Somme des temps d'arrêt
Temps d'arrêt total = SUM('fact_industriel'[temps_arret])

-- Ratio temps d'arrêt sur heures travaillées
Taux d'arrêt maintenance = DIVIDE([Temps d'arrêt total], SUM('fact_industriel'[heures_travail]), 0)
```

---

## INDICATEURS PRODUCTION

```dax
-- Coût total de production
Cout_total = SUM(fact_industriel[cout_total])

-- Objectif de coût (cible fixe)
Objectif_Cout = 100000

-- Ratio coût réel / objectif
Taux_Cout = DIVIDE([Cout_total], [Objectif_Cout])

-- Heures travaillées moins temps d'arrêt
Temps utile = SUM('fact_industriel'[heures_travail]) - SUM('fact_industriel'[temps_arret])

-- Disponibilité = temps utile / heures travaillées
Disponibilité production = DIVIDE([Temps utile], SUM('fact_industriel'[heures_travail]), 0)

-- Taux d'utilisation machine moyen
Taux utilisation machine = AVERAGE('fact_industriel'[taux_utilisation_machine])

-- Productivité globale = disponibilité × taux utilisation
Productivité globale = [Disponibilité production] * DIVIDE([Taux utilisation machine], 100, 0)

-- Productivité machine pondérée par les heures
Productivité machine =
DIVIDE(
    SUMX('fact_industriel', 'fact_industriel'[taux_utilisation_machine] * 'fact_industriel'[heures_travail]),
    SUM('fact_industriel'[heures_travail]),
    0
)

-- Moyenne simple du taux d'utilisation
Productivité moyenne = AVERAGE('fact_industriel'[taux_utilisation_machine])

-- Productivité filtrée sur le service Méthode
Productivité méthodes =
CALCULATE(
    AVERAGE('fact_industriel'[taux_utilisation_machine]),
    dim_service[service_name] = "Methode"
)

-- Rendement = temps utile / heures travaillées
Rendement = DIVIDE([Temps utile], SUM('fact_industriel'[heures_travail]), 0)
```

---

## INDICATEURS QUALITÉ

```dax
-- Total pièces défectueuses
Total défauts = SUM(fact_qualite[nb_defauts])

-- Total pièces contrôlées
Total contrôlé = SUM(fact_qualite[nb_pieces_controlees])

-- Taux de défaut = défauts / contrôlées
Taux de défaut = DIVIDE([Total défauts], [Total contrôlé], 0)

-- Coût total de non-qualité
Coût non qualité = SUM(fact_qualite[cout_non_qualite])

-- Coût moyen par défaut
Coût par défaut = DIVIDE([Coût non qualité], [Total défauts], 0)

-- Impact qualité = défauts par heure travaillée
Impact qualité = DIVIDE([Total défauts], SUM('fact_industriel'[heures_travail]), 0)

-- Cible qualité (seuil maximum accepté)
CIBLEQ = 0.02

-- Coloration conditionnelle basée sur la cible
coleur_tauxDefaut = IF([Taux de défaut] > [CIBLEQ], "RED", "GREEN")
```

---

## INDICATEURS FABRICATION MÉCANIQUE

```dax
-- TRS/OEE = Disponibilité × Performance × Qualité
TauxRendementSynthétique_TRS/OEE =

VAR Disponibilite =
    DIVIDE(
        SUM(fact_industriel[heures_travail]) - SUM(fact_industriel[temps_arret]),
        SUM(fact_industriel[heures_travail])
    )

VAR Performance =
    AVERAGE(fact_industriel[taux_utilisation_machine]) / 100

VAR Qualite =
    1 - DIVIDE(
        SUM(fact_qualite[nb_defauts]),
        SUM(fact_qualite[nb_pieces_controlees])
    )

RETURN
    Disponibilite * Performance * Qualite

-- Coût total filtré sur Fabrication Mécanique
Coût mécanique =
CALCULATE(
    SUM('fact_industriel'[cout_total]),
    dim_service[service_name] = "Fabrication_mecanique"
)
```

---

## COLONNE CALCULÉE - Dim_date

```dax
-- Numéro du mois pour tri chronologique
MonthNumberr = MONTH(Dim_date[Date])
```

---

## NOTES IMPORTANTES

- `Performance` dans le TRS/OEE est divisé par 100 car `taux_utilisation_machine` est stocké en valeur entière (ex: 70 = 70%), pas en décimal (0.70).
- `coleur_tauxDefaut` retourne "RED" quand le taux dépasse la cible (mauvais) et "GREEN" quand il est sous la cible (bon) - utilisé dans le formatage conditionnel des visuels.
- `CIBLEQ = 0.02` représente le seuil cible de 2% de taux de défaut.
