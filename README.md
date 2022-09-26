# Cosmos Node Setup

This repo explains how to set up [Cosmos](https://cosmos.network/) nodes. We created it because there are zero comprehensive articles, videos, etc. on doing this, and the official docs on how to just set up a node is confusing.

This repo is written for people who know nothing about running a Cosmos node. While concepts specific to operating Cosmos nodes (e.g., which AWS services to use, how to connect the node to Prometheus) are explained, concepts specific to Cosmos (e.g., what is a blockchain), AWS (e.g., what is AWS EBS), etc. aren't explained. This is because the official docs are good, and you can look up only what you need to based on your experience level.

## [Concepts](docs/concepts.md)

## Hardware

Using a serverless solution (e.g., AWS Fargate) would've been more scalable since the primary purpose of an archive node is to serve API requests. However, since a blockchain node is a DB and API server combined, it's impossible to use a serverless solution. This is because a serverless solution requires the DB to be scaled separately from the API server. Since the blockchain combines the two, the DB would get corrupted because the API servers would all be writing to the DB, and the DB isn't engineered to handle duplicate writes.

Here's how to set up the hardware:
- We recommend using a Linux OS; Ubuntu specifically.
- Use an SSD (preferably NVMe) for storage.

    Here are node specific recommendations for storage requirements:
    - Juno mainnet archive node: 1 TiB
    - Validator node: 100 GiB
- This point only applies to archive nodes since validator nodes can quickly be restored via a snapshot or state sync.

    Since the DB can easily get corrupted, and syncing all over again can take days, it's highly recommended taking a daily backup of either the entire storage device (this way you'll have the correct version of the node installed for the backup's DB) or the `$DAEMON_HOME/data` directory (i.e., the DB). You should keep two backups because if you only keep one, and the backup takes place just after the DB gets corrupted, then the only backup will also be corrupted.
- The architecture must be x86_64.
- 16 GiB of RAM.
- Four 3.2 GHz CPU cores.

We recommend the following if you're using AWS:
- Use AWS EC2 for the computer.
- Use AWS EBS for storage.
- If you're running an archive node, and have synced the blocks yourself because there's no snapshot available, use AWS DLM to create a daily backup of the AWS EBS volume.

Set up the firewall:
- Incoming traffic:
    - In order to get SSH access to your server, allow SSH connections over TCP on port 22 from your IP address.
    - If you require that clients be allowed to make API calls, allow HTTP connections on port 80 from any IP address, and HTTPS connections on port 443 from any IP address.
    - If you're going to install [PANIC](https://github.com/SimplyVC/panic), then you must allow HTTPS connections on ports 3333 and 8000 from your IP address.
    - Similar to how your node retrieves data from other nodes, it's recommended to allow other nodes to sync with yours by allowing TCP connections on port 26656 from any IP address.
- Outgoing traffic: All outgoing traffic is fine.

If you're using something like AWS, then you can set the firewall without touching the CLI. Otherwise, here's how to set up the firewall using the CLI:

```shell
read -P 'Enter your IP address: ' IP_ADDRESS

read -P 'Enter y if you require SSH access, and n otherwise: ' REQUIRES_SSH
if test $REQUIRES_SSH = 'y'
    sudo ufw allow from $IP_ADDRESS proto tcp to any port 22
end
   
read -P 'Enter y if you require clients to be able to make API calls, and n otherwise: ' REQUIRES_API
if test $REQUIRES_API = 'y'
    sudo ufw allow http
    sudo ufw allow https
end
   
read -P 'Enter y if you\'re going to install PANIC, and n otherwise: ' WILL_INSTALL_PANIC
if test $WILL_INSTALL_PANIC = 'y'
    sudo ufw allow from $IP_ADDRESS proto tcp to any port 3333,8000
end
   
read -P 'Enter y if you want to allow other nodes to sync with yours, and n otherwise: ' REQUIRES_SYNCING
if test $REQUIRES_SYNCING = 'y'
    sudo ufw allow from any to any port 26656 proto tcp
end
   
sudo ufw enable
sudo ufw status
```

## [Software Setup](docs/software-setup.md)

## [Operating the Node](docs/operating.md)

## Credits

- [Juno Docs](https://docs.junonetwork.io/juno/readme)
- [Sei Docs](https://docs.seinetwork.io/introduction/overview)
- [Stride Docs](https://docs.stride.zone/docs)
- [Osmosis Docs](https://docs.osmosis.zone)
- [NodeJumper](https://nodejumper.io/)
- [Validator Security Checklist](https://docs.evmos.org/validators/security/checklist.html)

## License

This project is under the [MIT License](LICENSE).
