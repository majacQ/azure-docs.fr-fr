---
title: 'Langage de requête Azure Cosmos DB : UPPER'
description: Découvrez la fonction système SQL UPPER dans Azure Cosmos DB.
author: ginamr
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 04/08/2021
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: 1a1d3c19bd76805fb26d365d4093a796cc134a41
ms.sourcegitcommit: 2d412ea97cad0a2f66c434794429ea80da9d65aa
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/14/2021
ms.locfileid: "122525756"
---
# <a name="upper-azure-cosmos-db"></a>UPPER (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](../includes/appliesto-sql-api.md)]

 Retourne une expression de chaîne après la conversion des caractères minuscules en caractères majuscules.  

La fonction système UPPER n’utilise pas l’index. Si vous envisagez d’effectuer des comparaisons non sensibles à la casse fréquentes, la fonction système UPPER peut consommer une quantité significative de RU. Si c’est le cas, au lieu d’utiliser la fonction système UPPER pour normaliser les données à chaque fois pour les comparaisons, vous pouvez normaliser la casse lors de l’insertion. Ensuite, une requête telle que SELECT * FROM c WHERE UPPER(c.name) = 'BOB' devient simplement SELECT * FROM c WHERE c.name = 'BOB'.

## <a name="syntax"></a>Syntaxe
  
```sql
UPPER(<str_expr>)  
```  
  
## <a name="arguments"></a>Arguments
  
*str_expr*  
   Est une expression de chaîne.  
  
## <a name="return-types"></a>Types de retour
  
  Retourne une expression de chaîne.  
  
## <a name="examples"></a>Exemples
  
  L’exemple suivant montre comment utiliser `UPPER` dans une requête.  
  
```sql
SELECT UPPER("Abc") AS upper  
```  
  
 Voici le jeu de résultats obtenu.  
  
```json
[{"upper": "ABC"}]  
```

## <a name="remarks"></a>Notes

Cette fonction système [n’utilise pas d’index](../index-overview.md#index-usage).

## <a name="next-steps"></a>Étapes suivantes

- [Fonctions de chaîne Azure Cosmos DB](sql-query-string-functions.md)
- [Fonctions système Azure Cosmos DB](sql-query-system-functions.md)
- [Présentation d’Azure Cosmos DB](../introduction.md)
