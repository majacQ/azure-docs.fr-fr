---
title: Activer l’authentification AD DS pour des partages de fichiers Azure
description: Découvrez comment activer l’authentification Active Directory Domain Services sur SMB pour les partages de fichiers Azure. Vos machines virtuelles Windows jointes à un domaine peuvent alors accéder aux partages de fichiers Azure en utilisant des informations d’identification AD DS.
author: roygara
ms.service: storage
ms.subservice: files
ms.topic: how-to
ms.date: 10/05/2021
ms.author: rogarana
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 9012f74f29c1e3ed768a32a6988c7aae527b44ce
ms.sourcegitcommit: 611b35ce0f667913105ab82b23aab05a67e89fb7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2021
ms.locfileid: "129999147"
---
# <a name="part-one-enable-ad-ds-authentication-for-your-azure-file-shares"></a>Première partie : activer l’authentification AD DS pour vos partages de fichiers Azure 

Avant d’activer l’authentification AD DS (Active Directory Domain Services), lisez l’[article de présentation](storage-files-identity-auth-active-directory-enable.md) pour comprendre les exigences et les scénarios pris en charge.

Cet article décrit le processus à suivre pour activer l’authentification AD DS (Active Directory Domain Services) sur votre compte de stockage. Après avoir activé la fonctionnalité, vous devez configurer votre compte de stockage et votre AD DS afin d’utiliser les informations d’identification AD DS pour vous authentifier auprès de votre partage de fichiers Azure. Pour activer l’authentification AD DS sur SMB pour les partages de fichiers Azure, vous devez inscrire votre compte de stockage à AD DS, puis définir les propriétés de domaine requises sur le compte de stockage.

Pour inscrire votre compte de stockage auprès d’AD DS, créez un compte qui le représente dans votre AD DS. Vous pouvez vous représenter ce processus comme s’il s’agissait de créer un compte représentant un serveur de fichiers Windows local dans votre AD DS. Lorsque la fonctionnalité est activée sur le compte de stockage, elle s’applique à tous les partages de fichiers nouveaux et existants dans le compte.

## <a name="applies-to"></a>S’applique à
| Type de partage de fichiers | SMB | NFS |
|-|:-:|:-:|
| Partages de fichiers Standard (GPv2), LRS/ZRS | ![Oui](../media/icons/yes-icon.png) | ![Non](../media/icons/no-icon.png) |
| Partages de fichiers Standard (GPv2), GRS/GZRS | ![Oui](../media/icons/yes-icon.png) | ![Non](../media/icons/no-icon.png) |
| Partages de fichiers Premium (FileStorage), LRS/ZRS | ![Oui](../media/icons/yes-icon.png) | ![Non](../media/icons/no-icon.png) |

## <a name="option-one-recommended-use-azfileshybrid-powershell-module"></a>Option n°1 (recommandée) : Utiliser le module PowerShell AzFilesHybrid

Les applets de commande du module PowerShell AzFilesHybrid effectuent les modifications nécessaires et activent la fonctionnalité pour vous. Comme certaines parties des applets de commande interagissent avec votre AD DS local, nous expliquons ce qu’elles font pour vous permettre de déterminer si les modifications respectent vos stratégies de conformité et de sécurité, et de vérifier que vous disposez des autorisations appropriées pour les exécuter. Nous vous recommandons d’utiliser le module AzFilesHybrid, mais si ce n’est pas possible, nous vous indiquons les étapes à effectuer manuellement.

### <a name="download-azfileshybrid-module"></a>Télécharger le module AzFilesHybrid

- Si [.NET Framework 4.7.2](https://dotnet.microsoft.com/download/dotnet-framework/net472) n’est pas installé, installez-le maintenant. Il est requis pour que le module s’importe correctement.
- [Téléchargez et décompressez le module AzFilesHybrid (module GA : v0.2.0+)](https://github.com/Azure-Samples/azure-files-samples/releases) Notez que le chiffrement Kerberos 256 AES est pris en charge sur la v0.2.2 et les versions ultérieures. Si vous avez activé la fonctionnalité avec une version de AzFilesHybrid inférieure à la v0.2.2 et souhaitez la mettre à jour pour prendre en charge le chiffrement Kerberos 256 AES, consultez [cet article](./storage-troubleshoot-windows-file-connection-problems.md#azure-files-on-premises-ad-ds-authentication-support-for-aes-256-kerberos-encryption).
- Installez et exécutez le module dans un appareil dont le domaine est joint à AD DS en local avec des informations d’identification AD DS et qui dispose des autorisations nécessaires pour créer un compte d’ouverture de session du service ou un compte d’ordinateur dans l’instance AD cible.
-  Exécutez le script à l’aide des informations d’identification AD DS en local synchronisées à votre Azure AD. Les informations d’identification AD DS locales doivent disposer du rôle Azure **Propriétaire** ou **Contributeur** sur le compte de stockage.

### <a name="run-join-azstorageaccountforauth"></a>Exécuter Join-AzStorageAccountForAuth

L’applet de commande `Join-AzStorageAccountForAuth` effectue l’équivalent d’une jonction de domaine hors connexion pour le compte de stockage spécifié. Le script utilise l’applet de commande pour créer un [compte d’ordinateur](/windows/security/identity-protection/access-control/active-directory-accounts#manage-default-local-accounts-in-active-directory) dans votre domaine Active Directory. Si pour une raison quelconque, vous ne pouvez pas utiliser un compte d’ordinateur, vous pouvez à la place modifier le script pour créer un [compte d’ouverture de session de service](/windows/win32/ad/about-service-logon-accounts). Si vous choisissez d’exécuter la commande manuellement, vous devez sélectionner le compte le mieux adapté à votre environnement.

Le compte AD DS créé par l’applet de commande représente le compte de stockage. Si le compte AD DS est créé sous une unité d’organisation (UO) qui applique l’expiration du mot de passe, vous devez mettre à jour le mot de passe avant la durée de vie maximale du mot de passe. Le fait de ne pas mettre à jour le mot de passe du compte avant cette date entraîne des échecs d’authentification lors de l’accès aux partages de fichiers Azure. Pour savoir comment mettre à jour le mot de passe, consultez [Mettre à jour le mot de passe du compte AD DS](storage-files-identity-ad-ds-update-password.md).

Remplacer les valeurs des espaces réservés par les vôtres dans les paramètres ci-dessous avant d’exécuter le script dans PowerShell.
> [!IMPORTANT]
> La cmdlet de jonction de domaine crée un compte AD pour représenter le compte de stockage (partage de fichiers) dans AD. Vous pouvez choisir de vous inscrire en tant que compte d’ordinateur ou compte de connexion au service. Pour plus d’informations, consultez la [FAQ](./storage-files-faq.md#security-authentication-and-access-control). Pour les comptes d’ordinateur, la durée de vie du mot de passe par défaut est définie à 30 jours dans AD. De même, le compte de connexion au service peut avoir une durée de vie de mot de passe par défaut définie sur le domaine AD ou l’unité d’organisation (UO).
> Pour les deux types de comptes, nous vous recommandons de vérifier le délai d’expiration du mot de passe défini dans votre environnement AD et de [mettre à jour le mot de passe de l’identité de votre compte de stockage](storage-files-identity-ad-ds-update-password.md) pour le compte AD, avant la durée de vie maximale du mot de passe. Vous pouvez [créer une unité d’organisation (UO) AD dans AD](/powershell/module/activedirectory/new-adorganizationalunit) et désactiver en conséquence la stratégie d’expiration de mot de passe pour les [comptes d’ordinateur](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/jj852252(v=ws.11)) ou les comptes de connexion au service. 

```PowerShell
# Change the execution policy to unblock importing AzFilesHybrid.psm1 module
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser

# Navigate to where AzFilesHybrid is unzipped and stored and run to copy the files into your path
.\CopyToPSPath.ps1 

# Import AzFilesHybrid module
Import-Module -Name AzFilesHybrid

# Login with an Azure AD credential that has either storage account owner or contributer Azure role assignment
# If you are logging into an Azure environment other than Public (ex. AzureUSGovernment) you will need to specify that.
# See https://docs.microsoft.com/azure/azure-government/documentation-government-get-started-connect-with-ps
# for more information.
Connect-AzAccount

# Define parameters, $StorageAccountName currently has a maximum limit of 15 characters
$SubscriptionId = "<your-subscription-id-here>"
$ResourceGroupName = "<resource-group-name-here>"
$StorageAccountName = "<storage-account-name-here>"
$DomainAccountType = "<ComputerAccount|ServiceLogonAccount>" # Default is set as ComputerAccount
# If you don't provide the OU name as an input parameter, the AD identity that represents the storage account is created under the root directory.
$OuDistinguishedName = "<ou-distinguishedname-here>"
# Specify the encryption agorithm used for Kerberos authentication. Default is configured as "'RC4','AES256'" which supports both 'RC4' and 'AES256' encryption.
$EncryptionType = "<AES256|RC4|AES256,RC4>"

# Select the target subscription for the current session
Select-AzSubscription -SubscriptionId $SubscriptionId 

# Register the target storage account with your active directory environment under the target OU (for example: specify the OU with Name as "UserAccounts" or DistinguishedName as "OU=UserAccounts,DC=CONTOSO,DC=COM"). 
# You can use to this PowerShell cmdlet: Get-ADOrganizationalUnit to find the Name and DistinguishedName of your target OU. If you are using the OU Name, specify it with -OrganizationalUnitName as shown below. If you are using the OU DistinguishedName, you can set it with -OrganizationalUnitDistinguishedName. You can choose to provide one of the two names to specify the target OU.
# You can choose to create the identity that represents the storage account as either a Service Logon Account or Computer Account (default parameter value), depends on the AD permission you have and preference. 
# Run Get-Help Join-AzStorageAccountForAuth for more details on this cmdlet.

Join-AzStorageAccountForAuth `
        -ResourceGroupName $ResourceGroupName `
        -StorageAccountName $StorageAccountName `
        -DomainAccountType $DomainAccountType `
        -OrganizationalUnitDistinguishedName $OuDistinguishedName `
        -EncryptionType $EncryptionType

#Run the command below if you want to enable AES 256 authentication. If you plan to use RC4, you can skip this step.
Update-AzStorageAccountAuthForAES256 -ResourceGroupName $ResourceGroupName -StorageAccountName $StorageAccountName

#You can run the Debug-AzStorageAccountAuth cmdlet to conduct a set of basic checks on your AD configuration with the logged on AD user. This cmdlet is supported on AzFilesHybrid v0.1.2+ version. For more details on the checks performed in this cmdlet, see Azure Files Windows troubleshooting guide.
Debug-AzStorageAccountAuth -StorageAccountName $StorageAccountName -ResourceGroupName $ResourceGroupName -Verbose
```

## <a name="option-two-manually-perform-the-enablement-actions"></a>Option n°2 : Exécuter manuellement les actions d’activation

Si vous avez déjà exécuté le script `Join-AzStorageAccountForAuth` ci-dessus, passez à la section [Confirmer que la fonctionnalité est activée](#confirm-the-feature-is-enabled). Vous n’avez pas besoin d’effectuer les étapes manuelles suivantes.

### <a name="check-the-environment"></a>Vérifier l’environnement

Vous devez d’abord vérifier l’état de votre environnement. En particulier, vous devez vérifier si [Active Directory PowerShell](/powershell/module/activedirectory/) est installé et si l’interpréteur de commandes est en cours d’exécution avec des privilèges d’administrateur. Vérifiez ensuite si le [module Az.Storage 2.0 (ou version plus récente)](https://www.powershellgallery.com/packages/Az.Storage/2.0.0) est installé, et installez-le si ce n’est pas le cas. Une fois ces vérifications terminées, vérifiez votre AD DS pour déterminer si un [compte d’ordinateur](/windows/security/identity-protection/access-control/active-directory-accounts#manage-default-local-accounts-in-active-directory) (par défaut) ou un [compte d’ouverture de session du service](/windows/win32/ad/about-service-logon-accounts) a déjà été créé avec SPN/UPN comme « cifs/votre-nom-de-compte-de-stockage-ici.file.core.windows.net ». Si le compte n’existe pas, créez-en un comme décrit dans la section suivante.

### <a name="create-an-identity-representing-the-storage-account-in-your-ad-manually"></a>Créer manuellement une identité représentant le compte de stockage dans votre AD

Pour créer ce compte manuellement, créez une clé Kerberos pour votre compte de stockage. Ensuite, utilisez cette clé Kerberos comme mot de passe pour votre compte avec les applets de commande PowerShell ci-dessous. Cette clé est utilisée uniquement lors de la configuration et ne peut pas être utilisée pour des opérations de contrôle ou de plan de données sur le compte de stockage. 

```PowerShell
# Create the Kerberos key on the storage account and get the Kerb1 key as the password for the AD identity to represent the storage account
$ResourceGroupName = "<resource-group-name-here>"
$StorageAccountName = "<storage-account-name-here>"

New-AzStorageAccountKey -ResourceGroupName $ResourceGroupName -Name $StorageAccountName -KeyName kerb1
Get-AzStorageAccountKey -ResourceGroupName $ResourceGroupName -Name $StorageAccountName -ListKerbKey | where-object{$_.Keyname -contains "kerb1"}
```

Une fois que vous avez cette clé, créez un compte de service ou d’ordinateur sous votre UO. Utilisez la spécification suivante (n’oubliez pas de remplacer l’exemple de texte par le nom de votre compte de stockage) :

SPN : « cifs/votre-nom-de-compte-de-stockage-ici.file.core.windows.net » Mot de passe : Clé Kerberos pour votre compte de stockage.

Si votre UO applique l’expiration du mot de passe, vous devez mettre à jour le mot de passe avant la durée de vie maximale du mot de passe pour éviter les échecs d’authentification lors de l’accès aux partages de fichiers Azure. Pour plus d’informations, consultez [Mettre à jour le mot de passe de l’identité de votre compte de stockage dans AD](storage-files-identity-ad-ds-update-password.md).

Conservez l’ID de sécurité de l’identité nouvellement créée, car vous en aurez besoin pour l’étape suivante. L’identité que vous avez créée et qui représente le compte de stockage n’a pas besoin d’être synchronisée avec Azure AD.

#### <a name="optional-enable-aes256-encryption"></a>(Facultatif) Activer le chiffrement AES256

Si vous souhaitez activer le chiffrement AES 256, suivez les étapes de cette section. Si vous comptez utiliser RC4, vous pouvez ignorer cette section.

L’objet de domaine qui représente votre compte de stockage doit remplir les conditions suivantes :
- Le nom du compte de stockage ne peut pas dépasser 15 caractères.
- L’objet de domaine doit être créé en tant qu’objet ordinateur dans le domaine AD local.
- À l’exception du caractère « $ » final, le nom du compte de stockage doit être le même que le SamAccountName de l’objet de l’ordinateur.

Si votre objet de domaine ne répond pas à ces exigences, supprimez-le et créez un nouvel objet de domaine.

Remplacez `<domain-object-identity>` et `<domain-name>` par vos valeurs, puis utilisez la commande suivante pour configurer la prise en charge de AES256 : 

```powershell
Set-ADComputer -Identity <domain-object-identity> -Server <domain-name> -KerberosEncryptionType "AES256"
```

Après avoir exécuté cette commande, remplacez `<domain-object-identity>` dans le script suivant par votre valeur, puis exécutez le script pour actualiser le mot de passe de votre objet de domaine :

```powershell
$KeyName = "kerb1" # Could be either the first or second kerberos key, this script assumes we're refreshing the first
$KerbKeys = New-AzStorageAccountKey -ResourceGroupName $ResourceGroupName -Name $StorageAccountName -KeyName $KeyName
$KerbKey = $KerbKeys | Where-Object {$_.KeyName -eq $KeyName} | Select-Object -ExpandProperty Value
$NewPassword = ConvertTo-SecureString -String $KerbKey -AsPlainText -Force

Set-ADAccountPassword -Identity <domain-object-identity> -Reset -NewPassword $NewPassword
```

### <a name="enable-the-feature-on-your-storage-account"></a>Activer la fonctionnalité sur votre compte de stockage

Vous pouvez maintenant activer la fonctionnalité sur votre compte de stockage. Fournissez quelques détails de configuration pour les propriétés de domaine dans la commande suivante, puis exécutez-la. L’ID de sécurité du compte de stockage requis dans la commande suivante est l’ID de sécurité de l’identité que vous avez créée dans AD DS dans la [section précédente](#create-an-identity-representing-the-storage-account-in-your-ad-manually).

```PowerShell
# Set the feature flag on the target storage account and provide the required AD domain information
Set-AzStorageAccount `
        -ResourceGroupName "<your-resource-group-name-here>" `
        -Name "<your-storage-account-name-here>" `
        -EnableActiveDirectoryDomainServicesForFile $true `
        -ActiveDirectoryDomainName "<your-domain-name-here>" `
        -ActiveDirectoryNetBiosDomainName "<your-netbios-domain-name-here>" `
        -ActiveDirectoryForestName "<your-forest-name-here>" `
        -ActiveDirectoryDomainGuid "<your-guid-here>" `
        -ActiveDirectoryDomainsid "<your-domain-sid-here>" `
        -ActiveDirectoryAzureStorageSid "<your-storage-account-sid>"
```

### <a name="debugging"></a>Débogage

Vous pouvez exécuter l’applet de commande Debug-AzStorageAccountAuth pour effectuer un ensemble de vérifications de base sur votre configuration AD avec l’utilisateur AD connecté. Cette applet de commande est prise en charge sur AzFilesHybrid 0.1.2 et versions ultérieures. Pour plus d’informations sur les vérifications effectuées dans cette applet de commande, consultez [Impossible de monter Azure Files avec les informations d’identification AD](storage-troubleshoot-windows-file-connection-problems.md#unable-to-mount-azure-files-with-ad-credentials) dans le guide de résolution des problèmes pour Windows.

```PowerShell
Debug-AzStorageAccountAuth -StorageAccountName $StorageAccountName -ResourceGroupName $ResourceGroupName -Verbose
```

## <a name="confirm-the-feature-is-enabled"></a>Confirmer que la fonctionnalité est activée

Vous pouvez vérifier si la fonctionnalité est activée sur votre compte de stockage avec le script suivant :

```PowerShell
# Get the target storage account
$storageaccount = Get-AzStorageAccount `
        -ResourceGroupName "<your-resource-group-name-here>" `
        -Name "<your-storage-account-name-here>"

# List the directory service of the selected service account
$storageAccount.AzureFilesIdentityBasedAuth.DirectoryServiceOptions

# List the directory domain information if the storage account has enabled AD DS authentication for file shares
$storageAccount.AzureFilesIdentityBasedAuth.ActiveDirectoryProperties
```

En cas de réussite, la sortie doit ressembler à ceci :

```PowerShell
DomainName:<yourDomainHere>
NetBiosDomainName:<yourNetBiosDomainNameHere>
ForestName:<yourForestNameHere>
DomainGuid:<yourGUIDHere>
DomainSid:<yourSIDHere>
AzureStorageID:<yourStorageSIDHere>
```
## <a name="next-steps"></a>Étapes suivantes

Vous avez maintenant activé la fonctionnalité sur votre compte de stockage. Pour utiliser la fonctionnalité, vous devez assigner des autorisations au niveau du partage. Passez à la section suivante.

[Deuxième partie : affecter des autorisations au niveau du partage à une identité](storage-files-identity-ad-ds-assign-permissions.md)
