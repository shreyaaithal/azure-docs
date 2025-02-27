---
title: 'Quickstart: Speech SDK for C# (Xamarin) platform setup - Speech service'
titleSuffix: Azure Cognitive Services
description: 'Use this guide to set up your platform for C# Xamarin with the Speech service SDK.'
services: cognitive-services
author: markamos
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 10/15/2020
ms.author: pafarley
ms.custom: devx-track-csharp, ignite-fall-2021
---

This guide shows how to install the [Speech SDK](~/articles/cognitive-services/speech-service/speech-sdk.md) for [Xamarin](/xamarin/get-started/what-is-xamarin), an open-source platform for building modern and performant applications for iOS, Android, and Windows with .NET. If you just want the package name to get started on your own, run `Install-Package Microsoft.CognitiveServices.Speech` in the NuGet console.

> [!NOTE]
> The Speech SDK for Xamarin supports Windows Desktop (x86 and x64) or Universal Windows Platform (x86, x64, ARM/ARM64), Android (x86, ARM32/64), and iOS (x64 simulator and ARM64).

[!INCLUDE [License Notice](~/includes/cognitive-services-speech-service-license-notice.md)]

## Prerequisites

This quickstart requires:

* On Windows, you need the [Microsoft Visual C++ Redistributable for Visual Studio 2019](https://support.microsoft.com/topic/the-latest-supported-visual-c-downloads-2647da03-1eea-4433-9aff-95f26a218cc0) for your platform. Installing this for the first time may require a restart.
* [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/)

## Create a Visual Studio project and install the Speech SDK

[!INCLUDE [](~/includes/cognitive-services-speech-service-quickstart-xamarin-create-proj.md)]

The Speech SDK is now installed. You may now delete or re-use the "helloworld" project you created in the previous steps.

## Next steps

[!INCLUDE [windows](../quickstart-list.md)]
