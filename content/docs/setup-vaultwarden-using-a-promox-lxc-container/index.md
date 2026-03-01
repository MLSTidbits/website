---
title: Setup Vaultwarden Using a Proxmox LXC Container
date: 2026-03-01
draft: false
featuredImage: ""
externalLink: ""
---

![password manager](./logo.png)

In a world where everything about our lives is digitized and for sale to the highest bidder, it’s more important than ever to take control of our data and protect our privacy. One way to do this is by using a password manager, which can help you generate and store strong, unique passwords for all your online accounts.

There are many password managers available from commercial options like [LastPass](https://www.lastpass.com/) and [1Password](https://1password.com/) to those built into the browsers. However, if you want to take control of your data and host your own password manager, [Vaultwarden](https://github.com/dani-garcia/vaultwarden) is a great option a smaller and more lightweight version of [Bitwarden](https://bitwarden.com/), which is a popular open-source password manager.

Having used **Vaultwarden** for a while now, I can confidently say that it’s a fantastic choice for anyone looking to host their own password manager. It’s easy to set up and use, and it offers all the features you would expect from a password manager, including password generation, secure storage, syncing across devices and all passwords are stored locally on each device. This means that if the server goes down you will still have access to your passwords on that device. The same cannot be said for the commercial options, which store your passwords in the cloud and can be vulnerable to data breaches [^1] and cyberheists [^2].

[^1]: [LastPass Data Breach](https://www.upguard.com/blog/lastpass-vulnerability-and-future-of-password-security)
[^2]: [$150 Million LastPass Cyberheist](https://krebsonsecurity.com/2025/03/feds-link-150m-cyberheist-to-2022-lastpass-hacks/)

## Step 1: Download a Template

I'm going to assume you already have Proxmox installed and running; and that you have a basic understanding of how to use it. If you don't, there are plenty of resources available online to help you get started. The first place to start is the [Proxmox documentation](https://pve.proxmox.com/pve-docs/).

![Create LXC Container](./CT-Storage.png)

Click on the "Templates" button and search for the latest version of Debian. Once you have found it, click on the "Download" button to download the template. Once the template is downloaded, click on the "Create CT" button to create a new container.

## Step 2: Configure the Container

`General`
: **Node**: Leave it as is
: **CT ID**: Let Proxmox assign an ID for you
: **Hostname**: Choose a hostname for your container (e.g., vaultwarden)
: **Password**: Leave it blank and select "Use SSH public key authentication" (we will set up SSH keys later)

Click on the "Load SSH Key File" button and select your public SSH key file (e.g., `~/.ssh/id_rsa.pub`). This will allow you to log in to the container using SSH without a password.

> [!TIP]
> If you don't have an SSH key pair, you can generate one using the following command:
>
> ```bash
> ssh-keygen -P '' -t rsa -b 4096 -C "$USER@$(hostname)" -f ~/.ssh/key/Github
> ```

Click on the "Next" button to continue.

`Template`
: **Template**: Select the Debian template you downloaded earlier.

Click on the "Next" button to continue.

`Disk`
: **Storage**: Select the storage location for your container (e.g., local-lvm)
: **Disk size**: Set the disk size for your container (e.g., 8 GB)

Click on the "Next" button to continue.

`CPU`
: **Cores**: Set the number of CPU cores for your container (e.g., 2)

Click on the "Next" button to continue.

`Memory`
: **Memory**: Set the amount of RAM for your container (e.g., 512 MB)
: **Swap**: Set the amount of swap space for your container (e.g., 512 MB)

Click on the "Next" button to continue.

`Network`
: **Bridge**: Select the network bridge for your container (e.g., vmbr0)
: **Static IP**: Set a static IP address for your container
: **IPv4/CIDR**: Set the IP to something outside of the DHCP range of the router (e.g., 192.168.1.200/24)
: **Gateway**: Set the gateway to the IP address of your router (e.g., 192.168.1.1)

`DNS` leave it as is offend set to use the hosts DNS settings.

Click on the "Next" button to continue.

`Confirm` review the settings for your container before clicking on the "Create" button chick the box to start the container after it is created.

## Step 3: Update and Install Dependencies

Once the container is created and running, you can log in to it using SSH. Open a terminal and use the following command to log in:

```bash
ssh -i ~/.ssh/id_rsa root@<container_ip>
```

Once you are logged in, you can update the package list and install the necessary dependencies for Vaultwarden using the following commands:

```bash
apt update && apt upgrade -y && apt install -y curl wget unzip mariadb-server argon2 podman
```

## Step 4: Configure MariaDB

[MariaDB](https://mariadb.org/) is a popular open-source relational database management system that is a fork of MySQL. It is offend used as the database backend for most self-hosted applications, including Vaultwarden because of its performance, reliability, and ease of use. In this step, we will configure MariaDB to work with Vaultwarden.

To start enter the MariaDB shell, use the following command: `mysql_secure_installation`. Once you are in the MariaDB shell, you can create a new database and user for Vaultwarden using the following commands:

Answer the prompts as follows:

* **N** - unix_socket authentication plugin
* **N** - change the root password
* **Y** - remove anonymous users
* **Y** - disallow root login remotely
* **Y** - remove test database and access to it
* **Y** - reload privilege tables

```sql
CREATE DATABASE vaultwarden;
CREATE USER 'vaultwarden'@'localhost' IDENTIFIED BY 'your_secure_password';
GRANT ALL PRIVILEGES ON vaultwarden.* TO 'vaultwarden'@'localhost';
```

Replace <mark>your_secure_password</mark> with a strong password for the Vaultwarden database user. Enter `quit` to exit the MariaDB shell. Now that we have configured MariaDB, we can move on to installing Vaultwarden.

## Step 5: Install Vaultwarden

Instead of using Docker to run Vaultwarden, we will use [Podman](https://podman.io/) which is a daemonless container engine that is compatible with Docker. Because it is daemonless, it does not require root privileges to run containers and it more compatible with LXC containers.

A [Docker](https://docs.docker.com/engine/install/debian/) container can run inside an LXC container, however, do to the way Docker works it will offend require a manual starting of the Docker daemon after a reboot of the main container. This is because **Docker** itself is meant to run on a host machine or virtual machine.

To install Vaultwarden using Podman, you can use the following command:

```bash
podman run -d \
    --name vaultwarden \
    --network host \
    -v /vlt/:/data/:Z \
    -e ROCKET_PORT=80 \
    -e DATABASE_URL='mysql://vaultwarden:your_secure_password@127.0.0.1:3306/vaultwarden' \
    -e ADMIN_TOKEN="$(echo -n 'your_secure_password' | argon2 "$(openssl rand -base64 32)" -e -id -k 65540 -t 3 -p 4)" \
    vaultwarden/server:latest
```

Replace <mark>your_secure_password</mark> with the password that you want the admin panel to use. This command will start a new container named **vaultwarden** using the latest version of the Vaultwarden server image. The container will be connected to the host network, and the data will be stored in the `/vlt/` directory on the host machine. The `ROCKET_PORT` environment variable is set to `80` to allow access to the Vaultwarden web interface on port 80. The `DATABASE_URL` environment variable is set to connect to the MariaDB database we configured earlier. The `ADMIN_TOKEN` environment variable is set to a randomly generated token that is hashed using Argon2, which is a secure password hashing algorithm. This token will be used to access the admin panel of Vaultwarden.

To access the Vaultwarden web interface you well need to point your browser the IP address of the container to a valid domain name or have a SSL certificate set up which can be more difficult then setting up a reverse proxy like [Nginx](https://www.nginx.com/) or [Caddy](https://caddyserver.com/).

> [!TIP]
> Using [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/) is a great way to securely expose your Vaultwarden instance to the internet without having to worry about SSL certificates or reverse proxies. And is easy to setup using a Raspberry Pi or any other server/device that is always on.
>
> Like the Vaultwarden container that is being set up in this guide.

To make sure that the container is running, you can use the following command: `podman ps`. This will show you a list of all the running containers on your system, and you should see the **vaultwarden** container in the list.

![Podman PS](./podman_ps.png "Screenshot of the output of the `podman ps` command showing the vaultwarden container running")

### Daemonize Vaultwarden

To ensure that the Vaultwarden container starts automatically when the main container is rebooted, you can use the following command to create a systemd service for the Vaultwarden container:

```bash
podman generate systemd --name vaultwarden > /usr/lib/systemd/system/vaultwarden.service
systemctl enable vaultwarden.service
```

There is no need to start the service because the container is already running. Now, if you reboot the main container, the Vaultwarden container will start automatically.

### Keeping It Updated

The best way to keep your Vaultwarden instance updated is to use a combination of **SystenD** and a small bash script that will pull the latest version of the Vaultwarden server image and restart the container. You can create a bash script with the following content:

```bash {filename="/usr/local/bin/update-vaultwarden"}
#!/usr/bin/env bash

# Check for updates
current_image="$(podman exec vaultwarden /vaultwarden --version | grep 'Vaultwarden' | awk '{print $2}'
latest_image="$(curl -s https://api.github.com/repos/dani-garcia/vaultwarden/releases/latest   | jq -r .tag_name)"

if [ "$current_image" != "$latest_image" ]; then
    echo "Updating Vaultwarden from version $current_image to $latest_image"
    podman stop vaultwarden
    podman rm vaultwarden

    # Reinstall the latest version of the Vaultwarden server image
    podman run -d \
        --name vaultwarden \
        --network host \
        -v /vlt/:/data/:Z \
        -e ROCKET_PORT=80 \
        -e DATABASE_URL='mysql://vaultwarden:your_secure_password@127.0.1:3306/vaultwarden' \
        -e ADMIN_TOKEN="$(echo -n 'your_secure_password' | argon2 "$(openssl rand -base64 32)" -e -id -k 65540 -t 3 -p 4)"
        docker.io/vaultwarden/server:latest
else
    echo "Vaultwarden is up to date (version $current_image)"
fi
```

This is a simple script that checks for updates to the Vaultwarden server image by comparing the current version of the image with the latest version available on GitHub. In order to run this script, you will need to have the `jq` command-line tool installed, which is used to parse the JSON response from the GitHub API. You can install it using the following command: `apt install -y jq`.

To run this script automatically, you can create a either a cron job or a systemd timer that runs the script at regular intervals (e.g., once a week). This will ensure that your Vaultwarden instance is always running the latest version of the server image, which includes important security updates and new features.

I prefer using a systemd timer because it is more reliable and easier to manage than cron jobs.

```ini {filename="/usr/lib/systemd/system/vaultwarden-update.timer"}
[Unit]
Description = Vaultwarden Update Service

[Timer]
OnCalendar = daily
Persistent = true

[Install]
WantedBy = timers.target
```

Create the service file for the update script:

```ini {filename="/usr/lib/systemd/system/vaultwarden-update.service"}
[Unit]
Description = Vaultwarden Update Service
Requires = vaultwarden.service

[Service]
Type = oneshot
ExecStart = /usr/local/bin/update-vaultwarden
```

I may end up creating a Debian/Ubuntu package in the future that will automate this process and make it easier for users to keep their Vaultwarden instance up to date without having to manually create systemd services and timers. Seeing as that I like to automate everything that I can so that be a bit more lazy in life when I'm not working my day job.

## Step 6: Access the Admin Panel

Once the container is running, you can access the Vaultwarden web interface by pointing your browser to the domain name <mark>vaultwarden.yourdomain.com/admin</mark> or the IP address of the container followed by <mark>/admin</mark>.

When you logged in you need to make a few changes  so that **Vaultwarden** reports no warning or errors.

1. Under "General Settings" change the "Domain URL" to the domain name or IP address of your Vaultwarden instance (e.g., `https://vaultwarden.yourdomain.com` or `https://192.168.1.100`).
2. Under "Advanced Settings" change the "Cliend IP Header" to <mark>X-Forwarded-For</mark>.

Scroll down to the bottom of the page and click on the "Save" button to save your changes. After you have made these changes, you should see no warnings or errors on the admin panel dashboard.

> [!WARNING]
> The user that has access to the admin panel can do a lot of damage to your Vaultwarden instance if they have malicious intent. So it is important to keep the admin token secure and only share it with trusted individuals.

## Step 7: Create a User Account

![Create User Account](./create-account.png)

The homepage for **Vaultwarden** is the login page, here is where you can create a new user account to access the password manager. To create a new user account, click on the "Create Account" link and fill out the form with your email address and a strong password. Once you have created your account, you can log in to the Vaultwarden web interface and start adding your passwords.

I strongly recommend using a phrase or sentence as your master password with at least 1 uppercase letter, 1 number, and 1 special character. This will make it much harder for attackers to guess your password and gain access to your password manager.

Use a line from a book, movie, or song that you like and modify it to make it unique. For example, if you like the book "The Lord of the Rings", you could use the line "One Ring to rule them all" and modify it to "OneRing2RuleThemAll!" as your master password. But make sure to choose something that is easy for you to remember but hard for others to guess.

## Step 8: Export and Import Passwords

I will not go into detail on how to export passwords from your current password manager and/or browser for that refer to the documentation for your current password manager or browser. However, once you have exported your passwords, you can import them into Vaultwarden using the web interface.

1. Log in to the Vaultwarden web interface and click on the "Settings" button in the top right corner of the page.
2. Click on the "Import Data" tab and select the file that you exported from your current password manager or browser.
3. Click on the "Import" button to import your passwords into Vaultwarden.
4. Once the import is complete, you should see all your passwords in the Vaultwarden web interface.

Now that you have imported your passwords, it is time to install the [Bitwarden](https://bitwarden.com/) browser extension for [Chrome](https://chromewebstore.google.com/detail/bitwarden-password-manage/nngceckbapebfimnlniiiahkandclblb) or [Firefox](https://addons.mozilla.org/en-US/firefox/addon/bitwarden-password-manager/) and log in to the extension using the account you created in Vaultwarden. This will allow you to access your passwords from your browser and use the password generator to create strong, unique passwords for all your online accounts.

Make sure that self hosted is selected in the "Server URL" field of the extension settings and that the URL is set to the domain name or IP address of your Vaultwarden instance (e.g., <mark>https://vaultwarden.yourdomain.com</mark> or <mark>https://your-ip-address</mark>).

> [!INFO]
> **Bitwarden** is also available as a mobile app for [iOS](https://apps.apple.com/app/bitwarden-password-manager/id1137397744) and [Android](https://play.google.com/store/apps/details?id=com.x8bit.bitwarden) which can be used to access your passwords on the go. Just make sure to set the "Server URL" field in the app settings to the domain name or IP address of your Vaultwarden instance.
>
> Follow the same steps as above to log in to the app using the account you created in Vaultwarden and you should see all your passwords in the app as well.

## Conclusion

Hosting your own password manager using Vaultwarden is a great way to take control of your data and protect your privacy. By following the steps outlined in this guide, you can set up Vaultwarden using a Proxmox LXC container and start securely storing your passwords today. Remember to keep your admin token secure and only share it with trusted individuals, and to use a strong master password to protect your password manager from unauthorized access.
