# Assigning Unique IP Addresses to Docker Containers on TrueNAS via Portainer

This guide will walk you through creating a `macvlan` network on your TrueNAS system, allowing your Docker containers to obtain unique IP addresses from your local network. You'll then learn how to deploy these containers using Portainer, ensuring they utilize this `macvlan` network, effectively giving them an IP address separate from your TrueNAS host.

## Prerequisites

*   **TrueNAS System:** Ensure your TrueNAS system is up and running.
*   **Docker Installed:** Docker must be installed and operational on your TrueNAS system.
*   **Portainer Installed:** Portainer should be installed and configured to manage your Docker environment on TrueNAS.
*   **Network Interface Name:** You need to know the name of the network interface that your TrueNAS system uses to connect to your local network (e.g., `enp3s0`, `eth0`).
*   **Available IP Range:** Identify an available subnet and gateway on your local network that is not currently being used by your TrueNAS host or other devices.

## Step 1: Create the `macvlan` Network on TrueNAS

The `macvlan` network driver creates a network where Docker containers directly appear as physical devices on your network, each with its own MAC and IP address. This step is performed directly on your TrueNAS system's shell, as Portainer currently make it possible to do it properly.

1.  **Open TrueNAS Shell:** Access the TrueNAS shell. You can do this via the TrueNAS web interface (`System > Shell`) or by SSHing into your TrueNAS system.

2.  **Execute the `docker network create` Command:**
    **Before running:** Replace the placeholder values with your specific network configuration:

    *   `--subnet=192.168.1.0/24`: Your local network's subnet (e.g., `192.168.1.0/24`).
    *   `--gateway=192.168.1.1`: Your network's gateway (router IP, e.g., `192.168.1.1`).
    *   `-o parent=enp3s0`: **Crucially, replace `enp3s0` with the actual name of your TrueNAS system's physical network interface.**

    ```bash
    docker network create -d macvlan \
          --subnet=192.168.1.0/24 \
          --gateway=192.168.1.1 \
          -o parent=enp3s0 macvlan-network
    ```

    *   **Explanation:**
        *   `docker network create -d macvlan`: Initializes a new network using the `macvlan` driver.
        *   `--subnet` and `--gateway`: Define the IP range and the router for the network.
        *   `-o parent=enp3s0`: This option ties the `macvlan` network to a specific physical interface on your TrueNAS system.
        *   `macvlan-network`: This is the chosen name for your new `macvlan` network. You can name it anything you like, but remember this name as you'll use it in Portainer.

3.  **Verify Network Creation (Optional):**
    To ensure the network was created, list your Docker networks:

    ```bash
    docker network ls
    ```

    You should see `macvlan-network` (or whatever you named it) listed in the output.

## Step 2: Deploy Containers via Portainer Using the `macvlan` Network

Now that the `macvlan` network is configured on Docker, you can instruct Portainer to deploy containers using this network.

1.  **Log in to Portainer:** Access your Portainer web interface.

2.  **Navigate to Networks:**
    *   In the left-hand sidebar, click on **Networks**.
    *   You should now see `macvlan-network` (or your chosen name) listed among the available networks. This confirms Portainer recognizes the network created in Step 1.

3.  **Add a Container (or Stack):**
    You can deploy a single container or use a stack (Docker Compose) to utilize this network.

    ### Option A: Deploying a Single Container

    1.  In the left-hand sidebar, click on **Containers**, then **Add container**.
    2.  Fill in the **Name** for your container (e.g., `my-webserver`).
    3.  Enter the **Image** (e.g., `nginx:latest`).
    4.  Scroll down to the **Network** section.
    5.  From the "Connected networks" dropdown, select your `macvlan-network`.
    6.  **Assign a Static IP (Recommended for Servers):**
        *   Next to the `macvlan-network` entry, click the pencil icon to edit network settings.
        *   In the "IPv4 address" field, enter a static IP address for your container that falls within the `macvlan-network` subnet (e.g., `192.168.1.101`). **Ensure this IP is unique and not in use.**
        *   Click "Update".
    7.  Configure any other desired settings (volumes, environment variables, etc.). **Do NOT map any ports from the container to the host in the "Port mappings" section, as the container will be directly accessible via its own IP on the `macvlan` network.**
    8.  Click **Deploy the container**.

    ### Option B: Deploying a Stack (Docker Compose)

    1.  In the left-hand sidebar, click on **Stacks**, then **Add stack**.
    2.  Provide a **Name** for your stack (e.g., `my-macvlan-services`).
    3.  In the **Web editor**, define your `docker-compose.yml` file. Here's an example:

        ```yaml
        version: '3.8'

        services:
          web:
            image: nginx:latest
            container_name: web-macvlan
            networks:
              macvlan-network:
                ipv4_address: 192.168.1.102 # Choose an available IP

        networks:
          macvlan-network:
            external: true # Tells Docker to use an existing network
        ```

        *   **Important:**
            *   `networks:` section under `web` service: Specifies which network the service should connect to.
            *   `ipv4_address`: Assigns a static IP address to the `web` service within the `macvlan-network`. **Choose an IP that is free in your subnet.**
            *   `networks:` section at the root level: Defines `macvlan-network` as `external: true`, indicating that it's a network that already exists and was created outside of this `docker-compose.yml` file.

    4.  Click **Deploy the stack**.

## Step 3: Verify and Test Connectivity

1.  **Check Container Details in Portainer:**
    *   Go to **Containers**, then click on your newly deployed container.
    *   Under "Network details", you should see your `macvlan-network` listed with the assigned IP address.

2.  **Ping the Container from another Device:**
    From a computer or device on the same local network as your TrueNAS, try to ping the IP address you assigned to your Docker container (e.g., `ping 192.168.1.101` or `ping 192.168.1.102`). It should be reachable.

3.  **Access the Service:**
    If your container is running a web server (like NGINX in the examples), you should be able to access it by navigating to its assigned IP address in a web browser (e.g., `http://192.168.1.101`).

## Troubleshooting

*   **Network Missing in Portainer:** If `macvlan-network` doesn't appear in Portainer's network list after creation, try refreshing the Portainer UI or restarting the Portainer container (though usually not necessary).
*   **Container Fails to Start:** Check the container logs in Portainer for errors. This could be due to an IP conflict if the assigned IP is already in use.
*   **Cannot Ping Container:**
    *   Verify the `macvlan-network` configuration (subnet, gateway, parent interface name).
    *   Ensure the IP you assigned to the container is not in use by another device.
    *   Check your router's firewall settings or any firewalls on TrueNAS that might be blocking traffic.
    *   Confirm the physical network cable for `enp3s0` (or your parent interface) is properly connected.
*   **TrueNAS Restart:** In rare cases, especially after initial `macvlan` setup, a TrueNAS system restart might help resolve unusual network issues.

By following these steps, you can successfully leverage `macvlan` networking on your TrueNAS system to provide unique and directly addressable IP addresses to your Docker containers, all managed conveniently through Portainer.
