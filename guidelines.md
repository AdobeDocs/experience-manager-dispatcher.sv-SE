---
source-git-commit: 2d90738d01fef6e37a2c25784ed4d1338c037c23
workflow-type: tm+mt
source-wordcount: '715'
ht-degree: 0%

---
# Riktlinjer för att bidra till Adobe Experience Manager-dokumentation

## Dokumentationsfilosofi

Adobe Experience Manager-användare arbetar i mycket konkurrenskraftiga miljöer och strävar efter att skapa digitala upplevelser som skiljer dem från deras konkurrenter. När Adobe levererar avancerade nya verktyg för AEM kompletteras dessa verktyg därför med korrekt och tydlig dokumentation som gör att kunderna omedelbart kan använda sina AEM och maximera avkastningen på sina investeringar.

Målet med den AEM dokumentationen är att ge AEM tillgång till dokumentation så snart som möjligt. Därför prioriterar Adobe korrekt, användbar dokumentation och strävar efter att kontinuerligt uppdatera och förbättra den.

## Dokumentationsbidrag

För att ständigt förbättra AEM dokumentation är AEM välkommen att bidra till dokumentationen. Vare sig det gäller förfrågningar eller frågor kan förbättringar av dokumentationen vara korrigeringar, förtydliganden, tillägg och andra exempel.

## Dokumentationsstandarder

Adobe välkomnar bidrag till sin dokumentation, men alla bidrag till AEM, antingen i form av en begäran om att tjänsten ska kunna dras in eller en fråga, bör överensstämma med Adobe normer för bidrag och dokumentation.

Bidrag som inte uppfyller dessa standarder kan avvisas.

### Adobe-dokument - standardanvändningsexempel.

AEM dokumentation täcker standardanvändningsfall. Användningsfall som inte omfattas av standardinstallationer och standardanvändning av produkten ingår inte i AEM.

### Adobe dokumenterar vanligtvis inte buggar eller deras temporära lösningar.

AEM dokumentation täcker standardanvändningsfall. Av den anledningen har buggar, effekter orsakade av buggar och tillfälliga lösningar för buggar inte dokumenterats.

Undantag från den här regeln gäller versionsinformationen där kända problem kan listas med möjliga lösningar som har godkänts av AEM produkthantering.

### Dokumentationsbidragen är inte till för att besvara tekniska frågor.

Alla idéer du behöver för att förbättra AEM dokumentation är välkomna som bidrag. Alla kommentarer, problem och pull-förfrågningar är avsedda som *avgifter* endast. De ska inte besvara dina frågor om hur du använder AEM, implementerar ditt AEM eller löser tekniska problem.

Du kan rapportera frågor om AEM eller tekniska fel genom att använda den normala supportprocessen via [Experience Cloud Enterprise Support Portal](https://experienceleague.adobe.com/?support-solution=General#support) eller diskuteras i [Experience Manager community](https://experienceleaguecommunities.adobe.com/t5/adobe-experience-manager/ct-p/adobe-experience-manager-community).

***AEM bidrag till dokumentationen ersätter inte Adobe kundtjänst*** och alla sådana bidrag som söker svar på frågor som rör stöd avvisas.

### Bidragen ska tydligt hänvisa till berörda dokumentationssidor.

Om du skapar ett problem som kan föreslå förbättringar av dokumentationen måste du inkludera länkar till de sidor som påverkas. Om du skapar ett problem med hjälp av **Redigera den här sidan** på en dokumentationssida skapas problemet automatiskt med en länk till sidan.

Detta gäller inte för pull-begäranden eftersom pull-begäranden till sin natur refererar till de berörda sidorna.

## Riktlinjer för dokumentation

Adobe begär att eventuella bidrag till dokumentationen ska följa vissa riktlinjer för format.

Följ de här riktlinjerna för att underlätta granskningen av ditt bidrag och därför går det snabbare att integrera i Adobe-dokumentationen.

### Språk och format

#### Språk

* AEM dokumentation skrivs och underhålls på amerikansk engelska.
* Håll meningar så enkla som möjligt.
* Se till att språket är klart och koncist.

Kom ihåg att läsarna av AEM är från hela världen och inte kan förväntas vara inbyggda eller flytande engelska. Undvik kollokvialism och håll språket så tydligt och enkelt som möjligt.

#### Följ Microsoft® Manual of Style

[The Microsoft® Manual of Style](https://learn.microsoft.com/en-us/style-guide/welcome/) är en kostnadsfri handbok för dokumentationsformat som fokuserar på programvarudokumentation och AEM dokumentation följer den här handboken när det är möjligt.

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

Undvik om möjligt direkta referenser till en viss version i dokumentationsinnehållet. Detta gör dokumentationen mer flexibel och utbyggbar för framtida versioner.

### Användning av dag, AEM, CQ, CRX

Använd alltid produktens fullständiga namn **Adobe Experience Manager** för första gången i en artikel och kan därefter kallas **AEM**.

Day, Day Software, CQ och CRX bör inte användas utom när det är oundvikligt, t.ex. i klassnamn eller med hänvisning till AEM.