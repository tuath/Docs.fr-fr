---
title: "État de session et d’application dans ASP.NET Core"
author: rick-anderson
description: "Approches de conservation d’application et l’état utilisateur (session) entre les demandes."
keywords: "ASP.NET Core, état de l’Application, l’état de session, querystring, valider"
ms.author: riande
manager: wpickett
ms.date: 06/08/2017
ms.topic: article
ms.assetid: 18cda488-0769-4cb9-82f6-4c6685f2045d
ms.technology: aspnet
ms.prod: asp.net-core
uid: fundamentals/app-state
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: c639d3b0d896b927bb2b70658032fc1bd8e87191
ms.sourcegitcommit: 78d28178345a0eea91556e4cd1adad98b1446db8
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/22/2017
---
# <a name="introduction-to-session-and-application-state-in-aspnet-core"></a>Introduction à l’état de session et d’application dans ASP.NET Core

Par [Rick Anderson](https://twitter.com/RickAndMSFT), [Steve Smith](https://ardalis.com/), et [Diana LaRose](https://github.com/DianaLaRose)

HTTP est un protocole sans état. Un serveur web traite chaque demande HTTP comme une demande indépendante et ne conserve pas les valeurs de l’utilisateur à partir de requêtes précédentes. Cet article décrit les différentes façons de conserver l’application et l’état de session entre les demandes. 

## <a name="session-state"></a>État de session

État de session est une fonctionnalité dans ASP.NET Core que vous pouvez utiliser pour enregistrer et stocker des données utilisateur pendant que l’utilisateur accède à votre application web. Consistant en une table de hachage ou de dictionnaire sur le serveur, l’état de session conserve les données entre les demandes à partir d’un navigateur. Les données de session sont sauvegardées par un cache.

ASP.NET Core maintient l’état de session en donnant au client un cookie qui contient l’ID de session, qui est envoyée au serveur avec chaque demande. Le serveur utilise l’ID de session pour extraire les données de session. Étant donné que le cookie de session est spécifique au navigateur, vous ne peuvez pas partager des sessions dans les navigateurs. Les cookies de session sont supprimées uniquement lorsque la session se termine. Si un cookie est reçu pour une session qui a expiré, une nouvelle session qui utilise le même cookie de session est créée. 

Le serveur conserve une session pendant une période limitée après la dernière demande. Vous pouvez définir le délai d’expiration d'une session ou utiliser la valeur par défaut de 20 minutes. L'état de session est idéal pour le stockage des données utilisateur qui sont spécifiques à une session particulière, mais ne doivent pas être persistantes de façon permanente. Les données sont supprimées du magasin de stockage lorsque vous appelez `Session.Clear` ou à l’expiration de la session dans le magasin de données. Le serveur ne peut pas détecter que le navigateur est fermé ou que le cookie de session est supprimé.

> [!WARNING]
> Ne stockez pas de données sensibles dans la session. Le client ne peut pas fermer le navigateur et effacer le cookie de session (et certains navigateurs maintenir les cookies de session sur windows). En outre, une session ne peut pas être restreinte à un seul utilisateur ; l’utilisateur suivant risque de continuer avec la même session.

Le fournisseur de session en mémoire stocke les données de session sur le serveur local. Si vous envisagez d’exécuter votre application web sur une batterie de serveurs, vous devez utiliser des sessions rémanentes pour lier chaque session à un serveur spécifique. La plateforme Windows Azure Web Sites par défaut, les sessions rémanentes (Application Request Routing ou ARR). Toutefois, les sessions rémanentes peuvent affecter l’évolutivité et compliquer la mise à jour des applications web. Une meilleure option consiste à utiliser le Redis ou distribuées SQL Server met en cache, qui ne nécessitent pas sessions rémanentes. Pour plus d’informations, consultez [fonctionne avec un Cache distribué de](xref:performance/caching/distributed). Pour plus d’informations sur la configuration des fournisseurs de services, consultez [configuration de Session](#configuring-session) plus loin dans cet article.

Le reste de cette section décrit les options pour le stockage des données utilisateur.

<a name="temp"></a>
### <a name="tempdata"></a>TempData

ASP.NET MVC de base expose le [TempData](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.controller#Microsoft_AspNetCore_Mvc_Controller_TempData) propriété sur un [contrôleur](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.controller). Cette propriété stocke les données jusqu’à ce qu’elles soient lues. Vous pouvez utiliser les méthodes `Keep` et `Peek` pour examiner les données sans suppression. `TempData`est particulièrement utile pour la redirection, lorsqu’il manque des données plus longtemps qu’une demande unique. `TempData`s’appuie sur l’état de session. 

## <a name="cookie-based-tempdata-provider"></a>Fournisseur de TempData basé sur cookie 

Dans ASP.NET Core 1.1 et ultérieures, vous pouvez utiliser le fournisseur TempData basé sur cookie pour stocker les TempData d’un utilisateur dans un cookie. Pour activer le fournisseur TempData basé sur cookie, inscrire le `CookieTempDataProvider` service `ConfigureServices`:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc();
    // Add CookieTempDataProvider after AddMvc and include ViewFeatures.
    // using Microsoft.AspNetCore.Mvc.ViewFeatures;
    services.AddSingleton<ITempDataProvider, CookieTempDataProvider>();
}
```

Les données de cookie sont codées avec le [Base64UrlTextEncoder](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.authentication.base64urltextencoder). Étant donné que le cookie est chiffré et mémorisé en bloc, la limite de taille de cookie unique ne s’applique pas. Les données de cookie ne sont pas compressées, étant donné que la compression des données d’encryped peut entraîner des problèmes de sécurité telles que la [CRIME](https://wikipedia.org/wiki/CRIME_(security_exploit)) et [violation](https://wikipedia.org/wiki/BREACH_(security_exploit)) attaques. Pour plus d’informations sur le fournisseur TempData basé sur cookie, consultez [CookieTempDataProvider](https://github.com/aspnet/Mvc/blob/dev/src/Microsoft.AspNetCore.Mvc.ViewFeatures/ViewFeatures/CookieTempDataProvider.cs).

### <a name="query-strings"></a>Chaînes de requête

Vous pouvez passer une quantité limitée de données à partir d’une demande à un autre en l’ajoutant à la chaîne de requête de la nouvelle demande. Cela est utile pour capturer l’état de manière persistante qui autorise les liens avec état incorporée à partager par courrier électronique ou de réseaux sociaux. Toutefois, pour cette raison, vous ne devez jamais utiliser des chaînes de requête pour les données sensibles. En plus facilement partagé, y compris les données dans les chaînes de requête peut créer des opportunités pour [Cross-Site demande Forgery (CSRF)](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)) attaques, ce qui peuvent amener les utilisateurs à visiter les sites malveillants lors de l’authentification. Des personnes malveillantes peuvent ensuite dérober des données utilisateur à partir de votre application ou des actions malveillantes part de l’utilisateur. N’importe quel état de session ou application préservé doit protéger contre les attaques CSRF. Pour plus d’informations sur les attaques CSRF, consultez [attaques empêchant Cross-Site Request Forgery (XSRF/CSRF) dans ASP.NET Core](../security/anti-request-forgery.md).

### <a name="post-data-and-hidden-fields"></a>Données de publication et les champs masqués

Données peuvent être enregistrées dans les champs masqués et validées sur la demande suivante. Cela est courant dans les formulaires à plusieurs pages. Toutefois, étant donné que le client peut potentiellement falsifier des données, le serveur doit toujours revalider. 

### <a name="cookies"></a>Cookies

Les cookies permettent de stocker des données spécifiques à l’utilisateur dans les applications web. Étant donné que les cookies sont envoyés avec chaque demande, leur taille doit être conservée au minimum. Dans l’idéal, doit être stocké dans un cookie uniquement un identificateur avec les données réelles stockées sur le serveur. La plupart des navigateurs limiter les cookies à 4 096 octets. En outre, qu’un nombre limité de cookies est disponible pour chaque domaine.  

Étant donné que les cookies sont au risque de falsification, ils doivent être validées sur le serveur. Bien que la durabilité du cookie sur un client est soumis à l’expiration et l’intervention de l’utilisateur, ils sont généralement la forme la plus durable de persistance des données sur le client.

Les cookies sont souvent utilisés pour la personnalisation, où le contenu est personnalisé pour un utilisateur connu. Étant donné que l’utilisateur est identifié uniquement et pas authentifié dans la plupart des cas, vous pouvez généralement sécuriser un cookie en stockant le nom d’utilisateur, nom de compte ou un ID d’utilisateur unique (par exemple, un GUID) dans le cookie. Vous pouvez ensuite utiliser le cookie pour accéder à l’infrastructure de personnalisation utilisateur d’un site.

### <a name="httpcontextitems"></a>HttpContext.Items

Le `Items` collection est un bon emplacement pour stocker des données qui sont nécessaire lors uniquement un traitement de la demande particulier. Contenu de la collection est supprimés après chaque demande. Le `Items` collection est mieux utilisée comme un moyen pour les composants ou d’intergiciel (middleware) pour communiquer lorsqu’ils fonctionnent à différents moments dans le temps au cours d’une demande et n’ont aucun moyen direct pour passer des paramètres. Pour plus d’informations, consultez [utilisation de HttpContext.Items](#working-with-httpcontextitems), plus loin dans cet article.

### <a name="cache"></a>d'instance/de clé

La mise en cache est un moyen efficace de stocker et récupérer des données. Vous pouvez contrôler la durée de vie des éléments du cache en fonction de l’heure et d’autres considérations. En savoir plus sur [mise en cache](../performance/caching/index.md).

<a name=session></a>

## <a name="configuring-session"></a>Configuration de Session

Le `Microsoft.AspNetCore.Session` package fournit un intergiciel (middleware) pour gérer l’état de session. Pour activer l’intergiciel de session, `Startup`doit contenir :

- Un de le [IDistributedCache](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.caching.distributed.idistributedcache) caches mémoire. Le `IDistributedCache` implémentation est utilisée comme un magasin de stockage pour la session.
- [AddSession](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.dependencyinjection.sessionservicecollectionextensions#Microsoft_Extensions_DependencyInjection_SessionServiceCollectionExtensions_AddSession_Microsoft_Extensions_DependencyInjection_IServiceCollection_) appeler, ce qui nécessite le package NuGet « Microsoft.AspNetCore.Session ».
- [UseSession](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.builder.sessionmiddlewareextensions#methods_) appeler.

Le code suivant montre comment définir le fournisseur de session en mémoire.

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

[!code-csharp[Main](app-state/sample/src/WebAppSessionDotNetCore2.0App/Startup.cs?highlight=11-19,24)]

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

[!code-csharp[Main](app-state/sample/src/WebAppSession/Startup.cs?highlight=11-19,24)]

---

Vous pouvez faire référence à la Session à partir de `HttpContext` une fois qu’il est installé et configuré.

Si vous essayez d’accéder à `Session` avant `UseSession` a été appelée, l’exception `InvalidOperationException: Session has not been configured for this application or request` est levée.

Si vous essayez de créer un nouveau `Session` (autrement dit, aucun cookie de session n’a été créé) une fois que vous avez déjà commencé l’écriture dans le `Response` diffuser en continu, l’exception `InvalidOperationException: The session cannot be established after the response has started` est levée. L’exception peut être trouvée dans le journal de serveur web ; Il ne sera pas affiché dans le navigateur.

### <a name="loading-session-asynchronously"></a>Chargement asynchrone de Session 

Le fournisseur de session par défaut dans ASP.NET Core charge l’enregistrement de session sous-jacent [IDistributedCache](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.caching.distributed.idistributedcache) magasin de façon asynchrone uniquement si le [ISession.LoadAsync](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.http.isession#Microsoft_AspNetCore_Http_ISession_LoadAsync) méthode est appelée explicitement avant  le `TryGetValue`, `Set`, ou `Remove` méthodes. Si `LoadAsync` n’est pas appelée en premier lieu, sous-jacent enregistrement de session est chargée de façon synchrone, ce qui pourrait avoir un impact sur la capacité de l’application à l’échelle.

Pour que les applications d’appliquer ce modèle, vous devez encapsuler le [DistributedSessionStore](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.session.distributedsessionstore) et [DistributedSession](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.session.distributedsession) implémentations avec des versions qui lèvent une exception si le `LoadAsync` méthode n’est pas appelé avant `TryGetValue`, `Set`, ou `Remove`. Enregistrer les versions encapsulées dans le conteneur de services.

### <a name="implementation-details"></a>Détails d’implémentation

Session utilise un cookie pour suivre et identifier les demandes à partir d’un seul navigateur. Par défaut, ce cookie est nommé ». AspNet.Session » et qu’il utilise un chemin d’accès « / ». Étant donné que la valeur par défaut du cookie ne spécifie pas un domaine, il n'est pas mis à disposition le script côté client dans la page (car `CookieHttpOnly` par défaut est `true`).

Pour remplacer les valeurs par défaut de la session, utilisez `SessionOptions`:

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

[!code-csharp[Main](app-state/sample/src/WebAppSessionDotNetCore2.0App/StartupCopy.cs?name=snippet1&highlight=8-12)]

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

[!code-csharp[Main](app-state/sample/src/WebAppSession/StartupCopy.cs?name=snippet1&highlight=8-12)]

---

Le serveur utilise le `IdleTimeout` propriété pour déterminer la durée pendant laquelle une session peut être inactive avant que son contenu est abandonné. Cette propriété est indépendante de l’expiration du cookie. Chaque demande qui passe par l’intergiciel de Session (lues ou écrites pour) réinitialise le délai d’attente.

Étant donné que `Session` est *sans verrouillage*, si deux demandes tentent de modifier le contenu de la session, il remplace le premier. `Session`est implémenté comme un *session cohérente*, ce qui signifie que tout le contenu est stocké ensemble. Deux requêtes qui sont modifier différentes parties de la session (clés différents) peuvent toujours avoir un impact sur eux.

## <a name="setting-and-getting-session-values"></a>Définition et l’obtention des valeurs de Session

Session est accessible via la `Session` propriété sur `HttpContext`. Cette propriété est un [ISession](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.http.isession) implémentation.

L’exemple suivant illustre la définition et l’obtention d’une valeur int et une chaîne :

[!code-csharp[Main](app-state/sample/src/WebAppSession/Controllers/HomeController.cs?name=snippet1)]

Si vous ajoutez les méthodes d’extension, vous pouvez définir et obtenir des objets sérialisables à la Session :

[!code-csharp[Main](app-state/sample/src/WebAppSession/Extensions/SessionExtensions.cs)]

L’exemple suivant montre comment définir et obtenir un objet sérialisable :

[!code-csharp[Main](app-state/sample/src/WebAppSession/Controllers/HomeController.cs?name=snippet2)]


## <a name="working-with-httpcontextitems"></a>Utilisation de HttpContext.Items

Le `HttpContext` abstraction fournit la prise en charge pour une collection de dictionnaires de type `IDictionary<object, object>`, appelé `Items`. Cette collection est disponible depuis le début d’une *HttpRequest* et supprimées à la fin de chaque demande. Il est accessible en affectant une valeur à une entrée de clé, ou en demandant la valeur d’une clé particulière.

Dans l’exemple ci-dessous, [intergiciel (middleware)](middleware.md) ajoute `isVerified` à la `Items` collection.

```csharp
app.Use(async (context, next) =>
{
    // perform some verification
    context.Items["isVerified"] = true;
    await next.Invoke();
});
```

Plus tard dans le pipeline, un autre intergiciel (middleware) peut y accéder :

```csharp
app.Run(async (context) =>
{
    await context.Response.WriteAsync("Verified request? " + 
        context.Items["isVerified"]);
});
```

Pour un intergiciel (middleware) qui servira uniquement par une application unique, `string` clés sont acceptables. Toutefois, intergiciel (middleware) qui est partagée entre les applications doit utiliser des clés de l’objet unique afin d’éviter tout risque de collision de clés. Si vous développez un intergiciel (middleware) qui doit fonctionner sur plusieurs applications, utilisez une clé d’objet unique définie dans votre classe d’intergiciel (middleware) comme indiqué ci-dessous :

```csharp
public class SampleMiddleware
{
    public static readonly object SampleKey = new Object();

    public async Task Invoke(HttpContext httpContext)
    {
        httpContext.Items[SampleKey] = "some value";
        // additional code omitted
    }
}
```

Tout autre code peut accéder à la valeur stockée dans `HttpContext.Items` à l’aide de la clé exposée par la classe de l’intergiciel (middleware) :

```csharp
public class HomeController : Controller
{
    public IActionResult Index()
    {
        string value = HttpContext.Items[SampleMiddleware.SampleKey];
    }
}
```

Cette approche a également l’avantage d’éliminer la répétition de « chaînes magiques » à plusieurs endroits dans le code.

<a name=appstate-errors></a>

## <a name="application-state-data"></a>Données de l’état

Utilisez [Injection de dépendance](xref:fundamentals/dependency-injection) pour rendre les données accessibles à tous les utilisateurs :

1. Définir un service qui contient les données (par exemple, une classe nommée `MyAppData`).

```csharp
public class MyAppData
{
    // Declare properties/methods/etc.
} 
```
2. Ajouter la classe de service `ConfigureServices` (par exemple `services.AddSingleton<MyAppData>();`).
3. Consommer la classe de service de données dans chaque contrôleur :

```csharp
public class MyController : Controller
{
    public MyController(MyAppData myService)
    {
        // Do something with the service (read some data from it, 
        // store it in a private field/property, etc.)
    }
} 
```

### <a name="common-errors-when-working-with-session"></a>Erreurs courantes lors de l’utilisation avec une session

* « Impossible de résoudre le service pour le type 'Microsoft.Extensions.Caching.Distributed.IDistributedCache' lors de la tentative d’activation 'Microsoft.AspNetCore.Session.DistributedSessionStore'. »

  Cela est généralement dû au moins une configuration `IDistributedCache` implémentation. Pour plus d’informations, consultez [fonctionne avec un Cache distribué de](xref:performance/caching/distributed) et [mise en mémoire cache](xref:performance/caching/memory).

### <a name="additional-resources"></a>Ressources supplémentaires


* [ASP.NET Core 1.x : exemple de code utilisé dans ce document](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/app-state/sample/src/WebAppSession)
* [ASP.NET Core 2.x : exemple de code utilisé dans ce document](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/app-state/sample/src/WebAppSessionDotNetCore2.0App)
