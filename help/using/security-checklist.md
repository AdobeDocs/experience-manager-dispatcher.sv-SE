---
title: Dispatcher Security Checklist
description: Läs mer om Dispatcher checklista för säkerhet som ska fyllas i innan du börjar producera.
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
jcr-lastmodifiedby: remove-legacypath-6-1
index: y
internal: n
snippet: y
exl-id: 49009810-b5bf-41fd-b544-19dd0c06b013
source-git-commit: c41b4026a64f9c90318e12de5397eb4c116056d9
workflow-type: tm+mt
source-wordcount: '582'
ht-degree: 0%

---

# Dispatcher Security Checklist{#the-dispatcher-security-checklist}

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-05T05:14:35.365-0400

<p>Food for thought listed on <a href="https://jira.corp.adobe.com/browse/DOC-5649">DOC-5649</a>. To be considered while proof-reading.</p> 
<p> </p>

 -->

Adobe rekommenderar att du slutför följande checklista innan du börjar producera.

>[!CAUTION]
>
>Fyll i checklistan för din version av AEM innan du publicerar. Se motsvarande [Adobe Experience Manager-dokumentation](https://experienceleague.adobe.com/en/docs/experience-manager-65/content/security/security-checklist).

## Använd den senaste versionen av Dispatcher {#use-the-latest-version-of-dispatcher}

Installera den senaste tillgängliga versionen som är tillgänglig för din plattform. Uppgradera din Dispatcher-instans och använd den senaste versionen för att dra nytta av produkt- och säkerhetsförbättringarna. Se [Installera Dispatcher](dispatcher-install.md).

>[!NOTE]
>
>Du kan kontrollera den aktuella versionen av din Dispatcher-installation genom att titta i Dispatcher loggfil.
>
>`[Thu Apr 30 17:30:49 2015] [I] [23171(140735307338496)] Dispatcher initialized (build 4.1.9)`
>
>Om du vill hitta loggfilen kontrollerar du Dispatcher-konfigurationen i `httpd.conf`.

## Begränsa klienter som kan tömma cachen {#restrict-clients-that-can-flush-your-cache}

Adobe rekommenderar att du [begränsar antalet klienter som kan tömma cachen.](dispatcher-configuration.md#limiting-the-clients-that-can-flush-the-cache)

## Aktivera HTTPS för transportlagrets säkerhet {#enable-https-for-transport-layer-security}

Adobe rekommenderar att HTTPS-transportlagret aktiveras både för författare och publicering.

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-26T04:41:28.841-0400

<p>Recommended to have SSL termination, front end SSL.</p> 
<p>Question is do we want to have SSL communication between dispatcher and AEM instances (publish and/or author).</p> 
<p>We might want to have two items:</p> 
<ul> 
 <li>MUST HTTPS clients -&gt; dispatcher / load balancer</li> 
 <li>NICE load balancer -&gt; dispatcher<br /> </li> 
 <li>NICE dispatcher -&gt; instances if sensitive information such as credit cards / or infrastructure requirements such as DMZ</li> 
</ul>

 -->

## Begränsa åtkomst {#restrict-access}

Begränsa den externa åtkomsten så mycket som möjligt när du konfigurerar Dispatcher. Se [Exempel/filteravsnitt](dispatcher-configuration.md#main-pars_184_1_title) i Dispatcher-dokumentationen.

## Kontrollera att åtkomst till administrativa URL:er nekas {#make-sure-access-to-administrative-urls-is-denied}

Se till att du använder filter för att blockera extern åtkomst till administrativa URL:er, t.ex. webbkonsolen.

En lista med URL:er som måste blockeras finns i [Testar Dispatcher Security](dispatcher-configuration.md#testing-dispatcher-security).

## Använd Tillåtelselista i stället för Blockeringslista {#use-allowlists-instead-of-blocklists}

Tillåtelselista är ett bättre sätt att tillhandahålla åtkomstkontroll eftersom de till sin natur antar att alla åtkomstbegäranden ska nekas såvida de inte uttryckligen ingår i tillåtelselista. Den här modellen ger mer restriktiv kontroll över nya begäranden som kanske inte har granskats än eller övervägts under en viss konfigurationsfas.

## Kör Dispatcher med en dedikerad systemanvändare {#run-dispatcher-with-a-dedicated-system-user}

Konfigurera Dispatcher så att ett dedikerat, minst privilegierat användarkonto kör webbservern. Adobe rekommenderar att du bara ger skrivåtkomst till Dispatcher cachemapp.

IIS-användare måste dessutom konfigurera sin webbplats på följande sätt:

1. Välj **Anslut som en specifik användare** i inställningen för fysisk sökväg för din webbplats.
1. Ange användaren.

## Förhindra DoS-attacker (denial of service) {#prevent-denial-of-service-dos-attacks}

En denial of service-attack (DoS) är ett försök att göra en datorresurs otillgänglig för de avsedda användarna.

På Dispatcher-nivå finns det två metoder för att konfigurera för att förhindra DoS-attacker: [Filter](https://experienceleague.adobe.com/en/docs#/filter)

* Använd modulen mod_rewrite (till exempel [Apache 2.4](https://httpd.apache.org/docs/2.4/mod/mod_rewrite.html)) för att utföra URL-valideringar (om URL-mönsterreglerna inte är för komplexa).

* Hindra Dispatcher från att cachelagra URL:er med falska tillägg genom att använda [filters](dispatcher-configuration.md#configuring-access-to-content-filter).\
  Ändra till exempel cachningsreglerna för att begränsa cachning till de förväntade MIME-typerna, som:

   * `.html`
   * `.jpg`
   * `.gif`
   * `.swf`
   * `.js`
   * `.doc`
   * `.pdf`
   * `.ppt`

  En exempelkonfigurationsfil kan visas för [att begränsa extern åtkomst](#restrict-access). Den innehåller begränsningar för MIME-typer.

Om du vill aktivera alla funktioner för publiceringsinstanserna konfigurerar du filter så att de inte får åtkomst till följande noder:

* `/etc/`
* `/libs/`

Konfigurera sedan filter för att ge åtkomst till följande nodsökvägar:

* `/etc/designs/*`
* `/etc/clientlibs/*`
* `/etc/segmentation.segment.js`
* `/libs/cq/personalization/components/clickstreamcloud/content/config.json`
* `/libs/wcm/stats/tracker.js`
* `/libs/cq/personalization/*` (JS, CSS och JSON)
* `/libs/cq/security/userinfo.json` (CQ-användarinformation)
* `/libs/granite/security/currentuser.json` (**data får inte cachelagras**)

* `/libs/cq/i18n/*` (Internalisering)

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-26T04:38:17.016-0400

<p>We need to highlight whether a path applies to all versions or specific ones.<br /> </p>

 -->

## Konfigurera Dispatcher för att förhindra CSRF-attacker {#configure-dispatcher-to-prevent-csrf-attacks}

AEM tillhandahåller ett [ramverk](https://experienceleague.adobe.com/en/docs/experience-manager-release-information/aem-release-updates/previous-updates/aem-previous-versions#verification-steps) som syftar till att förhindra attacker av typen Cross-Site Request. Om du vill använda ramverket på rätt sätt tillåtslista du stödet för CSRF-token i Dispatcher genom att göra följande:

1. Skapa ett filter som tillåter sökvägen `/libs/granite/csrf/token.json`;
1. Lägg till rubriken `CSRF-Token` i avsnittet `clientheaders` i Dispatcher-konfigurationen.

## Förhindra clickjacking {#prevent-clickjacking}

För att förhindra clickjacking rekommenderar Adobe att du konfigurerar webbservern så att HTTP-huvudet `X-FRAME-OPTIONS` anges till `SAMEORIGIN`.

Mer information om clickjacking finns på [OWASP-webbplatsen](https://owasp.org/www-community/attacks/Clickjacking).

## Utför ett penetrationsprov {#perform-a-penetration-test}

Adobe rekommenderar att du gör ett penetrationstest av din AEM-infrastruktur innan du börjar producera.

