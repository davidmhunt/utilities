This is a guide for setting up SSH servers within the CPSL lab. A few more helpful resources can be found here:

1. [Guide to configuring SSH in ubuntu](https://itsfoss.com/set-up-ssh-ubuntu/)
2. [Adding public key to ubuntu server](https://linuxhandbook.com/add-ssh-public-key-to-server/?ref=itsfoss.com)
3. [transfering files](https://www.baeldung.com/linux/transfer-files-ssh)
4. [Running jupyter notebooks over ssh](https://stackoverflow.com/questions/69244218/how-to-run-a-jupyter-notebook-through-a-remote-server-on-local-machine)

# Setuping up servers and clients

## Setting Up Ubuntu Server

### Check ssh status
1. First, check to see if the ssh package is already installed and running on the machine by running the following command in the terminal
```
service ssh status
```
You should get something stating that the service is active. If so, move onto setting up your ssh client. If it isn't installed, use the following steps to install ssh on the server

### Installing SSH on the server
1. Run the following commands in the terminal to install ssh
```
sudo apt update && sudo apt upgrade
sudo apt install openssh-server
```

## Setting Up Linux Client
To start, we will need to setup the openssh client on unbuntu.

### Installing Open SSH client on your machine
1. Check to see if ssh is already installed on your machine by using the following command
```
dpkg -l | grep ssh
```
If you see "openssh-client" listed, then ssh is already installed and you can move onto the post-installation steps

2. If you don't have ssh installed on your machine, go ahead and install it using the following command
```
sudo apt install openssh-client
```


## Setting Up Windows Client

### Installing Open SSH
To start, we will need to setup OpenSSH on windows. To do so, follow the [openssh windows installation guide](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse?tabs=gui)

#### Prerequisites
1. Check version of windows on the device. Open PowerShell as an administrator and type
```
winver.exe
```
2. Configm that Powershell is on at least version 5
```
$PSVersionTable.PSVersion
```
3. Configrm that your user is part of an administrator account using (should return true)
```
(New-Object Security.Principal.WindowsPrincipal([Security.Principal.WindowsIdentity]::GetCurrent())).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
```
#### Installation
1. Open "Settings", select "Apps", then select "Optional Features"
2. Check to make sure that "OpenSSH Client" is installed. If not, select "Add a feature" and install "OpenSSH Client"
3. Open the Services desktop app. (Select Start, type services.msc in the search box, and then select the Service app or press ENTER.)

# Connecting to server over SSH

## 1st connection
1. To initially connect to the ssh server, type the following
```
ssh username@ipaddress
```
where "username" is your username on the computer and "ipaddress" is the ipaddress of the server. Once connected, you will get something similar to the following prompt:
```
The authenticity of host 'servername (10.00.00.001)' can't be established.
ECDSA key fingerprint is SHA256:(<a large string>).
Are you sure you want to continue connecting (yes/no)?
```
After this, you will be prompted for a password. Enter the password to log into the server.

## Creating a public-private key for more secure authentication
We will now create a public-private key to allow for a more secure connection. The instructions are for Windows machines, but they should still work for linux/ubuntu machines as well
1. Open powershell and enter the following command
```
ssh-keygen -t ed25519
```
The output should display the following (where "username" is replace by your username). 
```
Generating public/private ed25519 key pair.
Enter file in which to save the key (C:\Users\username/.ssh/id_ed25519):
```
Press enter to accept the default settings. When prompted to enter a passphrase, you can either type a passphrase or simply leave it empty. The output after this should appear as follows:
```
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in C:\Users\username/.ssh/id_ed25519.
Your public key has been saved in C:\Users\username/.ssh/id_ed25519.pub.
The key fingerprint is:
SHA256:OIzc1yE7joL2Bzy8!gS0j8eGK7bYaH1FmF3sDuMeSj8 username@LOCAL-HOSTNAME

The key's randomart image is:
+--[ED25519 256]--+
|        .        |
|         o       |
|    . + + .      |
|   o B * = .     |
|   o= B S .      |
|   .=B O o       |
|  + =+% o        |
| *oo.O.E         |
|+.o+=o. .        |
+----[SHA256]-----+
```
Here, the .pub files are your public keys and the files without extensions are private keys. To protect the private keys, enter the following commands (uncommented out ones) in a powershell prompt (windows only)
```
# By default the ssh-agent service is disabled. Configure it to start automatically.
# Make sure you're running as an Administrator.
Get-Service ssh-agent | Set-Service -StartupType Automatic

# Start the service
Start-Service ssh-agent

# This should return a status of Running
Get-Service ssh-agent

# Now load your key files into ssh-agent
ssh-add $env:USERPROFILE\.ssh\id_ed25519
```

### Adding Public Key to Server

#### Linux users: 
1. If you are using linux, use the following command to add your public key to the server.
```
cd ~/.ssh/
ssh-copy-id -i id_rsa.pub username@ipaddress
```
here, replace id_rsa.pub with the name of the public key you created, username with your user name on the server, and ipaddress with the ip address of the server.

#### Windows users:
1. Follow the instructions under "Setting Up Remote SSH on VS Code" to add the newly created public key to the server

## Setting Up Remote SSH on VS Code
### Setup SSH Extensions in VS Code
1. In VSCode, install the following extensions:
    1. Remote - SSH
    2. Remote - SSH: Editing Configuration Files

### Setting up the ssh .config file
1. In the VS code side pane, open up the Remote Explorer tab denoted by this icon:  ![Remote Explorer Icon](/ssh/VS%20Code%20SSH%20Extension.png)
2. In the pane, under "SSH", click on the gear, and select "C:\Users\USERNAME\config" where USERNAME is your windows username. The gear is the gear that appears in the below image

![SSH Edit Config](/ssh/ssh_config_setting.png)

3. With the config file open, copy in the following text
```
Host DESIRED_NAME
    HostName IP_ADDRESS
    User USERNAME
    ForwardX11 yes
```
Here, DESIRED_NAME is a name of your choosing that you will use when ssh'ing into the server, IP_ADDRESS is the ip address of the server that you are sshing into, and USERNAME is your username on the server.

### Connecting to Server on VS Code
1. Re-launch vscode and go back to the Remote Explorer tab. Under SSH, you should now see the server listed by the DESIRED_NAME you set in the previous step. Click on either of the two icons next to the server name to connect via ssh. For now, you will be asked for the password for your account on the remote server.

![SSH Connecting VSCode](/ssh/ssh_connecting_to_ssh_vscode.png)

### Adding Public Key to Server using VSCode
1. Connect to the server in VS Code using the previous step. 
2. In VS Code, go to file -> open folder. Use the path /home/USERNAME/.ssh/ and click "OK". 
3. Open the file called authorized_keys. If one doesn't exist, create a new file called authorized_keys and open it. 
3. Back on the client (local machine), navigate to one of the two locations
    1. Linux: ~/home/USERNAME/.ssh
    2. Windows: C:\Users\USERNAME\.ssh
4. Using notepad (or the text editor of your choise), open the .pub file (ex:id_ed25519.pub) that you created in the previous sections (MAKE SURE NOT TO USE THE ONE WITHOUT THE .pub AS THIS IS YOUR PRIVATE KEY).
5. Copy all of the text from the .pub file and paste it onto a new line in the authorized_keys file on the server. 
6. Save the authorized_keys file

## Connecting to Remote Server
Once the config file has been setup and your public key has been added to the authorized_keys file on the server, there are now two ways to connect to the computer
1. VSCode: Use this method when writing code on the terminal
    1. In VSCode, go to the "Remote Explorer" tab, and connect to the server. You should no longer be prompted for a password
2. Via Terminal/PowerShell: Use this when entering terminal commands on the server
    1. Open terminal (linux) or powershell (windows) and type in the following command:
    ```
    ssh DESIRED_NAME
    ```
    where DESIRED_NAME is the name that you used in your config file.