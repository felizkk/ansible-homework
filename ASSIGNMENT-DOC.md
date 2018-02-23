Red Hat Delivery Specialist - Advanced Ansible
Homework Assignment Documentation

A. Tower Servers
Tower servers are deployed as HA (tower1,tower2, tower3) with postgresql
database replicated between support1 and support2 servers (Streaming)

B. QA Environment

B.1. Playbooks
- Provision_OSP.yml - OpenStack Deployment and rhel server instances creation
- Configure_3TA-OSP.yml - Deploy frontends, apps, and appdbs on OSP instances
- cleanup_OSP.yml - Destroy server instances

B.2. QA Inventory:
The frontend1, app1, app2, and appdb1 hosts are behind Openstack (private net)
and will be treated as In-Memory Inventory gathered through facts

B.3. Openstack Jumpbox:
For Tower to connect to OpenStack, I am using Jumpbox (workstation)
This is achieved by SSH proxy configured through ansible.cfg and ssh.cfg
on Ansible Project. The private key will be deployed to each tower via separate
playbook.

C. Production Environment

C.1. Playbooks:
- Provision_AWS.yml - AWS 3Tier Service order through script
- Configure_3TA-AWS.yml - Deploy frontends, apps, and appdbs on EC2 instances
- cleanup_AWS.yml - Remove installed/configured packages from EC2 instances

C.2. Production Inventory:
I use built-in ec2.py script via Tower's Web GUI inside the Inventory-Sources
menu. The AWS credential has been supplied.
I am using some static groups: frontends, apps, appdbs on the same Tower
Inventory and these 3 groups will become parrent of below AWS tag groups:
- tag_AnsibleGroup_appdbs
- tag_AnsibleGroup_apps
- tag_AnsibleGroup_frontends
The 3tier-* roles are used by both QA and Prod by calling frontends, apps,
and appdbs groups

D. Challenges and Notes

1. OSP Instances will auto-shutdown do to the OpenTLC Labs runtime settings.
Therefore, the OSP Provisioning tasks will have to ensure that instances are
always started with os_server_action module with action: started

2. I am doing Smoketests 3 times using ansible uri module to ensure that I
can observe the load-balancing behavior through different hosts displayed
on the Web site. This method is the same for both QA and Prod

3. The supplied order_svc.sh script calls jq command, which is not located on
the same inventory. Rather than putting all AWS scripts under 1 directory, I
expanded the ansible host PATH environment to include /home/cloud-user/bin so
jq can be called properly

4. I am not using static inventory file located on awx project directory.
All my Inventories are defined in Tower's database created via GUI. I only have
1 static host, which is the jumpbox/workstation host

5. I am configuring AWS instances directly from tower via internet. To do This
I need the SSH private key from the bastion host. I found the GUID from the
CloudForms login. I am then pasting the GUIDkey.pem contents to the Tower's
machine credential for ec2-user

6. For AWS Dynamic Inventory (via Sources), I can't use Instance Filter with
tag:instance_filter=three-tier-app-<my ID> as it didn't find anything.
I couldn't use Owner=*<my_name>* as well as it will pick up all VM hosts
including OSP instances which some of which are also on frontends, apps, appdbs.
I ended up using tag:Project=*<GUID>* to pick up only the AWS EC2 VM hosts.
I believe in real life, we will know what name we are going to use as Filter

7. Due to unfamiliarity with Amazon Tower API, and due to the fact if The
OpenTLC CloudForms service is destroyed I have to wait for another 20 minutes
plus finding new GUID and writing new private key (GUIDkey.pem) on Tower, for
this homework destroying AWS instance will be limited to resetting/removing the
RHEL packages + configs. So it is different with the QA Openstack where the
whole server instances are deleted and rebuilt later on.
Besides, this is Production! Nobody is supposed to delete production Servers

8. The HAProxy template file has been edited to support dynamic inventories of
both OSP and AWS
