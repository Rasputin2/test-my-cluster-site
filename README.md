# Table of Contents

- [Intro](#intro)
- [File_Structure](#file_structure)
- [Definitions](#definitions)
- [Local_Development](#local_development)
- [Deployment](#deployment)
- [Networking](#networking)


# Intro

The purpose of this project is to use raspberry pis to demonstrate the basic principles of networking and cloud computing.  To that end, the actual application is very basic.  It's sole function is to illustrate key concepts and I keep it simple so the details of application programming do not interfere with the cloud and networking elements.  References to layers are to the layers used on the Internet and not the 7 layer theoretical model from the Open Systems Interconnection (OSI) model. The layers are:

Layer 5: Application Layer (Data)
Layer 4: Transport Layer ("Segment" over TCP / "Datagram" over UDP)
Layer 3: Networking Layer ("packets")
Layer 2: Link Layer ("frames" over ethernet or wireless "links" identified with MAC addresses resolved by ARP)
Layer 1: Physical layer ("bits" over copper wire/wireless) 

So if I refer to Link layer you know what I mean. 

# File_Structure

The file structure is set up as a single workspace with two build units each consisting of one package. 

------------------------
microk8s
│   argo-cd.yaml
│   backend.yaml
│   frontend.yaml
│   ingress.yaml
│   namespace.yaml
src
├───backend
│   │   Dockerfile
│   │   pyproject.toml
│   │   
│   └───app
│          main.py
│          __init__.py
│               
└───frontend
    │   Dockerfile
    │   pyproject.toml
    │   
    └───app
            main.py
            __init__.py
tests
docker-compos.yaml
pyproject.toml
README.md
-------------------------

# Definitions

ARP
: Address Resolution Protocol

CD
: Continuous Deployment
: The process by which the current version of your code is instantly deployed after being changed

CI
: Continuous Integration
: The process by which you collaboratively make changes to code and push them to a version control system like Github

DHCP
: Dynamic Host Control Protocol
: Randomly assigns new IP addresses to devices on a local network

LAN
: Local Area Network

MAC
: Media Access Control : creates unique address for hardware

WAN
: Wide Area Network

# Local_Development

## Local_Development_No_Docker

### Step Zero: 

You should have an integrated development environment (IDE) within which you can code like PyCharm or VS Code or whatever suits your needs.  Once you have that, you need to get this code on your laptop.  Given that this tutorial involves a CI/CD process that you will need to perform in your own github account and your own raspberry pis, you should:

(i)  Fork (not clone) this repo
(ii) Clone the Forked version of this repo onto your laptop

### Step One: Ensure you have installed uv

#### Windows

```powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"```

#### Linux

```curl -LsSf https://astral.sh/uv/install.sh | sh```

### Step Two: Create venv

```uv venv```

### Step Three: Activate venv

#### Windows (if using powershell)

```.\.venv\Scripts\Activate.ps1```

#### Linux

```source .venv/bin/activate```

### Step Four: Install Packages

```uv sync```

### Step Five: Run Backend and Frontend -- No Docker

There are two (2) different ways of running this simple application locally.  The first is to run the FastAPI backend and the streamlit frontend directly on your computer without any intermediation by Docker containers.  That is what this step illustrates.  The other approach is shown below in a different step five.

#### Backend

In a new terminal within your IDE (but inside your test-my-cluster-site project root) run:

```uv run --package backend uvicorn src.backend.app.main:app --reload```

Once this runs, you can go to the localost website port 127.0.0.1:8000 and you should see "Hello World" displayed or, alternatively, you can go to http://127.0.0.1:8000/docs and see the "swagger ui" that comes automatically with the FastAPI library you downloaded when you ran the uv installation above. 

#### Frontend

You must leave that uvicorn FastAPI() terminal running and open a second terminal to run the frontend.  To run the frontend, use this command from the project root:

```uv run --package frontend streamlit run src/frontend/app/main.py```

You can see the app in your browser by going to localhost:8501.

***comment:*** If you run backend and frontend this way, the way frontend "knows" where to contact backend is via this line of code in src.app.frontend.main:

```BACKEND_URL = os.getenv("BACKEND_URL", "http://localhost:8000")``

Given that there is no environment variable set with the value of BACKEND_URL, the url defaults to localhost:8000 which is what we want because the FastAPI is listening on that exact port. Importantly, our app is not available on our Local Area Network much less the Internet.  The only place you can access it is via the computer that happens to be running the backend package. 

## Local_Development_Docker

When we move to our 'cloud', each "build unit" (your backend and frontend) will run within its own "container" within a "pod" which is spawned by and associated with a "Node".  We'll get to that in more detail in the lessons.  Long story short, though, to run the build units on a container on your laptop, we need Docker.  So, you must install Docker on your laptop, the Community Edition is fine. 

### Step Five: Run Backend and Frontend -- Docker

You must start the docker daemon (you can typically just click on the Docker Desktop icon and wait for it to load) before running these commands:

```docker compose up``` 

OR

```docker compose up --build --force-recreate``` if you change the image

# Deployment

Thus far, we have a very simple web application.  It is not running in a cloud environment.  It is not accessible on our home local area network (LAN).  It definitely is not accessible via the Internet.   

## Preparing_the_Pis

We are going to use two (2) Raspberry Pis as two different servers to create a small "cloud" infrastructure.  I used two Rastech Raspberry Pis with 64 bit Cortex A76 processors with the Ubuntu server distro of Linux installed with the Ubuntu distro.  But any Pi after Raspberry Pi 3 should work.  I also suggest that if you are connecting to your home network via ethernet you get an ethernet switch so that you can plug both Raspberry Pis + your laptop into the switch at the same time and plug the switch into your ethernet jack.  I am assuming you have read the instructions to set up the Pis and you have two running Pis connected to your local area network.  You need to mentally choose one to be your "Master" and the other as "Worker".  We'll formalize this distinction in a moment. You need to have two Pis with the Ubuntu server (not client) version of Linux installed before proceeding. 

### Assign Static IP Addresses to Each Pi

Absent some instruction to the contrary, your router will randomly assign new ip addresses to your Pis every time they restart using the Dynamic Host Control Protocol (DHCP).  To avoid that frustration, we need to assign static IP addresses to each Pi.  The trick here though is that we want to choose addresses that fall INSIDE of your home LAN's range but OUTSIDE of the range of addresses used by DHCP otherwise you will may have conflicts.  Alternatively, you can get on your ISP's website and log in and "reserve" IP addresses on your router (kind of like an Amazon firestick does when you plug it in).

Most home networks use something like:

Router / default gateway: 192.168.1.1
Subnet mask: 255.255.255.0

That usually means your usable LAN range is:

192.168.1.1 → router
192.168.1.2 - 192.168.1.254 → usable device addresses
192.168.1.255 → broadcast address (not usable)

The DHCP is harder to determine.  You can get some information by checking on the Pi or your laptop (windows) ```ipconfig``` or (linux) ```ifconfig```.  When in doubt, check your ISP's website. For purposes of this project, I am going to assume we chose the following IP addresses for each device and link:

|Pi Name|Link Layer|Static Address|
|-------|----------|--------------|
|Master|eth0|192.168.1.10|
|Master|wlan0|192.168.1.11|
|Servant|eth0|192.168.1.12|
|Servant|wlan0|192.168.1.13|

To achieve this result we need to take a number of steps on each Pi.  Assuming you have Ubuntu server installed, there are a couple of key components that we will be working with:

- systemd (the system that manages the operating system)
- systemd-networkd (used for networking)
- netplan (this reads yamls and configures systemd from those yamls)

We need to enable systemd-networkd on Master:

```sudo systemctl enable systemd-networkd```

We need to start systemd-networkd on Master:

```sudo systemctl start systemd-networkd```

To perform the next steps, you will need to navigate to the root (/) of Ubuntu on Master and find the cloud-init.yaml in your /etc/netplan/ folder.  You need to amend it (using a text editor, ```sudo vi <insert file name>``` or ```sudo nano <insert file name>```) to look like this:

network:
  version: 2
  renderer: networkd

  ethernets:
    eth0:
      dhcp4: false
      addresses:
        - 192.168.1.10/24
      routes:
        - to: 0.0.0.0/0
          via: 192.168.1.1
          metric: 200
      nameservers:
        addresses:
          - 1.1.1.1
          - 8.8.8.8
  wifis:
    wlan0:
      dhcp4: false
      addresses:
        - 192.168.1.11/24
      access-points:
        <YOUR_WIFI_SSID>:
          password: <YOUR_WIFI_PASSWORD>
      routes:
        - to: 0.0.0.0/0
          via: 192.168.1.1
          metric: 100
      nameservers:
        addresses:
          - 1.1.1.1
          - 8.8.8.8

This assumes your default gateway is 192.168.1.1 (you need to check).  It also assumes you chose the same static IP address that I did.  It further assumes you have the Pi connected via ethernet AND wifi.  If you don't use wifi you can delete wifis: and everything below it.  If you don't have ethernet, you can delete the section associated with ethernets. 

```sudo netplan generate``` # Should surface any major issues, if errors arise debug
```sudo netplan apply``` # Should apply the yaml and configure systemd-networkd

Now check the status of each of the Link layers attached to your Master.  You should hopefully see the specific static IP addresses applied to each link within each device. 

```networkctl status eth0```
```networkctl status wlan0```




## Continuous Integration


```git push```

on your "master" compute from within the microk8s folder run:

```microk8s apply -f .```

# Networking

The following explains the the networking aspects of the project.  I start with your home router's external IP address and then drill down to the k8s cluster and the backend and frontend services.  Before proceeding, recall that what we refer to as the "Internet" consists of a five layers:

| **Layer** | **Name**       | **Data Type** |
|-----------|----------------|---------------|
| Layer 5 | Application Layer (hyper-text transfer protocol or http)| Data | 
| Layer 4 | Transport Layer (TCP/UDP) | Segments (for TCP) or Datagrams (for UDP) |
| Layer 3 | Network Layer | Packets or Datagrams |
| Layer 2 | Link Layer | Frames |
| Layer 1 | Physical Layer (Copper Wire, Ethernet, Wireless) | Bits |

## External IP

Bits come into your home via your router and its external IP address.  The connection between your router and your ISP is facilitated by and part of the Wide Area Network (WAN). You can find your WAN information using this command:

### Windows && Linux

```nslookup myip.opendns.com resolver1.opendns.com```

Assume for our purposes that the external IP address is 111.111.111.111.  This address is inside the WAN.  You can You can find your real external address like this:

### Windows

```ipconfig```

### Linux

If you don't have curl, you may have to install it via ```sudo apt install curl```.

```curl -4 ifconfig.me```

## Local Area Network (LAN)

Your LAN creates the network for your home.  The link between the home network and the WAN is the LAN's default gateway.  You can find in windows using ```ipconfig``` and in linux using ```ip route```.  Assume the default gateway is 192.168.1.1.  You can test it by running ```tracert -4 www.google.com```.  The first "hop" should be to the default gateway. The LAN is also part of the Layer 3 Networking Layer.  

### Network Address Translation (NAT)

The network address translation (NAT) protocol changes the destination 
address the of incoming packet from x.x.x.x:80 to, for example, 192.168.1.1:80, port 80 on the default gateway on the LAN.  The NAT works much like a loch between two bodies of water.  The NAT converts an external address on the Wide Area Network (WAN) to an internal address on your Local Area Network (LAN) and back again.   

### Routing Table 

Now that the NAT has translated the destination to be 192.168.1.10, the Routing Table finds the Media Access Control ("MAC") Address of the network card associated with the physical hardware you are trying to reach.  The MAC is a globally unique address (like a GUID) that is associated with the network card on your Pi and the address is independent of what network it is a part of.  The MAC address is discovered by the Routing Table via the Address Resolution Protocol (ARP).  The ARP knows the MAC address because every device connected to the LAN sends out a message saying it is available on the LAN via the ARP, and the router responds asking for the device's MAC address.  

### Getting from the LAN to a Pod on the Cluster

![alt text](LAN_DIAGRAM.png)


