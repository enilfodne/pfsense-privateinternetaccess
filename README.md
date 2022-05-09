## Intro

Collection of scripts to allow:

- `pfSense` to connect to `PrivateInternetAccess` WireGuard servers
- Port-forwarding via `PrivateInternetAccess` API through, either, OpenVPN or a WireGuard tunnel

Notes:

- `PrivateInternetAccess` uses a gateway with `/16`, `/17` and `/18` masks in the `10.0.0.0/8` range.
Keep this in mind, in order to avoid routing conflicts
- You CANNOT specify an incoming port (for port-forwarding), a random port is assigned to you.

## Connect to a WireGuard server

1. Copy `privateinternetaccess-wireguard` to `/usr/local/bin/` on your `pfSense` machine

2. Copy the contents of `patches/wireguard.patch` and add them to `pfSense` via the Web interface (`System -> Patches`)

3. Create a WireGuard tunnel:
    - Set the tunnel description to `PrivateInternetAccess`
    - Set random port (to avoid conflicts)
    - Click `Generate` to create a public/private key-pair.
    - Click `Save`

4. Create WireGuard peer:
    - Set the tunnel, to the one you've just created or simply click `Add Peer`, while on the tunnel page.
    - Set the tunnel description to the region `id`, you want to connect to (you can get those, via the `vpninfo` API v6).
        - e.g. The tunnel description should be set to `nz`, when connecting to `New Zealand`
        - for full list of all valid `id`s, check `PrivateInternetAccess`'s `server_info` API
    - Uncheck `Dynamic` and enter random string for `Endpoint` and five digits for `Endpoint Port`
    - Set `Keep Alive` to `25`
    - Enter random string for `Public Key`
    - Under `Allowed IPs` add `0.0.0.0` with a netmask of `0` and click `Add Allowed IP`
    - Click `Save`

5. Create an interface
    - Go to `Interfaces`, select the newly created tunnel (`tun_wgX`) from the dropdown and click `Add`
    - Click on the newly added interface (should start with `OPT X`)
    - Check the `Enable interface` checkbox
    - Set the `IPv4 Configuration Type` to `Static IPv4`
    - Enter a random address from the `10.0.0.0/8` range as `IPv4 Address` and set a random netmask
    - Click `Add New Gateway` and set a random address within the range you set in the previous step as `Gateway IPv4`.
    - Click `Add` to save the gateway
    - Click `Save` to save the interface configuration
    - Click `Apply`

6. Setup `NAT` for your newly created interface
    - Go to `Firewal -> NAT`
    - Click `Outbound`, change `Mode` to `Hybrid` and click `Save`
    - Click `Add` on the bottom of the page
    - Set `Interface` as the interface, you've created for your WireGuard tunnel.
    - In `Source` set the network range/mask of the clients you wish to have access via the tunnel
    - Set `Address` under the `Translation` section to `Interface Address`
    - Click `Save`

7. Set your `PrivateInternetAccess` credentials
    - `ssh` into your `pfsense` machine
    - Select `8` to get dropped to a shell
    - Create (if you haven't already) the directory `/usr/local/etc/privateinternetaccess`
    - Use your favorite editor (`nano` is very user-friendly) to edit `/usr/local/etc/privateinternetaccess/credentials`
    - Inside the editor, enter your username on the first line and your password on the second.
    - Save and exit

8. Go to the `pfSense` Web interface, navigate to `VPN -> Wireguard` and restart the service

# Port-forwarding

1. Copy `privateinternetaccess-wireguard` to `/usr/local/bin/` on your `pfSense` machine

2. Setup your `devd` configuration
    - `ssh` into your `pfsense` machine
    - Select `8` to get dropped to a shell
    - Create (if you haven't already) the directory `/usr/local/etc/devd`
    - Use your favorite editor (`nano` is very user-friendly) to edit `/usr/local/etc/devd/privateinternetaccess.conf`
    - Copy the contents of the `devd/privateinternetaccess.conf` file to `/usr/local/etc/devd/privateinternetaccess.conf`
    - Change the `subsystem` interfaces to match your tunnel Interfaces
        - the interface names are `ovpncX` for `OpenVPN` and `tun_wgX` for `WireGuard`
    - Save and exit
    - Run `service devd restart` command and make sure there are no errors

3. Set port aliases for your clients
    - Navigate to `Firewall -> Aliases` and click `Ports`
    - Click `Add`
    - Choose a unique name
    - Set `Port` to a random 4-digit number, bigger than 1024
    - Click `Save`

4. Setup your client configuration
    - `ssh` into your `pfsense` machine
    - Select `8` to get dropped to a shell
    - Create (if you haven't already) the directory `/usr/local/etc/privateinternetaccess`
    - Use your favorite editor (`nano` is very user-friendly) to edit `/usr/local/etc/privateinternetaccess/portforward-settings.ini`
    - Copy the contents of the `etc/portforward-settings.ini` file to `/usr/local/etc/privateinternetaccess/portforward-settings.ini`
    - Change the different sections, according to the `ini` examples
    - Save and exit

5. Set your `PrivateInternetAccess` credentials
    - `ssh` into your `pfsense` machine
    - Select `8` to get dropped to a shell
    - Create (if you haven't already) the directory `/usr/local/etc/privateinternetaccess`
    - Use your favorite editor (`nano` is very user-friendly) to edit `/usr/local/etc/privateinternetaccess/credentials`
    - Inside the editor, enter your username on the first line and your password on the second.
    - Save and exit

6. Go to the `pfSense` Web interface, navigate to respective VPN type (`WireGuard` or `OpenVPN`) and restart the service.

