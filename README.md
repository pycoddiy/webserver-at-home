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
5. Locate the Nginx enabled sites directory `cd /etc/nginx/sites-enabled` and create the new configuration file `pycoddiy` with the following content:

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