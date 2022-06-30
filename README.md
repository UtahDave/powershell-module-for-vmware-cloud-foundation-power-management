# Shutting Down and Starting Up VMware Cloud Foundation by Using Windows PowerShell
# Disclaimer  
THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE
WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS
OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR
OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

# Supported Platforms
- Tested on up-to-date versions of Windows 10, Windows 2016, Windows 2019 and Windows 2022, each running Windows PowerShell Desktop 7.2.4  
- Supported target systems: vSAN ReadyNodes running VMware Cloud Foundation 4.4.x or VMware Cloud Foundation 4.3.x.  

# Limitations
- VMware Cloud Foundation on VxRail **IS NOT** supported.
- Site Recovery Manager, vSphere Replication, vRealize Suite, Workspace ONE Access and vSphere with Tanzu are not supported.
- You must shut down the ESXi hosts manually. The scripts only place the hosts in maintenance mode.
- The sample scripts work only on a workload domain with a single cluster.
- You must stop NSX Edge bare-metal nodes manually.
- If two or more VI workload domains use the same NSX Manager, you must shut down the customer workload VMs before running the script because NSX-T Data Center will be shut down with the first VI workload domain.
- To be able to shut down the customer VMs in the management domain or in a VI workload domain by using a script, 
they must have VMware Tools running. The virtual machines are shut down
in a random order by running the "Shutdown guest OS" command from vCenter Server.
- The SSH service on the ESXi hosts must be running.
- Scripts cannot handle simultaneous connections to multiple services. In the script's console, all sessions to services that are not used at the moment will be disconnected.
# Scripts for Shutdown and Startup of a Workload Domain
- **PowerManagement-ManagementDomain.ps1** - Shut down or start up all software components in the management
domain. The script does not support shutdown of vRealize Suite. Shut down the vRealize Suite components in your environment manually before running the script.

- **PowerManagement-WorkloadDomain.ps1** - Shut down or start up the management components for a VI
workload domain. The script does not support vSphere with Tanzu.

# Prerequisites
- Add forward and reverse DNS records for the client machine running the scripts must be added to the DNS server.
- Run the scripts on PowerShell Desktop 7.2.x or later.
- Run PowerShell as Administrator and install the VMware.PowerManagement PowerShell
module together with module dependencies from the PowerShell Gallery by running the
following commands:  
    > `Install-Module -Name VMware.PowerCLI -MinimumVersion 12.6.0`  
    > `Install-Module -Name PowerVCF -MinimumVersion 2.2.0`  
    > `Install-Module -Name Posh-SSH -MinimumVersion 3.0.4`  
    > `Install-Module -Name VMware.PowerManagement -MinimumVersion 1.0.0`  
- Verify that SDDC Manager is running.
- Before you shut down the management domain, get the credentials for the management
domain hosts and vCenter Server from SDDC Manager and save them for troubleshooting or a
subsequent manual startup. Because SDDC Manager is down during each of these operations,
you must save the credentials in advance.
To get the credentials, log in to the SDDC Manager appliance by using a Secure Shell (SSH)
client as **vcf** and run the `lookup_passwords` command.
- On Windows 10, configure the PowerShell execution policy with the permissions required to
run the commands.  
a) Run the `Execute Get-ExecutionPolicy` command to get the active execution policy.  
b) If the `Execute Get-ExecutionPolicy` command returns `Restricted`, run the
`Set-ExecutionPolicy RemoteSigned` command.
- If the target system uses self-signed or untrusted certificates, configure PowerCLI to ignore them.
# How to use the sample scripts
1. Enable SSH on the ESXi hosts in the workload domain by using the SoS utility of the SDDC
Manager appliance.
    - Log in to the SDDC Manager appliance by using a Secure Shell (SSH) client as vcf.
    - Switch to the root user by running the su command and entering the root password.
    - Run this command:
         > `/opt/vmware/sddc-support/sos --enable-ssh-esxi --domain domain-name`
2. On the Windows machine that is allocated to run the scripts, start Windows PowerShell 7.x.
3. Locate the home directory of the VMware.PowerManagement module by running this
PowerShell command.
    > `(Get-Module -ListAvailable VMware.PowerManagement*).path`  

> For example, the full path to the module might be:
    `C:\Program Files\WindowsPowerShell\Modules\VMware.PowerManagement\1.0.0.1000\VMware.PowerManagement.psd1`.  
4. Go to the `SampleScripts` folder that is located in the same folder as the
`VMware.PowerManagement.psd1` file.
5. To shut down or start up a VI workload domain, perform these steps.  
    - Replace the values in the sample code with values from your environment and run the
commands in the PowerShell console.  
    > `$sddcManagerFqdn = "sfo-vcf01.sfo.rainpole.io"`  
    > `$sddcManagerUser = "administrator@vsphere.local"`  
    > `$sddcManagerPass = "VMw@re1!"`  
    > `$sddcDomain = "sfo-w01"`  
    - Run the PowerManagement-WorkloadDomain.ps1 script.  
    > `PowerManagement-WorkloadDomain.ps1 -server $sddcManagerFqdn -user $sddcManagerUser -pass $sddcManagerPass -sddcDomain $sddcDomain -shutdown -shutdownCustomerVm`  
    When you use the `-shutdownCustomerVm` parameter, the customer virtual machines are shut down as the first step of the shutdown process.

6. To shut down or start up the management domain, perform these steps.
    - Replace the values in the sample code with values from your environment and run the
commands in the PowerShell console.
    > `$sddcManagerFqdn = "sfo-vcf01.sfo.rainpole.io"`  
    > `$sddcManagerUser = "administrator@vsphere.local"`  
    > `$sddcManagerPass = "VMw@re1!"`  
    - Run the `PowerManagement-ManagementDomain.ps1` script.  
    > `PowerManagement-ManagementDomain.ps1 -server $sddcManagerFqdn -user $sddcManagerUser -pass $sddcManagerPass -shutdown`  
    > `PowerManagement-ManagementDomain.ps1 -startup`  
    During the shutdown operation, the script generates a `ManagementStartupInput.json` file
in the current directory. The script then uses the file for the subsequent startup of the
management domain.  
    Because the script overwrites the JSON file every time you run it, if you are shutting down
multiple management domains, rename the file before shutting down the next domain.
___
### *Caution* Although the credentials in the JSON file are encrypted, treat the content of this file as sensitive data.
___

___
**Note** More usage examples are available in the scripts.
___
# Troubleshooting
- ESXi hosts must have SSH service up, running and accessible from the machine running the scripts.
- In case of a failure, run the script with the same parameters in order to overcome some errors.
- Identify the step that is causing the issue and continue the sequence following the manual guide in VMware Cloud Foundation documentation.
- During shutdown of the management domain, if SDDC Manager is already stopped, the only option is to continue by following the manual steps in the VMware Cloud Foundation documentation.

# Feedback
Please share with us your experience with using this Power Shell module. 
Use the "Send Feedback" form on the right in the <a href="https://docs.vmware.com/en/VMware-Cloud-Foundation/4.4/vcf-operations/GUID-65F5FE47-5831-4C72-B0DB-9D0C537446E2.html" target="_blank">Shutdown and Startup of VMware Cloud Foundation</a> documentation.