# v1.0 Governance Authority (Chain Builder) Onboarding

PC Chain Builder (gov auth): Partner chains builders are organizations that want to build their own blockchains according to their business cases. They will utilize the Partner chains Toolkit as well as other tools, to build and run a separate blockchain that can be interoperable with the Cardano network. They will each have their own operations and business models, and the Partner chain SDK aims to be versatile enough to support any business use case.

## Order of Operations
1. Install dependencies
    1. cardano-node
        1. Ogmios - v6.5.0
        2. Kupo - @TODO v9.0.0 compatible version?
        3. db sync - v13.3.0.0 (postgreSQL - v15.3)
    2. Download the Partner Chain node - v1.0
2. Run `generate-keys` wizard
3. Run `chain-config` wizard
    1. Choose chain parameters
    2. Provide signing key
4. Run `chain-spec` wizard
5. Define d-parameter and permissioned candidate list
6. Run PC node
7. Invite Registered SPO and Permissioned candidates to join the chain

### 1. Install Partner Chains dependencies

To run the Partner Chains stack, several dependencies need to be installed on the `cardano-node`.

Ogmios, Kupo and db-sync are essential to enable registration communication with the main chain (Cardano).

Ogmios is a lightweight bridge interface for `cardano-node`. It offers a WebSocket API that enables local clients to speak to the main chain via JSON/RPC.

Kupo is a fast, lightweight and configurable chain-index for Cardano.

### 1.1 Cardano node v9.0.0

Cardano node is required to start a partner chain. The installation of `cardano-node` is outside the scope of this guide. Refer to our [recommended guide](https://cardano-course.gitbook.io/cardano-course/handbook) for documentation and video instruction.

Once your node is synced with the `preview` network, you are ready to continue with this guide.

### 1.1 Cardano node dependencies

**_NOTE:_** Be mindful of file paths in the instruction sets below. Your `cardano-node` may have slightly different paths for certain files. Replace file paths below with the paths relevant to your node.

### 1.1.1 Ogmios - v6.5.0

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

### 1.1.2 Kupo - v @TODO

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

### 1.1.3 cardano-db-sync - v13.3.0.0

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

### 1.2 Download the Partner Chain node - v1.0

@TODO -- this is a private repo, needs to be public or zipped
The repository is [here](https://github.com/input-output-hk/sidechains-substrate-node)

### 2. Run the generate-keys Wizard

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

### 3. Run `chain-config` wizard

The prepare-configuration wizard is one of the tools used to build and configure a new partner chain. The governance authority(glossary) runs this wizard after the `generate-keys` wizard (link).

The configuration of the chain is stored in the file `partner-chains-cli-chain-config.json`.  This file should be present and identical for every node participating in the chain. It is called the chain config file in this document.

Information about the resources used by each node is stored in the file `partner-chain-cli-resources-config.json`. This file should be present for every node participating in the chain, but its contents are specific to each node. It is called the resources config file in this document.

To start the wizard, run the following command in the node repository:

`partner-chains-cli --prepare-configuration`

The wizard has three steps: update the bootnodes array, set the sidechain parameters, and store the main chain configuration.

Updating the bootnodes array
If any of the following actions cannot complete, the wizard exits with an appropriate error message.

If the `generate-keys` wizard(link) has been correctly run, the file `<substrate_node_base_path>/chains/partner_chains_template/keystore` contains the generated keys and `<substrate_node_base_path>/chains/partner_chains_template/keystore/network` contains the node key.

If the node key is found, the wizard prompts for a decision on whether the node is accessible via hostname or IP address (hostname/ip), and asks you to enter the appropriate network address.  It should be a publicly available address because it will be used by all clients as the first point of contact with the chain

The wizard updates the bootnodes array in the chain config file with the value constructed from the hostname or IP and the node ID. The copy of this file in the working folder can be manually edited to update the bootnodes array.

When this step succeeds, the bootnodes array in the chain config file contains the node ID and its network address. The wizard starts the next step.

Setting the sidechain parameters
If any of the following actions cannot complete, the wizard exits with an appropriate error message.

The wizard reads the chain config file. If it is missing, the wizard will create it. If either the chain config or resources config file has format errors, the wizard will inform the user that it has to be deleted or fixed manually and exit.

The wizard warns you that it will establish partner chain parameters: `chain_id` and `governance_authority`. If the chain config field chain_parameters.governance_authority is present, the wizard displays it and asks if it is correct. If it is incorrect, the wizard asks for the Cardano CLI command, suggesting the default from the resources config field `cardano_cli`, or `cardano-cli` if there is no value. The wizard updates the resources config with your choice. Note: this pattern applies to all the following configuration fields, so it will not be repeated for each one.

The wizard asks for the payment verification key file and proposes the default from the resources config field `cardano_payment_verification_key_file`. If the config is unavailable, the default is `payment.vkey`.

The wizard tries to read the payment verification key and derive its hash: `<cardano-cli-command> address key-hash --payment-verification-key-file <cardano-payment-verfication-key-file>`. If the command fails, the wizard exits with an appropriate message. Otherwise, it updates the chain config field `chain_parameters`.`governance_authority` with the key hash and informs you.

The wizard asks for the chain ID, informing you that the pair (governance authority, chain id)identifies a partner chain. It has to be unique, and allowable values are in the range of [0; 65535]. The chain config field `chain_parameters.chain_id` is used as default (and target value). 0 is the default.

When this step succeeds, the chain config file has the `chain_parameters` part set, the resources config file is updated, and the wizard starts the next step.

Storing the main chain configuration
The wizard shows the `sidechain-main-cli` version (from its binary).

The wizard asks for the Cardano network (mainnet, preprod, or preview), using the chain config field cardano.network as default. In absence of the field, the default is mainnet. For each network, it sets Cardano parameters using hardcoded values as defaults. Otherwise, it asks for each field's values, giving hints about expected content. For example, first_epoch_number is a number of the first epoch in the Shelley era.

The wizard informs you that to get the main chain follower configuration, it needs Kupo and Ogmios, and asks for: kupo.protocol (http/https, default http), kupo.hostname (default localhost), kupo.port (default 1442), and for: ogmios.protocol (http/https, default http), ogmios.hostname (default localhost), ogmios.port (default 1337).

The wizard runs the `sidechain-main-cli` addresses command with all the required parameters taken from the resources config and chain config files, parses the response, and updates the `committee_candidates_address`, `d_parameter_policy_id`, and `permissioned_candidates_policy_id` fields of `cardano_addresses` in the chain config file, notifying you about their values.

If the chain config array `initial_permissioned_candidates` is absent, the wizard sets it to an empty array.

The wizard reports that the chain config file is ready for distribution to network participants and also that the `create-chain-spec` wizard(link) should be executed when keys of permissioned candidates are gathered.

After the wizard completes, the chain config file has the cardano object set with the fields for main chain configuration: `security_parameter`, `active_slots_coeff`, `first_epoch_number`, `first_slot_number`, `epoch_duration_millis`, `first_epoch_timestamp_millis`, and the main chain follower configuration fields will be set: `committee_candidates_address`, `d_parameter_policy_id`, `permissioned_candidates_policy_id`.

Here’s a sample file:
```
{
  "bootnodes": [
    "/dns/myhost/tcp/3033/p2p/12D3KooWHBpeL1GgfnuykXzSvNt9wCbb1j9SEG6d4DJu5cnJR7sh"
  ],
  "cardano": {
    "active_slots_coeff": 0.05,
    "epoch_duration_millis": 432000000,
    "first_epoch_number": 208,
    "first_epoch_timestamp_millis": 1596059091000,
    "first_slot_number": 4492800,
    "network": 0,
    "security_parameter": 2160
  },
  "chain_parameters": {
    "block_stability_margin": 0,
    "chain_id": 0,
    "genesis_committee_utxo": "0000000000000000000000000000000000000000000000000000000000000000#0",
    "governance_authority": "0x76da17b2e3371ab7ca88ce0500441149f03cc5091009f99c99c080d9",
    "threshold_denominator": 3,
    "threshold_numerator": 2
  }
}
```

### 3.1 Choose chain parameters

@TODO -- more info here

### 3.2 Provide signing key

@TODO -- more info here

### 4. Run `chain-spec` wizard

The chain-spec wizard is one of the tools used to build and configure a new partner chain. 

The wizard reads the file `partner-chains-cli-chain-config.json`. This file should be present and identical for every node participating in the chain. It is called the chain config file in this document.

Before you run this command, someone must have installed Docker and run the `generate-keys` wizard(link) and the `prepare-configuration` wizard(link). Usually, the governance authority runs the command to generate the `chain-spec.json` file, which is then distributed to the  registered(glossary) or permissioned(glossary) candidates. Alternatively, each candidate can run this wizard themselves using the chain config file obtained from the governance authority.

To start the wizard, run the following command in the node repository:

`partner-chains-cli create-chain-spec`

The wizard displays the contents of `chain_parameters` and `initial_permissioned_candidates` from the chain config file. You can manually modify these values before running this wizard.

The chain specification file `chain-spec.json` will be created using these values, and a Docker image will be used for a deterministic build(glossary).

The wizard runs the Docker image and informs you of the full path to the `chain-spec.json` file. If you are the governance authority, you can distribute this file to block production committee candidates.

The next step is for the governance authority to run the `setup-main-chain-state` wizard(link) command.

### Define d-parameter and permissioned candidate list

@TODO -- more info here

### Run the PC node

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

### Invite Registered SPO and Permissioned candidates to join the chain

The partner chain is now ready to start accepting validator nodes. Permissioned candidates (@TODO -- link) and Registered candidates (@TODO -- link) have different onboarding processes. Please follow the respective steps for the type of user.
