## NOTE: may not be true in the future, but at least it's the workaround for now


### Prerequire
- Assume i'm staying at `/root` path (can be anywhere on your machine)
- Download file script: `curl https://cli.nexus.xyz/install.sh > nexus.sh`

### Update `nexus.sh` file
```
#!/bin/sh

/root/.cargo/bin/rustc --version || curl https://sh.rustup.rs -sSf | sh
NEXUS_HOME=$HOME/.nexus


git --version 2>&1 >/dev/null
GIT_IS_AVAILABLE=$?
if [ $GIT_IS_AVAILABLE != 0 ]; then
  echo Unable to find git. Please install it and try again.
  exit 1;
fi

if [ -d "$NEXUS_HOME/network-api" ]; then
  echo "$NEXUS_HOME/network-api exists. Updating.";
  (cd $NEXUS_HOME/network-api && git pull)
else
  mkdir -p $NEXUS_HOME
  (cd $NEXUS_HOME && git clone https://github.com/nexus-xyz/network-api)
fi

(cd $NEXUS_HOME/network-api/clients/cli && /root/.cargo/bin/cargo run --release --bin prover -- beta.orchestrator.nexus.xyz)
```

### Create a systemd service 
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

### Check the Status (waiting for a few minutes to let it build the code)
```
sudo systemctl status nexus.service
```
something like this, this is ok
```
● nexus.service - Nexus Process
     Loaded: loaded (/etc/systemd/system/nexus.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2024-10-26 16:23:13 CEST; 2min 16s ago
   Main PID: 951443 (nexus.sh)
      Tasks: 34 (limit: 77071)
     Memory: 1.2G
        CPU: 1min 14.009s
     CGroup: /system.slice/nexus.service
             ├─951443 /bin/sh /root/nexus/nexus.sh
             └─951458 target/release/prover beta.orchestrator.nexus.xyz

Oct 26 16:23:40 vmi2192653.contaboserver.net nexus.sh[951458]: Proved step 14 at 3.85 proof cycles/sec.
Oct 26 16:23:41 vmi2192653.contaboserver.net nexus.sh[951458]: Proved step 15 at 4.16 proof cycles/sec.
Oct 26 16:23:42 vmi2192653.contaboserver.net nexus.sh[951458]: Proved step 16 at 3.80 proof cycles/sec.
```


### Monitor the Logs
```
journalctl -u nexus.service -f

ctrl + c to exit
```

Whenever there is a new update, just need to download the script file and restart the service
```
sudo systemctl restart nexus.service
```