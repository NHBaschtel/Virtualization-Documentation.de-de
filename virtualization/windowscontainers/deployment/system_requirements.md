---
author: neilpeterson
---

# Anforderungen von Windows-Containern

**Dieser Inhalt ist vorläufig und kann geändert werden.**

In diesen Handbüchern sind die Anforderungen für einen Windows-Containerhost aufgeführt.

## Windows-Container in einem physischen System

- Die Rolle „Windows-Container“ ist nur unter Windows Server 2016 TP4 (Vollversion und Core) und Nano Server verfügbar.
- Wenn Hyper-V-Container ausgeführt werden, muss die Rolle „Hyper-V“ installiert werden.

## Windows-Container in einem virtuellen System

Wenn ein Windows-Containerhost von einem virtuellen Hyper-V-Computer ausgeführt wird und auch Hyper-V-Container hostet, muss die geschachtelte Virtualisierung aktiviert werden. Für die geschachtelte Virtualisierung ist Folgendes erforderlich:

- Mindestens 4 GB verfügbarer Arbeitsspeicher (RAM) für den virtualisierten Hyper-V-Host
- Windows Server 2016 Technical Preview 4 oder Windows 10 Build 10565 auf dem Hostsystem und Windows Server Technical Preview 4 (Vollversion, Core) oder Nano Server auf dem virtuellen Computer.
- Ein Prozessor mit Intel VT-x (dieses Feature steht zurzeit nur für Intel-Prozessoren zur Verfügung)
- Die Containerhost-VM benötigt zudem mindestens 2 virtuelle Prozessoren.


## Unterstützte Betriebssystemimages

Windows Server Technical Preview 4 wird mit zwei Container-Betriebssystemimages, Windows Server Core und Nano Server angeboten. Nicht alle Konfigurationen unterstützen beide Betriebssystemimages. Diese Tabelle enthält Details zu den unterstützten Konfigurationen.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>Hostbetriebssystem</center></th>
<th><center>Windows Server-Container</center></th>
<th><center>Hyper-V-Container</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>Vollständige Benutzeroberfläche für Windows Server 2016</center></td>
<td><center>Image des Core-Betriebssystems</center></td>
<td><center>Image des Nano-Betriebssystems</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Core</center></td>
<td><center>Image des Core-Betriebssystems</center></td>
<td><center> Image des Nano-Betriebssystems</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Nano</center></td>
<td><center> Image des Nano-Betriebssystems</center></td>
<td><center>Image des Nano-Betriebssystems</center></td>
</tr>
</tbody>
</table>






<!--HONumber=Mar16_HO1-->


