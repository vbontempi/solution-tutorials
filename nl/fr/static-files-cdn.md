---
copyright:
  years: 2017, 2019
lastupdated: "2019-03-07"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Accélération de la distribution de fichiers statiques à l'aide d'un CDN
{: #static-files-cdn}

Ce tutoriel vous explique comment héberger et servir des actifs de site Web (images, vidéos, documents) et du contenu généré par l'utilisateur dans un stockage {{site.data.keyword.cos_full_notm}}, et comment utiliser un réseau [{{site.data.keyword.cdn_full}} (CDN)](https://{DomainName}/catalog/infrastructure/cdn-powered-by-akamai) pour une distribution rapide et sécurisée aux utilisateurs autour du monde.

Les applications Web ont différents types de contenu : contenu HTML, images, vidéos, feuilles de style en cascade, fichiers JavaScript, contenu généré par l'utilisateur. Certains contenus changent souvent, d'autres moins, certains sont très souvent consultés par de nombreux utilisateurs, d'autres occasionnellement. Au fur et à mesure que l'audience de l'application augmente, vous souhaiterez peut-être décharger la fourniture de ces contenus à un autre composant, libérant ainsi des ressources pour votre application principale. Vous pouvez également faire en sorte que ces contenus soient diffusés à partir d'un emplacement proche des utilisateurs de votre application, où qu'ils se trouvent dans le monde. 

Il existe de nombreuses raisons pour lesquelles vous pouvez utiliser un réseau CDN dans les situations suivantes : 
* le CDN met le contenu en cache, en extrayant le contenu de l'origine (vos serveurs) uniquement s'il n'est pas disponible dans son cache ou s'il a expiré ; 
* avec plusieurs centres de données à travers le monde, le CDN diffuse le contenu mis en cache à partir de l'emplacement le plus proche pour vos utilisateurs ; 
* s'il s'exécute sur un domaine différent de celui de votre application principale, le navigateur peut charger davantage de contenu en parallèle - la plupart des navigateurs sont limités en nombre de connexions par nom d’hôte. 

## Objectifs
{: #objectives}

* Télécharger des fichiers vers un compartiment {{site.data.keyword.cos_full_notm}}.
* Rendre le contenu disponible globalement avec un réseau de diffusion de contenu (CDN). 
* Exposer des fichiers à l'aide d'une application Web Cloud Foundry. 

## Services utilisés
{: #services}

Ce tutoriel utilise les produits suivants : 
   * [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)
   * [{{site.data.keyword.cdn_full}}](https://{DomainName}/catalog/infrastructure/cdn-powered-by-akamai)

Ce tutoriel peut entraîner des coûts. Utilisez la [Calculatrice de prix](https://{DomainName}/pricing/) pour générer une estimation du coût en fonction de l'utilisation envisagée. 

## Architecture

<p style="text-align: center;">
![Architecture](images/solution3/Architecture.png)
</p>

1. L'utilisateur accède à l'application
2. L'application comprend du contenu distribué via un réseau de diffusion de contenu 
3. Si le contenu n'est pas disponible sur le CDN ou a expiré, le CDN extrait le contenu de l'origine. 

## Avant de commencer
{: #prereqs}

Contactez l'utilisateur principal de votre compte d'infrastructure pour obtenir les droits d'accès suivants : 
   * Gérer un compte CDN
   * Gérer le stockage
   * Clé d'API 

Ces droits sont nécessaires pour afficher et utiliser les services Storage et CDN. 

## Obtention du code d'application Web
{: #get_code}

Prenons une application Web simple avec différents types de contenu, tels que des images, des vidéos et des feuilles de style en cascade. Vous stockez le contenu dans un compartiment de stockage et configurez le CDN pour utiliser le compartiment comme origine. 

Pour commencer, extrayez le code de l'application : 

   ```sh
   git clone https://github.com/IBM-Cloud/webapp-with-cos-and-cdn
   ```
  {: pre}

## Création d'une instance Object Storage
{: #create_cos}

{{site.data.keyword.cos_full_notm}} fournit un stockage dans le cloud flexible, économique et évolutif pour les données non structurées.

1. Accédez au [catalogue](https://{DomainName}/catalog/) dans la console, puis sélectionnez [**Object Storage**](https://{DomainName}/catalog/services/cloud-object-storage) dans la section Storage.
2. Créez une instance de {{site.data.keyword.cos_full_notm}}
4. Dans le tableau de bord de service, cliquez sur **Créer un compartiment**.
5. Définissez un nom de compartiment unique, tel que `username-mywebsite` et cliquez sur **Créer**. Evitez les points (.) dans le nom du compartiment. 

## Téléchargement de fichiers vers un compartiment
{: #upload}

Dans cette section, vous utilisez l'outil de ligne de commande **curl** pour télécharger des fichiers dans le compartiment. 

1. Connectez-vous à {{site.data.keyword.Bluemix_notm}} à partir de la CLI et obtenez un jeton auprès d'IAM d'IBM Cloud.
   ```sh
   ibmcloud login
   ibmcloud iam oauth-tokens
   ```
   {: pre}
2. Copiez le jeton à partir de la sortie de la commande à l'étape précédente.
   ```
   IAM token:  Bearer <token>
   ```
   {: screen}
3. Définissez la valeur du jeton et du nom du compartiment sur une variable d'environnement pour faciliter l'accès. 
   ```sh
   export IAM_TOKEN=<REPLACE_WITH_TOKEN>
   export BUCKET_NAME=<REPLACE_WITH_BUCKET_NAME>
   ```
   {: pre}
4. Téléchargez les fichiers nommés `a-css-file.css`, `a-picture.png` et `a-video.mp4` à partir du répertoire de contenu du code de l'application Web que vous avez précédemment téléchargé. Téléchargez les fichiers à la racine du compartiment.
  ```sh
   cd content
  ```
  {: pre}
  ```sh
   curl -X "PUT" \
         "https://s3-api.us-geo.objectstorage.softlayer.net/$BUCKET_NAME/a-picture.png" \
        -H "x-amz-acl: public-read" \
        -H "Authorization: Bearer $IAM_TOKEN" \
        -H "Content-Type: image/png" \
        -T a-picture.png
  ```
  {: pre}
  ```sh
   curl -X "PUT" \
         "https://s3-api.us-geo.objectstorage.softlayer.net/$BUCKET_NAME/a-css-file.css" \
        -H "x-amz-acl: public-read" \
        -H "Authorization: Bearer $IAM_TOKEN" \
        -H "Content-Type: text/css" \
        -T a-css-file.css
  ```
  {: pre}
  ```sh
   curl -X "PUT" \
         "https://s3-api.us-geo.objectstorage.softlayer.net/$BUCKET_NAME/a-video.mp4" \
        -H "x-amz-acl: public-read" \
        -H "Authorization: Bearer $IAM_TOKEN" \
        -H "Content-Type: video/mp4" \
        -T a-video.mp4
  ```
  {: pre}
5. Affichez les fichiers depuis le tableau de bord.
   ![](images/solution3/Buckets.png)
6. Accédez aux fichiers via votre navigateur en utilisant un lien similaire à l'exemple suivant :

   `http://s3-api.us-geo.objectstorage.softlayer.net/YOUR_BUCKET_NAME/a-picture.png`

## Mise à disposition des fichiers globalement avec un CDN 

Dans cette section, vous allez créer un service CDN. Le service CDN diffuse le contenu où il est nécessaire. La première fois que le contenu est demandé, il est extrait du serveur hôte (votre compartiment dans {{site.data.keyword.cos_full_notm}}) vers le réseau et reste sur celui-ci pour que les autres utilisateurs puissent y accéder rapidement en évitant les temps d'attente du réseau pour atteindre à nouveau le serveur hôte. 

### Création d'une instance CDN

1. Accédez au catalogue dans la console, puis sélectionnez **Réseau de diffusion de contenu (CDN)** dans la section Réseau. Ce CDN est alimenté par Akamai. Cliquez sur **Create**.
2. Dans la boîte de dialogue suivante, définissez le nom **Hostname** du CDN sur votre domaine personnalisé. Bien que vous avez défini un domaine personnalisé, vous pouvez toujours accéder au contenu du CDN via le CNAME fourni par IBM. Ainsi, si vous n'envisagez pas d'utiliser un domaine personnalisé, vous pouvez définir un nom arbitraire. 
3. Définissez le préfixe de **Custom CNAME** sur une valeur unique.
4. Ensuite, dans la section **Configure your origin**, sélectionnez **Object Storage** pour configurer le CDN pour COS.
5. Définissez **Endpoint** sur le noeud final d'API de compartiment, tel que *s3-api.us-geo.objectstorage.softlayer.net*.
6. Laissez **Host header** et **Path** vides. Définissez **Bucket name** sur *your-bucket-name*.
7. Activez les ports HTTP (80) et HTTPS (443). 
8. Pour **SSL certificate**, sélectionnez *DV SAN Certificate* si vous souhaitez utiliser un domaine personnalisé. Sinon, pour accéder au stockage via CNAME, choisissez l'option **Wildcard Certificate*.
9. Cliquez sur **Create**.

### Accès au contenu via le CNAME du CDN 

1. Sélectionnez l'instance CDN [dans la liste](https://{DomainName}/classic/network/cdn).
2. Si vous avez déjà choisi *DV SAN Certificate*, vous êtes invité à valider le domaine une fois la configuration initiale terminée. Suivez les étapes indiquées lorsque vous cliquez sur **View domain validation**.
3. Le panneau **Details** affiche à la fois le nom **Hostname** et le **CNAME** de votre CDN.
4. Accédez à votre fichier sur `https://your-cdn-cname.cdnedge.bluemix.net/a-picture.png` ou, si vous utilisez un domaine personnalisé, sur `https://your-cdn-hostname/a-picture.png`. Si vous omettez le nom du fichier, vous devriez plutôt voir S3 ListBucketResult.

## Déploiement de l'application Cloud Foundry

L'application contient une page Web public/index.html qui inclut des références aux fichiers actuellement hébergés dans {{site.data.keyword.cos_full_notm}}. Le back-end `app.js` diffuse cette page Web et remplace une marque de réservation par l'emplacement actuel de votre CDN. De cette façon, tous les actifs utilisés par la page Web sont diffusés par le CDN. 

1. A partir d'un terminal, accédez au répertoire où vous avez extrait le code. 
   ```
   cd webapp-with-cos-and-cdn
   ```
   {: pre}
2. Transférez l'application sans la démarrer.
   ```
   ibmcloud cf push --no-start
   ```
   {: pre}
3. Configurez la variable d'environnement CDN_NAME afin que l'application puisse faire référence au contenu du CDN. 
   ```
   ibmcloud cf set-env webapp-with-cos-and-cdn CDN_CNAME your-cdn.cdnedge.bluemix.net
   ```
   {: pre}
4. Démarrez l'application.
   ```
   ibmcloud cf start webapp-with-cos-and-cdn
   ```
   {: pre}
5. Accédez à l'application avec votre navigateur Web, la feuille de style de page, une image et une vidéo sont chargées à partir du CDN. 

![](images/solution3/Application.png)

L'association d'un service CDN à {{site.data.keyword.cos_full_notm}} est une solution puissante qui vous permet d'héberger des fichiers et de les transmettre à des utilisateurs du monde entier. Vous pouvez également utiliser {{site.data.keyword.cos_full_notm}} pour stocker les fichiers que vos utilisateurs transfèrent vers votre application.

## Suppression de ressources 

* Supprimez l'application Cloud Foundry
* Supprimez le service Content Delivery Network
* Supprimez le service ou le compartiment {{site.data.keyword.cos_full_notm}} 

## Contenu associé

[{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)

[Gestion de l'accès à {{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage/iam?topic=cloud-object-storage-getting-started-with-iam#getting-started-with-iam)

[Initiation au service CDN](https://{DomainName}/docs/infrastructure/CDN?topic=CDN-getting-started#getting-started)
