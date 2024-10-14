## NOTE: may not be true in the future, but at least it's the workaround for now


### Prerequire
- Assume i'm staying at `/root` path (can be anywhere on your machine)
- Download file script: `curl https://cli.nexus.xyz/install.sh > nexus.sh`

### create a service 
`nano /etc/systemd/system/nexus.service`

and input this configuration
```
[Unit]
Description=Nexus Process
After=network.target

[Service]
ExecStart=/root/nexus.sh  # <==== make sure to change this file location to match where you put the file
Restart=on-failure
RestartSec=5
RestartPreventExitStatus=127
SuccessExitStatus=127
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

### Reload the systemd daemon
```
sudo systemctl daemon-reload
```

### Start and Enable the Service
```
sudo systemctl start nexus.service
sudo systemctl enable nexus.service
```

### Check the Status
```
sudo systemctl status nexus.service
```

### Monitor the Logs
```
journalctl -u nexus.service -f
```

Whenever there is a new update, just need to download the script file and restart the service
```
sudo systemctl restart nexus.service
```