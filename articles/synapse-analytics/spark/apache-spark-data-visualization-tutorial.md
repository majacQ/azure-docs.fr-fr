---
title: Visualiser les données avec Apache Spark
description: Créer des visualisations de données enrichies à l’aide d’Apache Spark et des notebooks Azure Synapse Analytics
services: synapse-analytics
author: midesa
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: machine-learning
ms.date: 10/20/2020
ms.author: midesa
ms.openlocfilehash: beef6768f9b2fb05efb77c16c32b0acbe46d1e85
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122524445"
---
# <a name="analyze-data-with-apache-spark"></a>Analyser des données avec Apache Spark

Dans ce tutoriel, vous découvrirez comment effectuer une analyse exploratoire des données à l’aide d’Azure Open Datasets et d’Apache Spark. Vous visualiserez ensuite les résultats dans un notebook Synapse Studio dans Azure Synapse Analytics.

Nous analyserons en particulier le jeu de données [New York City (NYC) Taxi](https://azure.microsoft.com/services/open-datasets/catalog/nyc-taxi-limousine-commission-yellow-taxi-trip-records/). Les données sont disponibles dans Azure Open Datasets. Ce sous-ensemble du jeu de données contient des données sur les courses de taxis jaunes, qui incluent des informations sur chaque course, les heures et lieux de départ et d’arrivée, le prix et d’autres attributs intéressants.
  
## <a name="before-you-begin"></a>Avant de commencer
Créez un pool Apache Spark en suivant le [didacticiel Créer un pool Apache Spark](../articles/../quickstart-create-apache-spark-pool-studio.md). 

## <a name="download-and-prepare-the-data"></a>Télécharger et préparer les données
1. Créez un notebook en utilisant le noyau PySpark. Pour obtenir des instructions, consultez [Créer un notebook](../quickstart-apache-spark-notebook.md#create-a-notebook). 
   
   > [!Note]
   > Grâce au noyau PySpark, il est inutile de créer des contextes explicitement. Le contexte Spark est créé automatiquement pour vous lorsque vous exécutez la première cellule de code.

2. Dans ce tutoriel, nous allons utiliser différentes bibliothèques pour visualiser le jeu de données. Pour effectuer cette analyse, importez les bibliothèques suivantes : 

   ```python
   import matplotlib.pyplot as plt
   import seaborn as sns
   import pandas as pd
   ```

3. Comme les données brutes sont au format Parquet, vous pouvez utiliser le contexte Spark pour extraire le fichier en mémoire directement en tant que DataFrame. Créez un DataFrame Spark en extrayant les données via l’API Open Datasets. Ici, nous utilisons les propriétés de *schéma lors de la lecture* du DataFrame Spark pour déduire les types de données et le schéma.

   ```python
   from azureml.opendatasets import NycTlcYellow
   from datetime import datetime
   from dateutil import parser

   end_date = parser.parse('2018-06-06')
   start_date = parser.parse('2018-05-01')
   nyc_tlc = NycTlcYellow(start_date=start_date, end_date=end_date)
   df = nyc_tlc.to_spark_dataframe()
   ```

4. Une fois les données lues, nous effectuerons un premier filtrage pour nettoyer le jeu de données. Nous pourrons supprimer les colonnes inutiles et ajouter des colonnes supplémentaires qui extraient des informations importantes. En outre, nous filtrerons également les anomalies dans le jeu de données.

   ```python
   # Filter the dataset 
   from pyspark.sql.functions import *

   filtered_df = df.select('vendorID', 'passengerCount', 'tripDistance','paymentType', 'fareAmount', 'tipAmount'\
                                   , date_format('tpepPickupDateTime', 'hh').alias('hour_of_day')\
                                   , dayofweek('tpepPickupDateTime').alias('day_of_week')\
                                   , dayofmonth(col('tpepPickupDateTime')).alias('day_of_month'))\
                               .filter((df.passengerCount > 0)\
                                   & (df.tipAmount >= 0)\
                                   & (df.fareAmount >= 1) & (df.fareAmount <= 250)\
                                   & (df.tripDistance > 0) & (df.tripDistance <= 200))

   filtered_df.createOrReplaceTempView("taxi_dataset")
   ```

## <a name="analyze-data"></a>Analyser des données
En tant qu’analyste de données, vous disposez d’un large éventail d’outils pour vous aider à extraire des informations à partir de ces données. Dans cette partie du tutoriel, nous allons découvrir quelques outils utiles disponibles dans les notebooks Azure Synapse Analytics. Dans cette analyse, nous souhaitons comprendre les facteurs qui donnent lieu à des pourboires plus élevés pour la période sélectionnée.

### <a name="apache-spark-sql-magic"></a>Magic SQL Apache Spark 
Tout d’abord, nous allons effectuer une analyse exploratoire des données en utilisant des commandes magic et SQL Apache Spark avec le notebook Azure Synapse. Une fois que nous avons notre requête, nous allons visualiser les résultats à l’aide de la capacité ```chart options``` intégrée. 

1. Dans votre notebook, créez une cellule et copiez le code ci-dessous. À l’aide de cette requête, nous souhaitons comprendre la façon dont les montants moyens des pourboires ont changé au cours de la période sélectionnée. Cette requête nous aidera également à identifier d’autres informations utiles, notamment le montant minimal/maximal d’un pourboire par jour et le montant moyen des tarifs.
   
   ```sql
   %%sql
   SELECT 
       day_of_month
       , MIN(tipAmount) AS minTipAmount
       , MAX(tipAmount) AS maxTipAmount
       , AVG(tipAmount) AS avgTipAmount
       , AVG(fareAmount) as fareAmount
   FROM taxi_dataset 
   GROUP BY day_of_month
   ORDER BY day_of_month ASC
   ```

2. Une fois l’exécution de la requête terminée, nous pouvons visualiser les résultats en basculant sur la vue graphique. Dans cet exemple, nous allons créer un graphique en courbes en spécifiant le champ ```day_of_month``` comme clé et ```avgTipAmount``` comme valeur. Une fois que vous avez effectué les sélections, sélectionnez **Appliquer** pour actualiser votre graphique. 
   
## <a name="visualize-data"></a>Visualiser les données
Outre les options intégrées de création de graphiques du notebook, vous pouvez utiliser des bibliothèques open source populaires pour créer vos propres visualisations. Dans les exemples suivants, nous allons utiliser Seaborn et Matplotlib. Il s’agit des bibliothèques Python couramment utilisées pour la visualisation des données. 

> [!Note]
> Par défaut, chaque pool Apache Spark dans Azure Synapse Analytics contient un ensemble de bibliothèques par défaut couramment utilisées. Vous pouvez afficher la liste complète des bibliothèques dans la documentation sur le [runtime Azure Synapse](../spark/apache-spark-version-support.md). De plus, pour que vos applications disposent d’un code tiers ou généré en local, vous pouvez [installer une bibliothèque](../spark/apache-spark-azure-portal-add-libraries.md) sur l’un de vos pools Spark.

1. Pour faciliter le développement et le rendre moins onéreux, nous allons réduire l’échantillon de jeu de données. Nous allons utiliser la capacité intégrée d’échantillonnage Apache Spark. En outre, Seaborn et Matplotlib nécessitent un DataFrame Pandas ou un tableau NumPy. Pour obtenir un DataFrame Pandas, utilisez la commande ```toPandas()``` pour convertir notre DataFrame.

   ```python
   # To make development easier, faster, and less expensive, downsample for now
   sampled_taxi_df = filtered_df.sample(True, 0.001, seed=1234)

   # The charting package needs a Pandas DataFrame or NumPy array to do the conversion
   sampled_taxi_pd_df = sampled_taxi_df.toPandas()
   ```

1. Nous souhaitons comprendre la distribution des pourboires dans notre jeu de données. Nous allons utiliser Matplotlib pour créer un histogramme montrant la répartition de la quantité et du nombre de pourboires. Sur la base de la distribution, nous pouvons voir que les pourboires sont biaisés autour de montants inférieurs ou égaux à 10 $.

   ```python
   # Look at a histogram of tips by count by using Matplotlib

   ax1 = sampled_taxi_pd_df['tipAmount'].plot(kind='hist', bins=25, facecolor='lightblue')
   ax1.set_title('Tip amount distribution')
   ax1.set_xlabel('Tip Amount ($)')
   ax1.set_ylabel('Counts')
   plt.suptitle('')
   plt.show()
   ```

   ![Histogramme des pourboires.](./media/apache-spark-machine-learning-mllib-notebook/histogram.png)

1. Ensuite, nous souhaitons comprendre la relation entre les pourboires pour un trajet donné et le jour de la semaine. Utilisez Seaborn pour créer un diagramme à surfaces résumant les tendances pour chaque jour de la semaine. 

   ```python
   # View the distribution of tips by day of week using Seaborn
   ax = sns.boxplot(x="day_of_week", y="tipAmount",data=sampled_taxi_pd_df, showfliers = False)
   ax.set_title('Tip amount distribution per day')
   ax.set_xlabel('Day of Week')
   ax.set_ylabel('Tip Amount ($)')
   plt.show()

   ```
   ![Graphique montrant la répartition des pourboires par jour.](./media/apache-spark-data-viz/data-analyst-tutorial-per-day.png)

4. Une autre hypothèse que nous pouvons faire est qu’il existe une relation positive entre le nombre de passagers et le montant total du pourboire des taxis. Pour vérifier cette relation, exécutez le code suivant pour générer un diagramme à surfaces illustrant la répartition des pourboires pour chaque nombre de passagers. 
   
   ```python
   # How many passengers tipped by various amounts 
   ax2 = sampled_taxi_pd_df.boxplot(column=['tipAmount'], by=['passengerCount'])
   ax2.set_title('Tip amount by Passenger count')
   ax2.set_xlabel('Passenger count')
   ax2.set_ylabel('Tip Amount ($)')
   ax2.set_ylim(0,30)
   plt.suptitle('')
   plt.show()
   ```
   ![Graphique qui affiche un tracé de boîte à moustaches.](./media/apache-spark-machine-learning-mllib-notebook/box-whisker-plot.png)

1. Enfin, nous souhaitons comprendre la relation entre le montant du pourboire et le montant de la course. Sur la base des résultats, nous pouvons constater qu’il y a plusieurs observations où les gens ne donnent pas de pourboire. Cependant, nous constatons également une relation positive entre le prix global de la course et le montant du pourboire.
   
   ```python
   # Look at the relationship between fare and tip amounts

   ax = sampled_taxi_pd_df.plot(kind='scatter', x= 'fareAmount', y = 'tipAmount', c='blue', alpha = 0.10, s=2.5*(sampled_taxi_pd_df['passengerCount']))
   ax.set_title('Tip amount by Fare amount')
   ax.set_xlabel('Fare Amount ($)')
   ax.set_ylabel('Tip Amount ($)')
   plt.axis([-2, 80, -2, 20])
   plt.suptitle('')
   plt.show()
   ```
   ![Nuage de points du montant du pourboire.](./media/apache-spark-machine-learning-mllib-notebook/scatter.png)

## <a name="shut-down-the-spark-instance"></a>Arrêter l’instance Spark

Une fois l’exécution de l’application terminée, arrêtez le notebook pour libérer les ressources. Fermez l’onglet ou sélectionnez **Terminer la session** dans le panneau d’état au bas du notebook.

## <a name="see-also"></a>Voir aussi

- [Vue d’ensemble : Apache Spark sur Azure Synapse Analytics](apache-spark-overview.md)
- [Créer un modèle Machine Learning avec Apache SparkML](../spark/apache-spark-machine-learning-mllib-notebook.md)

## <a name="next-steps"></a>Étapes suivantes

- [Azure Synapse Analytics](../index.yml)
- [Documentation officielle Apache Spark](https://spark.apache.org/docs/latest/)