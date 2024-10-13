---
title: "Keeping SSH Connections Persistent A Guide to Avoiding Timeouts"
date: 2023-12-22T14:32:16+03:00
draft: false
description: "Avoiding timeouts"
---

If you're running a VPS like I am, ensuring that your SSH connection stays active without timing out is crucial for smooth remote access. I recently faced a timeout issue with my DigitalOcean VPS running Ubuntu 24.04 and sought to make my SSH connection more persistent. This blog explains the entire process, from configuring your local machine (client) to modifying server settings. I'll also dive into the details of the `grep` command and the significance of the `240 60` settings in SSH configuration.

---

#### **What Causes SSH Timeouts?**

When you're connected to a server via SSH (Secure Shell), the connection can sometimes drop after periods of inactivity. This happens because, by default, the SSH connection closes if there's no communication between the client (your local machine) and the server for a set amount of time. To prevent this, we can configure "keep-alive" messages that regularly ping the server or client, keeping the connection alive even when there's no active input from you.

---

### **Step 1: Modifying Client-Side SSH Configuration**

We can configure your **local machine** to send periodic "keep-alive" signals to the server. This ensures that even if you're not actively using the terminal, the connection doesn't get dropped.

#### **Configuring SSH on the Local Machine (Client-Side)**

On your local machine (the computer you're using to SSH into the server), you can modify the SSH configuration to send "keep-alive" signals regularly.

1. **Open or Create the SSH Config File**:

   In most Linux and macOS systems, this file is located at `~/.ssh/config`. Open it with a text editor:

   ```bash
   nano ~/.ssh/config
   ```

   If the file doesn’t exist, create it.

2. **Add the Keep-Alive Settings**:

   In this file, add the following lines to ensure the SSH client sends a "keep-alive" signal to the server every 60 seconds:

   ```bash
   Host *
       ServerAliveInterval 60
       ServerAliveCountMax 240
   ```

   **Explanation**:
   - **ServerAliveInterval 60**: This tells your local machine to send a "keep-alive" signal to the server every 60 seconds.
   - **ServerAliveCountMax 240**: This allows the SSH connection to be kept alive for up to 240 intervals (which equals 240 minutes or 4 hours of inactivity) before timing out.

3. **Using the SSH Command Directly**:

   Alternatively, you can apply this keep-alive setting directly when using the `ssh` command without editing the config file:

   ```bash
   ssh -o ServerAliveInterval=60 user@your_vps_ip
   ```

---

### **Step 2: Modifying Server-Side SSH Configuration**

The server you're connecting to (your VPS) also has SSH timeout settings. To make sure the server doesn't kill the connection, you need to adjust its settings as well.

#### **Configuring SSH on the Server-Side (VPS)**

1. **Edit the SSHD Config File**:

   Connect to your VPS and open the SSH daemon configuration file (`sshd_config`) using a text editor like `nano`:

   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. **Add Keep-Alive Settings on the Server**:

   Add the following lines to the file to prevent the server from closing inactive SSH sessions:

   ```bash
   ClientAliveInterval 60
   ClientAliveCountMax 240
   ```

   **Explanation**:
   - **ClientAliveInterval 60**: The server will send a "keep-alive" message to the client every 60 seconds.
   - **ClientAliveCountMax 240**: The server will keep the connection alive for 240 intervals (or 240 minutes) of inactivity before it times out.

3. **Restart the SSH Service**:

   After modifying the `sshd_config`, you need to restart the SSH service for the changes to take effect. Since I'm running Ubuntu 24.04 on a DigitalOcean VPS, the SSH service name is `ssh` instead of the more common `sshd`. Here's how I discovered and restarted it:

   - First, I tried restarting `sshd`, but got an error:

     ```bash
     root@newyork:~# sudo systemctl restart sshd
     Failed to restart sshd.service: Unit sshd.service not found.
     ```

     This indicated that `sshd` isn't the correct service name on my system.

   - To find the correct service name, I used the following command:

     ```bash
     sudo systemctl list-units --type=service | grep ssh
     ```

     Output:
     ```bash
     ssh.service                                    loaded active running OpenBSD Secure Shell server
     ```

     Here, `ssh.service` is listed as the running SSH service, which is what I needed to restart.

   - I then restarted the correct service:

     ```bash
     sudo systemctl restart ssh
     ```

---

### **Understanding the Commands**

#### **What is `ServerAliveInterval` and `ClientAliveInterval`?**

- **ServerAliveInterval**: This is a setting on the client side (your local machine). It controls how often the client sends a message to the server to keep the connection alive. Setting this to `60` means the client sends a message every 60 seconds to check if the server is still there.

- **ClientAliveInterval**: This is a setting on the server side (your VPS). It controls how often the server sends a message to the client to keep the connection alive. If no response is received within the specified time (240 minutes in this case), the server will close the connection.

These intervals work in tandem to ensure both the client and server maintain communication even during periods of inactivity.

#### **What is `grep` and Why Did I Use It?**

`grep` is a powerful search utility in Unix/Linux that allows you to search for specific patterns within text. I used `grep` to search for the SSH service name within the list of active services on my server.

```bash
sudo systemctl list-units --type=service | grep ssh
```

In this case:
- `systemctl list-units --type=service`: Lists all the active services on the server.
- `| grep ssh`: Pipes the output of the previous command to `grep`, which filters and displays only lines that contain the word "ssh."

This helped me quickly find the correct service name (`ssh.service`) for SSH on my server.

---

### **Final Thoughts**

By making these changes on both the client and server sides, I was able to keep my SSH connection alive for up to 240 minutes of inactivity. This is useful when you're working remotely on a server but don't want to lose your connection due to temporary inactivity. Whether you're working on a long-running script or managing multiple sessions, these settings ensure your connection stays stable.

If you ever find yourself facing similar timeout issues, following these steps will help ensure that your SSH connections are persistent, reducing the frustration of dropped sessions.
