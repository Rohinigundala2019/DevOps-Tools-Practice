Here’s your guide to **Nexus Repository Manager** in a structured format:

---

### **1. Introduction to Nexus Repository Manager**
- **Nexus Repository OSS**: Free and open-source.
- **Nexus Repository Pro**: Paid version with advanced features.
- **Supported formats**: Maven, npm, PyPI, Docker, Helm, Raw, etc.

---

### **2. Installation & Setup**
#### **Installing Nexus on Linux**
```bash
wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz
tar -xvf latest-unix.tar.gz
mv nexus-<version> nexus
cd nexus
./bin/nexus start
```
- **Web Interface**: http://localhost:8081
- **Admin Login**:
  - Username: `admin`
  - Password: Found in `/sonatype-work/nexus3/admin.password`

---

### **3. Nexus Directory Structure**
- **/nexus/bin**: Start/Stop scripts.
- **/nexus/etc**: Configuration files.
- **/nexus/sonatype-work**: Repository storage and logs.
- **/nexus/logs**: Application logs.

---

### **4. Nexus User Management**
#### **Create a New User**
1. Login as admin.
2. Go to **Security → Users**.
3. Click **Create User**.
4. Assign roles (e.g., `nx-admin`, `nx-repository-view-<repo>-read`).

---

### **5. Repository Types**
1. **Hosted Repository**: Internal artifact storage (e.g., Maven or Docker registry).
2. **Proxy Repository**: Caches external repositories (e.g., Maven Central).
3. **Group Repository**: Aggregates multiple repositories under one URL.

---

### **6. Configuring a Repository**
#### **Creating a Maven Hosted Repository**
1. Go to **Repositories → Create Repository**.
2. Select **Maven (hosted)**.
3. Configure:
   - Version Policy: `Release/Snapshot/Mixed`
   - Write Policy: `Allow redeploy/Disable`
   - Storage: **Blob Store**
4. **Save & Deploy**.

---

### **7. Uploading Artifacts to Nexus**
#### **Upload via UI**
1. Go to **Repositories → Select Repository**.
2. Click **Upload**.
3. Select the artifact and provide metadata.

#### **Upload via Maven**
```xml
<servers>
  <server>
    <id>nexus</id>
    <username>admin</username>
    <password>admin123</password>
  </server>
</servers>
```
```bash
mvn deploy -DrepositoryId=nexus -Durl=http://localhost:8081/repository/maven-releases/
```

---

### **8. Proxying Remote Repositories**
#### Example: Proxying Maven Central
1. Go to **Repositories → Create Repository**.
2. Select **Maven (proxy)**.
3. Enter Remote URL: `https://repo.maven.apache.org/maven2`.
4. Enable **Caching** to store downloaded artifacts.
5. **Save and Apply**.

---

### **9. Configuring Nexus as a Docker Registry**
1. **Create Docker Hosted Repository**:
   - Name: `docker-internal`
   - HTTP Port: `8082`
2. **Push Docker Image**:
```bash
docker login -u admin -p admin123 http://localhost:8082
docker tag myimage localhost:8082/myrepo:latest
docker push localhost:8082/myrepo:latest
```

---

### **10. Cleanup Policies**
1. Go to **Admin → Cleanup Policies**.
2. Create policies to remove old/unneeded artifacts.
3. Apply policies to repositories.

---

### **11. Backup & Restore**
#### **Backup**
```bash
tar -czvf nexus-backup.tar.gz /path/to/nexus
```
#### **Restore**
```bash
tar -xzvf nexus-backup.tar.gz -C /path/to/nexus
```

---

### **12. Logs & Monitoring**
#### **Viewing Logs**
```bash
tail -f sonatype-work/nexus3/logs/nexus.log
```
#### **Health Check**
- Go to **System → Health Check** to monitor performance and security.

---

### **13. Security Hardening**
- Change **default admin password**.
- Disable **anonymous access** unless required.
- Use **HTTPS** for secure communication.
- Implement **Role-Based Access Control (RBAC)**.

---

### **14. Upgrading Nexus**
```bash
systemctl stop nexus
mv nexus nexus-old
wget latest-nexus.tar.gz
tar -xvf latest-nexus.tar.gz
mv nexus-new nexus
systemctl start nexus
```

---

### **15. Troubleshooting**
#### **Common Issues & Fixes**
- **Nexus not starting?**
  - Check logs: `cat sonatype-work/nexus3/logs/nexus.log`
- **Out of memory errors?**
  - Edit `nexus.vmoptions`: 
    ```bash
    -Xms2g
    -Xmx4g
    ```

- **Authentication Issues?**
  - Reset Admin Password:
    ```bash
    rm -rf sonatype-work/nexus3/admin.password
    ```

---

### **16. Nexus REST API**
- **List Repositories**:
```bash
curl -u admin:admin123 -X GET "http://localhost:8081/service/rest/v1/repositories"
```
- **Create Repository**:
```bash
curl -u admin:admin123 -X POST "http://localhost:8081/service/rest/v1/repositories" -H "Content-Type: application/json" -d @config.json
```

---

### **17. Integrating Nexus with CI/CD**
#### **Using Jenkins**
1. Install **Nexus Artifact Uploader Plugin**.
2. Configure Nexus credentials.
3. Upload artifacts as **Post Build Action**.

---

### **18. Best Practices**
- Use **Blob Stores** for efficient storage.
- Enable **Content Cleanup** for unused artifacts.
- Monitor **Disk Usage** regularly.
- Automate **Backups** with cron jobs.
- Enforce **Role-Based Access Control (RBAC)**.

---

### **19. Advanced Configuration and Customization**
#### **Configuring Reverse Proxy (Nginx)**
1. Install **Nginx**:
```bash
sudo apt update && sudo apt install nginx -y
```
2. Create Nginx Configuration:
```bash
server {
  listen 80;
  server_name nexus.example.com;
  location / {
    proxy_pass http://localhost:8081/;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }
}
```
3. Enable HTTPS with **Let's Encrypt**.

---

### **20. Managing Nexus Storage**
#### **Create Custom Blob Store**
1. Navigate to **Administration → Blob Stores**.
2. Choose **File** or **S3** (Pro version).
3. Set the storage path.
4. Assign the blob store to a repository.

#### **Automated Storage Cleanup**
- Set up **Cleanup Policies** for unused artifacts.
- Enable **Compact Blob Store** for better storage efficiency.

---

### **21. Running Nexus as a Systemd Service**
1. Create a service file for **Systemd**:
```ini
[Unit]
Description=Nexus Repository Manager
After=network.target

[Service]
Type=forking
User=nexus
Group=nexus
ExecStart=/opt/nexus/bin/nexus start
ExecStop=/opt/nexus/bin/nexus stop
Restart=on-abort

[Install]
WantedBy=multi-user.target
```
2. Enable and Start the service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable nexus
sudo systemctl start nexus
```

---

### **22. Integrating Nexus with Kubernetes**
#### **Deploying Nexus in Kubernetes**
1. **Create Persistent Volume**.
2. **Deploy Nexus Pod**.
3. **Expose as a Service** with LoadBalancer.

---

### **23. Configuring Nexus for High Availability (HA)**
1. Use an **External Database** (Pro version).
2. Set up a **Load Balancer** with **HAProxy** or **Nginx**.

---

### **24. Automating Nexus with Ansible**
#### **Install Nexus using Ansible**
```yaml
- name: Install Nexus
  hosts: nexus_servers
  become: yes
  tasks:
    - name: Download Nexus
      get_url:
        url: https://download.sonatype.com/nexus/3/latest-unix.tar.gz
        dest: /opt/nexus.tar.gz
    - name: Extract Nexus
      unarchive:
        src: /opt/nexus.tar.gz
        dest: /opt/
        remote_src: yes
    - name: Start Nexus
      command: "/opt/nexus/bin/nexus start"
```

---

### **25. Performance Tuning**
#### **Optimizing JVM Memory**
Edit `nexus.vmoptions`:
```diff
-Xms4g
-Xmx8g
```

Increase **Thread Pool Size**:
```ini
nexus.jetty.threads=200
```

---

### **26. Monitoring Nexus with Prometheus & Grafana**
1. Enable **Prometheus Metrics**.
2. Configure **Prometheus** to scrape metrics.
3. **Visualize in Grafana**.

---

### **27. Migrating from Nexus 2 to Nexus 3**
1. Backup Nexus 2 data.
2.

 Install Nexus 3.
3. **Use Migration Tool** (Pro version).
4. Manually transfer artifacts if using OSS.

---

### **28. Nexus Best Security Practices**
- Enforce **RBAC**.
- Always use **HTTPS**.
- Enable **IP Whitelisting**.
- **Update** Nexus regularly.

---

### **29. Troubleshooting & Common Errors**
- **Login Issues**: Reset password.
- **Nexus Crashes**: Increase heap size.
- **Slow Response**: Enable Blob Store Compression.

---

### **30. Final Tips**
- Use **CI/CD** pipelines to automate artifact deployment.
- Enable **geo-replication** for multi-region teams.
- Regularly monitor **storage usage**.

--- 

