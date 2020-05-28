# WireGuard Configuration

Linux does not define a canonical approach to network configuration. Traditionally each distribution has made its own choices with which adminstrators will become familiar. In recent years, a number of distributions have standardized systemd-networkd based network configuration. The systemd movement has been a topic of hot debate, though for new students of Linux, it offers the advantage of predictability.


!!! example "Manually configure wireguard with iproute2"
    ```
    ip link add dev wg0 type wireguard
    ip addr add dev wg0 10.99.0.4 peer 10.99.0.5
    wg set wg0 listen-port 51820 private-key /etc/wireguard/private-key \
        peer AglT3WUjY0j9Ic/SvfGGmGQ1r39MoQqlXDq9+hjOKhU= allowed-ips 0.0.0.0/0 \
        endpoint 1.2.3.4:51821
    ip link set up dev wg0
    ```

    __/etc/wireguard/privatekey__
    ```
    X/n17o9G+4o84NT4l53uV3Jf13WKiA9EVs9l3onsEno=
    ```


!!! example "Configuring wireguard persistently with systemd-networkd"
    After creating the files below, make sure that ownership and permissions are set to protect private keys.
    ```
    sudo chown root:systemd-network /etc/systemd/99-wg0.netdev
    sudo chmod 640 /etc/systemd/99-wg0.netdev
    ```

    __/etc/systemd/network/99-wg0.netdev__
    ```
    [NetDev]
    Name=wg0
    Kind=wireguard
    Description=WireGuard Tunnel Endpoint

    [WireGuard]
    ListenPort=51820
    PrivateKey=X/n17o9G+4o84NT4l53uV3Jf13WKiA9EVs9l3onsEno=

    [WireGuardPeer]
    PublicKey=AglT3WUjY0j9Ic/SvfGGmGQ1r39MoQqlXDq9+hjOKhU=
    AllowedIPs=0.0.0.0/0
    Endpoint=1.2.3.4:51821
    ```

    __/etc/systemd/network/99-wg0.network__
    ```
    [Match]
    Name=wg0

    [Address]
    Address=10.99.0.4/32
    Peer=10.99.0.5/32
    ```

!!! example "Configuring wireguard persistently with legacy Debian networking (ifupdown)"
    __/etc/network/interfaces__
    ```
    auto wg0
    iface wg0 inet static
        address 10.99.0.4
        pointopoint 10.99.0.5
        pre-up ip link add dev wg0 type wireguard
        pre-up wg setconf wg0 /etc/wireguard/wg0.conf
        post-down ip link delete dev wg0
    ```

    __/etc/wireguard/wg0.conf__
    ```
    [Interface]
    ListenPort = 51820
    PrivateKey = X/n17o9G+4o84NT4l53uV3Jf13WKiA9EVs9l3onsEno=
                 

    [Peer]
    PublicKey = AglT3WUjY0j9Ic/SvfGGmGQ1r39MoQqlXDq9+hjOKhU=
    AllowedIPs = 0.0.0.0/0
    Endpoint = 1.2.3.4:51821
    ```