#Jenkins : Continous Integration
This is like a hot keyword right now, and it may get a bit dodgy understand how it all works under the hood. I'm going to try and simplify things as best as I can. Lets get into it.</br>

So, Jenkins is a java application, and it is platform independent. It's used for continous integration/continous delivery. </br>

##Continous Integration + Continous Deployment
So, you got a bunch of devs, typing away on their local machine, they may be working on a local copy of the `dev` branch and comitting their code up using `Git` or `TFS` or whatever. It's the end of the day, everyone has committed their code and its time to
make the build.</br>Now, if the build fails because some buggy code, you're going to have to go back, look through all the commits for the day to find out what broke your build. It's going to be a hassle. Truest me. I may be young be I've seen it happen and it ain't pretty.

What Jenkins does is, as soon as a developer has comitted code in the shared repository, Jenkins takes that code and triggers a build.</br>So we are talking building exe, websites, apps etc. So, we find out immediately which commit broke our code and fix it. 
In the case our builds are successful, we can run tests (Gradle, JSHint, Nunit etc) as post build tests.</br>These tests will run automated and we shall get a report back. How neat is that. Once everything checks out, its deployed. Continously. Hence , continuous deployment.

##Downloading Jenkins
Click on [Download Jenkins](https://jenkins.io/) and you can go ahead and get the `war` file for your OS.</br> 
Place that war in a location, then open up a `cmd` or `terminal` in the associated folder.</br>

`java -jar jenkins.war` - to execute and install Jenkins.</br>
OR</br>
```
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt-get update
sudo apt-get install jenkins
```
For Centos Machines, follow [this](https://www.digitalocean.com/community/tutorials/how-to-set-up-jenkins-for-continuous-development-integration-on-centos-7) guide.</br>

For running it as a daemon, make a file named `jenkins.service`at `cd /etc/systemd/system/`.</br>
Vim into it and add the following ::
```
[Unit]
Description=Jenkins Service
After=network.target

[Service]
Type=simple
User=root
ExecStart=/usr/bin/java -jar /usr/local/bin/jenkins.war
Restart=on-abort

[Install]
WantedBy=multi-user.target
```
</br>
So, when you're ready to run, just do: `sudo systemctl daemon-reload`</br>
To start it : `sudo systemctl start jenkins.service`</br>


It's going to install everything and you'll have Jenkins on your browser : `localhost:8080` or use a different port ::
You'll also get a password, so keep it safe. Copy it somewhere and save it. Once the installation is over and you hop over to localhost:8080 , you're going to find a page that says 'unlock jenkins'. Yeah, copy and paste the password you just saved.
"But what if I forgot my password?" - (-_-) , go to `yourpath/.jenkins/secrets/initialAdminPassword` to get it. 
You enter the password, and you get into the intro screen. Go ahead and select the plug-ins you want. I used suggested plug-ins. Create username and all that jazz. Everything works out, you should see your Jenkins dashboard.
Jenkins is going to be a hidden folder.
</br>
For MAC users, if you can't see your `.Jenkins` folder, do this : `defaults write com.apple.finder AppleShowAllFiles TRUE` 
</br>
One thing to note is that once you install Jenkins, you're going to have a directory structure. That will house all your related files. </br>
`jobs` will have all your created jobs.</br>
`nodes` which has all the slave nodes.</br>
`plugins` has all the plugins (duh) , also all plugins have `.jpi` extension. So, if you don't find the one's you're looking for, just download and paste a .jpi .</br>
`userContent` allows you to share your files over http. Whatever you drop here, will be available to people you want to share with.</br>


Now, your Jenkins can exist on your server (Tomcat or any other falvour) but it can also exist as a standalone. If you are concerned about port-conflicts, do this :  `java -jar jenkins.war --httpPort=<port_num>`
</br>
You can deploy any project (C++, Java, Python) using jenkins. Just make a `RPM` out of your project.
##Security
Before you go do anything(create new jobs) with Jenkins, go ahead into the `Manage Jenkins` tab, and then select `Configure Global Security`, then click on the 'enable security' checkbox. You're going to find many options. Select the ones that work best for you.
For those who chose Security Realm = Jenkins own user database (Allow users to sign up) & Authorization = Logged-in users can do anything , make sure to create a user and log-in .

##Connect with GIT and deploy
First you need to have the GIT plugin installed. Go ahead and do that. </br>
Then, create a job (freestyle project) and add a description that suits you. Now, you should be having a GIT repo you want to associate with Jenkins : `https://github.com/username/repo.git`. Make sure to selecte (discard Old builds) so that you don't keep encroaching your disk space.
Specify the number of days to keep the builds, and specify the number of builds you want to keep.</br>
Select `Source Code Management` to `Git` and paste the `Repository URL`</br>
If your repo was public, you would not to put in any security credentials. If it is private, you're going to need to add it. </br>

To keep the system running, you need to `Build Triggers` as in to do it via scripts, after other projects are built, in a time frame or have it Poll for user commits to trigger a build.</br>
Note that you need to have `webhooks` installed in order to do this (which is going to be in the jenkins config).</br>

In the `Build` feature, you have a section to add `execute windows batch commands` or a linux vairant. </br>
For windows, it's going to be :`.\scripts\<filename>.bat --single=run recorders="tests.junit"`
For our Linux brethern, two simple commands for our build.</br>
```
npm install
node <app or server> 
```
</br>


If you look inside the `jobs` folder, you're going to find a `builds` folder and a `config.xml` file. </br>

Now, where ever you are running jenkins, make sure you have the build stuff working on the server.</br>

If you go and look at the build console, you'll see where and how stuff happened. If any failures occur, debug it. It's probably something missing, so apt-get install that stuff.</br>

## Nodes 
Go to manage Jenkins>Manage Nodes </br>
Click on `New Node` and enter name and activate the radiobutton on 'Dumb Slave'.</br>
Now, if you want more build processes taking place, increase the number of executors. </br>
Remote Root Directory is where all your config files are stored. In our master, we have everything stored in `~/opt/jenkins` and similarly we need to store those files in our Node as well. </br>
Set your Remote Root Directory to `/var/jenkins`. Make sure you've defined it : `mkdir /var/jenkins` </br>
Your `Launch Method` can be SSH (in which case you'd need to add SSH keys) along with a Host address.</br> Choose your own flavour of availibility. </br>
In Node Properties, you've got tool locations. If you're serving Java pages, you need to use your Java JDK
Once you save, everything fires up. Now you can go back to jobs, fire up a new job. In the job config page, you have an option called `Restrict where this project can be run`.</br>
When you click that, you get an option to select the Node name that you want to run it on. Breeze through your config and Build your job. You got your build going and concluded on a remote location.


#That's all for now, folks! :) 


 
