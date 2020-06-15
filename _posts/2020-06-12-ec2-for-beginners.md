---
layout: post
title: Beginner's Guide For Simple Web Server on EC2
---

This post is to help beginners on AWS get some hands on experience with building a very basic web server on an EC2 instance.

#### Prerequisites
- Have an AWS account (we will only be using services under the free tier so don't worry about this costing you any 💰).
- You are logged into the AWS Console.
- Not using windows (windows portion of the tutorial coming soon . . . maybe)

## Launch An EC2 Instance
In order to create our web server, we need to launch an EC2 instance. So from the AWS Management Console, navigate to the EC2 Dashboard found here under services:
![_config.yml]({{ site.baseurl }}/images/2020-06-12-mang-con.png)

This will take you to the EC2 Dashboard where you can see a breakdown of all running EC2 resources in your account. Assuming you have no instances running already, lets go ahead and click "Launch instance".

Next, you'll be taken through 6 steps to configure your instance. We will be leaving all the defaults since combing through all the specific details are out of scope of this tutorial.

1. **Choose AMI**
  - We'll stick to a basic Free tier Linux AMI (Amazon Machine Image). So go ahead and select this one below.
  ![_config.yml]({{ site.baseurl }}/images/2020-06-12-AMI_linux.png)

2. **Choose Instance Type**
  - Next you should see a massive table that shows the wide variety of instance types that AWS provides. This allows us to optimized our instance usage by selecting one geared towards our use case. (i.e. if you're doing research and need to run expensive calculations that require high performance processors, you would want to select an instance that is Compute Optimized). To find out more about the different instance types, checkout [this](https://aws.amazon.com/ec2/instance-types/) link.

    For this tutorial, let's stick to the Free tier general purpose t2.micro. Then click "Configure Instance Details"
  ![_config.yml]({{ site.baseurl }}/images/2020-06-12-instance-type.png)

3. **Configure Instance**
  - We aren't going to change any of the defaults on this page but just take note of all the options you have like how many instances to launch or the different VPC / Subnet values.

4. **Add Storage**
  - The configure the EBS (Elastic Block Storage), we could customize the volume type and size. But for now, just leave the defaults as is!

5. **Add Tags**
  - Tags are useful for managing all your different resources. If you have a large company with different departments, you may want to add a tag declaring which department that instance belongs to. Or even identify which employee created the instance along with what environment it belongs in (i.e. dev vs staging vs production).
  - As your organization grows, tags become very handy and allow you to easily filter through the different instances.

6. **Configure Security Groups (i.e. Virtual Firewall)**
  - Since we are making a web server that should be able to talk to the world, we'll want to add some rules that allows our instance to listen on the appropriate ports. By default SSH is enabled, which is how we will be connecting to our EC2 instance once launched.
  - Let's create a new security group and name it whatever you want. (i.e. something like "WebDMZ" so we know it's the security group for our web severs). Under the SSH rule, update "Source" to be anywhere so we will easily be able to SSH into our instance later (yes yes I know, bad practice). Normally this should be configured down to your specific IP address but this is just a basic tutorial to get your web server up.
  - Now let's add a new rule that allows people from the outside world to connect and view the content on our web server. Click "Add Rule", change the type to be "HTTP", and change the source to be anywhere so our instance will accept all web traffic coming in through port 80.
  - By the end, your setup should look like this:
  ![_config.yml]({{ site.baseurl }}/images/2020-06-12-security-groups.png)


7. **Review**
  - Here we can review the summary of the all the settings we just stepped through. If it all looks good, go ahead and smack that Launch button!
  - You should see a popup telling you to select a key pair that we will use to SSH into our instance. Let's create a new key pair and name it `tempKeyPair`. **Be sure to download this key pair before hitting launch instance and store somewhere safe. Anyone who has this pem file will be able to access your instance.**

  All that's left is to wait for the instance state to finish initializing.

## SSH & Install Apache

Now that the instance is live, let's connect to it via the command line. Open a new terminal and navigate to the directory you downloaded the `tempKeyPair.pem` file to. For me, that's my Downloads folder: `cd ~/Downloads`.
First thing we need to do is update the permissions on our `pem` file. To do so, run `chmod 400 tempKeyPair.pem`.

Now we can connect to our instance! From the EC2 dashboard, make sure the checkbox next to your running instance is selected and then hit Connect. You should see a modal like this:

![_config.yml]({{ site.baseurl }}/images/2020-06-12-ssh.png)

Go ahead and copy the example command (which should be customized based on your EC2 instance and keypair name). Paste that into the console and type 'yes' if prompted to continue connecting (this will add your computer to the list of known hosts). A successful connection will look like this:

![_config.yml]({{ site.baseurl }}/images/2020-06-12-ssh-connect.png)

Now we will run a series of commands to get our web service installed. Let's run the following:
1. `sudo su` this elevates our privileges to root so we can have full admin access to our instance.
2. `yum update -y` this updates the os will all the latest and greatest updates.
3. `yum install httpd -y` this installs apache and is what turns our little server into a web server!
4. To start the apache service, run `service httpd start`.
5. (optional) Now if our server ever reboots, normally we would want the apache service to restart as well. To make sure that happens we can run `chkconfig httpd on`.
6. To check that the apache service is up and running you can check the status at any time with `service httpd status`.
7. When we installed Apache, it created it's own directory at `/var/www/html` (this is the root directory of the web server). Right now, there is nothing there so lets update it and add in our very own HTML content that will render when people go to our site!
8. Open a new filed called `index.html` by opening the nano text editor. (`nano index.html`)
You can add whatever HTML content you want like so:

```html
<html>
  <body>
    <h1>
      Hello World, this is Brennon's Web Server!
    </h1>
  </body>
</html>
```
Then save & close the file (for nano hit `ctrl + x`, then `y`, and then `enter` to save the changes).

## Now for the moment of truth!
 Using the public IP address of your instance found here on the dashboard:
![_config.yml]({{ site.baseurl }}/images/2020-06-12-public-ip.png)

Copy & paste that into a new tab of your browser. You should see your `index.html` page!

![_config.yml]({{ site.baseurl }}/images/2020-06-12-html-page.png)

## Congratulations! 🎉
 You just configured your very own web server in the cloud that you can connect to using a public IP address.

### Cleanup
To cleanup the running EC2 instance, go ahead and hit the checkbox next to your instance and under "Actions -> Instance State" hit "Terminate". This will cleanup the EC2 resources we used during this tutorial.
