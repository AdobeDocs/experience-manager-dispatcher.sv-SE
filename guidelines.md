---
source-git-commit: c41b4026a64f9c90318e12de5397eb4c116056d9
workflow-type: tm+mt
source-wordcount: '714'
ht-degree: 0%

---
# Riktlinjer för att bidra till Adobe Experience Manager-dokumentation

## Dokumentationsfilosofi

Adobe Experience Manager-användare arbetar i mycket konkurrenskraftiga miljöer och strävar efter att skapa digitala upplevelser som skiljer dem från deras konkurrenter. När Adobe levererar avancerade verktyg i AEM kompletteras dessa med korrekt och tydlig dokumentation. Man kan omedelbart utnyttja AEM-investeringen och maximera avkastningen på investeringen.

Målet med AEM-dokumentationen är att skicka dokumentation till AEM-användare så snart som möjligt. Därför prioriterar Adobe korrekt, användbar dokumentation och strävar efter att uppdatera och förbättra den kontinuerligt.

## Dokumentationsbidrag

I syfte att kontinuerligt förbättra AEM-dokumentationen är det välkommet att AEM-användare bidrar till dokumentationen. Vare sig det gäller förfrågningar eller frågor kan förbättringar av dokumentationen vara korrigeringar, förtydliganden, tillägg och andra exempel.

## Dokumentationsstandarder

Adobe tar gärna emot bidrag till dokumentationen, men alla bidrag till AEM-dokumentationen i en ansökan eller i ett ärende bör följa Adobe standarder för bidrag och dokumentation.

Bidrag som inte uppfyller dessa standarder kan avvisas.

### Standardanvändningsexempel finns på Adobe.

AEM dokumentation täcker standardanvändningsfall. Användningsfall som inte omfattas av standardinstallation och -användning ingår inte i AEM-dokumentationen.

### Adobe dokumenterar vanligtvis inte buggar eller deras temporära lösningar.

AEM dokumentation täcker standardanvändningsfall. Av den anledningen dokumenteras inte buggar, effekter orsakade av buggar och tillfälliga lösningar för buggar.

Undantag från den här regeln gäller versionsinformationen där kända problem kan listas med möjliga lösningar som produkthanteringen har godkänt.

### Dokumentationsbidragen är inte till för att besvara tekniska frågor.

Alla idéer du behöver för att förbättra AEM dokumentation är välkomna som bidrag. Alla kommentarer, utgåvor och pull-begäranden är dock endast avsedda som *bidrag*. De ska inte besvara några frågor om hur man använder AEM, implementerar AEM-projekt eller löser tekniska problem.

Du kan rapportera eventuella frågor om AEM användning eller tekniska fel. Använd den normala supportprocessen via [Experience Cloud Enterprise Support-portalen](https://experienceleague.adobe.com/?support-solution=General#support) eller beskrivs i [Experience Manager-communityn](https://experienceleaguecommunities.adobe.com/t5/adobe-experience-manager/ct-p/adobe-experience-manager-community).

***AEM dokumentationsbidrag ersätter inte Adobe kundtjänst*** och eventuella bidrag som söker svar på supportrelaterade frågor avvisas.

### Bidragen ska tydligt hänvisa till berörda dokumentationssidor.

Om du skapar ett problem som kan föreslå förbättringar av dokumentationen måste du inkludera länkar till de sidor som påverkas. Om du skapar ett problem med länken **Redigera den här sidan** på en dokumentationssida skapas problemet automatiskt med en länk till sidan.

Den här processen gäller inte för pull-begäranden eftersom pull-begäranden till sin natur refererar till de berörda sidorna.

## Riktlinjer för dokumentation

Adobe ber att eventuella bidrag till dokumentationen ska följa vissa riktlinjer för format.

Om du följer dessa riktlinjer blir det enklare att granska ditt bidrag, och därför går det snabbare att integrera Adobe dokumentation.

### Språk och format

#### Språk

* AEM dokumentation skrivs och underhålls på amerikansk engelska.
* Håll meningar så enkla som möjligt.
* Se till att språket är klart och koncist.

Kom ihåg att läsarna av AEM-dokumentation är från hela världen och inte kan förväntas vara inbyggda eller flytande engelska högtalare. Undvik kollokvialism och håll språket så tydligt och enkelt som möjligt.

#### Följ Microsoft®-formathandboken

[Handboken för Microsoft® Style](https://learn.microsoft.com/en-us/style-guide/welcome/) är en kostnadsfri handbok för dokumentationsformat som fokuserar på programvarudokumentation och AEM-dokumentation följer den här handboken när det är möjligt.

### Formatering

| Objekt | Stil |
|---|---|
| Element eller alternativ i användargränssnittet | **fet** |
| Filnamn, sökväg, användarindata, parametervärden | `monospaced` |
| Kod, kommandorad | ```Code Block``` |

### Skärmbilder

Skärmbilder ska användas med omdöme och endast när en textbeskrivning är otillräcklig.

Markörer eller andra anteckningar i skärmbilder (som röda ramar, pilar eller text) bör inte användas. På så sätt är skärmbilderna enklare att återanvända eller replikera i lokaliserade versioner av dokumentationen.

### Versionsspecifika referenser

Undvik om möjligt direkta referenser till en viss version i dokumentationsinnehållet. Den här rekommendationen gör dokumentationen mer flexibel och utbyggbar för framtida versioner.

### Användning av Day, AEM, CQ, CRX

I en artikel ska du alltid referera till produkten med dess fullständiga namn **Adobe Experience Manager** första gången den används. Därefter kan den kallas **AEM**.

Day, Day Software, CQ och CRX får inte användas utom när det är oundvikligt, t.ex. i klassnamn eller med hänvisning till AEM historia.


