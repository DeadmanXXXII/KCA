# KCA
Bug hunt msrc 🤞 DB too.


KubecloudV2Azure.py
```pyhton
import os
from azure.identity import DefaultAzureCredential
from azure.mgmt.compute import ComputeManagementClient

# Azure subscription and resource details
SUBSCRIPTION_ID = "your_subscription_id"
RESOURCE_GROUP_NAME = "your_resource_group"
VM_NAME = "your_vm_name"
LOCATION = "your_location"

# Initialize Azure credentials and client
credential = DefaultAzureCredential()
compute_client = ComputeManagementClient(credential, SUBSCRIPTION_ID)

def create_virtual_machine():
    try:
        print(f"Creating virtual machine: {VM_NAME} in resource group: {RESOURCE_GROUP_NAME}...")
        vm_parameters = {
            "location": LOCATION,
            "os_profile": {
                "computer_name": VM_NAME,
                "admin_username": "your_admin_username",
                "admin_password": "your_admin_password",
            },
            "hardware_profile": {
                "vm_size": "Standard_DS1_v2"
            },
            "storage_profile": {
                "image_reference": {
                    "publisher": "Canonical",
                    "offer": "UbuntuServer",
                    "sku": "18.04-LTS",
                    "version": "latest"
                },
                "os_disk": {
                    "name": f"{VM_NAME}_os_disk",
                    "caching": "ReadWrite",
                    "create_option": "FromImage"
                }
            },
            "network_profile": {
                "network_interfaces": [{
                    "id": f"/subscriptions/{SUBSCRIPTION_ID}/resourceGroups/{RESOURCE_GROUP_NAME}/providers/Microsoft.Network/networkInterfaces/{VM_NAME}-nic"
                }]
            }
        }
        compute_client.virtual_machines.begin_create_or_update(RESOURCE_GROUP_NAME, VM_NAME, vm_parameters)
        print(f"Virtual machine {VM_NAME} created successfully.")

    except Exception as e:
        print(f"Error creating virtual machine: {e}")

if __name__ == "__main__":
    create_virtual_machine()
```

