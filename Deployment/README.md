Sure! Here’s a general guide on how to deploy your high-availability load balancer on Linode. We will be   customizing it based on  specific setup as we progress.

---

## Deployment Guide on Linode Cloud

### Context/Deployment Figure

https://drive.google.com/file/d/1cjuj35N0M4ikAUnN-izK_VbMaflFMnzG/view?usp=drive_link [INPROGRESS]

![image](https://github.com/user-attachments/assets/dd306968-6fc5-48dd-a044-2e672ab005d7)



### Step 1: Create Linode Instances

1. **Log in to your Linode account**.
2. **Create two Linode instances** for the load balancer (Primary and Secondary):
   - Go to the **Linodes** section and click on **Create Linode**.
   - Choose a distribution (e.g., Ubuntu 22.04 LTS).
   - Select a suitable plan (ensure it can handle the required traffic).
   - Choose a region that best suits your target audience.
   - Enable backups if desired.

### Step 2: Set Up Networking

1. **Assign static IPs**:
   - Ensure that both Linodes have static IP addresses.
   - Go to the **Networking** tab of each Linode to configure IP settings.

2. **Configure a public IP for the load balancer**:
   - Optionally, you can set up a separate public IP that points to both load balancers using Keepalived.

### Step 3: Install Required Software

1. **SSH into each Linode instance**:
   ```bash
   ssh root@[your-linode-ip]
   ```

2. **Update the package manager**:
   ```bash
   apt update && apt upgrade -y
   ```

3. **Install HAProxy**:
   ```bash
   apt install haproxy -y
   ```

4. **Install Keepalived**:
   ```bash
   apt install keepalived -y
   ```

5. **Install Certbot for SSL**:
   ```bash
   apt install certbot python3-certbot-nginx -y
   ```

### Step 4: Configure HAProxy

1. **Edit the HAProxy configuration file**:
   ```bash
   nano /etc/haproxy/haproxy.cfg
   ```

   Add the following configuration (customize as needed):

   ```plaintext
   frontend https_front
       bind *:443 ssl crt /etc/ssl/certs/your_cert.pem
       mode http
       redirect scheme https if !{ ssl_fc }
       default_backend app_servers

   backend app_servers
       mode http
       balance roundrobin
       server app1 [app_server_ip]:[port] check
       server app2 [app_server_ip]:[port] check
   ```

2. **Enable and start HAProxy**:
   ```bash
   systemctl enable haproxy
   systemctl start haproxy
   ```

### Step 5: Configure Keepalived

1. **Edit the Keepalived configuration file**:
   ```bash
   nano /etc/keepalived/keepalived.conf
   ```

   Add the following configuration (customize VIP and priorities):

   ```plaintext
   vrrp_instance VI_1 {
       state MASTER
       interface eth0
       virtual_router_id 51
       priority 101
       advert_int 1
       authentication {
           auth_type PASS
           auth_pass your_password
       }
       virtual_ipaddress {
           [your_virtual_ip]
       }
   }
   ```

   On the secondary load balancer, change `state MASTER` to `state BACKUP` and adjust the `priority`.

2. **Enable and start Keepalived**:
   ```bash
   systemctl enable keepalived
   systemctl start keepalived
   ```

### Step 6: Set Up SSL with Certbot

1. **Obtain an SSL certificate**:
   ```bash
   certbot certonly --standalone -d yourdomain.com
   ```

2. **Automate certificate renewal** (this is usually handled by Certbot automatically):
   ```bash
   echo "0 0 * * * root certbot renew --quiet" >> /etc/crontab
   ```

### Step 7: Configure Firewall

1. **Allow necessary ports** (adjust according to your firewall solution):
   ```bash
   ufw allow 80
   ufw allow 443
   ufw enable
   ```

### Step 8: Verify Deployment

1. **Check the status of HAProxy and Keepalived**:
   ```bash
   systemctl status haproxy
   systemctl status keepalived
   ```

2. **Access your load balancer** using the virtual IP in a web browser to ensure everything is working.

---

