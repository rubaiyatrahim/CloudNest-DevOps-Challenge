# CloudNest DevOps Challenge

**Student Name:** Md. Rubaiyat Rahim <br/>
**Batch:** DevOps Batch 14 <br/>
**Assignment Title:** The Friday Night Fix

## Task 1: Starting Fresh

To keep new work separate from the main branch, we need to create and switch to a new feature branch.

```
git checkout -b feature/new-client-demo
```

_Why:_ This creates a safe, isolated branch (`feature/new-client-demo`) based on the most up-to-date version of `main`. If things break, `main` remains untouched. <br/>

## Task 2: Interrupted Work

We are halfway through writing code when the urgent bug comes in. We must temporarily shelf our work without committing incomplete code.

```
# Make some incomplete changes
echo "console.log('work in progress');" > feature.js
git add feature.js
# Stash the work
git stash

# Fix the urgent bug
git checkout main
echo "Bug fixed" > bugfix.txt
git add bugfix.txt
git commit -m "Fix critical client bug"

# Restore the incomplete work
git checkout feature/new-client-demo
git stash pop
git commit -m "WIP: Feature for client demo"
```

_Why:_ `git stash` safely stores our modified tracked files and staged changes on a stack of unfinished changes, giving us a clean working directory to address the urgent bug. <br/>
![Task 2](screenshots/task2.png)

## Task 3: Cleaning the History

Nadia wants to see two different ways of bringing a feature branch up to date: a rebase (linear history) and a merge (preserved history). So, we duplicate our current feature branch so we can demonstrate both methods side-by-side.

```
git checkout feature/new-client-demo
git branch demo-rebase
git branch demo-merge

# The Merge Approach (Preserves history)
git checkout demo-merge
git merge main
# (If a text editor opens, save and close it to accept the merge commit)

# The Rebase Approach (Linear history)
git checkout demo-rebase
git rebase main
```

_The Merge Approach Result:_ This creates a "merge commit." It preserves the exact chronological history and shows that the feature branch lived independently before being joined back.<br/>
![Merge](screenshots/task3-1.png)
_The Rebase Approach Result:_ This rewinds our feature branch commits, pulls in the new main commits, and replays our feature commits on top. It looks like we wrote our feature after the latest main updates, keeping a clean, straight line of history.<br/>
![Rebase](screenshots/task3-2.png)

## Task 4: The Embarrassing Message

Fix a bad commit message.

```
git checkout feature/new-client-demo

# Create a file named asdf.txt
git add asdf.txt
git commit -m "asdf fix"

# Fix the message
git commit --amend -m "Fix database connection timeout issue"
```

## Task 5: Our Own CI

### 5.1 Create EC2 Instance

At first a new EC2 instance was created as follows:<br/>
![EC2](screenshots/ec2create.png)
<br/>

### 5.2 Configure Self-Hosted Runner

To replace the paid cloud service with a self-hosted runner, we will use GitHub Actions configured for a self-hosted machine.

#### 5.2.1. Go to our GitHub Repository > Settings > Actions > Runners.

#### 5.2.2. Click New self-hosted runner. Select Linux, x64.

#### 5.2.3. SSH into our Ubuntu server and run the exact download/configure commands GitHub provides.

_Download_

```
# Create a folder
$ mkdir actions-runner && cd actions-runner# Download the latest runner package
$ curl -o actions-runner-linux-x64-2.337.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.337.0/actions-runner-linux-x64-2.337.0.tar.gz# Optional: Validate the hash
$ echo "70920811a4f8ad4328818682bca5c6469c1c942fab52448868071d0063816613  actions-runner-linux-x64-2.337.0.tar.gz" | shasum -a 256 -c# Extract the installer
$ tar xzf ./actions-runner-linux-x64-2.337.0.tar.gz
```

_Configure_

```
# Create the runner and start the configuration experience
$ ./config.sh --url https://github.com/rubaiyatrahim/CloudNest-DevOps-Challenge --token ADERNOCSXS3YJ6565EI3WCDKYT6XU# Last step, run it!
$ ./run.sh
```

_Using your self-hosted runner_

```
# Use this YAML in your workflow file for each job
runs-on: self-hosted
```

### 5.3 Configure the Pipeline

Create a file in your project at `.github/workflows/ci.yml`:

```
name: CloudNest Self-Hosted CI
on: [push]
jobs:
  build-and-test:
    runs-on: self-hosted
    steps:
      - name: Check out repository code
        uses: actions/checkout@v3
      - name: Run Build pipeline
        run: echo "Building project..."
      - name: Run Tests
        run: echo "Testing project..."
```

### 5.4 Test Push

Commit the changes to main and push to see the CI pipeline running.<br/>
![CI](screenshots/task5-1.png)<br/>
CI found succeeded to run the pipeline in GitHub.<br/>
![CI-GitHub](screenshots/task5-2.png)<br/>
![CI-GitHub2](screenshots/task5-3.png)
_Why:_ By specifying `runs-on: self-hosted`, the CI pipeline bypasses GitHub's paid cloud runners and executes directly on the company-owned server where the runner agent is listening.

## Task 6: The Blind Server

We will install and run Node Exporter, Prometheus, Alloy and Grafana to complete this task.

### 6.1. Node Exporter Setup

Node Exporter collects system-level metrics (CPU, RAM, Disk, Network) and exposes them on port 9100.

#### 6.1.1. Download and Extract Binary:

```
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
tar xvfz node_exporter-1.7.0.linux-amd64.tar.gz
sudo mv node_exporter-1.7.0.linux-amd64/node_exporter /usr/local/bin/
rm -rf node_exporter-1.7.0.linux-amd64\*
```

#### 6.1.2. Create System User:

```
sudo useradd --no-create-home --shell /bin/false node_exporter
```

#### 6.1.3. Configure Systemd Service:

Create a service file: `sudo nano /etc/systemd/system/node_exporter.service`

```
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

#### 6.1.4. Start and Verify:

```
sudo systemctl daemon-reload
sudo systemctl enable --now node_exporter
sudo systemctl start node_exporter
sudo systemctl status node_exporter
```

#### 6.1.5. Add Inbound Rule to Allow Port `9100` from Anywhere

#### 6.1.6. **Screenshot Requirement:** Open `http://<SERVER_IP>:9100/metrics` in the browser.

![NodeExporter](screenshots/task6-1.png)

### 6.2. Prometheus Setup

Prometheus pulls the metrics exposed by Node Exporter.

#### 6.2.1. Download and Extract Binary:

```
wget https://github.com/prometheus/prometheus/releases/download/v2.49.1/prometheus-2.49.1.linux-amd64.tar.gz
tar xvfz prometheus-2.49.1.linux-amd64.tar.gz
sudo mv prometheus-2.49.1.linux-amd64/prometheus /usr/local/bin/
sudo mv prometheus-2.49.1.linux-amd64/promtool /usr/local/bin/
```

#### 6.2.2. Configure Directories and User:

```
sudo useradd --no-create-home --shell /bin/false prometheus
sudo mkdir /etc/prometheus /var/lib/prometheus
sudo mv prometheus-2.49.1.linux-amd64/consoles /etc/prometheus/
sudo mv prometheus-2.49.1.linux-amd64/console_libraries /etc/prometheus/
sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus
```

#### 6.2.3. Configure prometheus.yml:

Create the config file: `sudo nano /etc/prometheus/prometheus.yml`

```
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
```

Change the ownership of this file: `sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml`

#### 6.2.4. Configure Systemd Service:

Create a service file: `sudo nano /etc/systemd/system/prometheus.service`

```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
 --config.file /etc/prometheus/prometheus.yml \
 --storage.tsdb.path /var/lib/prometheus/ \
 --web.console.templates=/etc/prometheus/consoles \
 --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

#### 6.2.5. Start and Verify:

```
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable --now prometheus
sudo systemctl status prometheus
```

#### 6.2.6. Add Inbound Rule to Allow Port `9090` from Anywhere

#### 6.2.7. **Screenshot Requirements:** Open `http://<SERVER_IP>:9090/targets` in the browser.

![Prometheus](screenshots/task6-2.png)

### 6.3. Grafana Setup

#### 6.3.1. Install Grafana Manually:

```
sudo apt-get install -y apt-transport-https wget gnupg
sudo mkdir -p /etc/apt/keyrings
sudo wget -O /etc/apt/keyrings/grafana.asc https://apt.grafana.com/gpg-full.key
sudo chmod 644 /etc/apt/keyrings/grafana.asc
echo "deb [signed-by=/etc/apt/keyrings/grafana.asc] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
sudo apt-get update
sudo apt-get install grafana
```

#### 6.3.2. Start Service:

```
sudo systemctl start grafana-server
sudo systemctl enable --now grafana-server
sudo systemctl status grafana-server
```

#### 6.3.3. Add Inbound Rule to Allow Port `3000` from Anywhere

#### 6.3.4. Configure Datasource and Dashboard:

#### 6.3.4.1. Open `http://<SERVER_IP>:3000` (Default login: `admin / admin`).

#### 6.3.4.2. Go to Connections > Data Sources > Add data source -> Select Prometheus.

#### 6.3.4.3. Set URL to `http://localhost:9090`, then click Save & Test.

![Grafana-Prometheus](screenshots/task6-3.png)

### 6.4. Loki and Alloy Setup

#### 6.4.1. Install and Run Loki (The Log Database)

Loki will receive and store the logs that Alloy collects.

##### 6.4.1.1. Download the Loki binary and its default configuration file:

```
curl -O -L "https://github.com/grafana/loki/releases/download/v3.0.0/loki-linux-amd64.zip"
unzip loki-linux-amd64.zip
chmod a+x loki-linux-amd64
wget https://raw.githubusercontent.com/grafana/loki/main/cmd/loki/loki-local-config.yaml
```

##### 6.4.1.2. Run Loki in the background (it runs on port `3100` by default):

```
./loki-linux-amd64 -config.file=loki-local-config.yaml &
```

#### 6.4.2. Install Grafana Alloy (The Telemetry Agent)

Alloy will read our server's log files and forward them to Loki. Add the Grafana APT repository and install Alloy (for Debian/Ubuntu servers):

```
sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list

sudo apt-get update
sudo apt-get install alloy
```

#### 6.4.3. Configure Alloy

Alloy uses a specific .alloy configuration file to determine what to collect and where to send it.

- Open the Alloy configuration file: `sudo nano /etc/alloy/config.alloy`.
- Replace the contents with this configuration. This tells Alloy to read system logs from `/var/log` and send them to the Loki instance running on port `3100`:

```
// 1. Discover local log files
local.file_match "system_logs" {
  path_targets = [{"__path__" = "/var/log/*.log"}]
}

// 2. Read the discovered files
loki.source.file "log_reader" {
  targets    = local.file_match.system_logs.targets
  forward_to = [loki.write.local_loki.receiver]
}

// 3. Send the logs to your Loki instance
loki.write "local_loki" {
  endpoint {
    url = "http://localhost:3100/loki/api/v1/push"
  }
}
```

- Restart the Alloy service to apply the changes:

```
sudo systemctl restart alloy
sudo systemctl enable alloy
```

#### 6.4.4. Connect Loki to Grafana

Now that Alloy is streaming logs to Loki, we need to make them visible in our dashboard.

- Open our Grafana instance in the browser (`http://<SERVER_IP>:3000`).
- Navigate to Connections > Data Sources > Add new data source.
- Select Loki.
- Set the URL to `http://localhost:3100` and click Save & Test.
  ![Loki](screenshots/task6-4.png)
- To view the logs, go to Explore (the compass icon in the left menu), select Loki from the dropdown, and run a query like {filename="/var/log/syslog"} or {filename="/var/log/auth.log"}.
  ![Loki-2](screenshots/task6-5.png)
  <br/>
  That completes Task 6.

## Task 7: The Dashboard

Build a dashboard with 4 panels using PromQL.

- Open our Grafana instance in the browser (`http://<SERVER_IP>:3000`).
- Navigate to Dashboards > Create Dashboard.

### CPU Usage:

Add new panel name `CPU Usage` with the following PromQL:

```
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

### Memory Usage:

Add new panel name `Memory Usage` with the following PromQL:

```
100 * (1 - ((node_memory_MemFree_bytes + node_memory_Buffers_bytes + node_memory_Cached_bytes) / node_memory_MemTotal_bytes))
```

### Disk Space:

Add new panel name `Disk Space` with the following PromQL:

```
100 - ((node_filesystem_avail_bytes{mountpoint="/"} * 100) / node_filesystem_size_bytes{mountpoint="/"})
```

### Network Traffic:

Add new panel name `Network Traffic` with the following PromQL:

```
rate(node_network_receive_bytes_total[5m])
```

Finally, save the dashboard giving a name.
![Dashboard](screenshots/task7.png)
