---
title: "Programme d’installation de Twitter connexion externe"
author: rick-anderson
description: 
keywords: ASP.NET Core,
ms.author: riande
manager: wpickett
ms.date: 11/1/2016
ms.topic: article
ms.assetid: E5931607-31C0-4B20-B416-85E3550F5EA8
ms.technology: aspnet
ms.prod: asp.net-core
uid: security/authentication/twitter-logins
ms.openlocfilehash: 800f98285859a54198b76411aea000384de05cd3
ms.sourcegitcommit: 74e22e08e3b08cb576e5184d16f4af5656c13c0c
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/25/2017
---
# <a name="configuring-twitter-authentication"></a><span data-ttu-id="97a96-103">Configuration de l’authentification Twitter</span><span class="sxs-lookup"><span data-stu-id="97a96-103">Configuring Twitter authentication</span></span>

<a name=security-authentication-twitter-logins></a>

<span data-ttu-id="97a96-104">Par [Valeriy Novytskyy](https://github.com/01binary) et [Rick Anderson](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="97a96-104">By [Valeriy Novytskyy](https://github.com/01binary) and [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

<span data-ttu-id="97a96-105">Ce didacticiel vous montre comment permettre aux utilisateurs de [se connecter avec leur compte Twitter](https://dev.twitter.com/web/sign-in/desktop-browser) à l’aide d’un exemple de projet ASP.NET Core 2.0 créé sur le [page précédente](index.md).</span><span class="sxs-lookup"><span data-stu-id="97a96-105">This tutorial shows you how to enable your users to [sign in with their Twitter account](https://dev.twitter.com/web/sign-in/desktop-browser) using a sample ASP.NET Core 2.0 project created on the [previous page](index.md).</span></span>

## <a name="create-the-app-in-twitter"></a><span data-ttu-id="97a96-106">Créer l’application en Twitter</span><span class="sxs-lookup"><span data-stu-id="97a96-106">Create the app in Twitter</span></span>

* <span data-ttu-id="97a96-107">Accédez à [https://apps.twitter.com/](https://apps.twitter.com/) et connectez-vous.</span><span class="sxs-lookup"><span data-stu-id="97a96-107">Navigate to [https://apps.twitter.com/](https://apps.twitter.com/) and sign in.</span></span> <span data-ttu-id="97a96-108">Si vous n’avez pas encore un compte Twitter, utilisez le  **[s’inscrire maintenant](https://twitter.com/signup)**  lien pour en créer un.</span><span class="sxs-lookup"><span data-stu-id="97a96-108">If you don't already have a Twitter account, use the **[Sign up now](https://twitter.com/signup)** link to create one.</span></span> <span data-ttu-id="97a96-109">Une fois connecté, le **gestion des applications** page s’affiche :</span><span class="sxs-lookup"><span data-stu-id="97a96-109">After signing in, the **Application Management** page is shown:</span></span>

![Gestion des applications Twitter ouvert dans Microsoft Edge](index/_static/TwitterAppManage.png)

* <span data-ttu-id="97a96-111">Appuyez sur **créer une application** et remplir l’application **nom**, **Description** public et **site Web** URI (Cela peut être temporaire jusqu'à ce que vous Enregistrez le domaine nom) :</span><span class="sxs-lookup"><span data-stu-id="97a96-111">Tap **Create New App** and fill out the application **Name**, **Description** and public **Website** URI (this can be temporary until you register the domain name):</span></span>

![Créer une page d’application](index/_static/TwitterCreate.png)

* <span data-ttu-id="97a96-113">Entrez votre développement URI avec */signin-twitter* ajoutées dans le **URI de redirection OAuth valide** champ (par exemple : `https://localhost:44320/signin-twitter`).</span><span class="sxs-lookup"><span data-stu-id="97a96-113">Enter your development URI with */signin-twitter* appended into the **Valid OAuth Redirect URIs** field (for example: `https://localhost:44320/signin-twitter`).</span></span> <span data-ttu-id="97a96-114">Le schéma d’authentification Twitter configuré plus loin dans ce didacticiel va gérer automatiquement les demandes à */signin-twitter* itinéraire pour implémenter le flux OAuth.</span><span class="sxs-lookup"><span data-stu-id="97a96-114">The Twitter authentication scheme configured later in this tutorial will automatically handle requests at */signin-twitter* route to implement the OAuth flow.</span></span>

* <span data-ttu-id="97a96-115">Remplissez le reste du formulaire, puis appuyez sur **créer votre application Twitter**.</span><span class="sxs-lookup"><span data-stu-id="97a96-115">Fill out the rest of the form and tap **Create your Twitter application**.</span></span> <span data-ttu-id="97a96-116">Détails de l’application sont affichent :</span><span class="sxs-lookup"><span data-stu-id="97a96-116">New application details are displayed:</span></span>

![Onglet Détails de la page d’Application](index/_static/TwitterAppDetails.png)

* <span data-ttu-id="97a96-118">Lorsque vous déployez le site, vous devez revoir la **gestion des applications** page et inscrire un URI public.</span><span class="sxs-lookup"><span data-stu-id="97a96-118">When deploying the site you'll need to revisit the **Application Management** page and register a new public URI.</span></span>

## <a name="storing-twitter-consumerkey-and-consumersecret"></a><span data-ttu-id="97a96-119">Le stockage Twitter ConsumerKey et ConsumerSecret</span><span class="sxs-lookup"><span data-stu-id="97a96-119">Storing Twitter ConsumerKey and ConsumerSecret</span></span>

<span data-ttu-id="97a96-120">Lier les paramètres sensibles tels que Twitter `Consumer Key` et `Consumer Secret` à votre configuration d’application à l’aide du [Secret Manager](../../app-secrets.md).</span><span class="sxs-lookup"><span data-stu-id="97a96-120">Link sensitive settings like Twitter `Consumer Key` and `Consumer Secret` to your application configuration using the [Secret Manager](../../app-secrets.md).</span></span> <span data-ttu-id="97a96-121">Pour les besoins de ce didacticiel, nommez les jetons `Authentication:Twitter:ConsumerKey` et `Authentication:Twitter:ConsumerSecret`.</span><span class="sxs-lookup"><span data-stu-id="97a96-121">For the purposes of this tutorial, name the tokens `Authentication:Twitter:ConsumerKey` and `Authentication:Twitter:ConsumerSecret`.</span></span>

<span data-ttu-id="97a96-122">Ces jetons sont accessibles sur le **clés et les jetons d’accès** onglet après la création de votre nouvelle application Twitter :</span><span class="sxs-lookup"><span data-stu-id="97a96-122">These tokens can be found on the **Keys and Access Tokens** tab after creating your new Twitter application:</span></span>

![Onglet clés et les jetons d’accès](index/_static/TwitterKeys.png)

## <a name="configure-twitter-authentication"></a><span data-ttu-id="97a96-124">Configurer l’authentification Twitter</span><span class="sxs-lookup"><span data-stu-id="97a96-124">Configure Twitter Authentication</span></span>

<span data-ttu-id="97a96-125">Le modèle de projet utilisé dans ce didacticiel garantit que [Microsoft.AspNetCore.Authentication.Twitter](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Twitter) package est déjà installé.</span><span class="sxs-lookup"><span data-stu-id="97a96-125">The project template used in this tutorial ensures that [Microsoft.AspNetCore.Authentication.Twitter](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Twitter) package is already installed.</span></span>

* <span data-ttu-id="97a96-126">Pour installer ce package avec Visual Studio 2017, cliquez sur le projet et sélectionnez **gérer les Packages NuGet**.</span><span class="sxs-lookup"><span data-stu-id="97a96-126">To install this package with Visual Studio 2017, right-click on the project and select **Manage NuGet Packages**.</span></span>
* <span data-ttu-id="97a96-127">Pour installer avec l’interface CLI de .NET Core, exécutez le code suivant dans votre répertoire de projet :</span><span class="sxs-lookup"><span data-stu-id="97a96-127">To install with .NET Core CLI, execute the following in your project directory:</span></span>

   `dotnet add package Microsoft.AspNetCore.Authentication.Twitter`

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="97a96-128">ASP.NET Core 2.x</span><span class="sxs-lookup"><span data-stu-id="97a96-128">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

<span data-ttu-id="97a96-129">Ajoutez le service de Twitter dans le `ConfigureServices` méthode dans *Startup.cs* fichier :</span><span class="sxs-lookup"><span data-stu-id="97a96-129">Add the Twitter service in the `ConfigureServices` method in *Startup.cs* file:</span></span>

```csharp
services.AddAuthentication().AddTwitter(twitterOptions =>
{
    twitterOptions.ConsumerKey = Configuration["Authentication:Twitter:ConsumerKey"];
    twitterOptions.ConsumerSecret = Configuration["Authentication:Twitter:ConsumerSecret"];
});
```

<span data-ttu-id="97a96-130">Le `AddAuthentication` méthode doit uniquement être appelée qu’une seule fois lors de l’ajout de plusieurs fournisseurs d’authentification.</span><span class="sxs-lookup"><span data-stu-id="97a96-130">The `AddAuthentication` method should only be called once when adding multiple authentication providers.</span></span> <span data-ttu-id="97a96-131">Les appels suivants à ce dernier ont la possibilité de remplacement de tous configurés précédemment [AuthenticationOptions](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.builder.authenticationoptions) propriétés.</span><span class="sxs-lookup"><span data-stu-id="97a96-131">Subsequent calls to it have the potential of overriding any previously configured [AuthenticationOptions](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.builder.authenticationoptions) properties.</span></span>

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="97a96-132">ASP.NET Core 1.x</span><span class="sxs-lookup"><span data-stu-id="97a96-132">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

<span data-ttu-id="97a96-133">Ajouter l’intergiciel (middleware) Twitter dans le `Configure` méthode dans *Startup.cs* fichier :</span><span class="sxs-lookup"><span data-stu-id="97a96-133">Add the Twitter middleware in the `Configure` method in *Startup.cs* file:</span></span>

```csharp
app.UseTwitterAuthentication(new TwitterOptions()
{
    ConsumerKey = Configuration["Authentication:Twitter:ConsumerKey"],
    ConsumerSecret = Configuration["Authentication:Twitter:ConsumerSecret"]
});
```

---

<span data-ttu-id="97a96-134">Consultez le [TwitterOptions](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.builder.twitteroptions) référence des API pour plus d’informations sur les options de configuration prises en charge par l’authentification Twitter.</span><span class="sxs-lookup"><span data-stu-id="97a96-134">See the [TwitterOptions](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.builder.twitteroptions) API reference for more information on configuration options supported by Twitter authentication.</span></span> <span data-ttu-id="97a96-135">Cela peut être utilisé pour demander des différentes informations relatives à l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="97a96-135">This can be used to request different information about the user.</span></span>

## <a name="sign-in-with-twitter"></a><span data-ttu-id="97a96-136">Se connecter avec Twitter</span><span class="sxs-lookup"><span data-stu-id="97a96-136">Sign in with Twitter</span></span>

<span data-ttu-id="97a96-137">Exécutez votre application et cliquez sur **connecter**.</span><span class="sxs-lookup"><span data-stu-id="97a96-137">Run your application and click **Log in**.</span></span> <span data-ttu-id="97a96-138">Une option pour vous connecter avec Twitter s’affiche :</span><span class="sxs-lookup"><span data-stu-id="97a96-138">An option to sign in with Twitter appears:</span></span>

![Application Web : utilisateur non authentifié](index/_static/DoneTwitter.png)

<span data-ttu-id="97a96-140">En cliquant sur **Twitter** redirige vers Twitter pour l’authentification :</span><span class="sxs-lookup"><span data-stu-id="97a96-140">Clicking on **Twitter** redirects to Twitter for authentication:</span></span>

![Page d’authentification Twitter](index/_static/TwitterLogin.png)

<span data-ttu-id="97a96-142">Après avoir entré vos informations d’identification Twitter, vous êtes redirigé vers le site web où vous pouvez définir votre adresse de messagerie.</span><span class="sxs-lookup"><span data-stu-id="97a96-142">After entering your Twitter credentials, you are redirected back to the web site where you can set your email.</span></span>

<span data-ttu-id="97a96-143">Vous êtes désormais connecté à l’aide de vos informations d’identification Twitter :</span><span class="sxs-lookup"><span data-stu-id="97a96-143">You are now logged in using your Twitter credentials:</span></span>

![Application Web : utilisateur authentifié](index/_static/Done.png)

## <a name="troubleshooting"></a><span data-ttu-id="97a96-145">Résolution des problèmes</span><span class="sxs-lookup"><span data-stu-id="97a96-145">Troubleshooting</span></span>

* <span data-ttu-id="97a96-146">**ASP.NET Core 2.x uniquement :** si identité n’est pas configurée en appelant `services.AddIdentity` dans `ConfigureServices`, une tentative d’authentification entraîne *ArgumentException : l’option 'SignInScheme' doit être fournie*.</span><span class="sxs-lookup"><span data-stu-id="97a96-146">**ASP.NET Core 2.x only:** If Identity is not configured by calling `services.AddIdentity` in `ConfigureServices`, attempting to authenticate will result in *ArgumentException: The 'SignInScheme' option must be provided*.</span></span> <span data-ttu-id="97a96-147">Le modèle de projet utilisé dans ce didacticiel permet de s’assurer que cette opération est effectuée.</span><span class="sxs-lookup"><span data-stu-id="97a96-147">The project template used in this tutorial ensures that this is done.</span></span>
* <span data-ttu-id="97a96-148">Si la base de données de site n’a pas été créé en appliquant la migration initiale, vous obtiendrez *une opération de base de données a échoué lors du traitement de la demande* erreur.</span><span class="sxs-lookup"><span data-stu-id="97a96-148">If the site database has not been created by applying the initial migration, you will get *A database operation failed while processing the request* error.</span></span> <span data-ttu-id="97a96-149">Appuyez sur **s’appliquent les Migrations** pour créer la base de données et actualiser pour passer à l’erreur.</span><span class="sxs-lookup"><span data-stu-id="97a96-149">Tap **Apply Migrations** to create the database and refresh to continue past the error.</span></span>

## <a name="next-steps"></a><span data-ttu-id="97a96-150">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="97a96-150">Next steps</span></span>

* <span data-ttu-id="97a96-151">Cet article a montré comment vous pouvez vous authentifier avec Twitter.</span><span class="sxs-lookup"><span data-stu-id="97a96-151">This article showed how you can authenticate with Twitter.</span></span> <span data-ttu-id="97a96-152">Vous pouvez suivre une approche similaire pour s’authentifier auprès d’autres fournisseurs répertoriés sur le [page précédente](index.md).</span><span class="sxs-lookup"><span data-stu-id="97a96-152">You can follow a similar approach to authenticate with other providers listed on the [previous page](index.md).</span></span>

* <span data-ttu-id="97a96-153">Une fois que vous publiez votre site web à l’application web Azure, vous devez réinitialiser le `ConsumerSecret` dans le portail des développeurs Twitter.</span><span class="sxs-lookup"><span data-stu-id="97a96-153">Once you publish your web site to Azure web app, you should reset the `ConsumerSecret` in the Twitter developer portal.</span></span>

* <span data-ttu-id="97a96-154">Définir le `Authentication:Twitter:ConsumerKey` et `Authentication:Twitter:ConsumerSecret` en tant que paramètres de l’application dans le portail Azure.</span><span class="sxs-lookup"><span data-stu-id="97a96-154">Set the `Authentication:Twitter:ConsumerKey` and `Authentication:Twitter:ConsumerSecret` as application settings in the Azure portal.</span></span> <span data-ttu-id="97a96-155">Le système de configuration est configuré pour lire les clés à partir de variables d’environnement.</span><span class="sxs-lookup"><span data-stu-id="97a96-155">The configuration system is set up to read keys from environment variables.</span></span>