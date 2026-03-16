---
title: Dispatcher ETag Enhancement for CDN Revalidation
description: Tillgänglighet, supportstatus och beteende för INTERNAL_AEM_DISPATCHER_ETAG_ENHANCEMENT i AEM as a Cloud Service.
source-git-commit: ac0fafd060643903735ff565072ef2c5bee970be
workflow-type: tm+mt
source-wordcount: '308'
ht-degree: 0%

---

# Dispatcher ETag Enhancement for CDN Revalidation

## Ökning

Flaggan `INTERNAL_AEM_DISPATCHER_ETAG_ENHANCEMENT` gör att Dispatcher kan utvärdera begärandehuvudet `If-None-Match` vid cacheträffar. När det inkommande `If-None-Match`-värdet matchar det cachelagrade `ETag` kan Dispatcher returnera `304 Not Modified` i stället för `200 OK`.

Detta beteende är utformat för att minska onödig nyttolastöverföring mellan CDN och Dispatcher och öka villkorsstyrd cacheeffektivitet.

## Tillgänglighet

- Dispatcher-version: `2.0.264`
- AEM SDK-version: `aem-sdk-2026.2.24464.20260214T050318Z-260100`

## AEM as a Cloud Service support

I AEM as a Cloud Service stöds den här funktionen för kunder.

Kunder kan aktivera det genom att ställa in miljövariabeln `INTERNAL_AEM_DISPATCHER_ETAG_ENHANCEMENT` i Cloud Manager. Adobe kan även aktivera för kundens räkning vid behov.

När det här alternativet är aktiverat och CDN skickar `If-None-Match` och den relevanta `ETag` finns i Dispatcher-cachen, förväntas högre `304` svarsfrekvenser mellan CDN och Dispatcher. Denna ökning är det avsedda resultatet.

## Konfigurationsexempel (cache ETag-huvud)

För att den här förbättringen ska bli effektiv bör du se till att Dispatcher cachelagrar svarshuvudet `ETag` och att webbservern är konfigurerad så att du inte kan generera filsystembaserade ETags.

Exempel på `dispatcher.any`-cacheavsnitt:

```text
/cache {
  /headers {
    "Cache-Control"
    "Content-Type"
    "Expires"
    "Last-Modified"
    "ETag"
  }
}
```

Exempeldirektivet Apache i Dispatcher-värdkontexten:

```apache
FileETag none
```

Riktlinjer för huvud-cachning vid baslinje finns i [Cachelagra HTTP-svarshuvuden](dispatcher-configuration.md#caching-http-response-headers).

## Exempel på validering

När miljövariabeln har aktiverats och konfigurationsändringarna har distribuerats:

1. Begär en gång att cachelagra cachen och hämta den returnerade `ETag`.
1. Begär igen med `If-None-Match: <etag-value>`.
1. Bekräfta att Dispatcher returnerar `304 Not Modified` för cacheminnespåfyllnadsverifieringsflöden.

## Offentlig referens (relaterat beteende)

Om du vill få vägledning om huvudcachelagring och hantering av `ETag` i Dispatcher kan du läsa:

- [Konfigurera Dispatcher - cachelagra HTTP-svarshuvuden](https://experienceleague.adobe.com/sv/docs/experience-manager-dispatcher/using/configuring/dispatcher-configuration#caching-http-response-headers)

&quot;Den här funktionen är tillgänglig i Dispatcher `2.0.264` (AEM SDK `2026.2.24464`). När det här alternativet är aktiverat kan Dispatcher validera `If-None-Match` mot cachelagrade `ETag`-värden och returnera `304 Not Modified` vid cacheträffar. I AEM as a Cloud Service stöds detta och kan aktiveras via Cloud Manager-miljökonfiguration.&quot;
