📊 Tableau de Bord Industriel - Power BI

Un tableau de bord interactif de 4 pages analysant les performances industrielles d'une usine à travers 4 axes : Production, Qualité, Maintenance et Fabrication Mécanique.

📁 Structure du projet
├── DAX/
│   └── measures_industriel.md       # Toutes les mesures DAX du projet
├── Screenshots/
│   ├── page1_vue_ensemble.png
│   ├── page2_production.png
│   ├── page3_qualite.png
│   └── page4_maintenance.png
├── Projet power bi.pbix             # Fichier Power BI principal
└── README.md
📄 Pages du tableau de bord
1. Vue d'ensemble

Vue globale de la santé de l'usine en un coup d'oeil.

KPI cards : Coût total, TRS/OEE, Taux de défaut, Temps d'arrêt total, Coût Maintenance
Jauge dynamique : TRS/OEE avec min/max dynamiques (minTRS / maxTRS)
Graphique en courbes : Evolution des indicateurs clés par mois (Coût total, Temps d'arrêt)
Formatage conditionnel : Taux de défaut en rouge si supérieur à la cible 2%
2. Analyse de Production

Focus sur l'efficacité et les coûts de production.

KPI cards : Productivité globale, Productivité machine, Productivité moyenne, Disponibilité production
Graphique combiné : Coût total vs Objectif (barres + ligne de référence)
Petits multiples : Productivité machine par service (grille comparative)
Jauge : Disponibilité production
3. Analyse Qualité

Page la plus avancée - analyse des causes racines des défauts.

KPI cards : Total défauts, Total contrôlé, Coût non qualité, Coût par défaut
Graphique Pareto : Défauts par type triés par ordre décroissant
Carte de chaleur (Heatmap) : Matrice Ligne_production x Type de défaut avec formatage conditionnel (blanc vers rouge)
Treemap : Répartition des défauts par ligne et type (visualisation proportionnelle)
Graphique à barres : Taux de défaut par ligne de production avec ligne cible CIBLEQ (2%) et coloration conditionnelle
4. Maintenance & Fabrication Mécanique

Analyse des coûts et temps d'arrêt maintenance.

KPI cards : Coût Maintenance, % Coût maintenance, Taux d'arrêt maintenance, Coût mécanique
Jauge : Taux d'arrêt maintenance
Nuage de points : Coût Maintenance vs Temps d'arrêt par service (taille = coût)
Graphique à barres horizontales : Temps d'arrêt total par service
📐 Modèle de données
Table	Description
fact_industriel	Table de faits principale - couts, heures, temps d'arrêt (100 lignes)
fact_qualite	Données qualité - défauts, pièces contrôlées, coûts non qualité (100 lignes)
dim_service	Dimension service - 7 services (Production, Maintenance, Qualité...)
Dim_date	Dimension date - 2024, janvier à avril
📏 Mesures DAX principales
INDICATEURS MAINTENANCE
Mesure	Formule
Coût Maintenance	SUM(fact_industriel[cout_maintenance])
Coût total	SUM(fact_industriel[cout_total])
% Coût maintenance	DIVIDE([Cout Maintenance], [Cout total], 0)
Temps d'arrêt total	SUM(fact_industriel[temps_arret])
Taux d'arrêt maintenance	DIVIDE([Temps d'arret total], SUM(fact_industriel[heures_travail]), 0)
INDICATEURS PRODUCTION
Mesure	Formule
Cout_total	SUM(fact_industriel[cout_total])
Objectif_Cout	100000
Taux_Cout	DIVIDE([Cout_total], [Objectif_Cout])
Disponibilité production	DIVIDE([Temps utile], SUM(fact_industriel[heures_travail]), 0)
Productivité globale	[Disponibilite production] * DIVIDE([Taux utilisation machine], 100, 0)
Productivité moyenne	AVERAGE(fact_industriel[taux_utilisation_machine])
INDICATEURS QUALITE
Mesure	Formule
Total défauts	SUM(fact_qualite[nb_defauts])
Total contrôlé	SUM(fact_qualite[nb_pieces_controlees])
Taux de défaut	DIVIDE([Total defauts], [Total controle], 0)
Coût non qualité	SUM(fact_qualite[cout_non_qualite])
CIBLEQ	0.02
coleur_tauxDefaut	IF([Taux de defaut] > [CIBLEQ], "RED", "GREEN")
INDICATEURS FABRICATION MECANIQUE
Mesure	Formule
TRS/OEE	Disponibilite x Performance x Qualite
Coût mécanique	CALCULATE(SUM(fact_industriel[cout_total]), dim_service[service_name] = "Fabrication_mecanique")
🛠️ Technologies utilisées
Power BI Desktop
DAX (Data Analysis Expressions)
Power Query (M)
👨‍💻 Auteur

Yassir Bouchnaf
Projet réalisé dans le cadre d'un cours Power BI - Juin 2026
