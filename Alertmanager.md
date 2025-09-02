Alertmanager is responsible for receiving alerts generated from Prometheus and converting them into notifications. These notifications can include pages, webhooks, email messages, and chat messages.

**Alertmanager Architecture**

![Alt text](https://last9.ghost.io/content/images/2024/10/mermaid-diagram-2024-10-24-123753.png)

1. **Grouping**
Grouping categorizes alerts of similar nature into a single notification. This is especially useful during larger outages when many systems fail at once and hundreds to thousands of alerts may be firing simultaneously.

*Example*: Dozens or hundreds of instances of a service are running in your cluster when a network partition occurs. Half of your service instances can no longer reach the database. Alerting rules in Prometheus were configured to send an alert for each service instance if it cannot communicate with the database. As a result hundreds of alerts are sent to Alertmanager.

As a user, one only wants to get a single page while still being able to see exactly which service instances were affected. Thus one can configure Alertmanager to group alerts by their cluster and alertname so it sends a single compact notification. Grouping of alerts, timing for the grouped notifications, and the receivers of those notifications are configured by a routing tree in the configuration file.

2. **Inhibition**
Inhibition is a concept of suppressing notifications for certain alerts if certain other alerts are already firing.

*Example*: An alert is firing that informs that an entire cluster is not reachable. Alertmanager can be configured to mute all other alerts concerning this cluster if that particular alert is firing. This prevents notifications for hundreds or thousands of firing alerts that are unrelated to the actual issue.

Inhibitions are configured through the Alertmanager's configuration file.

3. **Silences**
Silences are a straightforward way to simply mute alerts for a given time. A silence is configured based on matchers, just like the routing tree. Incoming alerts are checked whether they match all the equality or regular expression matchers of an active silence. If they do, no notifications will be sent out for that alert. Silences are configured in the web interface of the Alertmanager.

4. **Client behavior**
The Alertmanager has special requirements for behavior of its client. Those are only relevant for advanced use cases where Prometheus is not used to send alerts.

**Alertmanager installation systemd**

This guide explains how to set up Alertmanager to be managed by systemd. The instructions are similar to the setups for Prometheus and Node Exporter. Follow the steps below to download Alertmanager, configure the necessary files and permissions, and finally run Alertmanager as a systemd service.

1. **Download and extract Alertmanager**
Download the Alertmanager package, extract its contents, and switch to the extracted directory:
```sh
wget https://github.com/prometheus/alertmanager/releases/download/v0.24.0/alertmanager-0.24.0.linux-amd64.tar.gz
# Note: You might encounter an SSL certificate issue:
tar xzf alertmanager-0.24.0.linux-amd64.tar.gz
cd alertmanager-0.24.0.linux-amd64
```
2. **Create Alertmanager user**
Create a dedicated user named `alertmanager` to run the application. This user does not require a home directory and should have a disabled login shell:
```sh
sudo useradd --no-create-home --shell /bin/false alertmanager
```
3. **Configure Alertmanager files**
Set up the configuration and data directories for Alertmanager, then change ownership to the new user.
```sh
sudo mkdir /etc/alertmanager
sudo mv alertmanager.yml /etc/alertmanager
sudo chown -R alertmanager:alertmanager /etc/alertmanager
sudo mkdir /var/lib/alertmanager
sudo chown -R alertmanager:alertmanager /var/lib/alertmanager
```
4. **Install the executables**
Copy the Alertmanager executables into /usr/local/bin so it is available system-wide:
```sh
sudo cp alertmanager /usr/local/bin
sudo cp amtool /usr/local/bin
```
After copying, update the ownership of the executable:
```sh
sudo chown alertmanager:alertmanager /usr/local/bin/alertmanager
```
If you choose to manage amtool similarly, adjust its ownership likewise:
```sh
sudo chown alertmanager:alertmanager /usr/local/bin/amtool
```
5. **Create systemd service file**
Create and configure a new systemd service file for Alertmanager:
```ini
# /etc/systemd/system/alertmanager.service
[Unit]
Description=Alert Manager
Wants=network-online.target
After=network-online.target


[Service]
Type=simple
User=alertmanager
Group=alertmanager
ExecStart=/usr/local/bin/alertmanager \
  --config.file=/etc/alertmanager/alertmanager.yml \
  --storage.path=/var/lib/alertmanager
Restart=always


[Install]
WantedBy=multi-user.target
```
Ensure the `ExecStart` command correctly points to the configuration file and data storage directory.
6. **Reload systemd and Start Alertmanager**
After setting up the service file, reload systemd to register the new service, then start and enable Alertmanager:
```sh
sudo systemctl daemon-reload
sudo systemctl start alertmanager
sudo systemctl enable alertmanager
```