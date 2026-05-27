# DTC
If you are working on DTC you will likely need to build and install the cdcl_umd_msgs repository, instructions can be found in the [README for the repository](https://github.com/UMD-CDCL/cdcl_umd_msgs)

The file structure on most DTC base station devices is:
```
/home/<USER>/
 └── DTC/
     ├── cdcl_ws/
     │   └── src/
     │       └── cdcl_umd_msgs/
     └── inference_engine/
```

## Network Setup for Control of Spot through the onboard computer (HP Z2)

### Settings on Spot

#### General

On network setup page > General, set Default interface to Payload

![General network settings page](https://i.imgur.com/bgRc5vI.png)

#### Ethernet

On network setup page > Ethernet, no special changes since WS3

![Ethernet network settings page](https://i.imgur.com/w3Cvin3.png)

#### WiFi

On network setup page > WiFi, disable WiFi Radio

![WiFi network settings page](https://i.imgur.com/wAZkMZe.png)

#### Payload

On network setup page > Payload,

1. IPv4 Address is `192.168.50.3`
2. Stored Default Route is `192.168.50.5`

![Payload network settings page](https://i.imgur.com/na9Q1h7.png)

---

### Settings on the HP

#### 1. Enable packet forwarding

Tell the Linux kernel to allow packet transit.

> Note: To make this survive reboots permanently, ensure `net.ipv4.ip_forward=1` is uncommented in `/etc/sysctl.conf`.

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

#### 2. Check interface names

Check names for interfaces using `ip a`.

In this example:

- Ethernet interface name on HP to Spot: `enxa0cec8b8349f`
- WiFi interface name on HP to RoboScout: `wls3f3`

#### 3. Forward tablet traffic to Spot

Forward ONLY the tablet's incoming traffic to Spot's payload IP.

> Note:
>
> - Tablet IP is `10.200.142.32` for Cairo. Change for other Spots accordingly.
> - Destination IP remains `192.168.50.3` for all Spots.

```bash
sudo iptables -t nat -A PREROUTING -i wls3f3 -s 10.200.142.32 -j DNAT --to-destination 192.168.50.3
```

#### 4. Masquerade outbound Ethernet traffic

Masquerade the traffic leaving single_gstreamerthe Ethernet port so Spot knows how to reply

```bash
sudo iptables -t nat -A POSTROUTING -o enxa0cec8b8349f -j MASQUERADE
```

#### 5. Bypass Docker firewall rules

Explicitly permit the tablet's traffic to traverse Docker's default DROP policy on the FORWARD chain.

##### Allow tablet traffic into the Ethernet port

> Note: Tablet IP is `10.200.142.32` for Cairo. Change for other Spots accordingly.

```bash
sudo iptables -A FORWARD -i wls3f3 -o enxa0cec8b8349f -s 10.200.142.32 -j ACCEPT
```

##### Allow Spot return traffic back to the tablet

> Note: Tablet IP is `10.200.142.32` for Cairo. Change for other Spots accordingly.

```bash
sudo iptables -A FORWARD -i enxa0cec8b8349f -o wls3f3 -d 10.200.142.32 -j ACCEPT
```

#### 6. Save configuration

Save configuration such that it persists upon reboot

```bash
sudo netfilter-persistent save
```

# FloatSci

