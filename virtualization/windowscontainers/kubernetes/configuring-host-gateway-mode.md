# <a name="host-gateway-mode"></a>Host-Gatewaymodus #
Eine der verfügbaren Optionen für Kubernetes-Networking ist der *Host-Gatewaymodus*, der die Konfiguration von statischen Routen zwischen Pod-Subnetzen auf allen Knoten beinhaltet.


## <a name="configuring-static-routes--linux"></a>Konfigurieren statischer Routen | Linux ##
Dazu verwenden wir `iptables`. Ersetzen Sie die Variable `$CLUSTER_PREFIX` durch das kompakte Subnetz (oder legen Sie diese damit fest), das alle Pods verwenden:

```bash
$CLUSTER_PREFIX="192.168"
sudo iptables -t nat -F
sudo iptables -t nat -A POSTROUTING ! -d $CLUSTER_PREFIX.0.0/16 \
              -m addrtype ! --dst-type LOCAL -j MASQUERADE
sudo sysctl -w net.ipv4.ip_forward=1
```

Dies richtet nur das grundlegende NATing für die Pods ein. Als nächstes muss der Datenverkehr für die Pods durch die primäre Schnittstelle geleitet werden. Ersetzen Sie erneut die Variable `$CLUSTER_PREFIX` je nach Bedarf sowie eventuell `eth0`:

```bash
sudo route add -net $CLUSTER_PREFIX.0.0 netmask 255.255.0.0 dev eth0
```

Abschließend müssen wir den nächsten Hop-Gateway **pro Knoten** hinzufügen. Wenn der erste Knoten auf einem Windows-Knoten beispielsweise `192.168.1.0/16` ist, dann:

```bash
sudo route add -net $CLUSTER.1.0 netmask 255.255.255.0 gw $CLUSTER.1.2 dev eth0
```

Muss eine ähnliche Route *für* jeden Knoten im Cluster *auf* jedem Knoten im Cluster hinzugefügt werden.


<a name="explanation-2-suffix"></a>
> [!Important]  
> **Nur** bei Windows-Knoten ist das Gateway die `.2` auf dem Subnetz. Bei Linux ist es ist wahrscheinlich immer `.1`. Diese Anomalie liegt daran, dass die Adresse `.1` als Gateway für das Bridging des Netzwerkadapters reserviert ist, der das Netzwerk des Hosts mit dem Netzwerk des virtuellen Pods verbindet.


## <a name="configuring-static-routes--windows"></a>Konfigurieren statischer Routen | Windows ##
Dazu verwenden wir `New-NetRoute`. Es steht Ihnen ein automatisiertes Skripts zur Verfügung, `AddRoutes.ps1`, in [diesem Repository](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/AddRoutes.ps1). Sie müssen die IP-Adresse des *Linux-Master* kennen sowie das Standardgateway des Knotens des *externen* Adapters in Windows (nicht den Pod-Gateway). Anschließend:


```powershell
$url = "https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/windows/AddRoutes.ps1"
wget $url -o AddRoutes.ps1
./AddRoutes.ps1 -MasterIp 10.1.2.3 -Gateway 10.1.3.1
```
