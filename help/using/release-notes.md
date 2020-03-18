---
title: Versionsinformation för AEM Dispatcher
seo-title: Versionsinformation för AEM Dispatcher
description: Versionsinformation om Adobe Experience Manager Dispatcher
seo-description: Versionsinformation om Adobe Experience Manager Dispatcher
uuid: ae3ccf62-0514-4c03-a3b9-71799a482cbd
topic-tags: release-notes
content-type: reference
products: SG_EXPERIENCEMANAGER/6.4
discoiquuid: ff3d38e0-71c9-4b41-85f9-fa896393aac5
translation-type: tm+mt
source-git-commit: 328bc82673783b4a2df2d68481fa7eec88b74b01

---


# Versionsinformation för AEM Dispatcher{#aem-dispatcher-release-notes}

## Versionsinformation {#release-information}

|  |  |
|--- |--- |
| Produkter | Adobe Experience Manager (AEM) Dispatcher |
| Version | 4.3.3 |
| Typ | Mindre release |
| Date | 18 oktober 2019 |
| Hämta URL | <ul><li>[Apache 2.4](release-notes.md#apache)</li><li>[Microsoft Internet Information Services (IIS)](release-notes.md#iis)</li></ul> |
| Kompatibilitet | AEM 6.1 eller senare |

## Systemkrav och krav {#system-requirements-and-prerequisites}

Mer information om krav och krav finns på sidan [Plattformar](https://helpx.adobe.com/experience-manager/6-4/sites/deploying/using/technical-requirements.html) som stöds.

Adobe rekommenderar starkt att du använder den senaste versionen av AEM Dispatcher för att få tillgång till de senaste funktionerna, de senaste felkorrigeringarna och bästa möjliga prestanda.

## Installationsanvisningar {#installation-instructions}

Detaljerade anvisningar finns i [Installera Dispatcher](dispatcher-install.md).

## Versionshistorik {#release-history}

### Version 4.3.3 (2019-Oct-18) {#october}

**Felkorrigeringar**:

* DISP-739 - LogLevel-dispatcher: **nivån** fungerar inte
* DISP-749 - Alpine Linux-dispatchern kraschar med spårloggsnivå

**Förbättringar**:

* DISP-813 - Stöd i Dispatcher för openssl 1.1.x
* DISP-814 - Apache 40x-fel under cachetömningar
* DISP-818 - mod_expirres lägger till Cache-Control-rubriker för innehåll som inte kan cache-lagras
* DISP-821 - Lagra inte loggkontext i socket
* DISP-822 - Dispatcher ska använda pell i stället för pselect
* DISP-824 - Secure DispatcherUseForwardedHost
* DISP-825 - Logga specialmeddelanden när det inte finns mer utrymme på disken
* DISP-826 - Stöd för uppdatering av URI:er med en frågesträng

**Nya funktioner**:

* DISP-703 - Träffgrad för servergruppsspecifik cache
* DISP-827 - lokal server för testning
* DISP-828 - Skapa testdockarbild för dispatcher

### Version 4.3.2 (2019-Jan-31) {#jan}

**Felkorrigeringar**:

* DISP-734 - Dispatcher orsakar krasch i insert_output_filter om det inte anges som hanterare
* DISP-735 - REs fungerar inte i Alpine Linux
* DISP-740 - Inläsning av dispatcher i macOS Mojave är inaktiverat som standard
* DISP-742 - Blockerade begäranden kan läcka information för att autentisera skyddade resurser

**Förbättringar**:

* DISP-746 - Omärkta strängar i dispatcher.any bör generera en varning

**Ny funktion**:

* DISP-747 - Ange begärandeinformation i Apache-miljön

### Version 4.3.1 (2018-Oct-16) {#oct}

**Felkorrigeringar**:

* DISP-656 - Dispatcher returnerar fel ETag Header
* DISP-694 - Utelämna varningar när aktiva anslutningar förblir oförändrade
* DISP-714 - Cookie-baserad sessionshantering fungerar inte i IIS
* DISP-715 - Säker flagga för återgiven cookie
* DISP-720 - Temporära filer som inte stängs kan leda till att filen blir slut (för många öppna filer)
* DISP-721 - Dispatcher avbryter poll() när Apache startar om underordnat
* DISP-722 - Cachefiler skapas med oktalt läge 0600
* DISP-723 - implicit 10-minuters timeout (och försök igen) när återgivningens tidsgräns är 0
* DISP-725 - Efterföljande tecken efter strängar konverteras tyst till namnlöst värde
* DISP-726 - Logga en varning när ingen servergrupp faktiskt matchar den inkommande värden
* DISP-727 - Dispatcher kontrollerar innehållslängden för begäranden om tomma cachefiler
* DISP-730-404 vid försök att komma åt rubrikfil via dispatcher
* DISP-731 - Dispatcher är sårbar för Log Injection
* DISP-732 - Dispatcher ska ta bort efterföljande&quot;/&quot; i URL:en
* DISP-733 - Dispatcher ska ange (beräkna) en åldersrubrik

**Förbättringar**:

* DISP-656 - Dispatcher returnerar fel ETag Header
* DISP-694 - Utelämna varningar när aktiva anslutningar förblir oförändrade
* DISP-715 - Säker flagga för återgiven cookie
* DISP-722 - Cachefiler skapas med oktalt läge 0600
* DISP-726 - Logga en varning när ingen servergrupp faktiskt matchar den inkommande värden

### Version 4.3.0 (2018-Jun-13) {#jun}

**Felkorrigeringar**:

* DISP-682 - Numerisk loggnivå används felaktigt
* DISP-685 - 32-bitars Solaris SPARC-binärfiler har en odefinierad referens till __divdi3
* DISP-688 - Dispatcher returnerar inte X-Cache-Info-huvudet på 404-svar
* DISP-690 - Rubriken Senast ändrad är inte tillgänglig
* DISP-691 - åtkomstfel i w3wp.exe
* DISP-693 - Behöver uppdatera arkitekturinformation för solarisservrar på hämtningssidan för dispatcher
* DISP-695 - Problem med DispatcherLog-nivån i Dispatcher-modulen 4.2.3
* DISP-698 - Dispatcher TTL måste ha stöd för s-maxage och privata direktiv
* DISP-700 - Modulen fungerar inte korrekt i Alpine Linux
* DISP-704 - Webbläsarbegäranden som innehåller %2b skickas till utgivaren utan kodning
* DISP-705 - Dispatcher kraschar på grund av att programmet är dubbelt ledigt eller skadat (snabbast)
* DISP-706 - Vid ogiltigförklaring följer dispatchern bakåtreferenssymboler som kan orsaka en oändlig slinga
* DISP-709 - Blockera vissa tillägg för innehålls-URL
* DISP-710 - Bygger för Linux som inte kan användas i Cent OS 6

**Förbättringar**:

* DISP-652 - Dispatcher visar fel datumhuvud

## Användbara resurser {#helpful-resources}

* [Översikt över AEM Dispatcher](dispatcher.md)

## Nedladdningar {#downloads}

### Apache 2.4 {#apache}

| Plattform | Arkitektur | Stöd för OpenSSL | Hämta |
|---|---|---|---|
| Linux | i686 (32-bitars) | Inget | [dispatcher-apache2.4-linux-i686-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-4.3.3.tar.gz) |
| Linux | i686 (32-bitars) | 1.0 | [dispatcher-apache2.4-linux-i686-ssl1.0-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-ssl1.0-4.3.3.tar.gz) |
| Linux | i686 (32-bitars) | 1.1 | [dispatcher-apache2.4-linux-i686-ssl1.1-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-ssl1.1-4.3.3.tar.gz) |
| Linux | x86_64 (64-bitars) | Inget | [dispatcher-apache2.4-linux-x86_64-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-4.3.3.tar.gz) |
| Linux | x86_64 (64-bitars) | 1.0 | [dispatcher-apache2.4-linux-x86_64-ssl1.0-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-ssl1.0-4.3.3.tar.gz) |
| Linux | x86_64 (64-bitars) | 1.1 | [dispatcher-apache2.4-linux-x86_64-ssl1.1-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-ssl1.1-4.3.3.tar.gz) |
| macOS | x86_64 (64-bitars) | Inget | [dispatcher-apache2.4-darwin-x86_64-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-darwin-x86_64-4.3.3.tar.gz) |

### IIS {#iis}

| Plattform | Arkitektur | Stöd för OpenSSL | Hämta |
|---|---|---|---|
| Windows | x86 (32-bitars) | Inget | [dispatcher-iis-windows-x86-4.3.3.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-4.3.3.zip) |
| Windows | x86 (32-bitars) | 1.0 | [dispatcher-iis-windows-x86-ssl1.0-4.3.3.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-ssl1.0-4.3.3.zip) |
| Windows | x86 (32-bitars) | 1.1 | [dispatcher-iis-windows-x86-ssl1.1-4.3.3.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-ssl1.1-4.3.3.zip) |
| Windows | x64 (64-bitars) | Inget | [dispatcher-iis-windows-x64-4.3.3.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-4.3.3.zip) |
| Windows | x64 (64-bitars) | 1.0 | [dispatcher-iis-windows-x64-ssl1.0-4.3.3.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-ssl1.0-4.3.3.zip) |
| Windows | x64 (64-bitars) | 1.1 | [dispatcher-iis-windows-x64-ssl1.1-4.3.3.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-ssl1.1-4.3.3.zip) |