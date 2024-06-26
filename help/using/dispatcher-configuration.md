---
title: Konfigurera AEM Dispatcher
description: Lär dig hur du konfigurerar Dispatcher. Lär dig mer om stöd för IPv4 och IPv6, konfigurationsfiler, miljövariabler och om att namnge instansen. Läs om hur du definierar servergrupper, identifierar virtuella värdar och mycket mer.
exl-id: 91159de3-4ccb-43d3-899f-9806265ff132
source-git-commit: 9be9f5935c21ebbf211b5da52280a31772993c2e
workflow-type: tm+mt
source-wordcount: '8876'
ht-degree: 0%

---

# Konfigurera Dispatcher {#configuring-dispatcher}

>[!NOTE]
>
>Dispatcher-versionerna är oberoende av AEM (Adobe Experience Manager). Du kan ha omdirigerats till den här sidan om du har följt en länk till Dispatcher-dokumentationen. Länken var inbäddad i dokumentationen för en tidigare version av AEM.

I följande avsnitt beskrivs hur du konfigurerar olika aspekter av Dispatcher.

## Stöd för IPv4 och IPv6 {#support-for-ipv-and-ipv}

Alla element i AEM och Dispatcher kan installeras i både IPv4- och IPv6-nätverk. Se [IPV4 och IPV6](https://experienceleague.adobe.com/en/docs/experience-manager-65/content/implementing/deploying/introduction/technical-requirements#ipv-and-ipv).

## Dispatcher-konfigurationsfiler {#dispatcher-configuration-files}

Som standard lagras Dispatcher-konfigurationen i `dispatcher.any` textfil, men du kan ändra namn och plats för filen under installationen.

Konfigurationsfilen innehåller en serie egenskaper med ett eller flera värden som styr Dispatcher-beteendet:

* Egenskapsnamn föregås av ett snedstreck `/`.
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

Om du till exempel vill ta med filen myFarm.any i `/farms` ska följande kod användas i konfigurationen:

```xml
/farms
  {
  $include "myFarm.any"
  }
```

Använd asterisken (`*`) som jokertecken.

Om filerna `farm_1.any` till `farm_5.any` innehåller konfigurationen av grupperna ett till fem, du kan inkludera dem enligt följande:

```xml
/farms
  {
  $include "farm_*.any"
  }
```

## Använda miljövariabler {#using-environment-variables}

Du kan använda miljövariabler i egenskaper med strängvärden i dispatcher.any-filen i stället för att hårdkoda värdena. Om du vill ta med värdet för en miljövariabel använder du formatet `${variable_name}`.

Om till exempel filen dispatcher.any finns i samma katalog som cachekatalogen, anger följande värde för [docroot](#specifying-the-cache-directory) kan användas:

```xml
/docroot "${PWD}/cache"
```

Om du till exempel skapar en miljövariabel med namnet `PUBLISH_IP` som lagrar värdnamnet för den AEM publiceringsinstansen, följande konfiguration av [`/renders`](#defining-page-renderers-renders) kan användas:

```xml
/renders {
  /0001 {
    /hostname "${PUBLISH_IP}"
    /port "8443"
  }
}
```

## Namnge Dispatcher-instansen {#naming-the-dispatcher-instance-name}

Använd `/name` för att ange ett unikt namn som identifierar Dispatcher-instansen. The `/name` -egenskapen är en egenskap på den översta nivån i konfigurationsstrukturen.

## Definiera grupper {#defining-farms-farms}

The `/farms` -egenskapen definierar en eller flera uppsättningar Dispatcher-beteenden, där varje uppsättning är kopplad till olika webbplatser eller URL-adresser. The `/farms` kan innehålla en eller flera grupper:

* Använd en enda servergrupp när du vill att Dispatcher ska hantera alla dina webbsidor eller webbplatser på samma sätt.
* Skapa flera grupper när olika delar av webbplatsen eller olika webbplatser kräver olika Dispatcher-beteenden.

The `/farms` -egenskapen är en egenskap på den översta nivån i konfigurationsstrukturen. Om du vill definiera en servergrupp lägger du till en underordnad egenskap i `/farms` -egenskap. Använd ett egenskapsnamn som unikt identifierar servergruppen i Dispatcher-instansen.

The `/farmname` egenskapen är flervärdesbaserad och innehåller andra egenskaper som definierar Dispatcher-beteendet:

* URL:erna för de sidor som servergruppen gäller för.
* En eller flera tjänst-URL:er (vanligtvis AEM publiceringsinstanser) som används för att återge dokument.
* Statistik som ska användas för belastningsutjämning av flera dokumentåtergivare.
* Flera andra beteenden, till exempel vilka filer som ska cachelagras och var de ska cachelagras.

Värdet kan innehålla alla alfanumeriska tecken (a-z, 0-9). I följande exempel visas skelettdefinitionen för två grupper med namnet `/daycom` och `/docsdaycom`:

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
>Om du använder mer än en renderingsgrupp utvärderas listan längst ned. Det här flödet är relevant när du definierar [Virtuella värdar](#identifying-virtual-hosts-virtualhosts) för era webbplatser.

Varje gruppegenskap kan innehålla följande underordnade egenskaper:

| Egenskapsnamn | Beskrivning |
|--- |--- |
| [/hemsida](#specify-a-default-page-iis-only-homepage) | Standardstartsida (valfritt) (endast IIS) |
| [/clientheaders](#specifying-the-http-headers-to-pass-through-clientheaders) | Rubrikerna från klientens HTTP-begäran som ska skickas. |
| [/virtualhosts](#identifying-virtual-hosts-virtualhosts) | Den här servergruppens virtuella värdar. |
| [/sessionmanagement](#enabling-secure-sessions-sessionmanagement) | Stöd för sessionshantering och autentisering. |
| [/renders](#defining-page-renderers-renders) | Servrarna som tillhandahåller återgivna sidor (vanligtvis AEM publiceringsinstanser). |
| [/filter](#configuring-access-to-content-filter) | Definierar de URL:er som Dispatcher aktiverar åtkomst till. |
| [/vanity_urls](#enabling-access-to-vanity-urls-vanity-urls) | Konfigurerar åtkomst till mål-URL:er. |
| [/propagateSyndPost](#forwarding-syndication-requests-propagatesyndpost) | Stöd för vidarebefordran av syndikeringsbegäranden. |
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
>The `/homepage`parametern (endast IIS) fungerar inte längre. I stället bör du använda [IIS URL-omskrivningsmodul](https://learn.microsoft.com/en-us/iis/extensions/url-rewrite-module/using-the-url-rewrite-module).
>
>Om du använder Apache ska du använda `mod_rewrite` -modul. Se dokumentationen för Apache-webbplatsen för mer information om `mod_rewrite` (till exempel [Apache 2.4](https://httpd.apache.org/docs/current/mod/mod_rewrite.html)). När du använder `mod_rewrite`är det lämpligt att använda flaggan genomgång|PT (skicka till nästa hanterare) för att tvinga omskrivningsmotorn att ange `uri` fält för den interna `request_rec` struktur till värdet av `filename` fält.

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

The `/clientheaders` -egenskapen definierar en lista med HTTP-huvuden som Dispatcher skickar från klientens HTTP-begäran till återgivaren (AEM instans).

Som standard vidarebefordrar Dispatcher standard-HTTP-rubrikerna till AEM. I vissa fall kanske du vill vidarebefordra ytterligare rubriker eller ta bort specifika rubriker:

* Lägg till rubriker, t.ex. anpassade rubriker, som din AEM förväntar sig i HTTP-begäran.
* Ta bort rubriker, t.ex. autentiseringsrubriker som bara är relevanta för webbservern.

Om du anpassar uppsättningen rubriker som ska skickas måste du ange en uttömmande lista över rubriker, inklusive de rubriker som normalt inkluderas som standard.

En Dispatcher-instans som hanterar sidaktiveringsbegäranden för publiceringsinstanser kräver `PATH` sidhuvud i `/clientheaders` -avsnitt. The `PATH` I kan du kommunicera mellan replikeringsagenten och Dispatcher.

Följande kod är en exempelkonfiguration för `/clientheaders`:

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

The `/virtualhosts` -egenskapen definierar en lista med alla värdnamn och URI-kombinationer som Dispatcher accepterar för den här servergruppen. Du kan använda asterisken (`*`) som jokertecken. Värden för /`virtualhosts` -egenskapen har följande format:

```xml
[scheme]host[uri][*]
```

* `scheme`: (Valfritt) Antingen `https://` eller `https://.`
* `host`: Värddatorns namn eller IP-adress och portnumret om det behövs. (Se [https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23))
* `uri`: (Valfritt) Sökvägen till resurserna.

I följande exempel hanteras begäranden för `.com` och `.ch` domains of myCompany, and all domains of mySubDivision:

```xml
   /virtualhosts
    {
    "www.myCompany.com"
    "www.myCompany.ch"
    "www.mySubDivison.*"
    }
```

Följande konfigurationshandtag *alla* begäranden:

```xml
   /virtualhosts
    {
    "*"
    }
```

### Matchar den virtuella värden {#resolving-the-virtual-host}

När Dispatcher tar emot en HTTP- eller HTTPS-begäran, hittas det virtuella värdvärde som bäst matchar `host,` `uri`och `scheme` begärans rubriker. Dispatcher utvärderar värdena i `virtualhosts` i följande ordning:

* Dispatcher börjar längst ned i servergruppen och fortsätter uppåt i filen dispatcher.any.
* För varje grupp börjar Dispatcher med det översta värdet i `virtualhosts` och flyttar ned värdeslistan.

Dispatcher hittar det mest matchande virtuella värdvärdet på följande sätt:

* Det första virtuella värdsystemet som påträffades matchar alla tre av `host`, `scheme`och `uri` av begäran används.
* Om nej `virtualhosts` värden har `scheme` och `uri` delar som båda matchar `scheme` och `uri` av begäran, den första påträffade virtuella värddatorn som matchar `host` av begäran används.
* Om nej `virtualhosts` värden har en värddel som matchar värddatorn för begäran. Det översta virtuella värdsystemet för den översta servergruppen används.

Därför bör du placera din virtuella standardvärd högst upp i `virtualhosts` -egenskap. Anta det i den översta servergruppen i `dispatcher.any` -fil.

### Exempel på virtuell värdupplösning {#example-virtual-host-resolution}

Följande exempel representerar ett fragment från en `dispatcher.any` en fil som definierar två Dispatcher-grupper och varje grupp definierar en `virtualhosts` -egenskap.

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

## Aktivera säkra sessioner - `/sessionmanagement` {#enabling-secure-sessions-sessionmanagement}

>[!CAUTION]
>
>`/allowAuthorized` Ange till `"0"` i `/cache` för att aktivera funktionen. Enligt informationen i [Cachelagring när autentisering används](#caching-when-authentication-is-used) -sektion, när du anger `/allowAuthorized 0 ` begäranden som innehåller autentiseringsinformation **not** cachelagrad. Om behörighetskänslig cachelagring krävs finns mer information i [Cachelagra skyddat innehåll](https://experienceleague.adobe.com/en/docs/experience-manager-dispatcher/using/configuring/permissions-cache) sida.

Skapa en säker session för åtkomst till renderingsgruppen så att användarna måste logga in för att komma åt alla sidor i gruppen. När användaren har loggat in kan han/hon komma åt sidor i servergruppen. Se [Skapa en stängd användargrupp](https://experienceleague.adobe.com/en/docs/experience-manager-65/content/security/cug#creating-the-user-group-to-be-used) om du vill ha information om hur du använder den här funktionen med CUG. Se även Skicka [Säkerhetschecklista](/help/using/security-checklist.md) innan du publicerar.

The `/sessionmanagement` egenskapen är en underegenskap till `/farms`.

>[!CAUTION]
>
>Om olika åtkomstkrav gäller för olika delar av webbplatsen måste du definiera flera grupper.

**/sessionmanagement** har flera underparametrar:

**/directory** (obligatoriskt)

Katalogen som lagrar sessionsinformationen. Om katalogen inte finns skapas den.

>[!CAUTION]
>
> När du konfigurerar katalogunderparametern **inte** peka på rotmappen (`/directory "/"`) eftersom det kan orsaka allvarliga problem. Ange alltid sökvägen till den mapp som lagrar sessionsinformationen. Till exempel:

```xml
/sessionmanagement
  {
  /directory "/usr/local/apache/.sessions"
  }
```

**/encode** (valfritt)

Hur sessionsinformationen kodas. Använd `md5` för kryptering med md5-algoritmen, eller `hex` för hexadecimal kodning. Om du krypterar sessionsdata kan en användare med åtkomst till filsystemet inte läsa sessionsinnehållet. Standardvärdet är `md5`.

**/header** (valfritt)

Namnet på HTTP-huvudet eller cookien som lagrar auktoriseringsinformationen. Om du lagrar informationen i http-huvudet använder du `HTTP:<header-name>`. Om du vill lagra informationen i en cookie använder du `Cookie:<header-name>`. Om du inte anger något värde `HTTP:authorization` används.

**/timeout** (valfritt)

Antalet sekunder tills sessionen tar slut efter att den har använts sist. Om inte angivet `"800"` används så sessionen tar lite längre tid än 13 minuter efter användarens senaste begäran.

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

The `/renders` -egenskapen definierar den URL som Dispatcher skickar begäranden till för att återge ett dokument. Följande exempel `/renders` används för att identifiera en enda AEM för återgivning:

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

Följande exempel `/renders` identifierar en AEM som körs på samma dator som Dispatcher:

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

Följande exempel `/renders` -avsnittet distribuerar återgivningsbegäranden lika mellan två AEM instanser:

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

Anger timeout för anslutningen som använder AEM i millisekunder. Standardvärdet är `"0"`, vilket får Dispatcher att vänta oändligt.

**/receiveTimeout**

Anger tiden i millisekunder som ett svar får ta. Standardvärdet är `"600000"`, vilket gör att Dispatcher väntar i 10 minuter. En inställning för `"0"` eliminerar tidsgränsen.

Om tidsgränsen nås när svarshuvuden tolkas returneras HTTP-statusen 504 (Felaktig gateway). Om tidsgränsen nås när svarstexten läses, returnerar Dispatcher det ofullständiga svaret till klienten. Eventuella cachelagrade filer som kan ha skrivits tas också bort.

**/ipv4**

Anger om Dispatcher använder `getaddrinfo` funktionen (för IPv6) eller `gethostbyname` funktion (för IPv4) för att hämta återgivningens IP-adress. Värdet 0 orsakar `getaddrinfo` som ska användas. Värdet för `1` orsaker `gethostbyname` som ska användas. Standardvärdet är `0`.

The `getaddrinfo` funktionen returnerar en lista med IP-adresser. Dispatcher itererar listan över adresser tills en TCP/IP-anslutning upprättas. Därför är `ipv4` egenskapen är viktig när återgivningsvärdnamnet är associerat med flera IP-adresser. Och värden, som svar på `getaddrinfo` returnerar en lista med IP-adresser som alltid finns i samma ordning. I sådana fall bör du använda `gethostbyname` så att den IP-adress som Dispatcher ansluter till är slumpmässig.

Amazon Elastic Load Balancing (ELB) är en tjänst som svarar på getaddrinfo med en lista över IP-adresser som kan vara i samma ordning.

**/secure**

Om `/secure` egenskapen har värdet `"1"`, används HTTPS för att kommunicera med AEM. Mer information finns i [Konfigurera Dispatcher att använda SSL](dispatcher-ssl.md#configuring-dispatcher-to-use-ssl).

**/always-resolve**

Med Dispatcher-version **4.1.6** kan du konfigurera `/always-resolve` egenskap enligt följande:

* När inställt på `"1"`åtgärdar det värdnamnet vid varje begäran (Dispatcher cachelagrar aldrig någon IP-adress). Det kan uppstå en liten prestandapåverkan på grund av det ytterligare anrop som krävs för att få värdinformation för varje begäran.
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

Använd `/filter` för att ange de HTTP-begäranden som Dispatcher accepterar. Alla andra begäranden skickas tillbaka till webbservern med felkoden 404 (sidan hittades inte). Om nej `/filter` finns, alla begäranden accepteras.

**Obs!** Begäranden [statfile](#naming-the-statfile) avvisas alltid.

>[!CAUTION]
>
>Se [Dispatcher Security Checklist](security-checklist.md) om du vill ha mer information när du begränsar åtkomsten med AEM Dispatcher. Läs även [AEM](https://experienceleague.adobe.com/en/docs/experience-manager-65/content/security/security-checklist#security) om du vill ha mer säkerhetsinformation om AEM.

The `/filter` -avsnittet består av en serie regler som antingen nekar eller tillåter åtkomst till innehåll enligt mönster i begärandoradsdelen av HTTP-begäran. Använd en tillåtelselista-strategi för `/filter` avsnitt:

* Förhindra åtkomst till allt.
* Tillåt åtkomst till innehåll efter behov.

>[!NOTE]
>
>Rensa cacheminnet när filterreglerna ändras.

### Definiera ett filter {#defining-a-filter}

Varje objekt i `/filter` -avsnittet innehåller en typ och ett mönster som matchas med ett visst element på begärandaden eller hela begärandraden. Varje filter kan innehålla följande objekt:

* **Typ**: `/type` anger om åtkomst ska beviljas eller nekas för begäranden som matchar mönstret. Värdet kan vara antingen `allow` eller `deny`.

* **Element i Request Line:** Inkludera `/method`, `/url`, `/query`, eller `/protocol`. Ta även med ett mönster för filtreringsbegäranden. Filtrera dem efter specifika delar av begärandoradsdelen i HTTP-begäran. Filtrering för element på begärandraden (i stället för på hela begärandraden) är den föredragna filtermetoden.

* **Avancerade element i begäranderaden:** Från och med Dispatcher 4.2.0 finns fyra nya filterelement att använda. De nya elementen är `/path`, `/selectors`, `/extension`och `/suffix` respektive. Inkludera ett eller flera av dessa objekt för ytterligare kontroll av URL-mönster.

>[!NOTE]
>
>Mer information om vilken del av förfrågningsraden som var och en av dessa element refererar till finns i [Sling URL-disposition](https://sling.apache.org/documentation/the-sling-engine/url-decomposition.html) wiki-sida.

* **globegenskaper**: `/glob` -egenskapen används för att matcha hela begäranderaden för HTTP-begäran.

>[!CAUTION]
>
>Filtrering med glober är föråldrat i Dispatcher. Därför bör du undvika att använda globala ikoner i `/filter` eftersom det kan leda till säkerhetsproblem. I stället för:
>
>`/glob "* *.css *"`
>
>Använd
>
>`/url "*.css"`

#### Begärandelen av HTTP-begäranden {#the-request-line-part-of-http-requests}

HTTP/1.1 definierar [request-line](https://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html) enligt följande:

`Method Request-URI HTTP-Version<CRLF>`

The `<CRLF>` tecken representerar en vagnretur följt av en radmatning. Följande exempel är den frågerad som tas emot när en klient begär sidan amerikansk-engelska på WKND-webbplatsen:

`GET /content/wknd/us/en.html HTTP.1.1<CRLF>`

Dina mönster måste ta hänsyn till blankstegstecknen på förfrågningsraden och `<CRLF>` tecken.

#### Dubbla citattecken jämfört med enkla citattecken {#double-quotes-vs-single-quotes}

Använd citattecken när du skapar filterregler `"pattern"` för enkla mönster. Om du använder Dispatcher 4.2.0 eller senare och mönstret innehåller ett reguljärt uttryck, måste du omsluta regex-mönstret `'(pattern1|pattern2)'` inom enkla citattecken.

#### Reguljära uttryck {#regular-expressions}

I Dispatcher-versioner senare än 4.2.0 kan du inkludera utökade reguljära uttryck för POSIX i dina filtermönster.

#### Felsöka filter {#troubleshooting-filters}

Om dina filter inte aktiveras på det sätt du förväntar dig ska du aktivera [Spårningsloggning](#trace-logging) på Dispatcher så att du kan se vilket filter som fångar upp begäran.

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

#### Exempelfilter: Aktivera begäranden om POST {#example-filter-enable-post-requests}

Följande exempelfilter tillåter att formulärdata skickas med metoden POST:

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

Nedan visas ett regelexempel som blockerar innehåll som hämtas från `/content` sökväg och dess underträd, med filter för sökväg, väljare och tillägg:

```xml
/006 {
        /type "deny"
        /path "/content/*"
        /selectors '(feed|rss|pages|languages|blueprint|infinity|tidy|sysview|docview|query|jcr:content|_jcr_content|search|childrenlist|ext|assets|assetsearch|[0-9-]+)'
        /extension '(json|xml|html|feed))'
        }
```

### Exempel `/filter` section {#example-filter-section}

När du konfigurerar Dispatcher bör du begränsa den externa åtkomsten så mycket som möjligt. Följande exempel ger minimal åtkomst för externa besökare:

* `/content`
* olika innehåll, t.ex. design och klientbibliotek. Till exempel:

   * `/etc/designs/default*`
   * `/etc/designs/mydesign*`

När du har skapat filter [testsidåtkomst](#testing-dispatcher-security) för att säkerställa att din AEM är säker.

Följande `/filter` i `dispatcher.any` -filen kan användas som bas i [Dispatcher-konfigurationsfil.](#dispatcher-configuration-files)

Det här exemplet baseras på den standardkonfigurationsfil som medföljer Dispatcher och är avsedd som exempel för användning i en produktionsmiljö. Objekt med prefix `#` inaktiveras (kommenteras ut). Du bör vara försiktig om du bestämmer dig för att aktivera någon av dessa saker (genom att ta bort `#` på den raden). Om du gör det kan det påverka säkerheten.

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
>När du använder Apache utformar du dina filter-URL-mönster enligt egenskapen DispatcherUseProcsedURL i modulen Dispatcher. (Se [Apache Web Server - Konfigurera Apache Web Server för Dispatcher](dispatcher-install.md##apache-web-server-configure-apache-web-server-for-dispatcher).)

<!----
>[!NOTE]
>
>Filters `0030` and `0031` regarding Dynamic Media are applicable to AEM 6.0 and higher. -->

Tänk på följande om du väljer att utöka åtkomsten:

* Inaktivera extern åtkomst till `/admin` om du använder CQ version 5.4 eller tidigare.

* Var försiktig när du tillåter åtkomst till filer i `/libs`. Åtkomst bör tillåtas på individuell basis.
* Neka åtkomst till replikeringskonfigurationen så att den inte kan ses:

   * `/etc/replication.xml*`
   * `/etc/replication.infinity.json*`

* Neka åtkomst till Google Gadgets omvänd proxy:

   * `/libs/opensocial/proxy*`

Beroende på installationen kan det finnas fler resurser under `/libs`, `/apps` eller någon annanstans, som måste göras tillgänglig. Du kan använda `access.log` som en metod för att avgöra resurser som används externt.

>[!CAUTION]
>
>Åtkomst till konsoler och kataloger kan utgöra en säkerhetsrisk för produktionsmiljöer. Om du inte har en explicit motivering ska de förbli inaktiverade (kommenterade ut).

>[!CAUTION]
>
>Om du [använda rapporter i en publiceringsmiljö](https://experienceleague.adobe.com/en/docs/experience-manager-65/content/sites/administering/operations/reporting#using-reports-in-a-publish-environment)bör du konfigurera Dispatcher så att åtkomst till `/etc/reports` för externa besökare.

### Begränsa frågesträngar {#restricting-query-strings}

Sedan Dispatcher version 4.1.5 använder du `/filter` för att begränsa frågesträngar. Vi rekommenderar att du uttryckligen tillåter frågesträngar och exkluderar generiska reserveringar via `allow` filterelement.

En enskild post kan ha antingen `glob` eller en kombination av `method`, `url`, `query`och `version`, men inte båda. I följande exempel tillåts `a=*` frågesträng och nekar alla andra frågesträngar för URL:er som tolkas som `/etc` nod:

```xml
/filter {
 /0001 { /type "deny" /method "POST" /url "/etc/*" }
 /0002 { /type "allow" /method "GET" /url "/etc/*" /query "a=*" }
}
```

>[!NOTE]
>
>Om en regel innehåller en `/query`matchar den bara begäranden som innehåller en frågesträng och som matchar det angivna frågemönstret.
>
>I exemplet ovan, om begär att `/etc` som inte har någon frågesträng bör också tillåtas, måste följande regler anges:
>

```xml
/filter {  
>/0001 { /type "deny" /method "*" /url "/path/*" }  
>/0002 { /type "allow" /method "GET" /url "/path/*" }  
>/0003 { /type "deny" /method "GET" /url "/path/*" /query "*" }  
>/0004 { /type "allow" /method "GET" /url "/path/*" /query "a=*" }  
}  
```

### Testar Dispatcher Security {#testing-dispatcher-security}

Dispatcher-filter ska blockera åtkomst till följande sidor och skript AEM publiceringsinstanser. Använd en webbläsare för att försöka öppna följande sidor som en besökare skulle göra och verifiera att koden 404 returneras. Justera filtren om du får andra resultat.

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

Om du vill göra Dispatcher-cachen ogiltig och försäkra dig om att du får ett code 403-svar skickar du följande kommando i en terminal eller kommandotolk:

`curl -H "CQ-Handle: /content" -H "CQ-Path: /content" https://yourhostname/dispatcher/invalidate.cache`

## Aktivera åtkomst till Vanity-URL:er {#enabling-access-to-vanity-urls-vanity-urls}

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (sbroders@adobe.com)
Last Modified Date: 2015-03-25T14:23:05.185-0400

<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">For https://jira.corp.adobe.com/browse/DOC-4812</p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">The "com.adobe.granite.dispatcher.vanityurl.content" package needs to be made public before publishing this contnet.</p>
 -->

Konfigurera Dispatcher för att aktivera åtkomst till användar-URL:er som är konfigurerade för dina AEM.

När åtkomst till tillfälliga URL:er är aktiverat anropar Dispatcher regelbundet en tjänst som körs på återgivningsinstansen för att få en lista över tillfälliga URL:er. Dispatcher lagrar den här listan i en lokal fil. När en begäran om en sida nekas på grund av ett filter i `/filter` -avsnittet söker Dispatcher igenom listan över användar-URL:er. Om den nekade URL:en finns med i listan tillåter Dispatcher åtkomst till den nekade URL:en.

Om du vill aktivera åtkomst till tillfälliga URL:er lägger du till en `/vanity_urls` till `/farms` -avsnittet, som i följande exempel:

```xml
 /vanity_urls {
      /url "/libs/granite/dispatcher/content/vanityUrls.html"
      /file "/tmp/vanity_urls"
      /delay 300
 }
```

The `/vanity_urls` -avsnittet innehåller följande egenskaper:

* `/url`: Sökvägen till den vanity URL-tjänst som körs på återgivningsinstansen. Värdet för den här egenskapen måste vara `"/libs/granite/dispatcher/content/vanityUrls.html"`.

* `/file`: Sökvägen till den lokala filen där Dispatcher lagrar listan över huvud-URL:er. Kontrollera att Dispatcher har skrivbehörighet till den här filen.
* `/delay`: (Sekunder) Tiden mellan anrop till tjänsten för huvud-URL.

>[!NOTE]
>
>Om återgivningen är en instans av AEM måste du installera [VanityURLS-Components-paket från Software Distribution](https://experience.adobe.com/#/downloads/content/software-distribution/en/aem.html?package=/content/software-distribution/en/details.html/content/dam/aem/public/adobe/packages/granite/vanityurls-components) för att aktivera tjänsten för huvud-URL. (Se [Programvarudistribution](https://experienceleague.adobe.com/en/docs/experience-manager-65/content/sites/administering/contentmanagement/package-manager#software-distribution) för mer information.)

Använd följande procedur för att aktivera åtkomst till mål-URL:er.

1. Om renderingstjänsten är en AEM instans installerar du `com.adobe.granite.dispatcher.vanityurl.content` paket på publiceringsinstansen (se anteckningen ovan).
1. För varje mål-URL som du har konfigurerat för en AEM- eller CQ-sida kontrollerar du att [`/filter`](#configuring-access-to-content-filter) konfiguration nekar URL:en. Om det behövs lägger du till ett filter som nekar URL-adressen.
1. Lägg till `/vanity_urls` avsnitt nedan `/farms`.
1. Starta om Apache-webbservern.

## Vidarebefordrar syndikeringsbegäranden - `/propagateSyndPost` {#forwarding-syndication-requests-propagatesyndpost}

Syndikeringsbegäranden är endast avsedda för Dispatcher, så som standard skickas de inte till återgivaren (till exempel en AEM).

Om det behövs anger du `/propagateSyndPost` egenskap till `"1"` för att vidarebefordra syndikeringsbegäranden till Dispatcher. Om den är inställd måste du se till att begäranden om POST inte nekas i filteravsnittet.

## Konfigurera Dispatcher Cache - `/cache` {#configuring-the-dispatcher-cache-cache}

The `/cache` styr hur Dispatcher cachelagrar dokument. Konfigurera flera underegenskaper för att implementera dina cachningsstrategier:

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
>För behörighetskänslig cachelagring, läs [Cachelagra skyddat innehåll](permissions-cache.md).

### Ange cachekatalogen {#specifying-the-cache-directory}

The `/docroot` identifierar katalogen där cachelagrade filer lagras.

>[!NOTE]
>
>Värdet måste vara samma sökväg som dokumentroten på webbservern så att Dispatcher och webbservern hanterar samma filer.\
>Webbservern ansvarar för att leverera rätt statuskod när Dispatcher-cachefilen används, och det är därför det är viktigt att den också kan hitta den.

Om du använder flera grupper måste varje grupp ha en annan dokumentrot.

### Namnge statusfilen {#naming-the-statfile}

The `/statfile` egenskapen identifierar filen som ska användas som statfile. Dispatcher använder den här filen för att registrera tidpunkten för den senaste innehållsuppdateringen. Statfile kan vara vilken fil som helst på webbservern.

Statusfilen har inget innehåll. När innehållet uppdateras uppdaterar Dispatcher tidsstämpeln. Standardstatusfilen heter `.stat` och lagras i dokumentet. Dispatcher blockerar åtkomsten till statusfilen.

>[!NOTE]
>
>If `/statfileslevel` är konfigurerad ignorerar Dispatcher `/statfile` egenskap och användning `.stat` som namn.

### Hantera gamla dokument när fel uppstår {#serving-stale-documents-when-errors-occur}

The `/serveStaleOnError` anger om Dispatcher returnerar ogiltiga dokument när återgivningsservern returnerar ett fel. Som standard tas det cachelagrade innehållet bort när en lägesfil ändras och det cachelagrade innehållet blir ogiltigt. Den här åtgärden utförs nästa gång den begärs.

If `/serveStaleOnError` är inställd på `"1"`, tar inte Dispatcher bort ogiltigt innehåll från cachen. Det vill säga, om inte återgivningsservern returnerar ett lyckat svar. Ett 5xx-svar från AEM eller en timeout för anslutningen gör att Dispatcher skickar det inaktuella innehållet och svarar med och HTTP-statusen 111 (förnyelsen misslyckades).

### Cachelagring när autentisering används {#caching-when-authentication-is-used}

The `/allowAuthorized` egenskapen styr om begäranden som innehåller någon av följande autentiseringsinformation cachelagras:

* The `authorization` header
* En cookie med namnet `authorization`
* En cookie med namnet `login-token`

Som standard cachelagras inte begäranden som innehåller den här autentiseringsinformationen eftersom autentiseringen inte utförs när ett cachelagrat dokument returneras till klienten. Den här konfigurationen förhindrar att Dispatcher skickar cachelagrade dokument till användare som inte har nödvändiga rättigheter.

Om dina krav tillåter cachelagring av autentiserade dokument, anger du `/allowAuthorized` till en:

`/allowAuthorized "1"`

>[!NOTE]
>
>Aktivera sessionshantering (med `/sessionmanagement` ), `/allowAuthorized` egenskapen måste anges till `"0"`.

### Ange vilka dokument som ska cachelagras {#specifying-the-documents-to-cache}

The `/rules` anger vilka dokument som cachelagras enligt dokumentets sökväg. Oavsett `/rules` egenskapen, Dispatcher cachelagrar aldrig ett dokument under följande omständigheter:

* URI för begäran innehåller ett frågetecken (`?`).
   * Anger en dynamisk sida, till exempel ett sökresultat som inte behöver cachas.
* Filtillägget saknas.
   * Webbservern behöver tillägget för att kunna avgöra dokumenttypen (MIME-typen).
* Autentiseringshuvudet är inställt (konfigurerbart).
* Om AEM svarar med följande rubriker:

   * `no-cache`
   * `no-store`
   * `must-revalidate`

>[!NOTE]
>
>Metoderna GET eller HEAD (för HTTP-huvudet) kan nås av Dispatcher. Mer information om cachelagring av svarshuvuden finns i [Cachelagra HTTP-svarshuvuden](#caching-http-response-headers) -avsnitt.

Varje objekt i `/rules` egenskapen innehåller [`glob`](#designing-patterns-for-glob-properties) mönster och en typ:

* The `glob` mönster används för att matcha dokumentets sökväg.
* Typen anger om de dokument som matchar `glob` mönster. Värdet kan vara `allow` (cachelagra dokumentet) eller `deny` (återge dokumentet).

Om du inte har dynamiska sidor (utöver de sidor som redan har undantagits av ovanstående regler) kan du konfigurera Dispatcher så att allt cachelagras. Regelavsnittet ser ut så här:

```xml
/rules
  {
    /0000  {  /glob "*"   /type "allow" }
  }
```

Mer information om Glob-egenskaper finns i [Designa mönster för globala egenskaper](#designing-patterns-for-glob-properties).

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

På Apache-webbservrar kan du komprimera de cachelagrade dokumenten. Komprimering gör att Apache kan returnera dokumentet i ett komprimerat format om klienten begär det. Komprimering utförs automatiskt genom att Apache-modulen aktiveras `mod_deflate`, till exempel:

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

Använd `/statfileslevel` egenskap som gör cachelagrade filer ogiltiga enligt deras sökväg:

* Dispatcher skapar `.stat`filer i varje mapp från mappen docroot till den nivå du anger. Dokumentmappen är nivå 0.
* Filerna blir ogiltiga genom att röra vid `.stat` -fil. The `.stat` filens senaste ändringsdatum jämförs med det senaste ändringsdatumet för ett cachelagrat dokument. Dokumentet uppdateras om `.stat` filen är nyare.

* När en fil på en viss nivå ogiltigförklaras, **alla** `.stat` filer från dokumentet **till** den ogiltiga filens nivå eller den konfigurerade `statsfilevel` (beroende på vilket som är minst) rörs.

   * Om du till exempel anger `statfileslevel` egenskapen till 6 och en fil ogiltigförklaras på nivå 5, därefter var `.stat` filen från docroot till 5 ändras. Om du fortsätter med det här exemplet och en fil blir ogiltig på nivå 7 är varje `stat` filen från docroot till sex ändras (eftersom `/statfileslevel = "6"`).

Endast resurser **längs banan** till den ogiltiga filen påverkas. Titta på följande exempel: en webbplats använder strukturen `/content/myWebsite/xx/.` Om du anger `statfileslevel` as 3, a `.stat`filen skapas enligt följande:

* `docroot`
* `/content`
* `/content/myWebsite`
* `/content/myWebsite/*xx*`

När en fil `/content/myWebsite/xx` blir ogiltigt, sedan var `.stat` från dokumentroten ned till `/content/myWebsite/xx`är rörd. Detta scenario gäller endast för `/content/myWebsite/xx` och inte till exempel `/content/myWebsite/yy` eller `/content/anotherWebSite`.

>[!NOTE]
>
>Du kan förhindra invalidering genom att skicka en extra rubrik `CQ-Action-Scope:ResourceOnly`. Den här metoden kan användas för att tömma vissa resurser utan att andra delar av cachen blir ogiltiga. Se [den här sidan](https://adobe-consulting-services.github.io/acs-aem-commons/features/dispatcher-flush-rules/index.html) och [Invalidera Dispatcher-cachen manuellt](https://experienceleague.adobe.com/en/docs/experience-manager-dispatcher/using/configuring/page-invalidate#configuring) om du vill ha mer information.

>[!NOTE]
>
>Om du anger ett värde för `/statfileslevel` -egenskapen, `/statfile` egenskapen ignoreras.

### Automatisk invalidering av cachelagrade filer {#automatically-invalidating-cached-files}

The `/invalidate` -egenskapen definierar de dokument som automatiskt görs ogiltiga när innehållet uppdateras.

Med automatisk ogiltigförklaring tar inte Dispatcher bort cachelagrade filer efter en innehållsuppdatering, men kontrollerar deras giltighet nästa gång de begärs. Dokument i cacheminnet som inte ogiltigförklaras automatiskt finns kvar i cacheminnet tills en innehållsuppdatering tar bort dem explicit.

Automatisk ogiltigförklaring används vanligtvis för HTML-sidor. HTML sidor innehåller ofta länkar till andra sidor, vilket gör det svårt att avgöra om en innehållsuppdatering påverkar en sida. Om du vill vara säker på att alla relevanta sidor blir ogiltiga när innehållet uppdateras, gör du alla sidor i HTML automatiskt ogiltiga. Följande konfiguration gör alla HTML sidor ogiltiga:

```xml
  /invalidate
  {
   /0000  { /glob "*" /type "deny" }
   /0001  { /glob "*.html" /type "allow" }
  }
```

Mer information om Glob-egenskaper finns i [Designa mönster för globala egenskaper](#designing-patterns-for-glob-properties).

Den här konfigurationen orsakar följande aktivitet när `/content/wknd/us/en` är aktiverad:

* Alla filer med mönstret en.* tas bort från `/content/wknd/us` mapp.
* The `/content/wknd/us/en./_jcr_content` mappen tas bort.
* Alla andra filer som matchar `/invalidate` -konfigurationen tas inte bort omedelbart. Dessa filer tas bort när nästa begäran görs. I exemplet `/content/wknd.html` tas inte bort; den tas bort när `/content/wknd.html` har begärts.

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

AEM integrering med Adobe Analytics ger konfigurationsdata i en `analytics.sitecatalyst.js` på din webbplats. Exemplet `dispatcher.any` filen som medföljer Dispatcher innehåller följande ogiltighetsregel för den här filen:

```xml
{
   /glob "*/analytics.sitecatalyst.js"  /type "allow"
}
```

### Använda anpassade ogiltighetsskript {#using-custom-invalidation-scripts}

The `/invalidateHandler` Med -egenskapen kan du definiera ett skript som anropas för varje invalideringsbegäran som tas emot av Dispatcher.

Den anropas med följande argument:

* Handtag - Innehållssökvägen som är ogiltig
* Åtgärd - Replikeringsåtgärden (till exempel Aktivera, Inaktivera)
* Åtgärdsomfång - Replikeringsåtgärdens omfång (tom, såvida inte en rubrik för `CQ-Action-Scope: ResourceOnly` skickas, se [Invaliderar cachelagrade sidor från AEM](page-invalidate.md) för mer information)

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

The `/allowedClients` -egenskapen definierar specifika klienter som tillåts tömma cachen. Globeringsmönstren matchas mot IP.

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

Mer information om Glob-egenskaper finns i [Designa mönster för globala egenskaper](#designing-patterns-for-glob-properties).

>[!CAUTION]
>
>Vi rekommenderar att du definierar `/allowedClients`.
>
>Om detta inte görs kan alla klienter utfärda ett anrop för att rensa cachen. Om du gör det flera gånger kan det påverka webbplatsens prestanda negativt.

### Ignorerar URL-parametrar {#ignoring-url-parameters}

The `ignoreUrlParams` -avsnittet anger vilka URL-parametrar som ska ignoreras när du avgör om en sida cachelagras eller levereras från cachen:

* När en begärande URL innehåller parametrar som alla ignoreras, cachelagras sidan.
* När en begärande URL innehåller en eller flera parametrar som inte ignoreras, cachelagras inte sidan.

När en parameter ignoreras för en sida cachelagras sidan första gången som sidan begärs. Efterföljande begäranden för sidan skickas till den cachelagrade sidan, oavsett värdet på parametern i begäran.

>[!NOTE]
>
>Vi rekommenderar att du konfigurerar `ignoreUrlParams` ställa in tillåtelselista. Därför ignoreras alla frågeparametrar och endast kända eller förväntade frågeparametrar undantas (&quot;nekas&quot;) från att ignoreras. Mer information och exempel finns i [den här sidan](https://github.com/adobe/aem-dispatcher-optimizer-tool/blob/main/docs/Rules.md#dot---the-dispatcher-publish-farm-cache-should-have-its-ignoreurlparams-rules-configured-in-an-allow-list-manner).

Om du vill ange vilka parametrar som ska ignoreras lägger du till regler i `ignoreUrlParams` egenskap:

* Om du vill cachelagra en sida oavsett vilken begäran som innehåller en URL-parameter, skapar du en glob-egenskap som tillåter parametern (att ignoreras).
* Om du vill förhindra att sidan cachas skapar du en globegenskap som nekar parametern (som ignoreras).

>[!NOTE]
>
>När du konfigurerar glob-egenskapen bör den matcha frågeparameternamnet. Om du till exempel vill ignorera parametern &quot;p1&quot; från följande URL `http://example.com/path/test.html?p1=test&p2=v2`ska globegenskapen vara:
> `/0002 { /glob "p1" /type "allow" }`

I följande exempel ignoreras alla parametrar utom `nocache` parameter. Därför cachelagrar Dispatcher aldrig URL:er som innehåller `nocache` parameter:

```xml
/ignoreUrlParams
{
    # ignore-all-url-parameters-by-dispatcher-and-requests-are-cached
    /0001 { /glob "*" /type "allow" }
    # allow-the-url-parameter-nocache-to-bypass-dispatcher-on-every-request
    /0002 { /glob "nocache" /type "deny" }
}
```

Med tanke på `ignoreUrlParams` i konfigurationsexemplet ovan gör följande HTTP-begäran att sidan cachelagras eftersom `willbecached` parametern ignoreras:

```xml
GET /mypage.html?willbecached=true
```

Med tanke på `ignoreUrlParams` konfigurationsexempel: följande HTTP-begäran orsakar sidan **not** som ska cachas eftersom `nocache` parametern ignoreras inte:

```xml
GET /mypage.html?nocache=true
GET /mypage.html?nocache=true&willbecached=true
```

Mer information om Glob-egenskaper finns i [Designa mönster för globala egenskaper](#designing-patterns-for-glob-properties).

### Cachelagra HTTP-svarshuvuden {#caching-http-response-headers}

>[!NOTE]
>
>Den här funktionen är tillgänglig med version **4.1.11** för Dispatcher.

The `/headers` Med -egenskapen kan du definiera de HTTP-huvudtyper som Dispatcher ska cachelagra. Vid den första begäran till en icke cachelagrad resurs lagras alla huvuden som matchar ett av de konfigurerade värdena (se konfigurationsexemplet nedan) i en separat fil bredvid cachefilen. Vid efterföljande begäranden till den cachelagrade resursen läggs de lagrade rubrikerna till i svaret.

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
>Det är inte tillåtet att använda ordningstecken. Mer information finns i [Designa mönster för globala egenskaper](#designing-patterns-for-glob-properties).

>[!NOTE]
>
>Om du vill att Dispatcher ska lagra och leverera ETag-svarshuvuden från AEM gör du följande:
>
>* Lägg till rubriknamnet i `/cache/headers`-avsnitt.
>* Lägg till följande [Apache-direktiv](https://httpd.apache.org/docs/2.4/mod/core.html#fileetag) i avsnittet Dispatcher-relaterat:
>
>```xml
>FileETag none
>```

### Cachefilbehörigheter för utskickare {#dispatcher-cache-file-permissions}

The `mode` anger vilken filbehörighet som ska tillämpas på nya kataloger och filer i cachen. The `umask` för anropsprocessen begränsar den här inställningen. Det är ett oktalt tal som konstruerats av summan av ett eller flera av följande värden:

* `0400` Tillåt läsning av ägare.
* `0200` Tillåt skrivning efter ägare.
* `0100` Tillåt ägaren att söka i kataloger.
* `0040` Tillåt läsning av gruppmedlemmar.
* `0020` Tillåt skrivning av gruppmedlemmar.
* `0010` Tillåt gruppmedlemmar att söka i katalogen.
* `0004` Tillåt läsning av andra.
* `0002` Tillåt andra att skriva.
* `0001` Tillåt andra att söka i katalogen.

Standardvärdet är `0755`, som gör det möjligt för ägaren att läsa, skriva eller söka och för gruppen och andra att läsa eller söka i.

### Begränsar .stat-filberöring {#throttling-stat-file-touching}

Med standardinställningen `/invalidate` egenskapen, all aktivering gör alla `.html` filer (när deras sökväg matchar `/invalidate` ). På en webbplats med stor trafik ökar multipla aktiveringar processorbelastningen på backend-sidan. I sådana fall är det önskvärt att strypa `.stat` filhantering för att hålla webbplatsen responsiv. Du kan utföra den här åtgärden med `/gracePeriod` -egenskap.

The `/gracePeriod` anger hur många sekunder en inaktuell, automatiskt ogiltigförklarad resurs fortfarande kan betjänas från cachen efter den senaste aktiveringen. Egenskapen kan användas i en inställning där en grupp av aktiveringar annars skulle göra hela cachen ogiltig upprepade gånger. Det rekommenderade värdet är 2 sekunder.

Mer information finns i `/invalidate` och `/statfileslevel`tidigare.

### Konfigurerar tidsbaserad cacheinvalidering - `/enableTTL` {#configuring-time-based-cache-invalidation-enablettl}

Tidsbaserad cacheogiltigförklaring beror på `/enableTTL` och det finns regelbundna förfallorubriker från HTTP-standarden. Om du ställer in egenskapen på 1 (`/enableTTL "1"`) utvärderas svarshuvuden från serverdelen. Om sidhuvudena innehåller en `Cache-Control`, `max-age` eller `Expires` datum skapas en extra, tom fil bredvid den cachelagrade filen med samma ändringsdatum som förfallodatumet. När den cachelagrade filen har begärts efter ändringstiden återbegärs den automatiskt från serverdelen.

Före Dispatcher 4.3.5 baserades logiken för TTL-ogiltigförklaring endast på det konfigurerade TTL-värdet. Med Dispatcher 4.3.5, båda de angivna TTL-värdena **och** Invalideringsreglerna för Dispatcher-cachen räknas med. För en cachelagrad fil:

1. If `/enableTTL` anges till 1, filens förfallodatum kontrolleras. Om filen har gått ut enligt angiven TTL utförs inga andra kontroller och den cachelagrade filen efterfrågas igen från serverdelen.
2. Om filen inte har gått ut, eller `/enableTTL` inte har konfigurerats, tillämpas standardreglerna för ogiltigförklaring av cachen, till exempel de regler som [`/statfileslevel`](#invalidating-files-by-folder-level) och [`/invalidate`](#automatically-invalidating-cached-files) set. Det här flödet innebär att Dispatcher kan göra filer som TTL-värdet inte har gått ut för ogiltiga.

Den nya implementeringen stöder användning där filerna har längre TTL-värden (till exempel CDN). Men filen kan fortfarande ogiltigförklaras även om TTL-värdet inte har gått ut. Det prioriterar innehållets aktualitet framför cache-träffkvoten för Dispatcher.

Omvänt, om du behöver **endast** förfallologiken som tillämpas på en fil och ange sedan `/enableTTL` till 1 och exkludera den filen från standardcacheminnets invalideringsmekanism. Du kan till exempel:

* Om du vill ignorera filen konfigurerar du [ogiltigförklara regler](#automatically-invalidating-cached-files) i cacheavsnittet. I kodutdraget nedan slutar alla filer i `.example.html` ignoreras och förfaller bara när den angivna TTL:en har passerats.

```xml
  /invalidate
  {
   /0000  { /glob "*" /type "deny" }
   /0001  { /glob "*.html" /type "allow" }
   /0002  { /glob "*.example.html" /type "deny" }
  }
```

* Utforma innehållsstrukturen på ett sådant sätt att du kan ställa in en hög [`/statfilelevel`](#invalidating-files-by-folder-level) så filen inte blir automatiskt ogiltig.

På så sätt säkerställer du att `.stat` filogiltigförklaring används inte och endast TTL-förfallodatum är aktivt för de angivna filerna.

>[!NOTE]
>
>Kom ihåg att inställningen `/enableTTL` till 1 aktiverar endast TTL-cachelagring på Dispatcher-sidan. TTL-informationen i den kompletterande filen (se ovan) tillhandahålls inte till någon annan användaragent som begär en sådan filtyp från Dispatcher. Om du vill ge cachelagringshuvuden till underordnade system som ett CDN eller en webbläsare, bör du konfigurera `/cache/headers` i enlighet med detta.

>[!NOTE]
>
>Den här funktionen är tillgänglig i version **4.1.11** eller senare av Dispatcher.

## Konfigurerar belastningsutjämning - `/statistics` {#configuring-load-balancing-statistics}

The `/statistics` -avsnittet definierar de filkategorier för vilka Dispatcher bedömer svarstiden för varje återgivning. Dispatcher använder poängen för att avgöra vilken rendering som ska skickas.

Varje kategori som du skapar definierar ett globmönster. Dispatcher jämför URI:n för det begärda innehållet med dessa mönster för att fastställa kategorin för det begärda innehållet:

* Kategoriernas ordning avgör i vilken ordning de jämförs med URI:n.
* Det första kategorimönstret som matchar URI:n är filens kategori. Inga fler kategorimönster utvärderas.

Dispatcher stöder högst åtta statistikkategorier. Om du definierar fler än åtta kategorier används bara de första 8.

**Återge markering**

Varje gång Dispatcher kräver en återgiven sida används följande algoritm för att välja återgivningen:

1. Om begäran innehåller återgivningsnamnet i en `renderid` cookie, Dispatcher använder den renderingen.
1. Om begäran inte innehåller `renderid` cookie, Dispatcher jämför återgivningsstatistiken:

   1. Dispatcher bestämmer kategorin för begärande-URI.
   1. Dispatcher avgör vilken rendering som har det lägsta svarspoängen för den kategorin och väljer vilken rendering som ska användas.

1. Om ingen rendering är markerad än använder du den första renderingen i listan.

Poängen för en återgivnings kategori baseras på tidigare svarstider och tidigare misslyckade och lyckade anslutningar som Dispatcher försöker utföra. För varje försök uppdateras poängen för kategorin för den begärda URI:n.

>[!NOTE]
>
>Om du inte använder belastningsutjämning kan du utelämna det här avsnittet.

### Definiera statistikkategorier {#defining-statistics-categories}

Definiera en kategori för varje dokumenttyp som du vill behålla statistik för för återgivningsmarkering. The `/statistics` -avsnittet innehåller en `/categories` -avsnitt. Om du vill definiera en kategori lägger du till en rad under `/categories` som har följande format:

`/name { /glob "pattern"}`

Kategorin `name` måste vara unik för servergruppen. The `pattern` beskrivs i [Designa mönster för globegenskaper](#designing-patterns-for-glob-properties) -avsnitt.

För att fastställa kategorin för en URI jämför Dispatcher URI:n med varje kategorimönster tills en matchning hittas. Dispatcher börjar med den första kategorin i listan och fortsätter i rätt ordning. Därför bör du placera kategorier med mer specifika mönster först.

Till exempel Dispatcher som standard `dispatcher.any` -filen definierar en HTML-kategori och en annan kategori. Kategorin HTML är mer specifik och visas därför först:

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

### Speglar serverns otillgänglighet i Dispatcher-statistik {#reflecting-server-unavailability-in-dispatcher-statistics}

The `/unavailablePenalty` anger den tid (i tiondelar av en sekund) som används för återgivningsstatistiken när en anslutning till återgivningen misslyckas. Dispatcher lägger till tiden i statistikkategorin som matchar den begärda URI:n.

Straffet tillämpas till exempel när TCP/IP-anslutningen till det angivna värdnamnet/porten inte kan upprättas. Orsaken är antingen att AEM inte körs (och inte lyssnar) eller på grund av ett nätverksrelaterat problem.

The `/unavailablePenalty` egenskapen är direkt underordnad `/farm` -sektion (en jämställd `/statistics` ).

Om nej `/unavailablePenalty` egenskapen finns, värdet är `"1"` används.

```xml
/unavailablePenalty "1"
```

## Identifiera en mapp för fästig anslutning - `/stickyConnectionsFor` {#identifying-a-sticky-connection-folder-stickyconnectionsfor}

The `/stickyConnectionsFor` -egenskapen definierar en mapp som innehåller klisterlappande dokument. Den här egenskapen används med URL:en. Dispatcher skickar alla begäranden från en enskild användare som finns i den här mappen till samma återgivningsinstans. Anteckningar säkerställer att sessionsdata finns och är konsekventa för alla dokument. Den här funktionen använder `renderid` cookie.

I följande exempel definieras en fast anslutning till mappen /products:

```xml
/stickyConnectionsFor "/products"
```

När en sida består av innehåll från flera innehållsnoder inkluderar du `/paths` egenskap som listar sökvägarna till innehållet. En sida innehåller till exempel innehåll från `/content/image`, `/content/video`och `/var/files/pdfs`. Med följande konfiguration aktiveras häftiga anslutningar för allt innehåll på sidan:

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

När klisterlappande anslutningar är aktiverade ställer modulen Dispatcher in `renderid` cookie. Den här cookien har inte `httponly` som bör läggas till för att öka säkerheten. Du lägger till `httponly` flagga genom att ange `httpOnly` -egenskapen i `/stickyConnections` nod i en `dispatcher.any` konfigurationsfil. Egenskapens värde (antingen `0` eller `1`) definierar om `renderid` cookie har `HttpOnly` attribut tillagt. Standardvärdet är `0`, vilket innebär att attributet inte läggs till.

Mer information om `httponly` flagga, läsa [den här sidan](https://owasp.org/www-community/HttpOnly).

### `secure` {#secure}

När klisterlappande anslutningar är aktiverade ställer modulen Dispatcher in `renderid` cookie. Den här cookien har inte `secure` som bör läggas till för att öka säkerheten. Du lägger till `secure` flagga som anger `secure` -egenskapen i `/stickyConnections` nod i en `dispatcher.any` konfigurationsfil. Egenskapens värde (antingen `0` eller `1`) definierar om `renderid` cookie har `secure` attribut tillagt. Standardvärdet är `0`, vilket innebär att attributet läggs till **if** den inkommande begäran är säker. Om värdet är `1`, läggs flaggan secure till oavsett om den inkommande begäran är säker eller inte.

## Hantera återgivningsanslutningsfel {#handling-render-connection-errors}

Konfigurera Dispatcher-beteendet när återgivningsservern returnerar ett 500-fel eller inte är tillgänglig.

### Ange en hälsokontrollsida {#specifying-a-health-check-page}

Använd `/health_check` för att ange en URL som kontrolleras när en 500-statuskod inträffar. Om den här sidan även returnerar en 500-statuskod anses instansen vara otillgänglig och en konfigurerbar tidsåtgång ( `/unavailablePenalty`) används på återgivningen innan du försöker igen.

```xml
/health_check
  {
  # Page gets contacted when an instance returns a 500
  /url "/health_check.html"
  }
```

### Ange fördröjning för sidåterförsök {#specifying-the-page-retry-delay}

The `/retryDelay` anger tiden (i sekunder) som Dispatcher väntar mellan anslutningsförsök med servergruppsåtergivningar. För varje rund är det maximala antalet gånger Dispatcher försöker ansluta till en rendering antalet renderingar i servergruppen.

Dispatcher använder värdet `"1"` if `/retryDelay` är inte explicit definierad. Standardvärdet är vanligtvis lämpligt.

```xml
/retryDelay "1"
```

### Konfigurera antalet återförsök {#configuring-the-number-of-retries}

The `/numberOfRetries` egenskapen anger det maximala antalet anslutningsförsök som Dispatcher utför med återgivningarna. Om Dispatcher inte kan ansluta till en återgivning efter detta antal försök returnerar Dispatcher ett misslyckat svar.

För varje rund är det maximala antalet gånger Dispatcher försöker ansluta till en rendering antalet renderingar i servergruppen. Det maximala antalet gånger som Dispatcher försöker ansluta är därför ( `/numberOfRetries`) x (antalet återgivningar).

Om värdet inte definieras explicit är standardvärdet `5`.

```xml
/numberOfRetries "5"
```

### Använda mekanismen för växling vid fel {#using-the-failover-mechanism}

Om du vill skicka om begäranden till olika återgivningar när den ursprungliga begäran misslyckas, aktiverar du failover-funktionen i Dispatcher-gruppen. När failover är aktiverat har Dispatcher följande beteende:

* När en begäran om en återgivning returnerar HTTP-status 503 (UNAVAILABLE), skickar Dispatcher begäran till en annan återgivning.
* När en begäran till en återgivning returnerar HTTP-status 50x (annan än 503) skickar Dispatcher en begäran för sidan som är konfigurerad för `health_check` -egenskap.
   * Om hälsokontrollen returnerar 500 (INTERNAL_SERVER_ERROR) skickar Dispatcher den ursprungliga begäran till en annan rendering.
   * Om hälsokontrollen returnerar HTTP-status 200 returnerar Dispatcher det ursprungliga HTTP 500-felet till klienten.

Om du vill aktivera redundans lägger du till följande rad i servergruppen (eller webbplatsen):

```xml
/failover "1"
```

>[!NOTE]
>
>Om du vill försöka göra om HTTP-begäranden som innehåller en brödtext skickar Dispatcher en `Expect: 100-continue` begär sidhuvud till renderingen innan det faktiska innehållet mellanlagras. CQ 5.5 med CQSE besvarar sedan omedelbart med antingen 100 (CONTINUE) eller en felkod. Andra serverletsbehållare stöds också.

## Ignorerar avbrottsfel - `/ignoreEINTR` {#ignoring-interruption-errors-ignoreeintr}

>[!CAUTION]
>
>Det här alternativet behövs inte. Använd den bara när du ser följande loggmeddelanden:
>
>`Error while reading response: Interrupted system call`

Alla filsystemorienterade systemanrop kan avbrytas `EINTR` om objektet för systemanropet finns på ett fjärrsystem som nås via NFS. Huruvida dessa systemanrop kan ta slut eller avbrytas baseras på hur det underliggande filsystemet monterades på den lokala datorn.

Använd `/ignoreEINTR` om instansen har en sådan konfiguration och loggen innehåller följande meddelande:

`Error while reading response: Interrupted system call`

Internt läser Dispatcher svaret från fjärrservern (d.v.s. AEM) med en slinga som kan representeras som:

```text
while (response not finished) {  
read more data  
}
```

Sådana meddelanden kan genereras när `EINTR` inträffar i `read more data` -avsnitt. Och det är orsaken till att en signal tas emot innan data tas emot.

Om du vill ignorera sådana avbrott kan du lägga till följande parameter i `dispatcher.any` (före `/farms`):

`/ignoreEINTR "1"`

Inställning `/ignoreEINTR` till `"1"` gör att Dispatcher fortsätter att försöka läsa data tills det fullständiga svaret har lästs. Standardvärdet är `0` och inaktiverar alternativet.

## Designa mönster för globala egenskaper {#designing-patterns-for-glob-properties}

Flera avsnitt i Dispatcher-konfigurationsfilen använder `glob` egenskaper som urvalskriterier för klientförfrågningar. Värdena för `glob` egenskaper är mönster som Dispatcher jämför med en aspekt av begäran, till exempel sökvägen till den begärda resursen eller klientens IP-adress. Objekten i `/filter` avsnittsanvändning `glob` mönster för att identifiera sökvägarna för de sidor som Dispatcher agerar på eller avvisar.

The `glob` -värden kan innehålla jokertecken och alfanumeriska tecken för att definiera mönstret.

| Jokertecken | Beskrivning | Exempel |
|--- |--- |--- |
| `*` | Matchar noll eller flera intilliggande förekomster av ett tecken i strängen. Något av följande avgör det slutliga tecknet i matchningen: <br/>Ett tecken i strängen matchar nästa tecken i mönstret och mönstertecknet har följande egenskaper:<br/><ul><li>Inte en `*`</li><li>Inte en `?`</li><li>Ett literalt tecken (inklusive ett blanksteg) eller en teckenklass.</li><li>Slutet på mönstret nås.</li></ul>I en teckenklass tolkas tecknet bokstavligen. | `*/geo*` Matchar en sida under `/content/geometrixx` noden och `/content/geometrixx-outdoors` nod. Följande HTTP-begäranden matchar globmönstret: <br/><ul><li>`"GET /content/geometrixx/en.html"`</li><li>`"GET /content/geometrixx-outdoors/en.html"` </li></ul><br/> `*outdoors/*` <br/>Matchar en sida under `/content/geometrixx-outdoors` nod. Följande HTTP-begäran matchar till exempel globmönstret: <br/><ul><li>`"GET /content/geometrixx-outdoors/en.html"`</li></ul> |
| `?` | Matchar ett enskilt tecken. Använd teckenklasser utanför. I en teckenklass tolkas det här tecknet bokstavligen. | `*outdoors/??/*`<br/> Matchar sidorna för alla språk på den externa geometrixplatsen. Följande HTTP-begäran matchar till exempel globmönstret: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>Följande begäran matchar inte globmönstret: <br/><ul><li>&quot;GET /content/geometrixx-outdoors/en.html&quot;</li></ul> |
| `[ and ]` | Avmarkerar början och slutet av en teckenklass. Teckenklasser kan innehålla ett eller flera teckenintervall och enskilda tecken.<br/>En matchning inträffar om måltecknet matchar något av tecknen i teckenklassen, eller inom ett definierat intervall.<br/>Om den avslutande klammerparentesen inte inkluderas skapas inga matchningar i mönstret. | `*[o]men.html*`<br/> Matchar följande HTTP-begäran:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>Den matchar inte följande HTTP-begäran:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/> `*[o/]men.html*` <br/>Matchar följande HTTP-begäranden: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `-` | Det betecknar ett teckenintervall. För användning i teckenklasser. Utanför en teckenklass tolkas detta tecken bokstavligen. | `*[m-p]men.html*` Matchar följande HTTP-begäran: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul>Den matchar inte följande HTTP-begäran:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `!` | Negerar tecknet eller teckenklassen som följer. Använd bara för negerande tecken och teckenintervall inuti teckenklasser. Motsvarar `^ wildcard`. <br/>Utanför en teckenklass tolkas detta tecken bokstavligen. | `*[!o]men.html*`<br/> Matchar följande HTTP-begäran: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>Den matchar inte följande HTTP-begäran: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>`*[!o!/]men.html*`<br/> Den matchar inte följande HTTP-begäran:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"` eller `"GET /content/geometrixx-outdoors/en/men. html"`</li></ul> |
| `^` | Negerar tecknet eller teckenintervallet som följer. Används för att endast negera tecken och teckenintervall inuti teckenklasser. Motsvarar `!` jokertecken. <br/>Utanför en teckenklass tolkas detta tecken bokstavligen. | Exemplen för `!` jokertecken används, ersätta `!` tecken i exempelmönster med `^` tecken. |


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

Mer information finns i webbserverns dokumentation och i filen Viktigt för Dispatcher-instansen.

**Apache-roterade eller pipade loggar**

Om du använder **Apache** webbserver kan du använda standardfunktionerna för Loggrotation, Piped Logs eller båda. Använd till exempel pipade loggar:

`DispatcherLog "| /usr/apache/bin/rotatelogs logs/dispatcher.log%Y%m%d 604800"`

Den här funktionen roterar automatiskt:

* loggfilen Dispatcher, med en tidsstämpel i tillägget (`logs/dispatcher.log%Y%m%d`).
* varje vecka (60 x 60 x 24 x 7 = 604800 sekunder).

Se dokumentationen för Apache Web Server om loggrotation och Pipe-loggar. Till exempel: [Apache 2.4](https://httpd.apache.org/docs/2.4/logs.html).

>[!NOTE]
>
>Efter installationen är standardloggnivån hög (d.v.s. nivå 3 = Felsökning) så att Dispatcher loggar alla fel och varningar. Den här nivån är användbar i de inledande faserna.
>
>En sådan nivå kräver dock mer resurser. När Dispatcher fungerar som den ska *efter dina behov* kan du sänka loggnivån.

### Spårningsloggning {#trace-logging}

Bland andra förbättringar för Dispatcher finns även version 4.2.0 för Trace Logging.

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

Om du vill bekräfta grundläggande åtgärder och interaktion för webbservern, Dispatcher och AEM kan du utföra följande steg:

1. Ange `loglevel` till `3`.

1. Starta webbservern. När du gör det startas också Dispatcher.
1. Starta AEM.
1. Kontrollera loggen och felfilerna för webbservern och Dispatcher.
   * Beroende på webbservern bör du se meddelanden som:
      * `[Thu May 30 05:16:36 2002] [notice] Apache/2.0.50 (Unix) configured` och
      * `[Fri Jan 19 17:22:16 2001] [I] [19096] Dispatcher initialized (build XXXX)`

1. Surfa på webbplatsen via webbservern. Bekräfta att innehållet visas som det ska.\
   På en lokal installation där AEM körs på port `4502` och webbservern på `80` gå till webbsideskonsolen med båda:
   * `https://localhost:4502/libs/wcm/core/content/siteadmin.html`
   * `https://localhost:80/libs/wcm/core/content/siteadmin.html`
   * Resultaten ska vara identiska. Bekräfta åtkomst till andra sidor med samma mekanism.

1. Kontrollera att cachekatalogen fylls.
1. Aktivera en sida om du vill kontrollera att cacheminnet rensas korrekt.
1. Om allt fungerar som det ska kan du minska `loglevel` till `0`.

## Använda flera utskickare {#using-multiple-dispatchers}

I komplexa inställningar kan du använda flera Dispatcher. Du kan till exempel använda:

* en Dispatcher för att publicera en webbplats på intranätet
* en andra Dispatcher, under en annan adress och med olika säkerhetsinställningar, för att publicera samma innehåll på Internet.

I så fall måste du se till att varje begäran endast går igenom en Dispatcher. En Dispatcher hanterar inte begäranden som kommer från en annan Dispatcher. Kontrollera därför att båda utskickarna har direktåtkomst till den AEM webbplatsen.

## Felsökning {#debugging}

När du lägger till sidhuvudet `X-Dispatcher-Info` Vid en begäran besvarar Dispatcher om målet har cache-lagrats, returnerats från cache-lagrats eller inte kunnat cachelagras alls. Svarshuvudet `X-Cache-Info` innehåller informationen i läsbar form. Du kan använda de här svarshuvuden för att felsöka problem med svar som cachas av Dispatcher.

Den här funktionen är inte aktiverad som standard, så för svarshuvudet `X-Cache-Info` För att ingå måste jordbruksföretaget innehålla följande uppgifter:

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

Dessutom finns `X-Dispatcher-Info` sidhuvudet behöver inget värde, men om du använder `curl` För testning måste du ange ett värde som ska skickas till sidhuvudet, till exempel:

```xml
curl -v -H "X-Dispatcher-Info: true" https://localhost/content/wknd/us/en.html
```

Nedan finns en lista med de svarshuvuden som `X-Dispatcher-Info` returnerar:

* **målfil cachelagrad**\
  Målfilen finns i cachen och Dispatcher har fastställt att den kan levereras.
* **cachelagring**\
  Målfilen finns inte i cacheminnet och Dispatcher har fastställt att det går att cachelagra utdata och leverera dem.
* **cachelagring: statusfilen är nyare**
Målfilen finns i cachen. En senare statusfil kan dock göra den ogiltig. Dispatcher tar bort målfilen, återskapar den från utdata och levererar den.
* **inte tillgänglig: dokumentroten finns inte**
Servergruppens konfiguration innehåller inte någon dokumentrot (konfigurationselement) `cache.docroot`).
* **inte tillgänglig: cachefilens sökväg är för lång**\
  Målfilen - sammanfogningen av dokumentroten och URL-filen - överskrider det längsta möjliga filnamnet på systemet.
* **inte tillgänglig: temporär filsökväg för lång**\
  Mallen för tillfälligt filnamn överskrider det längsta möjliga filnamnet på systemet. Dispatcher skapar först en tillfällig fil innan den cachelagrade filen faktiskt skapas eller skrivs över. Det tillfälliga filnamnet är målfilens namn med tecknen `_YYYYXXXXXX` som läggs till i den, där `Y` och `X` ersätts för att skapa ett unikt namn.
* **inte tillgänglig: URL för begäran saknar tillägg**\
  Begärans URL har inget tillägg, eller så finns det en sökväg som följer filtillägget, till exempel: `/test.html/a/path`.
* **inte tillgänglig: begäran måste vara GET eller HEAD**
HTTP-metoden är inte GET eller HEAD. Dispatcher förutsätter att utdata innehåller dynamiska data som inte ska cachas.
* **inte tillgänglig: begäran innehöll en frågesträng**\
  Begäran innehöll en frågesträng. Dispatcher antar att utdata är beroende av frågesträngen som har angetts och därför inte cachelagras.
* **inte tillgänglig: sessionshanteraren måste autentisera**\
  En sessionshanterare (konfigurationen innehåller en `sessionmanagement` nod) styr servergruppens cache och begäran innehöll inte rätt autentiseringsinformation.
* **inte tillgänglig: begäran innehåller tillstånd**\
  Servergruppen kan inte cachelagra utdata ( `allowAuthorized 0`) och begäran innehåller autentiseringsinformation.
* **inte tillgänglig: målet är en katalog**\
  Målfilen är en katalog. Platsen kan peka på ett konceptuellt fel, där både en URL och en delwebbadress innehåller cachelagrade utdata. Om en begäran till exempel `/test.html/a/file.ext` kommer först och innehåller cachelagrade utdata. Dispatcher kan inte cachelagra utdata från en efterföljande begäran till `/test.html`.
* **inte tillgänglig: begäran-URL har ett avslutande snedstreck**\
  Begärans URL har ett avslutande snedstreck.
* **inte tillgänglig: begärande-URL saknas i cacheregler**\
  Servergruppens cacheregler tillåter explicit cachelagring av utdata från en begäran-URL.
* **inte tillgänglig: åtkomstkontroll nekad åtkomst**\
  Gruppens behörighetskontroll nekade åtkomst till den cachelagrade filen.
* **inte tillgänglig: sessionen är ogiltig**
En sessionshanterare (konfigurationen innehåller en `sessionmanagement` nod) styr servergruppens cache och användarens session är ogiltig eller inte längre giltig.
* **ej tillgänglig: svaret innehåller`no_cache`**
Fjärrservern returnerade en `Dispatcher: no_cache` header, förbjuder Dispatcher att cachelagra utdata.
* **inte tillgänglig: längden på svarsinnehållet är noll**
Svarets innehållslängd är noll. Dispatcher skapar inte en fil med längden noll.

