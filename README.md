# postgres-cloudflare-docker

Create a Tunnel with the name provided and associate it with a UUID. The relationship between the UUID and the name is persistent. The command will not create a connection at this point.

The created Tunnel can serve traffic for multiple hostnames in your Cloudflare account and send traffic to multiple services available to cloudflared, including SSH, RDP, and most arbitrary TCP connections.

Query Postgres from Workers using a database connector. Retrieve data in your Cloudflare Workers applications from a PostgreSQL database using Postgres database connector

Set up a tunnel locally (CLI setup)

```java
1. Download and install cloudflared
   .deb install
   ​.rpm install
2. Authenticate cloudflared
3. Create a tunnel and give it a name
4. Create a configuration file
5. Start routing traffic
6. Run the tunnel
7. Check the tunnel

1. Overview
2. Basic project scaffolding
3. Cloudflare Tunnel authentication
4. Start and prepare Postgres database
    4.1. Start the Postgres server
    4.2. Import example dataset
5. Edit Worker and query Pagila dataset
    5.1. Database connection settings
    5.2. Query Pagila dataset
6. Worker deployment
    6.1. Set secrets
    6.2. Test the Worker
7. Cleanup
```

# 🚀 Javascript full-stack 🚀

https://github.com/coding-to-music/postgres-cloudflare-docker

By Cloudflare Documentation

https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-useful-terms/#default-cloudflared-directory

https://developers.cloudflare.comhttps://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide/#set-up-a-tunnel-locally-cli-setup

https://developers.cloudflare.comhttps://developers.cloudflare.com/cloudflare-one/connections/connect-apps/create-tunnel/

https://github.com/cloudflare/cloudflare-docs/blob/production/contenthttps://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide.md

https://github.com/cloudflare/cloudflare-docs/blob/production/contenthttps://developers.cloudflare.com/cloudflare-one/connections/connect-apps/create-tunnel/index.md

https://developers.cloudflare.com/workers/tutorials/query-postgres-from-workers-using-database-connectors/

## Environment Values

```java

```

## GitHub

```java
git init
git add .
git remote remove origin
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:coding-to-music/postgres-cloudflare-docker.git
git push -u origin main
```

## Naming and storing a configuration file

cloudflared will automatically look for a config.yaml or config.yml file in the default cloudflared directory .

You can give your configuration file a custom name and store it in any directory. However, when running tunnel, make sure to add the --config flag and specify the new path.

```java
cloudflared tunnel --config /path/your-config-file.yaml run tunnel-name
```

# How to setup a Cloudflare Tunnel

https://dev.to/realchaika/how-to-setup-a-cloudflare-tunnel-on-linux-40d9

## Installing Cloudflared
Cloudflare Tunnels use Cloudflared, a tunneling daemon to proxy the traffic from Cloudflare, and also to provide a CLI interface to make and manage tunnels.

### .deb install (Ubuntu, Linux Mint, Debian, etc)
```java
wget -q https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb && sudo dpkg -i cloudflared-linux-amd64.deb
```
### ​ .rpm install (Centos, Fedora, Rhel, OpenSusu, etc)
```java
wget -q https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-x86_64.rpm && sudo rpm -i cloudflared-linux-x86_64.rpm 
```

## Login to Cloudflared

```java
cloudflared tunnel login
```

This command should give you the link to sign into Cloudflare, and select a zone (website) to create tunnels on.

When done, it will download an account certificate (cert.pem file in the default cloudflared directory). This cert will be used to authorize future API Requests to create and manage tunnels. Once your tunnel is up and running, it will use its own credentials file, and you can safely delete this unless you want to keep managing/creating/deleting tunnels from this machine.

## Create a tunnel

```java
cloudflared tunnel create <name>
```

This command will create a named tunnel based on the name entered. It will generate a new tunnel, this includes generating a UUID for the tunnel, a tunnel credentials file in the default cloudflared directory, and a subdomain of .cfargotunnel.com that you can use to route requests to.

In this example, I'll be naming my tunnel "frontpage".

## Create your tunnel configuration file

Throughout the past two steps, after logging in and creating the account cert, and making a tunnel, generating the tunnel cert, cloudflared has listed the path to your .cloudflared directory, which is most likely based off your home directory.

```java
Something like "~/.cloudflared" or "/home/{username}/.cloudflared"
```

Navigate to that folder now. You should see cert.pem (your account cert) and a .json file named off the UUID of your tunnel.

Create a new file in the same directory, config.yml, and open it using your preferred text editor.

```java
url: http://localhost:80
tunnel: <Tunnel-UUID>
credentials-file: /home/{username}/.cloudflared/<Tunnel-UUID>.json
```

The URL line corresponds to the internal service you wish to expose. It's not necessary to use https://, the connection between Cloudflare Tunnel and Cloudflare's datacenter is already encrypted. This is just the tunnel connecting locally to the web server.

The Tunnel UUID is a 36 character value that corresponds with your named tunnel. It was displayed when you made the tunnel. You can also find it by going to your .cloudflared directory and looking for the newly created json credentials file for the tunnel you made. It should be named {Tunnel-UUID}.json.

## Route traffic to your tunnel

You just create a CNAME Record to route traffic to your tunnel. You can do so easily using the cloudflared cli

```java
cloudflared tunnel route dns <Tunnel UUID or Name> <Hostname>
```

For example, my tunnel is named frontpage and I wanted it to be accessible via example.chaika.dev. So 

I did

```java
cloudflared tunnel route dns frontpage example.chaika.dev
```

## Run your tunnel

Finally, you can test out your tunnel.

```java
cloudflared tunnel run <UUID or Name>
```

You can also specify a specific configuration file to run
```java
cloudflared tunnel --config path/config.yaml run
```

Once your tunnel is live, try accessing it via the hostname you routed it to. It may take a few seconds for the tunnel to be fully live/accessible. If something is wrong, the tunnel running in the CLI should tell you more information about errors.

## Run your tunnel as a service

Running your tunnel manually will work, but isn't the best. It won't automatically start if your machine reboots, have to ensure its open/running, etc.

Luckily, cloudflared supports installing itself as a service very easily.

```java
sudo cloudflared service install
```

You may need to manually specify config location. In my case, I did have to specify it.

For example,

```java
sudo cloudflared --config /home/{username}/.cloudflared/config.yml  service install 
```

Note that you specify the config argument before the 'service install' command parameters.

The configuration will be copied over to /etc/cloudflared

I would recommend copying over the tunnel credentials file ({Tunnel-UUID}.json) over to there as well.

Then, just launch the service and set it to start on boot

```java
sudo systemctl enable cloudflared
sudo systemctl start cloudflared
```

Ensure your tunnel started/is running fine:

```java
sudo systemctl status cloudflared
```

Test out your tunnel by visting the hostname you routed it to.


## Example: cloudflared run in docker

https://gist.github.com/joejordanbrown/b63f82a298da208a5e4780c2200af8fb

# Example: cloudflared run in docker

- https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup
- https://hub.docker.com/r/cloudflare/cloudflared
- https://github.com/cloudflare/cloudflared/

---

1. Authenticate with Cloudflare -> cloudflared tunnel login
```shell
sudo docker run -it --rm --name=cloudflared -v /root/.cloudflared:/home/nonroot/.cloudflared cloudflare/cloudflared:2022.2.0 tunnel login
```

2. Create tunnel -> cloudflared tunnel create `<tunnel-name>`
```shell
sudo docker run -it --rm --name=cloudflared -v /root/.cloudflared:/home/nonroot/.cloudflared cloudflare/cloudflared:2022.2.0 tunnel create example-tunnel
```

3. Add DNS route internal.example.com to tunnel -> cloudflared tunnel route dns `<tunnel-name>` `<route-hostname>`
```shell
sudo docker run -it --rm --name=cloudflared -v /root/.cloudflared:/home/nonroot/.cloudflared cloudflare/cloudflared:2022.2.0 tunnel route dns example-tunnel internal.example.com
```

4. Add DNS route app1.example.com to tunnel -> cloudflared tunnel route dns `<tunnel-name>` `<route-hostname>`
```shell
sudo docker run -it --rm --name=cloudflared -v /root/.cloudflared:/home/nonroot/.cloudflared cloudflare/cloudflared:2022.2.0 tunnel route dns example-tunnel app1.example.com
```

5. Run tunnel in detached docker container -> cloudflared tunnel run
```shell
sudo docker run -it --rm --name=cloudflared --network="host" -d -v /root/.cloudflared:/home/nonroot/.cloudflared cloudflare/cloudflared:2022.2.0 tunnel run
```

/root/.cloudflared/config.yml
```yaml
tunnel: *******************
credentials-file: /root/.cloudflared/***-*-*-*-****.json

ingress:
  - hostname: internal.example.com
    service: https://127.0.0.1:8443
    originRequest:
      noTLSVerify: true

  - hostname: app1.example.com
    service: http://192.168.1.10:8990

  - service: http_status:404
```

## Possible helpful GitHub issue

https://github.com/cloudflare/cloudflared/issues/504

References this article: https://ilayk.com/2021/03/25/cloudflared

```java
The Docker docs should also cover the config.yml more.

For those of you who stumble accross this issue in the future, here is my Docker cli command for creating the container. Note that you will have to tunnel login and tunnel create tunnel_name_here before running the tunnel.

Command:

docker run -d \
  --name cloudflared \
  -v ~/.config/cloudflared:/home/nonroot/.cloudflared/ \
  cloudflare/cloudflared:2021.11.0-amd64 \
  tunnel run ubuntu

config.yml

tunnel: tunnel_id_goes_here # output to terminal when running tunnel login
credentials-file: /home/nonroot/.cloudflared/credential_file_here.json
ingress:
   - hostname: mywebsite.com # your domain goes here
     service: http://localhost:8080 # service you want to expose
   - service: http://localhost:404 # backup service that will return 404 error from Cloudflare
```

Another example of a config.yml

```java
tunnel: Tunnel ID
credentials-file: /home/leon/.cloudflared/TunnelID.json
ingress:
        - hostname: leonnunes.dev
          service: http://localhost:8080
        #Catch-all rule, which just responds with 404 if traffic doesn't match any of
          #   # the earlier rules
        - service: http_status:404
```

## Cloudflared documentation

```java
NAME:
   cloudflared tunnel - Use Cloudflare Tunnel to expose private services to the Internet or to Cloudflare connected private users.

USAGE:
   cloudflared tunnel command [command options]  

DESCRIPTION:
   Cloudflare Tunnel allows to expose private services without opening any ingress port on this machine. It can expose:
     A) Locally reachable HTTP-based private services to the Internet on DNS with Cloudflare as authority (which you can
   then protect with Cloudflare Access).
     B) Locally reachable TCP/UDP-based private services to Cloudflare connected private users in the same account, e.g.,
   those enrolled to a Zero Trust WARP Client.
   
   You can manage your Tunnels via dash.teams.cloudflare.com. This approach will only require you to run a single command
   later in each machine where you wish to run a Tunnel.
   
   Alternatively, you can manage your Tunnels via the command line. Begin by obtaining a certificate to be able to do so:
   
     $ cloudflared tunnel login
   
   With your certificate installed you can then get started with Tunnels:
   
     $ cloudflared tunnel create my-first-tunnel
     $ cloudflared tunnel route dns my-first-tunnel my-first-tunnel.mydomain.com
     $ cloudflared tunnel run --hello-world my-first-tunnel
   
   You can now access my-first-tunnel.mydomain.com and be served an example page by your local cloudflared process.
   
   For exposing local TCP/UDP services by IP to your privately connected users, check out:
   
     $ cloudflared tunnel route ip --help
   
   See https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide/ for more info.

COMMANDS:
   login    Generate a configuration file with your login details
   create   Create a new tunnel with given name
   route    Define which traffic routed from Cloudflare edge to this tunnel: requests to a DNS hostname, to a Cloudflare Load Balancer, or traffic originating from Cloudflare WARP clients
   vnet     Configure and query virtual networks to manage private IP routes with overlapping IPs.
   run      Proxy a local web server by running the given tunnel
   list     List existing tunnels
   info     List details about the active connectors for a tunnel
   delete   Delete existing tunnel by UUID or name
   cleanup  Cleanup tunnel connections
   token    Fetch the credentials token for an existing tunnel (by name or UUID) that allows to run it
   help, h  Shows a list of commands or help for one command

OPTIONS:
   --config value                                      Specifies a config file in YAML format.
   --origincert value                                  Path to the certificate generated for your origin when you run cloudflared login. [$TUNNEL_ORIGIN_CERT]
   --autoupdate-freq value                             Autoupdate frequency. Default is 24h0m0s. (default: 24h0m0s)
   --no-autoupdate                                     Disable periodic check for updates, restarting the server with the new version. (default: false) [$NO_AUTOUPDATE]
   --metrics value                                     Listen address for metrics reporting. (default: "localhost:") [$TUNNEL_METRICS]
   --pidfile value                                     Write the application's PID to this file after first successful connection. [$TUNNEL_PIDFILE]
   --url URL                                           Connect to the local webserver at URL. (default: "http://localhost:8080") [$TUNNEL_URL]
   --hello-world                                       Run Hello World Server (default: false) [$TUNNEL_HELLO_WORLD]
   --socks5 --url                                      specify if this tunnel is running as a SOCK5 Server This flag only takes effect if you define your origin with --url and if you do not use ingress rules. The recommended way is to rely on ingress rules and define this property under `originRequest` as per https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/configuration/configuration-file/ingress (default: false) [$TUNNEL_SOCKS]
   --proxy-connect-timeout --url                       HTTP proxy timeout for establishing a new connection This flag only takes effect if you define your origin with --url and if you do not use ingress rules. The recommended way is to rely on ingress rules and define this property under `originRequest` as per https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/configuration/configuration-file/ingress (default: 30s)
   --proxy-tls-timeout --url                           HTTP proxy timeout for completing a TLS handshake This flag only takes effect if you define your origin with --url and if you do not use ingress rules. The recommended way is to rely on ingress rules and define this property under `originRequest` as per https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/configuration/configuration-file/ingress (default: 10s)
   --proxy-tcp-keepalive --url                         HTTP proxy TCP keepalive duration This flag only takes effect if you define your origin with --url and if you do not use ingress rules. The recommended way is to rely on ingress rules and define this property under `originRequest` as per https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/configuration/configuration-file/ingress (default: 30s)
   --proxy-no-happy-eyeballs --url                     HTTP proxy should disable "happy eyeballs" for IPv4/v6 fallback This flag only takes effect if you define your origin with --url and if you do not use ingress rules. The recommended way is to rely on ingress rules and define this property under `originRequest` as per https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/configuration/configuration-file/ingress (default: false)
   --proxy-keepalive-connections --url                 HTTP proxy maximum keepalive connection pool size This flag only takes effect if you define your origin with --url and if you do not use ingress rules. The recommended way is to rely on ingress rules and define this property under `originRequest` as per https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/configuration/configuration-file/ingress (default: 100)
   --proxy-keepalive-timeout --url                     HTTP proxy timeout for closing an idle connection This flag only takes effect if you define your origin with --url and if you do not use ingress rules. The recommended way is to rely on ingress rules and define this property under `originRequest` as per https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/configuration/configuration-file/ingress (default: 1m30s)
   --proxy-connection-timeout value                    DEPRECATED. No longer has any effect. (default: 1m30s)
   --proxy-expect-continue-timeout value               DEPRECATED. No longer has any effect. (default: 1m30s)
   --http-host-header --url                            Sets the HTTP Host header for the local webserver. This flag only takes effect if you define your origin with --url and if you do not use ingress rules. The recommended way is to rely on ingress rules and define this property under `originRequest` as per https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/configuration/configuration-file/ingress [$TUNNEL_HTTP_HOST_HEADER]
   --origin-server-name --url                          Hostname on the origin server certificate. This flag only takes effect if you define your origin with --url and if you do not use ingress rules. The recommended way is to rely on ingress rules and define this property under `originRequest` as per https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/configuration/configuration-file/ingress [$TUNNEL_ORIGIN_SERVER_NAME]
   --unix-socket value                                 Path to unix socket to use instead of --url [$TUNNEL_UNIX_SOCKET]
   --origin-ca-pool --url                              Path to the CA for the certificate of your origin. This option should be used only if your certificate is not signed by Cloudflare. This flag only takes effect if you define your origin with --url and if you do not use ingress rules. The recommended way is to rely on ingress rules and define this property under `originRequest` as per https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/configuration/configuration-file/ingress [$TUNNEL_ORIGIN_CA_POOL]
   --no-tls-verify --url                               Disables TLS verification of the certificate presented by your origin. Will allow any certificate from the origin to be accepted. Note: The connection from your machine to Cloudflare's Edge is still encrypted. This flag only takes effect if you define your origin with --url and if you do not use ingress rules. The recommended way is to rely on ingress rules and define this property under `originRequest` as per https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/configuration/configuration-file/ingress (default: false) [$NO_TLS_VERIFY]
   --no-chunked-encoding --url                         Disables chunked transfer encoding; useful if you are running a WSGI server. This flag only takes effect if you define your origin with --url and if you do not use ingress rules. The recommended way is to rely on ingress rules and define this property under `originRequest` as per https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/configuration/configuration-file/ingress (default: false) [$TUNNEL_NO_CHUNKED_ENCODING]
   --bastion                                           Runs as jump host (default: false) [$TUNNEL_BASTION]
   --proxy-address value                               Listen address for the proxy. (default: "127.0.0.1") [$TUNNEL_PROXY_ADDRESS]
   --proxy-port value                                  Listen port for the proxy. (default: 0) [$TUNNEL_PROXY_PORT]
   --loglevel value                                    Application logging level {debug, info, warn, error, fatal}. At debug level cloudflared will log request URL, method, protocol, content length, as well as, all request and response headers. This can expose sensitive information in your logs. (default: "info") [$TUNNEL_LOGLEVEL]
   --transport-loglevel value, --proto-loglevel value  Transport logging level(previously called protocol logging level) {debug, info, warn, error, fatal} (default: "info") [$TUNNEL_PROTO_LOGLEVEL, $TUNNEL_TRANSPORT_LOGLEVEL]
   --logfile value                                     Save application log to this file for reporting issues. [$TUNNEL_LOGFILE]
   --log-directory value                               Save application log to this directory for reporting issues. [$TUNNEL_LOGDIRECTORY]
   --trace-output value                                Name of trace output file, generated when cloudflared stops. [$TUNNEL_TRACE_OUTPUT]
   --proxy-dns                                         Run a DNS over HTTPS proxy server. (default: false) [$TUNNEL_DNS]
   --proxy-dns-port value                              Listen on given port for the DNS over HTTPS proxy server. (default: 53) [$TUNNEL_DNS_PORT]
   --proxy-dns-address value                           Listen address for the DNS over HTTPS proxy server. (default: "localhost") [$TUNNEL_DNS_ADDRESS]
   --proxy-dns-upstream value                          Upstream endpoint URL, you can specify multiple endpoints for redundancy. (default: "https://1.1.1.1/dns-query", "https://1.0.0.1/dns-query")  (accepts multiple inputs) [$TUNNEL_DNS_UPSTREAM]
   --proxy-dns-max-upstream-conns value                Maximum concurrent connections to upstream. Setting to 0 means unlimited. (default: 5) [$TUNNEL_DNS_MAX_UPSTREAM_CONNS]
   --proxy-dns-bootstrap value                         bootstrap endpoint URL, you can specify multiple endpoints for redundancy. (default: "https://162.159.36.1/dns-query", "https://162.159.46.1/dns-query", "https://[2606:4700:4700::1111]/dns-query", "https://[2606:4700:4700::1001]/dns-query")  (accepts multiple inputs) [$TUNNEL_DNS_BOOTSTRAP]
   --credentials-file value, --cred-file value         Filepath at which to read/write the tunnel credentials [$TUNNEL_CRED_FILE]
   --region value                                      Cloudflare Edge region to connect to. Omit or set to empty to connect to the global region. [$TUNNEL_REGION]
   --hostname value                                    Set a hostname on a Cloudflare zone to route traffic through this tunnel. [$TUNNEL_HOSTNAME]
   --lb-pool value                                     The name of a (new/existing) load balancing pool to add this origin to. [$TUNNEL_LB_POOL]
   --metrics-update-freq value                         Frequency to update tunnel metrics (default: 5s) [$TUNNEL_METRICS_UPDATE_FREQ]
   --tag KEY=VALUE                                     Custom tags used to identify this tunnel, in format KEY=VALUE. Multiple tags may be specified  (accepts multiple inputs) [$TUNNEL_TAG]
   --retries value                                     Maximum number of retries for connection/protocol errors. (default: 5) [$TUNNEL_RETRIES]
   --grace-period value                                When cloudflared receives SIGINT/SIGTERM it will stop accepting new requests, wait for in-progress requests to terminate, then shutdown. Waiting for in-progress requests will timeout after this grace period, or when a second SIGTERM/SIGINT is received. (default: 30s) [$TUNNEL_GRACE_PERIOD]
   --compression-quality value                         (beta) Use cross-stream compression instead HTTP compression. 0-off, 1-low, 2-medium, >=3-high. (default: 0) [$TUNNEL_COMPRESSION_LEVEL]
   --name value, -n value                              Stable name to identify the tunnel. Using this flag will create, route and run a tunnel. For production usage, execute each command separately [$TUNNEL_NAME]
   --ui                                                Launch tunnel UI. Tunnel logs are scrollable via 'j', 'k', or arrow keys. (default: false)
   --overwrite-dns, -f                                 Overwrites existing DNS records with this hostname (default: false) [$TUNNEL_FORCE_PROVISIONING_DNS]
   --help, -h                                          show help (default: false)
```


# Set up your first tunnel

https://github.com/cloudflare/cloudflare-docs/blob/production/contenthttps://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide.md

title: Set up your first tunnel

When setting up your first Cloudflare Tunnel, you have the option to create it:

- [Remotely on the Zero Trust dashboard](#set-up-a-tunnel-remotely-dashboard-setup)
- [Locally, using your CLI](#set-up-a-tunnel-locally-cli-setup)

## Prerequisites

Before you start, make sure you:

- [Add a website to Cloudflare](https://support.cloudflare.com/hc/en-us/articles/201720164-Creating-a-Cloudflare-account-and-adding-a-website).
- [Change your domain nameservers to Cloudflare](https://support.cloudflare.com/hc/en-us/articles/205195708).

First, download `cloudflared` on your machine. Visit the [downloads](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/installation/) page to find the right package for your OS.

Next, install `cloudflared`.

#### .deb install

Use the deb package manager to install `cloudflared` on compatible machines. `amd64 / x86-64` is used in this example.

```sh
wget -q https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb && dpkg -i cloudflared-linux-amd64.deb
```

#### ​.rpm install

Use the rpm package manager to install `cloudflared` on compatible machines. `amd64 / x86-64` is used in this example.

```sh
wget -q https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-x86_64.rpm
```

</div>
</details>

<details>
<summary>Build from source</summary>
<div>

You can also build the latest version of `cloudflared` from source with the following steps.

```sh
git clone https://github.com/cloudflare/cloudflared.git
cd cloudflared
make cloudflared
go install github.com/cloudflare/cloudflared/cmd/cloudflared
```

Depending on where you installed `cloudflared`, you can move it to a known path as well.

```bash
mv /root/cloudflared/cloudflared /usr/bin/cloudflared
```

</div>
</details>


# Useful terms

{{<render file="_cloudflared-new-ui.md">}}

Review terminology for tunnels setup locally through the CLI.

## Tunnel

A tunnel is a secure, outbound-only pathway you can establish between your origin and the Cloudflare edge. Each tunnel you create will be assigned a [name](#tunnel-name) and a [UUID](#tunnel-uuid).

## Tunnel UUID

A tunnel UUID is an alphanumeric, unique ID assigned to a tunnel. The tunnel UUID can be used in [configuration files](#configuration-file), and in general, whenever you need to reference a specific tunnel.

## Tunnel name

The `cloudflared tunnel create <NAME>` command creates a tunnel and assigns it a name. Once named, a tunnel is a persistent pathway within which you can stop and start as many [connectors](#connector) as needed, adding stability and ease of use to your tunnel experience. Tunnel names do not need to be hostnames; for example, you can assign your tunnel a name that represents your application/network, a particular server, or the cloud environment where it runs. Just choose any identifier that lets you easily reference a tunnel whenever you need.

## Connector

You can create and configure a tunnel once and run it as multiple different `cloudflared` processes. These processes are known as connectors, or replicas. DNS records and Cloudflare Load Balancers can still point to the tunnel and its UUID, while that tunnel sends traffic to the multiple instances of cloudflared that run through it. Using multiple connectors provides tunnels with high availability, scalability, and elasticity.

## Default `cloudflared` directory

`cloudflared` uses a default directory when storing credentials files for your tunnels, as well as the `cert.pem` file it generates when you run `cloudflared login`. The default directory is also where `cloudflared` will look for a [configuration file](#configuration-file) if no other file path is [specified](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/configuration/local-management/configuration-file/#storing-a-configuration-file) when running a tunnel.

| OS                          | Path to default directory                                                              |
| --------------------------- | -------------------------------------------------------------------------------------- |
| Windows                     | `%USERPROFILE%\.cloudflared`                                                           |
| MacOS and Unix-like systems | `~/.cloudflared`, `/etc/cloudflared`, and `/usr/local/etc/cloudflared`, in this order. |

## Configuration file

This is a `.yaml` file that functions as the operating manual for `cloudflared`. `cloudflared` will automatically look for the configuration file in the [default `cloudflared` directory](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-useful-terms/#default-cloudflared-directory), but you can store your configuration file in any directory. It is recommended to always specify the file path for your configuration file whenever you reference it. By creating a configuration file, you can have fine-grained control over how their instance of `cloudflared` will operate. This includes operations like what you want `cloudflared` to do with traffic (for example, proxy websockets to port `xxxx`, or ssh to port `yyyy`), where `cloudflared` should search for authorization (credentials file, tunnel token), and what mode it should run in (for example, [`warp-routing`](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/private-net/)). In the absence of a configuration file, cloudflared will proxy outbound traffic through port `8080`. For more information on how to create, store, and structure a configuration file, refer to the [dedicated instructions](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/configuration/local-management/configuration-file/).

## Ingress rule

[Ingress rules](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/configuration/local-management/ingress/) let you specify which local services traffic should be proxied to. If a rule does not specify a path, all paths will be matched. Ingress rules can be listed in your [configuration file](#configuration-file) or when running `cloudflared tunnel ingress`.

## Cert.pem

This is the certificate file issued by Cloudflare when you run `cloudflared tunnel login`. This file uses a certificate to authenticate your instance of `cloudflared` and it is required when you create new tunnels, delete existing tunnels, change DNS records, or configure tunnel routing from cloudflared. This file is not required to perform actions such as running an existing tunnel or managing tunnel routing from the Cloudflare dashboard. Refer to the [Tunnel permissions page](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-permissions/) for more details on when this file is needed.

The `cert.pem` origin certificate is valid for at least 10 years, and the service token it contains is valid until revoked.

## Credentials file

This file is created when you run `cloudflared tunnel create <NAME>`. It stores your tunnel’s credentials in JSON format, and is unique to each tunnel. This file functions as a token authenticating the tunnel it is associated with. Refer to the [Tunnel permissions page](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-permissions/) for more details on when this file is needed.

## Quick tunnels

Quick tunnels, when run, will generate a URL that consists of a random subdomain of the website `trycloudflare.com`, and point traffic to localhost on port 8080. If you have a web service running at that address, users who visit the generated subdomain will be able to visit your web service through Cloudflare’s network. Refer to [TryCloudflare](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/run-tunnel/trycloudflare/) for more information on how to run quick tunnels.

## Virtual Networks

A software abstraction that allows you to logically segregate resources on your private network. [Tunnel Virtual Networks](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/private-net/tunnel-virtual-networks/) are especially useful for exposing resources which have overlapping IP routes. To connect to a resource, end users would select a virtual network in their WARP client settings before entering the destination IP.

### 2. Authenticate `cloudflared`

```bash
cloudflared tunnel login
```

Running this command will:

- Open a browser window and prompt you to log in to your Cloudflare account. After logging in to your account, select your hostname.
- Generate an account certificate, the [cert.pem file](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-useful-terms/#cert-pem), in the [default `cloudflared` directory](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-useful-terms/#default-cloudflared-directory).

### 3. Create a tunnel and give it a name

```bash
cloudflared tunnel create <NAME>
```

Running this command will:

- Create a tunnel by establishing a persistent relationship between the [name you provide](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-useful-terms/#tunnel-name) and a [UUID](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-useful-terms/#tunnel-uuid) for your tunnel. At this point, no connection is active within the tunnel yet.
- Generate a [tunnel credentials file](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-useful-terms/#credentials-file) in the [default `cloudflared` directory](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-useful-terms/#default-cloudflared-directory).
- Create a subdomain of `.cfargotunnel.com`.

From the output of the command, take note of the tunnel’s UUID and the path to your tunnel’s credentials file.

Confirm that the tunnel has been successfully created by running:

```bash
cloudflared tunnel list
```

### 4. Create a configuration file

Create a [configuration file](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-useful-terms/#configuration-file) in your `.cloudflared` directory using any text editor. This file will configure the tunnel to route traffic from a given origin to the hostname of your choice.

Add the following fields to the file:

**If you are connecting an application**

```txt
url: http://localhost:8000
tunnel: <Tunnel-UUID>
credentials-file: /root/.cloudflared/<Tunnel-UUID>.json
```

**If you are connecting a network**

```txt
tunnel: <Tunnel-UUID>
credentials-file: /root/.cloudflared/<Tunnel-UUID>.json
warp-routing:
  enabled: true
```

Confirm that the configuration file has been successfully created by running:

```bash
cat config.yml
```

### 5. Start routing traffic

Now assign a CNAME record that points traffic to your tunnel subdomain.

**If you are connecting an application**

```bash
cloudflared tunnel route dns <UUID or NAME> <hostname>
```

**If you are connecting a network**

Add the IP/CIDR you would like to be routed through the tunnel.

```bash
cloudflared tunnel route ip add <IP/CIDR> <UUID or NAME>
```

You can confirm that the route has been successfully established by running:

```bash
cloudflared tunnel route ip show
```

### 6. Run the tunnel

Run the tunnel to proxy incoming traffic from the tunnel to any number of services running locally on your origin.

```bash
cloudflared tunnel run <UUID or NAME>
```

If your configuration file has a custom name or is not in the `.cloudflared` directory, add the `--config` flag and specify the path.

```sh
cloudflared tunnel --config /path/your-config-file.yaml run
```

{{<Aside>}}

Cloudflare Tunnel can install itself as a system service on Linux and Windows and as a launch agent on macOS. For more information, refer to [Run as a service](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/run-tunnel/as-a-service/).

{{</Aside>}}

### 7. Check the tunnel

Your tunnel configuration is complete! If you want to get information on the tunnel you just created, you can run:

```bash
cloudflared tunnel info
```

pcx-content-type: how-to

title: Create a Tunnel

# Create a Tunnel

| Before you start                                                                                                                                |
| ----------------------------------------------------------------------------------------------------------------------------------------------- |
| 1. [Add a website to Cloudflare](https://support.cloudflare.com/hc/en-us/articles/201720164-Creating-a-Cloudflare-account-and-adding-a-website) |
| 2. [Change your domain nameservers to Cloudflare](https://support.cloudflare.com/hc/en-us/articles/205195708)                                   |
| 3. [Install and authenticate `cloudflared`](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/)       |

## Create a Tunnel

To create a Tunnel, run the following command:

```sh
cloudflared tunnel create <NAME>
```

Replace `<NAME>` with the name you want to give to the Tunnel. The name assigned can be any string and does not need to relate to the hostname where traffic will be served.

This command will create a Tunnel with the name provided and associate it with a UUID. The relationship between the UUID and the name is persistent. The command will not create a connection at this point.

The created Tunnel can serve traffic for multiple hostnames in your Cloudflare account and send traffic to multiple services available to `cloudflared`, including SSH, RDP, and most arbitrary TCP connections.

![Create a tunnel](https://github.com/coding-to-music/postgres-cloudflare-docker/blob/main/images/ct1.png?raw=true)

Creating a Tunnel generates a credentials file for that specific Tunnel. This file is distinct from the cert.pem file. To run the Tunnel without managing DNS from `cloudflared`, you only need the credentials file.

{{<table-wrap>}}

| Action                                                              | `cert.pem` | Credentials file |
| ------------------------------------------------------------------- | ---------- | ---------------- |
| Create a new Tunnel                                                 | Required   | -                |
| Delete a Tunnel                                                     | Required   | -                |
| Run a Tunnel                                                        | Available  | Required         |
| Create DNS records<br/>from `cloudflared`                           | Required   | -                |
| Connect to load balancer<br/>pools from `cloudflared`               | Required   | -                |
| Route traffic to a running Tunnel<br/>from the Cloudflare dashboard | Available  | Available        |

{{</table-wrap>}}

## List available Tunnels

`cloudflared` can list all created Tunnels in your account, as well as those actively connected to Cloudflare, by running the following command:

```sh
cloudflared tunnel list
```

Note: the command requires the `cert.pem` file.

![List tunnels](https://github.com/coding-to-music/postgres-cloudflare-docker/blob/main/images/lt1.png?raw=true)

## Revoke and delete a Tunnel

You can delete an existing Tunnel with cloudflared. To delete a Tunnel, run the following command:

```sh
cloudflared tunnel delete <NAME>
```

{{<Aside>}}

The command requires the `cert.pem` file.

{{</Aside>}}

If there are still active connections on that Tunnel, then you will have to force the deletion with:

```sh
cloudflared tunnel delete -f <NAME>
```

This will cause those connections to be dropped.

Deleting the Tunnel also invalidates the credentials file associated with that Tunnel, meaning those connections can not be re-established.

{{<Aside>}}

Tunnels created in this method do not currently display in the **Traffic** tab of the [Cloudflare dashboard](https://dash.cloudflare.com). These connections will be added to the dashboard in a future release.

{{</Aside>}}

Cloudflare Tunnel deletes DNS records after 24-48 hours of a Tunnel being unregistered. Cloudflare Tunnel does not delete TLS certificates on your behalf once the Tunnel is shut down. If you want to clean up a Tunnel you’ve shut down, you can delete DNS records [in the DNS editor](https://dash.cloudflare.com/?zone=dns) and revoke TLS certificates in the Origin Certificates section of the [SSL/TLS tab of the Cloudflare dashboard](https://dash.cloudflare.com?to=/:account/:zone/ssl-tls/origin).

# Query Postgres from Workers using a database connector

<TutorialsBeforeYouStart />

## Overview

In this tutorial, you will learn how to retrieve data in your Cloudflare Workers applications from a PostgreSQL database using [Postgres database connector](https://github.com/cloudflare/worker-template-postgres).

{{<Aside type="note">}}

If you are using a MySQL database, refer to the [MySQL database connector](https://github.com/cloudflare/worker-template-mysql) template.

{{</Aside>}}

For a quick start, you will use Docker to run a local instance of Postgres and PgBouncer, and to securely expose the stack to the Internet using Cloudflare Tunnel.

## Basic project scaffolding

To get started:

1.  Run the following `git` command to clone a basic [Postgres database connector](https://github.com/cloudflare/worker-template-postgres) project.
2.  After running the `git clone` command, `cd` into the new project.

```sh
git clone https://github.com/cloudflare/worker-template-postgres/
cd worker-template-postgres
```

## Cloudflare Tunnel authentication

To create and manage secure Cloudflare Tunnels, you first need to authenticate `cloudflared` CLI.
Skip this step if you already have authenticated `cloudflared` locally.

```sh
docker run -v ~/.cloudflared:/etc/cloudflared cloudflare/cloudflared:2021.11.0 login

# should be without a specific old tag, use latest
docker run -v ~/.cloudflared:/etc/cloudflared cloudflare/cloudflared login
```

Running this command will:

- Prompt you to select your Cloudflare account and hostname.
- Download credentials and allow `cloudflared` to create Tunnels and DNS records.

## Start and prepare Postgres database

### Start the Postgres server

{{<Aside type="warning" header="Warning">}}

Cloudflare Tunnel will be accessible from the Internet once you run the following `docker compose` command. Cloudflare recommends that you secure your `TUNNEL_HOSTNAME` behind [Cloudflare Access](https://developers.cloudflare.com/cloudflare-one/applications/configure-apps/self-hosted-apps) before you continue.

{{</Aside>}}

You can find a prepared `docker-compose` file that does not require any changes in `scripts/postgres` with the following services:

1.  **postgres**
2.  **pgbouncer** - Placed in front of Postgres to provide connection pooling.
3.  **cloudflared** - Allows your applications to connect securely, through a encrypted tunnel, without opening any local ports.

Run the following commands to start all services. Replace `postgres-tunnel.example.com` with a hostname on your Cloudflare zone to route traffic through this tunnel.

```sh
cd scripts/postgres
export TUNNEL_HOSTNAME=postgres-tunnel.example.com

export TUNNEL_HOSTNAME=blog
docker-compose up

# Alternative: Run `docker-compose up -D` to start docker-compose detached
```

`docker-compose` will spin up and configure all the services for you, including the creation of the Tunnel's DNS record.
The DNS record will point to the Cloudflare Tunnel, which keeps a secure connection between a local instance of `cloudflared` and the Cloudflare network.

### Import example dataset

Once Postgres is up and running, seed the database with a schema and a dataset. For this tutorial, you will use the Pagila schema and dataset. Use `docker exec` to execute a command inside the running Postgres container and import [Pagila](https://github.com/devrimgunduz/pagila) schema and dataset.

```sh
curl https://raw.githubusercontent.com/devrimgunduz/pagila/master/pagila-schema.sql | docker exec -i postgres_postgresql_1 psql -U postgres -d postgres
curl https://raw.githubusercontent.com/devrimgunduz/pagila/master/pagila-data.sql | docker exec -i postgres_postgresql_1 psql -U postgres -d postgres
```

The above commands will download the SQL schema and dataset files from Pagila's GitHub repository and execute them in your local Postgres database instance.

## Edit Worker and query Pagila dataset

### Database connection settings

In `src/index.ts`, replace `https://dev.example.com` with your Cloudflare Tunnel hostname, ensuring that it is prefixed with the `https://` protocol:

```js
// src/index.ts

const client = new Client({
  user: 'postgres',
  database: 'postgres',
  hostname: 'https://REPLACE_WITH_TUNNEL_HOSTNAME',
  password: '',
  port: 5432,
});
```

At this point, you can deploy your Worker and make a request to it to verify that your database connection is working.

### Query Pagila dataset

The template script includes a simple query to select a number (`SELECT 42;`) that is executed in the database. Edit the script to query the imported Pagila dataset if the `pagila-table` query parameter is present.

```js
// Query the database.

// Parse the URL, and get the 'pagila-table' query parameter (which may not exist)
const url = new URL(request.url);
const pagilaTable = url.searchParams.get("pagila-table");

let result;
// if pagilaTable is defined, run a query on the Pagila dataset
if (
  [
    "actor",
    "address",
    "category",
    "city",
    "country",
    "customer",
    "film",
    "film_actor",
    "film_category",
    "inventory",
    "language",
    "payment",
    "payment_p2020_01",
    "payment_p2020_02",
    "payment_p2020_03",
    "payment_p2020_04",
    "payment_p2020_05",
    "payment_p2020_06",
    "rental",
    "staff",
    "store",
  ].includes(pagilaTable)
) {
  result = await client.queryObject(`SELECT * FROM ${pagilaTable};`);
} else {
  const param = 42;
  result = await client.queryObject(`SELECT ${param} as answer;`);
}

// Return result from database.
return new Response(JSON.stringify(result));
```

## Worker deployment

In `wrangler.toml`, enter your Cloudflare account ID in the line containing `account_id`:

{{<Aside type="note">}}

[Refer to Get started](/workers/get-started/guide#7-configure-your-project-for-deployment) if you need help finding your Cloudflare account ID.

{{</Aside>}}

```toml
---
filename: wrangler.toml
highlight: [3]
---
name = "worker-postgres-template"
type = "javascript"
account_id = ""
```

Publish your function:

```sh
wrangler publish
✨  Built successfully, built project size is 10 KiB.
✨  Successfully published your script to
 https://workers-postgres-template.example.workers.dev
```

### Set secrets

https://developers.cloudflare.com/cloudflare-one/identity/service-auth/service-tokens

Create and save [a Client ID and a Client Secret](https://developers.cloudflare.com/cloudflare-one/identity/service-auth/service-tokens) to Worker secrets in case your Tunnel is protected by Cloudflare Access.

```sh
wrangler secret put CF_CLIENT_ID
wrangler secret put CF_CLIENT_SECRET
```

### Test the Worker

Request some of the Pagila tables by adding the `?pagila-table` query parameter with a table name to the URL of the Worker.

```sh
curl https://example.workers.dev/?pagila-table=actor
curl https://example.workers.dev/?pagila-table=address
curl https://example.workers.dev/?pagila-table=country
curl https://example.workers.dev/?pagila-table=language
```

## Cleanup

Run the following command to stop and remove the Docker containers and networks:

```sh
docker compose down

# Stop and remove containers, networks
```

## Related resources

If you found this tutorial useful, continue building with other Cloudflare Workers tutorials below.


# Cloudflare Workers + PostgreSQL

https://developers.cloudflare.com/workers/tutorials/query-postgres-from-workers-using-database-connectors/

This repo contains example code and a PostgreSQL driver that can be used in any Workers project. If
you are interested in using the driver _outside_ of this template, copy the `driver/postgres` module
into your project's `node_modules` or directly alongside your source.

## Usage

Before you start, please refer to the **[official tutorial](https://developers.cloudflare.com/workers/tutorials/query-postgres-from-workers-using-database-connectors)**.

```typescript
const client = new Client({
    user: '<DATABASE_USER>',
    database: '<DATABASE_NAME>',
    // hostname is the full URL to your pre-created Cloudflare Tunnel, see documentation here:
    // https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/create-tunnel
    hostname: env.TUNNEL_HOST || 'https://dev.example.com',
    password: env.DATABASE_PASSWORD, // use a secret to store passwords
    port: '<DATABASE_PORT>',
})

await client.connect()
```

**Please Note:**
- you must use this config object format vs. a database connection string
- the `hostname` property must be the URL to your Cloudflare Tunnel, _NOT_ your database host
    - your Tunnel will be configured to connect to your database host

## Running the Postgres Demo

`postgres/docker-compose.yml`

This docker-compose composition will get you up and running with a local instance of `postgresql`, 
`pgbouncer` in front to provide connection pooling, and a copy of `cloudflared` to enable your 
applications to securely connect, through a encrypted tunnel, without opening any ports up locally.

### Usage

> from within `scripts/postgres`, run: 

1. **Create credentials file (first time only)**
```sh
docker run -v ~/.cloudflared:/etc/cloudflared cloudflare/cloudflared:2021.10.5 login
```

2. **Start a local dev stack (cloudflared/pgbouncer/postgres)**
```sh
TUNNEL_HOSTNAME=dev.example.com docker-compose up
```