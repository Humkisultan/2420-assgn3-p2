# Assignment 3 part 2

In this part, we are dealing with setting up multi-server setup.


### Task 1: Create DigitalOcean Droplets

1. Create two DigitalOcean droplets running Arch Linux. Assign the tag `web` to both.
2. Verify the droplets are running in the same region (SFO3) and VPC.

---

### Task 2: Create a Load Balancer

1. Create a load balancer in DigitalOcean:
   - Region: SFO3 (same as droplets).
   - VPC: Default (same as droplets).
   - Type: Public (external).
   - Use the `web` tag to balance traffic between both droplets.

### Task 3: Clone Updated Starter Code 

1. Clone the updated starter repository in local machine:
   ```bash
   git clone <starter_code_repository>
   cd <repository>
   ```
2. Use sftp to copy the file from the local directory to the droplet.

    ```bash
    sftp> put <<path to file>>
    ```

### Task 4: Update Server Configuration

1. Create sample files for the file server:
   ```bash
   sudo touch file-one
   sudo touch file-two
   ```
2. Update the Nginx configuration to include the `/documents` file server.

    ```bash
        server {
            listen 80;
            server_name _;

            root /var/lib/webgen;
            index index.html;

            location /documents {
                alias /var/lib/webgen/documents;
                autoindex on; # Enable directory listing
            }

            location / {
                try_files $uri $uri/ =404;
            }
        }
    ```


3. Repeat the setup for both droplets.


### Task 5: Create the System User

1. Create a system user `webgen` with a home directory at `/var/lib/webgen` and a non-login shell:
   ```bash
   sudo useradd -r -d /var/lib/webgen -s /usr/bin/nologin webgen
   ```
2. Ensure the `webgen` user owns its home directory and all subdirectories:
   ```bash
   sudo mkdir -p /var/lib/webgen/{bin,HTML,documents}
   sudo chown -R webgen:webgen /var/lib/webgen
   ```

### Directory Structure:
```
/var/lib/webgen/
├── bin/
│   └── generate_index
├── documents/
│   ├── file-one
│   └── file-two
└── HTML/
    └── index.html
```

### Task 6: Automate Index Generation

1. Create the service file `/etc/systemd/system/generate-index.service`:
   ```ini
   [Unit]
   Description=Generate index.html for system information
   After=network-online.target
   Wants=network-online.target

   [Service]
   User=webgen
   Group=webgen
   ExecStart=/var/lib/webgen/bin/generate_index

   [Install]
   WantedBy=multi-user.target
   ```
2. Create the timer file `/etc/systemd/system/generate-index.timer`:
   ```ini
   [Unit]
   Description=Run generate-index.service daily at 05:00

   [Timer]
   OnCalendar=*-*-* 05:00:00
   Persistent=true

   [Install]
   WantedBy=timers.target
   ```
3. Enable and start the timer:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable generate-index.timer
   sudo systemctl start generate-index.timer
   ```

**Verification:**
- Check timer status:
  ```bash
  sudo systemctl list-timers --all
  ```
- View logs:
  ```bash
  sudo journalctl -u generate-index.service
  ```


### Task 7: Set Up the Firewall

1. Install and configure UFW:
   ```bash
   sudo pacman -S ufw
   sudo ufw allow ssh
   sudo ufw allow http
   sudo ufw limit ssh
   sudo ufw enable
   ```

- Check firewall status:
  ```bash
  sudo ufw status verbose
  ```

## Verification

**Access the Web Server:**
   - Visit `http://<load_balancer_ip>` in a browser to verify the system information page.
   - Visit `http://<load_balancer_ip>/documents/` to verify file server functionality.
