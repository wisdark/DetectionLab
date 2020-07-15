# Building DetectionLab on ESXi
![Overview](https://github.com/clong/DetectionLab/blob/master/img/esxi_overview.jpeg?raw=true)

NOTE: This is an early release and it's possible that certain features may not work perfectly for everyone yet

## Prereqs (~30-60 minutes)
0. Have an ESXi instance version 6 or higher. VSphere is **NOT** required.
1. Install the [requirements from the ESXi Terraform Provider](https://github.com/josenk/terraform-provider-esxi#requirements)
    * If building on MacOS, don't forget to change the GOOS from linux to darwin!
        * `GOOS=linux` -> `GOOS=darwin`
2. Build and install the [terraform-provider-esxi](https://github.com/josenk/terraform-provider-esxi#building-the-provider) provider
3. Your ESXi must have at least two separate networks - one that is accessible from your current machine (VM Network) and a HostOnly network to allow the VMs to have internet access (HostOnly). 
4. [OVFTool](https://my.vmware.com/web/vmware/details?downloadGroup=OVFTOOL420&productId=618) must be installed and in your path. 
    * On MacOS, I solved this by creating a symbolic link to the ovftool included in VMWare Fusion: `sudo ln -s "/Applications/VMware Fusion.app/Contents/Library/VMware OVF Tool/ovftool" "/usr/local/bin/ovftool"`
5. On your ESXI, you must:
   1. Enable SSH
    2. Enable the "Guest IP Hack" 
    3. Open VNC ports on the firewall
    * Instructions for those steps are here: https://nickcharlton.net/posts/using-packer-esxi-6.html
6. [Install Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

## Steps

1. **(5 Minutes)** Edit the variables in `DetectionLab/ESXi/Packer/variables.json` to match your ESXi configuration. The `esxi_network_with_dhcp_and_internet` variable refers to any ESXi network that will be able to provide DHCP and internet access to the VM while it's being built in Packer.

2. **(45 Minutes)** From the `DetectionLab/ESXi/Packer` directory, run:
* `PACKER_CACHE_DIR=../../Packer/packer_cache packer build -var-file variables.json windows_10_esxi.json`
* `PACKER_CACHE_DIR=../../Packer/packer_cache packer build -var-file variables.json windows_2016_esxi.json`
* `PACKER_CACHE_DIR=../../Packer/packer_cache packer build -var-file variables.json ubuntu1804_esxi.json`

These commands can be run in parallel from three separate terminal sessions.

![Packer](https://github.com/clong/DetectionLab/blob/master/img/esxi_packer.png?raw=true)

3. **(1 Minute)** Once the Packer builds finish, verify that you now see Windows10, WindowsServer2016, and Ubuntu1804 in your ESXi console
![Ansible](https://github.com/clong/DetectionLab/blob/master/img/esxi_console.png?raw=true)
4. **(5 Minutes)** Edit the variables in `ESXi/variables.tf` to match your local ESXi configuration or [create a terraform.tfvars file](https://www.terraform.io/docs/configuration/variables.html#variable-definitions-tfvars-files) (RECOMMENDED) to override them.
5. **(25 Minutes)** From `DetectionLab/ESXi`, run `terraform init && terraform apply`
6. Once Terraform has finished bringing the hosts online, change your directory to `DetectionLab/ESXi/Ansible`
7. **(1 Minute)** Edit `DetectionLab/ESXi/Ansible/inventory.yml` and replace the IP Addresses with the respective IP Addresses of your ESXi VMs. **These IP addresses much be reachable from your host machine!**
8. **(3 Minute)s** Edit `DetectionLab/ESXi/resources/01-netcfg.yaml`. These are the IP addresses that will be applied to the logger network interfaces. These should be be able to be found in your ESXi console or from the Terraform outputs.
9. **(3 Minute)** Before running any Ansible playbooks, I highly recommend taking snapshots of all your VMs! If anything goes wrong with provisioning, you can simply restore the snapshot and easily debug the issue.
10. Change your directory to `DetectionLab/ESXi/Ansible`
11. **(30 Minutes)** Run `ansible-playbook -vvv detectionlab.yml` 
12. If all goes well, you should see the following and your lab is complete!
![Ansible](https://github.com/clong/DetectionLab/blob/master/img/esxi_ansible.png?raw=true)

If you run into any issues along the way, please open an issue on Github and I'll do my best to find a solution.

## Future work required
* It probably makes sense to abstract all of the logic in `bootstrap.sh` into individual Ansible tasks
* There's a lot of areas to make reliability improvements
* I'm guessing there's a way to parallelize some of this execution: https://medium.com/developer-space/parallel-playbook-execution-in-ansible-30799ccda4e0

## Debugging / Troubleshooting
* If an Ansible playbook fails, you can pick up where it left off with `ansible-playbook -vvv detectionlab.yml --start-at-task="taskname"`

## Credits
As usual, this work is based off the heavy lifting that others have done. My primary sources for this work were:
* [Josenk's Terraform-ESXI-Provider](https://github.com/josenk/terraform-provider-esxi) - Without this, there would be no way to deploy DL to ESXi without paying for VSphere. Send him/her some love 💌
* [Automate Windows VM Creation and Configuration in vSphere Using Packer, Terraform and Ansible - Dmitry Teslya](https://dteslya.engineer/automation/2019-02-19-configuring_vms_with_ansible/#setting-up-ansible)
* [Building Virtual Machines with Packer on ESXi 6 - Nick Charlton](https://nickcharlton.net/posts/using-packer-esxi-6.html) 
* [The DetectionLab work that juju4 has been doing on Azure and Ansible](https://github.com/juju4/DetectionLab/tree/devel-azureansible/Ansible)
* [lofi hip hop radio - beats to relax/study to](https://www.youtube.com/watch?v=5qap5aO4i9A) 🔉

Thank you to all of the sponsors who made this possible!
