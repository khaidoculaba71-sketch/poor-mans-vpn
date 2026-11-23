# poor-mans-vpn

For occasional VPN access, create a VM in the location you need and remove it when you're done. This Ansible playbook for WireGuard handles the server setup; you supply a client public key and configure the tunnel in your WireGuard app.

## Features

- Configure a WireGuard server on a VM over SSH.
- Register the client public key supplied at the prompt.
- Accept a client tunnel IP and WireGuard listen port, with defaults of `10.0.0.3` and `51820`.
- Route IPv4 traffic through the server with the client configuration below.

## Installation

Download or clone this project's source, then open its directory. The control machine needs Python, `virtualenv`, and SSH access to the server. Jinja templates are used through Ansible; there is no separate Jinja application to run.

Create the Python environment and install the dependencies:

```sh
virtualenv venv
source venv/bin/activate
pip install -r requirements.txt
deactivate
```

Create a VM in your preferred region. The donor project was tested on a DigitalOcean VM with Ubuntu 20.04 LTS; other images are not verified here.

Before running the playbook:

- Confirm that you can SSH into the VM with your default public key and an account with permission to configure the server.
- Allow inbound UDP traffic to the WireGuard listen port in your provider's firewall, if one is enabled.
- Install the WireGuard client app, create an empty tunnel, and copy its public key.

Keep the client private key on your client device.

## Usage

**Deactivate your WireGuard connection before running the playbook.**

From the project directory, activate the environment and run:

```sh
source venv/bin/activate
ansible-playbook -i <server public ip>, -u <server username> playbook.yaml
```

Replace the placeholders with your server's public IP and SSH username. For example, for an Ubuntu VM at the example address `203.0.113.10`:

```sh
ansible-playbook -i 203.0.113.10, -u ubuntu playbook.yaml
```

Keep the comma after the IP: it tells Ansible to treat the address as an inline inventory.

The playbook prompts for:

```text
Client public key: <copy from wireguard client app>
Allowed IPs [10.0.0.3]:
Wireguard listen port [51820]:
```

Press Enter to accept either default. Edit the tunnel in your WireGuard client to match:

```ini
[Interface]
PrivateKey = <auto generated for client>
Address = 10.0.0.3/24
DNS = 1.1.1.1, 1.0.0.1

[Peer]
PublicKey = <server wg public key / changes every time we run the playbook>
AllowedIPs = 0.0.0.0/0
Endpoint = <server public ip>:<wg port>
```

Use the server's WireGuard public key, not its SSH public key. If you changed the client IP or listen port at the prompts, update the corresponding client settings too. This example routes all IPv4 traffic through the tunnel; it does not configure IPv6 routing.

**Each playbook run changes the server's private and public keys.** Update the server public key in your client configuration after every run, then activate the tunnel.

When you finish, disconnect the tunnel and delete the VM through your provider if you no longer need it. The playbook does not handle VM deletion; check your provider's billing rules for any remaining resources.

## License

MIT. This project is based on [amritb/poor-mans-vpn](https://github.com/amritb/poor-mans-vpn). Retain the original MIT copyright and permission notice when redistributing the code; attribution here does not replace that notice.

## Related projects

The following repositories were supplied as other tools from the same author. Their functionality was not described in the available project information.

- [ProjectKangcutz/modelfusion](https://github.com/ProjectKangcutz/modelfusion): Check its README for its purpose and usage.
- [Sarthakkrishna/casskop-zoqe](https://github.com/Sarthakkrishna/casskop-zoqe): Check its README for its purpose and usage.
- [khuram-bti/taskomask](https://github.com/khuram-bti/taskomask): Check its README for its purpose and usage.

