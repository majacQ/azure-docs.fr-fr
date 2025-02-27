---
title: Outils de science des données et d’apprentissage automatique
titleSuffix: Azure Data Science Virtual Machine
description: Découvrez les outils et les frameworks de Machine Learning préinstallés sur Data Science Virtual Machine.
keywords: outils de science des données, machine virtuelle science des données, outils pour la science des données, science des données linux
services: machine-learning
ms.service: data-science-vm
author: timoklimmer
ms.author: tklimmer
ms.topic: conceptual
ms.date: 05/12/2021
ms.openlocfilehash: 86cbac686c2f994dff4042ea2a227156d9e45472
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122524873"
---
# <a name="machine-learning-and-data-science-tools-on-azure-data-science-virtual-machines"></a>Outils d’apprentissage automatique et de science des données sur les machines virtuelles de science des données (DSVM) Azure
Les machines virtuelles DSVM (Data Science Virtual Machine) offrent un ensemble complet d’outils et de bibliothèques de Machine Learning disponibles dans des langages courants tels que Python, R et Julia.

Voici une partie des outils et des bibliothèques de Machine Learning disponibles sur les machines virtuelles DSVM.

## <a name="azure-machine-learning-sdk-for-python"></a>Kit SDK Azure Machine Learning pour Python

Consultez la référence complète du [Kit de développement logiciel (SDK) Azure Machine Learning pour Python.](../overview-what-is-azure-machine-learning.md)

| Category | Valeur |
| ------------- | ------------- |
| Qu’est-ce que c’est ?   |   Azure Machine Learning est un service cloud que vous pouvez utiliser pour développer et déployer des modèles Machine Learning. Vous pouvez suivre vos modèles avec le SDK Python pendant les opérations de création, d’entraînement, de scaling et de gestion que vous effectuez sur ceux-ci. Déployez des modèles en tant que conteneurs et exécutez-les dans le cloud, localement, ou sur Azure IoT Edge.   |
| Éditions prises en charge     | Windows (environnement Conda : AzureML), Linux (environnement Conda : py36)    |
| Utilisations classiques      | Plateforme de Machine Learning générale      |
| Comment fonctionne la configuration ou l’installation ?      |  Installé avec prise en charge GPU   |
| Comment l’utiliser ou l’exécuter      | Comme kit SDK Python et dans Azure CLI. Activez pour l’environnement conda `AzureML` sur l’édition Windows *ou* pour `py36` sur l’édition Linux.      |
| Liens vers des exemples      | Des exemples de blocs-notes Jupyter sont inclus dans le répertoire `AzureML` sous blocs-notes.  |

## <a name="h2o"></a>H2O

| Category | Valeur |
| ------------- | ------------- |
| Qu’est-ce que c’est ?   | Une plateforme d’intelligence artificielle open source prenant en charge des fonctionnalités de Machine Learning en mémoire, distribuées, rapides et scalables.  |
| Versions prises en charge      | Linux   |
| Utilisations classiques      | Machine Learning scalable, distribué et à usage général   |
| Comment fonctionne la configuration ou l’installation ?      | H2O est installé dans `/dsvm/tools/h2o`.      |
| Comment l’utiliser ou l’exécuter      | Connectez-vous à la machine virtuelle à l’aide de X2Go. Démarrez un nouveau terminal et exécutez `java -jar /dsvm/tools/h2o/current/h2o.jar`. Ensuite, démarrez un navigateur web et connectez-vous à `http://localhost:54321`.      |
| Liens vers des exemples      | Des exemples sont disponibles sur la machine virtuelle dans Jupyter sous le répertoire `h2o`.      |

Il existe plusieurs autres bibliothèques de Machine Learning sur des machines virtuelles DSVM, notamment le package `scikit-learn` très prisé qui fait partie de la distribution Anaconda Python pour les machines virtuelles DSVM. Pour consulter la liste des packages disponibles dans Julia, Python et R, exécutez les gestionnaires de package correspondants.

## <a name="lightgbm"></a>LightGBM

| Category | Valeur |
| ------------- | ------------- |
| Qu’est-ce que c’est ?   | Un framework de boosting de gradient (GBDT, GBRT, GBM ou MART) rapide, distribué et hautes performances basé sur des algorithmes d’arbre de décision. Il est utilisé pour le classement, la classification et de nombreuses autres tâches de Machine Learning.    |
| Versions prises en charge      | Windows, Linux    |
| Utilisations classiques      | Framework de boosting de gradient à usage général      |
| Comment fonctionne la configuration ou l’installation ?      | Sur Windows, LightGBM est installé en tant que package Python. Sur Linux, le fichier exécutable de ligne de commande se trouve dans `/opt/LightGBM/lightgbm`, le package R est installé et des packages Python sont installés.     |
| Liens vers des exemples      | [Guide LightGBM](https://github.com/Microsoft/LightGBM/tree/master/examples/python-guide)   |

## <a name="rattle"></a>Rattle
| Category | Valeur |
| ------------- | ------------- |
| Qu’est-ce que c’est ?   |   Une interface graphique utilisateur pour l’exploration de données à l’aide de R.   |
| Éditions prises en charge     | Windows, Linux     |
| Utilisations classiques      | Outil général d’exploration de données doté d’une interface utilisateur pour R    |
| Comment l’utiliser ou l’exécuter      | En tant qu’outil d’interface utilisateur. Sur Windows, démarrez une invite de commandes, exécutez R, puis exécutez `rattle()` dans R. Sur Linux, connectez-vous avec X2Go, démarrez un terminal, exécutez R, puis dans R, exécutez `rattle()`. |
| Liens vers des exemples      | [Rattle](https://togaware.com/onepager/) |

## <a name="vowpal-wabbit"></a>Vowpal Wabbit
| Category | Valeur |
| ------------- | ------------- |
| Qu’est-ce que c’est ?   |   Une bibliothèque système d’entraînement rapide, open source et hors cœur    |
| Éditions prises en charge     | Windows, Linux     |
| Utilisations classiques      | Bibliothèque de Machine Learning générale      |
| Comment fonctionne la configuration ou l’installation ?      |  Windows : programme d’installation MSI<br/>Linux : apt-get |
| Comment l’utiliser ou l’exécuter      | En tant qu’outil en ligne de commande de chemin (`C:\Program Files\VowpalWabbit\vw.exe` sur Windows, `/usr/bin/vw` sur Linux)    |
| Liens vers des exemples      | [Exemples VowPal Wabbit](https://github.com/JohnLangford/vowpal_wabbit/wiki/Examples) |

## <a name="weka"></a>Weka
| Category | Valeur |
| ------------- | ------------- |
| Qu’est-ce que c’est ?   |  Une collection d’algorithmes de Machine Learning pour les tâches d’exploration de données. Les algorithmes peuvent être appliqués directement à un jeu de données ou être appelés à partir de votre propre code Java. Weka contient des outils pour le prétraitement des données, la classification, la régression, le clustering, les règles d’association et la visualisation. |
| Éditions prises en charge     | Windows, Linux     |
| Utilisations classiques      | Outil de Machine Learning général     |
| Comment l’utiliser ou l’exécuter      | Sur Windows, recherchez Weka dans le menu **Démarrer**. Sur Linux, connectez-vous avec X2Go, puis accédez à **Applications** > **Développement** > **Weka**. |
| Liens vers des exemples      | [Exemples Weka](https://www.cs.waikato.ac.nz/ml/weka/documentation.html) |

## <a name="xgboost"></a>XGBoost 
| Category | Valeur |
| ------------- | ------------- |
| Qu’est-ce que c’est ?   |   Une bibliothèque de boosting de gradient (GBDT, GBRT ou GBM) rapide, portable et distribuée pour Python, R, Java, Scala, C++, etc. S’exécute sur une seule machine, et sur Apache Hadoop et Spark.    |
| Éditions prises en charge     | Windows, Linux     |
| Utilisations classiques      | Bibliothèque de Machine Learning générale      |
| Comment fonctionne la configuration ou l’installation ?      |  Installé avec prise en charge GPU   |
| Comment l’utiliser ou l’exécuter      | Comme bibliothèque Python (2.7 et 3.6+), package R et outil en ligne de commande sur le chemin (`C:\dsvm\tools\xgboost\bin\xgboost.exe` pour Windows et `/dsvm/tools/xgboost/xgboost` pour Linux)    |
| Liens vers des exemples      | Des exemples sont fournis sur la machine virtuelle, dans `/dsvm/tools/xgboost/demo` sur Linux et `C:\dsvm\tools\xgboost\demo` sur Windows.   |
