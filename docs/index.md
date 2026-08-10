---
layout: default
title: ParspackCERT - Let's Encrypt SSL Certificates with Parspack DNS API
description: Automatically issue and renew Let's Encrypt SSL certificates using Parspack DNS API and Certbot DNS-01 challenge.
---

# ParspackCERT – Let's Encrypt SSL Certificates with Parspack DNS API

🔒 Producing and renewing SSL certificates for domains that use Parspack CDN.

...

# ParspackCERT – Let's Encrypt SSL Certificates with Parspack DNS API
# ParspackCERT <a href="https://azolfagharj.github.io/donate/"><img src="https://img.shields.io/badge/Donate-Support%20Development-orange?style=for-the-badge" alt="Donate"></a>

🔒 Producing and renewing SSL certificates for domains that use [Parspack](https://parspack.com/) CDN. A simple, fast way to get them from Let's Encrypt using [Certbot](https://certbot.eff.org/) and the Parspack DNS API. One script, one command, no trouble.

Automatically obtain and renew Let's Encrypt SSL/TLS certificates using the Parspack DNS API and Certbot DNS-01 challenge.
ParspackCERT is a Bash script for automatically creating and removing_acme-challenge TXT records through the Parspack CDN API.

## Description

parspackCERT is a single bash script that makes SSL certificate management simple. It uses the Parspack DNS API to automatically add the required `_acme-challenge` TXT records for domain validation—no manual DNS work needed.d

These days, with heavy internet restrictions in Iran, you often need certificates stored directly on your server. When you need to bypass CDN or turn off proxy mode temporarily, you want your site to work without stopping. This script handles that simply: download, configure once, and you're done. It works for both manual runs and automatic renewal with Certbot.

## Requirements

📋 Before you start:

- **[Parspack account](https://my.parspack.com/main/auth/log-in)**: Your domain must be added to the Parspack dashboard
- **Parspack CDN API token**: You can get it from the Parspack dashboard, CDN section. The token must have these permissions: *List of Service*, *Store a DNS record*, *Delete a DNS record*
- **Certbot**: Must be installed on the system (`sudo apt install certbot`)
- **Python 3**: Required for JSON parsing (usually pre-installed)
- **curl**: Required for API requests (usually pre-installed)

## Get & Use

⚡ Simple three steps:

1. Download the script and make it executable (one command):

   ```bash
   wget -O parspackCERT https://raw.githubusercontent.com/azolfagharj/parspackCERT/main/parspackCERT && \
   chmod +x parspackCERT
   ```

2. Edit the script and set your Parspack CDN API token:
   ```bash
   API_TOKEN="your-parspack-cdn-api-token"
   ```

3. Run the command for your domain (e.g. yourdomain.ir):

   ```bash
   sudo ./parspackCERT -d "yourdomain.ir"
   ```

### Help

Show usage information:

```bash
./parspackCERT
./parspackCERT --help
./parspackCERT -h
```

## Certificate Storage

📁 Certificates are saved automatically in the default Certbot location:

```
/etc/letsencrypt/live/<cert_name>/fullchain.pem
/etc/letsencrypt/live/<cert_name>/privkey.pem
```

The certificate name is comes from the first domain you specify.

## How It Works

📖 Behind the scenes, it's simple:

1. **User runs script**: Execute `./parspackCERT -d "example.com"` with your domain(s)
2. **Script runs Certbot**: The script invokes Certbot with manual DNS challenge mode
3. **Auth hook**: For each domain, Certbot calls the script with `auth` argument
4. **DNS record creation**: The script uses Parspack API to create `_acme-challenge` TXT record
5. **Validation**: Let's Encrypt checks the TXT record
6. **Cleanup hook**: Certbot calls the script with `cleanup` argument
7. **DNS record removal**: The script deletes the temporary TXT record

## Advanced Usage

For advanced users and deeper control—wildcards, multiple domains, dry run, and more:

### Syntax

```bash
./parspackCERT -d DOMAIN [-d DOMAIN ...] [CERTBOT_OPTIONS]
```

### Domain Formats

The script accepts the same domain format as Certbot:

| Format            | Example                        | Description                          |
|-------------------|--------------------------------|--------------------------------------|
| Single domain     | `-d "example.com"`             | Main domain only                     |
| Wildcard          | `-d "*.example.com"`           | Covers all subdomains                |
| Multiple domains  | `-d "a.com" -d "b.com"`       | Add multiple domains with repeated -d|
| Comma-separated   | `-d "a.com,b.com,c.com"`      | Alternative to multiple -d flags      |

### Examples

**Main domain only:**
```bash
./parspackCERT -d "yourdomain.com"
```

**Wildcard and main domain (covers example.com and *.example.com):**
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


## Automatic Renewal

🔄 Set it once and forget it. Certbot schedules automatic renewal (usually twice daily). The script stores the auth and cleanup hooks in the renewal configuration, so Certbot automatically uses parspackCERT—no repeat setup needed.

To test renewal without making changes:

```bash
sudo certbot renew --dry-run
```

For more details, see the [Certbot documentation on automated renewals](https://eff-certbot.readthedocs.io/en/stable/using.html#automated-renewals).


## Error Messages

⚠️ The script provides clear error messages for common issues:

| Error                           | Cause                          | Solution                                  |
|---------------------------------|--------------------------------|-------------------------------------------|
| API_TOKEN is empty              | Token not set in script        | Edit script and add your Parspack CDN API token |
| Invalid or expired API token    | Wrong or expired token         | Generate new token from Parspack dashboard, CDN section |
| Zone not found for domain       | Domain not in Parspack         | Add domain to Parspack dashboard first     |
| Connection failed               | Network or API unreachable     | Check network, firewall, API URL           |
| certbot is not installed        | Certbot missing                | Run `sudo apt install certbot`            |
| python3 not found               | Python 3 missing               | Run `sudo apt install python3`            |

## Troubleshooting

🔧 Common fixes:

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

📄 MIT License. See [LICENSE](LICENSE) for details. Simple, fast, reliable.

---
## Support this Project



 🤝 **Enjoying this free project?** <a href="https://azolfagharj.github.io/donate/">Consider supporting</a> its development

<a href="https://azolfagharj.github.io/donate/"><img src="https://img.shields.io/badge/Donate-Support%20Development-orange?style=for-the-badge" alt="Donate"></a>

---
