---
title: Introducción a la consulta de servicios entre Azure Monitor y Azure Data Explorer (versión preliminar)
description: Consulte los datos de Azure Data Explorer a través de las herramientas de Azure Log Analytics, y viceversa, para unir y analizar todos los datos en un solo lugar.
author: osalzberg
ms.author: bwren
ms.reviewer: bwren
ms.subservice: logs
ms.topic: conceptual
ms.date: 06/12/2020
ms.openlocfilehash: 2b68afcb9d200970ca4ea29b13175223f52c77e0
ms.sourcegitcommit: 78ecfbc831405e8d0f932c9aafcdf59589f81978
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/23/2021
ms.locfileid: "98730996"
---
# <a name="cross-service-query---azure-monitor-and-azure-data-explorer-preview"></a>Consulta entre servicios: Azure Monitor y Azure Data Explorer (versión preliminar)
Cree consultas entre los servicios [Azure Data Explorer](/azure/data-explorer/), [Application Insights](../app/app-insights-overview.md) y [Log Analytics](./data-platform-logs.md).
## <a name="azure-monitor-and-azure-data-explorer-cross-service-querying"></a>Consulta entre los servicios Azure Monitor y Azure Data Explorer
Esta experiencia le permite [crear consultas entre servicios entre Azure Data Explorer y Azure Monitor](/azure/data-explorer/query-monitor-data) y [crear consultas entre servicios entre Azure Monitor y Azure Data Explorer](./azure-monitor-data-explorer-proxy.md).

Por ejemplo, (consultando Azure Data Explorer desde Log Analytics):
```kusto
CustomEvents | where aField == 1
| join (adx("Help/Samples").TableName | where secondField == 3) on $left.Key == $right.key
```
Donde la consulta externa consulta una tabla en el área de trabajo y luego se combina con otra tabla en un clúster de Azure Data Explorer (en este caso, clustername=help, databasename=samples) mediante una nueva función "adx()", como la forma en que se puede realizar la misma consulta en otra área de trabajo desde el texto de la consulta.

> [!NOTE]
> * La capacidad de consultar los datos de Azure Monitor desde Azure Data Explorer, ya sea directamente desde las herramientas de cliente de Azure Data Explorer o indirectamente mediante la ejecución de una consulta en un clúster de Azure Data Explorer, está en su versión preliminar.
> * Póngase en contacto con el equipo de [consultas entre servicios](mailto:adxproxy@microsoft.com) si tiene cualquier pregunta.

## <a name="query-exported-log-analytics-data-from-azure-blob-storage-account"></a>Consulta de datos de Log Analytics exportados desde la cuenta de Azure Blob Storage

Exportar datos de Azure Monitor a una cuenta de Azure Storage permite la retención a bajo costo y la capacidad de reasignar registros a regiones distintas.

Use Azure Data Explorer para consultar datos exportados de las áreas de trabajo de Log Analytics. Una vez que se configuran, las tablas compatibles que se envían desde las áreas de trabajo hasta la cuenta de Azure Storage estarán disponibles como un origen de datos para Azure Data Explorer. [Consulta de datos exportados desde Azure Monitor mediante Azure Data Explorer (versión preliminar)](./azure-data-explorer-query-storage.md).

:::image type="content" source="media\azure-data-explorer-query-storage\exported-data-query.png" alt-text="Consulta de Azure Data Explorer desde el flujo de almacenamiento":::.

>[!tip] 
> * Para exportar todos los datos del área de trabajo de Log Analytics a una cuenta de Azure Storage o un centro de eventos, use la característica de exportación de datos del área de trabajo de Log Analytics de Azure Monitor Logs. [Consulte la exportación de datos del área de trabajo de Log Analytics en Azure Monitor (versión preliminar)](/azure/data-explorer/query-monitor-data).

## <a name="next-steps"></a>Pasos siguientes
Más información sobre:
* [Creación de consultas entre servicios entre Azure Data Explorer y Azure Monitor](/azure/data-explorer/query-monitor-data). Consulte datos de Azure Monitor desde Azure Data Explorer.
* [Creación de consultas entre servicios entre Azure Monitor y Azure Data Explorer](./azure-monitor-data-explorer-proxy.md). Consulte datos de Azure Data Explorer desde Azure Monitor.
* [Exportación de datos del área de trabajo de Log Analytics en Azure Monitor (versión preliminar)](/azure/data-explorer/query-monitor-data). Vincule y consulte una cuenta de Azure Blob Storage con datos exportados de Log Analytics.