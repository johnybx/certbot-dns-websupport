# certbot-dns-websupport

[Websupport.sk](https://www.websupport.sk) DNS Authenticator plugin for Certbot

This plugin automates the process of completing a `dns-01` challenge by
creating, and subsequently removing, TXT records using the Websupport Remote API.

---

## Installation

```bash
pip install certbot-dns-websupport
```

---

## Named Arguments

To start using DNS authentication for Websupport, pass the following arguments on
certbot's command line:

| Command                                                   | Description                                                                                                  |
| --------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| `--authenticator dns-websupport`                          | select the authenticator plugin (Required)                                                                   |
| `--dns-websupport-credentials "/path/to/credentials.ini"` | websupport user INI file. (Required)                                                                         |
| `--dns-websupport-propagation-seconds "120"`              | waiting time for DNS to propagate before the ACMEserver to verify the DNS (Default: 60, Recommended: >= 120) |

---

## Credentials file

Obtain an identifier and secret key (see [Account Page](https://admin.websupport.sk/sk/auth/apiKey))

An example `credentials.ini` file:

```ini
dns_websupport_identifier = <identifier>
dns_websupport_secret_key = <secret_key>
```

The path to this file can be provided interactively or using the
`--dns-websupport-credentials` command-line argument. Certbot
records the path to this file for use during renewal, but does not store the
file's contents.

**CAUTION:** You should protect these API credentials as you would the
password to your websupport account. Users who can read this file can use these
credentials to issue arbitrary API calls on your behalf. Users who can cause
Certbot to run using these credentials can complete a `dns-01` challenge to
acquire new certificates or revoke existing certificates for associated
domains, even if those domains aren't being managed by this server.

Certbot will emit a warning if it detects that the credentials file can be
accessed by other users on your system. The warning reads "Unsafe permissions
on credentials configuration file", followed by the path to the credentials
file. This warning will be emitted each time Certbot uses the credentials file,
including for renewal, and cannot be silenced except by addressing the issue
(e.g., by using a command like `chmod 600` to restrict access to the file).

<br>

**It is suggested to secure the .ini folder as follows:**

```commandline
chown root:root /etc/letsencrypt/.secrets
chmod 600 /etc/letsencrypt/.secrets
```

---

## Direct command

To acquire a single certificate for `*.example.com`, waiting 600 seconds for DNS propagation:

```commandline
certbot certonly \
    --authenticator dns-websupport \
    --dns-websupport-propagation-seconds "600" \
    --dns-websupport-credentials "/etc/letsencrypt/.secrets/credentials.ini" \
    --email full.name@example.com \
    --agree-tos \
    --non-interactive \
    --rsa-key-size 4096 \
    -d *.example.com
```

**NOTE:** Don't forget to properly edit your ini file name and mail address.

---

## Auto renew

Add cronjob:

```commandline
0 3 * * * certbot renew \
    --authenticator dns-websupport \
    --dns-websupport-propagation-seconds "600" \
    --dns-websupport-credentials "/etc/letsencrypt/.secrets/credentials.ini" \
    --post-hook "systemctl reload nginx"
```

## Docker

In order to create a docker container with a certbot-dns-websupport installation,
create an empty directory with the following `Dockerfile`:

```dockerfile
FROM certbot/certbot
RUN pip3 install certbot-dns-websupport
```

<br>

Proceed to build the image:

```commandline
docker build -t certbot/dns-websupport .
```

<br>

Once that's finished, the application can be run as follows:

```commandline
sudo docker run -it --rm \
    -v /var/lib/letsencrypt:/var/lib/letsencrypt \
    -v /etc/letsencrypt:/etc/letsencrypt \
    certbot/dns-websupport \
    certonly \
    --authenticator dns-websupport \
    --dns-websupport-propagation-seconds "600" \
    --dns-websupport-credentials "/etc/letsencrypt/.secrets/credentials.ini" \
    --email full.name@example.com \
    --agree-tos \
    --non-interactive \
    --rsa-key-size 4096 \
    -d *.example.com
```

**NOTE:** Check if your volumes on host system match this example (Depends if you installed your server on host system or inside docker). If not, you will have to edit this command. Also don't forget to properly edit your ini file name and mail address.

---
