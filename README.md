# parspackCERT – Automated SSL Certificates for [Parspack](https://parspack.com/) CDN

Obtain SSL certificates from Let's Encrypt using [Certbot](https://certbot.eff.org/) and the Parspack DNS API. This script automates the DNS-01 challenge by creating and removing TXT records through the Parspack CDN API when Certbot requests them.

## Description

parspackCERT is a single bash script that integrates with Certbot to obtain and renew SSL certificates. It uses the Parspack DNS API to automatically add the required `_acme-challenge` TXT records for domain validation. The script handles both manual execution and automatic renewal when Certbot runs its scheduled tasks.

## Requirements

- **[Parspack account](https://my.parspack.com/main/auth/log-in)**: Your domain must be added to the Parspack dashboard
- **Parspack CDN API token**: You can get it from the Parspack dashboard, CDN section. The token must have these permissions: *List of Service*, *Store a DNS record*, *Delete a DNS record*
- **Certbot**: Must be installed on the system (`sudo apt install certbot`)
- **Python 3**: Required for JSON parsing (usually pre-installed)
- **curl**: Required for API requests (usually pre-installed)

## Installation

1. Place the `parspackCERT` script in your project directory
2. Make it executable: `chmod +x parspackCERT`
3. Edit the script and set your Parspack CDN API token:
   ```bash
   API_TOKEN="your-parspack-cdn-api-token"
   ```

## Configuration

Open the script and configure the following variable:

| Variable   | Description                                      |
|-----------|--------------------------------------------------|
| API_TOKEN | Parspack CDN API token (get it from Parspack dashboard, CDN section). Required permissions: List of Service, Store a DNS record, Delete a DNS record |
| API_BASE  | API base URL (default: Parspack production)      |

## Usage

### Basic Syntax

```bash
./parspackCERT -d DOMAIN [-d DOMAIN ...] [CERTBOT_OPTIONS]
```

### Domain Formats

The script accepts the same domain format as Certbot:

| Format            | Example                        | Description                          |
|-------------------|--------------------------------|--------------------------------------|
| Single domain     | `-d "example.com"`             | Apex domain only                     |
| Wildcard          | `-d "*.example.com"`           | Covers all subdomains                |
| Multiple domains  | `-d "a.com" -d "b.com"`       | Add multiple domains with repeated -d|
| Comma-separated   | `-d "a.com,b.com,c.com"`      | Alternative to multiple -d flags      |

### Examples

**Apex domain only:**
```bash
./parspackCERT -d "yourdomain.com"
```

**Wildcard and apex (covers example.com and *.example.com):**
```bash
./parspackCERT -d "*.example.com" -d "example.com"
```

**Multiple domains:**
```bash
./parspackCERT -d "www.example.com" -d "api.example.com" -d "example.com"
```

**Dry run (test without obtaining certificate):**
```bash
./parspackCERT -d "example.com" --dry-run
```

**Force renewal (renew even if not expired):**
```bash
./parspackCERT -d "example.com" --force-renewal
```

### Help

Show usage information:

```bash
./parspackCERT
./parspackCERT --help
./parspackCERT -h
```

## Certificate Storage

Certificates are saved in the default Certbot location:

```
/etc/letsencrypt/live/<cert_name>/fullchain.pem
/etc/letsencrypt/live/<cert_name>/privkey.pem
```

The certificate name is derived from the first domain you specify.

## Automatic Renewal

Certbot schedules automatic renewal (typically twice daily). The script stores the auth and cleanup hooks in the renewal configuration, so Certbot will automatically use parspackCERT when renewing certificates.

To test renewal without making changes:

```bash
sudo certbot renew --dry-run
```

For more details, see the [Certbot documentation on automated renewals](https://eff-certbot.readthedocs.io/en/stable/using.html#automated-renewals).

## How It Works

1. **User runs script**: You execute `./parspackCERT -d "example.com"` with your domain(s)
2. **Script runs Certbot**: The script invokes Certbot with manual DNS challenge mode
3. **Auth hook**: For each domain, Certbot calls the script with `auth` argument
4. **DNS record creation**: The script uses Parspack API to create `_acme-challenge` TXT record
5. **Validation**: Let's Encrypt checks the TXT record
6. **Cleanup hook**: Certbot calls the script with `cleanup` argument
7. **DNS record removal**: The script deletes the temporary TXT record

## Error Messages

The script provides clear error messages for common issues:

| Error                           | Cause                          | Solution                                  |
|---------------------------------|--------------------------------|-------------------------------------------|
| API_TOKEN is empty              | Token not set in script        | Edit script and add your Parspack CDN API token |
| Invalid or expired API token    | Wrong or expired token         | Generate new token from Parspack dashboard, CDN section |
| Zone not found for domain       | Domain not in Parspack         | Add domain to Parspack dashboard first     |
| Connection failed               | Network or API unreachable     | Check network, firewall, API URL           |
| certbot is not installed        | Certbot missing                | Run `sudo apt install certbot`            |
| python3 not found               | Python 3 missing               | Run `sudo apt install python3`            |

## Troubleshooting

**Certificate not renewing automatically:**
- Check Certbot timer: `sudo systemctl status certbot.timer`
- Verify renewal config: `cat /etc/letsencrypt/renewal/yourdomain.com.conf`
- Ensure `manual_auth_hook` and `manual_cleanup_hook` point to parspackCERT

**DNS propagation delay:**
- The script waits 25 seconds after creating the TXT record
- For slow DNS, you may need to increase the sleep time in the script

**Permission denied:**
- Run with sudo when obtaining certificates: `sudo ./parspackCERT -d "example.com"`

## License

This script is provided as-is for use with Parspack CDN and Let's Encrypt certificates.
