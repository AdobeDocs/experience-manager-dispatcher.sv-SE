---
title: Installera Dispatcher
description: Lär dig hur du installerar Dispatcher-modulen på Microsoft&reg; Internet Information Server, Apache Web Server och Sun Java &trade; Web Server-iPlanet.
contentOwner: User
converted: true
topic-tags: dispatcher
content-type: reference
exl-id: 9375d1c0-8d9e-46cb-9810-fa4162a8c1ba
source-git-commit: 9be9f5935c21ebbf211b5da52280a31772993c2e
workflow-type: tm+mt
source-wordcount: '3748'
ht-degree: 0%

---

# Installera Dispatcher {#installing-dispatcher}

<!-- 

Comment Type: draft

<h2>Introduction</h2>

 -->

Använd sidan [Dispatcher Versionsinformation](release-notes.md) för att hämta den senaste Dispatcher-installationsfilen för ditt operativsystem och din webbserver. Dispatcher versionsnummer är oberoende av Adobe Experience Manager versionsnummer och är kompatibla med Adobe Experience Manager 6.x, 5.x och Adobe CQ 5.x.

>[!NOTE]
>
>Adobe Experience Manager 6.5 kräver Dispatcher version 4.3.2 eller senare. Dispatcher-versionerna är alltså inte beroende av AEM, till exempel är Dispatcher version 4.3.2 också kompatibel med Adobe Experience Manager 6.4.

Följande namngivningskonvention används:

`dispatcher-<web-server>-<operating-system>-<dispatcher-version-number>.<file-format>`

Filen `dispatcher-apache2.4-linux-x86_64-ssl-4.3.1.tar.gz` innehåller till exempel Dispatcher version 4.3.1 för en Apache 2.4-webbserver som körs på Linux® i686 och som paketeras med formatet **tar**.

I följande tabell visas webbserveridentifieraren som används i filnamn för varje webbserver:

| Webbserver | Installationspaket |
|--- |--- |
| Apache 2.4 | dispatcher-apache **2.4**-&lt;andra parametrar> |
| Microsoft® Internet Information Server 7.5, 8, 8.5, 10 | dispatcher-**iis**-&lt;andra parametrar> |
| Sun Java™ Web Server Planet | dispatcher-**ns**-&lt;andra parametrar> |

>[!CAUTION]
>
>Se till att du installerar den senaste versionen av Dispatcher som finns för din plattform. Uppgradera din Dispatcher-instans på årsbasis och använd den senaste versionen för att utnyttja produktförbättringarna.

>[!NOTE]
>
>Kunder som uppgraderar specifikt från version 4.3.3 till version 4.3.4 bör observera ett annat beteende i hur cachelagring av rubriker är inställt för innehåll som inte kan cachelagras. Mer information om den här ändringen finns på sidan [Versionsinformation](/help/using/release-notes.md#nov).

Varje arkiv innehåller följande filer:

* Dispatcher-modulerna
* en exempelkonfigurationsfil
* README-filen som innehåller installationsinstruktioner och sista-minuten-information
* den ÄNDRINGSfil som listar problem som har korrigerats i aktuella och tidigare versioner

>[!NOTE]
>
>Kontrollera README-filen om det finns några ändringar eller plattformsspecifika anteckningar som inte har gjorts tidigare innan du påbörjar installationen.

<!-- 

Comment Type: draft

<h3>Supported Web Servers</h3>

 -->

<!-- 

Comment Type: draft

<p>The following web servers are supported for use with Dispatcher version 4.1.12:</p>

 -->

<!-- 

Comment Type: draft

<p>The following sections detail the specific Web server installation procedures.</p>

 -->

## Microsoft® Internet Information Server {#microsoft-internet-information-server}

Mer information om hur du installerar den här webbservern finns i följande resurser:

* Microsoft® egen dokumentation på Internet Information Server
* [&quot;Den officiella Microsoft® IIS-webbplatsen&quot;](https://www.iis.net/)

### Nödvändiga IIS-komponenter {#required-iis-components}

IIS-versionerna 8.5 och 10 kräver att följande IIS-komponenter är installerade:

* ISAPI-tillägg

Du måste också lägga till webbserverrollen (IIS). Använd Serverhanteraren för att lägga till rollen och komponenterna.

## Microsoft® IIS - Installera Dispatcher-modulen {#microsoft-iis-installing-the-dispatcher-module}

Nödvändigt arkiv för Microsoft® Internet Information System:

* `dispatcher-iis-<operating-system>-<dispatcher-release-number>.zip`

ZIP-filen innehåller följande filer:

| Fil | Beskrivning |
|--- |--- |
| `disp_iis.dll` | Dispatcher Dynamic Link Library-fil. |
| `disp_iis.ini` | Konfigurationsfil för IIS. Det här exemplet kan uppdateras med dina krav. **Obs**: INI-filen måste ha samma namn-rot som dll-filen. |
| `dispatcher.any` | En exempelkonfigurationsfil för Dispatcher. |
| `author_dispatcher.any` | En exempelkonfigurationsfil för Dispatcher som arbetar med författarinstansen. |
| README | Viktigt-fil som innehåller installationsinstruktioner och sista-minuten-information. **Obs!** Kontrollera den här filen innan du startar installationen. |
| ÄNDRINGAR | Ändrar en fil med en lista över problem som har åtgärdats i aktuella och tidigare versioner. |

Så här kopierar du Dispatcher-filerna till rätt plats.

1. Använd Utforskaren i Windows för att skapa katalogen `<IIS_INSTALLDIR>/Scripts`, till exempel `C:\inetpub\Scripts`.

1. Extrahera följande filer från Dispatcher-paketet till den här skriptkatalogen:

   * `disp_iis.dll`
   * `disp_iis.ini`
   * En av följande filer beroende på om Dispatcher arbetar med en AEM författarinstans eller publiceringsinstans:
      * Författarinstans: `author_dispatcher.any`
      * Publish-instans: `dispatcher.any`

## Microsoft® IIS - Konfigurera Dispatcher INI-filen {#microsoft-iis-configure-the-dispatcher-ini-file}

Om du vill konfigurera Dispatcher-installationen redigerar du filen `disp_iis.ini`. Grundformatet för filen `.ini` är följande:

```xml
[main]
configpath=<path to dispatcher.any>
loglevel=1|2|3
servervariables=0|1
replaceauthorization=0|1
```

I följande tabell beskrivs varje egenskap.

| Parameter | Beskrivning |
|--- |--- |
| `configpath` | Platsen för `dispatcher.any` i det lokala filsystemet (absolut sökväg). |
| `logfile` | Platsen för filen `dispatcher.log`. Om den här platsen inte är inställd går loggmeddelanden till Windows-händelseloggen. |
| `loglevel` | Definierar loggnivån som används för att skicka meddelanden till händelseloggen. Följande värden kan anges på loggnivå för loggfilen: <br/>0 - endast felmeddelanden. <br/>1 - fel och varningar. <br/>2 - fel, varningar och informationsmeddelanden <br/>3 - fel, varningar, informationsmeddelanden och felsökningsmeddelanden. <br/>**Obs!**: Ange loggnivån till 3 under installation och testning och sedan till 0 när du kör i en produktionsmiljö. |
| `replaceauthorization` | Anger hur auktoriseringshuvuden i HTTP-begäran hanteras. Följande värden är giltiga:<br/>0 - auktoriseringshuvuden har inte ändrats. <br/>1 - Ersätter en rubrik med namnet &quot;Authorization&quot; som inte är &quot;Basic&quot; med motsvarande `Basic <IIS:LOGON\_USER>`.<br/> |
| `servervariables` | Definierar hur servervariabler behandlas.<br/> - IIS-servervariabler skickas inte till Dispatcher eller AEM. <br/>1 - alla IIS-servervariabler (till exempel `LOGON\_USER, QUERY\_STRING, ...`) skickas till Dispatcher tillsammans med begäranderubrikerna (och även till AEM om de inte cachelagras).  <br/>Servervariabler innehåller `AUTH\_USER, LOGON\_USER, HTTPS\_KEYSIZE` och många andra. I IIS-dokumentationen finns en fullständig lista med variabler, med information. |
| `enable_chunked_transfer` | Definierar om segmentöverföring ska aktiveras (1) eller inaktiveras (0) för klientsvaret. Standardvärdet är 0. |

En exempelkonfiguration:

```xml
[main]
configpath=C:\Inetpub\Scripts\dispatcher.any
loglevel=1
servervariables=1
replaceauthorization=0
```

### Konfigurera Microsoft® IIS {#configuring-microsoft-iis}

Konfigurera IIS för att integrera Dispatcher ISAPI-modulen. I IIS använder du programmappning med jokertecken.

### Konfigurera anonym åtkomst - IIS 8.5 och 10 {#configuring-anonymous-access-iis-and}

Standardagenten för tömningsreplikering på författarinstansen är konfigurerad så att den inte skickar säkerhetsuppgifter med tömningsbegäranden. Därför måste den webbplats där du använder Dispatcher-cachen tillåta anonym åtkomst.

Om webbplatsen använder en autentiseringsmetod måste Flush-replikeringsagenten konfigureras därefter.

1. Öppna IIS Manager och välj den webbplats som du använder som Dispatcher-cache.
1. Dubbelklicka på Autentisering i IIS-avsnittet i vyn Funktioner.
1. Om anonym autentisering inte är aktiverat väljer du Anonym autentisering och klickar på Aktivera i åtgärdsområdet.

### Integrera Dispatcher ISAPI-modulen - IIS 8.5 och 10 {#integrating-the-dispatcher-isapi-module-iis-and}

Använd följande procedur för att lägga till Dispatcher ISAPI-modulen i IIS.

1. Öppna IIS-hanteraren.
1. Markera webbplatsen som du använder som Dispatcher-cache.
1. Dubbelklicka på Hanterarmappningar i IIS-avsnittet i vyn Funktioner.
1. Klicka på Lägg till skriptmappning för jokertecken på åtgärdspanelen på sidan Hanterarmappningar, lägg till följande egenskapsvärden och klicka sedan på OK:

   * Sökväg till begäran: &#42;
   * Körbar fil: Den absoluta sökvägen för filen disp_iis.dll, till exempel `C:\inetpub\Scripts\disp_iis.dll`.
   * Namn: Ett beskrivande namn för hanterarmappningen, till exempel `Dispatcher`.

1. Klicka på **Ja** i dialogrutan som visas för att lägga till biblioteket disp_iis.dll i listan ISAPI- och CGI-begränsningar.

   För IIS 7.0 och 7.5 har konfigurationen slutförts. Fortsätt med de återstående stegen om du konfigurerar IIS 8.0.

1. (IIS 8.0) I listan med hanterarmappningar markerar du den som du skapade och klickar på Redigera i åtgärdsområdet.
1. (IIS 8.0) I dialogrutan Redigera skriptmappning klickar du på knappen Begär begränsningar.
1. (IIS 8.0) Om du vill vara säker på att hanteraren används för filer och mappar som ännu inte är cachelagrade avmarkerar du **Anropa endast hanterare om begäran är mappad till**. Klicka på **OK**.
1. (IIS 8.0) Klicka på OK i dialogrutan Redigera skriptmappning.

### Konfigurera åtkomst till cachen - IIS 8.5 och 10 {#configuring-access-to-the-cache-iis-and}

Ge standardanvändaren av programpoolen skrivåtkomst till den mapp som används som Dispatcher-cache.

1. Högerklicka på rotmappen för webbplatsen som du använder som Dispatcher-cache och klicka på Egenskaper, till exempel `C:\inetpub\wwwroot`.
1. Klicka på Redigera på fliken Dokumentskydd och sedan på Lägg till i dialogrutan Behörigheter. En dialogruta öppnas där du kan välja användarkonton. Klicka på knappen Platser, markera datornamnet och klicka sedan på OK.

   Håll den här dialogrutan öppen medan du slutför nästa steg.

1. i IIS Manager väljer du den IIS-webbplats som du använder för Dispatcher-cachen och klickar på Avancerade inställningar till höger i fönstret.
1. Markera värdet för programpoolsegenskapen och kopiera det till Urklipp.
1. Återgå till dialogrutan Öppna. Skriv `IIS AppPool\` i rutan Ange de objektnamn som ska markeras och klistra sedan in innehållet i Urklipp. Värdet ska se ut som i följande exempel:

   `IIS AppPool\DefaultAppPool`

1. Klicka på knappen Kontrollera namn. När Windows löser användarkontot klickar du på OK.
1. I dialogrutan Behörigheter för Dispatcher-mappen markerar du kontot som du just lade till, aktiverar alla behörigheter för kontot **förutom Fullständig behörighet** och klickar på OK. Klicka på OK så att du kan stänga dialogrutan Mappegenskaper.

### Registrerar JSON Mime-typen - IIS 8.5 och 10 {#registering-the-json-mime-type-iis-and}

Gör så här för att registrera JSON MIME-typen när du vill att Dispatcher ska tillåta JSON-anrop.

1. I IIS Manager väljer du din webbplats och dubbelklickar på Mime Types med hjälp av funktionsvyn.
1. Om JSON-tillägget inte finns med i listan klickar du på Lägg till på åtgärdspanelen, anger följande egenskapsvärden och klickar sedan på OK:

   * Filnamnstillägg: `.json`
   * MIME-typ: `application/json`

### Tar bort bin Dold Segment - IIS 8.5 och 10 {#removing-the-bin-hidden-segment-iis-and}

Använd följande procedur för att ta bort det dolda `bin`-segmentet. Webbplatser som inte är nya kan innehålla detta dolda segment.

1. I IIS Manager väljer du din webbplats och dubbelklickar på Begär filtrering med hjälp av funktionsvyn.
1. Markera segmentet `bin`, klicka på Ta bort och klicka på Ja i bekräftelsedialogrutan.

### Logga IIS-meddelanden till en fil - IIS 8.5 och 10 {#logging-iis-messages-to-a-file-iis-and}

Använd följande procedur för att skriva Dispatcher-loggmeddelanden till en loggfil i stället för till Windows-händelseloggen. Konfigurera Dispatcher för att använda loggfilen och ge IIS skrivåtkomst till filen.

1. Använd Utforskaren i Windows för att skapa en mapp med namnet `dispatcher` under loggmappen för IIS-installationen. Sökvägen till den här mappen för en typisk installation är `C:\inetpub\logs\dispatcher`.

1. Högerklicka på Dispatcher-mappen och klicka på **Egenskaper**.
1. Klicka på **Redigera** på fliken Säkerhet.
1. Klicka på **Lägg till** i dialogrutan Behörigheter. En dialogruta öppnas där du kan välja användarkonton. Klicka på knappen Platser, markera datornamnet och klicka sedan på OK.

   Håll den här dialogrutan öppen medan du slutför nästa steg.

1. i IIS Manager väljer du den IIS-webbplats som du använder för Dispatcher-cachen och klickar på Avancerade inställningar till höger i fönstret.
1. Markera värdet för programpoolsegenskapen och kopiera det till Urklipp.
1. Återgå till dialogrutan Öppna. Skriv `IIS AppPool\` i rutan Ange de objektnamn som ska markeras och klistra sedan in innehållet i Urklipp. Värdet ska se ut som i följande exempel:

   `IIS AppPool\DefaultAppPool`

1. Klicka på knappen Kontrollera namn. När Windows löser användarkontot klickar du på OK.
1. I dialogrutan Behörigheter för Dispatcher-mappen markerar du kontot som du just lade till, aktiverar alla behörigheter för kontot **förutom Fullständig behörighet,** och klickar på OK. Klicka på OK så att du kan stänga dialogrutan Mappegenskaper.
1. Öppna filen `disp_iis.ini` med en textredigerare.
1. Om du vill konfigurera platsen för loggfilen lägger du till en textrad som liknar följande exempel och sparar sedan filen:

   ```xml
   logfile=C:\inetpub\logs\dispatcher\dispatcher.log
   ```

### Nästa steg {#next-steps}

Innan du kan börja använda Dispatcher måste du känna till följande:

* [Konfigurera](dispatcher-configuration.md) Dispatcher
* [Konfigurera AEM](page-invalidate.md) för Dispatcher.

## Apache Web Server {#apache-web-server}

>[!CAUTION]
>
>Installationsanvisningar för både **Windows** och **UNIX®** beskrivs här. Var försiktig när du utför stegen.

### Installerar Apache Web Server {#installing-apache-web-server}

Information om hur du installerar en Apache-webbserver finns i installationsmanualen - antingen [online](https://httpd.apache.org/) eller i distributionen.

>[!CAUTION]
>
>Om du skapar en Apache-binär genom att kompilera källfilerna måste du aktivera **`dynamic modules support`**. Du kan aktivera det här alternativet med något av alternativen **- aktivera-delad**. Inkludera minst modulen `mod_so`.
>
>Mer information finns i installationsguiden för Apache Web Server.

Se även Apache HTTP Server [Säkerhetstips](https://httpd.apache.org/docs/2.4/misc/security_tips.html) och [Säkerhetsrapporter](https://httpd.apache.org/security_report.html).

### Apache Web Server - Lägg till Dispatcher Module {#apache-web-server-add-the-dispatcher-module}

Dispatcher finns som:

* **Windows**: ett DLL-bibliotek (Dynamic Link Library)
* **UNIX®**: ett dynamiskt delat objekt (DSO)

Installationsarkivfilerna innehåller följande filer - beroende på om du har valt Windows eller UNIX®:

| Fil | Beskrivning |
|--- |--- |
| disp_apache&lt;x.y>.dll | Windows: Dispatcher Dynamic Link Library-fil. |
| dispatcher-apache&lt;x.y>-&lt;rel-nr>.so | UNIX®: Dispatcher delade objektbiblioteksfil. |
| mod_dispatcher.so | UNIX®: En exempellänk. |
| http.conf.disp&lt;x> | En exempelkonfigurationsfil för Apache-servern. |
| dispatcher.any | En exempelkonfigurationsfil för Dispatcher. |
| README | Viktigt-fil som innehåller installationsinstruktioner och sista-minuten-information. **Obs!** Kontrollera den här filen innan du startar installationen. |
| ÄNDRINGAR | Ändrar en fil med en lista över problem som har korrigerats i den aktuella och tidigare versionen. |

Gör så här för att lägga till Dispatcher i din Apache-webbserver:

1. Placera Dispatcher-filen i lämplig katalog för Apache-modulen:

   * **Windows**: Montera `disp_apache<x.y>.dll` `<APACHE_ROOT>/modules`
   * **UNIX®**: Leta reda på katalogen `<APACHE_ROOT>/libexec` eller `<APACHE_ROOT>/modules` enligt din installation.\
     Kopiera `dispatcher-apache<options>.so` till den här katalogen.\
     Om du vill förenkla det långsiktiga underhållet kan du även skapa en symbolisk länk med namnet `mod_dispatcher.so` till Dispatcher:\
     `ln -s dispatcher-apache<x>-<os>-<rel-nr>.so mod_dispatcher.so`

1. Kopiera dispatcher.any-filen till katalogen `<APACHE_ROOT>/conf`.

   **Obs!** Du kan placera den här filen på en annan plats, förutsatt att egenskapen DispatcherLog för Dispatcher-modulen har konfigurerats på rätt sätt. (Se Dispatcher-specifika konfigurationsposter nedan.)

### Apache Web Server - Konfigurera SELinux-egenskaper {#apache-web-server-configure-selinux-properties}

Om du kör Dispatcher på Red Hat® Linux® Kernel 2.6 med SELinux aktiverat kan felmeddelanden som detta visas i Dispatcher loggfil.

`Mon Jun 30 00:03:59 2013] [E] [16561(139642697451488)] Unable to connect to backend rend01 (10.122.213.248:4502): Permission denied`

Detta fel beror troligen på en aktiverad SELinux-säkerhet. Gör i så fall följande:

* Konfigurera SELinux-kontexten för Dispatcher-modulfilen.
* Aktivera HTTPD-skript och -moduler för att skapa nätverksanslutningar.
* Konfigurera SELinux-kontexten för dokumentroten, där de cachelagrade filerna lagras.

Ange följande kommandon i ett terminalfönster och ersätt `[path to the dispatcher.so file]` med sökvägen till Dispatcher-modulen som du installerade på Apache Web Server, och *`path to the docroot`* med sökvägen till dokumentroten (till exempel `/opt/cq/cache`):

```shell
semanage fcontext -a -t httpd_modules_t [path to the dispatcher.so file]
setsebool -P httpd_can_network_connect on
chcon -R --type httpd_sys_rw_content_t [path to the docroot]
semanage fcontext -a -t httpd_sys_rw_content_t "[path to the docroot](/.*)?"
```

### Apache Web Server - Konfigurera Apache Web Server för Dispatcher {#apache-web-server-configure-apache-web-server-for-dispatcher}

Apache-webbservern måste konfigureras med `httpd.conf`. I Dispatcher installationspaket finns ett exempel på en konfigurationsfil med namnet `httpd.conf.disp<x>`.

Dessa steg är obligatoriska:

1. Navigera till `<APACHE_ROOT>/conf`.
1. Öppna `httpd.conf` för redigering.
1. Följande konfigurationsposter måste läggas till i den ordning som anges:

   * **LoadModule** för att läsa in modulen vid start.
   * Dispatcher-specifika konfigurationsposter, inklusive **DispatcherConfig, DispatcherLog** och **DispatcherLogLevel**.
   * **SetHandler** för att aktivera Dispatcher. **LoadModule**.
   * **ModMimeUsePathInfo** för att konfigurera beteendet för **mod_mime**.

1. (Valfritt) Vi rekommenderar att du ändrar ägaren till htdocs-katalogen:

   * Apache-servern startar som rot, men de underordnade processerna startar som daemon (av säkerhetsskäl). DocumentRoot (`<APACHE_ROOT>/htdocs`) måste tillhöra användarens daemon:

     ```xml
     cd <APACHE_ROOT>  
     chown -R daemon:daemon htdocs
     ```

**LoadModule**

I följande tabell visas exempel som kan användas. De exakta posterna är enligt din specifika Apache-webbserver:

|  |  |
|--- |--- |
| Windows | `... LoadModule dispatcher_module modules\disp_apache.dll ...` |
| UNIX® (antar symbolisk länk) | `... LoadModule dispatcher_module libexec/mod_dispatcher.so ...` |

>[!NOTE]
>
>Den första parametern för varje programsats måste skrivas exakt som i exemplen ovan.
>
>Se exempelkonfigurationsfilerna och dokumentationen för Apache Web Server för mer information om det här kommandot.

**Dispatcher-specifika konfigurationsposter**

De Dispatcher-specifika konfigurationsposterna placeras efter LoadModule-posten. I följande tabell visas en exempelkonfiguration som gäller för både UNIX® och Windows:

**Windows &amp; UNIX®**

```
...
<IfModule disp_apache2.c>
DispatcherConfig conf/dispatcher.any
DispatcherLog logs/dispatcher.log DispatcherLogLevel 3
DispatcherNoServerHeader 0 DispatcherDeclineRoot 0
DispatcherUseProcessedURL 0
DispatcherPassError 0
DispatcherKeepAliveTimeout 60
</IfModule>
...
```

>[!NOTE]
>
>Kunder som uppgraderar specifikt från version 4.3.3 till version 4.3.4 bör observera ett annat beteende i hur cachelagring av rubriker är inställt för innehåll som inte kan cachelagras. Mer information om den här ändringen finns på sidan [Versionsinformation](/help/using/release-notes.md#nov).

De enskilda konfigurationsparametrarna:

| Parameter | Beskrivning |
|--- |--- |
| DispatcherConfig | Plats och namn för Dispatcher-konfigurationsfilen. <br/>När den här egenskapen finns i huvudserverkonfigurationen ärver alla virtuella värdar egenskapsvärdet. Virtuella värdar kan dock innehålla en DispatcherConfig-egenskap som åsidosätter huvudserverkonfigurationen. |
| DispatcherLog | Loggfilens plats och namn. |
| DispatcherLogLevel | Loggnivå för loggfilen: <br/> 0 - Fel <br/> 1 - Varningar <br/> 2 - Information <br/> 3 - Felsökning <br/>**Obs!**: Ange loggnivån till 3 under installation och testning och sedan till 0 när loggen körs i en produktionsmiljö. |
| DispatcherNoServerHeader | *Den här parametern är föråldrad och ineffektiv.*<br/><br/> Definierar den serverrubrik som ska användas: <br/><ul><li>undefined eller 0 - HTTP-serverhuvudet innehåller AEM. </li><li>1 - Apache-serverhuvudet används.</li></ul> |
| DispatcherDeclineRoot | Definierar om begäranden till roten &quot;/&quot; ska nekas: <br/>**0** - acceptera begäranden till / <br/>**1** - Dispatcher hanterar inte begäranden till /. Använd i stället mod_alias för korrekt mappning. |
| DispatcherUseProcsedURL | Definierar om förbearbetade URL:er ska användas för all vidare bearbetning av Dispatcher: <br/>**0** - använd den ursprungliga URL:en som skickades till webbservern. <br/>**1** - Dispatcher använder den URL som redan har bearbetats av de hanterare som föregår Dispatcher (d.v.s. `mod_rewrite`) i stället för den ursprungliga URL-adress som skickades till webbservern. Till exempel matchas antingen den ursprungliga eller den bearbetade URL:en med Dispatcher-filter. URL:en används också som bas för cachefilens struktur. Information om mod_rewrite; till exempel Apache 2.4 finns i dokumentationen för Apache-webbplatsen. När du använder mod_rewrite använder du flaggan genomgång (skicka till nästa hanterare) för att tvinga omskrivningsmotorn att ange URI-fältet för den interna strukturen request_rec till värdet för filnamnsfältet. |
| DispatcherPassError | Definierar hur felkoder ska stödjas för feldokumenthantering: <br/>**0** - Dispatcher tolkar alla felsvar till klienten. <br/>**1** - Dispatcher buffrar inte ett felsvar till klienten (där statuskoden är större än eller lika med 400). I stället skickas statuskoden till Apache, vilket gör att ett ErrorDocument-direktiv kan bearbeta en sådan statuskod. <br/>**Kodintervall** - Ange ett intervall med felkoder som svaret skickas till Apache för. Andra felkoder skickas till klienten. Följande konfiguration skickar till exempel svar för fel 412 till klienten och alla andra fel skickas till Apache: DispatcherPassError 400-411,413-417 |
| DispatcherKeepAliveTimeout | Anger tidsgränsen för keep-alive i sekunder. Från och med Dispatcher version 4.2.0 är standardvärdet 60. Värdet 0 inaktiverar keep-alive. |
| DispatcherNoCanonURL | Om du anger den här parametern som På skickas den rå URL:en till serverdelen i stället för till den kanoniserade URL:en och inställningarna för DispatcherUseProcsedURL åsidosätts. Standardvärdet är Av. <br/>**Obs!**: Filterreglerna i Dispatcher-konfigurationen utvärderas alltid mot den sanerade URL:en, inte mot den obearbetade URL:en. |

>[!NOTE]
>
>Sökvägsposterna är relativa till rotkatalogen för Apache-webbservern.

>[!NOTE]
>
>Standardinställningarna för Serverhuvud är:
>
>`ServerTokens Full`
>
>`DispatcherNoServerHeader 0`
>
>Här visas AEM version för statistiska ändamål. Om du vill inaktivera sådan information i sidhuvudet kan du ange följande:
>
>`ServerTokens Prod`
>
>Mer information finns i [Apache-dokumentationen om ServerTokens-direktivet (till exempel för Apache 2.4)](https://httpd.apache.org/docs/2.4/mod/core.html) .

**SetHandler**

Efter dessa poster måste du lägga till en **SetHandler**-sats i kontexten för din konfiguration ( `<Directory>`, `<Location>`) för att Dispatcher ska kunna hantera inkommande begäranden. I följande exempel konfigureras Dispatcher för att hantera begäranden för hela webbplatsen:

**Windows och UNIX®**

```
...  
<Directory />  
<IfModule disp_apache2.c>  
SetHandler dispatcher-handler  
</IfModule>  
  
Options FollowSymLinks  
AllowOverride None  
</Directory>  
...
```

I följande exempel konfigureras Dispatcher för att hantera begäranden för en virtuell domän:

**Windows**

```
...  
<VirtualHost 123.45.67.89>  
ServerName www.mycompany.com  
DocumentRoot _\[cache-path\]_\\docs  
<Directory _\[cache-path\]_\\docs>  
<IfModule disp_apache2.c>  
SetHandler dispatcher-handler  
</IfModule>  
AllowOverride None  
</Directory>  
</VirtualHost>  
...
```

**UNIX®**

```
...  
<VirtualHost 123.45.67.89>  
ServerName www.mycompany.com  
DocumentRoot /usr/apachecache/docs  
<Directory /usr/apachecache/docs>  
<IfModule disp_apache2.c>  
SetHandler dispatcher-handler  
</IfModule>  
AllowOverride None  
</Directory>  
</VirtualHost>  
...
```

>[!NOTE]
>
>Parametern för programsatsen **SetHandler** måste skrivas *exakt som ovanstående exempel* eftersom det är namnet på hanteraren som definieras i modulen.
>
>Se exempelkonfigurationsfilerna och dokumentationen för Apache Web Server för mer information om det här kommandot.

**ModMimeUsePathInfo**

Efter programsatsen **SetHandler** bör du även lägga till definitionen **ModMimeUsePathInfo** .

>[!NOTE]
>
>Använd och konfigurera bara parametern `ModMimeUsePathInfo` om du använder Dispatcher version 4.0.9 eller senare.
>
>Dispatcher version 4.0.9 släpptes 2011. Om du använder en äldre version bör du uppgradera till en nyare version av Dispatcher.

Parametern **ModMimeUsePathInfo** måste anges `On` för alla Apache-konfigurationer:

`ModMimeUsePathInfo On`

Modulen mod_mime (till exempel [Apache Module mod_mime](https://httpd.apache.org/docs/2.4/mod/mod_mime.html)) används för att tilldela innehållsmetadata till det innehåll som valts för ett HTTP-svar. Standardinställningen innebär att `mod_mime` bestämmer innehållstypen. Därför beaktas endast den del av URL:en som mappar till en fil eller katalog.

När `On` anges av parametern `ModMimeUsePathInfo` att `mod_mime` ska avgöra innehållstypen baserat på URL:en *complete*. Det innebär att metadata för virtuella resurser används baserat på deras tillägg.

I följande exempel aktiveras **ModMimeUsePathInfo**:

**Windows och UNIX®**

```
...  
<Directory />  
<IfModule disp_apache2.c>  
SetHandler dispatcher-handler  
ModMimeUsePathInfo On  
</IfModule>  
  
Options FollowSymLinks  
AllowOverride None  
</Directory>  
...
```

### Aktivera stöd för HTTPS (UNIX® och Linux®) {#enable-support-for-https-unix-and-linux}

Dispatcher använder OpenSSL för att implementera säker kommunikation via HTTP. Från och med Dispatcher version **4.2.0** stöds OpenSSL 1.0.0 och OpenSSL 1.0.1. Dispatcher använder OpenSSL 1.0.0 som standard. Om du vill använda OpenSSL 1.0.1 skapar du symboliska länkar så att Dispatcher använder de installerade OpenSSL-biblioteken.

1. Öppna en terminal och ändra den aktuella katalogen till katalogen där OpenSSL-biblioteken är installerade, till exempel:

   ```shell
   cd /usr/lib64
   ```

1. Om du vill skapa symboliska länkar anger du följande kommandon:

   ```shell
   ln -s libssl.so libssl.so.1.0.1
   ln -s libcrypto.so libcrypto.so.1.0.1
   ```

>[!NOTE]
>
>Om du använder en anpassad version av Apache måste du se till att Apache och Dispatcher kompileras med samma version av [OpenSSL](https://www.openssl.org/source/).

### Nästa steg {#next-steps-1}

Innan du kan börja använda Dispatcher måste du nu göra följande:

* [Konfigurera](dispatcher-configuration.md) Dispatcher
* [Konfigurera AEM](page-invalidate.md) för Dispatcher.

## Sun Java™ System Web Server/iPlanet {#sun-java-system-web-server-iplanet}

>[!NOTE]
>
>Instruktioner för både Windows- och UNIX®-miljöer finns här.
>
>Var försiktig när du väljer vilken som ska köras.

### Sun Java™ System Web Server/iPlanet - installera webbservern {#sun-java-system-web-server-iplanet-installing-your-web-server}

Mer information om hur du installerar dessa webbservrar finns i respektive dokumentation:

* Sun Java™ System Web Server
* Webbserver för iPlanet

### Sun Java™ System Web Server / iPlanet - lägg till Dispatcher Module {#sun-java-system-web-server-iplanet-add-the-dispatcher-module}

Dispatcher finns som:

* **Windows**: ett DLL-bibliotek (Dynamic Link Library)
* **UNIX®**: ett dynamiskt delat objekt (DSO)

Installationsarkivfilerna innehåller följande filer - beroende på om du har valt Windows eller UNIX®:

| Fil | Beskrivning |
|---|---|
| `disp_ns.dll` | Windows: Dispatcher Dynamic Link Library-fil. |
| `dispatcher.so` | UNIX®: Dispatcher delade objektbiblioteksfil. |
| `dispatcher.so` | UNIX®: En exempellänk. |
| `obj.conf.disp` | En exempelkonfigurationsfil för webbservern iPlanet/Sun Java™ System. |
| `dispatcher.any` | En exempelkonfigurationsfil för Dispatcher. |
| README | Viktigt-fil som innehåller installationsinstruktioner och sista-minuten-information. **Obs!** Kontrollera den här filen innan du startar installationen. |
| ÄNDRINGAR | Ändrar en fil med en lista över problem som har korrigerats i den aktuella och tidigare versionen. |

Gör så här för att lägga till Dispatcher på webbservern:

1. Placera Dispatcher-filen i webbserverns `plugin`-katalog:

### Sun Java™ System Web Server / iPlanet - Konfigurera för Dispatcher {#sun-java-system-web-server-iplanet-configure-for-the-dispatcher}

Webbservern måste konfigureras med `obj.conf`. I Dispatcher installationspaket finns ett exempel på en konfigurationsfil med namnet `obj.conf.disp`.

1. Navigera till `<WEBSERVER_ROOT>/config`.
1. Öppna `obj.conf` för redigering.
1. Kopiera raden som börjar:\
   `Service fn="dispService"`\
   från `obj.conf.disp` till initieringsavsnittet för `obj.conf`.

1. Spara ändringarna.
1. Öppna `magnus.conf` för redigering.
1. Kopiera de två rader som börjar:\
   `Init funcs="dispService, dispInit"`\
   och\
   `Init fn="dispInit"`\
   från `obj.conf.disp` till initieringsavsnittet för `magnus.conf`.

1. Spara ändringarna.

>[!NOTE]
>
>Följande konfigurationer ska vara på en rad. Dessutom måste `$(SERVER_ROOT)` och `$(PRODUCT_SUBDIR)` ersättas med sina respektive värden.

**Init**

I följande tabell visas exempel som kan användas. De exakta posterna är beroende av din specifika webbserver:

**Windows och UNIX®**

```
...  
Init funcs="dispService,dispInit" fn="load-modules" shlib="$(SERVER\_ROOT)/plugins/dispatcher.so"  
Init fn="dispInit" config="$(PRODUCT\_SUBDIR)/dispatcher.any" loglevel="1" logfile="$(PRODUCT\_SUBDIR)/logs/dispatcher.log"  
keepalivetimeout="60"  
...
```

Var:

| Parameter | Beskrivning |
|--- |--- |
| `config` | Konfigurationsfilens plats och namn `dispatcher.any.` |
| `logfile` | Loggfilens plats och namn. |
| `loglevel` | Loggnivå för när meddelanden skrivs till loggfilen: <br/>**0** Fel <br/>**1** Varning <br/>**** **Information<br/>**  Felsökning <br/>**Obs!** Ange loggnivån till 3 under installation och testning och till 0 vid körning i en produktionsmiljö. |
| `keepalivetimeout` | Anger tidsgränsen för keep-alive i sekunder. Från och med Dispatcher version 4.2.0 är standardvärdet 60. Värdet 0 inaktiverar keep-alive. |

Beroende på dina behov kan du definiera Dispatcher som en tjänst för dina objekt. Om du vill konfigurera Dispatcher för hela webbplatsen redigerar du standardobjektet:

**Windows**

```
...  
NameTrans fn="document-root" root="$(PRODUCT\_SUBDIR)\\dispcache"  
...  
Service fn="dispService" method="(GET|HEAD|POST)" type="\*\\\*"  
...
```

**UNIX®**

```
...  
NameTrans fn="document-root" root="$(PRODUCT\_SUBDIR)/dispcache"  
...  
Service fn="dispService" method="(GET|HEAD|POST)" type="\*/\*"  
...
```

### Nästa steg {#next-steps-2}

Innan du kan börja använda Dispatcher måste du nu:

* [Konfigurera](dispatcher-configuration.md) Dispatcher
* [Konfigurera AEM](page-invalidate.md) för Dispatcher.
