# Algo VPN - Pihole Fork

[![Build Status](https://travis-ci.com/rodeodomino/algo-pihole.svg?branch=master)](https://travis-ci.com/rodeodomino/algo-pihole)

This is a fork of [@trailofbits](https://github.com/trailofbits) amazing work with [Algo](https://github.com/trailofbits/algo), replacing their local dnsmasq ad blocking with Pi-hole with [@dan-v](https://github.com/dan-v)'s [Pi-hole](https://github.com/dan-v/algo) code.

In addition to installing Pi-hole, it:

* Automatically sets up [@mmotti](https://github.com/mmotti)'s [Fetch Filter Lists](https://github.com/mmotti/pihole-filter-lists) and [Gravity Optimise](https://github.com/mmotti/pihole-gravity-optimise) scripts
* Checks for Pi-hole updates
* Includes a prebuilt whitelist, blacklist, regex, and adlist (available for you to edit) based off of [@WaLLy3K](https://v.firebog.net/hosts/lists.php) and [@mmotti](https://github.com/mmotti)'s work.
* Creates extra users for full-tunneling and split-tunneling
* Includes a new port to try to guard against VPN blocking

For split tunnel, simply replace `0.0.0.0/0, ::0/0` with `172.16.0.1/32, fcaa::1/64` for the `AllowedIPs` in the Wireguard .conf file.

Split tunnel caveats:

* This appears to work, but I need your help to confirm.
* I would like to automate the .conf file generation to track for a word in the user name and adjust the `AllowedIPs` accordingly (e.g., user phone-full would have `AllowedIPs` of `0.0.0.0/0` while `phone-pihole` would have `172.16.0.1/32`). If you know how to do that, please let me know.
* I cannot figure out how to do this for IPSEC, so only working on Wireguard for now.

----------------

[![Join the chat at https://gitter.im/trailofbits/algo](https://badges.gitter.im/trailofbits/algo.svg)](https://gitter.im/trailofbits/algo?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
[![Twitter](https://img.shields.io/twitter/url/https/twitter.com/fold_left.svg?style=social&label=Follow%20%40AlgoVPN)](https://twitter.com/AlgoVPN)

Algo VPN is a set of Ansible scripts that simplify the setup of a personal IPSEC and Wireguard VPN. It uses the most secure defaults available, works with common cloud providers, and does not require client software on most devices. See our [release announcement](https://blog.trailofbits.com/2016/12/12/meet-algo-the-vpn-that-works/) for more information.

## Features

* Supports only IKEv2 with strong crypto (AES-GCM, SHA2, and P-256) and [WireGuard](https://www.wireguard.com/)
* Generates Apple profiles to auto-configure iOS and macOS devices
* Includes a helper script to add and remove users
* Blocks ads with a local DNS resolver (optional)
* Sets up limited SSH users for tunneling traffic (optional)
* Based on current versions of Ubuntu and strongSwan
* Installs to DigitalOcean, Amazon Lightsail, Amazon EC2, Vultr, Microsoft Azure, Google Compute Engine, Scaleway, OpenStack, or your own Ubuntu 18.04 LTS server

## Anti-features

* Does not support legacy cipher suites or protocols like L2TP, IKEv1, or RSA
* Does not install Tor, OpenVPN, or other risky servers
* Does not depend on the security of [TLS](https://tools.ietf.org/html/rfc7457)
* Does not require client software on most platforms
* Does not claim to provide anonymity or censorship avoidance
* Does not claim to protect you from the [FSB](https://en.wikipedia.org/wiki/Federal_Security_Service), [MSS](https://en.wikipedia.org/wiki/Ministry_of_State_Security_(China)), [DGSE](https://en.wikipedia.org/wiki/Directorate-General_for_External_Security), or [FSM](https://en.wikipedia.org/wiki/Flying_Spaghetti_Monster)

## Deploy the Algo Server

The easiest way to get an Algo server running is to let it set up a _new_ virtual machine in the cloud for you.

1. **Setup an account on a cloud hosting provider.** Algo supports [DigitalOcean](https://m.do.co/c/4d7f4ff9cfe4) (most user friendly), [Amazon Lightsail](https://aws.amazon.com/lightsail/), [Amazon EC2](https://aws.amazon.com/), [Vultr](https://www.vultr.com/), [Microsoft Azure](https://azure.microsoft.com/), [Google Compute Engine](https://cloud.google.com/compute/), [Scaleway](https://www.scaleway.com/), and [DreamCompute](https://www.dreamhost.com/cloud/computing/) or other OpenStack-based cloud hosting.

2. **[Download Algo](https://github.com/rodeodomino/algo-pihole/archive/master.zip).** Unzip it in a convenient location on your local machine.

3. **Install Algo's core dependencies.** Open the Terminal. The `python` interpreter you use to deploy Algo must be python2. If you don't know what this means, you're probably fine. `cd` into the `algo-master` directory where you unzipped Algo, then run:

    - macOS:
      ```bash
      $ python -m ensurepip --user
      $ python -m pip install --user --upgrade virtualenv
      ```
    - Linux (deb-based):
      ```bash
      $ sudo apt-get update && sudo apt-get install \
          build-essential \
          libssl-dev \
          libffi-dev \
          python-dev \
          python-pip \
          python-setuptools \
          python-virtualenv -y
      ```
     - Linux (rpm-based): See the [Pre-Install Documentation for RedHat/CentOS 6.x](docs/deploy-from-redhat-centos6.md)
     - Windows: See the [Windows documentation](docs/deploy-from-windows.md)

4. **Install Algo's remaining dependencies.** Use the same Terminal window as the previous step and run:
    ```bash
    $ python -m virtualenv --python=`which python2` env &&
        source env/bin/activate &&
        python -m pip install -U pip virtualenv &&
        python -m pip install -r requirements.txt
    ```
    On macOS, you may be prompted to install `cc`. You should press accept if so.

5. **List the users to create.** Open `config.cfg` in your favorite text editor. Specify the users you wish to create in the `users` list.

6. **Start the deployment.** Return to your terminal. In the Algo directory, run `./algo` and follow the instructions. There are several optional features available. None are required for a fully functional VPN server. These optional features are described in greater detail in [deploy-from-ansible.md](docs/deploy-from-ansible.md).

That's it! You will get the message below when the server deployment process completes. You now have an Algo server on the internet. Take note of the p12 (user certificate) password in case you need it later, **it will only be displayed this time**.

You can now setup clients to connect it, e.g. your iPhone or laptop. Proceed to [Configure the VPN Clients](#configure-the-vpn-clients) below.

```
        "\"#----------------------------------------------------------------------#\"",
        "\"#                          Congratulations!                            #\"",
        "\"#                     Your Algo server is running.                     #\"",
        "\"#    Config files and certificates are in the ./configs/ directory.    #\"",
        "\"#              Go to https://whoer.net/ after connecting               #\"",
        "\"#        and ensure that all your traffic passes through the VPN.      #\"",
        "\"#                    Local DNS resolver 172.16.0.1                     #\"",
        "\"#                The p12 and SSH keys password is XXXXXXXX             #\"",
        "\"#----------------------------------------------------------------------#\"",
```

## Configure the VPN Clients

Certificates and configuration files that users will need are placed in the `configs` directory. Make sure to secure these files since many contain private keys. All files are saved under a subdirectory named with the IP address of your new Algo VPN server.

### Apple Devices

WireGuard is used to provide VPN services on Apple devices. Algo generates a WireGuard configuration file, `wireguard/<username>.conf`, and a QR code, `wireguard/<username>.png`, for each user defined in `config.cfg`.

On iOS, install the [WireGuard](https://itunes.apple.com/us/app/wireguard/id1441195209?mt=8) app from the iOS App Store. Then, use the WireGuard app to scan the QR code or AirDrop the configuration file to the device.

On macOS Mojave or later, install the [WireGuard](https://itunes.apple.com/us/app/wireguard/id1451685025?mt=12) app from the Mac App Store. WireGuard will appear in the menu bar once you run the app. Click on the WireGuard icon, choose **Import tunnel(s) from file...**, then select the appropriate WireGuard configuration file.

On either iOS or macOS, you can enable "Connect on Demand" and/or exclude certain trusted Wi-Fi networks (such as your home or work) by editing the tunnel configuration in the WireGuard app. (Algo can't do this automatically for you.)

Installing WireGuard is a little more complicated on older version of macOS. See [Using macOS as a Client with WireGuard](docs/client-macos-wireguard.md).

If you prefer to use the built-in IPSEC VPN on Apple devices, or need "Connect on Demand" or excluded Wi-Fi networks automatically configured, then see [Using Apple Devices as a Client with IPSEC](docs/client-apple-ipsec.md).

### Android Devices

WireGuard is used to provide VPN services on Android. Install the [WireGuard VPN Client](https://play.google.com/store/apps/details?id=com.wireguard.android). Import the corresponding `wireguard/<name>.conf` file to your device, then setup a new connection with it. See the [Android setup instructions](/docs/client-android.md) for more detailed walkthrough.

### Windows 10

Copy your PowerShell script `windows_{username}.ps1` to the Windows client and run the following command as Administrator to configure the VPN connection.
```
powershell -ExecutionPolicy ByPass -File windows_{username}.ps1 -Add
```

For a manual installation, see the [Windows setup instructions](/docs/client-windows.md).

### Linux Network Manager Clients (e.g., Ubuntu, Debian, or Fedora Desktop)

Network Manager does not support AES-GCM. In order to support Linux Desktop clients, choose the "compatible" cryptography during the deploy process and use at least Network Manager 1.4.1. See [Issue #263](https://github.com/trailofbits/algo/issues/263) for more information.

### Linux strongSwan Clients (e.g., OpenWRT, Ubuntu Server, etc.)

Install strongSwan, then copy the included ipsec_user.conf, ipsec_user.secrets, user.crt (user certificate), and user.key (private key) files to your client device. These will require customization based on your exact use case. These files were originally generated with a point-to-point OpenWRT-based VPN in mind.

#### Ubuntu Server 18.04 example

1. `sudo apt-get install strongswan libstrongswan-standard-plugins`: install strongSwan
2. `/etc/ipsec.d/certs`: copy `<name>.crt` from `algo-master/configs/<server_ip>/ipsec/manual/<name>.crt`
3. `/etc/ipsec.d/private`: copy `<name>.key` from `algo-master/configs/<server_ip>/ipsec/manual/<name>.key`
4. `/etc/ipsec.d/cacerts`: copy `cacert.pem` from `algo-master/configs/<server_ip>/ipsec/manual/cacert.pem`
5. `/etc/ipsec.secrets`: add your `user.key` to the list, e.g. `<server_ip> : ECDSA <name>.key`
6. `/etc/ipsec.conf`: add the connection from `ipsec_user.conf` and ensure `leftcert` matches the `<name>.crt` filename
7. `sudo ipsec restart`: pick up config changes
8. `sudo ipsec up <conn-name>`: start the ipsec tunnel
9. `sudo ipsec down <conn-name>`: shutdown the ipsec tunnel

One common use case is to let your server access your local LAN without going through the VPN. Set up a passthrough connection by adding the following to `/etc/ipsec.conf`:

    conn lan-passthrough
    leftsubnet=192.168.1.1/24 # Replace with your LAN subnet
    rightsubnet=192.168.1.1/24 # Replace with your LAN subnet
    authby=never # No authentication necessary
    type=pass # passthrough
    auto=route # no need to ipsec up lan-passthrough

To configure the connection to come up at boot time replace `auto=add` with `auto=start`.

### Other Devices

Depending on the platform, you may need one or multiple of the following files.

* cacert.pem: CA Certificate
* user.mobileconfig: Apple Profile
* user.p12: User Certificate and Private Key (in PKCS#12 format)
* ipsec_user.conf: strongSwan client configuration
* ipsec_user.secrets: strongSwan client configuration
* windows_user.ps1: Powershell script to help setup a VPN connection on Windows

## Setup an SSH Tunnel

If you turned on the optional SSH tunneling role, then local user accounts will be created for each user in `config.cfg` and SSH authorized_key files for them will be in the `configs` directory (user.ssh.pem). SSH user accounts do not have shell access, cannot authenticate with a password, and only have limited tunneling options (e.g., `ssh -N` is required). This ensures that SSH users have the least access required to setup a tunnel and can perform no other actions on the Algo server.

Use the example command below to start an SSH tunnel by replacing `user` and `ip` with your own. Once the tunnel is setup, you can configure a browser or other application to use 127.0.0.1:1080 as a SOCKS proxy to route traffic through the Algo server.

 `ssh -D 127.0.0.1:1080 -f -q -C -N user@ip -i configs/<server_ip>/ssh-tunnel/<user>.pem`

## SSH into Algo Server

To SSH into the Algo server for administrative purposes you can use the example command below by replacing `ip` with your own:

 `ssh root@ip -i ~/.ssh/algo.pem`

If you find yourself regularly logging into Algo then it will be useful to load your Algo ssh key automatically. Add the following snippet to the bottom of `~/.bash_profile` to add it to your shell environment permanently.

 `ssh-add ~/.ssh/algo > /dev/null 2>&1`

Note the admin username is `ubuntu` instead of `root` on providers other than Digital Ocean.

## Adding or Removing Users

If you chose to save the CA certificate during the deploy process, then Algo's own scripts can easily add and remove users from the VPN server.

1. Update the `users` list in your `config.cfg`
2. Open a terminal, `cd` to the algo directory, and activate the virtual environment with `source env/bin/activate`
3. Run the command: `./algo update-users`

After this process completes, the Algo VPN server will contain only the users listed in the `config.cfg` file.

## Additional Documentation

* Setup instructions
  - Documentation for available [Ansible roles](docs/setup-roles.md)
  - Deploy from [Fedora Workstation (26)](docs/deploy-from-fedora-workstation.md)
  - Deploy from [RedHat/CentOS 6.x](docs/deploy-from-redhat-centos6.md)
  - Deploy from [Windows](docs/deploy-from-windows.md)
  - Deploy from [Ansible](docs/deploy-from-ansible.md) directly
* Client setup
  - Setup [Android](docs/client-android.md) clients
  - Setup [Generic/Linux](docs/client-linux.md) clients with Ansible
  - Setup Ubuntu clients to use [WireGuard](docs/client-linux-wireguard.md)
  - Setup Apple devices to use [IPSEC](docs/client-apple-ipsec.md)
* Cloud setup
  - Configure [Amazon EC2](docs/cloud-amazon-ec2.md)
  - Configure [Azure](docs/cloud-azure.md)
  - Configure [DigitalOcean](docs/cloud-do.md)
  - Configure [Google Cloud Platform](docs/cloud-gce.md)
  - Configure [Vultr](docs/cloud-vultr.md)
* Advanced Deployment
  - Deploy to your own [FreeBSD](docs/deploy-to-freebsd.md) server
  - Deploy to your own [Ubuntu 18.04](docs/deploy-to-ubuntu.md) server
  - Deploy to an [unsupported cloud provider](docs/deploy-to-unsupported-cloud.md)
* [FAQ](docs/faq.md)
* [Troubleshooting](docs/troubleshooting.md)

If you read all the documentation and have further questions, [join the chat on Gitter](https://gitter.im/trailofbits/algo).

## Endorsements

> I've been ranting about the sorry state of VPN svcs for so long, probably about
> time to give a proper talk on the subject. TL;DR: use Algo.

-- [Kenn White](https://twitter.com/kennwhite/status/814166603587788800)

> Before picking a VPN provider/app, make sure you do some research
> https://research.csiro.au/ng/wp-content/uploads/sites/106/2016/08/paper-1.pdf ... – or consider Algo

-- [The Register](https://twitter.com/TheRegister/status/825076303657177088)

> Algo is really easy and secure.

-- [the grugq](https://twitter.com/thegrugq/status/786249040228786176)

> I played around with Algo VPN, a set of scripts that let you set up a VPN in the cloud in very little time, even if you don’t know much about development. I’ve got to say that I was quite impressed with Trail of Bits’ approach.

-- [Romain Dillet](https://twitter.com/romaindillet/status/851037243728965632) for [TechCrunch](https://techcrunch.com/2017/04/09/how-i-made-my-own-vpn-server-in-15-minutes/)

> If you’re uncomfortable shelling out the cash to an anonymous, random VPN provider, this is the best solution.

-- [Thorin Klosowski](https://twitter.com/kingthor) for [Lifehacker](http://lifehacker.com/how-to-set-up-your-own-completely-free-vpn-in-the-cloud-1794302432)

## Support Algo VPN
[![Flattr](https://button.flattr.com/flattr-badge-large.png)](https://flattr.com/submit/auto?fid=kxw60j&url=https%3A%2F%2Fgithub.com%2Ftrailofbits%2Falgo)
[![PayPal](https://www.paypalobjects.com/en_US/i/btn/btn_donate_SM.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=CYZZD39GXUJ3E)
[![Patreon](https://img.shields.io/badge/back_on-patreon-red.svg)](https://www.patreon.com/algovpn)
[![Bountysource](https://img.shields.io/bountysource/team/trailofbits/activity.svg)](https://www.bountysource.com/teams/trailofbits)

All donations support continued development. Thanks!

* We accept donations via [PayPal](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=CYZZD39GXUJ3E), [Patreon](https://www.patreon.com/algovpn), and [Flattr](https://flattr.com/submit/auto?fid=kxw60j&url=https%3A%2F%2Fgithub.com%2Ftrailofbits%2Falgo).
* Use our [referral code](https://m.do.co/c/4d7f4ff9cfe4) when you sign up to Digital Ocean for a $10 credit.
* We also accept and appreciate contributions of new code and bugfixes via Github Pull Requests.

Algo is licensed and distributed under the AGPLv3. If you want to distribute a closed-source modification or service based on Algo, then please consider <a href="mailto:opensource@trailofbits.com">purchasing an exception</a> . As with the methods above, this will help support continued development.
