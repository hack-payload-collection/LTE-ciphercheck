This tool quickly tests LTE networks for their cipher support. It's for use by telecom operators only.

## LTE Security Disabled—Misconfiguration in Commercial Networks.

Check out our research paper and talk at WiSec 2019 ([Paper](./img/wisec19-final123.pdf), [Talk](./img/WiSec19-LTE_Security_Disabled.pdf)):
> Merlin Chlosta, David Rupprecht, Thorsten Holz, and Christina Pöpper. 2019. LTE Security Disabled—Misconfiguration in Commercial Networks. In 12th ACM Conference on Security and Privacy in Wireless and Mobile Networks (WiSec ’19), May 15–17, 2019, Miami, FL, USA. ACM, New York, NY, USA, 6 pages. https://doi.org/10.1145/3317549.3324927

Contact me at [merlin.chlosta+eia0@rub.de](merlin.chlosta+eia0@rub.de)

# Encryption in LTE Networks

LTE networks protect user traffic and control data with encryption, and additionally integrity-protect control data. We're going to have a look at the method that is used for securing the data.

![Transport Security](./img/transport_security.png)

There are multiple ciphers available, thus, smartphone and network need to negotiate which one to use. Snow3G, AES and ZUC actually protect the messages. However, there's a 'NULL' algorithm for testing purposes and emergency calls -- it does not provide any protection.

![Cipher Support](./img/cipher_support.png)

Networks and smartphones must support AES and Snow3G. ZUC is optional. The NULL algorithm may only be selected for emergency purposes.

## Impact of Misconfigurations

If a network is poorly configured, man-in-the-middle attacks become trivial. If a network accepts unprotected connections, attackers can impersonate benign users.

![MitM](./img/mitm.png)

# Setup

The whole setup looks like this:

![Setup](./img/system_overview.png)

We typically use Ettus USRP B210 as Software Defined Radio, and the smartcard readers that are built into the Dell standard keyboards.

First, build the docker image:
```console
host:~$ git clone https://github.com/mrlnc/eia0.git
host:~$ cd eia0
host:eia0$ docker build -t sec_algo_test .
```

Run tests with the `start_test.sh` script, that feeds the required parameters to the docker image.

```
host:eia0$ ./start-test.sh --dl-earfcn 123 --apn internet --imei <IMEI of your smartphone>
```

## Advanced Configuration

Basically, this software is just [srsLTE](https://github.com/srsLTE/srsLTE) with minor changes. See the [srsLTE README](https://github.com/srsLTE/srsLTE/blob/master/README.md) for detailed build instructions, and [www.srslte.com](srslte.com) for documentation, guides and project news. srsLTE is released under the AGPLv3 license and uses software from the [OpenLTE project](http://sourceforge.net/projects/openlte) for some security functions and for NAS message parsing.

## Configuration

An example configuration file is located at `eia0/srsue/ue.conf.example`; copy it to `~/.config/srslte/ue.conf` for convenience.
```console
host:eia0$ mkdir ~/.config/srslte
host:eia0$ cp srsue/ue.conf.example ~/.config/srslte/ue.conf
```

Configure the cell's frequency:
```
[rf]
dl_earfcn = 3400
```

The test run will create a PCAP for each testcase -- about 600 files. So consider creating custom directory `/tmp/sec_results/` and point the PCAPs there:
```
[pcap]
enable = true
filename = /tmp/sec_results/ue
nas_enable = true
nas_filename = /tmp/sec_results/nas
```
That will result in files like `/tmp/sec_results/ue_ID_0451_EIA_00001100_EEA_00000010_try_0.pcap`. Last, configure the USIM:
```
[usim]
mode = pcsc
#algo = xor
#opc  = 63BFA50EE6523365FF14C1F45F88737D
#k    = 00112233445566778899aabbccddeeff
#imsi = 001010123456789
imei = 353490069873319
```
Fill in an IMEI of one of your devices. The other parameters are read from the SIM card.

# Testing Procedure

When the UE starts the connection procedure, it will transmit a list of supported ciphers. The network then selects one of these, based on it's own capabilities. If there's no match, or some policy prohibits some cipher (e.g., NULL), the network must reject the connection attempt.

For example, if the UE signals only NULL ciphers for encryption and integrity protection, the network should not establish a connection as in this example:
![Setup](./img/test_procedure.png)

We perform one connection setup for each possible combination of ciphers and check whether the network accepts or denies. Since there are 256 combinations, a single test run performs at least that many attaches to the network.

# Credits

srsLTE is a free and open-source LTE software suite developed by SRS (www.softwareradiosystems.com).

[Katharina Kohls](https://kkohls.org) allowed me to use the pictograms, taken from her research papers or presentations. Thanks!
