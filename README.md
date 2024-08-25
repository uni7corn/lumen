# Lumen

A private Lumina server that can be used with IDA Pro 7.2+.

[lumen.abda.nl](https://lumen.abda.nl/) runs this server.

You can read about the protocol research [here](https://abda.nl/posts/introducing-lumen/).

## Features

- Stores function signatures so you (and your team) can quickly identify functions that you found in the past using IDA's built-in Lumina features.
- Backed by PostgreSQL
- Experimental HTTP API that allows querying the database for comments by file or function hash.

## Getting Started

### Docker Method (Recommended)

In this method precompiled docker images will be downloaded, All you need is [docker-compose.yml](./docker-compose.yml).

1. Install `docker-engine` and `docker-compose`.
2. If using a custom TLS certificate, copy the private key (`.p12`/`.pfx` extension) to `./dockershare` and set the key password in `.env` as `PKCSPASSWD`.
3. If using a custom Lumen config, copy it to `./dockershare/config.toml`.
4. Otherwise, or if you have finished these steps, just run `docker-compose up`.
5. Regardless, if TLS is enabled in the `config.toml`, a `hexrays.crt` will be generated in `./dockershare` to be copied to the IDA install directory.

### Building from source with Rust

1. `git clone https://github.com/naim94a/lumen.git`
2. Get a rust toolchain: https://rustup.rs/
3. `cd lumen`
4. Setup a the database

   - install postgres
   - install diesel-cli and run migrations:

     ```bash
     cargo install diesel_cli --no-default-features -Fpostgres
     diesel --config-file common/diesel.toml \
        --database-url postgres://postgres:password@localhost/lumen \
        migration run
     ```

5. `cargo build --release`

### Usage

```bash
./lumen -c config.toml
```

### Configuring IDA

#### IDA Pro >= 8.1

If you used LUMEN in the past, remove the LUMINA settings in the ida.cfg or idauser.cfg files, otherwise you will get a warning about
bad config parameters.

##### Setup under Linux :

```bash
#!/bin/sh
export LUMINA_TLS=false
$1
```

- save as ida_lumen.sh, "chmod +x ida_lumen.sh", now you can run IDA using "./ida_lumen.sh ./ida" or "./ida_lumen ./ida64"

##### Setup under Windows :

```batch
set LUMINA_TLS=false
%1
```

- save as ida_lumen.bat, now you can run IDA using "./ida_lumen.bat ida.exe" or "./ida_lumen.bat ida64.exe"

##### Setup IDA

- Go to Options, General, Lumina. Select "Use a private server", then set your host and port and "guest" as username and password. Click on ok.

#### IDA Pro < 8.1

You will need IDA Pro 7.2 or above in order to use _lumen_.

> The following information may get sent to _lumen_ server: IDA key, Hostname, IDB path, original file path, file MD5, function signature, stack frames & comments.

- In your IDA's installation directory open "cfg\ida.cfg" with your favorite text editor _(Example: C:\Program Files\IDA Pro 7.5\cfg\ida.cfg)_
- Locate the commented out `LUMINA_HOST`, `LUMINA_PORT`, and change their values to the address of your _lumen_ server.
- If you didn't configure TLS, Add "LUMINA_TLS = NO" after the line with `LUMINA_PORT`.

Example:

```C
LUMINA_HOST = "192.168.1.1";
LUMINA_PORT = 1234

// Only if TLS isn't used:
LUMINA_TLS = NO
```

### Configuring TLS

IDA Pro uses a pinned certificate for Lumina's communcation, so adding a self-signed certificate to your root certificates won't work.
Luckily, we can override the hard-coded public key by writing a DER-base64 encoded certificate to "hexrays.crt" in IDA's install directory.

You may find the following commands useful:

```bash
# create a certificate
openssl req -x509 -newkey rsa:4096 -keyout lumen_key.pem -out lumen_crt.pem -days 365 -nodes

# convert to pkcs12 for lumen; used for `lumen.tls` in config
openssl pkcs12 -export -out lumen.p12 -inkey lumen_key.pem -in lumen_crt.pem

# export public-key for IDA; Copy hexrays.crt to IDA installation folder
openssl x509 -in lumen_crt.pem -out hexrays.crt
```

No attempt is made to merge function data - this may cause a situation where metadata is inconsistent.
Instead, the metadata with the highest calculated score is returned to the user.

---

Developed by [Naim A.](https://github.com/naim94a); License: MIT.
