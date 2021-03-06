---

copyright:
  years: 2018
lastupdated: "2018-05-25"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:pre: .pre}
{:tip: .tip}
{:table: .aria-labeledby="caption"}

# Einführung in das 'ibmcloud-secure-perimeter-health'-Image
{: #ibmcloud-secure-perimeter-health}

Das Image **ibmcloud-secure-perimeter-health** enthält ein Tool für das Scannen nach Sicherheitslücken in einem Secure Perimeter in IBM Cloud.
{:shortdesc}

## Funktionsweise
{: #how-it-works}

Damit sichergestellt wird, dass Secure Perimeter korrekt funktioniert, kann **ibmcloud-secure-perimeter-health** öffentliche oder private Netze in Ihrem IBM Cloud Infrastructure-Konto scannen und potenzielle Sicherheitslücken melden. Für die Verwendung des Images **ibmcloud-secure-perimeter-health** stehen zwei Möglichkeiten zur Verfügung:

-   Verwendung von **ibmcloud-secure-perimeter-health** als Pod in einem Kubernetes-Cluster in Secure Perimeter zum Scannen nach Sicherheitslücken in privaten Netzen.
-   Verwendung von **ibmcloud-secure-perimeter-health** als eigenständiger Docker-Container auf Ihrer Workstation zum Scannen nach Sicherheitslücken in öffentlichen Netzen.

Weitere Informationen zu Secure Perimeter finden Sie in den folgenden Blog-Artikeln:
  * [Secure Perimeter in IBM Cloud einrichten](https://developer.ibm.com/dwblog/2018/ibm-cloud-vyatta-set-up-secure-perimeter/)
  * [Automatisierten Secure Perimeter in IBM Cloud einrichten](https://developer.ibm.com/dwblog/2018/set-automated-secure-perimeter-ibm-cloud/)

Nach dem Scannen generiert das Image **ibmcloud-secure-perimeter-health** einen Bericht, der Informationen dazu enthält, welche Netze vom Secure Perimeter Segment aus erreichbar waren. Jeder Bericht enthält Details zum Namen des Netzgateways, zum VLAN, den zugehörigen Teilnetzen sowie gegebenenfalls zu Hosts, die Sicherheitsverstöße verursachen. Im Folgenden ist ein Beispielbericht eines Benutzers dargestellt, der nach Sicherheitslücken in einem privaten Netz gescannt hat:
```
#-------- Running Secure Perimeter exposure scan 2018-05-24 12:00:00 --------#

RESULTS:

sp-gateway-af6053a9:
	sps-priv-3deb5748:
		10.73.71.168/29:   PASS
	prv-neb1-cc4ee985:
		10.73.84.192/29:   FAIL
			host = 10.73.84.198, ports = [179, 22]
		10.73.63.8/29:     PASS
		10.73.72.128/26:   PASS
sp-gateway-8a9031ab:
	sps-priv-3deb5748:
		10.73.71.168/29:   PASS
	prv-neb1-cc4ee985:
		10.73.84.192/29:   PASS
		10.73.63.8/29:     PASS
```
{: screen}

## Enthaltene Elemente
{: #whats_included}

Das **ibmcloud-secure-perimeter-health**-Image bietet die folgenden Softwarepakete.
{:shortdesc}

-   Alpine Linux
-   Python-Laufzeit
-   SoftLayer-Python-Client
-   Nmap-Portscanner

## Einführung
{: #how_to_get_started}

Die folgenden Aufgaben beschreiben die Verwendung von **ibmcloud-secure-perimeter-health**:

1.  [Bereitstellen eines Kubernetes-Clusters in einem Secure Perimeter mit {{site.data.keyword.containerlong}}](#provision_cluster)
2.  [Scannen von privaten Netzen in einem Secure Perimeter](#private_networks)
3.  [Scannen von öffentlichen Netzen außerhalb eines Secure Perimeter](#public_networks)
4.  [Interpretieren von Scanergebnissen](#scan_results)
5.  [Referenzinformationen zu Containerargumenten](#reference_container_arg)
6.  [Referenzinformationen zu Umgebungsvariablen](#reference_env_var)


## Bereitstellen eines Kubernetes-Clusters in einem Secure Perimeter mit {{site.data.keyword.containerlong}}
{: #provision_cluster}

1.  Stellen Sie den Kubernetes-Custer im Abschnitt **Container** im IBM Cloud-Katalog bereit.
2.  Klicken Sie auf 'Erstellen'.
3.  Wählen Sie die öffentlichen und privaten Secure Perimeter Segment-VLANs in den VLAN-Dropdown-Menüs aus.
4.  Geben Sie alle übrigen Details nach Bedarf an.
5.  Klicken Sie auf 'Cluster erstellen'.

Lesen Sie die [{{site.data.keyword.containerlong}}](../../../containers/container_index.html#container_index)-Dokumentation für das Einrichten des Zugriffs auf Ihren Cluster nach dessen Bereitstellung.

## Scannen von privaten Netzen in einem Secure Perimeter
{: #private_networks}

Erstellen Sie einen Container-Pod, der auf dem Image **ibmcloud-secure-perimeter-health** basiert, und richten Sie einen Routinescan ein.

Führen Sie zunächst die folgenden Schritte aus:

-   Installieren Sie die erforderlichen [Befehlszeilenschnittstellen (CLIs)](../../../containers/cs_cli_install.html#cs_cli_install).
-   [Richten Sie Ihre Befehlszeilenschnittstelle](../../../containers/cs_cli_install.html#cs_cli_configure) auf Ihren Cluster aus.

1. Erstellen Sie eine Konfigurationsdatei mit dem Namen _health-pod.yaml_. Mit dieser Datei wird eine Hochverfügbarkeitsbereitstellung des Container-Pods erstellt.

    ```
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      name: health-pod
      labels:
        app: health-pod
    spec:
      replicas: 1
      selector:
        app: health-pod
      template:
        spec:
          containers:
          - name: health-pod
            image: registry.<region>.bluemix.net/ibm/ibmcloud-secure-perimeter-health:1.0.0
            args:
            - /usr/local/bin/python
            - /run.py
            - --scan
            - private
            - --exclude-vlan-ids
            - <Secure Perimeter Segment-VLAN-ID (privat)>
            - --poll-interval
            - 1800
            env:
            - name: SL_USER
              value: <IBM Cloud Infrastructure-Benutzername>
            - name: SL_APIKEY
              value: <IBM Cloud Infrastructure-API-Schlüssel>
    ```
    {: codeblock}

2. Erstellen Sie die Bereitstellung.

    ```
    kubectl apply -f health-pod.yaml
    ```
    {: pre}

3. Stellen Sie sicher, dass der Pod ausgeführt wird.

    ```
    kubectl get pods
    ```
    {: pre}

    ```
    NAME                                    READY     STATUS    RESTARTS   AGE
    health-pod-<random-id>                  1/1       Running   0          1hr
    ```
    {: screen}

## Scannen von öffentlichen Netzen außerhalb eines Secure Perimeter
{: #public_networks}

Erstellen Sie einen Docker-Container, der auf dem Image **ibmcloud-secure-perimeter-health** basiert, und scannen Sie öffentliche Netze.

Führen Sie zunächst die folgenden Schritte aus:

-  Docker installieren

1. Erstellen Sie wie folgt einen Docker-Container für die eigene Workstation:

    ```
    docker run -it -e SL_USER='$SL_USER' -e SL_APIKEY='$SL_APIKEY' registry.<region>.bluemix.net/ibm/ibmcloud-secure-perimeter-health:1.0.0 /usr/local/bin/python run.py --scan public --allowed-public-ports 80 443 9000-9999
    ```
    {: pre}

2. Nachdem der Container einen Bericht generiert hat, lesen Sie die Informationen im Abschnitt [Scanergebnisse analysieren](#scan_results), um die Ergebnisse zu interpretieren.

## Scanergebnisse analysieren
{: #scan_results}

**ibmcloud-secure-perimeter-health** generiert einen formatierten Bericht zum Secure Perimeter-Funktionsstatus:
```
#-------- Running Secure Perimeter exposure scan 2018-05-24 12:00:00 --------#

RESULTS:

sp-gateway-af6053a9:
	sps-priv-3deb5748:
		10.73.71.168/29:   PASS
	prv-neb1-cc4ee985:
		10.73.84.192/29:   FAIL
			host = 10.73.84.198, ports = [179, 22]
		10.73.63.8/29:     PASS
		10.73.72.128/26:   PASS
sp-gateway-8a9031ab:
	sps-priv-3deb5748:
		10.73.71.168/29:   PASS
	prv-neb1-cc4ee985:
		10.73.84.192/29:   PASS
		10.73.63.8/29:     PASS
```
{: screen}

Das Format des Berichts ist im Folgenden dargestellt:
```
<Gateway-Name>:
  <VLAN-Name>:
    <Teilnetz>:    PASS
    <Teilnetz>:    FAIL
      host = <IP-Adresse>, ports = [<zugänglicher Port>, <zugänglicher Port>, ...]
```
{: screen}

Das Image **ibmcloud-secure-perimeter-health** gibt für ein Teilnetz den Status `PASS` ('Bestanden') zurück, wenn kein Host im Teilnetz erreichbar war; andernfalls wird der Status `FAIL` ('Nicht bestanden') zurückgegeben und eine Liste der erreichbaren Host sowie der zugänglichen Ports wird ausgegeben.

## Referenzinformationen zu Containerargumenten
{: #reference_container_arg}

|Schlüssel|Beschreibung|Standardwert
|---|-------------|---|
|scan|Typ des Sicherheitslückenscans ("öffentlich" oder "privat") |Ohne (beide Scantypen)
|exclude-vlan-ids|Liste der VLANs nach IDs ohne Scannen|Ohne
|poll-interval|Sekunden bis zum nächsten Scan festlegen|0 (einmalige Ausführung)
|allowed-public-ports|Liste mit Ports, die in eine Whitelist für den Scan aufgenommen werden sollen|80, 443, 9000-9999
{: caption="Tabelle 1. Containerargumente" caption-side="top"}

## Referenzinformationen zu Umgebungsvariablen
{: #reference_env_var}

|Schlüssel|Beschreibung|
|---|-------------|
|SL_USER|Ihr Benutzername für IBM Cloud Infrastructure|
|SL_APIKEY|Ihr API-Schlüssel für IBM Cloud Infrastructure|
{: caption="Tabelle 2. Umgebungsvariablen" caption-side="top"}
