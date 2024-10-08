# odoo17-GCP-insallition

step-by-step guide for installing **Odoo 17** on **Google Cloud**, using a non-root user (`odoo17`) to handle the installation:

### Step 1: Set up a Google Cloud Account
1. Go to [Google Cloud Console](https://console.cloud.google.com).
2. If you don’t have an account, sign up for a free trial (GCP offers $300 in free credits for 90 days).

### Step 2: Create a Virtual Machine (VM) on GCP
1. In the **Google Cloud Console**, go to the **Compute Engine** section.
   - Navigate to **Compute Engine** > **VM instances**.
2. Click on **Create Instance**.
3. Configure the instance:
   - **Name**: Choose a name for your instance (e.g., `odoo-17-server`).
   - **Region**: Choose a region that is closest to your location.
   - **Machine Type**: Choose a machine type (e.g., `n1-standard-1` for small to medium workloads).
   - **Boot Disk**: 
     - Click on **Change** to select **Ubuntu 22.04 LTS** as the operating system.
     - Set the boot disk size to **20GB or more**, depending on your requirements.
   - **Firewall**: Enable **HTTP** and **HTTPS** traffic.
4. Click **Create** to launch your VM.

### Step 3: Connect to the VM
1. After your VM is created, click **SSH** to connect to the instance.
2. You’ll now have terminal access to the server.

### Step 4: Create a Non-Root User for Odoo
1. Create a new user called `odoo17`:

```bash
sudo adduser odoo17
```

2. Give the user `odoo17` sudo privileges:

```bash
sudo usermod -aG sudo odoo17
```

3. Switch to the new user:

```bash
su - odoo17
```

### Step 5: Update the System and Install Required Packages
Run the following commands to update your server and install dependencies for Odoo:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install git wget python3-pip build-essential libssl-dev libffi-dev python3-dev python3-venv libpq-dev libjpeg-dev libxml2-dev libxslt1-dev zlib1g-dev libsasl2-dev libldap2-dev libtiff5-dev libopenjp2-7-dev liblcms2-dev libblas-dev libatlas-base-dev libjpeg8-dev nodejs npm -y
```

### Step 6: Install PostgreSQL
Odoo requires **PostgreSQL** as the database:

```bash
sudo apt install postgresql postgresql-server-dev-all -y
```

Once installed, start PostgreSQL and create a user for Odoo:

```bash
sudo service postgresql start
sudo su - postgres
createuser --createdb --username postgres --no-createrole --no-superuser --pwprompt odoo17
# Enter a password for the odoo17 user
```
Exit the PostgreSQL shell:
```bash
exit
```

### Step 7: Install Wkhtmltopdf (for PDF reports)
Odoo requires `wkhtmltopdf` for PDF generation:

```bash
sudo apt install xfonts-75dpi xfonts-base -y
wget https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.6/wkhtmltox_0.12.6-1.bionic_amd64.deb
sudo dpkg -i wkhtmltox_0.12.6-1.bionic_amd64.deb
sudo apt --fix-broken install -y
```

### Step 8: Install Odoo 17
1. Download the **Odoo 17** source code to the `/opt` directory, using the non-root `odoo17` user:

```bash
sudo mkdir /opt/odoo17
sudo chown odoo17:odoo17 /opt/odoo17
cd /opt/odoo17
git clone https://github.com/odoo/odoo.git --depth 1 --branch 17.0 .
```

2. Create a Python virtual environment and install dependencies:

```bash
python3 -m venv odoo-venv
source odoo-venv/bin/activate
pip3 install wheel
pip3 install -r requirements.txt
deactivate
```

### Step 9: Configure Odoo
1. Create a configuration file for Odoo:

```bash
sudo nano /etc/odoo.conf
```

2. Edit the configuration file as follows:

```ini
[options]
   ; admin password
   admin_passwd = superadminpassword
   db_host = False
   db_port = False
   db_user = odoo17
   db_password = your-db-password
   addons_path = /opt/odoo17/addons
   logfile = /var/log/odoo/odoo.log
```

3. Save the file and exit.

### Step 10: Create a Systemd Service for Odoo
1. Create a systemd service file for Odoo:

```bash
sudo nano /etc/systemd/system/odoo17.service
```

2. Add the following content:

```ini
[Unit]
Description=Odoo17
Documentation=http://www.odoo.com
[Service]
# Ubuntu/Debian convention:
Type=simple
User=odoo17
ExecStart=/opt/odoo17/odoo-venv/bin/python3 /opt/odoo17/odoo-bin -c /etc/odoo.conf
[Install]
WantedBy=default.target
```

3. Reload systemd and enable the Odoo service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable odoo17.service
sudo systemctl start odoo17.service
```

### Step 11: Access Odoo 17
1. Open your browser and navigate to your VM's external IP (found in your GCP **Compute Engine** dashboard):
   ```
   http://YOUR_VM_EXTERNAL_IP:8069
   ```
2. You should now see the **Odoo 17** setup page.

3. Create a database and start using Odoo!

### Step 12: Configure Firewall (Optional)
To ensure that Odoo is accessible from the web, check the **firewall rules** in your GCP console:
   - Go to **VPC Network** > **Firewall** rules.
   - Make sure port **8069** (Odoo’s default port) is allowed for inbound traffic.

---

That's it! Now, Odoo 17 is installed using a non-root user (`odoo17`), ensuring better security and management.

---


To add **PostGIS** (a spatial extension for PostgreSQL) to your PostgreSQL installation, you can follow these steps. We'll assume you're still using the **non-root user `odoo17`** and are connected to your VM where PostgreSQL is installed.

### Step 1: Install PostGIS
First, you need to install the PostGIS extension for PostgreSQL:

1. **Update package lists and install PostGIS**:
   ```bash
   sudo apt update
   sudo apt install postgis postgresql-14-postgis-3 -y
   ```

2. **Verify the installation**:
   After installation, you can check if the `postgis` package is available by running:
   ```bash
   psql --version
   ```

### Step 2: Enable PostGIS on a PostgreSQL Database
Now that PostGIS is installed, you need to enable the extension on your PostgreSQL database.

1. **Switch to the PostgreSQL user**:
   ```bash
   sudo -i -u postgres
   ```

2. **Access the PostgreSQL shell**:
   ```bash
   psql
   ```

3. **Connect to your Odoo database**:
   - Replace `odoo_database` with the name of your Odoo database.
   ```bash
   \c odoo_database
   ```

4. **Enable the PostGIS extension**:
   ```bash
   CREATE EXTENSION postgis;
   CREATE EXTENSION postgis_topology;
   ```

5. **Verify PostGIS installation**:
   You can verify that PostGIS is installed correctly by running:
   ```bash
   SELECT PostGIS_Version();
   ```

6. **Exit PostgreSQL**:
   ```bash
   \q
   ```

7. **Return to the odoo17 user**:
   ```bash
   exit
   ```

### Step 3: Using PostGIS with Odoo
To integrate **PostGIS** with Odoo, you can:

- Develop custom Odoo modules that use spatial data stored in PostGIS.
- Use PostGIS functions for spatial queries, for example, integrating geographical information into project management.

---

You now have **PostGIS** installed and enabled on your PostgreSQL database, ready for use in spatial queries and GIS functionalities.

---

To install **Redis** on **Google Cloud Platform (GCP)** and configure it to work with **Odoo 17**, you can follow these steps. We will assume you're using a **non-root user (`odoo17`)** and have an active virtual machine (VM) instance on GCP where Odoo 17 is installed.

### Step 1: Connect to Your Google Cloud VM
1. Go to the **Google Cloud Console**.
2. Navigate to **Compute Engine** > **VM instances**.
3. Click **SSH** next to your VM to open a terminal session.

### Step 2: Install Redis on the VM
1. **Update the package list**:
   ```bash
   sudo apt update
   ```

2. **Install Redis**:
   ```bash
   sudo apt install redis-server -y
   ```

3. **Verify that Redis is installed**:
   After the installation is complete, verify it by checking the version:
   ```bash
   redis-server --version
   ```

### Step 3: Configure Redis for Odoo
1. **Edit the Redis configuration file** to improve performance and security for Odoo. Open the Redis configuration file:
   ```bash
   sudo nano /etc/redis/redis.conf
   ```

2. In the configuration file, make the following changes:

   - **Set a password** for Redis for better security:
     Find the line:
     ```ini
     # requirepass foobared
     ```
     Uncomment the line and set a strong password:
     ```ini
     requirepass your_redis_password
     ```

   - **Set `maxmemory`** to limit the maximum amount of memory Redis can use. For example, to allow Redis to use up to 1 GB of memory, find and set:
     ```ini
     maxmemory 1gb
     ```

   - **Set the eviction policy** to determine what Redis should do when the `maxmemory` limit is reached. To evict the least recently used keys, set the following:
     ```ini
     maxmemory-policy allkeys-lru
     ```

3. Save and exit the file: Press `CTRL+X`, then `Y`, and press `Enter`.

4. **Restart Redis** for the changes to take effect:
   ```bash
   sudo systemctl restart redis-server
   ```

5. **Enable Redis to start on boot**:
   ```bash
   sudo systemctl enable redis-server
   ```

### Step 4: Configure Odoo to Use Redis
1. Switch to the `odoo17` user (if not already):
   ```bash
   su - odoo17
   ```

2. **Open the Odoo configuration file** to enable Redis caching:
   ```bash
   sudo nano /etc/odoo.conf
   ```

3. Add the following lines to configure Redis for session storage and caching:

   ```ini
   [options]
   # Redis settings
   proxy_mode = True
   dbfilter = odoo17

   # Session and Cache Settings
   longpolling_port = 8072
   # Redis session storage
   redis_host = 127.0.0.1
   redis_port = 6379
   redis_password = your_redis_password
   ```

4. Save and exit the file.

### Step 5: Restart Odoo to Apply Changes
1. **Restart the Odoo service** so that the changes take effect:
   ```bash
   sudo systemctl restart odoo17.service
   ```

### Step 6: Verify Redis is Working with Odoo
You can verify if Odoo is using Redis by checking the Redis stats using the following command:

```bash
redis-cli -a your_redis_password info stats
```

You should see activity on Redis, indicating it's being used for caching and session management by Odoo.

### Step 7: Open Redis Port (Optional)
By default, Redis listens on **localhost** (127.0.0.1), which is fine for most setups. If you want to allow external access (e.g., from another server), you would need to adjust the Redis configuration to bind to an external IP and open the port in your GCP firewall.

To allow external access to Redis:
1. Edit `/etc/redis/redis.conf` and change the `bind` address:
   ```ini
   bind 0.0.0.0
   ```
2. Add a firewall rule on GCP to open port `6379` (Redis default port):
   - Go to **VPC Network** > **Firewall rules**.
   - Create a new firewall rule to allow traffic on port `6379`.

However, opening Redis to the internet is generally **not recommended** due to security risks. Use internal connections or VPNs if possible.

---

That's it! Redis is now installed and configured to work with your **Odoo 17** instance on Google Cloud.
