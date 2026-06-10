# High-Level Architecture

                                    ┌─────────────────┐
                                    │    End Users    │
                                    │ Web / Mobile App│
                                    └────────┬────────┘
                                             │
                                             ▼
                              ┌──────────────────────────┐
                              │       Cloudflare         │
                              │ DNS + SSL + CDN + WAF   │
                              └──────────┬───────────────┘
                                         │
                  ┌──────────────────────┼──────────────────────┐
                  │                      │                      │
                  ▼                      ▼                      ▼
      ┌──────────────────┐   ┌──────────────────┐   ┌──────────────────┐
      │   Frontend App   │   │    Backend API   │   │   Static Assets  │
      │      Vercel      │   │ Contabo VPS      │   │ Cloudflare CDN   │
      │ Next.js/React    │   │ NestJS + TS      │   │ (Cached Content) │
      └────────┬─────────┘   └────────┬─────────┘   └────────┬─────────┘
               │                      │                      │
               │                      │                      ▼
               │                      │          ┌─────────────────────┐
               │                      │          │  Object Storage     │
               │                      │          │   Backblaze B2      │
               │                      │          └─────────────────────┘
               │                      │
               │                      │
               │                      ▼
               │          ┌─────────────────────────────┐
               │          │      Internal Services      │
               │          └──────────┬───────┬──────────┘
               │                     │       │
               ▼                     ▼       ▼
    ┌────────────────┐    ┌─────────────┐ ┌─────────────┐
    │ PostgreSQL     │    │ Redis       │ │ MongoDB     │
    │ Main Database  │    │ Cache/Queue │ │ Analytics / │
    │ Transactional  │    │ Session     │ │ Logs / Docs │
    └────────────────┘    └─────────────┘ └─────────────┘

                               ┌─────────────────┐
                               │     Resend      │
                               │ Email Service   │
                               └────────▲────────┘
                                        │
                                        │
                               ┌────────┴────────┐
                               │     NestJS      │
                               │ Notification    │
                               └─────────────────┘

                               ┌─────────────────┐
                               │    Midtrans     │
                               │ Payment Gateway │
                               └────────▲────────┘
                                        │
                                        │
                               ┌────────┴────────┐
                               │     NestJS      │
                               │ Payment Module  │
                               └─────────────────┘


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
