# Comprehensive Guide to Setting Up a New VPS

Setting up a new Virtual Private Server (VPS) can be a daunting task, especially for beginners. This guide will walk you through the process of configuring a secure and functional VPS, including firewall setup, swap space creation, and Docker installation. We'll be using a DigitalOcean droplet running Ubuntu 24.04 (LTS) as an example, but most steps should be applicable to other VPS providers and recent Ubuntu versions.

## VPS Specifications

For this guide, we're using the following VPS configuration:

- Provider: DigitalOcean
- Name: newyork
- Specs: 1 GB Memory / 1 Intel vCPU / 25 GB Disk
- Location: NYC3
- OS: Ubuntu 24.04 (LTS) x64

## Initial Security Setup

### 1. Update System Packages

Always start by updating your system:

```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Configure Firewall (UFW)

Ubuntu comes with UFW (Uncomplicated Firewall) which we'll use to secure our VPS:

```bash
sudo ufw status
sudo ufw allow OpenSSH
sudo ufw enable
```

When enabling UFW, you'll see a warning about disrupting existing SSH connections. Type 'y' to proceed.

To add rate limiting to SSH connections (helps prevent brute-force attacks):

```bash
sudo ufw limit OpenSSH
```

This limits connection attempts to 6 per 30 seconds from the same IP address.

### 3. Create a Swap File

Even with 1GB of RAM, it's a good idea to create some swap space:

```bash
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

To make the swap file permanent, add it to `/etc/fstab`:

```bash
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

Verify the swap is active:

```bash
sudo swapon --show
free -h
```


## 4. Installing Docker

Now, let's install Docker using the standalone method:

### 1. Download Docker Compose standalone

```bash
sudo curl -SL https://github.com/docker/compose/releases/download/v2.29.6/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
```

### 2. Apply executable permissions

```bash
sudo chmod +x /usr/local/bin/docker-compose
```

### 3. Create a symbolic link (optional)

This step ensures that `docker-compose` is accessible from anywhere:

```bash
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

### 4. Verify the installation

```bash
docker-compose --version
```

You should see the version of Docker Compose you installed.

### 5. Install Docker Engine

Install the latest version of Docker Engine, containerd, and Docker Compose:

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### 4. Verify the installation

Run the hello-world image to check if Docker Engine is installed correctly:

```bash
sudo docker run hello-world
```

If successful, you'll see a welcome message from Docker.

### 6. Configure Docker to start on boot

To automatically start Docker and containerd on boot for Ubuntu 22.04 and higher:

```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

### 7. Manage Docker as a non-root user (optional)

To run Docker commands without sudo, add your user to the docker group:

```bash
sudo usermod -aG docker $USER
```

Log out and back in for this to take effect.

## Setting Up SSH Key Authentication

Using SSH keys for authentication is more secure than using passwords. Here's how to set it up:

### 1. Generate a new SSH key pair

On your local machine (not the VPS), run:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

Replace "your_email@example.com" with your actual email address.

### 2. Add the SSH key to the ssh-agent

Start the ssh-agent in the background:

```bash
eval "$(ssh-agent -s)"
```

Add your SSH private key to the ssh-agent:

```bash
ssh-add ~/.ssh/id_ed25519
```

### 3. Copy the public key

Display your public key and copy it to your clipboard:

```bash
cat ~/.ssh/id_ed25519.pub
```

### 4. Add the SSH key to your VPS

SSH into your VPS and add the public key to the authorized_keys file:

```bash
mkdir -p ~/.ssh
echo "your-public-key-here" >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

Replace "your-public-key-here" with the public key you copied in step 3.

### 5. Configure SSH to use key authentication

Edit the SSH configuration file:

```bash
sudo nano /etc/ssh/sshd_config
```

Ensure the following lines are present and uncommented:

```
PubkeyAuthentication yes
PasswordAuthentication no
```

Save the file and exit the editor.

### 6. Restart the SSH service

```bash
sudo systemctl restart ssh
```

Now you should be able to SSH into your VPS using your SSH key instead of a password.

[The rest of the content remains the same]


## Conclusion

You now have a secure, updated VPS with UFW configured, swap space added, and Docker installed. This setup provides a solid foundation for hosting various applications and services. Remember to keep your system updated regularly and monitor your firewall rules to maintain security.

As you continue to work with your VPS, you might want to explore additional topics such as setting up automatic security updates, configuring SSH key authentication, and deploying your first Docker containers. Happy server administrating!

================

# Creating a Non-Root User on Ubuntu and Checking SSH Setup

## Step 1: Create the New User

1. Log into your server as the root user.

2. Use the `adduser` command to create a new user. Replace `newuser` with your desired username:

   ```bash
   adduser newuser
   ```

3. You'll be prompted to set and confirm a password for the new user. Choose a strong, unique password.

4. Fill in any additional information if prompted (full name, phone, etc.). You can press Enter to skip these fields.

## Step 2: Grant Sudo Privileges

To allow your new user to perform administrative tasks, you need to grant them sudo privileges:

1. Add the new user to the sudo group:

   ```bash
   usermod -aG sudo newuser
   ```

## Step 3: Check Existing SSH Setup

Before making any changes to the SSH configuration, let's check your existing setup:

1. Check if you're currently using SSH key authentication:

   ```bash
   grep -i pubkeyauthentication /etc/ssh/sshd_config
   ```

   If you see `PubkeyAuthentication yes`, SSH key authentication is enabled.

2. Check for existing authorized keys:

   ```bash
   ls -la ~/.ssh/authorized_keys
   ```

   If this file exists, you're already using SSH keys.

3. View the contents of your authorized_keys file:

   ```bash
   cat ~/.ssh/authorized_keys
   ```

   This will show you any existing SSH public keys.

4. Check if root SSH login is currently allowed:

   ```bash
   grep -i permitrootlogin /etc/ssh/sshd_config
   ```

   If you see `PermitRootLogin yes`, root SSH login is currently allowed.

## Next Steps

Based on the results of these checks, you can decide how to proceed:

- If you're already using SSH keys and want to add a key for your new user, you can do so without overwriting existing keys.
- If root SSH login is allowed and you want to disable it, you can do so, but make sure you have an alternative way to access your server first.
- If you're not using SSH keys yet, you might want to set them up for enhanced security.

Remember, it's crucial to ensure you don't lock yourself out of your server. Always keep a backup method of access (like the DigitalOcean console) available when making changes to SSH configurations.


