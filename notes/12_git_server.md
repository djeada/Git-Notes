## Git Server

Setting up your own Git server provides several benefits compared to relying on hosted platforms:

- Hosting your own server provides **increased control** over repository storage locations and access configurations, allowing full ownership of your data.
- Self-hosting enables **enhanced security** by limiting outside access and allowing you to define and enforce strict access policies directly on your server.
- The ability to customize allows you to **integrate repositories** with deployment scripts or CI/CD pipelines tailored to fit your specific workflow needs.
- Self-hosted servers offer **scalability**, giving you the flexibility to expand resources as your number of projects or team size grows.

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

### Prerequisites

- A **Debian-based operating system** such as Debian or Ubuntu should be installed on the server.
- An **SSH server** like OpenSSH must be installed and running to enable secure remote connections and management.
- **Administrative access** with root or sudo privileges is required for installing necessary software and configuring the system.

### Install Git

I. **Update Package Lists**  

```bash
sudo apt update
```

II. **Install Git**  

```bash
sudo apt install git-core
```

This installs the latest version of Git available in your distribution’s package repositories.

### Set Up a Bare Repository

A bare repository is a central repository that **does not** contain a working directory (i.e., no files you can directly edit). Instead, it stores the Git version control data, making it ideal for sharing among multiple developers or for deployment processes.

#### Create a Directory for Your Repositories

Decide on a location for your repositories. Below, we’re using `/opt/git/` for organizational purposes:

```bash
sudo mkdir -p /opt/git/myrepo.git
```

#### Navigate into the Directory

```bash
cd /opt/git/myrepo.git
```

#### Initialize a Bare Repository

```bash
sudo git init --bare
```

Your new bare repository is now located at `/opt/git/myrepo.git`. Other team members (or even you on another machine) can clone and push to this remote repository.

### Configure User Access

#### Create a Dedicated Git User

For enhanced security and simplified access management, set up a dedicated `git` user:

```bash
sudo adduser git
```

Follow the prompts to configure the new user’s details.

#### Set a Password for the Git User

```bash
sudo passwd git
```
You can optionally skip setting a password if you plan to rely solely on SSH keys.

#### Configure SSH for the Git User

To allow key-based SSH authentication, switch to the `git` user and set up their `.ssh` directory:

```bash
sudo su git
mkdir ~/.ssh
chmod 700 ~/.ssh
touch ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

- The command **`mkdir ~/.ssh`** creates a directory to store SSH configuration files and authorized keys for the user.
- Using **`chmod 700 ~/.ssh`** sets permissions so that only the owner has read, write, and execute access to the `.ssh` directory.
- The **`touch ~/.ssh/authorized_keys`** command creates an empty file to store public keys for granting SSH access.
- Applying **`chmod 600 ~/.ssh/authorized_keys`** restricts the file's permissions so that only the user can read and write to it, ensuring secure access control.

#### Authorize Additional Users

To grant another developer access to this Git server, append their **public SSH key** to the `authorized_keys` file:

```bash
echo "public_key_content" >> ~/.ssh/authorized_keys
```

Replace `public_key_content` with the actual public key string (e.g., the contents of the user’s `id_rsa.pub` file). From now on, these users can authenticate as `git` via SSH.

### Using the Repository

With the server ready, developers can clone and work with the repository:

```bash
git clone git@yourserver:/opt/git/myrepo.git
```

- Replace **`yourserver`** with the hostname or IP address of your Git server when setting up SSH connections or accessing the repository.
- Specify **`/opt/git/myrepo.git`** as the path to the bare repository you created on the server for cloning, pushing, or pulling operations.

Once cloned, they can **pull** and **push** changes:

```bash
# Pull latest changes
git pull

# Stage changes, commit, and push
git add .
git commit -m "Update project files"
git push
```

### Additional Tips for a Robust Git Server

- GitWeb or cgit can provide a **user-friendly web interface** for browsing and managing repositories efficiently.
- Regularly backing up the repository directory, such as `/opt/git`, using **tools like rsync or tar** ensures data recovery in case of failures.
- Git hooks allow you to **customize repository behavior**, such as implementing post-receive hooks for automated deployments or notifications.
- Continuous Integration and Deployment (CI/CD) can be implemented using **tools like Jenkins, GitLab CI, or GitHub Actions**, which support automated testing, building, and deployment processes.
- Monitoring server performance with **tools like htop, nagios, or prometheus** helps track resource usage and identify potential bottlenecks.
- Reviewing system logs helps detect **suspicious activity or errors** that may affect server operations.
- Keeping the operating system updated ensures it has the **latest security patches**, reducing vulnerabilities.
- Using a firewall such as **ufw or iptables** helps restrict unnecessary inbound connections to improve server security.
- SSH key-based authentication is generally **more secure than passwords** and is recommended for accessing the Git server.
- Regularly updating Git ensures you have access to **the latest features and security fixes**, maintaining compatibility and reliability.

