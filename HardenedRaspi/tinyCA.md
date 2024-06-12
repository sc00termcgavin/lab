## Tiny Certificate Authority

> Refrence Guide: 
> https://smallstep.com/blog/build-a-tiny-ca-with-raspberry-pi-yubikey/

### Step 1: Install `YKMAN`

```shell 
$ sudo apt install -y yubikey-manager
```

### Step 2: Install `GO` for `step-ca`[server](https://smallstep.com/docs/step-ca/index.html#introduction-to-step-ca)
- GO Version: [1.22.3](https://go.dev/doc/install)

```shell
$ curl -LO https://go.dev/dl/go1.22.3.linux-arm64.tar.gz
$ sudo tar -C /usr/local -xzf go1.22.3.linux-arm64.tar.gz
$ echo "export PATH=\$PATH:/usr/local/go/bin" >> .profile
$ source .profile
```

### Step 3: Build & Install `Step-ca` & `Step`
- step-ca Source: Version [26.1](https://github.com/smallstep/certificates/releases/tag/v0.26.1)
- [Build with Yubikey support](https://smallstep.com/docs/step-ca/configuration/index.html#yubikey)

```shell
$ curl -LO https://github.com/smallstep/certificates/releases/download/v0.26.1/step-ca_0.26.1.tar.gz
$ mkdir step-ca
$ tar -xvzf step-ca_0.26.1.tar.gz -C step-ca
$ cd step-ca
```

- Build `step-ca`

```shell
$ sudo apt-get install -y libpcsclite-dev gcc make pkg-config
$ make bootstrap
$ make build GOFLAGS=""

$ sudo cp bin/step-ca /usr/local/bin
$ sudo setcap CAP_NET_BIND_SERVICE=+eip /usr/local/bin/step-ca
$ step-ca version
```

- [Install `step CLI`](https://github.com/smallstep/cli/releases/tag/v0.26.1) from a prebuilt binary

```
$ cd
$ curl -LO https://github.com/smallstep/cli/releases/download/v0.26.1/step_linux_0.26.1_arm64.tar.gz
$ tar xvzf step_linux_0.26.1_arm64.tar.gz
$ sudo cp step_0.26.1/bin/step /usr/local/bin
```

---

### Step 4: PKI Creation
> Create root & intermediate CA certificates & keys. Store them on the Yubikey
> Insert USB and generate keys directly on the drive to airgap the microSD card
> sudo diskutil eraseDisk [format] [DiskName] /dev/[DiskNodeID]
> Replace: `disk>N<` with DiskID 





```console
$ sudo fdisk -l

$ sudo fdisk /dev/sda

$ sudo mkfs.ext4 /dev/sda1 -v

Writing superblocks and filesystem accounting information: done 
```

### Step 5: Generate PKI on drive

```shell
$ sudo mount /dev/sda1 /mnt
$ cd /mnt
$ sudo mkdir ca
$ sudo chown pi-admin:pi-admin ca
$ export STEPPATH=/mnt/ca
$ step ca init --pki --name="Tiny" --deployment-type standalone

✔ What do you want your password to be? [leave empty and we'll generate one]: ...
Generating root certificate...
```


### Step 6: Import the CA into the Yubikey

```shell 
$ sudo systemctl enable pcscd
$ sudo systemctl start pcscd
$ ykman piv certificates import 9a /mnt/ca/certs/root_ca.crt
$ ykman piv keys import 9a /mnt/ca/secrets/root_ca_key
$ ykman piv certificates import 9c /mnt/ca/certs/intermediate_ca.crt
$ ykman piv keys import 9c /mnt/ca/secrets/intermediate_ca_key
$ ykman piv info
```

### Step 7: Copy CA certificate files
-  leave the private keys on the USB stick, and continue creating your CA.

```shell 
$ sudo cp /mnt/ca/certs/intermediate_ca.crt /mnt/ca/certs/root_ca.crt /root
$ cd
$ sudo umount /mnt
```

### Step 8: Configuring The CA

```shell
 sudo useradd step
$ sudo passwd -l step
$ sudo mkdir /etc/step-ca
$ export STEPPATH=/etc/step-ca
$ sudo --preserve-env step ca init --name="Tiny CA" \
    --dns="tinyca.internal,10.20.30.42" --address=":443" \
    --provisioner="you@example.com" \
    --deployment-type standalone \
    --remote-management
```

```shell
$ sudo mv /root/root_ca.crt /root/intermediate_ca.crt /etc/step-ca/certs
$ sudo rm -rf /etc/step-ca/secrets
```



### Step 9: Configure `step-ca`

```shell
sudo su -
nano /etc/step-ca/config/ca.json
```

> {
        "root": "/etc/step-ca/certs/root_ca.crt",
        "federatedRoots": [],
        "crt": "/etc/step-ca/certs/intermediate_ca.crt",
        "key": "yubikey:slot-id=9c",
        "kms": {
            "type": "yubikey",
            "pin": "123456"
        },
        "address": ":443",
..;.
}

```shell 
$ sudo chown -R step:step /etc/step-ca
$ sudo -u step step-ca /etc/step-ca/config/ca.json
```

```shell 
step ca bootstrap --ca-url "https://tinyca.internal" --fingerprint de3d02c6facbdebec297a05a145bc023fbb7d49abd9d0e7525fcd5dc04f8f0b5

```
step ca bootstrap --ca-url "https://tinyca.internal" --fingerprint