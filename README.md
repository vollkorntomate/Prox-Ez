# Prox-Ez: The Swiss Army Knife of HTTP auth

This HTTP proxy handles all HTTP authentications on your behalf.

It supports NTLM EPA (channel binding and service binding), kerberos, pass-the-hash, overpass-the-hash (pass-the-key) and pass-the-ticket (TGT and TGS).

Related articles:
- [Dissecting NTLM EPA with love & building a MitM proxy](https://www.synacktiv.com/publications/dissecting-ntlm-epa-with-love-building-a-mitm-proxy.html)
- [A study on Windows HTTP authentication (Part II)](https://www.synacktiv.com/publications/a-study-on-windows-http-authentication-part-ii.html)

> [!NOTE]
> This fork makes some changes to the way how credentials are passed to the proxy.
> It is tested for the combination of username, password and domain, but not for NT hashes. Although that should still work, I give no guarantee. Pull requests are very welcome!

## Installation

1. Install the dependancies
```sh
# In a venv
python3 -m venv venv
source venv/bin/activate
python3 -m pip install -r requirements.txt
```
2. Enjoy.

## Usage

### Quickstart

For the user `SOMEDOMAIN\someuser`, you can either pass the credentials via command-line (will prompt for the password):
```sh
python3 proxy.py --domain SOMEDOMAIN --username someuser --password
```

Or you can create a JSON file with the credentials (e.g. `credentials.json`):
```json
{
  "my.domain.com": {
    "domain": "SOMEDOMAIN",
    "username": "someuser",
    "password": "Sup3rSecret!"
  }
}
```
```sh
python3 proxy.py --creds credentials.json
```

Use `"_"` as a wildcard for all domains.

It will try to authenticate with the credentials on any website that requires authentication.

### With Burp Suite

In order to work with Burp Suite:
* Disable HTTP/2 support: `Proxy Settings` > Uncheck `HTTP/2` as well as `Network` > `HTTP` > Uncheck `HTTP/2`
* `Proxy` > `Options` > `Miscellaneous` > Uncheck `Set response header "Connection: close"`
* `Proxy` > `Options` > `Miscellaneous` > Uncheck `Set "Connection" header on incoming requests when using HTTP/1`

Afterwards, you just have to specify an upstream proxy in burp, so that it uses this proxy for the host you cannot authenticate with:
- In `Network` > `Connections` > `Upstream Proxy Servers` > click `Add` > specify the remote hostname that is causing problems with NTLM authentication, the proxy host and port configured in the tool and leave the `Authentication type` to `None`.
- You may also need to disable the socks proxy if enabled.

### Help

```
$ python3 proxy.py -h
usage: proxy.py [-h] [--listen-address LISTEN_ADDRESS] [--listen-port LISTEN_PORT] [--cacert CACERT] [--cakey CAKEY] [--cakey-pass CAKEY_PASS] [--certsdir CERTSDIR] [--singleprocess] [--debug] [--dump-keys DUMP_KEYS] [--username USERNAME] [--password [PASSWORD]] [--domain DOMAIN] [--creds CREDS] [--hashes HASHES] [--kerberos] [--dcip DCIP] [--spn SPN] [--spn-force-fqdn] [--no-epa]

Prox-Ez: The Swiss Army Knife of HTTP auth.

options:
  -h, --help            show this help message and exit
  --listen-address, -l LISTEN_ADDRESS
                        Address the proxy will be listening on, defaults to 127.0.0.1.
  --listen-port, -p LISTEN_PORT
                        Port the proxy will be listening on, defaults to 3128.
  --cacert CACERT       Filepath to the CA certificate, defaults to ./cacert.pem. Will be created if it does not exists.
  --cakey CAKEY         Filepath to the CA private key, defaults to ./cakey.pem. Will be created if it does not exists.
  --cakey-pass CAKEY_PASS
                        CA private key passphrase.
  --certsdir CERTSDIR   Path to the directory the generated certificates will be stored in, defaults to /tmp/Prox-Ez. Will be created if it does not exists.
  --singleprocess, -sp  Do you want to be slowwwww ?! Actually useful during debug.
  --debug, -d           Increase debug output.
  --dump-keys, -dk DUMP_KEYS
                        File to dump the SSL/TLS keys to. Useful when trying to debug. When this option is specified, --singleprocess is implied.
  --username, -u USERNAME
                        Username that will be used to authenticate.
  --password [PASSWORD] Will prompt for the password.
  --domain DOMAIN       Domain to which the username is joined (e.g. 'WORKGROUP').
  --creds CREDS         Path to the credentials file, for instance: { "my.hostname.com": { "domain": "WORKGROUP", "username": "user", "password": "Sup3rSecret!", "spn": "HTTP/anothername" }, "my.second.hostname.com": { "domain": "DOMAIN1", "username": "user1", "hashes": ":nthash1" } }
  --hashes HASHES       Could be used instead of password. It is associated with the domain and username given via --default_creds. format: lmhash:nthash or :nthash.
  --kerberos, -k        Enable kerberos authentication instead of NTLM.
  --dcip DCIP           IP Address of the domain controller (only for kerberos).
  --spn SPN             Use the provided SPN when an SPN is needed. More details in the article.
  --spn-force-fqdn      Force the usage of the FQDN as the SPN instead of what was specified in the URL.
  --no-epa              Deactivate the NTLM EPA feature.
```

### Known issues

- No support for websocket. It will yield assertion errors such as:
```
DEBUG:Proxy.ProxyToServerHelper:Our state: MIGHT_SWITCH_PROTOCOL; their state: SEND_RESPONSE
[...]
    assert self.conn.our_state in [h11.DONE, h11.MUST_CLOSE, h11.CLOSED] and self.conn.their_state is h11.SEND_RESPONSE
AssertionError
```

