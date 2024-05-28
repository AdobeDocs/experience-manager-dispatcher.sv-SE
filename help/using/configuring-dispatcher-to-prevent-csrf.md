---
title: Konfigurera Adobe Experience Manager Dispatcher för att förhindra CSRF-attacker
description: Lär dig hur du konfigurerar Adobe Experience Manager Dispatcher för att förhindra attacker med attacker som leder till cross-site request-attacker.
topic-tags: dispatcher
content-type: reference
exl-id: bcd38878-f977-46a6-b01a-03e4d90aef01
source-git-commit: 0a1aa854ea286a30c3527be8fc7c0998726a663f
workflow-type: tm+mt
source-wordcount: '235'
ht-degree: 0%

---

# Konfigurera Adobe Experience Manager Dispatcher för att förhindra CSRF-attacker{#configuring-dispatcher-to-prevent-csrf-attacks}

AEM (Adobe Experience Manager) tillhandahåller ett ramverk som syftar till att förhindra attacker med förfalskade begäranden på flera webbplatser. Gör följande ändringar i Dispatcher-konfigurationen om du vill använda det här ramverket på rätt sätt:

>[!NOTE]
>
>Var noga med att uppdatera regelnumren i exemplen nedan baserat på din befintliga konfiguration. Kom ihåg att Dispatchers använder den senaste matchande regeln för att bevilja eller neka tillstånd, så placera reglerna längst ned i den befintliga listan.

1. I `/clientheaders` i `author-farm.any` och `publish-farm.any`lägger du till följande post längst ned i listan:\
   `CSRF-Token`
1. I avsnittet /filters i `author-farm.any` och `publish-farm.any` eller `publish-filters.any` fil, lägg till följande rad för att tillåta begäranden för `/libs/granite/csrf/token.json` genom Dispatcher.\
   `/0999 { /type "allow" /glob " * /libs/granite/csrf/token.json*" }`

1. Under `/cache /rules` i `publish-farm.any`lägger du till en regel som blockerar Dispatcher från att cachelagra `token.json` -fil. Vanligtvis åsidosätter författare cachelagring, så du behöver inte lägga till regeln i `author-farm.any`.

   `/0999 { /glob "/libs/granite/csrf/token.json" /type "deny" }`

Kontrollera dispatcher.log i DEBUG-läge om du vill verifiera att konfigurationen fungerar. Det kan hjälpa dig att validera att `token.json` för att vara säker på att den inte cachelagras eller blockeras av filter. Du bör se meddelanden som liknar:\
`... checking [/libs/granite/csrf/token.json]  `
`... request URL not in cache rules: /libs/granite/csrf/token.json`\
`... cache-action for [/libs/granite/csrf/token.json]: NONE`

Du kan också validera att begäranden lyckas i Apache `access_log`. Begäranden för&quot;/libs/granite/csrf/token.json ska returnera en HTTP 200-statuskod.
