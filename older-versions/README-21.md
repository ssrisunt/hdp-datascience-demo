#### Airline delay demo on HDP 2.1


##### Setup VM - Option 1: Import prebuilt VM
There is a prebuilt VM with the demo running on a HDP 2.1 sandbox [here](https://dl.dropboxusercontent.com/u/114020/Hortonworks_Sandbox_2.1_MLdemo.ova) - (R/scalding not available on this one yet)

You can simply import it into VMware Fusion, start the VM and follow the instructions in the readme under /root. You do not need to follow the rest of the instructions below.

##### Setup VM - Option 2: Setup demo on HDP 2.1 sandbox VM

- Download HDP 2.1 sandbox VM image (Hortonworks_Sandbox_2.1.ova) from [Hortonworks website](http://hortonworks.com/products/hortonworks-sandbox/)
- Import Hortonworks_Sandbox_2.1.ova into VMWare
- Now follow demo setup instructions below


##### Demo setup instructions

- Before starting the VM, in the "Hard Disk" VM settings set disk size to 65GB
- Before starting the VM, open the .vmx file and set numvcpus = "4" and memsize = "16000"
```
#e.g. if you are using VMWare Fusion on OSX 
vi "/Users/<your userid>/Documents/Virtual Machines.localized/<your VMname>.vmwarevm/<your VMname>.vmx"
```
- Now start the VM
- After it boots up, find the IP address of the VM and add an entry into your machines hosts file e.g.
```
192.168.191.241 sandbox.hortonworks.com sandbox    
```
- Connect to the VM via SSH (password hadoop) and start Ambari server
```
ssh root@sandbox.hortonworks.com
/root/start_ambari.sh
```

- Make any config changes required via Ambari e.g. the below YARN config changes via Ambari and restart YARN
```
yarn.nodemanager.resource.memory-mb = 9216 
yarn.scheduler.minimum-allocation-mb = 1536
yarn.scheduler.maximum-allocation-mb = 9216
```

- Create demo user
```

#add demo user and create home dir
useradd -m -d /home/demo -G users demo 
echo "demo:demo" | chpasswd
cp /etc/sudoers /etc/sudoers.bak
echo "demo    ALL=(ALL)       NOPASSWD: ALL" >> /etc/sudoers
```

- Setup automated ssh for demo user (needed for the R/Scalding demo)
```
su demo
ssh-keygen -b 2048 -t rsa -f ~/.ssh/id_rsa -q -N ""
ssh-copy-id demo@sandbox
#enter demo
chmod 644 ~/.ssh/authorized_keys
chmod 755 ~/.ssh
#test it works
ssh demo@sandbox
exit

```

As demo user, install needed software and setup the demo. This may run for 30+min
```
su demo
cd
#pull latest code and setup scripts
git clone https://github.com/abajwa-hw/hdp-datascience-demo.git	

#execute yum install steps that require root. Also bring down unnecessary services to save VM memory
sudo /home/demo/hdp-datascience-demo/step1_runasroot.sh

#install/setup iPython notebook, pydoop, pyspark
/home/demo/hdp-datascience-demo/step2_runasdemo.sh

#download data needed for airline demo
/home/demo/hdp-datascience-demo/step3_setupairlinedemo.sh
```


##### Launch demo on HDP 2.1

To run the python demo execute below then point your browser to port where ipython notebook starts on and open airline_python.ipynb
e.g. http://sandbox.hortonworks.com:9999
```

source ~/.bashrc
cd /home/demo/hdp-datascience-demo/demo
ipython notebook
```

To run the Scala/Spark demo execute below then point your browser to port where ipython notebook starts on and open airline_spark.ipynb
e.g. http://sandbox.hortonworks.com:9999
```
source ~/.bashrc
cd /home/demo/hdp-datascience-demo/demo
ipython notebook --profile spark
```

![Image](../master/screenshots/ipython-notebook-home.png?raw=true)


##### iPython Notebook embedded in Ambari View

![Image](https://raw.githubusercontent.com/abajwa-hw/iframe-view/master/screenshots/Embedded-iPython.png)

- You can embed the notebook within an Ambari by building the [Iframe view](https://github.com/abajwa-hw/iframe-view) using its instructions and pointing the url to your notebook e.g. http://sandbox.hortonworks.com:9999/notebooks/airline_python-2.2.ipynb

- To allow iPython to be embedded you need to add the below config to the end of your profile, if not already there, before starting the notebook and restart ipython notebook, otherwise you will get security violations when you try opening it from within Ambari
```
su demo
vi ~/.ipython/profile_default/ipython_notebook_config.py
#add this to the bottom to allow the notebook to disable the security policy and be embedded cross-domain

c.NotebookApp.tornado_settings = {
   'headers': {
       'Content-Security-Policy': ""
   }
}
c.NotebookApp.webapp_settings = {'headers': {'X-Frame-Options': 'ALLOW-FROM all'}}

```
- If the view shows up as blank, you can use the Chrome Developer Tools > Console to debug why the iframe is getting blocked