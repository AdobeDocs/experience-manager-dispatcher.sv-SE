---
title: Optimera en webbplats för cacheprestanda
description: Lär dig hur du utformar din webbplats för att maximera fördelarna med cachning.
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
redirecttarget: https://helpx.adobe.com/se/experience-manager/6-4/sites/deploying/using/configuring-performance.html
index: y
internal: n
snippet: y
source-git-commit: c41b4026a64f9c90318e12de5397eb4c116056d9
workflow-type: tm+mt
source-wordcount: '1128'
ht-degree: 0%

---


# Optimera en webbplats för cacheprestanda {#optimizing-a-website-for-cache-performance}

<!-- 

Comment Type: remark
Last Modified By: Silviu Raiman (raiman)
Last Modified Date: 2017-10-25T04:13:34.919-0400

<p>This is a redirect to /experience-manager/6-2/sites/deploying/using/configuring-performance.html</p>

 -->

>[!NOTE]
>
>Dispatcher-versionerna är oberoende av AEM. Du kan ha omdirigerats till den här sidan om du har följt en länk till Dispatcher-dokumentationen. Länken var inbäddad i dokumentationen för en tidigare version av AEM.

Dispatcher har flera inbyggda mekanismer som du kan använda för att optimera prestanda. I det här avsnittet beskrivs hur du utformar din webbplats för att maximera fördelarna med cachning.

>[!NOTE]
>
>Det kan hjälpa dig att komma ihåg att Dispatcher lagrar cachen på en standardwebbserver. Det innebär att du:
>
>* kan cachelagra allt som du kan lagra som en sida och begära med en URL
>* kan inte lagra andra saker, t.ex. HTTP-huvuden, cookies, sessionsdata och formulärdata.
>
>I allmänhet innebär många cachelagringsstrategier att du väljer bra URL-adresser och inte förlitar dig på dessa ytterligare data.

## Använd konsekvent sidkodning {#using-consistent-page-encoding}

HTTP-begäranrubriker cachelagras inte, vilket innebär att problem kan uppstå om du lagrar sidkodningsinformation i sidhuvudet. I det här fallet används webbserverns standardkodning för sidan när Dispatcher skickar en sida från cachen. Det finns två sätt att undvika det här problemet:

* Om du bara använder en kodning kontrollerar du att kodningen som används på webbservern är densamma som standardkodningen på AEM webbplats.
* Om du vill ställa in kodningen använder du en `<META>`-tagg i HTML `head` -avsnittet, som i följande exempel:

```xml
        <META http-equiv="Content-Type" content="text/html; charset=EUC-JP">
```

## Undvik URL-parametrar {#avoid-url-parameters}

Undvik om möjligt URL-parametrar för sidor som du vill cachelagra. Om du till exempel har ett bildgalleri cachelagras aldrig följande URL (såvida inte Dispatcher [har konfigurerats därefter](dispatcher-configuration.md#main-pars_title_24)):

```xml
www.myCompany.com/pictures/gallery.html?event=christmas&amp;page=1
```

Du kan dock placera de här parametrarna i sidans URL-adress enligt följande:

```xml
www.myCompany.com/pictures/gallery.christmas.1.html
```

>[!NOTE]
>
>Den här URL:en anropar samma sida och samma mall som gallery.html. I malldefinitionen kan du ange vilket skript som ska återge sidan eller använda samma skript för alla sidor.

## Anpassa efter URL {#customize-by-url}

Om du tillåter användare att ändra teckensnittsstorleken (eller annan layoutanpassning) kontrollerar du att de olika anpassningarna återspeglas i webbadressen.

Till exempel cachelagras inte cookies, så om du sparar teckensnittsstorleken i en cookie (eller på liknande sätt) bevaras inte teckensnittsstorleken för den cachelagrade sidan. Därför returnerar Dispatcher dokument med valfri teckensnittsstorlek på måfå.

Om du tar med teckensnittsstorleken i URL:en som väljare undviker du det här problemet:

```xml
www.myCompany.com/news/main.large.html
```

>[!NOTE]
>
>I de flesta layoutaspekter går det även att använda formatmallar, klientskript eller båda. Antingen eller båda fungerar bra med cachning.
>
>Den här metoden är också användbar för en utskriftsversion där du kan använda en URL-adress som:
>
>`www.myCompany.com/news/main.print.html`
>
>Med hjälp av skriptordlistan för malldefinitionen kan du ange ett separat skript som återger utskriftssidorna.

## Gör bildfiler ogiltiga som titlar {#invalidating-image-files-used-as-titles}

Om du återger sidrubriker eller annan text som bilder kan du lagra filerna så att de tas bort när innehållet uppdateras på sidan:

1. Placera bildfilen i samma mapp som sidan.
1. Använd följande namnformat för bildfilen:

   `<page file name>.<image file name>`

Du kan till exempel lagra titeln för sidan myPage.html i filen myPage.title.gif. Den här filen tas automatiskt bort om sidan uppdateras, så alla ändringar av sidans titel återspeglas automatiskt i cachen.

>[!NOTE]
>
>Bildfilen finns inte nödvändigtvis i AEM-instansen. Du kan använda ett skript som skapar bildfilen dynamiskt. Dispatcher lagrar sedan filen på webbservern.

## Ogiltiga bildfiler som används för navigering {#invalidating-image-files-used-for-navigation}

Om du använder bilder för navigeringsposterna är metoden i stort sett densamma som med titlar, vilket är lite mer komplicerat. Lagra alla navigeringsbilder med målsidorna. Om du använder två bilder för normal och aktiv användning kan du använda följande skript:

* Ett skript som visar sidan som vanligt.
* Ett skript som bearbetar `.normal` begär och returnerar den normala bilden.
* Ett skript som bearbetar `.active` begär och returnerar den aktiverade bilden.

Det är viktigt att du skapar de här bilderna med samma namngivningshandtag som sidan för att vara säker på att en innehållsuppdatering tar bort dessa bilder och sidan.

För sidor som inte ändras finns bilderna kvar i cacheminnet, även om själva sidorna är automatiskt ogiltiga.

## Personalization {#personalization}

Dispatcher kan inte cachelagra personaliserade data, så vi rekommenderar att du begränsar personaliseringen till där det är nödvändigt. Så här visar du varför:

* Om du använder en fritt anpassningsbar startsida måste den sidan sammanställas varje gång en användare begär den.
* Om du däremot erbjuder ett alternativ på tio olika startsidor kan du cachelagra var och en av dem, vilket förbättrar prestandan.

>[!NOTE]
>
>Om du anpassar varje sida (till exempel genom att placera användarens namn i namnlisten) kan du inte cachelagra den, vilket kan få stor prestandapåverkan.
>
>Om du måste kan du dock göra följande:
>
>* Använd iFrames för att dela upp sidan i en del som är densamma för alla användare och en del som är densamma för alla sidor i användaren. Du kan sedan cachelagra båda dessa delar.
>* använda JavaScript på klientsidan för att visa personaliserad information. Du måste dock se till att sidan fortfarande visas korrekt om en användare stänger av JavaScript.
>

## Fästanslutningar {#sticky-connections}

[Antagliga anslutningar](dispatcher.md#TheBenefitsofLoadBalancing) säkerställer att dokumenten för en användare är sammansatta på samma server. Om en användare lämnar den här mappen och senare återgår till den, stannar anslutningen fortfarande kvar. Definiera en mapp så att den kan innehålla alla dokument som kräver klisterlappar för webbplatsen. Försök att inte ha med andra dokument i den. Om du gör det påverkas belastningsutjämningen om du använder personaliserade sidor och sessionsdata.

## MIME-typer {#mime-types}

Det finns två sätt som en webbläsare kan använda för att avgöra vilken typ av fil det är:

1. Med filnamnstillägget (till exempel .html, .gif och .jpg)
1. Med MIME-typen som servern skickar med filen.

För de flesta filer används MIME-typen i filtillägget:

1. Med filnamnstillägget (till exempel .html, .gif och .jpg)
1. Med MIME-typen som servern skickar med filen.

Om filnamnet saknar filtillägg visas det som oformaterad text.

MIME-typen är en del av HTTP-huvudet och därför cachelagras den inte av Dispatcher. Ditt AEM-program kan returnera filer som inte har något känt filtillägg. Om filerna i stället använder MIME-typen kan dessa filer visas felaktigt.

Följ dessa riktlinjer för att vara säker på att filerna cachelagras korrekt:

* Kontrollera att filerna alltid har rätt filtillägg.
* Undvik generiska filserverskript med URL-adresser som download.jsp?file=2214. Skriv om skriptet så att det använder URL:er som innehåller filspecifikationen. I föregående exempel skulle det vara `download.2214.pdf`.

