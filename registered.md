# v1.0 Registered Block Producer Onboarding

A Registered Block Producer is a network node that helps process and validate transaction blocks on the platform according to the protocol consensus mechanism. A Registered Block Producer will use the Partner Chains toolkit (as well as other tools) to contribute to the validity of the partner chain ledger and, in turn, secure it.

## Order of Operations
1. Become a Cardano SPO
2. Install Partner Chains dependencies 
    1. Cardano Node - v9.0.0
        1. Ogmios - v6.5.0
        2. Kupo - @TODO v9.0.0 compatible version?
        3. cardano-db-sync - v13.3.0.0(postgreSQL - v15.3)
    2. Download the Partner Chain node - v1.0
3. Run Wizard to generate Aura, Grandpa and Sidechain keys
4. Get chain parameters from the Governance Authority(Chain Builder)
5. Register for the Partner Chain
6. Run the Partner Chain node

**_NOTE:_** This guide is currently aimed at the `preview` testnet only. In most `cardano-node` operations, this can be set with `--testnet-magic 2`. If you need to setup an SPO, be sure its setup on **preview**.

### 1. Become a Cardano SPO on Preview

Cardano SPO keys are necessary to register as a Partner Chain validator. The installation for a Cardano SPO is outside the scope of this guide. Refer to our [recommended guide](https://cardano-course.gitbook.io/cardano-course/handbook) for documentation and video instruction.

Once you have the Cardano SPO keys you are ready to continue with this guide.

### 2. Install Partner Chains dependencies

To run the Partner Chains stack, several dependencies need to be installed on the `cardano-node`.

Ogmios, Kupo and db-sync are essential to enable registration communication with the main chain (Cardano).

Ogmios is a lightweight bridge interface for `cardano-node`. It offers a WebSocket API that enables local clients to speak to the main chain via JSON/RPC.

Kupo is a fast, lightweight and configurable chain-index for Cardano.

### 2.1 Cardano node dependencies

**_NOTE:_** Be mindful of file paths in the instruction sets below. Your `cardano-node` may have slightly different paths for certain files. Replace file paths below with the paths relevant to your node.

### 2.1.1 Ogmios - v6.5.0

It is recommend to install [Ogmios](https://github.com/CardanoSolutions/ogmios) via pre-built binaries.

You can also build from source, though it requires a significant amount of depencies.

1. Obtain the [binary](https://github.com/CardanoSolutions/ogmios/releases)
2. Change permissions on the file `chmod +x /home/ubuntu/ogmios`
3. Run Ogmios as a service

```
sudo tee /etc/systemd/system/ogmios.service > /dev/null <<EOF
[Unit]
Description=Ogmios Service
After=network.target

[Service]
User=ubuntu
Type=simple
ExecStart=/usr/local/bin/ogmios \
  --host=0.0.0.0 \
  --node-config=/home/ubuntu/testnet/configs/config.json \
  --node-socket=/home/ubuntu/testnet/node.socket
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload && \
sudo systemctl enable ogmios.service && \
sudo systemctl start ogmios.service
```

4. Observe logs

```
journalctl -fu ogmios.service
```

For further instructions, please see [Ogmios](https://ogmios.dev/getting-started/building/)

### 2.1.2 Kupo - v @TODO

It is recommended to install [Kupo](https://github.com/CardanoSolutions/kupo) via pre-built binaries as well. You can also build Kupo from source.

1. Obtain the [binary](https://github.com/CardanoSolutions/kupo/releases)
2. Change permissions on the file `chmod +x /home/ubuntu/kupo`
3. Run Kupo as a service
```
sudo tee /etc/systemd/system/kupo.service > /dev/null <<'EOF'
[Unit]
Description=Kupo Service
After=network.target

[Service]
User=ubuntu
Type=simple
Environment="HOME=/home/ubuntu"
ExecStart=/usr/local/bin/kupo \
  --node-socket $HOME/testnet/node.socket \
  --node-config $HOME/testnet/configs/config.json \
  --since origin \
  --defer-db-indexes \
  --match "*" \
  --workdir $HOME/kupo
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload && \
sudo systemctl enable kupo.service && \
sudo systemctl start kupo.service
```

Please refer to [Kupo](https://cardanosolutions.github.io/kupo/#section/Overview) for detailed instructions.

### 2.1.3 cardano-db-sync - v13.3.0.0

The Cardano node needs to be setup with db-sync. 

1. Acquire the [binary](https://github.com/IntersectMBO/cardano-db-sync/releases) and add it to the PATH
2. Setup PostgreSQL server

```
sudo apt install postgresql postgresql-contrib
sudo systemctl start postgresql.service
# Enter shell as default postgres user:
sudo -i -u postgres
psql
CREATE USER ubuntu WITH PASSWORD 'XXXXXXXXXXXXX';
ALTER ROLE ubuntu WITH SUPERUSER;
ALTER ROLE ubuntu WITH CREATEDB;
# Verify user is created and has the roles with \du
CREATE DATABASE cexplorer;
# Verify db is created with \l. If anytime a command doesn't work restart postgres service.
# This check should return empty. It will be filled with db sync relations.
PGPASSFILE=~/cardano-db-sync/config/pgpass-preview ./postgresql-setup.sh --check
```
3. Run db sync as a service

```
sudo tee /etc/systemd/system/cardano-db-sync.service > /dev/null <<EOF
[Unit]
Description=Cardano DB Sync Service
After=network.target

[Service]
Environment=PGPASSFILE=/home/ubuntu/cardano-db-sync/config/pgpass-preview
ExecStart=/usr/local/bin/cardano-db-sync --config /home/ubuntu/testnet/configs/db-sync-config.json --socket-path /home/ubuntu/testnet/node.socket --state-dir /home/ubuntu/testnet/db-sync/ledger-state --schema-dir /home/ubuntu/cardano-db-sync/schema/
User=ubuntu
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload && \
sudo systemctl enable cardano-db-sync.service && \
sudo systemctl start cardano-db-sync.service
```

4. Observe logs

```
journalctl -fu cardano-dy-sync.service
```

### 2.2 Download the Partner Chain node - v1.0

@TODO -- this is a private repo, needs to be public or zipped
The repository is [here](https://github.com/input-output-hk/sidechains-substrate-node)

### 3. Run the generate-keys Wizard

The generate-keys wizard is designed to simplify the process of getting started with a partner chains node. This is the initial step for network participants who do not yet have keys. It applies to the governance authority(glossary), registered candidate(glossary), and permissioned candidate(glossary).

Before you run the wizard, make sure all the dependencies for the partner chains stack are installed. The dependencies may change depending on your specific journey.

The generate-keys wizard will generate necessary keys and save them to your node’s keystore. The following keys will be created:

1. ECDSA cross-chain key
2. ED25519 Grandpa key
3. SR25519 Aura key

If these keys already exist in the node’s keystore, you will be asked to overwrite existing keys. The wizard will also generate a network key for your node if needed.

To complete key generation, the wizard requires the following inputs:
1. node base path
2. partner chains node executable

To start the wizard, run the following command in the node repository:
`cargo run --bin partner-chains-cli -- generate-keys`

Then input the node base path as well as the partner chains node executable. These are saved in `partner-chains-cli-resources-config.json`

Now the wizard will output `partner-chains-public-keys.json` containing three keys (Partner chain, Grandpa, Aura)
```js
{
	"sidechain_pub_key": "0x<key>",
	"aura_pub_key": "0x<key>",
	"grandpa_pub_key": "0x<key>"
}
```

These keys can then be shared with the governance authority pertaining to the specific user journey followed [@TODO -- link to user journeys]

### 4. Obtain Chain Parameters

@TODO -- add obtain instructions. Currently Midnight uses a docker-image for this, but I don't think this sould be our only option

### 5. Register for the Partner Chain

Registration is a three-step process, with the second step executed on the cold machine, so there are three wizards.

The configuration of the chain is stored in the file partner-chains-cli-chain-config.json. This file should be present and identical for every node participating in the chain. It is called the chain config file in this document.

Information about the resources used by each node is stored in the file partner-chain-cli-resources-config.json. This file should be present for every node participating in the chain, but its contents are specific to each node. It is called the resources config file in this document.

#### Register-1 wizard

This part obtains a registration UTXO.

To start the wizard, run the following command in the node repository:
`partner-chains-cli register1`

The wizard checks if the chain config file is present and valid. If not, it informs you that you should obtain it from the governance authority or run the prepare-configuration wizard(link), and exits.

The wizard checks for the keystore using the resources config base path. If it is missing, it informs you that you should run the `generate-keys` wizard first, and exits.

The wizard runs the same steps as the prepare-configuration wizard(link) to set the  `cardano_cli` and `cardano_payment_verification_key_file` fields.

The wizard asks for the Cardano node socket path, proposing the default from the resources config field `cardano_node_socket_path`. If the field is unavailable, the proposed default is `node.socket`.

The wizard derives a payment address from the payment verification key:

```
<cardano-cli-command> address build --payment-verification-key-file
<cardano-payment-verification-key-file> <cardano-network-parameter>
```

The `<cardano-network-parameter>` is `--testnet-magic <cardano-network>`  if `cardano.network` is not 0. Otherwise, it is empty.

The wizard executes the command to read user UTXOs:

`<cardano-cli-command> query utxo <cardano-network-parameter> --address <derived-address>`

It parses the output, filters the UTXOs, retains only the ones with `TxOutDatumNone`, and presents them to you, together with their lovelace balance, as a table. Use the up and down arrow keys to choose a row and press `enter` to select it.

If there are no suitable UTXOs, the wizard exits with a suitable message. If the `<cardano-cli-command>` fails, the wizard exits with a suitable message.

You **must not** spend the selected UTXO, because it needs to be consumed later in the registration process.

Now the wizard outputs the whole command for obtaining signatures. You use this command in the next step (`register-2 wizard`). You must run it on a newly prepared machine with the main chain cold signing key.

#### Register-2 wizard
This part obtains signatures for the registration message. All parameters, except the stake pool operator’s (SPO) cold signing key, are supplied as arguments.

The wizard informs you that it will use the SPO cold signing key for signing the registration message.

It asks for the path for the main chain cold signing key, proposing default from `cold.vkey`.

It parses the given file and fails with an appropriate message if the key is invalid.

It asks for the path for the sidechain signing key, proposing the default from `sidechain.skey`.

It outputs the final command for the `register-3 wizard`. It includes signatures, but no keys are present. You must run it on a machine with a running Cardano node.

#### Register-3 wizard
This part executes the registration command.

If the chain config file is not present or invalid, the wizard informs you that you should either obtain it from the governance authority or run the `prepare-configuration wizard`, and exits.

The wizard checks if the field `cardano_cli` is present, otherwise, it asks you for it. Since part 1 was run, it should be present.

If the resources config field `cardano_node_socket_path` is not present, the wizard will ask for it. Since part 1 was run, it should be present.

The wizard informs you that your payment signing key is used to sign the registration transaction and that the key will not be stored nor communicated over the network.

The wizard asks for the `cardano_payment_payment_signing_key_file` path.

The wizard executes the `partner-chains-cli register` command internally with all the required parameters.

The wizard will give you the option of displaying the registration status. If you choose to display it, the wizard informs you that it will query the DB Sync PostgreSQL state, use the `substrate-node` command to query the user registration status, and output the registration status for the epoch that is two epochs ahead of the current one.

### 6. Run the Partner Chain node

Registration is effective after 1-2 Cardano epochs. After the waiting period, the Partner Chain node is registered on the Partner Chain and is a selection option for the consensus committee.

The `start-node wizard` is used to start a partner chain node. It is run by a registered(glossary) or permissioned(glossary) candidate.

The configuration of the chain is stored in the file `partner-chains-cli-chain-config.json`. This file should be present and identical for every node participating in the chain. It is called the chain config file in this guide.

Information about the resources used by each node is stored in the file `partner-chain-cli-resources-config.json`. This file should be present for every node participating in the chain, but its contents are specific to each node. It is called the resources config file in this document.

To start the wizard, run the following command in the node repository:

`partner-chains-cli start-node`

First, the wizard checks if all required keys are present. If not, it reminds you to the run the `generate-keys wizard(link)` first, and exits.

If the `chain-spec` file is not present, you should obtain it from the governance authority or run the `create-chain-spec wizard(link)` before trying again.

Next, the wizard checks the chain config file. If it is missing or invalid, you should obtain it from the governance authority or run the `prepare-configuration wizard(link)` before trying again.

The resources config field `db_sync_postgres_connection_string` is used to connect to the DB Sync database. If the value is missing, the wizard prompts for it, using the default value `postgresql://postgres-user:postgres-password@localhost:5432/cexplorer`.

Unless the `--silent` option was used, the wizard outputs all relevant parameters and asks if they are correct. If not, you should edit the chain config and/or resources config files and run the wizard again.

The wizard sets the environment variables required by the node, and runs the node using the `partner-chains-cli` run command.