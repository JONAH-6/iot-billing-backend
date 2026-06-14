# IoT Billing Backend

Enterprise-grade Web3/IoT billing backend integrating Soroban smart contracts for hardware telemetry metering and payment processing.

## Architecture

```
src/
├── config/          # Environment validation (zod), configuration
├── core/
│   ├── ingestion/   # Telemetry validation, parsing, locks, backpressure, state machine
│   ├── blockchain/  # Soroban RPC, nonce pool, circuit breaker, fee optimizer, tx manager
│   ├── crypto/      # Zero-knowledge range proof verification
│   ├── utils/       # SafeMath (7-decimal precision, overflow protection)
│   └── diagnostics/ # OpenTelemetry tracing
├── database/        # Elastic pool manager, TimescaleDB migrations, continuous aggregates
├── api/             # Fastify server, Web3 auth, rate limiter, mTLS gateway, Prometheus metrics
├── tests/
│   ├── unit/        # Signature validation, crypto, blockchain, state machine, config
│   ├── integration/ # Full-pipeline integration tests
│   └── load/        # 50k concurrent client simulation
└── metrics/         # Prometheus instrumentation
```

## Prerequisites

- **Node.js** >= 20
- **PostgreSQL** 16+ with **TimescaleDB** extension
- **Redis** (for nonce cache and rate limiter persistence)

## Quick Start

```bash
git clone https://github.com/IoT-Billing-Service/iot-billing-backend.git
cd iot-billing-backend
cp .env.example .env
npm install
npm run db:migrate
npm run db:seed
npm run dev
```

## Scripts

| Command | Description |
|---------|-------------|
| `npm run dev` | Start development server with hot reload |
| `npm run build` | Compile TypeScript to JavaScript |
| `npm start` | Run compiled production server |
| `npm test` | Run unit tests |
| `npm run test:integration` | Run integration tests (requires TimescaleDB) |
| `npm run test:load` | Execute load simulation |
| `npm run typecheck` | TypeScript type checking |
| `npm run lint` | ESLint check |
| `npm run format` | Prettier formatting check |
| `npm run db:migrate` | Run Prisma migrations |
| `npm run db:studio` | Open Prisma Studio |

## CI/CD

GitHub Actions runs on every push/PR to `main`:

1. **Lint & Format** — ESLint + Prettier
2. **Type Check** — `tsc --noEmit`
3. **Unit Tests** — Vitest (21 tests)
4. **Integration Tests** — TimescaleDB container + Prisma generate
5. **Build** — TypeScript compilation

## Key Technical Decisions

- **Fastify** over Express: 2-3x throughput, schema-based serialization, native async
- **Ed25519 signatures** for hardware payloads: stateless, high-performance, Stellar-native
- **TimescaleDB** hypertables: automatic partitioning, compression (85%+ ratio), continuous aggregates
- **PostgreSQL advisory locks**: distributed mutual exclusion without external dependencies
- **Circuit breaker pattern**: protects Soroban RPC from cascading failures
- **Two-phase commit state machine**: PENDING → TENTATIVE → SETTLED/ROLLED_BACK with reconciliation

## Environment Variables

See `.env.example` for all required configuration. Key variables:

- `DATABASE_URL` — PostgreSQL connection string
- `TIMESCALEDB_URL` — TimescaleDB connection string
- `SOROBAN_RPC_URL` — Stellar Soroban RPC endpoint
- `JWT_SECRET` — 32+ character signing secret
- `REDIS_URL` — Redis connection string
