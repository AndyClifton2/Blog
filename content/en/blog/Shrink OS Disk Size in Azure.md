---
Author: "Andy Clifton"
title: "Verklein OS Disk Size in Azure"
description: "Verklein OS Disk Size in Azur"
date: 2024-05-02
tags: ["emoji"]
thumbnail: "/img/Resize.png"
---

# Verkleinen OS Disk Size in Azure

### ==> Disclaimer work in progress <==

## Voorwoord

Een OS disk die te groot aangemaakt is, het blijft een lastig probleem. Het zijn onnodige kosten binnen azure. en het aanpassen van de Size van een Azure OS disk kan niet zomaar. Je krijgt dan de melding:
![Image](/Images/ShrinkOS/ReduceSize.png)

Toch zijn er mogelijkheden om dit aan te passen en daardoor kosten te besparen. De eis om dit te kunnen doen is uiteraard dat er genoeg schijfruimte aanwezig op de Disk in de VM om dit kunnen doen.

Dit probleem kom je zeer regelmatig tegen op het moment dat er vanuit een On-premise omgeving een server gemigreerd word. (grote OS disk voor groeien vast aangemaakt).
Als er voldoende vrije disk ruimte aanwezig is om de schijf te brengen naar 64Gb of 32Gb is dit mogelijk.
Hou wel rekening met de aanpassing van IOPS en Throughput op het moment dat disks kleiner gemaakt word.
Hieronder de stappen om deze aanpassing te maken.


## Kleiner maken van de disk in de VM.

Log in op de server via RDP.



![Image](/Images/PasswordProtection/AAD.png)

Open **Authentication methods**
.

![Image](/Images/PasswordProtection/AuthenticationMethods.png)

4)	Nu wordt er een nieuwe window geopend om password protection setting te configureren.

Vul in: 
~~~
Lockout Threshold = hoeveel pogingen voor het account locked (by default op 10)
Lockout duration in seconds = Tijd dat je niet kunt inloggen (by default op 60 seconden)

In Custom banned password :
Enforce custom list  = Yes
Custom banned password list = hier vul je alle wachtwoorden in waarvan je niet wilt dat ze toegevoegd worden, 
                              denk aan password, welkom, bedrijfsnaam, voornamen, achternamen, postcodes, etc

~~~

![Image](/Images/PasswordProtection/customsmartlockout.png)

5)	Om de policy ook door te voeren op je on-premise AD omgeving vul je het volgende in:

~~~
Enable password protection on Windows Server Active Directory = Yes
Mode = Enforced of Audit (bij audit zal hij alleen loggings maken)
~~~
![Image](/Images/PasswordProtection/PPAD.png)

Druk op **Save** hiermee sla je alle wijzigingen op.

![Image](/Images/PasswordProtection/save.png)


## Installeren Azure agents op on-premise servers.

Om de password protection ook toe te passen op je on-premise omgeving moeten we 2 azure componenten installeren.

1. Azure AD password protection proxy service
2. Azure AD password protection DC agent

De manier van werken van deze agents worden perfect uitgelegd in onderstaande foto.
![Image](/Images/PasswordProtection/adpp.png)

Voordat we de Azure AD password protection service gaan installeren moeten we het volgende bedenken.
~~~
-	Op dit moment mag je 2 proxy servers installeren onder 1 forest.
-	Het wordt ondersteund om het te installeren op een DC maar dan moet deze server wel een internet connectie hebben.
-	Om de service te kunnen registreren moet je Domain Admin rechten hebben.
-	Proxy servers moeten constant een communicatiemogelijkheid hebben tot de DC agent.
-	Je moet een Tenant administrator account hebben om in Azure de proxy’s te kunnen configureren.
~~~
Start van de installatie:


1. **Log in** op de server als Domain Admin
2. Ga naar deze [hier](https://www.microsoft.com/download/details.aspx?id=57071)  en download AzureADPasswordProtectionProxy.msi
![Image](/Images/PasswordProtection/download.png)

3.Dubbelklik op de msi file en start de installatie.

  ![Image](/Images/PasswordProtection/msi.png)

4. Om te kijken of de service draait: open powershell en vul de volgende rules in:

````
Import-Module AzureADPasswordProtection
Get-Service AzureADPasswordProtectionProxy | fl

````
![Image](/Images/PasswordProtection/powershell.png)

5. nu moeten we de proxy gaan registreren.Doe dit in powershell met de volgende rule:

````
$ADadmin = Get-Credential ( Je hebt hiervoor je Global admin account nodig)
Register-AzureADPasswordProtectionProxy -AzureCredential $ADadmin.
````
Volgende stap is het installeren van de Azure AD password protection DC agent te installeren (deze moet op een domain controller geïnstalleerd worden, deze installatie heeft een reboot nodig).

6. **Login** op de server als Domain admin
7.  Ga naar deze [hier](https://www.microsoft.com/download/details.aspx?id=57071)  en download AzureADPasswordProtectionDCAgent.msi.
8.Dubbelklik op de msi file en start de installatie.
![Image](/Images/PasswordProtection/msi2.png)
9. Na de installatie heb je een reboot nodig

![Image](/Images/PasswordProtection/reboot.png)

Nu is de installatie klaar en kun je testen of de bestaande wachtwoorden geweigerd worden.


