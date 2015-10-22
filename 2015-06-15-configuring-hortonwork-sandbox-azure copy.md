---
layout: post
title: Configuring Hortonworks Sandbox on Azure
date: '2014-06-15T12:34:00.001-07:00'
author: Saptak Sen
tags:
- azure
- hadoop
modified_time: '2014-06-15T15:11:18.054-07:00'
---
For folks attending the workshop at Hadoop Summit, San Jose 2015 we provided Microsoft Azure Pass. If you already have an Azure account skip this step. If you are following along at home you can sign-up for an [Azure Trial](http://azure.microsoft.com/en-us/pricing/free-trial/) or [download the Hortonworks Sandbox](http://hortonworks.com/sandbox) on your machine.

To use the Microsoft Azure Pass navigate to [http://www.microsoftazurepass.com](http://www.microsoftazurepass.com)

![](https://www.dropbox.com/s/o5x4rbcgbybmqfo/Screenshot%202015-06-04%2019.42.35.png?dl=1)

On  the next step, you will need to provide a Live Id that is not already tied up with an existing Azure account.

![](https://www.dropbox.com/s/rcwmsioszdms1c6/Screenshot%202015-06-04%2019.42.46.png?dl=1)

You will receive $100 credit, which in my estimate is worth Hortonworks Sandbox node running on a A3 size VM for about 12 days. Your mileage may vary:

![](https://www.dropbox.com/s/vkl3mgn2prukcmt/Screenshot%202015-06-04%2019.43.15.png?dl=1)

Complete the signup:

![](https://www.dropbox.com/s/iruyqizq1q4b9qd/Screenshot%202015-06-04%2019.45.41.png?dl=1)

Wait, your Azure account will be ready in a few minutes

![](https://www.dropbox.com/s/hpunlstu4b1yvjw/Screenshot%202015-06-04%2019.47.16.png?dl=1)

Start by logging into the Azure Portal with your Azure account:[https://portal.azure.com/](https://portal.azure.com/)

![](http://hortonassets.s3.amazonaws.com/tutorial/azure-sandbox/01.png)  


Navigate to the `MarketPlace` 

![](http://hortonassets.s3.amazonaws.com/tutorial/azure-sandbox/04.jpeg)  


Search for Hortonworks. Click on the Hortonworks Sandbox icon.

![](http://hortonassets.s3.amazonaws.com/tutorial/azure-sandbox/08.png)  


To go directly to the Hortonworks Sandbox on Azure page navigate to [http://azure.microsoft.com/en-us/marketplace/partners/hortonworks/hortonworks-sandbox-sandbox22/](http://azure.microsoft.com/en-us/marketplace/partners/hortonworks/hortonworks-sandbox-sandbox22/)

This will launch the wizard to configure Hortonworks Sandbox for deployment.

![](http://hortonassets.s3.amazonaws.com/tutorial/azure-sandbox/10.png)  


Note the highlighted text in the instructions above. You will need to note down the `hostname` and the `username`/`password` that you enter in the next steps to be able to access the Hortonworks Sandbox once deployed.

![](https://www.dropbox.com/s/0umj26zoaa68h86/Screenshot%202015-06-04%2019.53.55.png?dl=1)  


I recommend you select `Standard A4` for the pricing tier, so that you get more memory to play with. You can change other configurations if you want, but the defaults work well. I usually change the location to the datacenter to where my preexisting Azure resources are or the one closest to me.

![](http://hortonassets.s3.amazonaws.com/tutorial/azure-sandbox/14.png)  


Click `Buy` if you agree with everything on this page.

![](http://hortonassets.s3.amazonaws.com/tutorial/azure-sandbox/16.png)  


At this point it should take you back to the Azure portal home page where you can see the deployment in progress.

![](https://www.dropbox.com/s/pjqok0bppc6i3hl/Screenshot%202015-06-04%2019.55.54.png?dl=1)  


Once the deployment completes you will see this page with configuration and status of you VM. Again it is important to note down the DNS name of your VM which you will use in the next steps.

![](https://www.dropbox.com/s/9z9jsjqdmklq6gi/Screenshot%202015-06-04%2020.00.03.png?dl=1)  


If you scroll down you can see the Estimated spend and other metrics for your VM.

![](http://hortonassets.s3.amazonaws.com/tutorial/azure-sandbox/22.png)  

Azure sets the SSH port dynamically for security reasons. Let's look up the SSH port.

To look up the SSH port click on the `Settings` icon on the top panel

![](https://www.dropbox.com/s/wr8e07l3vssyb49/Screenshot%202015-06-04%2020.01.49.png?dl=1)

Click on `Endpoints` and scroll down to note the public SSH port for your VM

![](https://www.dropbox.com/s/cfwk3qykqjx2ihr/Screenshot%202015-06-04%2020.02.25.png?dl=1)

Now we can use the command below to login.

```bash
ssh <username>@<hostname>.cloudapp.net -p <port>;
```

Replace `<username>`, `<hostname>` and `<port>` as you have noted down during your deployment. The password is the same password that you have provided during deployment.

![](https://www.dropbox.com/s/w49sxetruslq065/Screenshot%202015-06-04%2020.14.20.png?dl=1)

Once you login, let's change the `root` password to a known password using the command

```bash
sudo passwd root
```

![](https://www.dropbox.com/s/flioznhrl97ik35/Screenshot%202015-06-07%2015.43.44.png?dl=1)

Now we can login as `root` with the command `su`

![](https://www.dropbox.com/s/e1euvrfwau5v9u4/Screenshot%202015-06-07%2015.48.04.png?dl=1)

Next we are going to change the hostname of the VM to `sandbox.hortonworks.com` with the command

```
sudo hostname sandbox.hortonworks.com
```

![](https://www.dropbox.com/s/p01exun3ww9v75e/Screenshot%202015-06-07%2020.21.21.png?dl=1)

Let’s navigate to the home page of your Sandbox by pointing your browser to the URL: `http://<hostname>.cloudapp.net:8888` , where `<hostname>` is the hostname you entered during configuration.

If you are doing it for the first time, it will take you to the registration page.

![](http://hortonassets.s3.amazonaws.com/tutorial/azure-sandbox/24.png)  


Once you register, you will see the homepage of your Sandbox.

![](http://hortonassets.s3.amazonaws.com/tutorial/azure-sandbox/26.png)  

Now navigate to port 8000 of your Hortonworks Sandbox on Azure from the browser.

![](http://hortonassets.s3.amazonaws.com/tutorial/azure-sandbox/28.png)  


Enable Ambari by clicking on the `Enable` button. Ambari is crucial for managing your HDP instance.

![](http://hortonassets.s3.amazonaws.com/tutorial/azure-sandbox/30.png)

If you want a full list of tutorial that you can use with your newly minted Hortonworks Sandbox on Azure go to [http://hortonworks.com/tutorials](http://hortonworks.com/tutorials).
