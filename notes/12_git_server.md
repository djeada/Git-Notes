## Git Server

Setting up your own Git server provides several benefits compared to relying on hosted platforms:

- **Increased Control**: You decide where your repositories are stored and how they are accessed.
- **Enhanced Security**: Limit outside access and define strict access policies on your own server.
- **Customization**: Integrate your repositories with custom deployment scripts or CI/CD pipelines tailored to your workflow.
- **Scalability**: Grow your server as your number of projects or developers increases.

Below is a high-level diagram of how various Git clients interact with a central bare repository:

```
+-------------------+              +-------------------+
|   Git Client 1    |              |   Git Client 2    |
| - Clone           |              | - Push            |
| - Pull            |              | - Pull Requests   |
| - Push            |              | - Merge           |
+--------+----------+              +----------+--------+
         |                                   |
         |                                   |
         +----+----------------------+-------+
              |                      |
              |    Git Operations    |
              |                      |
         +----v----------------------v----+
         |                                |
         |       Git Server (Bare)        |
         | - Central Repository Storage   |
         | - Access Control               |
         | - Version History Management   |
         | - Branches & Tags Handling     |
         |                                |
         +--------------------------------+
```

## Prerequisites

I. **Debian-based OS**: Have a Debian-based operating system installed (e.g., Debian, Ubuntu).  

II. **SSH Server**: Ensure `ssh` (OpenSSH) is installed and running so you can securely connect and manage the server remotely.  

III. **Administrative Access**: Have root or sudo privileges for installing software and configuring the system.  

## 1. Install Git

I. **Update Package Lists**  

   ```bash
   sudo apt update
   ```
II. **Install Git**  
   ```bash
   sudo apt install git-core
   ```

This installs the latest version of Git available in your distribution’s package repositories.

## 2. Set Up a Bare Repository

A bare repository is a central repository that **does not** contain a working directory (i.e., no files you can directly edit). Instead, it stores the Git version control data, making it ideal for sharing among multiple developers or for deployment processes.

### 2.1 Create a Directory for Your Repositories

Decide on a location for your repositories. Below, we’re using `/opt/git/` for organizational purposes:

```bash
sudo mkdir -p /opt/git/myrepo.git
```
### 2.2 Navigate into the Directory

```bash
cd /opt/git/myrepo.git
```

### 2.3 Initialize a Bare Repository

```bash
sudo git init --bare
```
Your new bare repository is now located at `/opt/git/myrepo.git`. Other team members (or even you on another machine) can clone and push to this remote repository.


## 3. Configure User Access

### 3.1 Create a Dedicated Git User

For enhanced security and simplified access management, set up a dedicated `git` user:

```bash
sudo adduser git
```

Follow the prompts to configure the new user’s details.

### 3.2 Set a Password for the Git User

```bash
sudo passwd git
```
You can optionally skip setting a password if you plan to rely solely on SSH keys.

### 3.3 Configure SSH for the Git User

To allow key-based SSH authentication, switch to the `git` user and set up their `.ssh` directory:

```bash
sudo su git
mkdir ~/.ssh
chmod 700 ~/.ssh
touch ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

- **`mkdir ~/.ssh`**: Creates a directory to store SSH config and authorized keys.  
- **`chmod 700 ~/.ssh`**: Grants read, write, and execute permissions to the `git` user only.  
- **`touch ~/.ssh/authorized_keys`**: Creates the file that will store public keys.  
- **`chmod 600 ~/.ssh/authorized_keys`**: Ensures that only the `git` user can read and write to this file.

### 3.4 Authorize Additional Users

To grant another developer access to this Git server, append their **public SSH key** to the `authorized_keys` file:

```bash
echo "public_key_content" >> ~/.ssh/authorized_keys
```
Replace `public_key_content` with the actual public key string (e.g., the contents of the user’s `id_rsa.pub` file). From now on, these users can authenticate as `git` via SSH.


## 4. Using the Repository

With the server ready, developers can clone and work with the repository:

```bash
git clone git@yourserver:/opt/git/myrepo.git
```

- **`git@yourserver`**: Replace `yourserver` with the hostname or IP address of your Git server.  
- **`/opt/git/myrepo.git`**: The path to the bare repository you created.

Once cloned, they can **pull** and **push** changes:

```bash

# Pull latest changes
git pull

# Stage changes, commit, and push
git add .
git commit -m "Update project files"
git push
```

## 5. Additional Tips for a Robust Git Server

I. **GitWeb or cgit**  
   - For a user-friendly web interface to browse and manage repositories.  
II. **Repository Backups**  
   - Regularly back up `/opt/git` or wherever your repositories reside. Tools like `rsync` or `tar` can help automate this.  
III. **Git Hooks**  
   - Customize repository behavior with hooks (e.g., post-receive hooks for deployments).  
IV. **Continuous Integration / Continuous Deployment (CI/CD)**  
   - Integrate Jenkins, GitLab CI, or GitHub Actions (self-hosted runners) for automated builds, tests, or deployments.  
V. **Monitoring & Logging**  
   - Implement system monitoring (e.g., `htop`, `nagios`, or `prometheus`) to track server resource usage.  
   - Check system logs for suspicious activity or errors.  
VI. **Server Security**  
   - Keep your OS up-to-date with security patches.  
   - Configure a firewall (e.g., `ufw` or `iptables`) to restrict unwanted inbound connections.  
   - Use SSH key-based authentication instead of passwords whenever possible.  
VII. **Keep Git Up to Date**  
   - Regularly update Git to ensure you have the latest security patches and features.  
