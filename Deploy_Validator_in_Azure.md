# Deployment of shardnet.near node in Azure

**Useful links:**

Wallet: <https://wallet.shardnet.near.org/>

Explorer: <https://explorer.shardnet.near.org/>

Azure Free Account: <https://azure.microsoft.com/en-ca/free/>

Azure Portal: <https://portal.azure.com/>

___
___

## Step 1 -  Create free Azure Account/Subscription

* Visit <https://azure.microsoft.com/en-ca/free/> and click "Start Free"
* Provide existing or create a new Microsoft account (Office 365, Outlook, Hotmail)
* Complete creation of free Azure account. You will be provided with credit that depends on your region.
* Login to Azure portal <https://portal.azure.com/> using your account and select to upgrade your account to Pay-As-You-Go.
    You still can use your credit but also will get access to all hardware tiers including Premium SSD storage.
* Waite until upgrade for your account will be completed.

___

## Step 2 -  Create Azure resources for VM/Node deployment

       The following Azure resources will be required before VM creation: 
       Resource Group - Logical container to keep information about rest of resources. (nearpool_RG)
       Virtual Network(vNet) - IP address space that will be used. (near_vnet 10.250.250.0/24) 
       Subnet - Network subnet that is part of vNet address space. (default 10.250.250.0/24)

* Create Resource Group by navigating to Azure portal Home and type "Resource" in a search field and select it.

    Add a new resource group "nearpool_RG" by clicking on "+ Create".

![Azure_Home](https://github.com/akashkov/Near-Validator-at-Azure-/blob/main/Azure_Home.jpg?raw=true)

* Create Virtual Network by navigating into new resource group.

    Click on "+ Create". Type "virtual networks" in a search and select it in result.

![Create_vNet](https://github.com/akashkov/Near-Validator-at-Azure-/blob/main/Create_vNet.png?raw=true)

* Configure vNet with Address space 10.250.250.0/24 and default subnet 10.250.250.0/24.

![Configure_vNet](https://github.com/akashkov/Near-Validator-at-Azure-/blob/main/vNet_Config.png?raw=true)

* Create Ubuntu Virtual Machine in resource group nearpool_RG using following information:

      Operating system: Linux (ubuntu 20.04)
      VM Size: Standard D2as v4
      Authentication type: Password
      Inbound ports: SSH (22)
      OS disk type: Standard SSD (locally-redundant storage)
      Subnet: default (10.250.250.0/24)
      Public IP: (new) vm_name-ip
      Enable accelerated networking: CheckBox

![Create_VM](https://github.com/akashkov/Near-Validator-at-Azure-/blob/main/Create_VM.png?raw=true)

* When VM's status will change to Running, confirm SSH connectivity to VM using VM's public IP address using your username and password.

  > ssh username@VMpublicIP

* Logoff from VM and shutdown VM using portal(Click on Stop). Confirm if you want to preserve public IP address(VM will get same public IP everytime).

___

## Step 3 -  Adjust VM properties

* Attach additional Premium SSD drive 500GB.
    At this moment VM should be stopped.

     In Disk section select "Create and attach a new disk" as a Data disk.

![Attach_Data_Disk](https://github.com/akashkov/Near-Validator-at-Azure-/blob/main/Add_SSD.png?raw=true)

* Start VM, by selecting it in Resource group and Clicking Start in Overview section.

___

## Step 4 -  Mount data disk as ~/.near/data

* Connect/SSH to VM using public IP address.

    > ssh username@vm_public_ip

* Check device name for new 500GB disk (sda).

      $ lsblk
        NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
        loop0     7:0    0 67.8M  1 loop /snap/lxd/22753
        loop1     7:1    0   62M  1 loop /snap/core20/1581
        loop2     7:2    0 61.9M  1 loop /snap/core20/1518
        loop3     7:3    0   47M  1 loop /snap/snapd/16292
        sda       8:0    0  500G  0 disk
        sdb       8:16   0   30G  0 disk
        ├─sdb1    8:17   0 29.9G  0 part /
        ├─sdb14   8:30   0    4M  0 part
        └─sdb15   8:31   0  106M  0 part /boot/efi
        sdc       8:32   0   16G  0 disk
        └─sdc1    8:33   0   16G  0 part /mnt

* Create partition on new disk.

      $ sudo fdisk /dev/sda

        Welcome to fdisk (util-linux 2.34).
        Changes will remain in memory only, until you decide to write them.
        Be careful before using the write command.


        Command (m for help): n
        Partition type
           p   primary (0 primary, 0 extended, 4 free)
           e   extended (container for logical partitions)
        Select (default p): p
        Partition number (1-4, default 1): 
        First sector (2048-1048575999, default 2048): 
        Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-1048575999, default 1048575999): 

        Created a new partition 1 of type 'Linux' and of size 500 GiB.
        Partition #1 contains a ext4 signature.

        Do you want to remove the signature? [Y]es/[N]o: Y

        The signature will be removed by a write command.

        Command (m for help): w
        The partition table has been altered.
        Calling ioctl() to re-read partition table.
        Syncing disks.

* Confirm device name for new partition (sda1)

      $ lsblk
        NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
        loop0     7:0    0 67.8M  1 loop /snap/lxd/22753
        loop1     7:1    0   62M  1 loop /snap/core20/1581
        loop2     7:2    0 61.9M  1 loop /snap/core20/1518
        loop3     7:3    0   47M  1 loop /snap/snapd/16292
        sda       8:0    0  500G  0 disk 
        └─sda1    8:1    0  500G  0 part 
        sdb       8:16   0   30G  0 disk 
        ├─sdb1    8:17   0 29.9G  0 part /
        ├─sdb14   8:30   0    4M  0 part 
        └─sdb15   8:31   0  106M  0 part /boot/efi
        sdc       8:32   0   16G  0 disk 
        └─sdc1    8:33   0   16G  0 part /mnt

* Format partition sda1 with ext4 filesystem.

      $ sudo mkfs.ext4 /dev/sda1
        mke2fs 1.45.5 (07-Jan-2020)
        Discarding device blocks: done
        Creating filesystem with 131071744 4k blocks and 32768000 inodes
        Filesystem UUID: d0e1aa64-5257-4669-ac4a-1610f72c48f5
        Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968,
        102400000

        Allocating group tables: done                            
        Writing inode tables: done                            
        Creating journal (262144 blocks): 
        done
        Writing superblocks and filesystem accounting information: done

* Check UUID for sda1 partition.

      $ sudo blkid 
        /dev/sdb1: LABEL="cloudimg-rootfs" UUID="b201084f-154f-4029-a5b8-f903ac9fc29b" TYPE="ext4" PARTUUID="341f0b90-b764-4a16-a222-6380a3a474f1"
        /dev/sdb15: LABEL_FATBOOT="UEFI" LABEL="UEFI" UUID="8FF9-31A2" TYPE="vfat" PARTUUID="c9c611b7-af79-407d-9631-919507f6d9b4"
        /dev/sdc1: UUID="6ebfc4f9-ceee-40c7-a4ed-ef396520232c" TYPE="ext4" PARTUUID="285dbfcc-01"
        /dev/loop0: TYPE="squashfs"
        /dev/loop1: TYPE="squashfs"
        /dev/loop2: TYPE="squashfs"
        /dev/loop3: TYPE="squashfs"
        /dev/sda1: UUID="d0e1aa64-5257-4669-ac4a-1610f72c48f5" TYPE="ext4" PARTUUID="fc094452-01"
        /dev/sdb14: PARTUUID="0d7407bf-e546-4806-a857-7f604d64c83c"

* Edit /etc/fstab to mount sda1 as /home/your_username/.near  
  Add line in /etc/fstab (please use UUID for your device) and reboot node

      $ sudo nano /etc/fstab

        UUID="d0e1aa64-5257-4669-ac4a-1610f72c48f5"       /home/your_username/.near  ext4 defaults  0 0

      $ sudo reboot now

* Connect/SSH to node using public IP address and confirm that 500GB partition is mapped to /home/your_username/.near

  Device name can change, but it is mapped by UUID.

      $ lsblk
        NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
        loop0     7:0    0 67.8M  1 loop /snap/lxd/22753
        loop1     7:1    0 61.9M  1 loop /snap/core20/1518
        loop2     7:2    0   47M  1 loop /snap/snapd/16292
        loop3     7:3    0   62M  1 loop /snap/core20/1581
        sda       8:0    0   30G  0 disk 
        ├─sda1    8:1    0 29.9G  0 part /
        ├─sda14   8:14   0    4M  0 part 
        └─sda15   8:15   0  106M  0 part /boot/efi
        sdb       8:16   0   16G  0 disk 
        └─sdb1    8:17   0   16G  0 part /mnt
        sdc       8:32   0  500G  0 disk 
        └─sdc1    8:33   0  500G  0 part /home/your_username/.near

Change ownership for /home/andy/.near to your account:

        $ sudo chown your_username:your_username /home/your_username/.near

___

## Step 5 -  Provisioning near node using standard procedures

  Update Linux

      $ sudo apt update && sudo apt upgrade -y

  Install developer tools Node.js and npm, confirm versions:

      $ curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -  
      $ sudo apt install build-essential nodejs
      $ PATH="$PATH"

      $ node -v
      v18.6.0

      $ npm -v
      8.14.0

  Install NEAR-CLI

      $ sudo npm install -g near-cli

  Configure Environment to work with shardnet

      $ echo 'export NEAR_ENV=shardnet' >> ~/.bashrc
      $ source ~/.bashrc

  Test Near-cli using following commands:

      $ near proposals
      $ naar validators current
      $ near validators next

  Install required software & set the configuration for Near node:

      $ sudo apt install -y git binutils-dev libcurl4-openssl-dev zlib1g-dev libdw-dev libiberty-dev cmake gcc g++ python docker.io protobuf-compiler libssl-dev pkg-config clang llvm cargo

  Install Python pip:

      $ sudo apt install python3-pip

  Set configuration:

      $ USER_BASE_BIN=$(python3 -m site --user-base)/bin
      $ export PATH="$USER_BASE_BIN:$PATH"

  Install Building env

      $ sudo apt install clang build-essential make
  
  Install Rust & Cargo, please select option 1

      $ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

![Rust_and_Cargo](https://github.com/near/stakewars-iii/blob/main/challenges/images/rust.png?raw=true)

  Source the environment

      $ source $HOME/.cargo/env
  
### Clone nearcore project from GitHub

      $ git clone https://github.com/near/nearcore
      $ cd nearcore
      $ git fetch

Checkout to the commit needed. Please refer to the commit defined in [this file](https://github.com/near/stakewars-iii/blob/main/commit.md  "title text!")

      $ git checkout <commit>

### Compile nearcore binary

(Need to be in nearcore folder) 

      $ cargo build -p neard --release --features shardnet

Initialize working directory

    $ ./target/release/neard --home ~/.near init --chain-id shardnet --download-genesis

![Initializing](https://github.com/near/stakewars-iii/blob/main/challenges/images/initialize.png?raw=true)

This command will create the directory structure and will generate config.json, node_key.json, and genesis.json on the network you have passed.

* config.json - Configuration parameters which are responsive for how the node will work. The config.json contains needed information for a node to run on the network, how to communicate with peers, and how to reach consensus. Although some options are configurable. In general validators have opted to use the default config.json provided.

* genesis.json - A file with all the data the network started with at genesis. This contains initial accounts, contracts, access keys, and other records which represents the initial state of the blockchain. The genesis.json file is a snapshot of the network state at a point in time. In contacts accounts, balances, active validators, and other information about the network.

* node_key.json - A file which contains a public and private key for the node. Also includes an optional account_id parameter which is required to run a validator node (not covered in this doc).

* data/ - A folder in which a NEAR node will write it's state.

### Replace the config.json

From the generated config.json, there two parameters to modify:

* boot_nodes: If you had not specify the boot nodes to use during init in Step 3, the generated config.json shows an empty array, so we will need to replace it with a full one specifying the boot nodes.
* tracked_shards: In the generated config.json, this field is an empty. You will have to replace it to "tracked_shards": [0]

      $ rm ~/.near/config.json
      $ wget -O ~/.near/config.json https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/config.json
  
### Get latest snapshot

Install AWS Cli

      $ sudo apt-get install awscli -y

Download snapshot (genesis.json)

      $ cd ~/.near
      $ wget https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/genesis.json

### Run the node

To start your node simply run the following command:

      $cd ~/nearcore
      $./target/release/neard --home ~/.near run

![Start_neard](https://github.com/near/stakewars-iii/blob/main/challenges/images/download.png?raw=true)

___

## Step 6 -  Activating the node as validator

### Authorize Wallet

A full access key needs to be installed locally to be able to sign transactions via NEAR-CLI.
Since this deployment of the near node doesn't have graphical interface installed on this server, we need to authorize wallet on remote system with GUI and import wallet key to this server. You can use your primary Windows or Mac computer to install your wallet key. You need to install near-cli on your computer first, same way we installed it for this deployment.


### On your primary computer run

    $ near login

![AImport_Wallet](https://github.com/akashkov/Near-Validator-at-Azure-/blob/main/Import_wallet.png?raw=true)

WEB browser will open a new window. 
Select if you want to import existing account or create a new one (In this example, a new account will be created)

![Create a New](https://github.com/akashkov/Near-Validator-at-Azure-/blob/main/Create_New_or_Import.png?raw=true)

Enter a name for new Wallet

![Reserve_Name](https://github.com/akashkov/Near-Validator-at-Azure-/blob/main/Reserve_name.png?raw=true)

Reserve a name for you Wallet and complete a process. Keep your secure passphrase in safe place.

![Generate_Passphrase](https://github.com/akashkov/Near-Validator-at-Azure-/blob/main/create_wallet.png?raw=true)

Connect your new Wallet to you computer.

![Generate_Passphrase](https://github.com/akashkov/Near-Validator-at-Azure-/blob/main/Connect_Account.png?raw=true)

Confirm your new Wallet full account access.

![Confirm_Access](https://github.com/akashkov/Near-Validator-at-Azure-/blob/main/Confirm_import.png?raw=true)

Close confirmation window to finish.

![Confirm_Access](https://github.com/akashkov/Near-Validator-at-Azure-/blob/main/Close_Import.png?raw=true)

At this pont mynew-wallet.shardnet.near.json was created in a folder ~/.near-credentials/shardnet/ at your computer. 
You need to copy content of this file from you computer to server in Azure.

      $ cat ~/.near-credentials/shardnet/mynew-wallet.shardnet.near.json
        { "account_id":"mynew-wallet.shardnet.near","public_key":"ed25519:YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY","private_key":"ed25519:XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"}                                            

On node in Azure:

      $ mkdir -p ~/.near-credentials/shardnet
      $ touch ~/.near-credentials/shardnet/mynew-wallet.shardnet.near.json
      $ nano ~/.near-credentials/shardnet/mynew-wallet.shardnet.near.json

Edit file by coping content of the file from your computer.

### Create a validator_key.json

* Generate the Key file:

      $ near generate-key <pool_id>

<pool_id> ---> xx.factory.shardnet.near WHERE xx is you pool name

* Copy the file generated to shardnet folder: Make sure to replace <pool_id> by your accountId

      $ cp ~/.near-credentials/shardnet/YOUR_WALLET.json ~/.near/validator_key.json

* Edit “account_id” => xx.factory.shardnet.near, where xx is your PoolName
* Change private_key to secret_key

Note: The account_id must match the staking pool contract name or you will not be able to sign blocks.\

File content must be in the following pattern:

      {
        "account_id": "xx.factory.shardnet.near",
        "public_key": "ed25519:HeaBJ3xLgvZacQWmEctTeUqyfSU4SDEnEwckWxd92W2G",
        "secret_key": "ed25519:****"
      }

### Configure neard run as service

      $ sudo vi /etc/systemd/system/neard.service

### Paste

    Unit]
    Description=NEARd Daemon Service

    [Service]
    Type=simple
    User=<USER>
    #Group=near
    WorkingDirectory=/home/<USER>/.near
    ExecStart=/home/<USER>/nearcore/target/release/neard run
    Restart=on-failure
    RestartSec=30
    KillSignal=SIGINT
    TimeoutStopSec=45
    KillMode=mixed

    [Install]
    WantedBy=multi-user.target

>Note: Change USER to your username

Register and start neard service:

      $ sudo systemctl enable neard
      $ sudo systemctl start neard

Watch logs with command:

      $ journalctl -n 100 -f -u neard


## Step 7 -  Mounting a staking pool

### Deploy a Staking Pool Contract

Calls the staking pool factory, creates a new staking pool with the specified name, and deploys it to the indicated accountId.

      $ near call factory.shardnet.near create_staking_pool '{"staking_pool_id": "<pool id>", "owner_id": "<accountId>", "stake_public_key": "<public key>", "reward_fee_fraction": {"numerator": 5, "denominator": 100}, "code_hash":"DD428g9eqLL8fWUxv8QSpVFzyHi1Qd16P8ephYCTmMSZ"}' --accountId="<accountId>" --amount=30 --gas=300000000000000

From the example above, you need to replace:

  * Pool ID: Staking pool name, the factory automatically adds its name to this parameter, creating {pool_id}.{staking_pool_factory}.
    It is important that {pool_id}.{staking_pool_factory} would much account_id in validator_key.json file. For ensample if account_id in
    validator_key.json file is stakewars.factory.shardnet.near, your should replace "pool id" with "stakewars".

  * Owner ID: The SHARDNET account (i.e. stakewares.shardnet.near) that will manage the staking pool.

  * Public Key: The public key in your validator_key.json file.

  * 5: The fee the pool will charge (e.g. in this case 5 over 100 is 5% of fees).

  * Account Id: The SHARDNET account deploying and signing the mount tx. Usually the same as the Owner ID.

You should see result of the staking pool creation, that includes HTTP link. Check result in browser, it should has "True" at the End.

Your staking pool has to have sufficient number of near in order your node became validator. We created pool with only 30 near.
Use following commands to confirm Validators set price (minimum amount of near required to became validator), and to add 200 additional nears to pool.

      $ near validators current | grep "seat price"
        Validators (total: 325, seat price: 160):
      $ near call <staking_pool_id> deposit_and_stake --amount 200 --accountId <accountId> --gas=300000000000000

In this example we need to stake at least 130 Nears to much seat price of 160.

You have now configure your Staking pool.

Check your pool is now visible on https://explorer.shardnet.near.org/nodes/validators

The following commands can be used to work with you pool

  Check your account total balance

      $ near view <staking_pool_id> get_account_total_balance '{"account_id": "<accountId>"}'

  Check your account staked balance

      $ near view <staking_pool_id> get_account_staked_balance '{"account_id": "<accountId>"}'

  Check your account unstaked balance

      $ near view <staking_pool_id> get_account_unstaked_balance '{"account_id": "<accountId>"}'

  Check Available for Withdrawal. You can only withdraw funds from a contract if they are unlocked.

      $ near call <staking_pool_id> pause_staking '{}' --accountId <accountId>


Now when your pool has sufficient stake(more than seat price) we can run Ping command.

A ping issues a new proposal and updates the staking balances for your delegators. A ping should be issued each epoch to keep reported rewards current.

      $ near call <staking_pool_id> ping '{}' --accountId <accountId> --gas=300000000000000

You should see HTTP link in result of this command.
Additionally you can run near proposal command with filter for your pool name.
Result should include Proposal(Accepted) for yourpool.factory.shardnet.near  

      $ near proposals | grep yourpool
      | Proposal(Accepted) | yourpool.factory.shardnet.near         | 230                | 1       


