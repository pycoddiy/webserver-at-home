# Web Server at Home

This is a step-by-step guide how to set up a web server on your local Ubuntu machine.

## Configure your static IP address
In the terminal perform the following actions to configure your Wi-Fi internet connection using static IP.

1. Identify your Wi-Fi interface by typing `ip a` to list all network interfaces and their addresses
2. Look for interface corresponding to Wi-Fi connection, such as `wlp2s0`, `wlan0`, etc.
3. Find the netplan configuration file `cd /etc/netplan` and then `ls`. Look for a file with the name similar to `01-network-manager.yaml`. Open the file with any editor, e.g. `nano 01-network-manager.yaml`.
4. Edit the file according to the following template:

```yaml
network:
  version: 2
  renderer: NetworkManager
  wifis:
    wlp2s0:
      dhcp4: no
      dhcp6: no
      addresses: [10.0.0.155/24] # This is a local network static address
      gateway4: 10.0.0.1 # Gateway/Router address
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4] # Google's DNS servers to be used for name resolution
      access-points:
        "SpecifySSID": # Your Wi-Fi network SSID
          password: "EnterPASSWORD" # Password for Wi-Fi
```

5. Apply configuration by running `sudo netplan try`.
6. Ensure that your IP is not static by running `ip a` again.

## Set up port forwarding for your router
This section covers necessary steps to set up port forwarding so that your web server is accessible from outside world. This is mainly a sketch because exact steps depend on your router model and its software version.

1. Connect to your router by typing `10.0.0.1`. Use credetails to log in to the router.
2. Find router's external IP address given by your ISP. You can see this address by visiting `whatismyip.com`.
3. In settings (typically advanced settings or alike) find the **Port Forwarding** section and add new port forwarding rule. For web server select **Service Type** as `HTTP`, set **External Start Port** and **Internal Start Port** to `80`, select internal IP address `10.0.0.155` as in the above YAML file.
4. Log out and save your settings.

## Install Nginx server
It is up to you what server to use. This section focuses on the installation of Nginx web server, one of the fastest and the most robust web servers.

1. In terminal run `sudo apt install nginx`.
2. Open browser and type `localhost` to ensure that Nginx successfully installed and is running its default webpage **Welcome to nginx!**. This page can be found at `/var/www/html`.
3. Let us create a welcome page to be located at `/var/www/pycoddiy.com`. Run `cd /var/www`, run `sudo mkdir pycoddiy.com`.
4. Copy `index.html` located in this repository to the target location `/var/www/pycoddiy.com`
5. Locate the Nginx enabled sites directory `cd /etc/nginx/sites-enabled` and create the new configuration file `pycoddiy.com.conf` with the following content:

```nginx
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/pycoddiy.com;

        index index.html index.htm index.nginx-debian.html;

        server_name pycoddiy.com;

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
        }
}
```

6. Restart Nginx service `sudo service nginx restart`
7. Check server status `sudo service nginx status`
8. Open browser and enter `10.0.0.155` to make sure the web page is successfully displayed.
9. Check the same by entering external IP address in the the browser.
10. Additionally check if the server works from another device running on a different network by entering external IP address in the web browser.

## Registering web server with domain provider
My provider is NameCheap.com and the following instruction is how to register the web server we created within NameCheap.com DNS server.

1. Log into `namecheap.com` account and in the Dashboard gselect the domain for which you want to add **A Record**. Click **Manage** button and then navigate to the **Advanced DNS** tab.
2. In the host records section click **Add New Record** button.
3. Select **A Record** in the type, then `@` in the host, and then add your external IP address.
4. Repeat by adding another record with the same values as in previous step except `www` in the host.
5. Repeat the same with `*` in the host.
6. Wait for 30-60 min for changes to propagate.

# Configuring VPN Server at Home
This bonus section will guide you through the process of installation of OpenVPN as a VPN server.

## Creating OpenVPN client profile
1. In the home directory download the installation script `wget https://git.io/vpn -O openvpn-install.sh`
2. Allow the script to run `sudo chmod +x openvpn-install.sh`
3. Execute `sudo bash openvpn-install.sh`. Select recommended choices.
4. Note the port `1194`.
5. Name the client, e.g. `client_my_phone`
6. Press any key to start the client profile initialization. This will create the respective file `client_my_phone.ovpn` in your current directory.

## Set up port forwarding in the router

This section covers necessary steps to set up port forwarding so that your VPN server is accessible from outside world. This is mainly a sketch because exact steps depend on your router model and its software version.

1. Connect to your router by typing `10.0.0.1`. Use credetails to log in to the router.
2. In settings (typically advanced settings or alike) find the **Port Forwarding** section and add new port forwarding rule. For web server select **Service Type** as `VPN-PPTP`, set **External Start Port** and **Internal Start Port** to `1194`, select internal IP address `10.0.0.155`. Select `UDP` as a protocol.
3. Log out and save your settings.
4. Transfer client profile to the target device, such as Android phone.
5. Install OpenVPN client there.
6. Upload client profile by clickling **Upload File**.

If you need more clients to be connected, generate new `ovpn` files by running the shell script `openvpn-install.sh`

# Configuring HTTPS for Nginx
For enhanced security HTTPS protocols has become a de-facto standard in web site hosting. This section addresses key steps to turn HTTP site to HTTPS.

1. Create directory where SSL certificates will be stored `sudo mkdir /etc/nginx/ssl`.
2. Remove all permissions for group and world `sudo chmod 700 /etc/nginx/ssl`.
3. Generate private RSA key `cd /etc/ssl` and `sudo openssl genrsa --out pycoddiy.com-private.key 2048` 
4. Create CSR file `sudo openssl req -new -key pycoddiy.com-private.key -out pycoddiy.com-private.csr`. Follow prompts and answer questions.
5. Copy the content of the `csr` file `cat pycoddiy.com-private.csr` for submission to a Certificate Authority (CA). 
6. I used NameCheap.com to issues a Certificate Authority (CA). The process is fairly similar to any other entity. Simply follow the instructions provided there. 
7. Once the certificate is issued, you download it to the server. **NOTE: NameCheap.com provides the certificate in the zip file. You have to unzip it and then combine `pycoddiy_com.ca-bundle` and `pycoddiy_com.crt` files into the chained certificate file, i.e. `pycoddiy_com_chain.crt`** 
8. Copy chained certificate to the server `/etc/nginx/ssl/pycoddiy_com_chain.crt`.
8. Edit Nginx config `/etc/nginx/sites-enabled/pycoddiy.com.conf`:

```nginx
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        server_name pycoddiy.com;

        return 301 https://$server_name$request_uri;
}

server {
        listen 443 ssl;

        server_name pycoddiy.com

        ssl_certificate /etc/nginx/ssl/pycoddiy_com_chain.crt
        ssl_certificate_key /etc/ssl/pycoddiy.com-private.key

        location / {
                try_files $uri $uri/ =404;

                root /var/www/pycoddiy.com;

                index index.html index.htm;
        }
}
```

9. Restart Nginx service `sudo service nginx restart` and `sudo service nginx status`.
10. If you get **Connection Refused** error, it is likely your SSL traffic is filtered by your router. Add another port forwarding rule for `10.0.0.155` with an open port `443`