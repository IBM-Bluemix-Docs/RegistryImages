---

copyright:
  years: 2018, 2019
lastupdated: "2019-02-21"

keywords: IBM Cloud Container Registry, ibmcloud-secure-perimeter-health, container image, health, Secure Perimeter, scan, public image

subcollection: RegistryImages

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:deprecated: .deprecated}
{:table: .aria-labeledby="caption"}

# Iniciación a la imagen `ibmcloud-secure-perimeter-health`
{: #ibmcloud-secure-perimeter-health}

La imagen `ibmcloud-secure-perimeter-health` contiene una herramienta para escanear vulnerabilidades en un Secure Perimeter en {{site.data.keyword.cloud}}.
{:shortdesc}

Puede acceder a las imágenes que suministra {{site.data.keyword.IBM_notm}} desde la línea de mandatos; consulte las [imágenes públicas de IBM](/docs/services/Registry?topic=registry-public_images#public_images).
{: tip}

## Modo de funcionamiento
{: #sph_how-it-works}

Para asegurarse de que su Secure Perimeter esté funcionando correctamente, `ibmcloud-secure-perimeter-health` puede escanear redes públicas o privadas en su cuenta de infraestructura de {{site.data.keyword.cloud_notm}} e informar de las vulnerabilidades. Puede utilizar la imagen **ibmcloud-secure-perimeter-health** de dos formas:

- Utilice `ibmcloud-secure-perimeter-health` como un pod en un clúster de Kubernetes dentro del Secure Perimeter para escanear exposiciones de red privada.
- Utilice `ibmcloud-secure-perimeter-health` como un contenedor autónomo de Docker de su estación de trabajo para escanear exposiciones de red pública.

Se puede encontrar más información sobre Secure Perimeter en estos artículos del blog:

- [Configure un perímetro seguro en {{site.data.keyword.cloud_notm}} ![Icono de enlace externo](../../../icons/launch-glyph.svg "Icono de enlace externo")](https://developer.ibm.com/dwblog/2018/ibm-cloud-vyatta-set-up-secure-perimeter/).
- [Configure un perímetro seguro automatizado en {{site.data.keyword.cloud_notm}} ![Icono de enlace externo](../../../icons/launch-glyph.svg "Icono de enlace externo")](https://developer.ibm.com/dwblog/2018/set-automated-secure-perimeter-ibm-cloud/).

Después del escaneado, la imagen `ibmcloud-secure-perimeter-health` crea un informe sobre a qué redes se ha llegado desde dentro del Secure Perimeter Segment. Cada informe detalla el nombre de la pasarela de red, VLAN, sus subredes, y cualquier host erróneo. Un informe de ejemplo de un usuario que ha escaneado vulnerabilidades de red privada:

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

## Qué incluye
{: #sph_whats_included}

La imagen `ibmcloud-secure-perimeter-health` proporciona los siguientes paquetes de software.
{:shortdesc}

- Alpine Linux
- Tiempo de ejecución de Python
- SoftLayer Python Client
- Escáner de puerto de nmap

## Suministro de un clúster de Kubernetes dentro de un perímetro seguro mediante {{site.data.keyword.containerlong_notm}}
{: #sph_provision_cluster}

1. Suministre el clúster de Kubernetes desde la sección **Contenedores** del catálogo de the {{site.data.keyword.cloud_notm}}.
2. Pulse **Crear**.
3. Seleccione las VLAN públicas y privadas de Secure Perimeter Segment desde los menús desplegables de VLAN.
4. Especifique los demás detalles que necesite.
5. Pulse **Crear clúster**.

Revise la documentación de [{{site.data.keyword.containerlong_notm}}](/docs/containers?topic=containers-container_index#container_index) sobre cómo obtener acceso al clúster una vez que se haya desplegado.

## Escanee las redes privadas dentro de un Secure Perimeter
{: #sph_private_networks}

Cree un pod del contenedor a partir de la imagen `ibmcloud-secure-perimeter-health`, y configure un escaneo rutinario.

**Antes de empezar**

- Instale las [CLI](/docs/containers?topic=containers-cs_cli_install#cs_cli_install) necesarias.
- [Defina su clúster](/docs/containers?topic=containers-cs_cli_install#cs_cli_configure) como destino de la CLI.

1. Cree un archivo de configuración con el nombre _health-pod.yaml_. Este archivo crea un despliegue altamente disponible del pod de contenedor.

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
            image: registry.bluemix.net/ibm/ibmcloud-secure-perimeter-health:1.0.0
            args:
            - /usr/local/bin/python
            - /run.py
            - --scan
            - private
            - --exclude-vlan-ids
            - <Private Secure Perimeter Segment VLAN ID>
            - --poll-interval
            - 1800
            env:
            - name: SL_USER
              value: <IBM Cloud infrastructure user name>
            - name: SL_APIKEY
              value: <IBM Cloud infrastructure api key>
    ```
    {: codeblock}
    
2. Cree el despliegue.

    ```
    kubectl apply -f health-pod.yaml
    ```
    {: pre}

3. Confirme que se esté ejecutando el pod.

    ```
    kubectl get pods
    ```
    {: pre}

    ```
    NAME                                    READY     STATUS    RESTARTS   AGE
    health-pod-<random-id>                  1/1       Running   0          1hr
    ```
    {: screen}

## Escanee redes públicas fuera de un Secure Perimeter
{: #sph_public_networks}

Cree un contenedor de Docker desde la imagen `ibmcloud-secure-perimeter-health` y escanee redes públicas.

**Antes de empezar**

- Instale Docker.

1. Cree un contenedor de Docker desde su propia estación de trabajo de la forma siguiente:

    ```
    docker run -it -e SL_USER='$SL_USER' -e SL_APIKEY='$SL_APIKEY' registry.bluemix.net/ibm/ibmcloud-secure-perimeter-health:1.0.0 /usr/local/bin/python run.py --scan public --allowed-public-ports 80 443 9000-9999
    ```
    {: pre}

2. Una vez que el contenedor cree un informe, revise la sección [Análisis de resultados del escaneo](#sph_scan_results) para comprender los resultados.

## Análisis de los resultados del escaneo
{: #sph_scan_results}

`ibmcloud-secure-perimeter-health` crea un informe formateado en estado de funcionamiento de un Secure Perimeter:

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

El formato del informe es el siguiente:

```
<gateway name>:
  <vlan name>:
    <subnet>:    PASS
    <subnet>:    FAIL
      host = <ip address>, ports = [<exposed port>, <exposed port>, ...]
```
{: screen}

`ibmcloud-secure-perimeter-health` determina una subred como `PASS` si no se han encontrado hosts en la subred; de lo contrario, devolverá `FAIL` y listará los hosts a los que se ha llegado, junto con los puertos que estaban accesibles.

## Referencia de argumentos del contenedor
{: #sph_reference_container_arg}

|Clave|Descripción|Valor predeterminado
|---|-------------|---|
|scan|El tipo de escaneo de exposición ("public" o "private") |Ninguno (escanear ambos)
|exclude-vlan-ids|Lista de VLAN por ID para evitar el escaneado|Ninguno
|poll-interval|Establecer el número de segundos hasta el siguiente escaneo|0 (ejecutar una vez)
|allowed-public-ports|Lista de puertos a incluir en una lista blanca en el escaneado|80, 443, 9000-9999
{: caption="Tabla 1. Argumentos del contenedor" caption-side="top"}

## Referencia de variables de entorno
{: #sph_reference_env_var}

|Clave|Descripción|
|---|-------------|
|SL_USER|El nombre de usuario de la infraestructura de IBM Cloud|
|SL_APIKEY|La clave de API de la infraestructura de IBM Cloud|
{: caption="Tabla 2. Variables de entorno" caption-side="top"}
