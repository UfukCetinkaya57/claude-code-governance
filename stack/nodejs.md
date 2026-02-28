# NODE.JS / EXPRESS STACK REFERANSI

## Tespit
- `package.json` + `express` veya `fastify` dependency -> bu stack aktif

---

## Teknoloji Tablosu

| Katman | Arac |
|--------|------|
| Framework | Express.js (veya Fastify, NestJS) |
| ORM | Prisma |
| Validation | Zod |
| Auth | jose (onerilen) veya jsonwebtoken (legacy). jose: Web Crypto, Edge uyumlu, JWK/JWKS destegi |
| Password | argon2 (argon2id, onerilen) veya bcryptjs (pure JS, native binding sorunu yok) |
| Logging | Pino (onerilen, 2-5x hizli) veya Winston |
| Cache | ioredis |
| Test | Vitest (onerilen, TS native, 5-20x hizli) veya Jest + Supertest |
| API Docs | swagger-jsdoc + swagger-ui-express |
| Security | Helmet + cors |
| Rate Limit | express-rate-limit |

---

## Middleware Sirasi

```javascript
app.use(helmet());                    // 1. Security headers
app.use(cors(corsOptions));           // 2. CORS
app.use(express.json({ limit: '10mb' })); // 3. Body parser
app.use(requestLogger);              // 4. Request logging
app.use(rateLimiter);                // 5. Rate limiting
app.use('/api/v1', routes);          // 6. Routes
app.use(errorHandler);               // 7. Global error handler (EN SON)
```

Sira ONEMLIDIR. Error handler her zaman en sonda.

---

## Proje Yapisi

```
src/
├── config/          # Environment, DB, logger config
├── controllers/     # Route handler'lar (sadece HTTP concern)
├── services/        # Is mantigi
├── repositories/    # Veri erisimi (opsiyonel, Prisma direkt service'te de olabilir)
├── routes/          # Express route tanimlari
├── middlewares/     # Auth, error handler, rate limit, validation
├── validators/      # Zod schemalari
├── types/           # TypeScript interface/type tanimlari
├── utils/           # Helper fonksiyonlar (JWT, password, errors)
├── prisma/          # Prisma schema ve migration'lar
└── scripts/         # Seed, migration script'leri
```

---

## Prisma Kaliplari

### Schema
```prisma
model User {
  // NOT: Prisma uuid() UUID v4 uretir. UUID v7 icin uygulama katmaninda uret
  // veya DB fonksiyonu kullan: @default(dbgenerated("gen_random_uuid_v7()"))
  id        String   @id @default(uuid())
  email     String   @unique
  password  String
  role      Role     @default(USER)
  isActive  Boolean  @default(true) @map("is_active")
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")
  deletedAt DateTime? @map("deleted_at")

  @@map("users")
}
```

### Zorunlu Kurallar
- `@map()` ile camelCase -> snake_case donusumu
- `@@map()` ile tablo adi snake_case, cogul
- `@default(uuid())` PK icin (UUID v4 uretir — UUID v7 gerekiyorsa bkz. schema ornegi ustundeki not)
- `findMany` -> pagination ile (`skip`, `take`)
- `select` -> sadece gerekli alanlar
- `include` -> N+1 onleme (iliski yuklemesi)

### Migration
```bash
npx prisma migrate dev --name {isim_aciklayici}
npx prisma generate
```

---

## Zod Validation

```typescript
const createUserSchema = z.object({
  email: z.string().email('Gecerli bir email giriniz').max(255),
  password: z.string()
    .min(8, 'En az 8 karakter')
    .regex(/[A-Z]/, 'En az bir buyuk harf')
    .regex(/[a-z]/, 'En az bir kucuk harf')
    .regex(/[0-9]/, 'En az bir rakam')
    .regex(/[^a-zA-Z0-9]/, 'En az bir ozel karakter'),
});

// Middleware olarak kullanim
const validate = (schema) => (req, res, next) => {
  const result = schema.safeParse(req.body);
  if (!result.success) return res.status(422).json({ /* hata */ });
  req.validated = result.data;
  next();
};
```

---

## Logging

### Pino (Onerilen — 2-5x hizli, JSON native)
```typescript
import pino from 'pino';

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  transport: process.env.NODE_ENV !== 'production'
    ? { target: 'pino-pretty' }
    : undefined,
});

// Correlation ID middleware (pino-http ile)
import pinoHttp from 'pino-http';
app.use(pinoHttp({
  logger,
  genReqId: (req) => req.headers['x-correlation-id'] || uuidv4(),
}));
```

### Winston (Alternatif)
```typescript
const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  defaultMeta: { service: 'api' },
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
  ],
});
```

---

## Authentication (JWT)

```typescript
// Onerilen: jose (modern, standart uyumlu)
import * as jose from 'jose';

const privateKey = await jose.importPKCS8(privateKeyPem, 'RS256');
const publicKey = await jose.importSPKI(publicKeyPem, 'RS256');

// Token olusturma
const accessToken = await new jose.SignJWT({ email: user.email, role: user.role })
  .setProtectedHeader({ alg: 'RS256' })
  .setSubject(user.id)
  .setIssuedAt()
  .setIssuer('your-api')
  .setAudience('your-app')
  .setExpirationTime('1h')
  .sign(privateKey);

// Token dogrulama
const { payload } = await jose.jwtVerify(token, publicKey, {
  issuer: 'your-api',
  audience: 'your-app',
});
```

Password hashing (bkz. `guvenlik/CLAUDE.md` — argon2id onerilen):
```typescript
// Onerilen: argon2
import argon2 from 'argon2';
const hash = await argon2.hash(password, { type: argon2.argon2id });
const isValid = await argon2.verify(hash, password);

// Kabul edilir: bcryptjs (pure JS, native binding sorunu yok)
import bcrypt from 'bcryptjs';
const hash = await bcrypt.hash(password, 12);
const isValid = await bcrypt.compare(password, hash);
```

---

## Error Handling

```typescript
// Custom error class
class AppError extends Error {
  constructor(public statusCode: number, public code: string, message: string) {
    super(message);
  }
}

// Global error handler (EN SON middleware)
const errorHandler = (err, req, res, next) => {
  logger.error({ err, correlationId: req.correlationId });

  if (err instanceof AppError) {
    return res.status(err.statusCode).json({
      success: false,
      error: { code: err.code, message: err.message },
      requestId: req.correlationId,
    });
  }

  // Beklenmeyen hata -> detay gizle
  res.status(500).json({
    success: false,
    error: { code: 'INTERNAL_ERROR', message: 'Bir hata olustu' },
    requestId: req.correlationId,
  });
};
```

---

## Test

### Unit Test (Vitest — Onerilen)
```typescript
import { describe, it, expect, vi } from 'vitest';

describe('UserService', () => {
  it('should throw NotFoundError when user does not exist', async () => {
    const mockRepo = { findById: vi.fn().mockResolvedValue(null) };
    const service = new UserService(mockRepo);

    await expect(service.getById('fake-id')).rejects.toThrow(NotFoundError);
  });
});
```

### Integration Test (Supertest)
```typescript
import { describe, it, expect } from 'vitest';

describe('GET /api/v1/users', () => {
  it('should return 200 with user list', async () => {
    const res = await request(app).get('/api/v1/users');
    expect(res.status).toBe(200);
    expect(res.body.success).toBe(true);
  });
});
```

Not: Jest'ten Vitest'e gecis minimum — `jest.fn()` -> `vi.fn()`, config `vitest.config.ts` dosyasinda.

---

## TypeScript Kurallari

- `strict: true` -> tsconfig.json'da ZORUNLU
- `any` kullanimi YASAK (eslint rule ile engelle)
- Interface kullan (DTO'lar, service contractlar)
- Type kullan (union, utility types)
- `as` type assertion minimumda tut
- ESM tercih et (`"type": "module"` in package.json) — CommonJS legacy projelerde kabul edilir
- Import extension'lari: ESM'de `.js` extension zorunlu olabilir (tsconfig `moduleResolution` ayarina dikkat)

---

## Health Check

```typescript
// Basit health endpoint
app.get('/health/live', (req, res) => res.json({ status: 'ok' }));

app.get('/health/ready', async (req, res) => {
  try {
    await prisma.$queryRaw`SELECT 1`; // DB kontrol
    // await redis.ping();            // Redis kontrol (varsa)
    res.json({ status: 'ready' });
  } catch (err) {
    res.status(503).json({ status: 'not ready', error: err.message });
  }
});
```

---

## OpenTelemetry

```typescript
// tracing.ts — uygulama baslamadan ONCE import edilmeli
import { NodeSDK } from '@opentelemetry/sdk-node';
import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node';
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http';

const sdk = new NodeSDK({
  traceExporter: new OTLPTraceExporter({
    url: process.env.OTEL_EXPORTER_OTLP_ENDPOINT || 'http://localhost:4318/v1/traces',
  }),
  instrumentations: [getNodeAutoInstrumentations()],
  serviceName: process.env.OTEL_SERVICE_NAME || 'my-api',
});
sdk.start();
```

Paketler: `@opentelemetry/sdk-node`, `@opentelemetry/auto-instrumentations-node`, `@opentelemetry/exporter-trace-otlp-http`

---

## Docker

```dockerfile
FROM node:22-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npx prisma generate && npm run build

FROM node:22-alpine
WORKDIR /app
COPY --from=build /app/package*.json ./
RUN npm ci --only=production
COPY --from=build /app/dist ./dist
COPY --from=build /app/prisma ./prisma
ENV NODE_ENV=production
USER node
CMD ["node", "dist/index.js"]
```
