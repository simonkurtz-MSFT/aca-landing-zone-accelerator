# Deploy the spoke network

In the prior step, you deployed the [regional hub](../01-hub/README.md), now you'll lay out the spoke that will contain the Azure Container Apps instance and related resources.

## Networking in this architecture

The regional spoke network in which your application platform is laid into acts as the first line of defense for your workload. This network perimeter forms a security boundary where you will restrict the network line of sight into your resources. It also gives your application platform the ability to use private link to talk to adjacent platform-as-a-service resources such as Key Vault and Azure Container Registry. And finally it acts as a layer to restrict and tunnel egressing traffic. All of this adds up to ensure that workload traffic stays as isolated as possible and free from any possible external influence, including other enterprise workloads.

## Expected results

After executing these steps you'll have the spoke resource group (`rg-lzaaca-spoke-dev-reg`, by default) populated with a virtual network, subnets, and peering to the regional hub. Based on how you [configured the naming and deployment parameters](../../README.md#steps), your result may be slightly different.

![A picture of the networking components in the spoke resource group.](./media/spoke.png)

### Resources

- Spoke resource group
- Spoke virtual network
- Peering to and from the hub
- Jump box virtual machine (optional)

## Steps

:warning: If you are running these commands on windows, you might need to run the command below to ensure windows captures the RESOURCEID_VNET_HUB correctly:

```bash
export MSYS_NO_PATHCONV=1
```

1. Set the desired region for the whole reference implementation.

   :stop_sign: Update this to your desired region.

   ```bash
   LOCATION=eastus # or any location that suits your needs, ensure Azure OpenAI service is available there.
   ```

:warning: You will need to get the IP address of your Azure firewall or whatever network appliance you are using and replace the `[IP OF THE NETWORK APPLIANCE] placeholder in the deploy.spoke.paramters.jsonc file with it.

1. Get the private IP address of your Azure firewall

   ```bash
   NETWORK_APPLIANCE_IP_ADDRESS=$(az deployment sub show -n  acalza01-hub --query properties.outputs.networkApplianceIpAddress.value -o tsv)
   echo NETWORK_APPLIANCE_IP_ADDRESS: $NETWORK_APPLIANCE_IP_ADDRESS
   ```

1. Create the regional spoke network.

   ```bash
   RESOURCEID_VNET_HUB=$(az deployment sub show -n acalza01-hub --query properties.outputs.hubVNetId.value -o tsv)
   echo RESOURCEID_VNET_HUB: $RESOURCEID_VNET_HUB
   ```

   ```bash
   # [This takes about two minutes to run.]
   az deployment sub create \
      -n acalza01-spokenetwork \
      -l $LOCATION \
      -f 02-spoke/deploy.spoke.bicep -p 02-spoke/deploy.spoke.parameters.jsonc \
      -p hubVNetId=${RESOURCEID_VNET_HUB} \
      -p networkApplianceIpAddress=${NETWORK_APPLIANCE_IP_ADDRESS}
   ```

1. Explore your networking resources. *Optional.*

   You may wish to take this moment to familiarize yourself with the resources that have been deployed so far to Azure. They have all been networking resources, establishing the network and access boundaries from within which your application platform will be executing. Check out the following resource groups in the [Azure portal](https://portal.azure.com).

   ```bash
   RESOURCENAME_RESOURCEGROUP_HUB=$(az deployment sub show -n acalza01-hub --query properties.outputs.resourceGroupName.value -o tsv)
   RESOURCENAME_RESOURCEGROUP_SPOKE=$(az deployment sub show -n acalza01-spokenetwork --query properties.outputs.spokeResourceGroupName.value -o tsv)

   echo Hub Resource Group: $RESOURCENAME_RESOURCEGROUP_HUB && \
   echo Spoke Resource Group: $RESOURCENAME_RESOURCEGROUP_SPOKE
   ```

## Azure landing zone platform alignment

The creation of the hub resources, spoke virtual network, and routing configuration are usually the responsibility of the connectivity platform team. While the creation of subnets, NSGs, and the workload resources are delegated to the workload team. The deployment steps so far have been a mix of both roles. Be sure to understand your organization's separation of duties in landing zone deployments, and use your organization's subscription vending solution to . From this point on in the walkthrough, the steps are indeed all responsibilities of the workload team.

## Next step

:arrow_forward: [Deploy long-lifecycle resources](../03-supporting-services/README.md)
