---
title: Konfigurera AEM Dispatcher
description: Lär dig konfigurera Dispatcher. Lär dig mer om stöd för IPv4 och IPv6, konfigurationsfiler, miljövariabler och om att namnge instansen. Läs om hur du definierar servergrupper, identifierar virtuella värdar och mycket mer.
exl-id: 91159de3-4ccb-43d3-899f-9806265ff132
source-git-commit: a9ef9d7d2fe5c421cd8039579fd84961ea901def
workflow-type: tm+mt
source-wordcount: '8941'
ht-degree: 0%

---

# Konfigurera Dispatcher {#configuring-dispatcher}

>[!NOTE]
>
>Dispatcher-versionerna är oberoende av AEM (Adobe Experience Manager). Du kan ha omdirigerats till den här sidan om du har följt en länk till Dispatcher-dokumentationen. Länken var inbäddad i dokumentationen för en tidigare version av AEM.

I följande avsnitt beskrivs hur du konfigurerar olika aspekter av Dispatcher.

## Stöd för IPv4 och IPv6 {#support-for-ipv-and-ipv}

Alla element i AEM och Dispatcher kan installeras i både IPv4- och IPv6-nätverk. Se [IPV4 och IPV6](https://experienceleague.adobe.com/sv/docs/experience-manager-65/content/implementing/deploying/introduction/technical-requirements#ipv-and-ipv).

## Dispatcher konfigurationsfiler {#dispatcher-configuration-files}

Som standard lagras Dispatcher-konfigurationen i textfilen `dispatcher.any`, men du kan ändra namn och plats för filen under installationen.

Konfigurationsfilen innehåller en serie egenskaper med ett eller flera värden som styr Dispatcher beteende:

* Egenskapsnamnen har ett snedstreck (`/`) som prefix.
* Egenskaper med flera värden omsluter underordnade objekt med klammerparenteser `{ }`.

En exempelkonfiguration är strukturerad på följande sätt:

```xml
# name of the dispatcher
/name "internet-server"

# each farm configures a set off (loadbalanced) renders
/farms
 {
  # first farm entry (label is not important, just for your convenience)
   /website
     {  
     /clientheaders
       {
       # List of headers that are passed on
       }
     /virtualhosts
       {
       # List of URLs for this Web site
       }
     /sessionmanagement
       {
       # settings for user authentification
       }
     /renders
       {
       # List of AEM instances that render the documents
       }
     /filter
       {
       # List of filters
       }
     /vanity_urls
       {
       # List of vanity URLs
       }
     /cache
       {
       # Cache configuration
       /rules
         {
         # List of cachable documents
         }
       /invalidate
         {
         # List of auto-invalidated documents
         }
       }
     /statistics
       {
       /categories
         {
         # The document categories that are used for load balancing estimates
         }
       }
     /stickyConnectionsFor "/myFolder"
     /health_check
       {
       # Page gets contacted when an instance returns a 500
       }
     /retryDelay "1"
     /numberOfRetries "5"
     /unavailablePenalty "1"
     /failover "1"
     }
 }
```

Du kan inkludera andra filer som bidrar till konfigurationen:

* Om konfigurationsfilen är stor kan du dela upp den i flera mindre filer (som är enklare att hantera) och inkludera alla.
* Inkludera filer som genereras automatiskt.

Om du till exempel vill ta med filen myFarm.any i konfigurationen `/farms` använder du följande kod:

```xml
/farms
  {
  $include "myFarm.any"
  }
```

Använd asterisken (`*`) som jokertecken om du vill ange ett intervall med filer som ska inkluderas.

Om filerna `farm_1.any` till `farm_5.any` till exempel innehåller konfigurationen av grupperna ett till fem kan du inkludera dem så här:

```xml
/farms
  {
  $include "farm_*.any"
  }
```

## Använda miljövariabler {#using-environment-variables}

Du kan använda miljövariabler i egenskaper med strängvärden i dispatcher.any-filen i stället för att hårdkoda värdena. Använd formatet `${variable_name}` om du vill inkludera värdet för en miljövariabel.

Om till exempel filen dispatcher.any finns i samma katalog som cachekatalogen kan följande värde för egenskapen [docroot](#specifying-the-cache-directory) användas:

```xml
/docroot "${PWD}/cache"
```

Om du till exempel skapar en miljövariabel med namnet `PUBLISH_IP` som lagrar värdnamnet för AEM-publiceringsinstansen kan följande konfiguration av egenskapen [`/renders`](#defining-page-renderers-renders) användas:

```xml
/renders {
  /0001 {
    /hostname "${PUBLISH_IP}"
    /port "8443"
  }
}
```

## Namnge Dispatcher-instansen {#naming-the-dispatcher-instance-name}

Använd egenskapen `/name` för att ange ett unikt namn som identifierar din Dispatcher-instans. Egenskapen `/name` är en egenskap på den översta nivån i konfigurationsstrukturen.

## Definiera grupper {#defining-farms-farms}

Egenskapen `/farms` definierar en eller flera uppsättningar Dispatcher-beteenden, där varje uppsättning är associerad med olika webbplatser eller URL-adresser. Egenskapen `/farms` kan innehålla en eller flera grupper:

* Använd en enda servergrupp när du vill att Dispatcher ska hantera alla dina webbsidor eller webbplatser på samma sätt.
* Skapa flera servergrupper när olika delar av webbplatsen eller olika webbplatser kräver olika Dispatcher-beteenden.

Egenskapen `/farms` är en egenskap på den översta nivån i konfigurationsstrukturen. Om du vill definiera en servergrupp lägger du till en underordnad egenskap i egenskapen `/farms`. Använd ett egenskapsnamn som unikt identifierar servergruppen i Dispatcher-instansen.

Egenskapen `/farmname` har flera värden och innehåller andra egenskaper som definierar Dispatcher-beteendet:

* URL:erna för de sidor som servergruppen gäller för.
* En eller flera tjänst-URL:er (vanligtvis AEM publiceringsinstanser) som används för att återge dokument.
* Statistik som ska användas för belastningsutjämning av flera dokumentåtergivare.
* Flera andra beteenden, till exempel vilka filer som ska cachelagras och var de ska cachelagras.

Värdet kan innehålla alla alfanumeriska tecken (a-z, 0-9). I följande exempel visas skelettdefinitionen för två grupper som heter `/daycom` och `/docsdaycom`:

```xml
#name of dispatcher
/name "day sites"

#farms section defines a list of farms or sites
/farms
{
   /daycom
   {
       ...
   }
   /docdaycom
   {
      ...
   }
}
```

>[!NOTE]
>
>Om du använder mer än en renderingsgrupp utvärderas listan längst ned. Det här flödet är relevant när du definierar [virtuella värdar](#identifying-virtual-hosts-virtualhosts) för dina webbplatser.

Varje gruppegenskap kan innehålla följande underordnade egenskaper:

| Egenskapsnamn | Beskrivning |
|--- |--- |
| [/hemsida ](#specify-a-default-page-iis-only-homepage) | Standardstartsida (valfritt) (endast IIS) |
| [/clientheaders](#specifying-the-http-headers-to-pass-through-clientheaders) | Rubrikerna från klientens HTTP-begäran som ska skickas. |
| [/virtualhosts](#identifying-virtual-hosts-virtualhosts) | Den här servergruppens virtuella värdar. |
| [/sessionmanagement](#enabling-secure-sessions-sessionmanagement) | Stöd för sessionshantering och autentisering. |
| [/renders](#defining-page-renderers-renders) | Servrarna som innehåller återgivna sidor (vanligtvis AEM publiceringsinstanser). |
| [/filter](#configuring-access-to-content-filter) | Definierar de URL:er som Dispatcher aktiverar åtkomst till. |
| [/vanity_urls](#enabling-access-to-vanity-urls-vanity-urls) | Konfigurerar åtkomst till mål-URL:er. |
| [/spridateSyndPost](#forwarding-syndication-requests-propagatesyndpost) | Stöd för vidarebefordran av syndikeringsbegäranden. |
| [/cache](#configuring-the-dispatcher-cache-cache) | Konfigurerar cachelagring. |
| [/statistik](#configuring-load-balancing-statistics) | Definiera statistikkategorier för belastningsutjämningsberäkningar. |
| [/stickyConnectionsFor](#identifying-a-sticky-connection-folder-stickyconnectionsfor) | Mappen som innehåller anteckningsdokument. |
| [/health_check](#specifying-a-health-check-page) | Den URL som ska användas för att fastställa servertillgängligheten. |
| [/retryDelay](#specifying-the-page-retry-delay) | Fördröjningen innan ett nytt försök att ansluta misslyckades. |
| [/unavailablePenalty](#reflecting-server-unavailability-in-dispatcher-statistics) | Påföljder som påverkar statistik för belastningsutjämningsberäkningar. |
| [/failover](#using-the-failover-mechanism) | Skicka om begäranden till olika återgivningar när den ursprungliga begäran misslyckas. |
| [/auth_checker](permissions-cache.md) | Information om behörighetskänslig cachelagring finns i [Cachelagra skyddat innehåll](permissions-cache.md). |

## Ange en standardsida (endast IIS) - `/homepage` {#specify-a-default-page-iis-only-homepage}

>[!CAUTION]
>
>Parametern `/homepage` (endast IIS) fungerar inte längre. Använd i stället [IIS URL Rewrite Module](https://learn.microsoft.com/en-us/iis/extensions/url-rewrite-module/using-the-url-rewrite-module).
>
>Om du använder Apache bör du använda modulen `mod_rewrite`. Information om `mod_rewrite` finns i dokumentationen för Apache-webbplatsen (till exempel [Apache 2.4](https://httpd.apache.org/docs/current/mod/mod_rewrite.html)). När du använder `mod_rewrite` bör du använda flaggan genomgång|PT (skicka till nästa hanterare) för att tvinga omskrivningsmotorn att ange `uri`-fältet i den interna `request_rec`-strukturen till värdet för fältet `filename`.

<!-- 

Comment Type: draft

<p>The optional /homepage parameter specifies the page that Dispatcher returns when a client requests an undeterminable page or file.</p> 
<p>Typically this situation occurs when a user specifies an URL for which neither IIS or AEM provides an automatic redirection target. For example, if the AEM render instance is shut down after the content is cached, the content redirect URL is unavailable.</p> 
<p>The following example configuration displays the <span class="code">index.html</span> page in such circumstances:</p>

 -->

<!-- 

Comment Type: draft

<codeblock gutter="true" class="syntax xml">
  /homepage&nbsp;"/index.html" 
</codeblock>

 -->

<!-- 

Comment Type: draft

<p>The <span class="code">/homepage</span> section is located inside the <span class="code">/farms</span> section, for example:<br /> </p>

 -->

<!-- 

Comment Type: draft

<codeblock gutter="true" class="syntax xml">
  #name&nbsp;of&nbsp;dispatcher!!discoiqbr!!/name&nbsp;"day&nbsp;sites"!!discoiqbr!!!!discoiqbr!!#farms&nbsp;section&nbsp;defines&nbsp;a&nbsp;list&nbsp;of&nbsp;farms&nbsp;or&nbsp;sites!!discoiqbr!!/farms!!discoiqbr!!{!!discoiqbr!!&nbsp;&nbsp;&nbsp;/daycom!!discoiqbr!!&nbsp;&nbsp;&nbsp;{!!discoiqbr!!&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/homepage&nbsp;"/index.html"!!discoiqbr!!&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;...!!discoiqbr!!&nbsp;&nbsp;&nbsp;}!!discoiqbr!!&nbsp;&nbsp;&nbsp;/docdaycom!!discoiqbr!!&nbsp;&nbsp;&nbsp;{!!discoiqbr!!&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;...!!discoiqbr!!&nbsp;&nbsp;&nbsp;}!!discoiqbr!!} 
</codeblock>

 -->

## Ange vilka HTTP-huvuden som ska passera igenom {#specifying-the-http-headers-to-pass-through-clientheaders}

Egenskapen `/clientheaders` definierar en lista med HTTP-rubriker som Dispatcher skickar från klientens HTTP-begäran till återgivaren (AEM-instansen).

Som standard vidarebefordrar Dispatcher standardrubrikerna för HTTP till AEM-instansen. I vissa fall kanske du vill vidarebefordra ytterligare rubriker eller ta bort specifika rubriker:

* Lägg till rubriker, t.ex. anpassade rubriker, som din AEM-instans förväntar sig i HTTP-begäran.
* Ta bort rubriker, t.ex. autentiseringsrubriker som bara är relevanta för webbservern.

Om du anpassar uppsättningen rubriker som ska skickas måste du ange en uttömmande lista över rubriker, inklusive de rubriker som normalt inkluderas som standard.

En Dispatcher-instans som hanterar sidaktiveringsbegäranden för publiceringsinstanser kräver till exempel rubriken `PATH` i avsnittet `/clientheaders`. Rubriken `PATH` möjliggör kommunikation mellan replikeringsagenten och Dispatcher.

Följande kod är ett exempel på konfiguration för `/clientheaders`:

```shell
/clientheaders
  {
  "CSRF-Token"
  "X-Forwarded-Proto"
  "referer"
  "user-agent"
  "authorization"
  "from"
  "content-type"
  "content-length"
  "accept-charset"
  "accept-encoding"
  "accept-language"
  "accept"
  "host"
  "max-forwards"
  "proxy-authorization"
  "proxy-connection"
  "range"
  "cookie"
  "cq-action"
  "cq-handle"
  "handle"
  "action"
  "cqstats"
  "depth"
  "translate"
  "expires"
  "date"
  "dav"
  "ms-author-via"
  "if"
  "lock-token"
  "x-expected-entity-length"
  "destination"
  "PATH"
  }
```

## Identifiera virtuella värdar {#identifying-virtual-hosts-virtualhosts}

Egenskapen `/virtualhosts` definierar en lista över alla värdnamn och URI-kombinationer som Dispatcher accepterar för den här servergruppen. Du kan använda asterisken (`*`) som jokertecken. Värden för egenskapen /`virtualhosts` har följande format:

```xml
[scheme]host[uri][*]
```

* `scheme`: (Valfritt) Antingen `https://` eller `https://.`
* `host`: Värddatorns namn eller IP-adress och portnumret om det behövs. (Se [https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23))
* `uri`: (Valfritt) Sökvägen till resurserna.

I följande exempel hanteras begäranden för domänerna `.com` och `.ch` för myCompany och alla domäner för mySubDivision:

```xml
   /virtualhosts
    {
    "www.myCompany.com"
    "www.myCompany.ch"
    "www.mySubDivison.*"
    }
```

Följande konfiguration hanterar *alla*-begäranden:

```xml
   /virtualhosts
    {
    "*"
    }
```

### Matchar den virtuella värden {#resolving-the-virtual-host}

När Dispatcher tar emot en HTTP- eller HTTPS-begäran hittar den det virtuella värdvärdet som bäst matchar begärans `host,` `uri`- och `scheme`-huvuden. Dispatcher utvärderar värdena i egenskaperna `virtualhosts` i följande ordning:

* Dispatcher börjar längst ned i servergruppen och fortsätter uppåt i filen dispatcher.any.
* För varje servergrupp börjar Dispatcher med det översta värdet i egenskapen `virtualhosts` och går nedåt i listan över värden.

Dispatcher hittar det mest matchande virtuella värdvärdet på följande sätt:

* Den första påträffade virtuella värden som matchar alla tre av `host`, `scheme` och `uri` för begäran används.
* Om inga `virtualhosts`-värden har `scheme` och `uri` delar som matchar både `scheme` och `uri` i begäran, används den första påträffade virtuella värden som matchar `host` i begäran.
* Om inga `virtualhosts`-värden har en värddel som matchar värddatorn för begäran, används den översta virtuella värddatorn för den översta servergruppen.

Därför bör du placera din virtuella standardvärd högst upp i egenskapen `virtualhosts`. Placera den i den översta servergruppen för din `dispatcher.any`-fil.

### Exempel på virtuell värdupplösning {#example-virtual-host-resolution}

Följande exempel representerar ett utdrag från en `dispatcher.any`-fil som definierar två Dispatcher-grupper, och varje servergrupp definierar en `virtualhosts` -egenskap.

```xml
/farms
  {
  /myProducts
    {
    /virtualhosts
      {
      "www.mycompany.com/products/*"
      }
    /renders
      {
      /hostname "server1.myCompany.com"
      /port "80"
      }
    }
  /myCompany
    {
    /virtualhosts
      {
      "www.mycompany.com"
      }
    /renders
      {
      /hostname "server2.myCompany.com"
      /port "80"
      }
    }
  }
```

I följande tabell visas de virtuella värdarna som matchas för de angivna HTTP-begäranden:

| Begär URL | Löst virtuellt värdsystem |
|---|---|
| `https://www.mycompany.com/products/gloves.html` | `www.mycompany.com/products/` |
| `https://www.mycompany.com/about.html` | `www.mycompany.com` |

## Aktiverar säkra sessioner - `/sessionmanagement` {#enabling-secure-sessions-sessionmanagement}

>[!CAUTION]
>
>`/allowAuthorized` Ange som `"0"` i avsnittet `/cache` om du vill aktivera den här funktionen. Så som anges i avsnittet [Cachelagring när autentisering används](#caching-when-authentication-is-used), cachelagras **inte** när du anger `/allowAuthorized 0 ` begäranden som innehåller autentiseringsinformation. Om behörighetskänslig cachelagring krävs, se sidan [Cachelagra skyddat innehåll](https://experienceleague.adobe.com/sv/docs/experience-manager-dispatcher/using/configuring/permissions-cache).

Skapa en säker session för åtkomst till renderingsgruppen så att användarna måste logga in för att komma åt alla sidor i gruppen. När användaren har loggat in kan han/hon komma åt sidor i servergruppen. Mer information om hur du använder den här funktionen med CUG:er finns i [Skapa en stängd användargrupp](https://experienceleague.adobe.com/sv/docs/experience-manager-65/content/security/cug#creating-the-user-group-to-be-used). Se även Dispatcher [checklista för säkerhet](/help/using/security-checklist.md) innan du publicerar.

Egenskapen `/sessionmanagement` är en underegenskap till `/farms`.

>[!CAUTION]
>
>Om olika åtkomstkrav gäller för olika delar av webbplatsen måste du definiera flera grupper.

**/sessionmanagement** har flera underparametrar:

**/katalog** (obligatoriskt)

Katalogen som lagrar sessionsinformationen. Om katalogen inte finns skapas den.

>[!CAUTION]
>
> När du konfigurerar katalogunderparametern pekar **inte** till rotmappen (`/directory "/"`) eftersom det kan orsaka allvarliga problem. Ange alltid sökvägen till den mapp som lagrar sessionsinformationen. Till exempel:

```xml
/sessionmanagement
  {
  /directory "/usr/local/apache/.sessions"
  }
```

**/encode** (valfritt)

Hur sessionsinformationen kodas. Använd `md5` för kryptering med md5-algoritmen eller `hex` för hexadecimal kodning. Om du krypterar sessionsdata kan en användare med åtkomst till filsystemet inte läsa sessionsinnehållet. Standardvärdet är `md5`.

**/header** (valfritt)

Namnet på HTTP-huvudet eller cookien som lagrar auktoriseringsinformationen. Om du lagrar informationen i http-huvudet använder du `HTTP:<header-name>`. Om du vill lagra informationen i en cookie använder du `Cookie:<header-name>`. Om du inte anger något värde används `HTTP:authorization`.

**/timeout** (valfritt)

Antalet sekunder tills sessionen tar slut efter att den har använts sist. Om `"800"` inte anges används, så sessionen tar lite tid över 13 minuter efter användarens senaste begäran.

Ett exempel på konfiguration ser ut så här:

```xml
/sessionmanagement
  {
  /directory "/usr/local/apache/.sessions"
  /encode "md5"
  /header "HTTP:authorization"
  /timeout "800"
  }
```

## Definiera sidåtergivare {#defining-page-renderers-renders}

Egenskapen `/renders` definierar den URL som Dispatcher skickar begäranden till för att återge ett dokument. I följande exempel `/renders` identifieras en enda AEM-instans för återgivning:

```xml
/renders
  {
    /myRenderer
      {
      # hostname or IP of the renderer
      /hostname "aem.myCompany.com"
      # port of the renderer
      /port "4503"
      # connection timeout in milliseconds, "0" (default) waits indefinitely
      /timeout "0"
      }
  }
```

I följande exempelavsnitt `/renders` identifieras en AEM-instans som körs på samma dator som Dispatcher:

```xml
/renders
  {
    /myRenderer
     {
     /hostname "127.0.0.1"
     /port "4503"
     }
  }
```

I följande exempel `/renders` distribueras återgivningsbegäranden lika mellan två AEM-instanser:

```xml
/renders
  {
    /myFirstRenderer
      {
      /hostname "aem.myCompany.com"
      /port "4503"
      }
    /mySecondRenderer
      {
      /hostname "127.0.0.1"
      /port "4503"
      }
  }
```

### Återgivningsalternativ {#renders-options}

**/timeout**

Anger timeout för anslutningen som ansluter till AEM-instansen i millisekunder. Standardvärdet är `"0"`, vilket gör att Dispatcher väntar oändligt.

**/receiveTimeout**

Anger tiden i millisekunder som ett svar får ta. Standardvärdet är `"600000"`, vilket gör att Dispatcher väntar i 10 minuter. En inställning på `"0"` tar bort tidsgränsen.

Om tidsgränsen nås när svarshuvuden tolkas returneras HTTP-statusen 504 (Felaktig gateway). Om tidsgränsen nås när svarstexten läses, returnerar Dispatcher det ofullständiga svaret till klienten. Eventuella cachelagrade filer som kan ha skrivits tas också bort.

**/ipv4**

Anger om Dispatcher använder funktionen `getaddrinfo` (för IPv6) eller funktionen `gethostbyname` (för IPv4) för att hämta återgivningens IP-adress. Värdet 0 gör att `getaddrinfo` används. Värdet `1` gör att `gethostbyname` används. Standardvärdet är `0`.

Funktionen `getaddrinfo` returnerar en lista med IP-adresser. Dispatcher itererar listan över adresser tills en TCP/IP-anslutning upprättas. Därför är egenskapen `ipv4` viktig när återgivningsvärdnamnet är associerat med flera IP-adresser. Värden returnerar dessutom, som svar på funktionen `getaddrinfo`, en lista över IP-adresser som alltid finns i samma ordning. I det här fallet bör du använda funktionen `gethostbyname` så att IP-adressen som Dispatcher ansluter till är slumpmässig.

Amazon Elastic Load Balancing (ELB) är en tjänst som svarar på getaddrinfo med en lista över IP-adresser som kan vara i samma ordning.

**/säker**

Om egenskapen `/secure` har värdet `"1"` använder Dispatcher HTTPS för att kommunicera med AEM-instansen. Mer information finns i [Konfigurera Dispatcher att använda SSL](dispatcher-ssl.md#configuring-dispatcher-to-use-ssl).

**/always-resolve**

Med Dispatcher version **4.1.6** kan du konfigurera egenskapen `/always-resolve` enligt följande:

* När värdet är `"1"` tolkas värdnamnet på varje begäran (Dispatcher cachelagrar aldrig någon IP-adress). Det kan uppstå en liten prestandapåverkan på grund av det ytterligare anrop som krävs för att få värdinformation för varje begäran.
* Om egenskapen inte anges cache-lagras IP-adressen som standard.

Den här egenskapen kan även användas om du stöter på problem med dynamisk IP-upplösning, vilket visas i följande exempel:

```xml
/renders {
  /0001 {
     /hostname "host-name-here"
     /port "4502"
     /ipv4 "1"
     /always-resolve "1"
     }
  }
```

## Konfigurera åtkomst till innehåll {#configuring-access-to-content-filter}

Använd avsnittet `/filter` för att ange de HTTP-begäranden som Dispatcher godkänner. Alla andra begäranden skickas tillbaka till webbservern med felkoden 404 (sidan hittades inte). Om det inte finns något `/filter`-avsnitt accepteras alla begäranden.

**Obs!** Begäranden för [statfile](#naming-the-statfile) nekas alltid.

>[!CAUTION]
>
>Se [Dispatcher Security Checklist](security-checklist.md) för mer information om begränsningar av åtkomst med AEM Dispatcher. Läs även [AEM Security Checklist](https://experienceleague.adobe.com/sv/docs/experience-manager-65/content/security/security-checklist#security) om du vill ha mer säkerhetsinformation om din AEM-installation.

Avsnittet `/filter` består av en serie regler som antingen nekar eller tillåter åtkomst till innehåll enligt mönster i begärandoradsdelen av HTTP-begäran. Använd en tillåtelselista-strategi för ditt `/filter`-avsnitt:

* Förhindra åtkomst till allt.
* Tillåt åtkomst till innehåll efter behov.

>[!NOTE]
>
>Rensa cacheminnet när filterreglerna ändras.

### Definiera ett filter {#defining-a-filter}

Varje objekt i avsnittet `/filter` innehåller en typ och ett mönster som matchas med ett visst element i förfrågningsraden eller hela förfrågningsraden. Varje filter kan innehålla följande objekt:

* **Typ**: `/type` anger om åtkomst ska beviljas eller nekas för begäranden som matchar mönstret. Värdet kan vara antingen `allow` eller `deny`.

* **Element i begäranderaden:** Inkludera `/method`, `/url`, `/query` eller `/protocol`. Ta även med ett mönster för filtreringsbegäranden. Filtrera dem efter specifika delar av begärandoradsdelen i HTTP-begäran. Filtrering för element på begärandraden (i stället för på hela begärandraden) är den föredragna filtermetoden.

* **Avancerade element i begärandoraden:** Från och med Dispatcher 4.2.0 finns fyra nya filterelement att använda. De nya elementen är `/path`, `/selectors`, `/extension` och `/suffix`. Inkludera ett eller flera av dessa objekt för ytterligare kontroll av URL-mönster.

>[!NOTE]
>
>Mer information om vilken del av förfrågningsraden som var och en av dessa element refererar till finns på wiki-sidan [Sling URL Decomposition](https://sling.apache.org/documentation/the-sling-engine/url-decomposition.html) .

* **globegenskapen**: Egenskapen `/glob` används för att matcha hela förfrågningsraden i HTTP-begäran.

>[!CAUTION]
>
>Filtrering med glober är föråldrat i Dispatcher. Därför bör du undvika att använda globala ikoner i `/filter`-avsnitten eftersom det kan leda till säkerhetsproblem. I stället för:
>
>`/glob "* *.css *"`
>
>Använd
>
>`/url "*.css"`

#### Begärandelen av HTTP-begäranden {#the-request-line-part-of-http-requests}

HTTP/1.1 definierar [request-line](https://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html) enligt följande:

`Method Request-URI HTTP-Version<CRLF>`

Tecknen `<CRLF>` representerar en vagnretur följt av en radmatning. Följande exempel är den frågerad som tas emot när en klient begär sidan amerikansk-engelska på WKND-webbplatsen:

`GET /content/wknd/us/en.html HTTP.1.1<CRLF>`

Dina mönster måste ta hänsyn till blankstegstecknen på raden med begäran och tecknen `<CRLF>`.

#### Dubbla citattecken jämfört med enkla citattecken {#double-quotes-vs-single-quotes}

När du skapar filterregler använder du citattecken `"pattern"` för enkla mönster. Om du använder Dispatcher 4.2.0 eller senare och mönstret innehåller ett reguljärt uttryck, måste du omsluta regex-mönstret `'(pattern1|pattern2)'` med enkla citattecken.

#### Reguljära uttryck {#regular-expressions}

I Dispatcher-versioner senare än 4.2.0 kan du inkludera utökade reguljära uttryck för POSIX i dina filtermönster.

#### Felsöka filter {#troubleshooting-filters}

Om dina filter inte aktiveras på rätt sätt aktiverar du [Trace Logging](#trace-logging) på Dispatcher så att du kan se vilket filter som fångar upp begäran.

#### Exempelfilter: Neka alla {#example-filter-deny-all}

Följande exempelfilteravsnitt gör att Dispatcher nekar begäranden för alla filer. Neka åtkomst till alla filer och tillåt sedan åtkomst till specifika områden.

```xml
/0001  { /type "deny" /url "*"  }
```

Begäranden till ett explicit nekat område resulterar i att 404-felkoden (sidan hittades inte) returneras.

#### Exempelfilter: Neka åtkomst till specifika områden {#example-filter-deny-access-to-specific-areas}

Med filter kan du också neka åtkomst till olika element, till exempel ASP-sidor och känsliga områden i en publiceringsinstans. Följande filter nekar åtkomst till ASP-sidor:

```xml
/0002  { /type "deny" /url "*.asp"  }
```

#### Exempelfilter: Aktivera POST-begäranden {#example-filter-enable-post-requests}

Följande exempelfilter tillåter att formulärdata skickas med POST-metoden:

```xml
/filter {
    /0001  { /glob "*" /type "deny" }
    /0002 { /type "allow" /method "POST" /url "/content/[.]*.form.html" }
}
```

#### Exempelfilter: Tillåt åtkomst till arbetsflödeskonsolen {#example-filter-allow-access-to-the-workflow-console}

I följande exempel visas ett filter som används för att ge extern åtkomst till arbetsflödeskonsolen:

```xml
/filter {
    /0001  { /glob "*" /type "deny" }
    /0002  {  /type "allow"  /url "/libs/cq/workflow/content/console*"  }
}
```

Om publiceringsinstansen använder en webbprogramkontext (till exempel publicera) kan den också läggas till i filterdefinitionen.

```xml
/0003   { /type "deny"  /url "/publish/libs/cq/workflow/content/console/archive*"  }
```

Om du måste komma åt enstaka sidor inom det begränsade området kan du tillåta åtkomst till dem. Om du till exempel vill tillåta åtkomst till fliken Arkiv i arbetsflödeskonsolen lägger du till följande avsnitt:

```xml
/0004  { /type "allow"  /url "/libs/cq/workflow/content/console/archive*"   }
```

>[!NOTE]
>
>När flera filtermönster används i en begäran gäller det senast använda filtermönstret.

#### Exempelfilter: Använda reguljära uttryck {#example-filter-using-regular-expressions}

Det här filtret aktiverar tillägg i icke-offentliga innehållskataloger med hjälp av ett reguljärt uttryck, som definieras här mellan enkla citattecken:

```xml
/005  {  /type "allow" /extension '(css|gif|ico|js|png|swf|jpe?g)' }
```

#### Exempelfilter: Filtrera extra element för en URL för en begäran {#example-filter-filter-additional-elements-of-a-request-url}

Nedan visas ett regelexempel som blockerar innehåll som hämtas från sökvägen `/content` och dess underträd, med filter för sökväg, väljare och tillägg:

```xml
/006 {
        /type "deny"
        /path "/content/*"
        /selectors '(feed|rss|pages|languages|blueprint|infinity|tidy|sysview|docview|query|jcr:content|_jcr_content|search|childrenlist|ext|assets|assetsearch|[0-9-]+)'
        /extension '(json|xml|html|feed))'
        }
```

### Exempel på `/filter`-avsnitt {#example-filter-section}

Begränsa den externa åtkomsten så mycket som möjligt när du konfigurerar Dispatcher. Följande exempel ger minimal åtkomst för externa besökare:

* `/content`
* olika innehåll, t.ex. design och klientbibliotek. Till exempel:

   * `/etc/designs/default*`
   * `/etc/designs/mydesign*`

När du har skapat filter [testar du sidåtkomst](#testing-dispatcher-security) för att se till att din AEM-instans är säker.

Följande `/filter`-avsnitt i `dispatcher.any`-filen kan användas som bas i din [Dispatcher-konfigurationsfil.](#dispatcher-configuration-files)

Det här exemplet baseras på standardkonfigurationsfilen som medföljer Dispatcher och är avsedd som exempel för användning i en produktionsmiljö. Objekt som föregås av `#` inaktiveras (kommenteras bort). Du bör vara försiktig om du bestämmer dig för att aktivera något av dessa objekt (genom att ta bort `#` på den raden). Om du gör det kan det påverka säkerheten.

Neka åtkomst till allt och tillåt sedan åtkomst till specifika (begränsade) element:

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-26T04:32:37.986-0400

<p>We should mention the config files that are shipped with the dispatcher distribution and only give a few examples here. This aims to avoid confusion and reduce content maintenance.<br /> </p>

 -->

```xml
  /filter
      {
      # Deny everything first and then allow specific entries
      /0001  { /type "deny" /url "*"  }

      # Open consoles
#     /0011 { /type "allow" /url "/admin/*"  }  # allow servlet engine admin
#     /0012 { /type "allow" /url "/crx/*"    }  # allow content repository
#     /0013 { /type "allow" /url "/system/*" }  # allow OSGi console

      # Allow non-public content directories
#     /0021 { /type "allow" /url "/apps/*"   }  # allow apps access
#     /0022 { /type "allow" /url "/bin/*"    }
      /0023 { /type "allow" /url "/content*" }  # disable this rule to allow mapped content only

#     /0024 { /type "allow" /url "/libs/*"   }
#     /0025 { /type "deny"  /url "/libs/shindig/proxy*" } # if you enable /libs close access to proxy

#     /0026 { /type "allow" /url "/home/*"   }
#     /0027 { /type "allow" /url "/tmp/*"    }
#     /0028 { /type "allow" /url "/var/*"    }

      # Enable extensions in non-public content directories, using a regular expression
      /0041
        {
        /type "allow"
        /extension '(css|gif|ico|js|png|swf|jpe?g)'
        }

      # Enable features
      /0062 { /type "allow" /url "/libs/cq/personalization/*"  }  # enable personalization

      # Deny content grabbing, on all accessible pages, using regular expressions
      /0081
        {
        /type "deny"
        /selectors '((sys|doc)view|query|[0-9-]+)'
        /extension '(json|xml)'
        }
      # Deny content grabbing for /content and its subtree
      /0082
        {
        /type "deny"
        /path "/content/*"
        /selectors '(feed|rss|pages|languages|blueprint|infinity|tidy)'
        /extension '(json|xml|html)'
        }

#     /0087 { /type "allow" /method "GET" /extension 'json' "*.1.json" }  # allow one-level json requests
}
```

>[!NOTE]
>
>När du använder Apache utformar du dina filter-URL-mönster enligt egenskapen DispatcherUseProcsedURL i Dispatcher-modulen. (Se [Apache-webbserver - Konfigurera Apache-webbservern för Dispatcher](dispatcher-install.md##apache-web-server-configure-apache-web-server-for-dispatcher).)

<!----
>[!NOTE]
>
>Filters `0030` and `0031` regarding Dynamic Media are applicable to AEM 6.0 and higher. -->

Tänk på följande om du väljer att utöka åtkomsten:

* Inaktivera extern åtkomst till `/admin` om du använder CQ version 5.4 eller tidigare.

* Du måste vara försiktig när du tillåter åtkomst till filer i `/libs`. Åtkomst bör tillåtas på individuell basis.
* Neka åtkomst till replikeringskonfigurationen så att den inte kan ses:

   * `/etc/replication.xml*`
   * `/etc/replication.infinity.json*`

* Neka åtkomst till Google Gadgets omvänd proxy:

   * `/libs/opensocial/proxy*`

Beroende på installationen kan det finnas fler resurser under `/libs`, `/apps` eller någon annanstans som måste göras tillgängliga. Du kan använda filen `access.log` som en metod för att avgöra vilka resurser som ska användas externt.

>[!CAUTION]
>
>Åtkomst till konsoler och kataloger kan utgöra en säkerhetsrisk för produktionsmiljöer. Om du inte har en explicit motivering ska de förbli inaktiverade (kommenterade ut).

>[!CAUTION]
>
>Om du [använder rapporter i en publiceringsmiljö](https://experienceleague.adobe.com/sv/docs/experience-manager-65/content/sites/administering/operations/reporting#using-reports-in-a-publish-environment) bör du konfigurera Dispatcher att neka åtkomst till `/etc/reports` för externa besökare.

### Begränsa frågesträngar {#restricting-query-strings}

Sedan Dispatcher version 4.1.5 använder du avsnittet `/filter` för att begränsa frågesträngar. Vi rekommenderar att du uttryckligen tillåter frågesträngar och exkluderar generiska reserveringar via `allow` filterelement.

En enskild post kan ha antingen `glob` eller en kombination av `method`, `url`, `query` och `version`, men inte båda. I följande exempel tillåts frågesträngen `a=*` och alla andra frågesträngar för URL:er som tolkas till noden `/etc` nekas:

```xml
/filter {
 /0001 { /type "deny" /method "POST" /url "/etc/*" }
 /0002 { /type "allow" /method "GET" /url "/etc/*" /query "a=*" }
}
```

>[!NOTE]
>
>Om en regel innehåller en `/query` matchar den bara begäranden som innehåller en frågesträng och matchar det angivna frågemönstret.
>
>Om begäranden till `/etc` som inte har någon frågesträng också ska tillåtas i ovanstående exempel krävs följande regler:
>

```xml
/filter {  
>/0001 { /type "deny" /method "*" /url "/path/*" }  
>/0002 { /type "allow" /method "GET" /url "/path/*" }  
>/0003 { /type "deny" /method "GET" /url "/path/*" /query "*" }  
>/0004 { /type "allow" /method "GET" /url "/path/*" /query "a=*" }  
}  
```

### Testa Dispatcher Security {#testing-dispatcher-security}

Dispatcher-filter bör blockera åtkomsten till följande sidor och skript på AEM publiceringsinstanser. Använd en webbläsare för att försöka öppna följande sidor som en besökare skulle göra och verifiera att koden 404 returneras. Justera filtren om du får andra resultat.

Du bör se normal sidåtergivning för `/content/add_valid_page.html?debug=layout`.

* `/admin`
* `/system/console`
* `/dav/crx.default`
* `/crx`
* `/bin/crxde/logs`
* `/jcr:system/jcr:versionStorage.json`
* `/_jcr_system/_jcr_versionStorage.json`
* `/libs/wcm/core/content/siteadmin.html`
* `/libs/collab/core/content/admin.html`
* `/libs/cq/ui/content/dumplibs.html`
* `/var/linkchecker.html`
* `/etc/linkchecker.html`
* `/home/users/a/admin/profile.json`
* `/home/users/a/admin/profile.xml`
* `/libs/cq/core/content/login.json`
* `/content/../libs/foundation/components/text/text.jsp`
* `/content/.{.}/libs/foundation/components/text/text.jsp`
* `/apps/sling/config/org.apache.felix.webconsole.internal.servlet.OsgiManager.config/jcr%3acontent/jcr%3adata`
* `/libs/foundation/components/primary/cq/workflow/components/participants/json.GET.servlet`
* `/content.pages.json`
* `/content.languages.json`
* `/content.blueprint.json`
* `/content.-1.json`
* `/content.10.json`
* `/content.infinity.json`
* `/content.tidy.json`
* `/content.tidy.-1.blubber.json`
* `/content/dam.tidy.-100.json`
* `/content/content/geometrixx.sitemap.txt `
* `/content/add_valid_page.query.json?statement=//*`
* `/content/add_valid_page.qu%65ry.js%6Fn?statement=//*`
* `/content/add_valid_page.query.json?statement=//*[@transportPassword]/(@transportPassword%20|%20@transportUri%20|%20@transportUser)`
* `/content/add_valid_path_to_a_page/_jcr_content.json`
* `/content/add_valid_path_to_a_page/jcr:content.json`
* `/content/add_valid_path_to_a_page/_jcr_content.feed`
* `/content/add_valid_path_to_a_page/jcr:content.feed`
* `/content/add_valid_path_to_a_page/pagename._jcr_content.feed`
* `/content/add_valid_path_to_a_page/pagename.jcr:content.feed`
* `/content/add_valid_path_to_a_page/pagename.docview.xml`
* `/content/add_valid_path_to_a_page/pagename.docview.json`
* `/content/add_valid_path_to_a_page/pagename.sysview.xml`
* `/etc.xml`
* `/content.feed.xml`
* `/content.rss.xml`
* `/content.feed.html`
* `/content/add_valid_page.html?debug=layout`
* `/projects`
* `/tagging`
* `/etc/replication.html`
* `/etc/cloudservices.html`
* `/welcome`

Om du vill avgöra om anonym skrivåtkomst är aktiverat skickar du följande kommando i en terminal eller kommandotolk. Det går inte att skriva data till noden.

`curl -X POST "https://anonymous:anonymous@hostname:port/content/usergenerated/mytestnode"`

Om du vill försöka göra Dispatcher-cachen ogiltig och försäkra dig om att du får ett code 403-svar skickar du följande kommando i en terminal eller kommandotolk:

`curl -H "CQ-Handle: /content" -H "CQ-Path: /content" https://yourhostname/dispatcher/invalidate.cache`

## Aktivera åtkomst till Vanity-URL:er {#enabling-access-to-vanity-urls-vanity-urls}

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (sbroders@adobe.com)
Last Modified Date: 2015-03-25T14:23:05.185-0400

<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">For https://jira.corp.adobe.com/browse/DOC-4812</p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">The "com.adobe.granite.dispatcher.vanityurl.content" package needs to be made public before publishing this contnet.</p>
 -->

Konfigurera Dispatcher för att aktivera åtkomst till tillfälliga URL:er som är konfigurerade för dina AEM-sidor.

När åtkomst till tillfälliga URL:er är aktiverat, anropar Dispatcher regelbundet en tjänst som körs på återgivningsinstansen för att få en lista över tillfälliga URL:er. Dispatcher sparar listan i en lokal fil. När en begäran om en sida nekas på grund av ett filter i avsnittet `/filter`, läser Dispatcher igenom listan med användar-URL:er. Om den nekade URL:en finns med i listan, tillåter Dispatcher åtkomst till fågel-URL:en.

Om du vill aktivera åtkomst till mål-URL:er lägger du till ett `/vanity_urls`-avsnitt i avsnittet `/farms`, som i följande exempel:

```xml
 /vanity_urls {
      /url "/libs/granite/dispatcher/content/vanityUrls.html"
      /file "/tmp/vanity_urls"
      /delay 300
 }
```

Avsnittet `/vanity_urls` innehåller följande egenskaper:

* `/url`: Sökvägen till den överordnade URL-tjänst som körs på återgivningsinstansen. Egenskapens värde måste vara `"/libs/granite/dispatcher/content/vanityUrls.html"`.

* `/file`: Sökvägen till den lokala filen där Dispatcher lagrar listan över huvud-URL:er. Kontrollera att Dispatcher har skrivbehörighet till den här filen.
* `/delay`: (sekunder) Tiden mellan anrop till tjänsten för huvud-URL.

>[!NOTE]
>
>Om din återgivning är en instans av AEM måste du installera paketet [VanityURLS-Components från Software Distribution](https://experience.adobe.com/#/downloads/content/software-distribution/en/aem.html?package=/content/software-distribution/en/details.html/content/dam/aem/public/adobe/packages/granite/vanityurls-components) för att aktivera tjänsten för huvud-URL. (Mer information finns i [Programvarudistribution](https://experienceleague.adobe.com/sv/docs/experience-manager-65/content/sites/administering/contentmanagement/package-manager#software-distribution).)

Använd följande procedur för att aktivera åtkomst till mål-URL:er.

1. Om din renderingstjänst är en AEM-instans installerar du paketet `com.adobe.granite.dispatcher.vanityurl.content` på publiceringsinstansen (se anteckningen ovan).
1. Kontrollera att konfigurationen av [`/filter`](#configuring-access-to-content-filter) nekar URL:en för varje mål-URL som du har konfigurerat för en AEM- eller CQ-sida. Om det behövs lägger du till ett filter som nekar URL-adressen.
1. Lägg till avsnittet `/vanity_urls` nedanför `/farms`.
1. Starta om Apache-webbservern.

Med Dispatcher **version 4.3.6** har en ny `/loadOnStartup`-parameter lagts till. Genom att använda den här parametern kan du konfigurera inläsningen av mål-URL:er vid start enligt följande:

Genom att lägga till `/loadOnStartup 0` (se exemplet nedan) kan du inaktivera inläsningen av mål-URL:er vid start.

```
/vanity_urls {
        /url "/libs/granite/dispatcher/content/vanityUrls.html"
        /file "/tmp/vanity_urls"
        /loadOnStartup 0
        /delay 60
      } 
```

När `/loadOnStartup 1` läser in URL:er för vanity vid start. Tänk på att `/loadOnStartup 1` är det aktuella standardvärdet för den här parametern.

## Vidarebefordrar syndikeringsbegäranden - `/propagateSyndPost` {#forwarding-syndication-requests-propagatesyndpost}

Syndikeringsbegäranden är endast avsedda för Dispatcher, så som standard skickas de inte till återgivaren (till exempel en AEM-instans).

Om det behövs ställer du in egenskapen `/propagateSyndPost` på `"1"` för att vidarebefordra syndikeringsbegäranden till Dispatcher. Om den anges måste du se till att POST-begäranden inte nekas i filteravsnittet.

## Konfigurerar Dispatcher-cachen - `/cache` {#configuring-the-dispatcher-cache-cache}

Avsnittet `/cache` styr hur Dispatcher cachelagrar dokument. Konfigurera flera underegenskaper för att implementera dina cachningsstrategier:

* `/docroot`
* `/statfile`
* `/serveStaleOnError`
* `/allowAuthorized`
* `/rules`
* `/statfileslevel`
* `/invalidate`
* `/invalidateHandler`
* `/allowedClients`
* `/ignoreUrlParams`
* `/headers`
* `/mode`
* `/gracePeriod`
* `/enableTTL`

Ett exempel på cacheavsnitt kan se ut så här:

```xml
/cache
  {
  /docroot "/opt/dispatcher/cache"
  /statfile  "/tmp/dispatcher-website.stat"
  /allowAuthorized "0"

  /rules
    {
    # List of files that are cached
    }

  /invalidate
    {
    # List of files that are auto-invalidated
    }
  }
  
```

>[!NOTE]
>
>Läs [Cachelagring av skyddat innehåll](permissions-cache.md) för behörighetskänslig cachning.

### Ange cachekatalogen {#specifying-the-cache-directory}

Egenskapen `/docroot` identifierar katalogen där cachelagrade filer lagras.

>[!NOTE]
>
>Värdet måste vara samma sökväg som dokumentroten på webbservern så att Dispatcher och webbservern hanterar samma filer.\
>Webbservern ansvarar för att leverera rätt statuskod när cachefilen för Dispatcher används, det är därför det är viktigt att den också kan hitta den.

Om du använder flera grupper måste varje grupp ha en annan dokumentrot.

### Namnge statusfilen {#naming-the-statfile}

Egenskapen `/statfile` identifierar den fil som ska användas som statusfil. Dispatcher använder den här filen för att registrera tidpunkten för den senaste innehållsuppdateringen. Statfile kan vara vilken fil som helst på webbservern.

Statusfilen har inget innehåll. När innehållet uppdateras uppdaterar Dispatcher tidsstämpeln. Standardstatusfilen har namnet `.stat` och lagras i dokumentroten. Dispatcher blockerar åtkomsten till statfile.

>[!NOTE]
>
>Om `/statfileslevel` har konfigurerats ignorerar Dispatcher egenskapen `/statfile` och använder `.stat` som namn.

### Hantera gamla dokument när fel uppstår {#serving-stale-documents-when-errors-occur}

Egenskapen `/serveStaleOnError` styr om Dispatcher returnerar ogiltiga dokument när återgivningsservern returnerar ett fel. Som standard tas det cachelagrade innehållet bort när en lägesfil ändras och det cachelagrade innehållet blir ogiltigt. Den här åtgärden utförs nästa gång den begärs.

Om `/serveStaleOnError` är inställt på `"1"` tar Dispatcher inte bort ogiltigt innehåll från cachen. Det vill säga, om inte återgivningsservern returnerar ett lyckat svar. Ett 502-, 503- eller 504-svar från AEM eller en timeout för anslutningen gör att Dispatcher skickar det inaktuella innehållet och svarar med HTTP-statusen 111 (förnyelsen misslyckades).

### Cachelagring när autentisering används {#caching-when-authentication-is-used}

Egenskapen `/allowAuthorized` styr om begäranden som innehåller någon av följande autentiseringsinformation cachelagras:

* Rubriken `authorization`
* En cookie med namnet `authorization`
* En cookie med namnet `login-token`

Som standard cachelagras inte begäranden som innehåller den här autentiseringsinformationen eftersom autentiseringen inte utförs när ett cachelagrat dokument returneras till klienten. Den här konfigurationen förhindrar att Dispatcher skickar cachelagrade dokument till användare som inte har nödvändiga rättigheter.

Om dina krav tillåter cachelagring av autentiserade dokument anger du `/allowAuthorized` till ett:

`/allowAuthorized "1"`

>[!NOTE]
>
>Om du vill aktivera sessionshantering (med egenskapen `/sessionmanagement`) måste egenskapen `/allowAuthorized` anges till `"0"`.

### Ange vilka dokument som ska cachelagras {#specifying-the-documents-to-cache}

Egenskapen `/rules` styr vilka dokument som cachelagras enligt dokumentsökvägen. Oavsett egenskapen `/rules` cachelagrar Dispatcher aldrig ett dokument under följande omständigheter:

* URI för begäran innehåller ett frågetecken (`?`).
   * Anger en dynamisk sida, till exempel ett sökresultat som inte behöver cachas.
* Filtillägget saknas.
   * Webbservern behöver tillägget för att kunna avgöra dokumenttypen (MIME-typen).
* Autentiseringshuvudet är inställt (konfigurerbart).
* Om AEM-instansen svarar med följande rubriker:

   * `no-cache`
   * `no-store`
   * `must-revalidate`

>[!NOTE]
>
>GET- eller HEAD-metoderna (för HTTP-rubriken) kan nås av Dispatcher. Mer information om cachelagring av svarshuvuden finns i avsnittet [Caching HTTP Response Headers](#caching-http-response-headers).

Varje objekt i egenskapen `/rules` innehåller ett [`glob`](#designing-patterns-for-glob-properties)-mönster och en typ:

* Mönstret `glob` används för att matcha dokumentets sökväg.
* Typen anger om de dokument som matchar mönstret `glob` ska cachelagras. Värdet kan vara `allow` (cachelagra dokumentet) eller `deny` (återge dokumentet).

Om du inte har dynamiska sidor (utöver de sidor som redan har undantagits av ovanstående regler) kan du konfigurera Dispatcher att cachelagra allt. Regelavsnittet ser ut så här:

```xml
/rules
  {
    /0000  {  /glob "*"   /type "allow" }
  }
```

Mer information om Glob-egenskaper finns i [Designa mönster för Glob-egenskaper](#designing-patterns-for-glob-properties).

Om det finns dynamiska avsnitt på sidan (till exempel ett nyhetsprogram) eller i en sluten användargrupp kan du definiera undantag:

>[!NOTE]
>
>Cachelagra inte stängda användargrupper. Orsaken är att användarrättigheter inte kontrolleras för cachelagrade sidor.

```xml
/rules
  {
   /0000  { /glob "*" /type "allow" }
   /0001  { /glob "/en/news/*" /type "deny" }
   /0002  { /glob "*/private/*" /type "deny"  }
  }
```

**Komprimering**

På Apache-webbservrar kan du komprimera de cachelagrade dokumenten. Komprimering gör att Apache kan returnera dokumentet i ett komprimerat format om klienten begär det. Komprimering utförs automatiskt genom att Apache-modulen `mod_deflate` aktiveras, till exempel:

```xml
AddOutputFilterByType DEFLATE text/plain
```

Modulen installeras som standard med Apache 2.x.

<!-- 

Comment Type: draft

<note type="note"> 
 <p>Depending on the Apache web server version you can enable <span class="code">gzip</span> compression as follows:</p> 
 <ul> 
  <li>For Apache 1.3 you can enable the <span class="code">mod_gzip </span>module. The module must be downloaded and installed.</li> 
  <li>For Apache 2.x you can enable the <span class="code">mod_deflate</span> module. The module is installed by default with Apache 2.x.<br /> </li> 
 </ul> 
 <p> </p> 
</note>

 -->

<!-- 

Comment Type: draft

<p>The following rule caches all documents in compressed form; Apache can return either the uncompressed or the compressed form to the client:</p>

 -->

<!-- 

Comment Type: draft

<codeblock gutter="true" class="syntax xml">
  /rules!!discoiqbr!!&nbsp;&nbsp;{!!discoiqbr!!&nbsp;&nbsp;&nbsp;/rulelabel&nbsp;&nbsp;{&nbsp;&nbsp;/glob&nbsp;"*"&nbsp;/type&nbsp;"allow"&nbsp;&nbsp;/compress&nbsp;"gzip"&nbsp;}!!discoiqbr!!&nbsp;&nbsp;} 
</codeblock>

 -->

<!-- 

Comment Type: remark
Last Modified By: Silviu Raiman (raiman)
Last Modified Date: 2017-11-13T09:23:24.326-0500

<p>Hidden the <span class="code">mod_gzip</span> content as requested in CQDOC-11124.</p>

 -->

### Ogiltiga filer per mappnivå {#invalidating-files-by-folder-level}

Använd egenskapen `/statfileslevel` för att göra cachelagrade filer ogiltiga enligt deras sökväg:

* Dispatcher skapar `.stat` filer i varje mapp från mappen docroot till den nivå du anger. Dokumentmappen är nivå 0.
* Filerna blir ogiltiga genom att filen `.stat` klickas på. Filens senaste ändringsdatum för `.stat` jämförs med det senaste ändringsdatumet för ett cachelagrat dokument. Dokumentet uppdateras om filen `.stat` är nyare.

* När en fil på en viss nivå är ogiltig, ändras **alla** `.stat` filer från docroot **till** på nivån för den ogiltiga filen eller den konfigurerade `statsfilevel` (beroende på vilken som är minst).

   * Om du till exempel anger egenskapen `statfileslevel` som 6 och en fil blir ogiltig på nivå 5, kommer alla `.stat`-filer från docroot att ändras till 5. Om du fortsätter med det här exemplet kommer alla `stat`-filer från docroot till sex att påverkas (sedan `/statfileslevel = "6"`) om en fil blir ogiltig på nivå 7.

Endast resurserna **längs sökvägen** till den ogiltiga filen påverkas. Titta på följande exempel: en webbplats använder strukturen `/content/myWebsite/xx/.` Om du anger `statfileslevel` som 3 skapas en `.stat`-fil enligt följande:

* `docroot`
* `/content`
* `/content/myWebsite`
* `/content/myWebsite/*xx*`

När en fil i `/content/myWebsite/xx` blir ogiltig kommer alla `.stat`-filer från docroot ned till `/content/myWebsite/xx` att påverkas. Detta scenario är endast fallet för `/content/myWebsite/xx` och inte till exempel `/content/myWebsite/yy` eller `/content/anotherWebSite`.

>[!NOTE]
>
>Ogiltigförklaring kan förhindras genom att ett extra huvud `CQ-Action-Scope:ResourceOnly` skickas. Den här metoden kan användas för att tömma vissa resurser utan att andra delar av cachen blir ogiltiga. Mer information finns på [den här sidan](https://adobe-consulting-services.github.io/acs-aem-commons/features/dispatcher-flush-rules/index.html) och [Invalidera Dispatcher-cachen manuellt](https://experienceleague.adobe.com/sv/docs/experience-manager-dispatcher/using/configuring/page-invalidate#configuring).

>[!NOTE]
>
>Om du anger ett värde för egenskapen `/statfileslevel` ignoreras egenskapen `/statfile`.

### Automatisk invalidering av cachelagrade filer {#automatically-invalidating-cached-files}

Egenskapen `/invalidate` definierar de dokument som automatiskt görs ogiltiga när innehållet uppdateras.

Med automatisk ogiltigförklaring tar Dispatcher inte bort cachelagrade filer efter en innehållsuppdatering, utan kontrollerar deras giltighet nästa gång de begärs. Dokument i cacheminnet som inte ogiltigförklaras automatiskt finns kvar i cacheminnet tills en innehållsuppdatering tar bort dem explicit.

Automatisk ogiltigförklaring används vanligtvis för HTML-sidor. HTML-sidor innehåller ofta länkar till andra sidor, vilket gör det svårt att avgöra om en innehållsuppdatering påverkar en sida. Om du vill vara säker på att alla relevanta sidor blir ogiltiga när innehållet uppdateras, gör du alla HTML-sidor automatiskt ogiltiga. Följande konfiguration gör alla HTML-sidor ogiltiga:

```xml
  /invalidate
  {
   /0000  { /glob "*" /type "deny" }
   /0001  { /glob "*.html" /type "allow" }
  }
```

Mer information om Glob-egenskaper finns i [Designa mönster för Glob-egenskaper](#designing-patterns-for-glob-properties).

Den här konfigurationen orsakar följande aktivitet när `/content/wknd/us/en` aktiveras:

* Alla filer med mönstret en.* har tagits bort från mappen `/content/wknd/us`.
* Mappen `/content/wknd/us/en./_jcr_content` har tagits bort.
* Alla andra filer som matchar konfigurationen för `/invalidate` tas inte bort omedelbart. Dessa filer tas bort när nästa begäran görs. I exemplet tas inte `/content/wknd.html` bort, utan tas bort när `/content/wknd.html` begärs.

Om du erbjuder automatiskt genererade PDF- och ZIP-filer för nedladdning kan du behöva göra dessa filer ogiltiga automatiskt. Ett konfigurationsexempel ser ut så här:

```xml
/invalidate
  {
   /0000 { /glob "*" /type "deny" }
   /0001 { /glob "*.html" /type "allow" }
   /0002 { /glob "*.zip" /type "allow" }
   /0003 { /glob "*.pdf" /type "allow" }
  }
```

AEM-integreringen med Adobe Analytics levererar konfigurationsdata i en `analytics.sitecatalyst.js`-fil på din webbplats. Exempelfilen `dispatcher.any` som tillhandahålls med Dispatcher innehåller följande ogiltighetsregel för den här filen:

```xml
{
   /glob "*/analytics.sitecatalyst.js"  /type "allow"
}
```

### Använda anpassade ogiltighetsskript {#using-custom-invalidation-scripts}

Med egenskapen `/invalidateHandler` kan du definiera ett skript som anropas för varje ogiltigförklaringsbegäran som tas emot av Dispatcher.

Den anropas med följande argument:

* Handtag - Innehållssökvägen som är ogiltig
* Åtgärd - Replikeringsåtgärden (till exempel Aktivera, Inaktivera)
* Åtgärdsomfång - Replikeringsåtgärdens omfång (tomt, om inte en rubrik för `CQ-Action-Scope: ResourceOnly` skickas, se [Invalidera cachelagrade sidor från AEM](page-invalidate.md) för mer information)

Den här metoden kan användas för att täcka flera olika användningsfall. Om du till exempel gör andra programspecifika cacheminnen ogiltiga eller hanterar fall där den externa URL-adressen för en sida och dess plats i dokumentroten inte matchar innehållssökvägen.

Följande exempelskript loggar varje ogiltig begäran till en fil.

```xml
/invalidateHandler "/opt/dispatcher/scripts/invalidate.sh"
```

#### Exempel på ogiltighetshanterarskript {#sample-invalidation-handler-script}

```shell
#!/bin/bash

printf "%-15s: %s %s" $1 $2 $3>> /opt/dispatcher/logs/invalidate.log
```

### Begränsa de klienter som kan tömma cachen {#limiting-the-clients-that-can-flush-the-cache}

Egenskapen `/allowedClients` definierar specifika klienter som tillåts tömma cachen. Globeringsmönstren matchas mot IP.

Följande exempel:

1. nekar åtkomst till alla klienter
1. explicit tillåter åtkomst till localhost

```xml
/allowedClients
  {
   /0001 { /glob "*.*.*.*"  /type "deny" }
   /0002 { /glob "127.0.0.1" /type "allow" }
  }
```

Mer information om Glob-egenskaper finns i [Designa mönster för Glob-egenskaper](#designing-patterns-for-glob-properties).

>[!CAUTION]
>
>Vi rekommenderar att du definierar `/allowedClients`.
>
>Om detta inte görs kan alla klienter utfärda ett anrop för att rensa cachen. Om du gör det flera gånger kan det påverka webbplatsens prestanda negativt.

### Ignorerar URL-parametrar {#ignoring-url-parameters}

Avsnittet `ignoreUrlParams` definierar vilka URL-parametrar som ska ignoreras när du avgör om en sida cachelagras eller levereras från cache:

* När en begärande URL innehåller parametrar som alla ignoreras, cachelagras sidan.
* När en begärande URL innehåller en eller flera parametrar som inte ignoreras, cachelagras inte sidan.

När en parameter ignoreras för en sida cachelagras sidan första gången som sidan begärs. Efterföljande begäranden för sidan skickas till den cachelagrade sidan, oavsett värdet på parametern i begäran.

>[!NOTE]
>
>Vi rekommenderar att du konfigurerar inställningen `ignoreUrlParams` på ett sätt som tillåtslista. Därför ignoreras alla frågeparametrar och endast kända eller förväntade frågeparametrar undantas (&quot;nekas&quot;) från att ignoreras. Mer information och exempel finns på [den här sidan](https://github.com/adobe/aem-dispatcher-optimizer-tool/blob/main/docs/Rules.md#dot---the-dispatcher-publish-farm-cache-should-have-its-ignoreurlparams-rules-configured-in-an-allow-list-manner).

Om du vill ange vilka parametrar som ska ignoreras lägger du till glob-regler i egenskapen `ignoreUrlParams`:

* Om du vill cachelagra en sida oavsett vilken begäran som innehåller en URL-parameter, skapar du en glob-egenskap som tillåter parametern (att ignoreras).
* Om du vill förhindra att sidan cachas skapar du en globegenskap som nekar parametern (som ignoreras).

>[!NOTE]
>
>När du konfigurerar glob-egenskapen bör den matcha frågeparameternamnet. Om du till exempel vill ignorera parametern &quot;p1&quot; från följande URL `http://example.com/path/test.html?p1=test&p2=v2`, ska egenskapen glob vara:
> `/0002 { /glob "p1" /type "allow" }`

I följande exempel ignoreras alla parametrar, förutom parametern `nocache`. Därför cachelagrar Dispatcher aldrig URL-adresser som innehåller parametern `nocache`:

```xml
/ignoreUrlParams
{
    # ignore-all-url-parameters-by-dispatcher-and-requests-are-cached
    /0001 { /glob "*" /type "allow" }
    # allow-the-url-parameter-nocache-to-bypass-dispatcher-on-every-request
    /0002 { /glob "nocache" /type "deny" }
}
```

I kontexten för konfigurationsexemplet `ignoreUrlParams` ovan gör följande HTTP-begäran att sidan cachas eftersom parametern `willbecached` ignoreras:

```xml
GET /mypage.html?willbecached=true
```

I kontexten för konfigurationsexemplet `ignoreUrlParams` gör följande HTTP-begäran att sidan **not** cachelagras eftersom parametern `nocache` inte ignoreras:

```xml
GET /mypage.html?nocache=true
GET /mypage.html?nocache=true&willbecached=true
```

Mer information om Glob-egenskaper finns i [Designa mönster för Glob-egenskaper](#designing-patterns-for-glob-properties).

### Cachelagra HTTP-svarshuvuden {#caching-http-response-headers}

>[!NOTE]
>
>Den här funktionen är tillgänglig med version **4.1.11** av Dispatcher.

Med egenskapen `/headers` kan du definiera de HTTP-huvudtyper som Dispatcher ska cachelagra. Vid den första begäran till en icke cachelagrad resurs lagras alla huvuden som matchar ett av de konfigurerade värdena (se konfigurationsexemplet nedan) i en separat fil bredvid cachefilen. Vid efterföljande begäranden till den cachelagrade resursen läggs de lagrade rubrikerna till i svaret.

Nedan visas ett exempel från standardkonfigurationen:

```xml
/cache {
  ...
  /headers {
    "Cache-Control"
    "Content-Disposition"
    "Content-Type"
    "Expires"
    "Last-Modified"
    "X-Content-Type-Options"
    "Last-Modified"
  }
}
```

>[!NOTE]
>
>Det är inte tillåtet att använda ordningstecken. Mer information finns i [Utforma mönster för globala egenskaper](#designing-patterns-for-glob-properties).

>[!NOTE]
>
>Om du vill att Dispatcher ska lagra och leverera ETag-svarshuvuden från AEM gör du följande:
>
>* Lägg till rubriknamnet i avsnittet `/cache/headers`.
>* Lägg till följande [Apache-direktiv](https://httpd.apache.org/docs/2.4/mod/core.html#fileetag) i det Dispatcher-relaterade avsnittet:
>
>```xml
>FileETag none
>```

### Filbehörigheter för Dispatcher-cache {#dispatcher-cache-file-permissions}

Egenskapen `mode` anger vilka filbehörigheter som ska gälla för nya kataloger och filer i cachen. `umask` i anropsprocessen begränsar den här inställningen. Det är ett oktalt tal som konstruerats av summan av ett eller flera av följande värden:

* `0400` Tillåt läsning av ägare.
* `0200` Tillåt skrivning efter ägare.
* `0100` Tillåt ägaren att söka i kataloger.
* `0040` Tillåt läsning av gruppmedlemmar.
* `0020` Tillåt skrivning av gruppmedlemmar.
* `0010` Tillåt gruppmedlemmar att söka i katalogen.
* `0004` Tillåt läsning av andra.
* `0002` Tillåt andra att skriva.
* `0001` Tillåt andra att söka i katalogen.

Standardvärdet är `0755`, vilket gör att ägaren kan läsa, skriva eller söka i och att gruppen och andra kan läsa eller söka i.

### Begränsar .stat-filberöring {#throttling-stat-file-touching}

Med standardegenskapen `/invalidate` gör varje aktivering alla `.html`-filer ogiltiga (när deras sökväg matchar avsnittet `/invalidate`). På en webbplats med stor trafik ökar multipla aktiveringar processorbelastningen på backend-sidan. I ett sådant scenario är det önskvärt att strypa filberöringen `.stat` så att webbplatsen förblir responsiv. Du kan utföra den här åtgärden med egenskapen `/gracePeriod`.

Egenskapen `/gracePeriod` definierar hur många sekunder en inaktuell, automatiskt ogiltigförklarad resurs fortfarande kan hanteras från cacheminnet efter den senaste aktiveringen. Egenskapen kan användas i en inställning där en grupp av aktiveringar annars skulle göra hela cachen ogiltig upprepade gånger. Det rekommenderade värdet är 2 sekunder.

Mer information finns i `/invalidate` och `/statfileslevel`tidigare.

### Konfigurerar tidsbaserad cacheinvalidering - `/enableTTL` {#configuring-time-based-cache-invalidation-enablettl}

Tidsbaserad cacheogiltigförklaring beror på egenskapen `/enableTTL` och förekomsten av vanliga förfallorubriker från HTTP-standarden. Om du ställer in egenskapen på 1 (`/enableTTL "1"`) utvärderas svarshuvuden från serverdelen. Om rubrikerna innehåller ett `Cache-Control`-, `max-age`- eller `Expires`-datum skapas en tom hjälpfil bredvid den cachelagrade filen, med en ändringstid som är lika med förfallodatumet. När den cachelagrade filen har begärts efter ändringstiden återbegärs den automatiskt från serverdelen.

Före Dispatcher 4.3.5 baserades logiken för TTL-ogiltigförklaring endast på det konfigurerade TTL-värdet. I Dispatcher 4.3.5 beaktas både de angivna TTL-värdena **och**, Dispatcher-cacheminnets ogiltighetsregler. För en cachelagrad fil:

1. Om `/enableTTL` är inställt på 1 kontrolleras filens förfallodatum. Om filen har gått ut enligt angiven TTL utförs inga andra kontroller och den cachelagrade filen efterfrågas igen från serverdelen.
2. Om filen inte har gått ut eller om `/enableTTL` inte har konfigurerats, tillämpas standardreglerna för ogiltigförklaring av cachen, till exempel de regler som [`/statfileslevel`](#invalidating-files-by-folder-level) och [`/invalidate`](#automatically-invalidating-cached-files) har angetts. Det här flödet innebär att Dispatcher kan göra filer som TTL-värdet inte har upphört att gälla för ogiltiga.

Den nya implementeringen stöder användning där filerna har längre TTL-värden (till exempel CDN). Men filen kan fortfarande ogiltigförklaras även om TTL-värdet inte har gått ut. Det prioriterar innehållets aktualitet framför cache-träffkvoten på Dispatcher.

Om du däremot behöver **endast** anger du förfallologiken som används för en fil till `/enableTTL` till 1 och exkluderar den filen från standardcacheminnets invalideringsmekanism. Du kan till exempel:

* Om du vill ignorera filen konfigurerar du [invalideringsreglerna](#automatically-invalidating-cached-files) i cacheavsnittet. I kodutdraget nedan ignoreras alla filer som slutar på `.example.html` och upphör att gälla först när den angivna TTL:en har passerats.

```xml
  /invalidate
  {
   /0000  { /glob "*" /type "deny" }
   /0001  { /glob "*.html" /type "allow" }
   /0002  { /glob "*.example.html" /type "deny" }
  }
```

* Utforma innehållsstrukturen på ett sådant sätt att du kan ange ett högt [`/statfilelevel`](#invalidating-files-by-folder-level) så att filen inte automatiskt blir ogiltig.

Om du gör det ser du till att `.stat`-filogiltigförklaring inte används och att bara TTL-förfallotiden är aktiv för de angivna filerna.

>[!NOTE]
>
>Tänk på att om du anger `/enableTTL` till 1 aktiveras endast TTL-cachelagring på Dispatcher-sidan. TTL-informationen i den kompletterande filen (se ovan) tillhandahålls inte till någon annan användaragent som begär en sådan filtyp från Dispatcher. Om du vill ge cachelagringshuvuden till underordnade system som ett CDN eller en webbläsare, bör du konfigurera avsnittet `/cache/headers` i enlighet med detta.

>[!NOTE]
>
>Den här funktionen är tillgänglig i Dispatcher version **4.1.11** eller senare.

## Konfigurerar belastningsutjämning - `/statistics` {#configuring-load-balancing-statistics}

Avsnittet `/statistics` definierar de filkategorier för vilka Dispatcher bedömer svarstiden för varje återgivning. Dispatcher använder poängen för att avgöra vilken rendering som ska skickas.

Varje kategori som du skapar definierar ett globmönster. Dispatcher jämför URI:n för det begärda innehållet med dessa mönster för att fastställa kategorin för det begärda innehållet:

* Kategoriernas ordning avgör i vilken ordning de jämförs med URI:n.
* Det första kategorimönstret som matchar URI:n är filens kategori. Inga fler kategorimönster utvärderas.

Dispatcher har stöd för högst åtta statistikkategorier. Om du definierar fler än åtta kategorier används bara de första 8.

**Återge markering**

Varje gång Dispatcher kräver en återgiven sida används följande algoritm för att välja återgivning:

1. Om begäran innehåller återgivningsnamnet i en `renderid`-cookie använder Dispatcher den återgivningen.
1. Om begäran inte innehåller någon `renderid`-cookie jämför Dispatcher renderingsstatistiken:

   1. Dispatcher bestämmer kategorin för begärande-URI.
   1. Dispatcher avgör vilken rendering som har lägst svarspoäng för den kategorin och väljer vilken rendering som ska användas.

1. Om ingen rendering är markerad än använder du den första renderingen i listan.

Poängen för en återgivnings kategori baseras på tidigare svarstider och tidigare misslyckade och lyckade anslutningar som Dispatcher försöker genomföra. För varje försök uppdateras poängen för kategorin för den begärda URI:n.

>[!NOTE]
>
>Om du inte använder belastningsutjämning kan du utelämna det här avsnittet.

### Definiera statistikkategorier {#defining-statistics-categories}

Definiera en kategori för varje dokumenttyp som du vill behålla statistik för för återgivningsmarkering. Avsnittet `/statistics` innehåller ett `/categories`-avsnitt. Om du vill definiera en kategori lägger du till en rad under avsnittet `/categories` som har följande format:

`/name { /glob "pattern"}`

Kategorin `name` måste vara unik för servergruppen. `pattern` beskrivs i avsnittet [Designa mönster för globegenskaper](#designing-patterns-for-glob-properties).

För att fastställa kategorin för en URI jämför Dispatcher URI:n med varje kategorimönster tills en matchning hittas. Dispatcher börjar med den första kategorin i listan och fortsätter i rätt ordning. Därför bör du placera kategorier med mer specifika mönster först.

Dispatcher-standardfilen `dispatcher.any` definierar till exempel en HTML-kategori och en annan kategori. Kategorin HTML är mer specifik och visas därför först:

```xml
/statistics
  {
  /categories
    {
      /html { /glob "*.html" }
      /others  { /glob "*" }
    }
  }
```

I följande exempel finns även en kategori för söksidor:

```xml
/statistics
  {
  /categories
    {
      /search { /glob "*search.html" }
      /html { /glob "*.html" }
      /others  { /glob "*" }
    }
  }
```

### Speglar om att servern inte är tillgänglig i Dispatcher-statistik {#reflecting-server-unavailability-in-dispatcher-statistics}

Egenskapen `/unavailablePenalty` anger den tid (i tiondelar av en sekund) som används för återgivningsstatistiken när en anslutning till återgivningen misslyckas. Dispatcher lägger till tiden i statistikkategorin som matchar den begärda URI:n.

Straffet tillämpas till exempel när TCP/IP-anslutningen till det angivna värdnamnet/porten inte kan upprättas. Orsaken är antingen att AEM inte körs (och inte lyssnar) eller på grund av ett nätverksrelaterat problem.

Egenskapen `/unavailablePenalty` är direkt underordnad avsnittet `/farm` (en jämställd del i avsnittet `/statistics`).

Om det inte finns någon `/unavailablePenalty`-egenskap används värdet `"1"`.

```xml
/unavailablePenalty "1"
```

## Identifierar en mapp för fästig anslutning - `/stickyConnectionsFor` {#identifying-a-sticky-connection-folder-stickyconnectionsfor}

Egenskapen `/stickyConnectionsFor` definierar en mapp som innehåller klisterlappande dokument. Den här egenskapen används med URL:en. Dispatcher skickar alla begäranden från en enskild användare som finns i den här mappen till samma återgivningsinstans. Anteckningar säkerställer att sessionsdata finns och är konsekventa för alla dokument. Den här mekanismen använder cookien `renderid`.

I följande exempel definieras en fast anslutning till mappen /products:

```xml
/stickyConnectionsFor "/products"
```

När en sida består av innehåll från flera innehållsnoder inkluderar du egenskapen `/paths` som listar sökvägarna till innehållet. En sida innehåller till exempel innehåll från `/content/image`, `/content/video` och `/var/files/pdfs`. Med följande konfiguration aktiveras häftiga anslutningar för allt innehåll på sidan:

```xml
/stickyConnections {
  /paths {
    "/content/image"
    "/content/video"
    "/var/files/pdfs"
  }
}
```

### `httpOnly` {#httponly}

När klisterlappande anslutningar är aktiverade ställer Dispatcher-modulen in cookien `renderid`. Den här cookien har inte flaggan `httponly`, som bör läggas till för att öka säkerheten. Du lägger till flaggan `httponly` genom att ange egenskapen `httpOnly` i noden `/stickyConnections` i konfigurationsfilen `dispatcher.any`. Egenskapens värde (antingen `0` eller `1`) definierar om `renderid`-cookien har attributet `HttpOnly` tillagt. Standardvärdet är `0`, vilket innebär att attributet inte läggs till.

Mer information om flaggan `httponly` finns på [den här sidan](https://owasp.org/www-community/HttpOnly).

### `secure` {#secure}

När klisterlappande anslutningar är aktiverade ställer Dispatcher-modulen in cookien `renderid`. Den här cookien har inte flaggan `secure`, som bör läggas till för att öka säkerheten. Du lägger till flaggan `secure` som anger egenskapen `secure` i noden `/stickyConnections` i konfigurationsfilen `dispatcher.any`. Egenskapens värde (antingen `0` eller `1`) definierar om `renderid`-cookien har attributet `secure` tillagt. Standardvärdet är `0`, vilket innebär att attributet läggs till **om** den inkommande begäran är säker. Om värdet är `1` läggs flaggan secure till oavsett om den inkommande begäran är säker eller inte.

## Hantera återgivningsanslutningsfel {#handling-render-connection-errors}

Konfigurera Dispatcher när återgivningsservern returnerar ett 500-fel eller inte är tillgänglig.

### Ange en hälsokontrollsida {#specifying-a-health-check-page}

Använd egenskapen `/health_check` för att ange en URL som kontrolleras när en 500-statuskod inträffar. Om den här sidan även returnerar en 500-statuskod anses instansen vara otillgänglig och en konfigurerbar tidsåtgång ( `/unavailablePenalty`) tillämpas på återgivningen innan du försöker igen.

```xml
/health_check
  {
  # Page gets contacted when an instance returns a 500
  /url "/health_check.html"
  }
```

### Ange fördröjning för sidåterförsök {#specifying-the-page-retry-delay}

Egenskapen `/retryDelay` anger den tid (i sekunder) som Dispatcher väntar mellan anslutningsförsök med servergruppens återgivningar. För varje rund är det högsta antalet gånger Dispatcher försöker ansluta till en rendering antalet renderingar i servergruppen.

Dispatcher använder värdet `"1"` om `/retryDelay` inte uttryckligen har definierats. Standardvärdet är vanligtvis lämpligt.

```xml
/retryDelay "1"
```

### Konfigurera antalet återförsök {#configuring-the-number-of-retries}

Egenskapen `/numberOfRetries` anger det maximala antalet anslutningsförsök som Dispatcher utför med återgivningarna. Om Dispatcher inte kan ansluta till en rendering efter detta antal försök returnerar Dispatcher ett misslyckat svar.

För varje rund är det högsta antalet gånger Dispatcher försöker ansluta till en rendering antalet renderingar i servergruppen. Det högsta antalet gånger som Dispatcher försöker ansluta är därför ( `/numberOfRetries`) x (antalet återgivningar).

Om värdet inte definieras explicit är standardvärdet `5`.

```xml
/numberOfRetries "5"
```

### Använda mekanismen för växling vid fel {#using-the-failover-mechanism}

Om du vill skicka om begäranden till olika återgivningar när den ursprungliga begäran misslyckas, aktiverar du redundansfunktionen i Dispatcher-servergruppen. När redundans är aktiverat fungerar Dispatcher på följande sätt:

* När en begäran om en återgivning returnerar HTTP-status 503 (UNAVAILABLE), skickar Dispatcher begäran till en annan återgivning.
* När en begäran till en återgivning returnerar HTTP-status 50x (annan än 503), skickar Dispatcher en begäran om sidan som är konfigurerad för egenskapen `health_check`.
   * Om hälsokontrollen returnerar 500 (INTERNAL_SERVER_ERROR) skickar Dispatcher den ursprungliga begäran till en annan rendering.
   * Om hälsokontrollen returnerar HTTP-status 200 returnerar Dispatcher det ursprungliga HTTP 500-felet till klienten.

Om du vill aktivera redundans lägger du till följande rad i servergruppen (eller webbplatsen):

```xml
/failover "1"
```

>[!NOTE]
>
>Om du vill försöka återanvända HTTP-begäranden som innehåller en brödtext skickar Dispatcher en `Expect: 100-continue`-begäranderubrik till återgivningen innan det faktiska innehållet mellanlagras. CQ 5.5 med CQSE besvarar sedan omedelbart med antingen 100 (CONTINUE) eller en felkod. Andra serverletsbehållare stöds också.

## Ignorerar avbrott - `/ignoreEINTR` {#ignoring-interruption-errors-ignoreeintr}

>[!CAUTION]
>
>Det här alternativet behövs inte. Använd den bara när du ser följande loggmeddelanden:
>
>`Error while reading response: Interrupted system call`

Alla filsystemorienterade systemanrop kan avbrytas `EINTR` om objektet för systemanropet finns på ett fjärrsystem som nås via NFS. Huruvida dessa systemanrop kan ta slut eller avbrytas baseras på hur det underliggande filsystemet monterades på den lokala datorn.

Använd parametern `/ignoreEINTR` om instansen har en sådan konfiguration och loggen innehåller följande meddelande:

`Error while reading response: Interrupted system call`

Internt läser Dispatcher svaret från fjärrservern (dvs. AEM) med en slinga som kan representeras som:

```text
while (response not finished) {  
read more data  
}
```

Sådana meddelanden kan genereras när `EINTR` inträffar i avsnittet `read more data`. Och det är orsaken till att en signal tas emot innan data tas emot.

Om du vill ignorera sådana avbrott kan du lägga till följande parameter i `dispatcher.any` (före `/farms`):

`/ignoreEINTR "1"`

Om `/ignoreEINTR` anges till `"1"` fortsätter Dispatcher att försöka läsa data tills det fullständiga svaret har lästs. Standardvärdet är `0` och alternativet inaktiveras.

## Designa mönster för globala egenskaper {#designing-patterns-for-glob-properties}

I flera avsnitt i Dispatcher konfigurationsfil används egenskaperna `glob` som urvalskriterier för klientbegäranden. Värdena för `glob`-egenskaperna är mönster som Dispatcher jämför med en aspekt av begäran, till exempel sökvägen till den begärda resursen eller klientens IP-adress. Objekten i avsnittet `/filter` använder till exempel `glob` mönster för att identifiera sökvägarna till de sidor som Dispatcher agerar på eller avvisar.

Värdena för `glob` kan innehålla jokertecken och alfanumeriska tecken för att definiera mönstret.

| Jokertecken | Beskrivning | Exempel |
|--- |--- |--- |
| `*` | Matchar noll eller flera intilliggande förekomster av ett tecken i strängen. Något av följande avgör det sista tecknet i matchningen: <br/>Ett tecken i strängen matchar nästa tecken i mönstret och mönstertecknet har följande egenskaper:<br/><ul><li>Inte en `*`</li><li>Inte en `?`</li><li>Ett literalt tecken (inklusive ett blanksteg) eller en teckenklass.</li><li>Slutet på mönstret nås.</li></ul>I en teckenklass tolkas tecknet bokstavligen. | `*/geo*` Matchar alla sidor under noden `/content/geometrixx` och noden `/content/geometrixx-outdoors`. Följande HTTP-begäranden matchar mönstret: <br/><ul><li>`"GET /content/geometrixx/en.html"`</li><li>`"GET /content/geometrixx-outdoors/en.html"` </li></ul><br/> `*outdoors/*` <br/>Matchar alla sidor under noden `/content/geometrixx-outdoors`. Följande HTTP-begäran matchar till exempel globmönstret: <br/><ul><li>`"GET /content/geometrixx-outdoors/en.html"`</li></ul> |
| `?` | Matchar ett enskilt tecken. Använd teckenklasser utanför. I en teckenklass tolkas det här tecknet bokstavligen. | `*outdoors/??/*`<br/> Matchar sidorna för alla språk i den externa geometrixx-webbplatsen. Följande HTTP-begäran matchar till exempel globmönstret: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>Följande begäran matchar inte mönstret: <br/><ul><li>&quot;GET /content/geometrixx-outdoors/en.html&quot;</li></ul> |
| `[ and ]` | Avmarkerar början och slutet av en teckenklass. Teckenklasser kan innehålla ett eller flera teckenintervall och enskilda tecken.<br/>En matchning inträffar om måltecknet matchar något av tecknen i teckenklassen, eller inom ett definierat intervall.<br/>Om den avslutande parentesen inte inkluderas skapas inga matchningar i mönstret. | `*[o]men.html*`<br/> Matchar följande HTTP-begäran:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>Den matchar inte följande HTTP-begäran:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/> `*[o/]men.html*` <br/>Matchar följande HTTP-begäranden: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `-` | Det betecknar ett teckenintervall. För användning i teckenklasser. Utanför en teckenklass tolkas detta tecken bokstavligen. | `*[m-p]men.html*` Matchar följande HTTP-begäran: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul>Den matchar inte följande HTTP-begäran:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `!` | Negerar tecknet eller teckenklassen som följer. Använd bara för negerande tecken och teckenintervall inuti teckenklasser. Motsvarar `^ wildcard`. <br/>Utanför en teckenklass tolkas det här tecknet bokstavligen. | `*[ !o]men.html*`<br/> Matchar följande HTTP-begäran: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>Den matchar inte följande HTTP-begäran: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>`*[ !o!/]men.html*`<br/> Den matchar inte följande HTTP-begäran:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"` eller `"GET /content/geometrixx-outdoors/en/men. html"`</li></ul> |
| `^` | Negerar tecknet eller teckenintervallet som följer. Används för att endast negera tecken och teckenintervall inuti teckenklasser. Motsvarar jokertecknet `!`. <br/>Utanför en teckenklass tolkas det här tecknet bokstavligen. | Exemplen för jokertecknet `!` används och `!`-tecknen i exempelmönstret ersätts med `^`-tecken. |


<!--- need to troubleshoot table

The following table describes the wildcard characters.

<table border="1" cellpadding="1" cellspacing="0" width="100%"> 
 <tbody> 
  <tr> 
   <th>Wildcard character</th> 
   <th>Description</th> 
   <th>Examples</th> 
  </tr> 
  <tr> 
   <td>*</td> 
   <td><p>Matches zero or more contiguous instances of any character in the string. The final character of the match is determined by either of the following situations:</p> 
    <ul> 
     <li>A character in the string matches the next character in the pattern, and the pattern character has the following characteristics: 
      <ul> 
       <li>Not a *</li> 
       <li>Not a ?</li> 
       <li>A literal character (including a space) or a character class.</li> 
      </ul> </li> 
     <li>The end of the pattern is reached.</li> 
    </ul> <p>Inside a character class, the character is interpreted literally.</p> </td> 
   <td><p>*/geo*</p> <p>Matches any page below the /content/geometrixx node and the /content/geometrixx-outdoors node. The following HTTP requests match the glob pattern:</p> 
    <ul> 
     <li>"GET /content/geometrixx/en.html"</li> 
     <li>"GET /content/geometrixx-outdoors/en.html" </li> 
    </ul> <p>*outdoors/*</p> <p>Matches any page below the /content/geometrixx-outdoors node. For example, the following HTTP request matches the glob pattern:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en.html"</li> 
    </ul> </td> 
  </tr> 
  <tr> 
   <td>?</td> 
   <td><p>Matches any single character. Use outside character classes.</p> <p>Inside a character class, this character is interpreted literally.</p> </td> 
   <td><p>*outdoors/??/*</p> <p>Matches the pages for any language in the geometrixx-outdoors site. For example, the following HTTP request matches the glob pattern:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/men.html"</li> 
    </ul> <p>The following request does not match the glob pattern:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en.html" </li> 
    </ul> </td> 
  </tr> 
  <tr> 
   <td>[ and ]</td> 
   <td><p>Demarks the beginning and end of a character class.</p> <p>Character classes can include one or more character ranges and single characters.</p> <p>A match occurs if the target character matches any of the characters in the character class, or within a defined range.</p> <p>If the closing bracket is not included, the pattern produces no matches.</p> <p></p> <p></p> <p></p> </td> 
   <td><p>*[o]men.html*</p> <p>Matches the following HTTP request:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/women.html"</li> 
    </ul> <p>Does not match the following HTTP request:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/men.html" </li> 
    </ul> <p>*[o/]men.html*</p> <p>Matches the following HTTP requests: </p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/women.html" </li> 
     <li> "GET /content/geometrixx-outdoors/en/men.html" </li> 
    </ul> <p></p> </td> 
  </tr> 
  <tr> 
   <td>-</td> 
   <td><p>Denotes a range of characters. For use in character classes.<br /> </p> <p>Outside of a character class, this character is interpreted literally.</p> <p></p> </td> 
   <td><p>*[m-p]men.html*</p> <p>Matches the following HTTP request:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/women.html"</li> 
    </ul> <p>Does not match the following HTTP request:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/men.html"</li> 
    </ul> <p></p> </td> 
  </tr> 
  <tr> 
   <td>!</td> 
   <td><p>Negates the character or character class that follows. Use only for negating characters and character ranges inside character classes. Equivalent to the ^ wildcard.</p> <p>Outside of a character class, this character is interpreted literally.</p> <p></p> </td> 
   <td><p>*[!o]men.html*</p> <p>Matches the following HTTP request: </p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/men.html"</li> 
    </ul> <p>Does not match the following HTTP request</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/women.html" </li> 
    </ul> <p>*[!o!/]men.html*</p> <p>Does not match the following HTTP request:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/women.html" or "GET /content/geometrixx-outdoors/en/men. html" </li> 
    </ul> </td> 
  </tr> 
  <tr> 
   <td>^</td> 
   <td><p>Negates the character or character range that follows. Use for negating only characters and character ranges inside character classes. Equivalent to the ! wildcard character.</p> <p>Outside of a character class, this charcter is interpreted literally.</p> </td> 
   <td>The examples for the ! wildcard character apply, substituting the ! characters in the example patterns with ^ characters.</td> 
  </tr> 
 </tbody> 
</table>
-->

## Loggning {#logging}

I webbserverkonfigurationen kan du ange:

* Platsen för Dispatcher-loggfilen.
* Loggnivån.

Mer information finns i webbserverdokumentationen och filen Viktigt för din Dispatcher-instans.

**Apache-loggar som roterats eller pipats**

Om du använder en **Apache**-webbserver kan du använda standardfunktionerna för Loggrotation, Pipe-loggar eller båda. Använd till exempel pipade loggar:

`DispatcherLog "| /usr/apache/bin/rotatelogs logs/dispatcher.log%Y%m%d 604800"`

Den här funktionen roterar automatiskt:

* Dispatcher-loggfilen med en tidsstämpel i tillägget (`logs/dispatcher.log%Y%m%d`).
* varje vecka (60 x 60 x 24 x 7 = 604800 sekunder).

Se dokumentationen för Apache Web Server om loggrotation och Pipe-loggar. Exempel: [Apache 2.4](https://httpd.apache.org/docs/2.4/logs.html).

>[!NOTE]
>
>Efter installationen är standardloggnivån hög (d.v.s. nivå 3 = Felsökning) så att Dispatcher loggar alla fel och varningar. Den här nivån är användbar i de inledande faserna.
>
>En sådan nivå kräver dock mer resurser. När Dispatcher fungerar jämnt *enligt dina krav* kan du sänka loggnivån.

### Spårningsloggning {#trace-logging}

Bland andra förbättringar för Dispatcher finns även version 4.2.0 med Trace Logging.

Den här möjligheten är högre än felsökningsloggning som visar ytterligare information i loggarna. Loggning läggs till för:

* Värdena för de vidarebefordrade rubrikerna.
* Regeln som tillämpas för en viss åtgärd.

Du kan aktivera spårningsloggning genom att ange loggnivån till `4` på webbservern.

Nedan visas ett exempel på loggar med spårning aktiverat:

```xml
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[Host] = "localhost:8443"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[User-Agent] = "curl/7.43.0"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[Accept] = "*/*"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-SSL-Client-Cert] = "(null)"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[Via] = "1.1 localhost:8443 (dispatcher)"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-For] = "::1"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-SSL] = "on"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-SSL-Cipher] = "DHE-RSA-AES256-SHA"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-SSL-Session-ID] = "ba931f5e4925c2dde572d766fdd436375e15a0fd24577b91f4a4d51232a934ae"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-Port] = "8443"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[Server-Agent] = "Communique-Dispatcher"
```

Och en händelse loggas när en fil som matchar en blockeringsregel begärs:

```xml
[Thu Mar 03 14:42:45 2016] [T] [11831] 'GET /content.infinity.json HTTP/1.1' was blocked because of /0082
```

## Bekräfta grundläggande åtgärd {#confirming-basic-operation}

Så här bekräftar du grundläggande åtgärder och interaktion för webbservern, Dispatcher- och AEM-instansen:

1. Ange `loglevel` som `3`.

1. Starta webbservern. När du gör det startar även Dispatcher.
1. Starta AEM-instansen.
1. Kontrollera loggen och felfilerna för webbservern och Dispatcher.
   * Beroende på webbservern bör du se meddelanden som:
      * `[Thu May 30 05:16:36 2002] [notice] Apache/2.0.50 (Unix) configured` och
      * `[Fri Jan 19 17:22:16 2001] [I] [19096] Dispatcher initialized (build XXXX)`

1. Surfa på webbplatsen via webbservern. Bekräfta att innehållet visas som det ska.\
   På en lokal installation där AEM körs på port `4502` och webbservern på `80` kan du till exempel komma åt webbplatskonsolen med hjälp av båda:
   * `https://localhost:4502/libs/wcm/core/content/siteadmin.html`
   * `https://localhost:80/libs/wcm/core/content/siteadmin.html`
   * Resultaten ska vara identiska. Bekräfta åtkomst till andra sidor med samma mekanism.

1. Kontrollera att cachekatalogen fylls.
1. Aktivera en sida om du vill kontrollera att cacheminnet rensas korrekt.
1. Om allt fungerar som det ska kan du minska antalet `loglevel` till `0`.

## Använda flera utskickare {#using-multiple-dispatchers}

I komplexa inställningar kan du använda flera Dispatcher. Du kan till exempel använda:

* en Dispatcher för att publicera en webbplats på intranätet
* en andra Dispatcher, med en annan adress och olika säkerhetsinställningar, för att publicera samma innehåll på Internet.

I så fall måste du se till att varje begäran endast går igenom en Dispatcher. En Dispatcher hanterar inte begäranden som kommer från andra Dispatcher. Kontrollera därför att båda utskickarna har direktåtkomst till AEM webbplats.

## Felsökning {#debugging}

När du lägger till rubriken `X-Dispatcher-Info` i en begäran, besvarar Dispatcher om målet har cachelagrats, returnerats från cachelagrat eller inte kunnat cachelagras alls. Svarshuvudet `X-Cache-Info` innehåller den här informationen i ett läsbart format. Du kan använda dessa svarshuvuden för att felsöka problem som rör svar som cachelagrats av Dispatcher.

Den här funktionen är inte aktiverad som standard, så för att svarsrubriken `X-Cache-Info` ska inkluderas måste servergruppen innehålla följande post:

```xml
/info "1"
```

Exempel:

```xml
/farm
{
    /mywebsite
    {
        # Include X-Cache-Info response header if X-Dispatcher-Info is in request header
        /info "1"
    }
}
```

Dessutom behöver rubriken `X-Dispatcher-Info` inte något värde, men om du använder `curl` för testning måste du ange ett värde som ska skickas till rubriken, till exempel:

```xml
curl -v -H "X-Dispatcher-Info: true" https://localhost/content/wknd/us/en.html
```

Nedan finns en lista med de svarshuvuden som `X-Dispatcher-Info` returnerar:

* **målfilen cache-lagrad**\
  Målfilen finns i cachen och Dispatcher har fastställt att den kan levereras.
* **cachelagring**\
  Målfilen finns inte i cacheminnet och Dispatcher har fastställt att det går att cachelagra utdata och leverera dem.
* **cachelagring: statusfilen är nyare**
Målfilen finns i cachen. En senare statusfil kan dock göra den ogiltig. Dispatcher tar bort målfilen, återskapar den från utdata och levererar den.
* **kan inte nås: dokumentroten finns inte**
Servergruppens konfiguration innehåller inte någon dokumentrot (konfigurationselement `cache.docroot`).
* **går inte att komma åt: Sökvägen till cachefilen är för lång**\
  Målfilen - sammanfogningen av dokumentroten och URL-filen - överskrider det längsta möjliga filnamnet på systemet.
* **går inte att komma åt: den tillfälliga filsökvägen är för lång**\
  Mallen för tillfälligt filnamn överskrider det längsta möjliga filnamnet på systemet. Dispatcher skapar först en temporär fil innan den cachelagrade filen faktiskt skapas eller skrivs över. Det tillfälliga filnamnet är målfilens namn med tecknen `_YYYYXXXXXX` tillagda, där `Y` och `X` ersätts för att skapa ett unikt namn.
* **kan inte nås: URL för begäran saknar tillägg**\
  Begär-URL:en har inget tillägg, eller så finns det en sökväg som följer filtillägget, till exempel: `/test.html/a/path`.
* **går inte att komma åt: begäran måste vara en GET eller HEAD**
HTTP-metoden är inte en GET eller en HEAD. Dispatcher förutsätter att utdata innehåller dynamiska data som inte ska cachelagras.
* **kunde inte nås: begäran innehöll en frågesträng**\
  Begäran innehöll en frågesträng. Dispatcher förutsätter att utdata är beroende av den frågesträng som har angetts och därför inte cachelagras.
* **kan inte nås: sessionshanteraren måste autentisera**\
  En sessionshanterare (konfigurationen innehåller en `sessionmanagement`-nod) styr servergruppens cache och begäran innehöll inte rätt autentiseringsinformation.
* **kan inte nås: begäran innehåller auktorisering**\
  Servergruppen kan inte cachelagra utdata ( `allowAuthorized 0`) och begäran innehåller autentiseringsinformation.
* **kan inte nås: målet är en katalog**\
  Målfilen är en katalog. Platsen kan peka på ett konceptuellt fel, där både en URL och en delwebbadress innehåller cachelagrade utdata. Om en begäran till `/test.html/a/file.ext` till exempel kommer först och innehåller cachelagrade utdata, kan Dispatcher inte cachelagra utdata från en efterföljande begäran till `/test.html`.
* **kan inte nås: URL för begäran har ett avslutande snedstreck**\
  Begärans URL har ett avslutande snedstreck.
* **går inte att komma åt: URL för begäran saknas i cachereglerna**\
  Servergruppens cacheregler tillåter explicit cachelagring av utdata från en begäran-URL.
* **inte tillgänglig: åtkomstkontroll nekade åtkomst**\
  Gruppens behörighetskontroll nekade åtkomst till den cachelagrade filen.
* **kan inte nås: sessionen är ogiltig**
En sessionshanterare (konfigurationen innehåller en `sessionmanagement` -nod) styr servergruppens cache och användarens session är ogiltig eller inte längre giltig.
* **kan inte nås: svaret innehåller`no_cache`**
Fjärrservern returnerade ett `Dispatcher: no_cache` -huvud och förbjuder Dispatcher att cachelagra utdata.
* **kan inte nås: längden på svarsinnehållet är noll**
Svarets innehållslängd är noll. Dispatcher skapar inte en fil med längden noll.

