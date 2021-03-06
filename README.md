# FsnNodeHealth.py
Python program running in the background on your home PC, or backup VPS to check that your node is running properly. If there is a problem, it emails you to let you know. The programme is light weight and can be left running on your home PC or backup VPS indefinitely. You can either run it manually in a cmd window (Windows) or a bash shell (Linux), or you can add it to your startup folder (Windows) or use /etc/init.d (Linux) to make it automatically start when the machine reboots.

If you run the monitor from a backup VPS, you could manually 'fail over' using this if you receive an error warning email. If you maintain a reasonably up to date blockchain using a separate dummy Fusion wallet, then you will be back up and running quickly, before losing any tickets. Please note that there is no automatic fail over yet in place.

You can optionally record your node's data into a csv file for import into Excel/LibreOffice.

Details about your specific setup are in the the python program FsnNodeHealth.py, which can be edited using Notepad (Windows) or nano/vim (Linux) to change the IP addresses and email parameters and to understand how to run it.


# fusion_health_server_VPS.py
Python programme running on your VPS where your node is situated, that sends some docker log data to a port so that your home PC, or backup VPS can collect it. This lightweight programme runs in the background.



#
# FUNCTIONALITY

These two programs working together will check that :-

(1) The VPS is connected to the internet and can be pinged. Programme makes sure that YOUR internet is working first.

(2) Your home PC, or backup VPS can access the fusion docker logs and can extract the mined and imported blocks of your node.

(3) The mined and imported blocks of your node are advancing and are not too far out of sync. You can configure exactly how close you want it to be.

(4) The latest mined block of your node is close to the block height.  You can configure exactly how close you want it to be.

(5) There is sufficient free RAM on your node. You can configure exactly how much free RAM you want there to be.

(6) There is sufficient free disk space on your node's / partition. You can configure exactly how much free disk space you want there to be.

In addition the programme reports back to the home PC, or backup VPS how many FSN rewards have been earned.



# STEPS TO TAKE TO GET IT RUNNING

I've tried to be a careful as possible with my instructions here, but it assumes some level of knowledge about Linux. If you are reasonably proficient with the bash shell and Ubuntu or Debian, then you should have no problem by simply following the instructions below. If you are uncomfortable with Linux, I can handhold you through the process of setting up this programme and running a spare backup blockchain for a fee of 75 FSN payable when you are up and running. Please DM me @marcelsecu if you want this type of support.

FIRST:  download the code to your home PC, or backup VPS (either the zip or git clone https://github.com/marcelcure/FsnNodeHealthCheck.git), modify FsnNodeHealth.py for your email, IP address of your VPS and other parameters in the section labelled 'USER CONFIGURABLE SECTION'. Use Notepad (Windows) or nano/vim (Linux) to do this.

Make sure that you have port forwarding set on the firewall for TCP and UDP for the port 50505 (unless you have chosen a different one).

SECOND: download fusion_health_server_VPS.py to your VPS where your node is running :-

git clone https://github.com/marcelcure/FsnNodeHealthCheck.git

If you have a firewall set, make sure that you have port forwarding set on your VPS for TCP and UDP for the port 50505 (unless you have chosen a different one).

THIRD: run fusion_health_server_VPS.py on your VPS  

cd FsnNodeHealthCheck

chmod +x fusion_health_server_VPS.py  (only the first time, or if you git update)

Then start a screen process (you may have to install this first with the command sudo apt install screen ):-

screen

./fusion_health_server_VPS.py | tee --output-error=warn -a fusion_log.txt

This then waits for you to run FsnNodeHealth.py on your home PC or backup VPS (step shown below). 

If you see errors or the programme just stops, then try running the command without logging the output first to see any error messages. Just type ./fusion_health_server_VPS.py  You should see a message like 'waiting for client connection', which indicates that it is working properly. You can CTRL-C this when you have resolved any problems (e.g. port access problems) and are happy and then you can run it outputting to the fusion_log.txt file again so that you can close down your shell on your VPS.

You can safely detach from this screen shell using CTRL-A d  (that is CTRL-A followed by d key) without stopping the programme and you can also log out of the command shell too. When you come back to your staking VPS, you can reattach to this shell by typing the screen -r command.

NB If you are on Digital Ocean, you need to install a dependency for the netifaces python package. Run these 4 commands :-

sudo apt install python3-setuptools

sudo easy_install3 pip

sudo apt install python3-dev

sudo pip3 install netifaces

NB2 If you are running on AWS EC2  (Elastic Cloud Computing) VPS, you have to enable some rules in the configuration stage for port access and to allow the VPS to be seen from outside for services other than SSH.

There is a tab saying 6. Configure Security Group. In that tab, you see a rule for SSH, which you should leave. You select 'Add rule' and then choose 'ICMP IPv4' and to the right of that, choose 'Anywhere' and save that. You can then add rules for TCP and UDP access for port 50505 for 'anywhere', (or your monitor IP perhaps).


FINALLY:  run FsnNodeHealth.py on your home PC or backup VPS

for Linux :-

chmod +x FsnNodeHealth.py  (Only first time)

./FsnNodeHealth.py

or if you want to run this programme in the background so that you can close the shell then use the screen cammand as described before :-

screen

./FsnNodeHealth.py

and then CTRL-A d  (you can reattach to the screen with the command screen -r )

OR for Windows PC :-

python FsnNodeHealth.py

You will need to download and install python for Windows, since it doesn't come pre-installed. 

See https://www.python.org/downloads/  Click the button to update your PATH and start a new command shell before you proceed to refresh your PATH variable.

Please note that you will need to leave your PC on 24/7 with the command window open. If you are running a backup VPS as your monitor, then of course this isn't an issue.

Programmes can be stopped with CTRL-C but to stop fusion_health_server_VPS.py that is running in the background, you first have to attach to the screen process with the command screen.

If you stop fusion_health_server_VPS.py, then FsnNodeHealth.py will think that there is a problem and email you. This is a good check to make sure it is working OK. If you stop FsnNodeHealth.py on your home PC or backup VPS, then fusion_health_server_VPS.py will simply wait for you to reconnect (wait 1 minute before running FsnNodeHealth.py again to allow fusion_health_server_VPS.py to reset itself). Another sanity check is to put an nonexistant IP address for your VPS to check that emails are sent to you.


# HOW TO CHECK MULTIPLE NODES

You can set up the system to have one backup for multiple nodes by simply running fusion_health_server_VPS.py on each node and then running FsnNodeHealth.py multiple times on your home PC, or backup VPS machine. You must assign a different port number and IP address for each node in the configuration section. Make sure that the port number in fusion_health_server_VPS.py is the same as in each instance of FsnNodeHealth.py. I suggest that you use consecutive port numbers 50505, 50506, 50507 etc.

Another way to do it is to have 2 nodes each running with a different wallet and checking each other. This time each VPS runs both programmes, but with different port numbers and IP's. This is less satisfactory though, since you will not be able to fail over, when this is implemented. Another problem would be if ethernet connectivity was compromised on either of the nodes, since BOTH would then generate errors and send an email to you. It is better to have a completely separate, idle and up to date blockchain on a VPS.

The steps to set up 1 backup VPS checking 2 staking VPS's (stakingA and stakingB) are as follows :-

(1) On stakingA log in, download the code and change directory and make executable :-

git clone https://github.com/marcelcure/FsnNodeHealthCheck.git

cd FsnNodeHealthCheck

chmod +x fusion_health_server_VPS.py

(2) Simply start the programme :-

screen

./fusion_health_server_VPS.py | tee --output-error=warn -a fusion_log.txt

Make sure you see the message 'waiting for client connection' in the log file fusion_log.txt and then CTRL-A d and then log out.

(3) On VPS stakingB do the same things, but before you run it, alter the port to 50506 :-

nano fusion_health_server_VPS.py

edit line to be :-

PORT = 50506        # Port to listen on (use a non-privilidged port  > 1023)

Then CTRL-X Y  to exit editor

(4) On your backup VPS you should make separate folders for each staking VPS instant and download the code into each folder and then edit the code file to reflect the correct port number and other details :-

mkdir stakingA

cd stakingA

git clone https://github.com/marcelcure/FsnNodeHealthCheck.git

cd FsnNodeHealthCheck

chmod +x FsnNodeHealth.py

nano FsnNodeHealth.py

change all details to reflect your setup :- host_IP, pub_key, mail_user, mail_password, to email address.

CTRL-X Y  to save.

(5)  Start the monitor in a screen session :-

screen

./FsnNodeHealth.py

Make sure that you connect to stakingA by waiting a couple of minutes and that you see blockchain output. 

CTRL-A d detaches from the screen, but leaves the programme running.

Then go back to your home folder :-

cd 

(6) Setup for the next staking monitor for stakingB VPS in a similar way:-

mkdir stakingB

cd stakingB

git clone https://github.com/marcelcure/FsnNodeHealthCheck.git

cd FsnNodeHealthCheck

chmod +x FsnNodeHealth.py

nano FsnNodeHealth.py

change all details to reflect your setup :- host_IP, pub_key, mail_user, mail_password, to email address.

This time also change the port configuration to be :-

PORT = 50506     #   OK to leave 'as is' unless you are ...

CTRL-X Y  to save.

(7)  Start the monitor in the same screen session :-

screen

./FsnNodeHealth.py

Make sure that you connect to stakingB by waiting a couple of minutes and that you see blockchain output. 

You can exit the screen with CTRL-A d and then logout.

You can list the two screens running with the command screen -list

You can check to see if the monitor is working by reattaching to each of the backup VPS's using screen -r <choose the screen to attach to> and when you have seen it working, CTRL-A d and then screen -r <choose the other screen> CTRL-A d again. Then logout of your backup VPS
  
The backup VPS can also be used to run a 'hot spare' blockchain. This is best done as a normal user, not root.

Setup a dummy Fusion wallet and send it 0.1 FSN for gas. Get it running using Joey's script :-

curl -sL https://raw.githubusercontent.com/FUSIONFoundation/efsn/master/QuickNodeSetup/fsnNode.sh -o fsnNode.sh && sudo bash fsnNode.sh

Let the blockchain sync.

If you get email warnings from the monitor and you decide to switch to the backup VPS, you will need to reconfigure it to point to your staking wallet using fsnNode.sh. Please be careful that you do not run two nodes using the same staking wallet, since the punishment is severe - lose staking rights.


# PROBLEMS

Feel free to contact me on the Fusion TG to report bugs or to suggest improvements. Since I am using this myself, I will be happy to implement enhancements, if they are not too time consuming.

One problem I have found is that sometimes an old fusion_health_server_VPS process is left running on the VPS. This should be removed before running fusion_health_server_VPS.py

To check if there is an old process left behind :-

ps ax|grep fusion_health_server_VPS

You see an output similar to below 

14257 pts/1    S      0:00 python3 ./fusion_health_server_VPS.py

16335 pts/1    S+     0:00 grep --color=auto fusion_health_server_VPS


If you see it running (ignore the line with grep in it), then find the process ID number  (PID) and kill it :-

kill PID     - insert the actual number in the first column instead of PID

Another problem is that if you CTRL-C FsnNodeHealth.py on your home PC and then re-run it immediately, then fusion_health_server_VPS.py on the VPS won't be ready for it (you have to wait 1 minute). This will mean you have to kill the process fusion_health_server_VPS.py on the VPS and restart it, follwed by FsnNodeHealth.py on your home PC or backup VPS.

# TO DO

Soon I will change FsnNodeHealth.py to optionally use web3-fusion-extend using JS to extract the block info.

I will implement an automatic fail over to a spare 'hot' VPS, once I am sure that there are no false positive error warnings.
