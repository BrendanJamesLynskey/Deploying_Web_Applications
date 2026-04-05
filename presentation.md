# Deploying Web Applications

**From localhost to production -- the complete journey**

Topics: Hosting | DevOps | Infrastructure | DNS & HTTPS

Pipeline: Code -> Build -> Configure -> Deploy -> Monitor

Everything you need to know to get your web application off your laptop and in front of real users -- covering free hosting, cloud platforms, DNS, HTTPS, and production best practices.

---

## Table of Contents

1. [Topics](#slide-01--topics)
2. [What Is Deployment?](#slide-02--what-is-deployment)
3. [Static Site Hosting (Free Tier)](#slide-03--static-site-hosting-free-tier)
4. [Deploying to GitHub Pages](#slide-04--deploying-to-github-pages)
5. [Deploying to Vercel & Netlify](#slide-05--deploying-to-vercel--netlify)
6. [Platform-as-a-Service (PaaS)](#slide-06--platform-as-a-service-paas)
7. [Cloud Providers Overview](#slide-07--cloud-providers-overview)
8. [AWS Deployment Options](#slide-08--aws-deployment-options)
9. [VMs vs Containers vs Serverless](#slide-09--vms-vs-containers-vs-serverless)
10. [Domain Names & DNS](#slide-10--domain-names--dns)
11. [HTTPS & SSL/TLS](#slide-11--https--ssltls)
12. [Environment Variables & Secrets](#slide-12--environment-variables--secrets)
13. [Reverse Proxies & Load Balancers](#slide-13--reverse-proxies--load-balancers)
14. [Deployment Strategies](#slide-14--deployment-strategies)
15. [Monitoring & Logging](#slide-15--monitoring--logging)
16. [Cost Comparison](#slide-16--cost-comparison)
17. [Common Mistakes & Pitfalls](#slide-17--common-mistakes--pitfalls)
18. [Deployment Checklist](#slide-18--deployment-checklist)
19. [Putting It Together: Full Deploy Pipeline](#slide-19--putting-it-together-full-deploy-pipeline)
20. [Summary & Further Reading](#slide-20--summary--further-reading)

---

## Slide 01 -- Topics

### Foundations

- What deployment actually means
- Dev vs staging vs production
- Static site hosting (free tier)
- Deploying to GitHub Pages

### Platforms & Cloud

- Vercel & Netlify workflows
- Platform-as-a-Service (PaaS)
- Cloud providers (AWS, GCP, Azure)
- VMs vs containers vs serverless

### Networking & Security

- Domain names & DNS
- HTTPS & SSL/TLS certificates
- Environment variables & secrets
- Reverse proxies & load balancers

### Production Readiness

- Deployment strategies
- Monitoring & logging
- Cost comparison & scaling
- Common mistakes & checklist

---

## Slide 02 -- What Is Deployment?

Deployment is the process of making your application available to users on a server, not just on your machine.

```
Development  --git push-->  Staging  --promote-->  Production
localhost:3000              staging.app.com         app.com
Your machine only           Team testing            Real users
```

### Development

- Runs on your laptop
- Hot reload, debug tools
- Fake/test data
- No need for HTTPS

### Staging

- Mirrors production setup
- Used for QA & testing
- Safe to break things
- Often password-protected

### Production

- Real users, real data
- Must be fast & reliable
- HTTPS required
- Monitoring active

---

## Slide 03 -- Static Site Hosting (Free Tier)

For sites built with HTML/CSS/JS, React, Vue, Next.js (static export), etc. -- no server needed.

| Platform | Free Tier | Custom Domain | HTTPS | Build from Git | Best For |
|---|---|---|---|---|---|
| **GitHub Pages** | Unlimited (public repos) | Yes | Yes (auto) | Yes (Actions) | Docs, portfolios, blogs |
| **Netlify** | 100 GB/month bandwidth | Yes | Yes (auto) | Yes (built-in) | JAMstack, forms, functions |
| **Vercel** | 100 GB/month bandwidth | Yes | Yes (auto) | Yes (built-in) | Next.js, React, SSR |
| **Cloudflare Pages** | Unlimited bandwidth | Yes | Yes (auto) | Yes (built-in) | Speed, global CDN |

**Key takeaway:** All four platforms offer **free HTTPS**, **custom domains**, and **automatic deploys from Git**. For static sites, you should almost never pay for hosting.

---

## Slide 04 -- Deploying to GitHub Pages

### Option 1: Direct from branch

Push HTML to `main` or a `gh-pages` branch. Enable in repo Settings -> Pages.

```bash
# Create and push to gh-pages branch
git checkout -b gh-pages
git push origin gh-pages

# Your site is live at:
# https://username.github.io/repo-name/
```

### Option 2: GitHub Actions (recommended)

Build step runs in CI, deploys the output. Works for any framework.

**GitHub Actions workflow:**

```yaml
# .github/workflows/deploy.yml
name: Deploy to GitHub Pages
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      pages: write
      id-token: write
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-pages-artifact@v3
        with:
          path: ./dist
      - uses: actions/deploy-pages@v4
```

---

## Slide 05 -- Deploying to Vercel & Netlify

### Vercel (ideal for Next.js)

- Connect GitHub repo -> auto-deploys
- Preview deployments on every PR
- Serverless functions built in
- Edge middleware support

```bash
# CLI deploy (one-time setup)
npm i -g vercel
vercel          # deploys to preview
vercel --prod   # deploys to production
```

```json
// vercel.json (optional config)
{
  "buildCommand": "npm run build",
  "outputDirectory": "dist",
  "rewrites": [
    { "source": "/(.*)", "destination": "/" }
  ]
}
```

### Netlify

- Drag-and-drop deploy or Git-based
- Built-in form handling
- Netlify Functions (AWS Lambda)
- Split testing / branch deploys

```bash
# CLI deploy
npm i -g netlify-cli
netlify deploy        # preview
netlify deploy --prod # production
```

```toml
# netlify.toml
[build]
  command = "npm run build"
  publish = "dist"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

---

## Slide 06 -- Platform-as-a-Service (PaaS)

For apps that need a **backend server** (Node, Python, Ruby, Go, etc.). PaaS handles infrastructure so you focus on code.

| Platform | Free Tier | Docker Support | Databases | Starting Price | Notes |
|---|---|---|---|---|---|
| **Heroku** | None (removed 2022) | Yes | Postgres add-on | $5/mo (Eco dynos) | Pioneer of PaaS, mature ecosystem |
| **Railway** | $5 credit/month | Yes | Postgres, MySQL, Redis | Usage-based | Modern Heroku alternative |
| **Render** | Static sites free; services sleep | Yes | Postgres (free 90 days) | $7/mo | Auto-deploy from Git |
| **Fly.io** | 3 shared VMs free | Yes (primary) | Postgres, SQLite (LiteFS) | Usage-based | Edge deployment, low latency |

### Typical PaaS workflow

```bash
# Railway example
npm i -g @railway/cli
railway login
railway init
railway up   # deploys your app
```

### When to choose PaaS

- You have a backend (API, database)
- You want zero infrastructure management
- Small-to-medium scale applications
- Rapid prototyping and MVPs

---

## Slide 07 -- Cloud Providers Overview

The "big three" offer hundreds of services. Here are the ones relevant to web deployment.

### AWS (Amazon)

- **EC2** -- virtual machines
- **S3 + CloudFront** -- static hosting + CDN
- **Lambda** -- serverless functions
- **ECS / Fargate** -- containers
- **Elastic Beanstalk** -- managed PaaS
- **RDS** -- managed databases

**Free tier:** 12 months of t2.micro EC2, 5 GB S3, 1M Lambda requests/mo

### GCP (Google)

- **Compute Engine** -- VMs
- **Cloud Run** -- containers
- **Cloud Functions** -- serverless
- **Firebase Hosting** -- static + SSR
- **App Engine** -- managed PaaS
- **Cloud SQL** -- managed databases

**Free tier:** e2-micro VM always free, 2M Cloud Functions/mo

### Azure (Microsoft)

- **Virtual Machines**
- **App Service** -- managed PaaS
- **Azure Functions** -- serverless
- **Static Web Apps** -- free hosting
- **Container Apps**
- **Azure SQL**

**Free tier:** B1S VM (12 months), 1M Functions/mo

### Warning: Cloud billing

Cloud providers charge per use. **Always set billing alerts** and review costs weekly. A misconfigured service can run up a large bill quickly.

---

## Slide 08 -- AWS Deployment Options

```
Users --> CloudFront (CDN + HTTPS) --> S3 (static files)        [React, Vue, static sites]
                                   --> ECS / Fargate             [Docker containers]
                                   --> Lambda                    [API routes, serverless]

                                       EC2 (VMs)                [Full control]
                                       Elastic Beanstalk        [Managed PaaS]
```

### S3 + CloudFront (static sites)

```bash
# Upload static site to S3
aws s3 sync ./dist s3://my-bucket --delete

# Create CloudFront distribution pointing to bucket
# Gives you HTTPS + global CDN
aws cloudfront create-invalidation \
  --distribution-id EDIST123 \
  --paths "/*"
```

### Elastic Beanstalk (full apps)

```bash
# Initialize and deploy
eb init my-app --platform node.js
eb create production
eb deploy

# Manages EC2, load balancer, auto-scaling
# Like Heroku but on AWS
```

---

## Slide 09 -- VMs vs Containers vs Serverless

| Aspect | Virtual Machines | Containers | Serverless |
|---|---|---|---|
| **Abstraction** | Full OS, you manage everything | App + dependencies packaged | Just your function code |
| **Startup time** | Minutes | Seconds | Milliseconds (cold: seconds) |
| **Scaling** | Manual or auto-scaling groups | Orchestrator (K8s, ECS) | Automatic, instant |
| **Cost model** | Pay for uptime (always on) | Pay for uptime (can scale to 0) | Pay per invocation |
| **Control** | Full (SSH, install anything) | High (Dockerfile defines env) | Limited (runtime constraints) |
| **Examples** | EC2, Compute Engine, Droplets | ECS, Cloud Run, Fly.io | Lambda, Cloud Functions, Vercel |
| **Best for** | Legacy apps, custom runtimes | Microservices, complex apps | APIs, event handling, low traffic |

### Simple Dockerfile

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --production
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

### Recommendation

**Starting out?** Use serverless (Vercel, Netlify) or PaaS (Railway, Render). Move to containers when you need more control. VMs are for when you need full OS access or run specialized software.

---

## Slide 10 -- Domain Names & DNS

DNS (Domain Name System) translates human-readable domain names into IP addresses that computers use.

```
Browser (myapp.com) --> DNS Resolver (Lookup records) --> Nameserver (A: 93.184.216.34) --> Your Server (93.184.216.34)
```

### Key DNS Record Types

| Record | Points To | Example |
|---|---|---|
| **A** | IPv4 address | 93.184.216.34 |
| **AAAA** | IPv6 address | 2606:2800:220:1::248 |
| **CNAME** | Another domain | myapp.netlify.app |
| **TXT** | Text data | Verification, SPF |
| **MX** | Mail server | mail.example.com |

### Where to buy domains

- **Cloudflare Registrar** -- at-cost pricing
- **Namecheap** -- popular, affordable
- **Google Domains** -- now Squarespace
- **Porkbun** -- competitive pricing

Typical cost: **$10-15/year** for .com domains

### Pro tip: CNAME for hosted platforms

If deploying to Vercel/Netlify, use a **CNAME** record pointing to their domain (e.g., `cname.vercel-dns.com`). They handle the rest.

---

## Slide 11 -- HTTPS & SSL/TLS

HTTPS encrypts data between the browser and your server. It is **required** for modern web apps (browsers warn on HTTP).

### TLS Handshake (simplified)

```
Browser                              Server
   |--- ClientHello + supported ciphers --->|
   |<-- ServerHello + certificate ----------|
   |--- Key exchange ---------------------->|
   |<========= Encrypted data =============>|
```

### Why HTTPS matters

- Encrypts user data in transit
- Required for HTTP/2, service workers
- SEO ranking signal (Google)
- Browser shows "Not Secure" without it

### Let's Encrypt (free certificates)

Free, automated, open certificate authority. Certificates renew every 90 days.

```bash
# Using Certbot (most common)
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d myapp.com -d www.myapp.com

# Auto-renewal (runs twice daily)
sudo certbot renew --dry-run
```

### Managed HTTPS (zero config)

- **Vercel / Netlify / Cloudflare Pages** -- automatic
- **AWS CloudFront** -- ACM certificates (free)
- **Caddy** -- automatic HTTPS built in
- **Railway / Render** -- automatic for custom domains

---

## Slide 12 -- Environment Variables & Secrets

Configuration that changes between environments (API keys, database URLs, feature flags) should **never be hardcoded** in your source code.

### Never do this

```javascript
// WRONG: secrets in code
const dbUrl = "postgres://user:p4ssw0rd@db.com/prod";
const apiKey = "sk-abc123secret456";
```

### Do this instead

```javascript
// RIGHT: read from environment
const dbUrl = process.env.DATABASE_URL;
const apiKey = process.env.API_KEY;
```

### .env file (local development)

```bash
# .env (add to .gitignore!)
DATABASE_URL=postgres://localhost/myapp
API_KEY=dev-test-key-12345
NODE_ENV=development
```

### Setting env vars on platforms

```bash
# Vercel
vercel env add DATABASE_URL

# Netlify
netlify env:set DATABASE_URL "postgres://..."

# Railway
railway variables set DATABASE_URL="..."

# Render: Dashboard -> Environment tab

# AWS (Elastic Beanstalk)
eb setenv DATABASE_URL="postgres://..."

# Docker
docker run -e DATABASE_URL="..." myapp
```

### Secrets management (production)

- **AWS Secrets Manager** / Parameter Store
- **Google Secret Manager**
- **HashiCorp Vault** (self-hosted)
- **Doppler** / **Infisical** (SaaS)
- **GitHub Actions secrets** for CI/CD

---

## Slide 13 -- Reverse Proxies & Load Balancers

A reverse proxy sits between users and your app server(s), handling HTTPS termination, load balancing, caching, and more.

```
Users --HTTPS--> Reverse Proxy (Nginx / Caddy / ALB) --> App Server 1
                 (TLS termination + routing)          --> App Server 2
                                                      --> App Server 3
```

### Nginx config

```nginx
server {
    listen 443 ssl;
    server_name myapp.com;

    ssl_certificate /etc/letsencrypt/live/myapp.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/myapp.com/privkey.pem;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Caddy (auto HTTPS)

```text
# Caddyfile - that's it! Auto HTTPS.
myapp.com {
    reverse_proxy localhost:3000
}
```

Caddy automatically provisions and renews Let's Encrypt certificates.

### AWS Application Load Balancer

- Managed, no servers to run
- Routes to ECS tasks or EC2 instances
- ACM certificates (free HTTPS)
- Path-based & host-based routing

---

## Slide 14 -- Deployment Strategies

How you roll out new code to production without causing downtime or breaking things for users.

### Rolling Deployment

```
[v2] [v2] [v1] [v1]   -- Replace one at a time
```

- Gradually replace old instances
- Zero downtime
- Both versions run briefly
- **Default for K8s, ECS**

### Blue/Green

```
Blue (v1 - live)    Green (v2 - new)
        \               /
     Switch traffic instantly
```

- Two identical environments
- Instant switch via DNS/LB
- Easy rollback (switch back)
- **Costs 2x resources**

### Canary

```
[------95% traffic -> v1------] [v2]
                                 5%
        Gradually increase v2
```

- Route small % to new version
- Monitor errors & metrics
- Gradually increase if healthy
- **Safest for critical apps**

### For most beginners

Platforms like Vercel, Netlify, and Railway handle deployment strategy automatically. You get **atomic deploys** -- the new version replaces the old instantly with zero downtime. You only need to think about these strategies when running your own infrastructure.

---

## Slide 15 -- Monitoring & Logging

Deploying is not "done" when the code is live. You need to know when things break and why.

### What to monitor

| Metric | Why It Matters |
|---|---|
| **Response time** | Users leave if pages are slow |
| **Error rate** | 5xx errors mean your app is broken |
| **Uptime** | Is your site actually accessible? |
| **CPU / Memory** | Are you about to hit limits? |
| **Request count** | Traffic spikes, DDoS detection |

### Free monitoring tools

- **UptimeRobot** -- uptime pings (free 50 monitors)
- **Better Stack** -- uptime + status pages
- **Sentry** -- error tracking (free tier)
- **Vercel/Netlify analytics** -- built-in

### Structured logging

```javascript
// Use structured JSON logs, not console.log
const logger = require('pino')();

logger.info({ userId: 123, action: 'login' },
  'User logged in');

logger.error({ err, requestId: req.id },
  'Payment failed');
```

JSON logs are searchable. Plain text logs are not.

### Production logging stack

- **Datadog** -- full observability (paid)
- **Grafana + Loki** -- open source
- **AWS CloudWatch** -- built into AWS
- **Axiom** -- generous free tier

---

## Slide 16 -- Cost Comparison

What it actually costs to host a web application at different scales.

| Scenario | Platform | Monthly Cost | Includes |
|---|---|---|---|
| Static portfolio site | GitHub Pages / Cloudflare Pages | **$0** | Hosting, HTTPS, CDN |
| React SPA + API routes | Vercel free tier | **$0** | 100 GB BW, serverless functions |
| Node.js app + Postgres | Railway | **$5-15** | App server + managed database |
| Full-stack app, moderate traffic | Render / Fly.io | **$15-50** | Always-on server, database, SSL |
| Production app, team | AWS (ECS + RDS + CloudFront) | **$50-200** | Containers, managed DB, CDN |
| High-traffic SaaS | AWS / GCP (multi-service) | **$500+** | Auto-scaling, redundancy, monitoring |

### Start free, scale later

- Most hobby projects cost **$0/month**
- Upgrade only when you hit free tier limits
- Domain name is usually the first cost (~$12/year)
- Managed databases are the biggest cost driver

### Watch out for

- **AWS data transfer** charges (egress fees)
- Forgotten EC2 instances running 24/7
- Oversized database instances
- Not using reserved instances for predictable workloads
- Serverless at high volume can exceed VM costs

---

## Slide 17 -- Common Mistakes & Pitfalls

### 1. Committing secrets to Git

API keys, database passwords, .env files pushed to public repos. Use `.gitignore` and rotate any leaked credentials immediately.

```bash
# .gitignore
.env
.env.local
*.pem
node_modules/
```

### 2. No HTTPS

Running HTTP in production. All platforms offer free HTTPS. There is no excuse not to use it.

### 3. Hardcoding localhost URLs

Using `http://localhost:3000` in frontend code that runs in production. Use environment variables for API URLs.

### 4. No error handling

Unhandled exceptions crash your server. Add global error handlers and process managers (`pm2`, container restart policies).

### 5. Skipping build step

Deploying development source instead of production build. Always run `npm run build` and serve the optimized output.

### 6. Wrong Node/Python version

Production uses a different runtime version than development. Pin versions in `package.json` engines, `.nvmrc`, or Dockerfile.

```json
// package.json
{
  "engines": {
    "node": ">=20.0.0"
  }
}
```

### 7. No health check endpoint

Load balancers and orchestrators need a `/health` route to know your app is alive.

```javascript
app.get('/health', (req, res) => {
  res.status(200).json({ status: 'ok' });
});
```

---

## Slide 18 -- Deployment Checklist

Run through this before every production deploy.

### Before Deploy

- [ ] All tests passing
- [ ] Build succeeds with no warnings/errors
- [ ] Environment variables configured
- [ ] No secrets in source code or Git history
- [ ] `.gitignore` covers .env, node_modules, etc.
- [ ] Database migrations prepared
- [ ] Dependencies pinned (lockfile committed)
- [ ] Node/Python version specified

### Networking

- [ ] Custom domain configured with DNS
- [ ] HTTPS working (no mixed content)
- [ ] www -> non-www redirect (or vice versa)
- [ ] CORS configured for your domain
- [ ] API URLs use environment variables

### After Deploy

- [ ] Visit the live URL -- does it load?
- [ ] Test critical user flows (signup, login, payment)
- [ ] Check browser console for errors
- [ ] Verify HTTPS padlock in browser
- [ ] Check response times are acceptable
- [ ] Monitoring / alerting configured
- [ ] Error tracking active (Sentry, etc.)

### Performance

- [ ] Assets minified and compressed (gzip/brotli)
- [ ] Images optimized (WebP, lazy loading)
- [ ] CDN configured for static assets
- [ ] Cache headers set appropriately
- [ ] Lighthouse score acceptable (> 90)

---

## Slide 19 -- Putting It Together: Full Deploy Pipeline

A complete GitHub Actions workflow that builds, tests, containerizes, and deploys.

```yaml
# .github/workflows/deploy.yml
name: Build & Deploy
on:
  push:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci
      - run: npm test
      - run: npm run build

  deploy:
    needs: test
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4
      - uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@v5
        with:
          push: true
          tags: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
      # Deploy to your platform of choice:
      # - fly deploy
      # - railway up
      # - aws ecs update-service ...
```

---

## Slide 20 -- Summary & Further Reading

### Key Takeaways

- **Static sites** -> Vercel, Netlify, GitHub Pages, Cloudflare Pages (all free)
- **Backend apps** -> Railway, Render, Fly.io (cheap PaaS)
- **Complex/large apps** -> AWS, GCP, Azure with containers
- **Always use HTTPS** -- free everywhere
- **Never commit secrets** -- use env vars
- Start simple, add complexity only when needed

### Recommended learning path

1. Deploy a static site to GitHub Pages
2. Deploy a Next.js app to Vercel
3. Deploy a Node.js + Postgres app to Railway
4. Add a custom domain + HTTPS
5. Set up CI/CD with GitHub Actions
6. Try Docker + Fly.io or Cloud Run

### Further reading

- **The Twelve-Factor App** -- 12factor.net
- **Docker Getting Started** -- docs.docker.com
- **Vercel Docs** -- vercel.com/docs
- **Netlify Docs** -- docs.netlify.com
- **AWS Well-Architected** -- Framework guide
- **Caddy Documentation** -- caddyserver.com/docs
- **Let's Encrypt** -- letsencrypt.org

### Quick reference

```
Write Code -> Push to Git -> CI Builds & Tests -> Deploy
```

The best deployment is the one you **don't have to think about**. Automate everything.
