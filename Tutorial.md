
# Tutorial:

##  Section One: Starting from a Fresh Debian 12 server on DigitalOcean


### 1. Create the SSH directory

Run **Terminal** as **administrator** to do the following tasks:

> ##### Change directory
> Using command `cd`: to change the directory to the location where you want the SSH directory to be created, .
For example, I  wish to switch to ***c:\Users\song***
Type the following command in ***Terminal*** and press **ENTER**:

`cd c:\Users\song`

> ##### Created the .ssh folder under the directory
> Type the following command in ***Terminal*** and press **ENTER**:

`mkdir .ssh`

> ##### Create your new SSH key pair
> Type the following command in ***Terminal*** and press **ENTER**:

`ssh-keygen -t ed25519 -f .ssh/do-key -C "your-email-address"`

> *Note: Replace ***<your-email-address>*** with your actual email address. This email address is used for reference and does not serve any email-related function.

### 2.	Create a DigtalOcean account

#### 2.1 Sign up and verify your account on https://www.digitalocean.com/

#### 2.2 Generate public key in ***Terminal***

> Type the following command in ***Terminal*** and press **ENTER**:

`Get-Content c:\Users\song\.ssh\do-key.pub | Set-Clipboard`
> *Note: Replace ***<D:\PrivateServer>*** with your actual directory path to your .ssh folder. **Check whether your public key was successfully copied to the clipboard!** 

#### 2.3 Add public key to your DigitalOcean account
- Go to ***Settings*** in DigitalOcean account
- Click **Security** menu
- Click **Add SSH Key** to add your public key
- Press **CTRL + V** to paste your public key into the input box, then provide a name for the key. Finally, click **Add SSH Key** to complete

> *Note: A green notification will pop up at the top right corner of the screen, and you are able to locate your newly created SSH Key under SSH keys section now.

#### 2.4	Create a Droplet
- Click ***Create*** at the top right corner in your account dashboard, and choose **Droplets** from the pull down list.
- In the ***Choose Region*** section, select the region that is close to you.
- In the ***Choose an image*** section, select **Debian**.
- In the ***Choose size*** section, select your preferred ***Droplet Type*** and ***CPU options***.
- In the ***Choose Authentication Method*** section, select **SSH Key**.
> *Note: If you have ***only one*** SSH key, the SSH key will be automatically selected. 
> If you have ***multiple*** SSH public keys associated with your DitgitalOcean account, Select **all**.
- In the ***Finalize details*** section, modify the ***Hostname*** to your preference.
- Click ***Create Droplet*** to complete the process.
> *Note: A progress bar indicates the status of your Droplet's readiness.
> Once the process is complete, you will see your remote server's IP address.


## Section Two: Setting up a regular user to access your remote server

### 3.	Accessing your remote server via Terminal
Return to ***Windows Terminal*** command prompt. 
Type the following command in ***Terminal*** and press **ENTER**.
`ssh -i path-to-your-key root@IP address`
> *Note: Replace ***`<path-to-your-key>`*** with the actual path to your SSH key and *`<IP-address>`* with your remote server's IP address. 
> For example, if your SSH key is located at ***c:\Users\song\\.ssh\do-key.pub***, <br> and your server's IP address is ***147.182.234.89***, <br> your command would be: 
`ssh -i c:\Users\song\.ssh\do-key.pub root@147.182.234.89`

### 4.	Create a new regular user

#### 4.1 After logging into your remote server,  create a new regular user with following commands:
```bash
useradd -ms /bin/bash <user-name>
```
> *Note: Replace ***`<user-name>`*** with your preferred username.
> - `-s`  sets the path to the user’s login shell (**bash** here)
> - `-m`  create the user’s home directory if it does not exist

```bash
passwd <user-name>
```
> *Note: Replace ***`<user-name>`*** with your chosen username.

####  4.2 Add the user to the sudo group to allow the user to perform administrative tasks with the following command:
```bash
usermod -aG sudo <user-name>
```
> *Note: Replace ***`<user-name>`*** with your chosen username.
> - `-a`, --append: Add the user to the supplementary group(s). Use only with the `-G` option.

#### 4.3 Make new user possible to connect to the server via SSH with the following commands:
```bash
cp -r /root/.ssh /home/<username>
```
> *Note: Replace ***`<username>`*** with your chosen username.
> Copy the .ssh directory from the root users home directory to the new users home directory (including any files in the directory).

```bash
chown <username>:<username> .ssh
```
> *Note: Replace ***`<username>`*** with your chosen username. 
> Change ownership of the directory, and files in the directory so that the copy in your new users directory is owned by the new user and the new users primary group.

#### 4.4 Test that new user can connect to the server

You can do this by replacing ***root*** with your created ***`<user-name>`*** in the ssh command you use to connect to your server.

#### 4.5 Edit SSH configuration so that the root user can no longer connect to the server via SSH
> - The file we are looking for is the `sshd_config` file.  
> - Open the file in ***vim***. Use sudo for this.  
> ```bash
> sudo vim sshd_config
> ```  
> - Look for the line ***PermitRootLogin yes*** and change ***yes***, to ***no***.

## Section Three: Install and configure Nginx

### 5. Nginx

#### 5.1 Install nginx
First update your local package index to reflect the latest upstream changes
```bash
sudo apt update
```
Then, install the `nginx` package:
```bash
sudo apt install nginx
```
Confirm the installation, entering `Y`, then press `Enter` to proceed. `apt` will then install Nginx and any required dependencies to your server.

#### 5.2 Configure Nginx to serve a sample website
check if Nginx service status is ***enabled*** and ***running*** after installation by command:
```bash
sudo systemctl status nginx
```
change directory to ***/var/www*** first, then create a new ***my-site*** directory in it
```bash
sudo mkdir my-site
```

create an ***index.html*** document in the ***my-site*** directory, and add the specific html code into it, save it then.
```bash
sudo vim index.html
```

##### Creating our own server block

Create a new file named ***my-site.conf*** in ***/etc/nginx/sites-available***, and paste the following nginx config into it.
```bash
server {
 listen 80;
  
 root /var/www/my-site;
  
 index index.html;
  
 server_name _;
  
 location / {
 # First attempt to serve request as file, then
 # as directory, then fall back to displaying a 404.
 try_files $uri $uri/ =404;
 }
}
```

##### create a symbolic link 
create a symbolic link to new config file in ***/etc/nginx/sites-enabled*** with the following command:
```bash
sudo ln -s /etc/nginx/sites-available/my-site.conf /etc/nginx
/sites-enabled/
```
Since we already have default symbolic link in nginx service, we need to remove the default symbolic link by:
```bash
sudo unlink default
```
Then run the following command to test your nginx configurations.
```bash
sudo nginx -t
```
If you don't see any errors, restart the nginx service by the following command:
```bash
sudo systemctl restart nginx
```
Look up your ip address with:
```bash
ip a
```
 And try to run the following command to get your sample HTML:
 ```bash
 curl your-ip
 ```
 > *Note: Replace ***your-ip*** with the actual ip you found with `ip a` command.

<br></br>
**Congratulations! You have finished the tasks in in tutorial!**

