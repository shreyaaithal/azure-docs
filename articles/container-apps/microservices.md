---
title: Microservices with Azure Containers Apps Preview
description: Build a microservice in Azure Container Apps.
services: app-service
author: craigshoemaker
ms.service: app-service
ms.topic: conceptual
ms.date: 09/16/2021
ms.author: cshoe
ms.custom: ignite-fall-2021
---

# Microservices with Azure Containers Apps Preview

[Microservice architectures](https://azure.microsoft.com/solutions/microservice-applications/#overview) allow you to independently develop, upgrade, version, and scale core areas of functionality in an overall system. Azure Container Apps provides the foundation for deploying microservices featuring:

- Independent [scaling](scale-app.md), [versioning](application-lifecycle-management.md), and [upgrades](application-lifecycle-management.md)
- [Service discovery](connect-apps.md)
- Native [Dapr integration](microservices-dapr.md)

:::image type="content" source="media/microservices/azure-container-services-microservices.png" alt-text="Container apps are deployed as microservices.":::

A Container Apps [environment](environment.md) provides a security boundary around a group of container apps. A single container app typically represents a microservice, which is composed of pods made up of one or more [containers](containers.md).

## Dapr integration

When implementing a system composed of microservices, function calls are spread across the network. To support the distributed nature of microservices, you need to account for failures, retries, and timeouts. While Container Apps features the building blocks for running microservices, use of [Dapr](https://docs.dapr.io/concepts/overview/) provides an even richer microservices programming model. Dapr includes features like observability, pub/sub, and service-to-service invocation with mutual TLS, retries, and more.

For more information on using Dapr, see [Build microservices with Dapr](microservices-dapr.md).

## Next steps

> [!div class="nextstepaction"]
> [Scaling](scale-app.md)
