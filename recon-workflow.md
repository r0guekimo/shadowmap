# Recon & Bug Bounty Methodology

Personal workflow for web app recon, enumeration, and vulnerability discovery during authorized bug bounty engagements. Run only against in-scope targets under a valid program/VDP.

---

## 1. Subdomain Enumeration

Run in parallel, merge output.

```bash
subfinder -d target.com -silent -o subdomains.txt
amass enum -passive -d target.com >> subdomains.txt
assetfinder --subs-only target.com >> subdomains.txt

# Merge and deduplicate
cat subdomains.txt | sort -u > all_subs.txt
wc -l all_subs.txt  # sanity check
```

Certificate transparency — often surfaces internal/staging/forgotten assets:

```bash
curl -s "https://crt.sh/?q=%.target.com&output=json" | \
  jq -r '.[].name_value' | sed 's/\*\.//g' | sort -u >> all_subs.txt
```

---

## 2. Active Probing

Resolve live hosts only — don't waste time testing dead IPs.

```bash
cat all_subs.txt | httpx -silent -status-code -title -tech-detect -o live_hosts.txt

# Extract just URLs for further processing
cat live_hosts.txt | awk '{print $1}' > live_urls.txt
```

Targeted port scan on live hosts (not a full range scan on everything):

```bash
cat live_urls.txt | cut -d/ -f3 | \
  naabu -p 80,443,8080,8443,8888,3000,4000,5000,9000,9090 -silent -o ports.txt
```

Deeper port/service scan when a specific host warrants it:

```bash
nmap -p- --open -sV -sC -T4 -oN nmap_result.txt target.com
```

---

## 3. Historical & Content Discovery

Historical URLs — gold for finding forgotten/deprecated endpoints:

```bash
waybackurls target.com | sort -u > wayback.txt
gau target.com -o gau.txt
cat wayback.txt gau.txt | sort -u > historical_urls.txt

# Pull out JS files specifically
cat historical_urls.txt | grep -iE '\.js$' | grep -ivE '\.json$' | sort -u | tee js.txt
```

Screenshot capture across subdomains, for quick visual triage:

```bash
cat subdomains.txt | aquatone -out screenshots
```

---

## 4. Directory & Endpoint Fuzzing

Fuzz directories on interesting hosts only — not blindly across everything:

```bash
ffuf -u https://target.com/FUZZ \
  -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt \
  -mc 200,201,301,302,403 \
  -t 50 \
  -o ffuf_root.json
```

API endpoint discovery — use an API-specific wordlist:

```bash
ffuf -u https://api.target.com/FUZZ \
  -w /usr/share/seclists/Discovery/Web-Content/api/objects.txt \
  -mc 200,201,401,403 \
  -t 40
```

JS file mining — this is where a lot of real findings live:

```bash
cat live_urls.txt | getJS --complete > js_files.txt

cat js_files.txt | while read url; do
  curl -s "$url" | grep -oP '(\/api\/[a-zA-Z0-9_\-\/]+)|(\/v[0-9]+\/[a-zA-Z0-9_\-\/]+)'
done | sort -u > js_endpoints.txt
```

Quick low-hanging fruit + misconfig detection:

```bash
nuclei -l live_urls.txt \
  -t exposures/ -t misconfiguration/ -t technologies/ \
  -severity medium,high,critical \
  -o nuclei_results.txt
```

---

## 5. JavaScript Analysis

```bash
python3 ~/Tools/LinkFinder/linkfinder.py -i https://target.com/script.js -o result.html
cat js_files.txt | gf apikeys > potential_keys.txt
```

Manually review anything `gf apikeys` flags — false positive rate is high, but worth checking each one.

---

## 6. Parameter Discovery

```bash
paramspider -d target.com --level high -o params.txt
arjun -u https://target.com/api -m GET -o params.json
```

---

## 7. Vulnerability Testing

> Run these only against explicitly in-scope targets, per the program's testing rules.

**XSS:**
```bash
cat params.txt | dalfox pipe -o xss_result.txt
```

**SQL Injection:**
```bash
sqlmap -u "https://target.com/index.php?id=1" --dbs --batch --random-agent
```

**SSRF:**
```bash
interactsh-client
```

**LFI/RFI:**
```bash
fimap -u "https://target.com/index.php?file=test"
```

**Open Redirect:**
```bash
python3 ~/Tools/Oralyzer/oralyzer.py -l urls.txt -p payload.txt
```

---

## 8. Misconfiguration & Recon Add-ons

**Security headers / server fingerprinting:**
```bash
nikto -h target.com
httpx -u target.com -sc -title -server -o headers.txt
```

**WAF detection:**
```bash
wafw00f https://target.com
```

**CMS enumeration:**
```bash
python3 ~/Tools/CMSeeK/cmseek.py -u target.com
```

**S3 bucket enumeration:**
```bash
python3 AWSBucketDump.py -l target-buckets.txt -D
```

**Exposed `.git` directories:**
```bash
git-dumper https://target.com/.git ./output-folder/
```

**API recon with predefined route lists:**
```bash
kiterunner -u https://target.com -w wordlist/apis.txt
```

---

## Notes

- Always confirm scope and authorization before running anything beyond passive recon.
- Rate-limit aggressive scans (`ffuf`, `naabu`, `nuclei`) on smaller/sensitive targets.
- Keep raw output files (`*.txt`, `*.json`) out of version control — see `.gitignore`.
