# noip-rfc2136
Simple HTTP(S) server that pretends to be a No-IP compatible DynDNS service,  but actually performs RFC2136 updates to a DNS server.

## Background
I originally wrote this so that I could update a BIND9 dynamic zone directly from a Unifi UDM-Pro. 
I was a little disappointed to discovered that the only Dynamic DNS options were cloud providers, 
so I opted to write something that pretended to be a No-IP compatible cloud DNS API, but instead does proper 
RFC2136 `nsupdate` style modifications to a zone. This enables me to continue using my self-hosted BIND zones and not 
have to move to another cloud service.

Maybe you will find this useful too. The code as it stands is currently running and working well for me, so I though I would 
publish it for anyone else who has a similar need.

## Requirements
A working BIND configuration that allows dynamic updates with TSIG keys
See also: `requirements.txt`

## Features
 * Implements enough of [The No-IP API](https://www.noip.com/integrate/request) to be functional.
 * Supports secure BIND9 updates using TSIG keys
 * Option to use HTTP basic auth
 * Option to use HTTPS certificates
 * Option to set configuration using environment variables

## Limitations
 * No IPv6 support. Not supported by UDM-Pro at time of writing, so I haven't put it in.
 * No support for multiple domains. See above, but also I have no need for it.
 * HTTP basic auth as currently implemented only allows one user/password combination. Unless multiple domains are implmented, I don't see the need for any more than this.
 * Does not load BIND `.key` files, the key name and secret must be configured. I do not have a way of robustly parsing these yet.

## Usage
Edit the sample YAML file to suit and save as `config.yaml`, or define them as environment variables:

### DNS

| YAML key | Description | Optional | Default | Environment variable |
| :--- | :--- | :--- | :--- | :--- |
| `nameserver` | The IP of the DNS server you want to update | No | N/A | `NOIP_RFC2136_DNS_NAMESERVER` |
| `zone` | The zone you want to update | No | N/A | ` NOIP_RFC2136_DNS_ZONE` |
| `ttl` | TTL of the record | Yes | 30 | ` NOIP_RFC2136_DNS_TTL` |
| `create_enabled` | Allow creation of records in DNS if they do not exist | Yes | False | ` NOIP_RFC2136_DNS_CREATE_ENABLED` |
| `tsig_key_name` | String containing the TSIG key name | No | N/A | ` NOIP_RFC2136_DNS_TSIG_KEY_NAME` |
| `tsig_key_secret` | Base64 encoded string containing the TSIG key secret | No | N/A | ` NOIP_RFC2136_DNS_TSIG_KEY_SECRET` |
| `tsig_key_algorithm` | The algorithm used to generate the key, eg `hmac-sha256` | No | N/A | ` NOIP_RFC2136_DNS_TSIG_KEY_ALGORITHM` |

### HTTP server config

| YAML key | Description | Optional | Default | Environment variable |
| :--- | :--- | :--- | :--- | :--- |
|  `host` | The IP address the HTTP should bind to. Set to `None` to listen on ALL addresses (not recommended) | Yes | `localhost` | `NOIP_RFC2136_LISTEN_HOST` |
|  `port` | The port the HTTP server should listen on | Yes | `8000` | `NOIP_RFC2136_LISTEN_PORT` |

### HTTPS config

| YAML key | Description | Optional | Default | Environment variable |
| :--- | :--- | :--- | :--- | :--- |
|  `enabled` | Set to `True` to enable HTTPS, `False` for plain HTTP | Yes | `False` | `NOIP_RFC2136_HTTPS_ENABLED` |
|  `key_file` | Path to HTTPS private key file | Yes | `None` | `NOIP_RFC2136_HTTPS_KEY_FILE` |
|  `cert_file` | Path HTTPS certificate file | Yes | `None` | `NOIP_RFC2136_HTTPS_CERT_FILE` |

### HTTP basic auth config

| YAML key | Description | Optional | Default | Environment variable |
| :--- | :--- | :--- | :--- | :--- |
| `enabled` |  Set to `True` to enable HTTP basic auth, `False` to disable | Yes | `False` | `NOIP_RFC2136_AUTH_ENABLED` |
| `username` | String containing HTTP basic username | Yes | `None` | `NOIP_RFC2136_BASIC_AUTH_USERNAME` |
| `password` | String containing HTTP basic password | Yes | `None` | `NOIP_RFC2136_BASIC_AUTH_PASSWORD` |

### Logging config
| YAML key | Description | Optional | Default | Environment variable |
| :--- | :--- | :--- | :--- | :--- |
| `level` | The log level. Can be one of `NOTSET` `DEBUG` `INFO` `WARNING` `ERROR` `CRITICAL` | Yes | `INFO` | `NOIP_RFC2136_LOG_LEVEL` |

### UDM-Pro settings:
To use with a UDM-Pro, put these settings in the Dynamic DNS section of the Unifi Controller GUI:
 * Service: `No-IP`
 * Hostname: `dyndns.example.com`
 * Username: As configured in `basic_auth_user`
 * Password: As configured in `basic_auth_pass`
 * Server: `example.com/\/nic/update?hostname=%h&myip=%i`

The `Server` field should be the URL where `noip-rfc2136.py` can be accessed. I have not found a way to escape a port number into the unifi config, although the underlying process `inadyn` supports this.
Instead I have a HTTPS-enabled webserver running in front of this script that proxies all requests to `/nic/*` to `http://127.0.0.1:8000`. There's some discussion on the how this piece of functionality works
[on the UI forums](https://community.ui.com/questions/UDM-DynDNS-Google-Domains/fe9ba35d-66c3-437d-8323-debe2af55879)

## Future enhancements
Not much planned, but would nice to be able to fully support IPv6 as per the No-IP API.
