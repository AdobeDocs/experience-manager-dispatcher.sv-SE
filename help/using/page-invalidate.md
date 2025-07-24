---
title: Invalidera cachelagrade sidor från AEM
description: Lär dig hur du konfigurerar interaktionen mellan Dispatcher och AEM för att säkerställa effektiv cachehantering.
cmgrlastmodified: 01.11.2007 08 22 29 [aheimoz]
pageversionid: 1193211344162
template: /apps/docs/templates/contentpage
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
exl-id: 90eb6a78-e867-456d-b1cf-f62f49c91851
source-git-commit: c41b4026a64f9c90318e12de5397eb4c116056d9
workflow-type: tm+mt
source-wordcount: '1407'
ht-degree: 0%

---

# Invalidera cachelagrade sidor från AEM {#invalidating-cached-pages-from-aem}

När du använder Dispatcher med AEM måste interaktionen konfigureras för att säkerställa effektiv cachehantering. Beroende på din miljö kan konfigurationen även öka prestandan.

## Konfigurera AEM-användarkonton {#setting-up-aem-user-accounts}

Standardanvändarkontot `admin` används för att autentisera de replikeringsagenter som är installerade som standard. Skapa ett dedikerat användarkonto som kan användas med replikeringsagenter.

Mer information finns i avsnittet [Konfigurera replikerings- och transportanvändare](https://experienceleague.adobe.com/sv/docs/experience-manager-release-information/aem-release-updates/previous-updates/aem-previous-versions#VerificationSteps) i checklistan för AEM-säkerhet.

<!-- OLD URL from above https://helpx.adobe.com/se/experience-manager/6-3/sites/administering/using/security-checklist.html#VerificationSteps -->

## Invalidera Dispatcher-cachen från författarmiljön {#invalidating-dispatcher-cache-from-the-authoring-environment}

En replikeringsagent på AEM-författarinstansen skickar en cacheogiltigförklaring till Dispatcher när en sida publiceras. Dispatcher uppdaterar filen så småningom i cachen när nytt innehåll publiceras.

<!-- 

Comment Type: draft

<p>Cache invalidation requests for a page are also generated for any aliases or vanity URLs that are configured in the page properties. </p>

 -->

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-0436B4A35714BFF67F000101@AdobeID)
Last Modified Date: 2017-05-25T10:37:23.679-0400

<p>Hiding this information due to <a href="https://jira.corp.adobe.com/browse/CQDOC-5805">CQDOC-5805</a>.</p>

 -->

Använd följande procedur för att konfigurera en replikeringsagent på AEM-författarinstansen. Konfigurationen gör Dispatcher-cachen ogiltig vid sidaktivering:

1. Öppna AEM Tools Console. (`https://localhost:4502/miscadmin#/etc`)
1. Öppna den nödvändiga replikeringsagenten under Verktyg/replikering/Agenter på författare. Du kan använda Dispatcher Flush-agenten som är installerad som standard.
1. Klicka på Redigera och kontrollera att **Aktiverad** är markerat på fliken Inställningar.

1. (valfritt) Markera alternativet **Aliasuppdatering** om du vill aktivera ogiltiga aliassökvägar eller ogiltiga huvudsökvägar.
1. Gå till Dispatcher på fliken Transport genom att ange URI.

   Om du använder standardagenten för Dispatcher Flush uppdaterar du värdnamnet och porten, till exempel https://&lt;*dispatcherHost*>:&lt;*portApache*>/dispatcher/invalidate.cache

   **Obs!** För Dispatcher Flush-agenter används URI-egenskapen endast om du använder sökvägsbaserade virtuella värdposter för att skilja mellan grupper. Du använder det här fältet för att ange gruppen som ogiltig. Servergrupp nr 1 har till exempel en virtuell värd på `www.mysite.com/path1/*` och servergrupp nr 2 har en virtuell värd på `www.mysite.com/path2/*`. Du kan använda URL:en `/path1/invalidate.cache` för att ange den första servergruppen som mål och `/path2/invalidate.cache` för den andra servergruppen. Mer information finns i [Använda Dispatcher med flera domäner](dispatcher-domains.md).

1. Konfigurera andra parametrar efter behov.
1. Klicka på OK så att du kan aktivera agenten.

Du kan också komma åt och konfigurera Dispatcher Flush-agenten från [AEM Touch-gränssnittet](https://experienceleague.adobe.com/sv/docs/experience-manager-65/content/implementing/deploying/configuring/replication#configuring-a-dispatcher-flush-agent).

Mer information om hur du aktiverar åtkomst till mål-URL:er finns i [Aktivera åtkomst till Vanity-URL:er](dispatcher-configuration.md#enabling-access-to-vanity-urls-vanity-urls).

>[!NOTE]
>
>Agenten för tömning av Dispatcher-cachen behöver inget användarnamn och lösenord, men om den är konfigurerad skickas de med grundläggande autentisering.

Det finns två möjliga problem med den här metoden:

* Det måste gå att nå Dispatcher från utvecklingsinstansen. Om ditt nätverk (till exempel brandväggen) är konfigurerat så att åtkomsten mellan dessa två är begränsad, kanske detta inte är fallet.

* Publicering och cacheminnet blir ogiltiga samtidigt. Beroende på tidpunkten kan en användare begära en sida precis efter att den tagits bort från cachen, och precis innan den nya sidan publiceras. AEM returnerar nu den gamla sidan och Dispatcher cachelagrar den igen. Det här är mer en fråga för stora sajter.

## Invalidera Dispatcher-cachen från en publiceringsinstans {#invalidating-dispatcher-cache-from-a-publishing-instance}

Under vissa omständigheter kan du göra prestandavinster genom att överföra cachehantering från redigeringsmiljön till en publiceringsinstans. Det är då publiceringsmiljön (inte AEM-redigeringsmiljön) som skickar en cacheogiltigförklaring till Dispatcher när en publicerad sida tas emot.

Dessa omständigheter omfattar följande:

<!-- 

Comment Type: draft

<p>Cache invalidation requests for a page are also generated for any aliases or vanity URLs that are configured in the page properties. </p>

 -->

* Förhindra möjliga timingkonflikter mellan AEM Dispatcher och publiceringsinstansen (se [Invalidera Dispatcher-cachen från redigeringsmiljön](#invalidating-dispatcher-cache-from-the-authoring-environment)).
* Systemet innehåller flera publiceringsinstanser som finns på servrar med höga prestanda och endast en redigeringsinstans.

>[!NOTE]
>
>En erfaren AEM-administratör bör fatta beslut om att använda den här metoden.

En replikeringsagent som körs på publiceringsinstansen styr Dispatcher-tömningen. Konfigurationen görs dock i redigeringsmiljön och överförs sedan genom att agenten aktiveras:

1. Öppna AEM Tools Console.
1. Öppna den nödvändiga replikeringsagenten under Verktyg/replikering/Agenter vid publicering. Du kan använda Dispatcher Flush-agenten som är installerad som standard.
1. Klicka på Redigera och kontrollera att **Aktiverad** är markerat på fliken Inställningar.
1. (valfritt) Markera alternativet **Aliasuppdatering** om du vill aktivera ogiltiga aliassökvägar eller ogiltiga huvudsökvägar.
1. Gå till Dispatcher på fliken Transport genom att ange önskad URI.\
   Om du använder standardagenten för Dispatcher Flush ska du uppdatera värdnamnet och porten, till exempel `http://<dispatcherHost>:<portApache>/dispatcher/invalidate.cache`

   **Obs!** För Dispatcher Flush-agenter används URI-egenskapen endast om du använder sökvägsbaserade virtuella värdposter för att skilja mellan grupper. Du använder det här fältet för att ange gruppen som ogiltig. Servergrupp nr 1 har till exempel en virtuell värd på `www.mysite.com/path1/*` och servergrupp nr 2 har en virtuell värd på `www.mysite.com/path2/*`. Du kan använda URL:en `/path1/invalidate.cache` för att ange den första servergruppen som mål och `/path2/invalidate.cache` för den andra servergruppen. Mer information finns i [Använda Dispatcher med flera domäner](dispatcher-domains.md).

1. Konfigurera andra parametrar efter behov.
1. Logga in på publiceringsinstansen och validera justeringsagentens konfiguration. Kontrollera även att det är aktiverat.
1. Upprepa för alla publiceringsinstanser som påverkas.

När du har konfigurerat och aktiverar en sida från författaren till publiceringen initierar den här agenten en standardreplikering. Loggen innehåller meddelanden som anger begäranden från din publiceringsserver, som i följande exempel:

1. `<publishserver> 13:29:47 127.0.0.1 POST /dispatcher/invalidate.cache 200`

## Gör Dispatcher-cachen ogiltig manuellt {#manually-invalidating-the-dispatcher-cache}

Om du vill göra Dispatcher-cachen ogiltig (eller tömma) utan att aktivera en sida kan du skicka en HTTP-begäran till Dispatcher. Du kan till exempel skapa ett AEM-program som gör att administratörer eller andra program kan tömma cachen.

HTTP-begäran gör att Dispatcher tar bort specifika filer från cachen. Dispatcher uppdaterar sedan cacheminnet med en ny kopia.

### Ta bort cachelagrade filer {#delete-cached-files}

Skicka en HTTP-begäran som gör att Dispatcher tar bort filer från cachen. Dispatcher cachelagrar filerna igen endast när det tar emot en klientbegäran för sidan. Att ta bort cachelagrade filer på det här sättet är lämpligt för webbplatser som sannolikt inte tar emot samtidiga begäranden för samma sida.

HTTP-begäran har följande format:

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate  
CQ-Handle: path-pattern
Content-Length: 0
```

Dispatcher tömmer (tar bort) de cachelagrade filer och mappar som har namn som matchar värdet i huvudet `CQ-Handler`. En `CQ-Handle` av `/content/geomtrixx-outdoors/en` matchar följande objekt:

* Alla filer (med filtillägg) med namnet `en` i katalogen `geometrixx-outdoors`

* Alla kataloger med namnet `_jcr_content` nedanför katalogen `en` (som, om det finns, innehåller cachelagrade återgivningar av undernoder på sidan)

Alla andra filer i Dispatcher-cachen (eller upp till en viss nivå, beroende på `/statfileslevel`-inställningen) ogiltigförklaras genom att filen `.stat` klickas. Filens senaste ändringsdatum jämförs med det senaste ändringsdatumet för ett cachelagrat dokument och dokumentet hämtas igen om filen `.stat` är nyare. Mer information finns i [Invaliderar filer efter mappnivå](dispatcher-configuration.md#main-pars_title_26).

Invalidering (d.v.s. beröring av .stat-filer) kan förhindras genom att en extra rubrik `CQ-Action-Scope: ResourceOnly` skickas. Den här funktionen kan användas för att tömma vissa resurser. Allt utan att andra delar av cachen blir ogiltiga, som JSON-data. Dessa data skapas dynamiskt och kräver regelbunden tömning oberoende av cachen. Som exempel kan du representera data som hämtas från ett tredjepartssystem för att visa nyheter, aktiekurser och så vidare.

### Ta bort och cacha filer {#delete-and-recache-files}

Skicka en HTTP-begäran som får Dispatcher att ta bort cachelagrade filer och omedelbart hämta och cacha filen. Ta bort och cachelagra omedelbart om filer när det är troligt att webbplatser tar emot samtidiga klientbegäranden för samma sida. Omedelbar cachelagring säkerställer att Dispatcher bara hämtar och cachelagrar sidan en gång, i stället för en gång för varje samtidig klientbegäran.

**Obs!** Borttagning och cachelagring av filer bör endast utföras på publiceringsinstansen. När det utförs från författarinstansen inträffar konkurrensförhållanden när försök att hämta resurser görs innan de har publicerats.

HTTP-begäran har följande format:

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate   
`Content-Type: text/plain  
CQ-Handle: path-pattern  
Content-Length: numchars in bodypage_path0

page_path1 
...  
page_pathn
```

De sidsökvägar som ska cachelagras omedelbart visas på separata rader i meddelandetexten. Värdet `CQ-Handle` är sökvägen till en sida som gör sidorna ogiltiga. (Se parametern `/statfileslevel` i konfigurationsobjektet [ Cache](dispatcher-configuration.md#main-pars_146_44_0010).) Följande exempel på HTTP-begäran tar bort och cachelagrar `/content/geometrixx-outdoors/en.html page`:

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate  
Content-Type: text/plain   
CQ-Handle: /content/geometrixx-outdoors/en/men.html  
Content-Length: 36

/content/geometrixx-outdoors/en.html
```

### Exempel på tömningsservett {#example-flush-servlet}

I följande kod implementeras en servlet som skickar en invalideringsbegäran till Dispatcher. Servern tar emot ett begärandemeddelande som innehåller `handle` och `page` parametrar. De här parametrarna anger värdet för rubriken `CQ-Handle` och sökvägen till sidan som ska cachelagras. Servern använder värdena för att konstruera HTTP-begäran för Dispatcher.

När servern distribueras till publiceringsinstansen gör följande URL att Dispatcher tar bort /content/geometrixx-outdoors/en.html och sedan cachelagrar en ny kopia.

`10.36.79.223:4503/bin/flushcache/html?page=/content/geometrixx-outdoors/en.html&handle=/content/geometrixx-outdoors/en/men.html`

>[!NOTE]
>
>Den här exempelservern är inte säker och visar bara hur HTTP Post-begärandemeddelandet används. Lösningen bör skydda åtkomsten till serverutrymmet.
>

```java
package com.adobe.example;

import org.apache.felix.scr.annotations.Component;
import org.apache.felix.scr.annotations.Service;
import org.apache.felix.scr.annotations.Property;

import org.apache.sling.api.SlingHttpServletRequest;
import org.apache.sling.api.SlingHttpServletResponse;
import org.apache.sling.api.servlets.SlingSafeMethodsServlet;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import org.apache.commons.httpclient.*;
import org.apache.commons.httpclient.methods.PostMethod;
import org.apache.commons.httpclient.methods.StringRequestEntity;

@Component(metatype=true)
@Service
public class Flushcache extends SlingSafeMethodsServlet {

 @Property(value="/bin/flushcache")
 static final String SERVLET_PATH="sling.servlet.paths";

 private Logger logger = LoggerFactory.getLogger(this.getClass());

 public void doGet(SlingHttpServletRequest request, SlingHttpServletResponse response) {
  try{ 
      //retrieve the request parameters
      String handle = request.getParameter("handle");
      String page = request.getParameter("page");

      //hard-coding connection properties is a bad practice, but is done here to simplify the example
      String server = "localhost"; 
      String uri = "/dispatcher/invalidate.cache";

      HttpClient client = new HttpClient();

      PostMethod post = new PostMethod("https://"+host+uri);
      post.setRequestHeader("CQ-Action", "Activate");
      post.setRequestHeader("CQ-Handle",handle);
   
      StringRequestEntity body = new StringRequestEntity(page,null,null);
      post.setRequestEntity(body);
      post.setRequestHeader("Content-length", String.valueOf(body.getContentLength()));
      client.executeMethod(post);
      post.releaseConnection();
      //log the results
      logger.info("result: " + post.getResponseBodyAsString());
      }
  }catch(Exception e){
      logger.error("Flushcache servlet exception: " + e.getMessage());
  }
 }
}
```
