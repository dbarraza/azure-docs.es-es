---
title: Creación de un clúster privado con Red Hat OpenShift 3.11 en Azure | Microsoft Docs
description: Creación de un clúster privado con Red Hat OpenShift 3.11 en Azure
author: sakthi-vetrivel
ms.author: suvetriv
ms.service: container-service
ms.topic: conceptual
ms.date: 03/02/2020
keywords: aro, openshift, private cluster, red hat
ms.openlocfilehash: 37e9dc996fddf2b592ea6bf7fff1e1f4825f3ca8
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/20/2020
ms.locfileid: "92220635"
---
# <a name="create-a-private-cluster-with-azure-red-hat-openshift-311"></a>Creación de un clúster privado con Red Hat OpenShift 3.11 en Azure

> [!IMPORTANT]
> Red Hat OpenShift en Azure 3.11 se retirará el 30 de junio de 2022. La compatibilidad con la creación de nuevos clústeres de Red Hat OpenShift en Azure 3.11 continúa hasta el 30 de noviembre de 2020. Después de la retirada, los clústeres de Red Hat OpenShift en Azure 3.11 que queden se cerrarán para evitar vulnerabilidades de seguridad.
> 
> Siga esta guía para [crear un clúster de la versión 4 de Red Hat OpenShift en Azure](tutorial-create-cluster.md).
> Si tiene alguna pregunta específica, póngase en [contacto con nosotros](mailto:arofeedback@microsoft.com).

Los clústeres privados ofrecen las ventajas siguientes:

* Los clústeres privados no exponen los componentes del plano de control del clúster (por ejemplo, los servidores de API) en una dirección IP pública.
* Los clientes pueden configurar la red virtual de un clúster privado, lo que le permite configurar las redes para permitir el emparejamiento con otras redes virtuales incluso en los entornos de ExpressRoute. También puede configurar DNS personalizados en la red virtual para que se integren con los servicios internos.

## <a name="before-you-begin"></a>Antes de empezar

Los campos del siguiente fragmento de código de configuración son nuevos y deben incluirse en la configuración del clúster. `managementSubnetCidr` debe estar dentro de la red virtual del clúster y se usa en Azure para administrar el clúster.

```json
properties:
 networkProfile:
   managementSubnetCidr: 10.0.1.0/24
 masterPoolProfile:
   apiProperties:
     privateApiServer: true
```

Un clúster privado se puede implementar mediante los scripts de ejemplo que se proporcionan a continuación. Una vez implementado el clúster, ejecute el comando `cluster get` y vea la propiedad `properties.FQDN` para determinar la dirección IP privada del servidor de API de OpenShift.

La red virtual del clúster se habrá creado con permisos para que pueda modificarla. Después, puede configurar las redes para acceder a la red virtual (ExpressRoute, VPN, emparejamiento de redes virtuales) según sea necesario para sus necesidades.

Si cambia los nombres de los servidores DNS en la red virtual del clúster, tendrá que emitir una actualización en el clúster con la propiedad `properties.RefreshCluster` establecida en `true` para que se puedan restablecer las imágenes iniciales de las máquinas virtuales. Esta actualización le permitirá seleccionar los nuevos servidores de nombres.

## <a name="sample-configuration-scripts"></a>Scripts de configuración de ejemplo

Use los scripts de ejemplo de esta sección para configurar e implementar el clúster privado.

### <a name="environment"></a>Entorno

Rellene las variables de entorno siguientes con sus propios valores.

> [!NOTE]
> La ubicación debe establecerse en `eastus2` ya que esta es actualmente la única ubicación admitida para los clústeres privados.

``` bash
export CLUSTER_NAME=
export LOCATION=eastus2
export TOKEN=$(az account get-access-token --query 'accessToken' -o tsv)
export SUBID=
export TENANT_ID=
export ADMIN_GROUP=
export CLIENT_ID=
export SECRET=
```

### <a name="private-clusterjson"></a>private-cluster.json

Mediante las variables de entorno definidas anteriormente, este es un ejemplo de configuración de clúster con un clúster privado habilitado.

```json
{
   "location": "$LOCATION",
   "name": "$CLUSTER_NAME",
   "properties": {
       "openShiftVersion": "v3.11",
       "networkProfile": {
           "vnetCIDR": "10.0.0.0/8",
           "managementSubnetCIDR" : "10.0.1.0/24"
       },
       "authProfile": {
           "identityProviders": [
               {
                   "name": "Azure AD",
                   "provider": {
                       "kind": "AADIdentityProvider",
                       "clientId": "$CLIENT_ID",
                       "secret": "$SECRET",
                       "tenantId": "$TENANT_ID",
                       "customerAdminGroupID": "$ADMIN_GROUP"
                   }
               }
           ]
       },
       "masterPoolProfile": {
           "name": "master",
           "count": 3,
           "vmSize": "Standard_D4s_v3",
           "osType": "Linux",
           "subnetCIDR": "10.0.0.0/24",
           "apiProperties": {
                "privateApiServer": true
            }
       },
       "agentPoolProfiles": [
           {
               "role": "compute",
               "name": "compute",
               "count": 1,
               "vmSize": "Standard_D4s_v3",
               "osType": "Linux",
               "subnetCIDR": "10.0.0.0/24"
           },
           {
               "role": "infra",
               "name": "infra",
               "count": 3,
               "vmSize": "Standard_D4s_v3",
               "osType": "Linux",
               "subnetCIDR": "10.0.0.0/24"
           }
       ],
       "routerProfiles": [
           {
               "name": "default"
           }
       ]
   }
}
```

## <a name="deploy-a-private-cluster"></a>Implementación de un clúster privado

Después de configurar el clúster privado con los scripts de ejemplo anteriores, ejecute el siguiente comando para implementar el clúster privado.

``` bash
az group create --name $CLUSTER_NAME --location $LOCATION
cat private-cluster.json | envsubst | curl -v -X PUT \
-H 'Content-Type: application/json; charset=utf-8' \
-H 'Authorization: Bearer '$TOKEN'' -d @- \
 https://management.azure.com/subscriptions/$SUBID/resourceGroups/$CLUSTER_NAME/providers/Microsoft.ContainerService/openShiftManagedClusters/$CLUSTER_NAME?api-version=2019-10-27-preview
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo acceder a la consola de OpenShift, consulte el [tutorial de la consola web](https://docs.openshift.com/container-platform/3.11/getting_started/developers_console.html).
