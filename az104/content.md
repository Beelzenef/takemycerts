# AZ-104

## Contenido

Tabla de contenidos para la certificación Az-104

- [AZ-104](#az-104)
  - [Contenido](#contenido)
  - [Azure Active Directory](#azure-active-directory)
  - [Role-Based Access Control (RBAC)](#role-based-access-control-rbac)
    - [Ver roles asignados](#ver-roles-asignados)
    - [Asignar roles](#asignar-roles)
    - [Deny assignments](#deny-assignments)
    - [Roles custom](#roles-custom)
  - [Manage subscriptions and governance](#manage-subscriptions-and-governance)
    - [Manage subscriptions](#manage-subscriptions)
      - [Configurar Cost Management](#configurar-cost-management)
    - [Manage governance](#manage-governance)
      - [Azure Management Groups](#azure-management-groups)
      - [Azure Policy](#azure-policy)
      - [Resource locks](#resource-locks)
      - [Resource groups](#resource-groups)
  - [Implement and manage storage](#implement-and-manage-storage)
    - [Azure Storage data objects](#azure-storage-data-objects)
    - [Storage accounts](#storage-accounts)
      - [Storage account endpoints](#storage-account-endpoints)
      - [Performance tiers](#performance-tiers)
      - [Access tiers](#access-tiers)
      - [Sobre redundancia](#sobre-redundancia)
      - [Sobre acceso y autorización](#sobre-acceso-y-autorización)

## Azure Active Directory

...

## Role-Based Access Control (RBAC)

Existen varios roles que podemos asignar a diferentes elementos. Se pueden asignar roles a:

- Users
- Groups
- Applications

Los *security principals* son:

- Users
- Groups
- Service principal
- Managed identity

Los core roles o principales son los siguientes:

- Owner
- Contributor
- Reader
- User access administrator

Funcionan en diferentes _scopes_:

- Mangement groups
- Subscriptions
- Resource groups
- Resources

Jerarquicamente, se visualizan así:

![Scopes hierarchy](../imgs/az104-subexample.png)

### Ver roles asignados

```ps
Get-AzRoleAssignment -ResourceGroupName $rgName
```

### Asignar roles

```ps
New-AzRoleAssigment -SignInName $email `
    -RoleDefinitionName $roleName
    -ResourceGroupName $rgName
```

### Deny assignments

Se pueden establecer denegaciones de asignación, para evitar que un usuario concreto escale de privilegios o obtenga roles específicos.

Estas denegaciones sobreescriben cualquier asignación, están por encima de cualquier otra asignación, para asegurarnos de que se cumplen medidas de seguridad.

Las denegaciones de asignación solo se pueden realizar mediante [Azure Policies](https://docs.microsoft.com/en-us/azure/governance/policy/overview).

### Roles custom

Podemos crear roles custom, para que se ajusten a nuestras necesidades específicas en nuestro management group o subscription.

Para crer un rol custom, tenemos tres vías posibles:

- Azure Portal
- ARM templates
- Powershell scripts

¿Qué información necesitamos?

- Nombre
- ID, valor *nullable*
- IsCustom, `boolean`
- Description
- Actions, array de `string`
- AssignableScopes, array de `string`

Un ejemplo de script para creación de rol custom:

```ps
$role = Get-AzRoleDefinition $roleNameAsTemplate

$role.Id = $null
$role.Name = $roleName
$role.Description = $roleDescription

$role.Actions.Clear()
$role.Actions.Add($roleAction1)
# more actions to add

$role.AssignableScopes.Clear()
$role.AssignableScopes.Add($roleScope1)
# more scopes to add

New-AzRoleDefinition -Role $role
```

¿Cómo saber qué *actions* puedo asignar a un rol? Podemos consultar las *actions* que tienen otros roles, para saber qué podemos asignar. Por ejemplo:

```ps
(Get-AzRoleDefinition "Virtual Machine Contributor").actions
```

Y así se nos listarán las *actions* que tenemos disponibles.

## Manage subscriptions and governance

### Manage subscriptions

Un *tenant* puede tener varias *subscriptions.*

Se pueden transferir *subscriptions* entre diferentes *tenants*.

Se pueden mover *resources* y *resource groups* entre diferentes *subscriptions,* siempre que se den ciertas condiciones que lo hagan posible.

#### Configurar Cost Management

Existen varios servicios que nos ayudan con la gestión de costes en nuestra suscripción.

- Analisis de costes y tendencias con Cost Analysis
- Cost Alerts para hacer saltar alertas con *thresholds* definidos
- Recommendations para seguir dentro del *budget* definido

Al final, son herramientas gráficas que nos ayudan a ver cómo gastamos los créditos de Azure. Podemos ver diferentes scopes, agrupados por tipo, crear alertas que despierten bajo ciertas condiciones...

Las tags nos ayudan a organizar los recursos por diversos parámetors. Son pares clave-valor. Para crear tags, se necesitan permisos de escritura en `Microsoft.Resources/tags`.

Obtener recursos por tags:

```ps
(Get-AzResource -TagValue $tagToSearch).Name
```

Crear tags para un recurso o grupo de recursos:

```ps
$tags = @{ 'project' = 'taleeng'; 'location' = 'London' }
New-AzTag -ResourceId $resId -Tag $tags
```

### Manage governance

#### Azure Management Groups

Los Azure Management Groups sirven para:

- gestionar de forma más eficiente el acceso, Policies y compliance
- engloba a las suscripciones como un nuevo scope
- las medidas (como Policies) aplicadas a un AMG se heredan a todas las suscripciones dentro del mismo

Existe un *management group* llamado `root`, y dentro de este puede haber más *management groups.*

Un *management group* puede contener varias *subscriptions,* pero una *subscription* solo puede estar dentro de un *management group.*

#### Azure Policy

Son reglas que se deben cumplir y mantener la *compliance* de la suscripción, siendo aplicadas en recursos existentes o futuros.

Se aplican en determinados *scopes,* ya sean grupos de recursos o recursos.

Una *initiative* es una colección de definiciones de *policies.*

```ps
$rg = Get-AzResourceGroup -Name $rgName

$definition = Get-AzPolicyDefinition | Where-Object { $_.Properties.DisplayName -eq $policyName }

New-AzPolicyAssignment -Name $policyName -DisplayName $policyDisplayName -Scope $rg.ResourceId -PolicyDefinition $definition
```

También es posible construir policies custom a través de Powershell.

#### Resource locks

Todos los recursos pueden tener un lock, ya sean recursos o grupos de recursos. Son heredables (si se aplica en un grupo de recursos, todos los recursos tienen ese lock).

Existen dos tipos:

- Read-only
- Delete

Estos locks se aplican a todos los roles y usuarios.

Se pueden ejecutar mediante:

- ARM templates
- Powershell
- Azure CLI

Ejemplo en Powershell:

```ps
New-AzResourceLock -LockLevel CanNotDelete -LockName $lockName -ResourceName $resName
```

#### Resource groups

Algunos recursos se pueden  mover entre grupos de recursos y suscripciones.

Lo que no cambia es la localización del recurso.

## Implement and manage storage

### Azure Storage data objects

Tipos de objeto:

- Blob
- File
- Queue
- Table
- Disks

### Storage accounts

Alta disponibilidad, puede contener todo los Azure storage objects, con namespaces únicos para acceder a elementos almacenados.

Tipos:

- General-purpose v2
- General-purpose v1
- BlockBlobStorage
- FileStorage
- BlobStorage

#### Storage account endpoints

- Azure Blob
  - az-storage.blob.core.windows.net
- Azure Table
  - az-storage.table.core.windows.net
- Azure Queue
  - az-storage.queue.core.windows.net
- Azure File
  - az-storage.file.core.windows.net

La ruta a un fichero bien puede ser, dependiendo del tipo:

`https://(storageacc-name).(type).core.windows.net/(container-name)/(file-name)`

#### Performance tiers

- Standard
  - Con backup para recuperación de datos ante desastres
  - Disponible para almacenar imágenes y vídeos
- Premium
  - Disponible para BlockBlobStorage, FileStorage GPv1+2
  - Interactivo
  - Con analíticas y servicios de AI/ML

Una vez se ha desplegado, no se puede modificar su performance tier.

#### Access tiers

- Hot: mayor coste de almacenamiento, menor coste de acceso
- Cool: menor coste de almacenamiento, mayor coste de acceso, 30 días mínimo
- Archive: menor coste de almacenamiento, mayor coste de acceso, 180 días mínimo

#### Sobre redundancia

Depende de cuantas regiones con la que estés operando.

- LRS
- ZRS
- GRS
- GZRS

![Azure storage durability and availability scenarios](../imgs/az104-storageaccs-redundancy.png)

![Azure storage account supported capabilities](../imgs/az104-storageaccs-capabilities.png)

#### Sobre acceso y autorización

![Authorizing access to storage data](../imgs/az104-storageaccs-access.png)
