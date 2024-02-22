# Nimbus VC

### Prepare the validator data directory

1\) Create 3 new folders to store the validator client data, validator keystore, and the validator keystore password

2\) Copy the validator keystores and it's password file into their respective folders

3\) Change the owner of this new folder to the `nimbus` user

4\) Restrict permissions on this new folder such that only the owner is able to read, write, and execute files in this folder

```sh
sudo mkdir -p /var/lib/nimbus_validator/validator_keystores /var/lib/nimbus_validator/keystore_password
sudo cp ~/validator_keys/<validator_keystore.json> /var/lib/nimbus_validator/validator_keystores
sudo cp ~/validator_keys/<validator_keystore_password.txt> /var/lib/nimbus_validator/keystore_password
sudo chown -R nimbus:nimbus /var/lib/nimbus_validator
sudo chmod 700 /var/lib/nimbus_validator
```

### Configure the validator client service

Create a systemd configuration file for the Nimbus Validator Client service to run in the background.

```bash
sudo nano /etc/systemd/system/nimbusvalidator.service
```

Paste the configuration parameters below into the file:

```
[Unit]
Description=Nimbus Validator Client (Holesky)
Wants=network-online.target
After=network-online.target

[Service]
User=nimbus
Group=nimbus
Type=simple
Restart=always
RestartSec=5
ExecStart=/usr/local/bin/nimbus_validator_client \
  --data-dir=/var/lib/nimbus_validator \
  --validators-dir=/var/lib/nimbus_validator/validator_keystores \
  --secrets-dir=/var/lib/nimbus_validator/keystore_password \
  --payload-builder=true \
  --beacon-node=http://127.0.0.1:5052 \
  --metrics \
  --metrics-port=8108 \
  --suggested-fee-recipient=0x9188DC42d33ad7d14cE460AADfcE00C2eeC19BbD \
  --graffiti="<your_graffiti_of_choice>" \
  --doppelganger-detection

[Install]
WantedBy=multi-user.target
```

Once you're done, save with `Ctrl+O` and `Enter`, then exit with `Ctrl+X`. Understand and review your configuration summary below, and amend if needed.

**Nimbus Validator Client configuration summary:**

1. `--data-dir`: Specify the directory for Nimbus to store the validator info
2. `--validators-dir`: Specify the directory for Nimbus to retrieve the validator keystores
3. `--secrets-dir`: Specify the directory for Nimbus to retrieve the password decrypting the validator keystores
4. `--payload-builder`: Required when using external builders to build blocks (e.g. MEV relays)
5. `--beacon-node`: URLs to connect to the main and backup consensus clients if any.&#x20;
6. `--metrics`: Enable metrics for monitoring
7. `--metrics-port`: Set the port for retrieving metrics
8. `--suggested-fee-recipient`: ETH wallet address to receive rewards from block proposals and MEV bribes
9. `--graffiti`: Optional text to display on-chain when your validator proposes a block
10. `--doppelganger-detection`: Helps prevents slashing due to double signing by checking if your validator keys are already active on the network. _**Not a fool-proof solution.**_

### Start the Nimbus Validator Client service

Reload the systemd daemon to register the changes made, start the Nimbus Validator Client, and check its status to make sure its running.

```bash
sudo systemctl daemon-reload
sudo systemctl start nimbusvalidator.service
sudo systemctl status nimbusvalidator.service
```

The output should say the Nimbus Validator Client is **“active (running)”.** Press CTRL-C to exit and the Nimbus Validator Client will continue to run.

Use the following command to check the logs for any warnings or errors:

```bash
sudo journalctl -fu nimbusvalidator -o cat | ccze -A
```

**Expected output:**

<figure><img src="../../.gitbook/assets/image (142).png" alt=""><figcaption></figcaption></figure>

Press `CTRL-C` to exit.

If the Nimbus Validator Client service is running smoothly, we can now enable it to fire up automatically when rebooting the system.

```bash
sudo systemctl enable nimbusvalidator
```

**Expected output:**

```
Created symlink /etc/systemd/system/multi-user.target.wants/nimbusvalidator.service → /etc/systemd/system/nimbusvalidator.service.
```

### Remove duplicates of validator keystores

To prevent configuration mistakes leading to double signing in the future, remove duplicate copies of the validator signing keystores once everything is running smoothly.

```sh
sudo rm -r ~/validator_keys
```

## Resources

* Releases: [https://github.com/status-im/nimbus-eth2/releases](https://github.com/status-im/nimbus-eth2/releases)
* Documentation: [https://nimbus.guide/install.html](https://nimbus.guide/install.html)
* Discord: [https://discord.gg/BWKx5Xta](https://discord.gg/BWKx5Xta)