# Anforderungen von Windows-Containern

**Dieser Inhalt ist vorläufig und kann geändert werden.**

In diesen Handbüchern sind die Anforderungen für einen Windows-Containerhost aufgeführt.

## Unterstützte Betriebssystemimages

Windows Server Technical Preview 4 wird mit zwei Container-Betriebssystemimages, Windows Server Core und Nano Server angeboten. Nicht alle Konfigurationen unterstützen beide Betriebssystemimages. Diese Tabelle enthält Details zu den unterstützten Konfigurationen.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<tr valign="top">
<td><center>**Hostbetriebssystem**</center></td>
<td><center>**Windows Server-Container**</center></td>
<td><center>**Hyper-V-Container**</center></td>
</tr>
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
</table>

## Hyper-V-Containeranforderungen

Wenn ein Windows-Containerhost von einem virtuellen Hyper-V-Computer ausgeführt wird und auch Hyper-V-Container hostet, muss die geschachtelte Virtualisierung aktiviert werden. Für die geschachtelte Virtualisierung ist Folgendes erforderlich:

- Mindestens 4 GB verfügbarer Arbeitsspeicher (RAM) für den virtualisierten Hyper-V-Host
- Windows Server 2016 Technical Preview 4 oder Windows 10 Build 10565 auf dem physischen und dem virtualisierten Host
- Ein Prozessor mit Intel VT-x (dieses Feature steht zurzeit nur für Intel-Prozessoren zur Verfügung)
- Die Containerhost-VM benötigt zudem mindestens 2 virtuelle Prozessoren.





