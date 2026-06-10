# High-Level Architecture

                                          ┌─────────────────┐
                                          │    End Users    │
                                          │ Web / Mobile    │
                                          └────────┬────────┘
                                                   │
                                                   ▼
                                   ┌──────────────────────────┐
                                   │       Cloudflare         │
                                   │ DNS + SSL + CDN + WAF   │
                                   └──────────┬───────────────┘
                                              │
               ┌──────────────────────────────┼──────────────────────────────┐
               │                              │                              │
               ▼                              ▼                              ▼
    ┌──────────────────┐          ┌──────────────────┐          ┌──────────────────┐
    │ app.example.com  │          │ api.example.com  │          │ cdn.example.com  │
    │      Vercel      │          │  Contabo VPS     │          │ Cloudflare CDN   │
    └──────────────────┘          └────────┬─────────┘          └────────┬─────────┘
                                           │                             │
                                           ▼                             ▼
                                 ┌──────────────────┐          ┌──────────────────┐
                                 │      Nginx       │          │   Backblaze B2   │
                                 │ Reverse Proxy    │◄────────►│ Object Storage   │
                                 └────────┬─────────┘          └──────────────────┘
                                          │
                                          ▼
                                 ┌──────────────────┐
                                 │     NestJS       │
                                 │  API Services    │
                                 └────────┬─────────┘
                                          │
                    ┌─────────────────────┼─────────────────────┐
                    │                     │                     │
                    ▼                     ▼                     ▼
          ┌────────────────┐   ┌────────────────┐   ┌────────────────┐
          │ PostgreSQL     │   │ Redis          │   │ MongoDB        │
          │ Main Database  │   │ Cache / Queue  │   │ Logs / Events  │
          └────────────────┘   └────────────────┘   └────────────────┘

                                          │
                    ┌──────────────────────────────────────────┐
                    │                     │                    │
                    ▼                     ▼                    ▼
          ┌────────────────┐     ┌────────────────┐   ┌────────────────┐
          │    Resend      │     │    Midtrans    │   │  Backblaze B2  │
          │ Email Service  │     │ Payment Gateway│   │ Object Storage │
          └────────────────┘     └────────────────┘   └────────────────┘


# CI/CD Architecture

                            ┌───────────────┐
                            │   Developer   │
                            └───────┬───────┘
                                    │ git push
                                    ▼
                            ┌───────────────────┐
                            │      GitHub       │
                            │   Source Control  │
                            └─────────┬─────────┘
                                      │
                                      ▼
                            ┌───────────────────┐
                            │ GitHub Actions    │
                            │ Build & Test      │
                            └─────────┬─────────┘
                                      │
                                      ├─────────────► Deploy Frontend
                                      │
                                      │              ┌─────────────┐
                                      │              │   Vercel    │
                                      │              └─────────────┘
                                      │
                                      ▼
                            ┌───────────────────┐
                            │ Deploy Backend    │
                            │ SSH / Docker      │
                            └─────────┬─────────┘
                                      │
                                      ▼
                            ┌───────────────────┐
                            │   Contabo VPS     │
                            │ Docker + NestJS   │
                            └───────────────────┘


# CI/CD Flow
                            Developer
                                │
                                ▼
                            GitHub
                                │
                                ▼
                            GitHub Actions
                                │
                                ├── Build Frontend
                                │        │
                                │        ▼
                                │      Vercel
                                │
                                └── Build Backend
                                         │
                                         ▼
                                     SSH Deploy
                                         │
                                         ▼
                                    Contabo VPS
                                         │
                                         ▼
                                  Docker Compose
                                         │
                                         ▼
                                      Nginx
                                         │
                                         ▼
                                      NestJS

# Recommended Production Setup

                              Hostinger
                                  │
                                  ▼
                              Cloudflare
                                  │
                                  ├── app.yourdomain.com ─────► Vercel
                                  │
                                  ├── api.yourdomain.com ─────► Contabo VPS
                                  │
                                  └── cdn.yourdomain.com ─────► Cloudflare CDN
                                                                     │
                                                                     ▼
                                                               Backblaze B2

# Backend Server Layout (Contabo VPS)

                            Contabo VPS
                            │
                            ├── Nginx
                            │   ├── app reverse proxy
                            │   ├── api reverse proxy
                            │   ├── SSL handling
                            │   └── rate limiting
                            │
                            ├── Docker
                            │   │
                            │   ├── NestJS Container
                            │   ├── PostgreSQL Container
                            │   ├── Redis Container
                            │   └── MongoDB Container
                            │
                            └── Logs

# Request Flow
## API Request

                            User
                              │
                              ▼
                            Cloudflare
                              │
                              ▼
                            api.example.com
                              │
                              ▼
                            Nginx
                              │
                              ▼
                            NestJS
                              │
                              ├── PostgreSQL
                              ├── Redis
                              └── MongoDB

## File Upload
                          User
                            │
                            ▼
                          Frontend (Vercel)
                            │
                            ▼
                          NestJS API
                            │
                            ▼
                          Backblaze B2

Store only: ``avatars/user-123.jpg`` inside PostgreSQL.

## File Download
                      User
                        │
                        ▼
                      cdn.example.com
                        │
                        ▼
                      Cloudflare CDN
                        │
                        ▼ (Cache MISS)
                      Backblaze B2

Subsequent requests:
                      
                      User
                        │
                        ▼
                      Cloudflare CDN
                        │
                        ▼ (Cache HIT)
                      Served directly


# Suggested Domain Structure
                      
                      urbanest.id
                      │
                      ├── app.urbanest.id
                      │      └── Vercel
                      │
                      ├── api.urbanest.id
                      │      └── Cloudflare → Nginx → NestJS
                      │
                      ├── cdn.urbanest.id
                      │      └── Cloudflare CDN → Backblaze B2
                      │
                      ├── vendor.urbanest.id
                      │      └── Vercel
                      │
                      └── admin.urbanest.id 
                             └── Vercel                      

# Technology Responsibilities

| Component      | Responsibility                          |
| -------------- | --------------------------------------- |
| NestJS         | Business logic, APIs, authentication    |
| PostgreSQL     | Transactions, users, payments, vouchers |
| Redis          | Cache, sessions, rate limiting, queues  |
| MongoDB        | Logs, analytics, flexible documents     |
| Cloudflare DNS | Domain management                       |
| Cloudflare CDN | Static file caching                     |
| Backblaze B2   | Image/document storage                  |
| Resend         | OTP, password reset, notifications      |
| Midtrans       | Payment processing                      |
| GitHub Actions | CI/CD pipeline                          |
| Vercel         | Frontend hosting                        |
| Contabo VPS    | Backend hosting                         |
| Hostinger      | Domain registrar                        |
| Nginx          | Web Server/Proxy Management             |


# The Final Production Flow
                                
                                Hostinger Domain
                                        │
                                        ▼
                                Cloudflare
                                        │
                                        ├── Frontend → Vercel
                                        │
                                        ├── API → Nginx → NestJS
                                        │                    │
                                        │                    ├── PostgreSQL
                                        │                    ├── Redis
                                        │                    └── MongoDB
                                        │
                                        └── CDN → Backblaze B2
                                                    ▲
                                                    │
                                                NestJS Uploads
                                
                                External Services:
                                - Resend (Email)
                                - Midtrans (Payments)
                                
                                CI/CD:
                                GitHub → GitHub Actions → Contabo VPS / Vercel
