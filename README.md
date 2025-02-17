REDSOCKS2
=========
This is a modified version of original redsocks.
The name is changed to REDSOCKS2 to distinguish with original redsocks.
REDSOCKS2 contains several new features besides many bug fixes to original
redsocks.

1. Redirect TCP connections which are blocked via proxy automatically without
need of blacklist.
2. Redirect UDP based DNS requests via TCP connection.
3. Integrated [shadowsocks](http://shadowsocks.org/) proxy support(IPv4 Only).
4. Redirect TCP connections without proxy.
5. Redirect TCP connections via specified network interface.
6. UDP transparent proxy via shadowsocks proxy.
7. Support Ful-cone NAT Traversal when working with shadowsocks or socks5 proxy.
8. Integrated HTTPS proxy support(HTTP CONNECT over SSL).
9. Support TCP Fast Open on local server side and shadowsocks client side
10.Support port reuse ([SO_REUSEPORT](https://lwn.net/Articles/542629/))

[Chinese Reference](https://github.com/semigodking/redsocks/wiki)

macOS Big Sur install
---

```bash
brew install openssl libevent

git clone https://github.com/wangxian/redsocks.git
cd redsocks

make DISABLE_SHADOWSOCKS=true


```


HOW TO BUILD
------------
### Prerequisites
The following libraries are required.

* libevent2
* OpenSSL or PolarSSL

### Steps
On general Linux, simply run command below to build with OpenSSL.

```
$ make
```

To compile with PolarSSL

```
$ make USE_CRYPTO_POLARSSL=true
```

To compile static binaries (with Tomatoware)

```
$ make ENABLE_STATIC=true
```

By default, HTTPS proxy support is disabled. To enable this feature, you need to
compile like (Require libevent2 compiled with OpenSSL support):
```
$ make ENABLE_HTTPS_PROXY=true
```

To compile on newer systems with OpenSSL 1.1.0 and newer (disables shadowsocks support):
```
$ git apply patches/disable-ss.patch
$ make
```

To compile on newer systems with OpenSSL 1.1.1+ (just disable shadowsocks support, no patch need and worked with ENABLE_HTTPS_PROXY. DO NOT APPLY THE PATCH!):
```
$ make DISABLE_SHADOWSOCKS=true
```

Since this variant of redsocks is customized for running with Openwrt, please
read documents here (http://wiki.openwrt.org/doc/devel/crosscompile) for how
to cross compile.

### MacOS
To build on a MacOS system, you will have to install OpenSSL headers and libevent2
For this, brew is your best friends
```
$ brew install openssl libevent
```
Makefile include the folder of openssl headers and lib installed by brew.

To build with PF and run on MacOS, you will need some pf headers that are not included with a standard MacOS installation.
You can find them on this repository : https://github.com/apple/darwin-xnu
And the Makefile will going find this file for you

Configurations
--------------
Please see 'redsocks.conf.example' for whole picture of configuration file.
Below are additional sample configuration sections for different usage.
Operations required to iptables are not listed here.

### Redirect Blocked Traffic via Proxy Automatically
To use the autoproxy feature, please change the redsocks section in
configuration file like this:

	redsocks {
	 bind = "192.168.1.1:1081";
	 relay = "192.168.1.1:9050";
	 type = socks5; // I use socks5 proxy for GFW'ed IP
	 autoproxy = 1; // I want autoproxy feature enabled on this section.
	 // timeout is meaningful when 'autoproxy' is non-zero.
	 // It specified timeout value when trying to connect to destination
	 // directly. Default is 10 seconds. When it is set to 0, default
	 // timeout value will be used.
	 // NOTE: decreasing the timeout value may lead increase of chance for
	 // normal IP to be misjudged.
	 timeout = 13;
	 //type = http-connect;
	 //login = username;
	 //password = passwd;
	}

### Redirect Blocked Traffic via VPN Automatically
Suppose you have VPN connection setup with interface tun0. You want all
all blocked traffic pass through via VPN connection while normal traffic
pass through via default internet connection.

	redsocks {
		bind = "192.168.1.1:1081";
		interface = tun0; // Outgoing interface for blocked traffic
		type = direct;
		timeout = 13;
		autoproxy = 1;
	}

### Redirect Blocked Traffic via shadowsocks proxy
Similar like other redsocks section. The encryption method is specified
by field 'login'.

	redsocks {
		bind = "192.168.1.1:1080";
		type = shadowsocks;
		relay = "192.168.1.1:8388";
		timeout = 13;
		autoproxy = 1;
		login = "aes-128-cfb"; // field 'login' is reused as encryption
							   // method of shadowsocks
		password = "your password"; // Your shadowsocks password
	}

	redudp {
		bind = "127.0.0.1:1053";
		relay = "123.123.123.123:1082";
		type = shadowsocks;
		login = rc4-md5;
		password = "ss server password";
		dest = "8.8.8.8:53";
		udp_timeout = 3;
	}


List of supported encryption methods(Compiled with OpenSSL):

	table
	rc4
	rc4-md5
	aes-128-cfb
	aes-192-cfb
	aes-256-cfb
	bf-cfb
	camellia-128-cfb
	camellia-192-cfb
	camellia-256-cfb
	cast5-cfb
	des-cfb
	idea-cfb
	rc2-cfb
	seed-cfb

List of supported encryption methods(Compiled with PolarSSL):

	table
	ARC4-128
	AES-128-CFB128
	AES-192-CFB128
	AES-256-CFB128
	BLOWFISH-CFB64
	CAMELLIA-128-CFB128
	CAMELLIA-192-CFB128
	CAMELLIA-256-CFB128

### Work with GoAgent
To make redsocks2 works with GoAgent proxy, you need to set proxy type as
'http-relay' for HTTP protocol and 'http-connect' for HTTPS protocol  
respectively.
Suppose your goagent local proxy is running at the same server as redsocks2,
The configuration for forwarding connections to GoAgent is like below:

	redsocks {
	 bind = "192.168.1.1:1081"; //HTTP should be redirect to this port.
	 relay = "192.168.1.1:8080";
	 type = http-relay; // Must be 'htt-relay' for HTTP traffic.
	 autoproxy = 1; // I want autoproxy feature enabled on this section.
	 // timeout is meaningful when 'autoproxy' is non-zero.
	 // It specified timeout value when trying to connect to destination
	 // directly. Default is 10 seconds. When it is set to 0, default
	 // timeout value will be used.
	 timeout = 13;
	}
	redsocks {
	 bind = "192.168.1.1:1082"; //HTTPS should be redirect to this port.
	 relay = "192.168.1.1:8080";
	 type = http-connect; // Must be 'htt-connect' for HTTPS traffic.
	 autoproxy = 1; // I want autoproxy feature enabled on this section.
	 // timeout is meaningful when 'autoproxy' is non-zero.
	 // It specified timeout value when trying to connect to destination
	 // directly. Default is 10 seconds. When it is set to 0, default
	 // timeout value will be used.
	 timeout = 13;
	}

### Redirect UDP based DNS Request via TCP connection
Sending DNS request via TCP connection is one way to prevent from DNS
poisoning. You can redirect all UDP based DNS requests via TCP connection
with the following config section.

    tcpdns {
    	// Transform UDP DNS requests into TCP DNS requests.
    	// You can also redirect connections to external TCP DNS server to
    	// REDSOCKS transparent proxy via iptables.
	bind = "192.168.1.1:1053"; // Local server to act as DNS server
	tcpdns1 = "8.8.4.4:53";    // DNS server that supports TCP DNS requests
    	tcpdns2 = 8.8.8.8;      // DNS server that supports TCP DNS requests
    	timeout = 4;            // Timeout value for TCP DNS requests
    }

Then, you can either redirect all your DNS requests to the local IP:port
configured above by iptables, or just change your system default DNS upstream
server as the local IP:port configured above.

AUTHOR
------
[Zhuofei Wang](mailto:semigodking.com) semigodking@gmail.com
