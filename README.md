# V2T2L
"Voice to Text to Language" - solution for live conference translations. Modular, scalable, powered by Microsoft Azure.

> **This repo is still work in progress, being regularly updated.**

## Scenario

This is a prototype, the goal is to provide a solution for near real-time conference transcription & translation.

1. English speaker is giving a presentation.
2. Our application streams voice from microphone to the cloud.
3. Cloud backend gets transcript (textual representation) of the speech.
4. Cloud backend also translates this transcript to multiple languages (currently 3).
5. Users (audience) subscribe to language of their choice on their devices (phones, tablets, laptops...).
6. Users get translated sentences as the speaker carries on with their talk.

The solution currently supports English speakers, but is designed in a way that any other language can be processed as well (by employing different speech-to-text provider).

## Architecture

Cornerstone of this solution is [Speech Service](https://docs.microsoft.com/en-us/azure/cognitive-services/Speech/Home), which is part of [Microsoft Cognitive Services](https://azure.microsoft.com/en-us/services/cognitive-services/).

![](_images/V2T2L.png)

Technology used:

* **JavaScript** - client applications for speakers & attendees
* **SignalR** - WebSocket communication
* **ASP.NET Core** - web APIs
* **Cognitive Services** - speech to text
* **Blob Storage** - storing of transcripts & translations
* **Storage Queues** - decoupling backend communication
* **Azure Functions** - orchestration of translations, sending results to attendees
* **Microsoft Translator** - translation to various languages, with neural networks (where possible)

## Prerequisites

To run the solution locally, you will need **[Visual Studio 2017](https://visualstudio.microsoft.com/vs/)** with **Azure development** workload and [**Azure Functions tools**](https://docs.microsoft.com/en-us/azure/azure-functions/functions-develop-vs) installed.

Besides that there are a few cloud dependencies:

* **Speech API** - create new resource in Azure or get a [free key](https://azure.microsoft.com/en-us/try/cognitive-services/?api=speech-services).
* **Azure Storage** - you can use local emulator, or create new Storage Account in Azure.
* **Translator Text** - create new resource in Azure.

## Solution structure

Most of our code is .NET, so we have used standard Solution-based structure. There are the following projects:

* **AttendeeApi** is the SignalR hub, where attendees connect from their devices. It takes care of registration and also distributes translations.
* **AttendeeApp** is the HTML/JavaScript client application which attendees use to choose a language and consume content.
* **Common** is just a Portable Class Library with constants and classes used by other projects.
* **HubApi** contains the recording web application and hosts the SignalR hub where speakers connect.
* **StreamGenerator** is a simple console app which sends stream of bytes to the API (from properly formatted WAV file).
* **TranslationOrchestrator** is a set of Azure Functions which take care of translating text to all supported languages in parallel and sending notification to *AttendeeApi*.

## Azure resources

Name, Type, Description

| Name                 | Type                 | Description                                                  |
| -------------------- | -------------------- | ------------------------------------------------------------ |
| attendee             | Web App              | Hosting for attendee client app. <br />Could be also Storage with static page hosting. |
| attendeeapi          | Web App              | SignalR Hub for attendee client.                             |
| hubapi               | Web App              | SignalR Hub for speaker client.<br />Hosting for speaker client app.<br />Application settings:<br />- `EventGridHost`<br />- `EventGridKey`<br />- `SpeechToTextApiKey`<br />- `StorageConnectionString`<br />- WebSockets -> On |
| orchestrator         | Function App         | Application settings:<br />- `AttendeeApiUrl`<br />- `AzureWebJobsStorage`<br />- `ServiceBusConnection` (unused)<br />- `StorageConnectionString`<br />- `TranslatorServiceKey` |
| orchestratorPlan     | App Service Plan     | Consumption plan for *orchestrator* Function App.            |
| hubPlan              | App Service Plan     | App Service Plan for all other App Services.                 |
| speech               | Speech (preview)     | Speech to Text API                                           |
| translate            | Translator Text      | Translator API                                               |
| translation-insights | Application Insights | Telemetry tied to *orchestrator* Function App.               |
| transcript           | Event Grid Topic     | Topic with event subscription.<br />Sending event type `TranscriptAdded` to *orchestrator* Function. |
| storage              | Storage Account      | Common Storage account for transcripts, translations and *orchestrator*. |

