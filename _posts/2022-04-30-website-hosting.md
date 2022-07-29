---
title:  "AWS server website"
toc: true
author: jumaruba
date: 2022-05-29 18:32:00 -0500
categories: [Theory]
tags: [aws]
image_path: /assets/images/2022-05-29-website-hosting
---

# AWS Free Tier  

To get free website hosting need create our server and [Amazon Free Tier](https://aws.amazon.com/free/) is here to help us. Our website for sure doesn't need big processing capacity, thus we can get a [Amazon EC2](https://aws.amazon.com/pt/ec2) machine to host it. We can have 750 hour for month during 12 months!  

After creating an account, you must be able to see the *Aws Management Console*. 

![AWS management console]({{page.image_path}}/aws_management_console.png)


Give a name to your machine and:
- Select `ubuntu` as operational system;
- 64-bit architecture;
- Allow HTTP traffic and HTTPs traffic from the internet;  
- A free tier can get up to 30GB of SSD, thus we can get 15 GB of space that should be enough to our needs. 

Now you are ready to create the machine. This process may take 2~7 minutes to be done.  

## Connecting with ssh 
In the **Instances** table check if our machine is running: 

![Instance]({{page.image_path}}/Instance.png)

Clicking in the instance with the right button in the mouse select the `connect` option. 

![Select connect option]({{page.image_path}}/connect_1.png)

In some moment while you was creating your machine, a `.pem` was downloaded to your machine. Be sure that you know where this file is located. We gonna need it! Ok, after clicking the `connect` button in the instance pannel, select the `SSH client` in the option in the top bar. The `example` is exactly the command that should be copied to the terminal. 

![Run with ssh command]({{page.image_path}}/connect_2.png)

To run this command you must: 
- Run it in the same folder where your `.pem` file is located; 
- Have openSSH installed in your pc. 

This command will open the ubuntu bash in your terminal!

![AWS ubuntu terminal]({{page.image_path}}/connect_3.png)

## Creating a simple website

### Dependencies 
Let's configure our machine to host a simple server. Create a folder called `server`, install `nodejs` and create a simple website. 

```bash 
mkdir server
sudo apt install nodejs
cd server 
```

### Creating script 

To learn how to create a cool website you can follow [this](https://www.digitalocean.com/community/tutorials/how-to-create-a-web-server-in-node-js-with-the-http-module-pt) instructions or, at least for now, you can create a simple index.js file and paste the following code: 

```javascript
var http = require('http'); // Import Node.js core module

var server = http.createServer(function (req, res) {   //create web server
    if (req.url == '/') { //check the URL of the current request
        // set response header
        res.writeHead(200, { 'Content-Type': 'text/html' }); 
        // set response content    
        res.write('<html><body><p>This is home Page.</p></body></html>');
        res.end();
    }

});

server.listen(5000); //6 - listen for any incoming requests
console.log('Node.js web server at port 5000 is running..')
```

This piece of code, will print `This home Page.` everytime a person access the Amazon url in port 5000.  

Now, let's execute the server: 

```
node server.js
```

### Accessing the website
Getting back to the [instances pannel](https://us-east-1.console.aws.amazon.com/ec2/v2/home?region=us-east-1#Instances:v=3) you can get the website url and let's try to access it: 

![Instance pannel]({{page.image_path}}/instances_pannel.png)

Don't forget to input the port in the url: 
```
ec2-xx-xxx-xxx-xxx.compute-1.amazonaws.com:5000
```

Probably you weren't able to access your website. Actually, we need to open the port to the exterior so we can access the website in the port 5000. The next section is dedicated to solve this "problem".  


## Port permissions 

### Creating a security group 
To open a port to the exterior, the `network security` options in the lateral menu and create a **security group**. 

Inside the security group add an **inbound rule**  and add the following rule:

![Create port rule]({{page.image_path}}/security_port.png)

### Updating instance security group 
Back to the instances pannel, select the option to change the security groups and add the recently created security group.


![Change security group]({{page.image_path}}/change_security_group.png)


## The port 80
This section was based in a [stack overflow post](https://stackoverflow.com/questions/60372618/nodejs-listen-eacces-permission-denied-0-0-0-080)

The port 80 is special and is protected by the linux system. Thus, to run a server in port 80, run the following commands: 

```bash
> sudo apt-get install libcap2-bin 
> sudo setcap cap_net_bind_service=+ep `readlink -f \`which node\``
```

This allows you to run a server in the port without having to give `root` permissions to the application. Even if after running this the program presents errors, follow the steps from the previous sections and configure the port 80. The port 80, though, is already configured in a default rule. Thus, remove the default rule and add your own. 


## Running the website in background

To run the website in background you can follow one of two approaches: 

### Using [PM2](https://stackoverflow.com/questions/36907766/amazon-ec2-nodejs-server-stops-itself-after-2-days-even-after-using-sudo-nohup)
PM2 is a process manager. You can do a lot of things with this package, but in essence you can initialize and check the status as: 

```bash 
npm install -g pm2  # installing 
pm2 start my_script.js  # start
pm2 status # check the status
```

## Using [screen](https://dev.to/akhileshthite/how-to-keep-ec2-instance-running-after-ssh-is-terminated-45k8)
```bash 
screen -d -m 
```
## Killing processes in a port
If a process is using port 5000, but you actually don't know what process it is, so you can kill it, run the command: 

```bash
sudo ss -lptn 'sport = :5000'
```

This should return what is the pid of the process that is using the port 5000. Knowing the pid, you can kill it by running:

```bash
sudo kill -kill <pid>
```


# Conclusion 

After following the last step your serve will be on, until the free plan is over. 




