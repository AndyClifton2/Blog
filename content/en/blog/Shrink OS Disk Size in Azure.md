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



![Image](/Images/ShrinkOS/rdp.png)

Als je ingelogd bent ga je naar Disk Management.
.

![Image](/Images/ShrinkOS/disk.png)

Als diskmanagement geopend is kijk dan bij de C schijf hoeveel vrije ruimte je hebt. Zorg ervoor dat je genoeg over houd voor een groei. (+/- 15/20% over is voldoende)

![Image](/Images/ShrinkOS/diskc.png)

Als we kijken in disk management zien we onderstaande disk nummer staan. onthou of bewaar deze, deze hebben we in het vervolg nog nodig.

![Image](/Images/ShrinkOS/disk0.png)
Open vervolgens powershell en vul de onderstaande regel in om je disk te shrinken.

~~~
Get-Partition -DiskNumber 0  (het disknummer is hetzelfde nummer als dat je hierboven hebt gevonden.)

~~~


![Image](/Images/ShrinkOS/disknumber.png)

In bovenstaande voorbeeld zien we dat disknummer 4 de disk is die we moeten gebruiken omdat hier de C schijf op staat.
In ons voorbeeld verkleinen we de disk naar 90Gb. Maar je kunt natuurlijk zelf kiezen wat voor jou het beste past.
Vul volgende regel in in Powershell: 

~~~
Get-Partition -DiskNumber 0 -PartitionNumber 4 | Resize-Partition -Size 90GB

~~~


Soms krijg je onderstaande error:

![Image](/Images/ShrinkOS/error.png)
~~~
The specified shrink size is too big and will cause the volume to be smaller than the minimum volume size
~~~

De oplossing is eigenlijk vrij simpel, probeer de powershell regel nog 2/3 keer en dan lukt het wel. dit omdat er gequery word en er soms files in een bepaald block in gebruik zijn.

![Image](/Images/ShrinkOS/disknew.png)

Zet de machine uit via Shutdown:

![Image](/Images/ShrinkOS/shutdown.png)

Als de machine uitstaat moeten we hem ook nog deallocate.
Ga naar de azure portal en zoek op Virtual Machines.

![Image](/Images/ShrinkOS/VirtualMachines.png)

Zoek op de virtuele machine en klik op Stop.
![Image](/Images/ShrinkOS/stop.png)