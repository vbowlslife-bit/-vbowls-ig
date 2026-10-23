# Instagram AutoDM Dashboard

A personal, single-user Instagram automation dashboard for auto-replying to comments and DMs based on configurable rules.

## Features

- **Dashboard**: Real-time stats (DMs sent, comments handled, active automations, failed sends, leads captured), 14-day activity charts, top automations, recent activity feed
- **Automations**: Create, edit, duplicate, delete automations with 3-step wizard (Trigger → Message → Review)
- **Trigger Types**: Comment on specific post, comment on any post, story reply, incoming DM keyword
- **Message Features**: Variable substitution (`{{username}}`, `{{first_name}}`), link buttons, public reply variants, delay, send-once-per-user, ask-for-follow nudge, email capture
- **Inbox**: Conversation list + chat view, manual replies with 24h window notice, automation attribution
- **Leads**: Table with search, date filters, CSV export, delete
- **Logs**: Full webhook event log with filters, expandable JSON payloads
- **Settings**: Instagram connection (OAuth + manual token), webhook configuration, global pause, test message, mock mode indicator
- **Mock Mode**: Full demo with 6 posts, 4 automations, 50 contacts, 200+ messages, 20 leads, 200 event logs
- **Simulate Panel**: Test comment, DM, and story reply triggers in mock mode

## Tech Stack

- Next.js 15 (App Router) + TypeScript
- Tailwind CSS + shadcn/ui + lucide-react
- Prisma + SQLite
- Recharts for analytics
- In-process job queue with retry/backoff
- Iron-session for authentication
- Sonner for toast notifications

## Getting Started

### Prerequisites

- Node.js 18+
- npm

### Installation

```bash
cd insta-dashboard
npm run setup
```

This will:
1. Run Prisma migrations
2. Seed the database with demo data

### Development

```bash
npm run dev
```

Open http://localhost:3000

### Login

Default password: `admin123` (set via `ADMIN_PASSWORD` in `.env`)

## Project Structure

```
src/
├── app/                    # Next.js App Router pages
│   ├── api/               # API routes
│   │   ├── auth/          # Authentication endpoints
│   │   ├── automations/   # Automation CRUD
│   │   ├── contacts/      # Contact management
│   │   ├── dashboard/     # Dashboard stats
│   │   ├── leads/         # Lead management
│   │   ├── logs/          # Event logs
│   │   ├── messages/      # Message sending
│   │   ├── settings/      # Settings & test message
│   │   ├── simulate/      # Mock simulation
│   │   └── webhooks/      # Instagram webhook handler
│   ├── automations/       # Automations pages
│   ├── dashboard/         # Dashboard page
│   ├── inbox/             # Inbox page
│   ├── leads/             # Leads page
│   ├── logs/              # Logs page
│   ├── login/             # Login page
│   └── settings/          # Settings page
├── components/
│   ├── automation/        # Automation components (PhonePreview, SimulatePanel)
│   ├── layout/            # Layout components (Sidebar, Header, DashboardLayout)
│   └── ui/                # shadcn/ui components
├── lib/
│   ├── auth.ts            # Session management
│   ├── automation/        # Automation engine & queue
│   ├── instagram/         # Instagram API (mock + real)
│   ├── prisma.ts          # Prisma client
│   └── utils.ts           # Utility functions
└── middleware.ts          # Auth middleware

prisma/
├── schema.prisma          # Database schema
├── seed.ts                # Main seed file
├── seed-*.ts              # Modular seed files
└── migrations/            # Database migrations
```

## Environment Variables

See `.env.example` for all options:

| Variable | Description | Default |
|----------|-------------|---------|
| `DATABASE_URL` | SQLite database path | `file:./dev.db` |
| `ADMIN_PASSWORD` | Login password (bcrypt hashed) | `admin123` |
| `SESSION_PASSWORD` | Session encryption key (32+ chars) | Required |
| `MOCK_INSTAGRAM` | Enable mock mode | `true` |
| `META_APP_ID` | Meta App ID (real mode) | - |
| `META_APP_SECRET` | Meta App Secret (real mode) | - |
| `INSTAGRAM_REDIRECT_URI` | OAuth redirect URI | `http://localhost:3000/api/auth/instagram/callback` |
| `WEBHOOK_VERIFY_TOKEN` | Webhook verification token | Required |
| `WEBHOOK_URL` | Public webhook URL (ngrok) | Required |
| `NEXT_PUBLIC_APP_URL` | App URL | `http://localhost:3000` |

## Using the Simulate Panel

In mock mode (`MOCK_INSTAGRAM=true`), the Dashboard includes a Simulate Panel to test automations:

1. **Simulate Comment**: Triggers `COMMENT_ON_POST` or `COMMENT_ANY_POST` automations
2. **Simulate DM**: Triggers `DM_KEYWORD` automations
3. **Simulate Story Reply**: Triggers `STORY_REPLY` automations

Results appear instantly in Inbox and Logs.

## Real Instagram Setup

See `SETUP.md` for detailed instructions on:
- Creating a Meta Developer App (Business type)
- Adding Instagram product
- Configuring permissions (`instagram_business_basic`, `instagram_business_manage_messages`, `instagram_business_manage_comments`)
- Adding yourself as Instagram Tester
- Generating access tokens
- Setting up ngrok/cloudflared for webhooks
- Subscribing to `comments` and `messages` webhook fields
- Testing with a second Instagram account

## License

MIT