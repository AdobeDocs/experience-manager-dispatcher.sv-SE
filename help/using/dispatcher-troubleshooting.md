---
title: Felsökning av Dispatcher-problem
description: Lär dig att felsöka Dispatcher-problem.
cmgrlastmodified: 01.11.2007 08 22 29 [aheimoz]
pageversionid: 1193211344162
template: /apps/docs/templates/contentpage
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
exl-id: 29f338ab-5d25-48a4-9309-058e0cc94cff
source-git-commit: 0a1aa854ea286a30c3527be8fc7c0998726a663f
workflow-type: tm+mt
source-wordcount: '539'
ht-degree: 0%

---

# Felsökning av Dispatcher-problem {#troubleshooting-dispatcher-problems}

>[!NOTE]
>
>Dispatcher-versionerna är oberoende av AEM. Dispatcher-dokumentationen är dock inbäddad i AEM. Använd alltid den Dispatcher-dokumentation som är inbäddad i dokumentationen för den senaste versionen av AEM.
>
>Du kan ha omdirigerats till den här sidan om du har följt en länk till Dispatcher-dokumentationen. Länken är inbäddad i dokumentationen för en tidigare version av AEM.

>[!NOTE]
>
>Mer information finns i [Dispatcher Knowledge Base](https://helpx.adobe.com/experience-manager/kb/index/dispatcher.html), [Troubleshooting Dispatcher Flushing Issues](https://experienceleague.adobe.com/search.html?lang=en#q=troubleshooting%20dispatcher%20flushing%20issues&amp;sort=relevancy&amp;f:el_product=[Experience%20Manager]) och i [Dispatcher Top Issues FAQ](dispatcher-faq.md).

## Kontrollera den grundläggande konfigurationen {#check-the-basic-configuration}

Som alltid är de första stegen att kontrollera grunderna:

* [Bekräfta grundläggande åtgärd](/help/using/dispatcher-configuration.md#confirming-basic-operation)
* Kontrollera alla loggfiler för webbservern och Dispatcher. Öka `loglevel` som används för Dispatcher [log](/help/using/dispatcher-configuration.md#logging) om det behövs.

* [Kontrollera konfigurationen](/help/using/dispatcher-configuration.md):

   * Har du flera utskickare?

      * Har du fastställt vilken Dispatcher som hanterar den webbplats/sida som du undersöker?

   * Har du implementerat filter?

      * Påverkar de här filtren det du utforskar?

## Diagnostikverktyg för IIS {#iis-diagnostic-tools}

IIS innehåller olika spårningsverktyg, beroende på den faktiska versionen:

* IIS 6 - IIS-diagnostikverktygen kan hämtas och konfigureras
* IIS 7 - kalkeringen är helt integrerad

De här verktygen kan hjälpa dig att övervaka aktiviteten.

## IIS och 404 hittades inte {#iis-and-not-found}

När du använder IIS kanske `404 Not Found` returneras i olika scenarier. Om så är fallet, se följande artiklar i kunskapsbasen.

* [IIS 6/7: POSTEN returnerar 404 ](https://helpx.adobe.com/experience-manager/kb/IIS6IsapiFilters.html)
* [IIS 6: Begäranden till URL:er som innehåller bassökvägen `/bin` returnerar `404 Not Found`](https://helpx.adobe.com/experience-manager/kb/RequestsToBinDirectoryFailInIIS6.html)

Kontrollera också att Dispatcher cacherot och IIS-dokumentroten är inställda på samma katalog.

## Problem med att ta bort arbetsflödesmodeller {#problems-deleting-workflow-models}

**Symtomen**

Problem med att ta bort arbetsflödesmodeller vid åtkomst till en AEM författarinstans via Dispatcher.

**Steg att återskapa:**

1. Logga in på din författarinstans (bekräfta att begäranden dirigeras via Dispatcher).
1. Skapa ett arbetsflöde, till exempel med titeln inställd på workflowToDelete.
1. Bekräfta att arbetsflödet har skapats.
1. Markera och högerklicka på arbetsflödet och klicka sedan på **Ta bort**.

1. Bekräfta genom att klicka på **Ja**.
1. En felmeddelanderuta med följande information visas:\
   `ERROR 'Could not delete workflow model!!`.

**Upplösning**

Lägg till följande rubriker i avsnittet `/clientheaders` i `dispatcher.any`-filen:

* `x-http-method-override`
* `x-requested-with`

```
{  
{  
/clientheaders  
{  
...  
"x-http-method-override"  
"x-requested-with"  
}
```

## Interferens med mod_dir (Apache) {#interference-with-mod-dir-apache}

Den här processen beskriver hur Dispatcher interagerar med `mod_dir` i Apache-webbservern, eftersom den kan ge olika, potentiellt oväntade effekter:

### Apache 1.3 {#apache}

I Apache 1.3 hanterar `mod_dir` varje begäran där URL:en mappas till en katalog i filsystemet.

Den kommer antingen att

* omdirigera begäran till en befintlig `index.html`-fil
* generera en kataloglista

När Dispatcher är aktiverat bearbetar den sådana förfrågningar genom att registrera sig själv som hanterare för innehållstypen `httpd/unix-directory`.

### Apache 2.x {#apache-x}

I Apache 2.x är det annorlunda. En modul kan hantera olika faser av begäran, t.ex. URL-korrigering. `mod_dir` hanterar det här steget genom att dirigera om en begäran (när URL:en mappar till en katalog) till URL:en med `/` tillagt.

Dispatcher fångar inte upp korrigeringen `mod_dir`, men hanterar begäran fullständigt till den omdirigerade URL:en (d.v.s. med `/` tillagd). Den här processen kan utgöra ett problem om fjärrservern (till exempel AEM) hanterar begäranden till `/a_path` på ett annat sätt än förfrågningar till `/a_path/` (när `/a_path` mappar till en befintlig katalog).

Om detta händer måste du antingen:

* inaktivera `mod_dir` för underträdet `Directory` eller `Location` som hanteras av Dispatcher

* använd `DirectorySlash Off` för att konfigurera `mod_dir` att inte lägga till `/`
