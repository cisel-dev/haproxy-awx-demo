<a href="https://www.cisel.ch/"><img src="https://www.cisel.ch/wp-content/uploads/2019/05/cisel_80.png" title="CISEL" alt="CISEL"></a>

<!-- [![CISEL](https://www.cisel.ch/)](https://www.cisel.ch/) -->

In this article we will show you what we have set up to automatically update and test the configurations files on 2 failever HAproxy servers.

Both haproxy servers are configured in failover with the keepalived service

We will have a Git repo as the only source of truth, then AWX which will be used to validate the configuration files and apply them on the HAproxy servers.

This setup is in order to avoid uncontrolled changes on your HAproxy config and allow easy rollbacks.


Below an image that summarizes the environment and the flows involved

![2020-12-18 09_57_39-Automate HAproxy with ansible and AWX.drawio - diagrams.net.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1608281880970/1L2HRvTkB.png)

This public repository contains example yaml and config files that you can use to create your own process:  [https://github.com/cisel-dev/haproxy-awx-demo.git](Link) 


1. git_haproxy.cfg : 
This file is the reason why we have created this automation. It contains the configuration we want to apply on HAproxy servers. Below you can find a very simple example of an haproxy.cfg file. Please adapt it according to your needs
[https://github.com/cisel-dev/haproxy-awx-demo/blob/main/haproxy.cfg](Link) 
 
2. git_keepalived.conf : 
This is the keepalived configuration file. If you want to add a listener or frontend in haproxy with a new virtual IP, you will need to declare this IP in keepalived.conf before to be able to use it in haproxy. Here also you will find an example file that you will need to adapt.
[https://github.com/cisel-dev/haproxy-awx-demo/blob/main/keepalived_apply.yaml](Link) 

3. clone_git_haproxy_project.yaml : This playbook is used to clone files from the git repo to haproxy servers. It will also do a first validation of the git_haproxy.cfg file to avoid performing the following steps if the file is not valid. You need to change the project_dir setting to match your setup.
[https://github.com/cisel-dev/haproxy-awx-demo/blob/main/clone_git_haproxy_project.yaml](Link) 

4. keepalived_apply.yaml : This playbook will apply the new keepalived git_keepalived.conf on both servers. The priority between MASTER and SLAVE is configured according to the hostname of the servers. In our case we want to setup HAPROXY01 as master and HAPROXY02 as slave. To do so, the priority will be set to 101 if the server name contains "01" and to 100 if the server name contains "02". You need to change the project_dir setting to match your setup.
[https://github.com/cisel-dev/haproxy-awx-demo/blob/main/keepalived_apply.yaml](Link) 

5. haproxy_apply.yaml : This playbook will backup the old configuration. Validate the git_haproxy.cfg file and if validate set it as the actual config. Then the haproxy service is restarted to apply the new config file. You need to change the project_dir setting to match your setup.
[https://github.com/cisel-dev/haproxy-awx-demo/blob/main/haproxy_apply.yaml](Link) 

Now that we have all the configuration files and required playbook we need to create the AWX project, templates and workflow.

First we create a new Project that target our dedicated git repository

![2020-12-18 10_43_07-Automate HAproxy with ansible and AWX.drawio - diagrams.net.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1608284619619/xU2aI2AAX.png)

You also need to have your haproxy servers in an inventory. For this example we created a manual inventory with 2 hosts inside, HAPROXY01 and HAPROXY02. In reality we use dynamic inventories that are interfaced with service-now or other inventory/cmdb tools.

Then we will have to create 3 jobs and 1 workflow to execute these playbooks.

![2020-12-18 10_44_31-Automate HAproxy with ansible and AWX.drawio - diagrams.net.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1608284687515/MT0-Rig--.png)

Job to execute the clone_git_haproxy_project.yaml

![2020-12-18 10_45_46-Automate HAproxy with ansible and AWX.drawio - diagrams.net.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1608284759271/g88W27IPx.png)

Job to execute the keepalived_apply.yaml

![2020-12-18 10_46_33-Automate HAproxy with ansible and AWX.drawio - diagrams.net.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1608284809648/Ln99XF5oR.png)


Job to execute haproxy_apply.yaml

![2020-12-18 10_47_17-Automate HAproxy with ansible and AWX.drawio - diagrams.net.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1608284854282/GgXyf0nHv.png)

The Workflow will first execute the clone_git_haproxy_project, then we will apply the keepalived configuration and finally apply the haproxy configuration if valid.

![2020-12-18 10_49_03-Automate HAproxy with ansible and AWX.drawio - diagrams.net.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1608284961456/2di9mdKR-.png)

![2020-12-18 10_49_37-Automate HAproxy with ansible and AWX.drawio - diagrams.net.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1608284995301/62wujfnsH.png)

As a first try you can copy your actual and working haproxy and keepalived config to your repo and then execute the workflow.

This setup works with 2 HAproxy in failover mode but it can be adapted to be used on standalone haproxy server.


Feel free to contact us directly if you have any question at cloud@cisel.ch
[https://www.cisel.ch](Link) 

## Team

| <a href="https://www.cisel.ch" target="_blank">**CISEL**</a> | <a href="https://www.cisel.ch" target="_blank">**CISEL**</a> | <a href="https://www.cisel.ch" target="_blank">**CISEL**</a> |
| :---: |:---:| :---:|
| FCA | SME | QBA |

---

## Contact

Reach us at one of the following places!

- Website at <a href="https://www.cisel.ch" target="_blank">`cisel.ch`</a>
- LinkedIn at <a href="https://www.linkedin.com/company/cisel-informatique-sa/" target="_blank">`CISEL Informatique SA`</a>

---

## License
- We used the readme.md example from  <a href="https://gist.github.com/fvcproductions/1bfc2d4aecb01a834b46" target="_blank">fvcproductions</a> pour créer ce template.
- Copyright 2020 © <a href="https://www.cisel.ch" target="_blank">CISEL</a>.
