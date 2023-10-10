# Splunk-Snort-SIEM


<h1>Splunk SIEM with Snort IDS</h1>

<h2>Lab Description:</h2>
<p>In this Repository you will find instructions on how to deploy a Splunk SIEM with logs forwarded from a Snort IDS. In walk through lab you will find each step to configure the enviroment.</p>><br />
    ADD COMPLETION PHOTOS

<h2>Installation and configuration of Snort</h2>
<p>In this section we will deploy a linux machine that properly runs Snort</p><br />
<p>Note prerequisites: virtual box installed and a copy of Ubuntu 22.0.04 ISO</p>
<p>Iso can be downloaded here: https://ubuntu.com/download/desktop</p>

1. Create virtual machine with at least 2 CPU and 4 GB RAM <br />
   ![Image Alt Text](images/2023-10-09_17-32.png)
2. Update and Upgrade machine:
   - sudo apt update && upgrade -y
3. Before the next step know your interface name and your IP address range. You will be required to enter them following the next step.
4. Install snort with the following command:
   - sudo apt-get install snort -y<br />
   When promted enter you address range and interface name
5. Now we need to edit the snort configuration file to add the ip range.
   - sudo nano /etc/snort/snort.conf<br />
    ![Image Alt Text](images/snort-conf.png)
6. Note in the snort configuration file view the location for local rules that we will create in future steps:
   - the location of the local rules is: /etc/snort/rules/local.rules
    ![Image Alt Text](images/snort-localrules.png)
8. Test confirguation succesfullness with the following command:
    - sudo snort -T -i enp3s0 -c /etc/snort/snort.conf <br />
    - This command will test your your configuration. If it is correct it will show succesfull confirguation<br />
    
<h2>Snort Rules/h2>
<p>In this section we are going to create local rules to add to our snort rules. A very useful tool to use when creating snort rules is a tool called Snorpy. </p>

1. We need to edit the local rules file to add files:
    - sudo nano /etc/snort/rules/local.rules
2.










