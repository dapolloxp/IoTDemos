# <span style="color:#0080FF">Creating an End-to-End IoT infrastructure in a Private Network</span>
The default configuration of Azure services allows public IP access to those services. For example an on-premises device with Internet access can connect to an Azure IoT Hub using a URL that resolves to a public IP address. For some scenarios or corporations this might be inappropriate, so we need to turn off public IP access to the relevant Azure services and configure all the services and communications to be on a private network. Though the process for how to do this is documented on a service-by-service basis for the various Azure services, how to connect them together is not entirely obvious.

 The goal of this document is provide a sample of securing all the services and connections in a simple IoT scenario. 

 Note: *This document does not include mouse-click-by-mouse-click instructions on how to deploy the various services. Rather it assumes a basic knowledge of Azure, and includes only what components need to be configured, and examples of the configuration.*

<span style="color:#FF0000">[TO DO:]</span>
- <span style="color:#FF0000">[Check: hide Azure subscription name and ID, and change/obfuscate all public IP addresses in the images included below.]</span>
- <span style="color:#FF0000">[Explain the reason we use IoT Routing to an Event Hub. Need John input.]</span>
- <span style="color:#FF0000">[Rewrite the DNS section. Need David input.]</span>
- <span style="color:#FF0000">[Edit text between images to improve flow.]</span>

 ## <span style="color:#0080FF">Contributors</span>
 - Spyros Sakellariadis, Program Manager, Industry Innovation, Enterprise Commercial Business
 - David Apolinar, Cloud Solution Architect, US Financial Services Industry
 - John Lian, Program Manager, Azure IoT Platform

## <span style="color:#0080FF">Major elements shown in end-to-end solution</span>
This sample does not include all possible services or configurations, of course, only a few in order to demonstrate the basic structures. The components that will be described include:

1. **VPN** - IPsec tunnel between on-premises systems and Azure
2. **Azure IoT Hub** and **Event Hub** - securing against public IP access
3. **Azure VM** - configuring a typical Azure asset with only private IP access
4. **DNS** - name resolution for assets with no public IP access

## <span style="color:#0080FF">High level architecture</span>
The IoT sample described consists of some on-premises components as well as Azure. To provde a visual reference for the items discussed, here are the high level architectures.

### <span style="color:#0080FF">On-premises configuration</span>
The following diagram shows the elements in the sample's local environment.

<img src="images/On-premises_config.jpg" width="600"/><p>

As shown in the diagram above, a 3rd party gateway is installed on a computer which serves to pull telemetry from IoT devices and forwards the data to Azure IoT Hub. Guidance on how to set up such a gateway, IoTWorX from ICONICS, is available in the following locations, and there is no need to replicate it here:

- [Using IoTWorX as a Gateway](https://iconics.com/Documents/WhitePapers/Using-IoTWorX-as-a-Gateway), and 
- [Installing IoTWorX on IoT Edge](https://iconics.com/Documents/Whitepapers/Installing-IoTWorX-on-IoT-Edge)

In addition, a hardware firewall is installed in the local environment, which serves as the local endpoint of the site-to-site VPN to Azure. Configuration of 
the firewall depends upon the make and model of the firewall, of which there are many. Some sample configurations can be found here:

- [Azure VPN Gateways VPN device configuration samples](https://github.com/Azure/Azure-vpn-config-samples)
- [Creating a Site-to-Site VPN from a Barracuda firewall to Azure](https://github.com/spyrossak/Azure-vpn-config-samples/blob/master/Barracuda/barracuda.md)

Finally, there is a DNS server in the local environment. Configuration of this server is described later in this document.

### <span style="color:#0080FF">Azure configuration</span>
The following diagram shows the elements in the sample's Azure environment. 

<img src="images/Azure_config.jpg" width="600"/><p>

Each of these elements is described in the following sections.

## <span style="color:#0080FF">Setting up the site-to-site VPN</span>
The following diagram shows the elements in the sample Azure environment needed to create an IPsec site-to-site Virtual Private Network. 

<img src="images/Site-to-site-VPN.jpg" width="450"/><p>


A 'How-to Guide' for creating a site-to-site VPN is published on the Microsoft website here: [Create a Site-to-Site connection in the Azure portal](https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal)

Configuration of these elements in the end-to-end sample is shown below. 


### <span style="color:#0080FF">Virtual network</span>
From the Azure portal, start the process of creating a virtual network by selecting **Create a Resource** > **Virtual network**. During setup, accept the proposed private IP address range, for example 10.2.0.0/16. When deployment is complete, select the resource. The result should look similar to the following, with the exception of the DNS server, which will be added later:

<img src="images/Virtual_Network.jpg" width="800"/><p>
### <span style="color:#0080FF">Virtual network gateway</span>
From the Azure portal, select **Create a Resource** > **Virtual network gateway**. Select the Virtual Network created earlier, and accept the proposed public IP address. The result should look similar to this:

<img src="images/Virtual_Network_Gateway.jpg" width="800"/><p>


### <span style="color:#0080FF">Local network gateway</span>
From the Azure portal, select **Create a Resource** > **Local network gateway**. This object represents the device on-premises that is the local endpoint for the IPsec tunnel. The IP address of the Local network gateway needs to be the public IP of that device, for example, the public IP address of the firewall or router on the WAN port provided by the ISP providing connectivity to the local site. (Here shown as 24.x.x.x). The Address space needs to be the address space of the local network behind that firewall or router. Configuration in the end-to-end sample is as follows:


<img src="images/Local_Network_Gateway.jpg" width="800"/><p>

### <span style="color:#0080FF">Connection</span>
From the Azure portal, select **Create a Resource** > **Connection**. The purpose is to create an object that represents the connection between the Virtual Network Gateway and the Local Network Gateway. Pick the local and Azure network gateways created above during setup of the connection. The IP addresses will be added automatically. Configuration in the end-to-end sample is as follows: 

<img src="images/Connection.jpg" width="800"/><p>

### <span style="color:#0080FF">Peering</span>
Finally, in the end-to-end sample we created a second vnet in another Azure region, in order to emulate more complex environments where all of the assets are not in the same region. Having done this, all we need to do is create a vnet peering. From the vnet resource page, configure the peering between the two vnets in the Peerings section:

<img src="images/ADLSvnetPeering.jpg" width="800"/><p>

## <span style="color:#0080FF">Deploying an IoT Hub with only private IP access</span>
The following diagram shows the elements in the end-to-end sample Azure environment. 

<img src="images/IoTHub.jpg" width="450"/><p>
A 'How-to Guide' for configuring IoT Hub in a vnet is published on the Microsoft website here: [IoT Hub support for virtual networks with Private Link and Managed Identity](https://docs.microsoft.com/en-us/azure/iot-hub/virtual-network-support)

Configuration of these elements in the end-to-end sample  is shown below. From the Azure portal, select **Create a Resource** > **IoT Hub**. When deployment is complete, select the resource. The Overview should be similar to the following:

### <span style="color:#0080FF">Overview</span>
<img src="images/IoTHubOverview.jpg" width="800"/><p>
### <span style="color:#0080FF">Public access</span>
The goal is to disable public IP access to the IoT Hub, which is done in in the Networking section. Disable Public Access:

<img src="images/IoTHubPublicAccess.jpg" width="800"/><p>

### <span style="color:#0080FF">Private endpoints</span>
Next, create private IP endpoints for this hub. Select the Private endpoint connections tab:

<img src="images/IoTHubPrivateEndpoints.jpg" width="800"/><p>


Select **+ Private endpoint** to create the private endpoint. The result should look similar to this:

<img src="images/IoTHubPrivateEndpoint.jpg" width="800"/><p>

Note you cannot enumerate IoT Devices or use Device Explorer to see telemetry incoming to IoT Hub because public IP addresses are blocked.

<span style="color:#FF0000">[John - please elaborate/rephrase this section]</span> In order to secure all access to the IoT Hub, it is best to route all messages from the IoT Hub to an independent Event Hub. First, create an Event Hub. From the Azure portal, select **Create a Resource** > **Event Hub**. When deployment is complete, configuration should be similar to this: 

### <span style="color:#0080FF">Overview</span>
<img src="images/EventHubNamespace.jpg" width="800"/><p>

### <span style="color:#0080FF">Event Hub Networking</span>

<img src="images/EventHubNetworking.jpg" width="800"/><p>

### <span style="color:#0080FF">Event Hub Privte Endpoints</span>

<img src="images/EventHubPrivateEndpoints.jpg" width="800"/><p>

### <span style="color:#0080FF">Event Hub Access Control</span>

<img src="images/EventHubAccessControl.jpg" width="800"/><p>

### <span style="color:#0080FF">Event Hub Access Control</span>

<span style="color:#FF0000">[Need explanation]</span>

<img src="images/EventHubsInstanceSharedAccessPolicies.jpg" width="1000"/><p>


### <span style="color:#0080FF">Message routing</span>
Having created an Event Hub with only private endpoints, now forward all telemetry coming in to the IoT Hub to that Event Hub, using the IoT Hub Message Routing Feature:

<img src="images/IoTHubMessageRouting.jpg" width="800"/><p>

### <span style="color:#0080FF">Message routing detail</span>
Routing details in the sample are set so as to forward everything to the Event Hub, with a consequence that no data can be retrieved from the IoT Hub itself by any application. To do this, set the routing details as follows:

<img src="images/IoTHubMessageRoutingDetail.jpg" width="800"/><p>

## <span style="color:#0080FF">Azure virtual machine</span>

Guidance for creating an Azure virtual machine is published on the Microsoft website here: [Quickstart: Create a Windows virtual machine in the Azure portal](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/quick-create-portal).

Configuration of the virtual machine in the end-to-end sample is shown below. From the Azure portal, select **Create a Resource** > **Windows Server 2019 Datacenter**. During setup enter selections so that it is deployed in the virtual network with only private IP access. When deployment is complete, the configuration should be similar to the following:


<img src="images/VMOverview.jpg" width="800"/><p>
<img src="images/VMIPConfig1.jpg" width="800"/><p>
<img src="images/VMIPconfigurations.jpg" width="800"/><p>
<img src="images/VMNetworking.jpg" width="800"/><p>

To verify that data arriving at the Event Hub is visible within the virtual machine, you can use Visual Studio code with the Event Hub explorer tool installed. After launching Visual Studio code and selecting the Event Hub above, right click and select Start Monitoring. You should see the data that was sent by the on-premises gateway to the Azure IoT Hub and forwarded on to the Event Hub:

<img src="images/EventHubTelemetryReceived.jpg" width="800"/><p>

## <span style="color:#0080FF">Deploying DNS servers</span>
DNS servers are needed to resolve URLs for services in Azure. When those services are initially deployed, they are accessed using a URL that resolves to their public IP address. For example

mydemovm.eastus.cloudapp.azure.com may resolve to 52.168.x.x

However, since we are preventing access to any public IP address and using only private endpoints, applications would have to resolve to the private IP address. For example

mydemovm.eastus.cloudapp.azure.com may need to resolve to 10.2.0.x

To do this, a DNS conditional forwarder is needed locally, to resolve requests from on-premises devices to Azure services, and in Azure, to resolve requests from one Azure service to another. 


### <span style="color:#0080FF">Azure</span>
The following diagram shows the elements in the sample Azure environment. 

<img src="images/DNS-Azure.jpg" width="450"/><p>
<img src="images/DNS-Azure-VMNetworking.jpg" width="450"/><p>
<img src="images/DNS-Azure-DNSManager.jpg" width="450"/><p>
<img src="images/DNS-Azure-DNSManagerCF1.jpg" width="450"/><p>
<img src="images/DNS-Azure-DNSManagerCF2" width="450"/><p>






