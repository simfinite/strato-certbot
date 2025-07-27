# strato-certbot
Wildcard certificates for strato.de

## Install

### Install dependencies globally

The hook scripts provided by this project can be directly executed from the cloned repository. In this case, dependencies specified in the `pyproject.toml` must be globally installed, which can be easiest achieved using the `uv` package manager: `sudo uv pip install -r pyproject.toml`.

### Install as package via pipx

If global installation of dependencies is not an option (e.g. recent Ubuntu versions), installation via `pipx` into a dedicated virtual environment is recommended. As certbot is run from a cron job with root privileges, the hook scripts provided by this project must be globally available. This can be achieved with pipx by setting `PIPX_HOME` and `PIPX_BIN_DIR`:
```
sudo PIPX_HOME=/opt/pipx PIPX_BIN_DIR=/usr/local/bin pipx install --force git+https://https://github.com/Buxdehuda/strato-certbot
```

This install all dependencies into a dedicated virtual environment within `/opt/pipx` and making the hook scripts available via entry points `strato-auth-hook` and `strato-cleanup-hook`.

Add the `--force` option for upgrading to the latest version.


## Setup

Create the file `strato-auth.json` and place it either in a safe folder, e.g. 
`/root/.secrets`, or into the folder the repository was cloned to if not 
installing strato-certbot as a package.

Make sure to make this file only readable for root:

`sudo chmod 0400 strato-auth.json`

```json
{
  "api_url": "https://www.strato.de/apps/CustomerService",
  "username": "<username>",
  "password": "<password>"
}
```

The api url needs to be filled with the correct url from your country. 
So as an example for Germany its 'https://www.strato.de/apps/CustomerService', but for the Netherlands its 'https://www.strato.nl/apps/CustomerService#skl'


### Two-Factor Authentification

To be able to authenticate two-factor, device name and TOTP secret must be entered into the JSON. If it is not used, it can either be empty strings or the entries can be removed completely (see above).

```json
{
  "api_url": "https://www.strato.de/apps/CustomerService",
  "username": "<username>",
  "password": "<password>",
  "totp_secret": "<secret>",
  "totp_devicename": "<devicename>"
}
```

### Waiting time

Sometimes it takes a while until the desired DNS record is published, which allows Certbot to verify the domain. To prevent this, a waiting time can be set.

```json
{
  "api_url": "https://www.strato.de/apps/CustomerService",
  "username": "<username>",
  "password": "<password>",
  "waiting_time": <seconds>
}
```

## Get certificate

Run Certbot in manual mode with the hook scripts automating the process.

An absolute path to the `strato_auth.json` must be specified as argument to the hook scripts, e.g. via a variable `STRATO_AUTH_PATH`.

If installed as a package via pipx, the hook scripts are globally available:
```shell
sudo certbot certonly --manual --preferred-challenges dns --manual-auth-hook "strato-auth-hook $STRATO_AUTH_PATH" --manual-cleanup-hook "strato-cleanup-hook $STRATO_AUTH_PATH" -d example.com -d *.example.com
```

If running directly from a cloned repository, the absolute paths to the hook scripts must be specified:
```shell
sudo certbot certonly --manual --preferred-challenges dns --manual-auth-hook "$(pwd)/strato_certbot/auth_hook.py $STRATO_AUTH_PATH" --manual-cleanup-hook "$(pwd)/strato_certbot/cleanup_hook.py $STRATO_AUTH_PATH" -d example.com -d *.example.com
```

This will generate a wildcard certificate for your domain without the need to manually enter the TXT records.

## Docker

The Dockerfile wraps these hook scripts into a certbot runtime.
The result is a volume with certificates.

### Setup

Change into the `docker` directory.

Edit and copy the certbot.env.sample to certbot.env

### Build

Run `./build.sh`

### Run

Run `./run.sh`

### Get certificates

A docker volume named "letsencrypt" will be created, the certificates can be found there ( `docker volume inspect letsencrypt` )


## Development Hints

Use certbot's `--dry-run` option to test the whole process without acquiring an actual valid certificate:

```shell
sudo certbot certonly --manual --preferred-challenges dns --manual-auth-hook "strato-auth-hook $STRATO_AUTH_PATH" --manual-cleanup-hook "strato-cleanup-hook $STRATO_AUTH_PATH" -d example.com -d *.example.com --dry-run
```

Test the hook scripts separately:
```shell
CERTBOT_DOMAIN=example.com CERTBOT_VALIDATION=test_value strato_certbot/auth_hook.py $STRATO_AUTH_PATH
CERTBOT_DOMAIN=example.com CERTBOT_VALIDATION=test_value strato_certbot/cleanup_hook.py $STRATO_AUTH_PATH
```