---
title: Qu’est-ce que le Machine Learning (ML) automatisé ? AutoML
titleSuffix: Azure Machine Learning
description: Découvrez comment Azure Machine Learning peut générer automatiquement un modèle en utilisant les paramètres et les critères que vous fournissez avec le Machine Learning automatisé.
services: machine-learning
ms.service: machine-learning
ms.subservice: automl
ms.topic: conceptual
author: cartacioS
ms.author: sacartac
ms.date: 10/21/2021
ms.custom: automl
ms.openlocfilehash: 16d96eb508725e22bc1956a8b78d003f1512487b
ms.sourcegitcommit: 2ed2d9d6227cf5e7ba9ecf52bf518dff63457a59
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/16/2021
ms.locfileid: "132518657"
---
# <a name="what-is-automated-machine-learning-automl"></a>Qu’est-ce que le Machine Learning automatisé (AutoML) ?

Le Machine Learning automatisé, également appelé ML automatisé ou AutoML, est le processus d’automatisation des tâches fastidieuses et itératives de développement de modèle Machine Learning. Il permet aux chercheurs de données, analystes et développeurs de créer des modèles ML à grande échelle, efficaces et productifs, tout en maintenant la qualité du modèle. L’apprentissage automatique automatisé dans Azure Machine Learning se base sur une innovation de notre [division Microsoft Research](https://www.microsoft.com/research/project/automl/).

Le développement de modèle Machine Learning traditionnel consomme beaucoup de ressources, nécessitant une connaissance significative du domaine et du temps pour produire et comparer des dizaines de modèles. Le Machine Learning automatisé permet de réduire le temps nécessaire pour obtenir des modèles ML prêts pour la production avec une facilité et une efficacité extraordinaires.

<a name="parity"></a>

## <a name="ways-to-use-automl-in-azure-machine-learning"></a>Façons d’utiliser AutoML dans Azure Machine Learning

Azure Machine Learning offre les deux expériences suivantes pour utiliser le Machine Learning automatisé. Consultez les sections suivantes pour comprendre la [disponibilité des fonctionnalités dans chaque expérience](#parity).

* Pour les clients expérimentés en codage, [Kit SDK Python d’Azure Machine Learning](/python/api/overview/azure/ml/intro).  Lancez-vous avec [Tutoriel : utiliser le Machine Learning automatisé pour prédire le prix des tarifs de taxi](tutorial-auto-train-models.md).

* Pour les clients avec une expérience limitée ou inexistante en codage, Azure Machine Learning studio sur [https://ml.azure.com](https://ml.azure.com/).  Démarrage avec ces tutoriels :
    * [Tutoriel : créer un modèle de classification avec le ML automatisé dans Azure Machine Learning](tutorial-first-experiment-automated-ml.md).
    *  [Tutoriel : Prévoir la demande avec le Machine Learning automatisé](tutorial-automated-ml-forecast.md)

### <a name="experiment-settings"></a>Paramètres de l’expérience 

Les paramètres suivants vous permettent de configurer votre expérience de machine learning automatisé. 

| |SDK Python|L’expérience web Studio|
|----|:----:|:----:|
|**Division des données en jeux d’entraînement/de validation**| ✓|✓
|**Prise en charge des tâches de Machine Learning : classification, régression et prévision**| ✓| ✓
|**Prise en charge des tâches de vision par ordinateur (préversion) : classification d’images, détection d’objet et segmentation d’instance**| ✓| 
|**Optimisation basée sur une métrique principale**| ✓| ✓
|**Prise en charge du calcul Azure ML en tant que cible de calcul** | ✓|✓
|**Configuration de l’horizon de prévision, des décalages de cibles et de la fenêtre dynamique**|✓|✓
|**Définition des critères de sortie** |✓|✓ 
|**Définition des itérations simultanées**| ✓|✓
|**Suppression de colonnes**| ✓|✓
|**Blocage des algorithmes**|✓|✓
|**Validation croisée** |✓|✓
|**Prise en charge de la formation sur les clusters Azure Databricks**| ✓|
|**Affichage des noms de caractéristiques traités**|✓|
|**Résumé de la caractérisation**| ✓|
|**Caractérisation des congés**|✓|
|**Niveaux de verbosité des fichiers journaux**| ✓|

### <a name="model-settings"></a>Paramètres du modèle

Ces paramètres peuvent être appliqués au meilleur modèle à la suite de votre expérience de machine learning automatisé.

| |SDK Python|L’expérience web Studio|
|----|:----:|:----:|
|**Inscription, déploiement et description du meilleur modèle**| ✓|✓|
|**Activation des modèles de l’ensemble de vote et de l’ensemble d’empilement**| ✓|✓|
|**Affichage du meilleur modèle selon une métrique non principale**|✓||
|**Activation/désactivation de la compatibilité du modèle ONNX**|✓||
|**Tester le modèle** | ✓| ✓ (préversion)|

### <a name="run-control-settings"></a>Paramètres de contrôle d’exécution

Ces paramètres vous permettent d’examiner et de contrôler les exécutions de votre expérience et ses exécutions enfants. 

| |SDK Python|L’expérience web Studio|
|----|:----:|:----:|
|**Exécuter le tableau récapitulatif**| ✓|✓|
|**Annuler les exécutions et les exécutions enfants**| ✓|✓|
|**Obtenir des garde-fous**| ✓|✓|
|**Mettre en pause et reprendre les exécutions**| ✓| |

## <a name="when-to-use-automl-classification-regression-forecasting--computer-vision"></a>Quand utiliser AutoML : classification, régression, prévision et vision par ordinateur

Appliquez le Machine Learning automatisé lorsque vous souhaitez qu’Azure Machine Learning effectue l’apprentissage d’un modèle et le règle à votre place à l’aide de la métrique cible que vous spécifiez. Le ML automatisé démocratise le processus de développement de modèle Machine Learning et permet à ses utilisateurs, quel que soit leur expertise en matière de science des données, d’identifier un pipeline de Machine Learning de bout en bout pour tout problème.

Les scientifiques des données, analystes et développeurs de différents secteurs peuvent utiliser le Machine Learning automatisé pour ce qui suit :
+ Implémenter des solutions ML sans disposer d’une connaissance approfondie de la programmation.
+ Économiser du temps et des ressources.
+ Tirer parti des bonnes pratiques de science des données.
+ Fournir une résolution de problème agile.

### <a name="classification"></a>classification ;

La classification est une tâche Machine Learning courante. Il s’agit d’un type d’apprentissage supervisé dans lequel des modèles apprennent à utiliser des données d’apprentissage et appliquent ces apprentissages à de nouvelles données. Azure Machine Learning offre des caractérisations spécifiquement pour ces tâches, telles que des caractériseurs de réseau neuronal profond pour la classification. Apprenez-en davantage sur les [optons de caractérisation](how-to-configure-auto-features.md#featurization). 

L’objectif principal des modèles de classification est de prédire les catégories dans lesquelles les nouvelles données seront classées, sur la base des apprentissages de leurs données d’apprentissage. Les exemples de classification courants incluent la détection des fraudes, la reconnaissance de l’écriture manuscrite et la détection d’objets. Pour plus d’informations et voir un exemple, consultez [Créer un modèle de classification avec ML automatisé](tutorial-first-experiment-automated-ml.md).

Pour obtenir des exemples de classification et de Machine Learning automatisé, consultez les notebooks Python suivants : [Détection des fraudes](https://github.com/Azure/azureml-examples/blob/main/python-sdk/tutorials/automl-with-azureml/classification-credit-card-fraud/auto-ml-classification-credit-card-fraud.ipynb), [Prédictions marketing](https://github.com/Azure/azureml-examples/blob/main/python-sdk/tutorials/automl-with-azureml/classification-bank-marketing-all-features/auto-ml-classification-bank-marketing-all-features.ipynb) et [Classification des données de groupe de discussion](https://github.com/Azure/azureml-examples/tree/main/python-sdk/tutorials/automl-with-azureml/classification-text-dnn)

### <a name="regression"></a>régression ;

À l’instar de la classification, les tâches de régression sont également une tâche d’apprentissage supervisé courante. Azure Machine Learning offre des [caractérisations spécifiquement pour ces tâches](how-to-configure-auto-features.md#featurization).

À la différence d’une classification dans laquelle les valeurs de sortie prédites sont catégoriques, les modèles de régression prédisent des valeurs de sortie numériques en fonction de prédictions indépendantes. Dans une régression, l’objectif est d’aider à établir la relation entre ces variables de prédiction indépendantes en estimant l’impact d’une variable sur les autres. Par exemple, le coût de l’automobile basé sur des caractéristiques telles que la consommation de carburant, la cote de sécurité, etc. Apprenez-en davantage et découvrez un exemple de [Régression avec Machine Learning automatisé](tutorial-auto-train-models.md).

Pour obtenir des exemples de régression et de Machine Learning automatisé pour les prédictions, consultez les notebooks Python suivants : [Prédiction des performances du processeur](https://github.com/Azure/azureml-examples/tree/main/python-sdk/tutorials/automl-with-azureml/regression-explanation-featurization) 

### <a name="time-series-forecasting"></a>Prévision de série chronologique

L’établissement de prévisions fait partie intégrante de toute entreprise, qu’il s’agisse du chiffre d’affaires, de l’inventaire, des ventes ou de la demande des clients. Vous pouvez utiliser le Machine Learning automatisé pour combiner des techniques et approches, et obtenir une prévision de série chronologique recommandée de haute qualité. Pour en savoir plus, consultez [Machine Learning automatisé pour la prévision de séries chronologiques](how-to-auto-train-forecast.md). 

Une expérience de série chronologique automatisée est traitée comme un problème de régression multivariable. Les valeurs de série chronologique passées « pivotent » afin de devenir des dimensions supplémentaires pour le régresseur, avec d’autres prédicteurs. Contrairement aux méthodes de séries chronologiques classique, cette méthode présente l’avantage d’incorporer naturellement plusieurs variables contextuelles et leurs relations entre elles pendant l’apprentissage. Le Machine Learning automatisé effectue l’apprentissage d’un modèle unique, mais souvent ramifié en interne, pour tous les éléments du jeu de données et les horizons de prédiction. Plus de données sont ainsi disponibles pour estimer les paramètres du modèle et la généralisation en séries invisibles devient possible.

La configuration de prévisions avancée inclut les éléments suivants :
* détection et personnalisation de congé
* série chronologique et apprenants DNN (Auto-ARIMA, Prophet, ForecastTCN)
* prise en charge de plusieurs modèles via le regroupement
* validation croisée d’origine
* décalages configurables
* caractéristiques des agrégations des fenêtres dynamiques


Pour obtenir des exemples de régression et de Machine Learning automatisé pour les prédictions, consultez les notebooks Python suivants : [Prévision des ventes](https://github.com/Azure/azureml-examples/blob/main/python-sdk/tutorials/automl-with-azureml/forecasting-orange-juice-sales/auto-ml-forecasting-orange-juice-sales.ipynb), [Prévision de la demande](https://github.com/Azure/azureml-examples/blob/main/python-sdk/tutorials/automl-with-azureml/forecasting-energy-demand/auto-ml-forecasting-energy-demand.ipynb) et [Prévision de la production de boissons](https://github.com/Azure/azureml-examples/blob/main/python-sdk/tutorials/automl-with-azureml/forecasting-beer-remote/auto-ml-forecasting-beer-remote.ipynb).

### <a name="computer-vision-preview"></a>Vision par ordinateur (préversion)

> [!IMPORTANT]
> Cette fonctionnalité est actuellement disponible en préversion publique. Cette préversion est fournie sans contrat de niveau de service. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Le Machine Learning automatisé pour les images (préversion) ajoute la prise en charge des tâches de vision par ordinateur, ce qui vous permet de générer facilement des modèles ayant fait l’objet d’un apprentissage sur des données d’image pour des scénarios comme la classification d’images et la détection d’objets. 

Avec cette fonctionnalité, vous pouvez : 
 
* Réaliser une intégration fluide à la fonctionnalité [d’étiquetage des données Azure Machine Learning](./how-to-create-image-labeling-projects.md)
* Utiliser des données étiquetées pour générer des modèles d’image
* Optimiser les performances du modèle en spécifiant l’algorithme du modèle et en réglant les hyperparamètres 
* Télécharger ou déployer le modèle obtenu en tant que service web dans Azure Machine Learning 
* Rendre les modèles opérationnels à grande échelle, en tirant parti des fonctionnalités d’Azure Machine Learning [MLOps](concept-model-management-and-deployment.md) et [Pipelines ML](concept-ml-pipelines.md). 

La création de modèles AutoML pour les tâches de vision est prise en charge via le kit SDK Azure ML Python. Vous pouvez accéder aux exécutions, aux modèles et aux sorties de l’expérimentation obtenus à partir de l’interface utilisateur Azure Machine Learning Studio.

Découvrez comment [configurer l’apprentissage AutoML pour les modèles de vision par ordinateur](how-to-auto-train-image-models.md).

![Exemples de tâches de vision par ordinateur. Image de : http://cs231n.stanford.edu/slides/2021/lecture_15.pdf ](./media/concept-automated-ml/automl-computer-vision-tasks.png) Image de : http://cs231n.stanford.edu/slides/2021/lecture_15.pdf

La fonctionnalité de ML automatisé pour les images prend en charge les tâches de vision par ordinateur suivantes : 

Tâche | Description
----|----
Classification d’images multiclasse | Tâches où une image est classifiée avec une seule étiquette d’un ensemble de classes. Chaque image est classifiée par exemple en tant qu’image de « chat », de « chien » ou de « canard »
Classification d’images multiétiquette | Tâches où une image peut avoir une ou plusieurs étiquettes d’un ensemble d’étiquettes. Une image peut être étiquetée par exemple « chat » et « chien »
Détection d’objets| Tâches permettant d’identifier des objets dans une image et de localiser chaque objet avec un rectangle englobant. On peut par exemple rechercher tous les chiens et tous les chats dans une image et les entourer d’un cadre englobant.
Segmentation d’instance | Tâches permettant d’identifier des objets dans une image au niveau des pixels, en dessinant un polygone autour de chaque objet dans l’image.

## <a name="how-automated-ml-works"></a>Fonctionnement du Machine Learning automatisé

Pendant l’entraînement, Azure Machine Learning crée un certain nombre de pipelines en parallèle qui testent différents algorithmes et paramètres pour vous. Le service effectue des itérations dans les algorithmes de Machine Learning associés aux sélections de fonctionnalités, où chaque itération produit un modèle avec un score d’apprentissage. Plus le score est élevé, plus le modèle est considéré comme « adapté » à vos données.  Il s’arrête une fois qu’il réunit les critères de sortie définis dans l’expérience. 

**Azure Machine Learning** vous permet de concevoir et d’exécuter vos expériences d’entraînement de Machine Learning automatisé en effectuant les étapes suivantes :

1. **Identifier le problème de Machine Learning** à résoudre : classification, prévision, régression ou vision par ordinateur (préversion).

1. **Indiquez si vous souhaitez utiliser le SDK Python ou l’expérience web Studio** : apprenez-en davantage sur la parité entre le [SDK Python et l’expérience web Studio](#parity).

   * Si vous disposez d’une expérience limitée ou inexistante en programmation, essayez l’expérience web Azure Machine Learning Studio sur [https://ml.azure.com](https://ml.azure.com/).  
   * Pour les développeurs Python, consultez le [SDK Python Azure Machine Learning](how-to-configure-auto-train.md). 
    
1. **Spécifier la source et le format des données d’apprentissage étiquetées** : tableaux NumPy ou cadre de données Pandas.

1. **Configurer la cible de calcul pour l’apprentissage du modèle** : [ordinateur local, calculs Azure Machine Learning, machines virtuelles à distance ou Azure Databricks](how-to-set-up-training-targets.md).

1. **Configurer les paramètres de Machine Learning automatisé** qui déterminent le nombre d’itérations sur les différents modèles, les réglages d’hyperparamètre, le prétraitement et la personnalisation avancés, ainsi que les métriques à examiner lors du choix du meilleur modèle.  
1. **Lancer l’exécution de l’apprentissage.**

1. **Passer en revue les résultats** 

Le diagramme suivant illustre ce processus. 
![Machine learning automatisé](./media/concept-automated-ml/automl-concept-diagram2.png)


Vous pouvez également inspecter les informations d’exécution journalisées qui [contiennent les métriques collectées](how-to-understand-automated-ml.md) pendant l’exécution. L’exécution de l’apprentissage produit un objet sérialisé Python (fichier `.pkl`) contenant le modèle et le prétraitement des données.

Bien que la création de modèles soit automatisée, vous pouvez également [découvrir l’importance ou la pertinence des fonctionnalités](how-to-configure-auto-train.md#explain) pour les modèles générés.

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE2Xc9t]

<a name="local-remote"></a>

## <a name="guidance-on-local-vs-remote-managed-ml-compute-targets"></a>Conseils concernant les cibles de calcul de Machine Learning managées locales et distantes

L’interface web pour le Machine Learning automatisé utilise toujours une [cible de calcul](concept-compute-target.md) distante.  Toutefois, lorsque vous utilisez le Kit de développement logiciel (SDK) Python, vous devez choisir une cible de calcul locale ou distante pour un apprentissage de Machine Learning automatisé.

* **Calcul local** : l’apprentissage se produit sur votre ordinateur portable ou votre machine virtuelle locaux. 
* **Calcul distant** : l’apprentissage se produit sur des clusters de calcul Machine Learning.  

### <a name="choose-compute-target"></a>Choisir une cible de calcul
Lors du choix de votre cible de calcul, considérez les facteurs suivants :

 * **Choisir un calcul local** : si votre scénario concerne des explorations initiales ou des démonstrations utilisant des données de petite taille et des apprentissages courts (de quelques secondes à quelques minutes par exécution enfant), un apprentissage sur votre ordinateur local peut constituer un meilleur choix.  Il n’y a pas de temps de configuration. Les ressources d’infrastructure (votre PC ou votre machine virtuelle) sont directement disponibles.
 * **Choisir un cluster de calcul de Machine Learning distant** : si vous effectuez un apprentissage avec des jeux de données plus volumineux, tel un apprentissage de production créant des modèles nécessitant des apprentissages plus longs, un calcul distant offre des performances de temps de bout en bout bien meilleures, car `AutoML` parallélise les apprentissages dans les nœuds du cluster. Sur un calcul distant, le temps de démarrage de l’infrastructure interne ajoute environ 1,5 minute par exécution enfant, et des minutes supplémentaires pour l’infrastructure du cluster si les machines virtuelles ne sont pas encore opérationnelles.

### <a name="pros-and-cons"></a>Avantages et inconvénients
Tenez compte des avantages et des inconvénients suivants lorsque au moment de choisir entre le calcul local ou distant.

|  | Avantages (avantages)  |Inconvénients (handicaps)  |
|---------|---------|---------|---------|
|**Cible de calcul locale** |  <li> Aucune heure de démarrage de l’environnement   | <li>  Sous-ensemble de fonctionnalités<li>  Impossible de paralléliser les exécutions <li> Pire pour des données volumineuses <li>Aucune diffusion de données pendant l’apprentissage <li>  Aucun caractérisation basée sur DNN <li> Kit de développement logiciel (SDK) Python uniquement |
|**Clusters de calcul ML distants**|  <li> Ensemble complet de fonctionnalités <li> Parallélisation des exécutions enfants <li>   Prise en charge de données volumineuses<li>  Caractérisation basée sur DNN <li>  Extensibilité dynamique du cluster de calcul à la demande <li> Expérience sans code (interface utilisateur web) également disponible  |  <li> Temps de démarrage pour les nœuds de cluster <li> Temps de démarrage pour chaque exécution enfant    |

### <a name="feature-availability"></a>Disponibilité des fonctionnalités 

D’autres fonctionnalités sont disponibles lorsque vous utilisez le calcul distant, comme indiqué dans le tableau ci-dessous. 

| Fonctionnalité                                                    | Remote | Local | 
|------------------------------------------------------------|--------|-------|
| Streaming de données (prise en charge de données volumineuses jusqu’à 100 Go)          | ✓      |       | 
| Caractérisation de texte et apprentissage basés sur DNN-BERT             | ✓      |       |
| Prise en charge de GPU prêt à l’emploi (apprentissage et inférence)        | ✓      |       |
| Prise en charge de la classification d’image et de l’étiquetage                  | ✓      |       |
| Modèles auto-ARIMA, Prophet et ForecastTCN pour les prévisions | ✓      |       | 
| Exécutions/itérations multiples en parallèle                       | ✓      |       |
| Créer des modèles avec interprétabilité dans l’interface utilisateur de l’expérience web du studio AutoML      | ✓      |       |
| Personnalisation de l’ingénierie des fonctionnalités dans l’interface utilisateur de l’expérience web du studio| ✓      |       |
| Optimisation des hyperparamètres d’Azure Machine Learning                             | ✓      |       |
| Prise en charge des flux de travail du pipeline Azure Machine Learning                         | ✓      |       |
| Continuer une exécution                                             | ✓      |       |
| Prévisions                                                | ✓      | ✓     |
| Créer et exécuter des expériences dans des blocs-notes                    | ✓      | ✓     |
| Inscrire et visualiser les informations et les métriques de l’expérience dans l’interface utilisateur | ✓      | ✓     |
| Garde-fous des données                                            | ✓      | ✓     |

## <a name="training-validation-and-test-data"></a>Formation, validation et test des données 

Avec le ML automatisé, vous fournissez les **données d’apprentissage** pour former des modèles de ML, et vous pouvez spécifier le type de validation de modèle à effectuer. Le ML automatisé effectue une validation de modèle dans le cadre de la formation. Autrement dit, le ML automatisé utilise des **données de validation** pour régler les hyperparamètres de modèle en fonction de l’algorithme appliqué afin de trouver la combinaison la plus appropriée qui correspond le mieux aux données d’apprentissage. Toutefois, les mêmes données de validation sont utilisées pour chaque itération de paramétrage, ce qui introduit le biais de l’évaluation du modèle, puisque le modèle continue à s’améliorer et à s’ajuster par rapport aux données de validation. 

Pour vous aider à confirmer que ce biais n’est pas appliqué au modèle recommandé final, le ML automatisé prend en charge l’utilisation de **données de test** pour évaluer le modèle final que le ML automatisé recommande à la fin de votre expérience. Lorsque vous fournissez des données de test dans le cadre de votre configuration d’expérimentation AutoML, ce modèle recommandé est testé par défaut à la fin de votre expérience (préversion). 

>[!IMPORTANT]
> Le test de vos modèles avec un jeu de données de test pour évaluer les modèles générés est une fonctionnalité en préversion. Cette capacité est une caractéristique [expérimentale](/python/api/overview/azure/ml/#stable-vs-experimental) en préversion qui peut évoluer à tout moment.

Découvrez comment [Configurer les expériences AutoML pour utiliser des données de test (préversion) avec le kit de développement logiciel (SDK)](how-to-configure-cross-validation-data-splits.md#provide-test-data-preview) ou avec [Azure Machine Learning Studio](how-to-use-automated-ml-for-ml-models.md#create-and-run-experiment).

Vous pouvez également [tester n’importe quel modèle de ML automatisé (préversion)](how-to-configure-auto-train.md#test-existing-automated-ml-model) existant, y compris les modèles des exécutions enfants, en fournissant vos propres données de test ou en mettant de côté une partie de vos données d’apprentissage. 

## <a name="feature-engineering"></a>Ingénierie des caractéristiques

L’ingénierie des caractéristiques est le processus qui consiste à utiliser la connaissance du domaine des données pour créer des fonctionnalités qui aident les algorithmes de ML à améliorer leur apprentissage. Dans Azure Machine Learning, des techniques de mise à l’échelle et de normalisation sont appliquées pour faciliter l’ingénierie de caractéristiques. Collectivement, ces techniques et l’ingénierie de caractéristiques sont appelées caractérisation.

Pour les expériences de Machine Learning automatisé, la caractérisation s’applique automatiquement, mais peut également être personnalisée en fonction de vos données. [En savoir plus sur la personnalisation incluse](how-to-configure-auto-features.md#featurization).  

> [!NOTE]
> Les étapes de caractérisation du Machine Learning automatisé (normalisation des fonctionnalités, gestion des données manquantes, conversion de texte en valeurs numériques, etc.) font partie du modèle sous-jacent. Lorsque vous utilisez le modèle pour des prédictions, les étapes de caractérisation qui sont appliquées pendant la formation sont appliquées automatiquement à vos données d’entrée.

### <a name="automatic-featurization-standard"></a>Caractérisation automatique (standard)

Dans chaque expérience de Machine Learning automatisé, vos données sont automatiquement mises à l’échelle et normalisées pour faciliter la bonne exécution des algorithmes. Pendant l’apprentissage du modèle, l’une des techniques suivantes de mise à l’échelle ou de normalisation est appliquée à chaque modèle. Découvrez comment AutoML permet [d’empêcher le surajustement et le déséquilibre des données](concept-manage-ml-pitfalls.md) dans vos modèles.

|Traitement&nbsp;&&nbsp;de la mise à l’échelle| Description |
| ------------- | ------------- |
| [StandardScaleWrapper](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.StandardScaler.html)  | Normaliser les fonctionnalités en supprimant la moyenne et en mettant à l’échelle de l’écart d’unité.  |
| [MinMaxScalar](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.MinMaxScaler.html)  | Transformer les fonctionnalités en les mettant à l’échelle des valeurs minimale et maximale de cette colonne.  |
| [MaxAbsScaler](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.MaxAbsScaler.html#sklearn.preprocessing.MaxAbsScaler) |Mettre à l’échelle chaque fonctionnalité en fonction de sa valeur absolue maximale. |
| [RobustScalar](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.RobustScaler.html) | Mettre à l’échelle les fonctionnalités par plage de quantiles |
| [PCA](https://scikit-learn.org/stable/modules/generated/sklearn.decomposition.PCA.html) |Réduction de dimensionnalité linéaire à l’aide d’une décomposition de valeur singulière des données afin de les projeter vers un espace dimensionnel inférieur. |
| [TruncatedSVDWrapper](https://scikit-learn.org/stable/modules/generated/sklearn.decomposition.TruncatedSVD.html) |Ce transformateur effectue une réduction de dimensionnalité linéaire au moyen d’une décomposition de valeur singulière tronquée. Contrairement à PCA, cet estimateur ne centre pas les données avant de calculer la décomposition de valeur singulière, ce qui signifie qu’il peut travailler efficacement avec des matrices scipy.sparse. |
| [SparseNormalizer](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.Normalizer.html) | Chaque exemple (autrement dit, chaque ligne de la matrice de données) avec au moins un composant différent de zéro est remis à l’échelle indépendamment des autres exemples afin que sa norme (l1 ou l2) soit égale à un. |

### <a name="customize-featurization"></a>Personnaliser la caractérisation

Des techniques de caractérisation supplémentaires, telles que l’encodage et les transformations, sont également disponibles. 

Activez ce paramètre avec :

+ Azure Machine Learning Studio : Activez **Caractérisation automatique** dans la section **Afficher des configurations supplémentaires** [en suivant ces étapes](how-to-use-automated-ml-for-ml-models.md#customize-featurization).

+ Kit de développement logiciel (SDK) Python : Spécifiez `"feauturization": 'auto' / 'off' / 'FeaturizationConfig'` dans votre objet [AutoMLConfig](/python/api/azureml-train-automl-client/azureml.train.automl.automlconfig.automlconfig). En savoir plus sur [l’activation de la caractérisation](how-to-configure-auto-features.md). 

## <a name="ensemble-models"></a><a name="ensemble"></a> Modèles ensemblistes

Le Machine Learning automatisé prend en charge les modèles ensemblistes, qui sont activés par défaut. L’apprentissage ensembliste améliore les résultats de Machine Learning et les performances prédictives en combinant plusieurs modèles. Les itérations d’ensembles apparaissent comme les dernières itérations de votre exécution. Le Machine Learning automatisé utilise des méthodes ensemblistes de vote et d’empilement pour combiner les modèles :

* **Vote** : prédictions basées sur la moyenne pondérée des probabilités de classe prédites (pour les tâches de classification) ou des cibles de régression prédites (pour les tâches de régression).
* **Empilement** : l’empilement combine des modèles hétérogènes et entraîne un métamodèle basé sur la sortie de différents modèles. Actuellement, les métamodèles par défaut sont LogisticRegression pour les tâches de classification, et ElasticNet pour les tâches de régression et de prévision.

L’[algorithme de sélection d’ensemble Caruana](http://www.niculescu-mizil.org/papers/shotgun.icml04.revised.rev2.pdf), avec initialisation des ensembles triés, est utilisé pour déterminer les modèles qui doivent être utilisés au sein de l’ensemble. Pour résumer, cet algorithme initialise l’ensemble avec un maximum de cinq modèles ayant obtenu les meilleurs scores, puis vérifie que ces scores se situent dans une marge de plus ou moins 5 % par rapport au meilleur score, afin d’éviter un ensemble de niveau médiocre. Ensuite, pour chaque itération d’ensemble, un nouveau modèle est ajouté à l’ensemble existant et le score est calculé. Si un nouveau modèle a amélioré le score existant de l’ensemble, l’ensemble est mis à jour pour inclure le nouveau modèle.

Pour savoir comment modifier les paramètres par défaut de l’ensemble au niveau du Machine Learning automatisé, consultez [cette procédure](how-to-configure-auto-train.md#ensemble).

<a name="use-with-onnx"></a>

## <a name="automl--onnx"></a>AutoML et ONNX

Avec Azure Machine Learning, vous pouvez utiliser le Machine Learning automatisé pour générer un modèle Python et le convertir au format ONNX. Une fois que les modèles sont au format ONNX, ils peuvent s’exécuter sur une multitude de plateformes et d’appareils. Apprenez-en davantage sur l’[accélération des modèles ML avec ONNX](concept-onnx.md).

Découvrez comment convertir au format ONNX [dans cet exemple de notebook Jupyter](https://github.com/Azure/azureml-examples/tree/main/python-sdk/tutorials/automl-with-azureml/classification-bank-marketing-all-features). Découvrez quels sont les [algorithmes pris en charge dans ONNX](how-to-configure-auto-train.md#supported-models).

Le runtime ONNX prenant également en charge C#, vous pouvez utiliser le modèle généré automatiquement dans vos applications C# sans avoir besoin de recodage ou des latences réseau introduites par les points de terminaison REST. En savoir plus sur [l’utilisation d’un modèle ONNX AutoML dans une application .NET avec ML.NET](./how-to-use-automl-onnx-model-dotnet.md) et sur [l’inférence de modèles ONNX avec l’API C# du runtime ONNX](https://onnxruntime.ai/docs/api/csharp-api.html). 

## <a name="next-steps"></a>Étapes suivantes

Plusieurs ressources sont disponibles pour vous aider à utiliser AutoML. 

### <a name="tutorials-how-tos"></a>Tutoriels et guides pratiques
Les tutoriels sont des exemples illustrant de bout en bout des scénarios d’utilisation d’AutoML.
+ **Pour une première expérience de code**, suivez le [Tutoriel : effectuer l’apprentissage d’un modèle de régression avec AutoML et Python](tutorial-auto-train-models.md).

+ **Pour une expérience avec peu de code ou sans code**, consultez le [Tutoriel : Entraînement d’un modèle de classification avec AutoML sans code dans Azure Machine Learning studio](tutorial-first-experiment-automated-ml.md).

+ **Pour utiliser AutoML pour l’apprentissage des modèles de vision par ordinateur**, consultez [Tutoriel : apprentissage d’un modèle de détection d’objets (préversion) avec AutoML et Python](tutorial-auto-train-image-models.md).
   
Des articles de guide pratique fournissent des détails supplémentaires sur les fonctionnalités du Machine Learning automatisé. Par exemple, 

+ Configurez les paramètres pour des expériences d’entraînement automatique
    + [Sans code dans Azure Machine Learning studio](how-to-use-automated-ml-for-ml-models.md). 
    + [Avec le SDK Python](how-to-configure-auto-train.md).

+  Découvrez comment [entraîner des modèles de prévision avec des données de série chronologique](how-to-auto-train-forecast.md).

+  Découvrez comment [effectuer l’apprentissage des modèles de vision par ordinateur avec Python](how-to-auto-train-image-models.md).
   
### <a name="jupyter-notebook-samples"></a>Exemples de blocs-notes Jupyter 

Passez en revue les exemples de code détaillé et les cas d’usage disponibles dans le [dépôt GitHub d’exemples de blocs-notes pour le Machine Learning automatisé](https://github.com/Azure/azureml-examples/tree/main/python-sdk/tutorials/automl-with-azureml).

### <a name="python-sdk-reference"></a>Référence du Kit de développement logiciel (SDK) Python

Approfondissez votre expertise des modèles de conception de Kit de développement logiciel (SDK) et des spécifications de classe avec la [documentation de référence sur la classe AutoML](/python/api/azureml-train-automl-client/azureml.train.automl.automlconfig.automlconfig). 

> [!Note]
> Des fonctionnalités de machine learning automatisé sont également disponible dans d’autres solutions Microsoft telles que [ML.NET](/dotnet/machine-learning/automl-overview), [HDInsight](../hdinsight/spark/apache-spark-run-machine-learning-automl.md), [Power BI](/power-bi/service-machine-learning-automated) et [SQL Server](https://cloudblogs.microsoft.com/sqlserver/2019/01/09/how-to-automate-machine-learning-on-sql-server-2019-big-data-clusters/)