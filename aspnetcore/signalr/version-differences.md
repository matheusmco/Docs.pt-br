---
title: Diferenças entre o SignalR e SignalR do ASP.NET Core
author: tdykstra
description: Diferenças entre o SignalR e SignalR do ASP.NET Core
monikerRange: '>= aspnetcore-2.1'
ms.author: tdykstra
ms.date: 08/20/2018
uid: signalr/version-differences
ms.openlocfilehash: b904f57af3700b6e1e2143913dfa08da9bf8bbd2
ms.sourcegitcommit: d27317c16f113e7c111583042ec7e4c5a26adf6f
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 08/20/2018
ms.locfileid: "41830914"
---
# <a name="differences-between-aspnet-signalr-and-aspnet-core-signalr"></a>Diferenças entre o SignalR do ASP.NET e o SignalR do ASP.NET Core

SignalR do ASP.NET Core não é compatível com clientes ou servidores para ASP.NET SignalR. Este artigo fornece detalhes sobre os recursos que foram removidos ou alterados no SignalR do ASP.NET Core.

## <a name="how-to-identify-the-signalr-version"></a>Como identificar a versão do SignalR

|                      | ASP.NET SignalR | SignalR do ASP.NET Core |
| -------------------- | --------------- | -------------------- |
| Pacote do NuGet Server | [Microsoft.AspNet.SignalR](https://www.nuget.org/packages/Microsoft.AspNet.SignalR/) | [Microsoft](https://www.nuget.org/packages/Microsoft.AspNetCore.App/) (.NET Core)<br>[Microsoft](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR/) (.NET Framework) |
| Pacotes NuGet de cliente | [ASPNET](https://www.nuget.org/packages/Microsoft.AspNet.SignalR.Client/)<br>[Microsoft.AspNet.SignalR.JS](https://www.nuget.org/packages/Microsoft.AspNet.SignalR.JS/) | [Aspnetcore](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client/) |
| Pacote npm de cliente | [SignalR](https://www.npmjs.com/package/signalr) | [@aspnet/signalr](https://www.npmjs.com/package/@aspnet/signalr) |
| Tipo de aplicativo de servidor | ASP.NET (System. Web) ou a auto-hospedagem de OWIN | ASP.NET Core |
| Plataformas de servidor com suporte | .NET framework 4.5 ou posterior | .NET Framework 4.6.1 ou posterior<br>.NET core 2.1 ou posterior |

## <a name="feature-differences"></a>Diferenças de recursos

### <a name="automatic-reconnects"></a>Reconexão automática

Reconexão automática não têm mais suporte. Anteriormente, o SignalR tentou se reconectar ao servidor se a conexão foi descartada. Agora, se o cliente é desconectado o usuário explicitamente deve iniciar uma nova conexão se eles desejam se reconectar.

### <a name="protocol-support"></a>Suporte de protocolo

SignalR do ASP.NET Core dá suporte a JSON, bem como um novo protocolo binário, com base em [MessagePack](xref:signalr/messagepackhubprotocol). Além disso, os protocolos personalizados podem ser criados.

## <a name="differences-on-the-server"></a>Diferenças no servidor

As bibliotecas do lado do servidor SignalR do ASP.NET Core são incluídas na [metapacote do Microsoft](xref:fundamentals/metapackage-app) que faz parte do pacote a **aplicativo Web ASP.NET Core** modelo Razor e MVC projetos.

SignalR do ASP.NET Core é um middleware do ASP.NET Core, portanto, ele deve ser configurado por meio da chamada [AddSignalR](/dotnet/api/microsoft.extensions.dependencyinjection.signalrdependencyinjectionextensions.addsignalr) em `Startup.ConfigureServices`.

```csharp
services.AddSignalR()
```

Para configurar o roteamento, mapear as rotas para os hubs de dentro de [UseSignalR](/dotnet/api/microsoft.aspnetcore.builder.signalrappbuilderextensions.usesignalr) chamada de método no `Startup.Configure` método.

```csharp
app.UseSignalR(routes =>
{
    routes.MapHub<ChatHub>("/hub");
});
```

### <a name="sticky-sessions-now-required"></a>Sessões adesivas agora é necessárias

Devido a como a expansão funcionou no SignalR do ASP.NET, os clientes podem se reconectar e enviar mensagens para qualquer servidor no farm. Devido a alterações para o modelo de expansão, bem como a não dar suporte a reconexão, isso não é mais suportado. Depois que o cliente se conecta ao servidor, ele deve interagir com o mesmo servidor durante a conexão.

### <a name="single-hub-per-connection"></a>Hub único por conexão

O SignalR do ASP.NET Core, o modelo de conexão foi simplificado. As conexões são feitas diretamente a um único hub, em vez de uma única conexão está sendo usado para compartilhar o acesso a vários hubs.

### <a name="streaming"></a>Streaming

ASP.NET SignalR Core agora dá suporte à [dados de streaming](xref:signalr/streaming) do hub para o cliente.

### <a name="state"></a>Estado

A capacidade de passar o estado arbitrário entre clientes e o hub (geralmente chamado de HubState) foi removida, bem como suporte para mensagens de progresso. Não há nenhum equivalente de proxies de hub no momento.

## <a name="differences-on-the-client"></a>Diferenças no cliente

### <a name="typescript"></a>TypeScript

O cliente SignalR do ASP.NET Core é escrito em [TypeScript](https://www.typescriptlang.org/). Você pode escrever em JavaScript ou TypeScript ao usar o [cliente JavaScript](xref:signalr/javascript-client).

### <a name="the-javascript-client-is-hosted-at-npmhttpswwwnpmjscom"></a>O cliente JavaScript é hospedado em [npm](https://www.npmjs.com/)

Nas versões anteriores, o cliente JavaScript foi obtido por meio de um pacote do NuGet no Visual Studio. Para as versões de núcleo, o [ @aspnet/signalr ](https://www.npmjs.com/package/@aspnet/signalr) pacote npm contém as bibliotecas de JavaScript. Este pacote não está incluído na **aplicativo Web ASP.NET Core** modelo. Usar npm para obter e instalar o `@aspnet/signalr` pacote npm.

```console
npm init -y
npm install @aspnet/signalr
```

### <a name="jquery"></a>jQuery

A dependência no jQuery foi removida, no entanto, projetos ainda podem usar jQuery.

### <a name="javascript-client-method-syntax"></a>Sintaxe de método de cliente JavaScript

A sintaxe de JavaScript foi alterado da versão anterior do SignalR. Em vez de usar o `$connection` de objeto, criar uma conexão usando o [HubConnectionBuilder](/javascript/api/%40aspnet/signalr/hubconnectionbuilder) API.

```javascript
const connection = new signalR.HubConnectionBuilder()
    .withUrl("/hub")
    .build();
```

Use o [em](/javascript/api/@aspnet/signalr/HubConnection#on) método para especificar que o hub pode chamar métodos do cliente.

```javascript
connection.on("ReceiveMessage", (user, message) => {
    const msg = message.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;");
    const encodedMsg = user + " says " + msg;
    log(encodedMsg);
});
```

Depois de criar o método do cliente, inicie a conexão de hub. Cadeia de um [catch](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise/catch) método para fazer logon ou lidar com erros.

```javascript
connection.start().catch(err => console.error(err.toString()));
```

### <a name="hub-proxies"></a>Proxies de Hub

Os proxies de Hub não automaticamente são gerados. Em vez disso, o nome do método é passado para o [invocar](/javascript/api/%40aspnet/signalr/hubconnection#invoke) API como uma cadeia de caracteres.

### <a name="net-and-other-clients"></a>.NET e outros clientes

O `Microsoft.AspNetCore.SignalR.Client` pacote NuGet contém as bibliotecas de cliente .NET para o SignalR do ASP.NET Core.

Use o [HubConnectionBuilder](/dotnet/api/microsoft.aspnetcore.signalr.client.hubconnectionbuilder) para criar e compilar uma instância de uma conexão a um hub.

```csharp
connection = new HubConnectionBuilder()
    .WithUrl("url")
    .Build();
```

## <a name="additional-resources"></a>Recursos adicionais

* [Hubs](xref:signalr/hubs)
* [Cliente JavaScript](xref:signalr/javascript-client)
* [Cliente .NET](xref:signalr/dotnet-client)
* [Plataformas compatíveis](xref:signalr/supported-platforms)
