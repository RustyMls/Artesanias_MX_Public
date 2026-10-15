# Artesanías Mexicanas

Aplicación web para un negocio de artesanías textiles mexicanas: catálogo y tienda en línea para el público, y un sistema de punto de venta e inventario para el personal, con un único stock compartido entre bodega, local físico y ventas por mayor.

## Stack

- **Frontend** — `frontend-web/`: Next.js 16 (App Router, TypeScript, Tailwind CSS 4)
- **Backend** — `backend-inventario/`: Node.js, Express 5, TypeScript, Prisma 6
- **Base de datos**: PostgreSQL 16
- **Correo**: Resend (invitaciones y recuperación de contraseña)
- **Producción**: Caddy como reverse proxy con HTTPS automático, servicios systemd, Ubuntu 24.04 LTS

## Funcionalidades

- **Tienda pública**: catálogo con variantes (modelo con diseños/colores), vista rápida, carrito y apartados por WhatsApp.
- **Punto de venta (POS)**: cobro al menudeo y mayoreo, recibos imprimibles (térmica de 80 mm), reimpresión del día, apartados con vencimiento automático.
- **Inventario**: alta de productos (sueltos, grupos y variantes), traspasos entre bodega y local, ajustes por conteo, fotos optimizadas a WebP.
- **Compras y bultos**: registro de compras a proveedores, apertura/reventa de bultos cerrados, costo promedio ponderado por pieza.
- **Balances**: ingresos, utilidad bruta y margen, histórico de ventas, top de productos por periodo.
- **Usuarios**: invitación por correo, login con correo y contraseña, recuperación de contraseña, roles `VENDEDOR` / `ADMINISTRADOR`.
- **Concurrencia**: bloqueo optimista con columnas `version` en stock, productos y ventas — evita vender la misma pieza única dos veces a la vez (local y en línea).

## Estructura del repo

```
backend-inventario/   API Express + Prisma (PostgreSQL)
frontend-web/          App Next.js (tienda, POS, panel admin)
docker-compose.yml     PostgreSQL 16 para desarrollo local
AGENTS.md / CLAUDE.md  Historial de decisiones de arquitectura e infraestructura
```

## Desarrollo local

Requisitos: Node.js 24, PostgreSQL 16 (o el `docker-compose.yml` incluido), npm.

**Backend**

```bash
cd backend-inventario
cp .env.example .env     # completa DATABASE_URL, JWT_SECRET, etc.
npm install
npm run db:deploy        # aplica migraciones
npm run db:seed          # catálogo base y usuarios de prueba
npm run dev               # http://localhost:4000
```

**Frontend**

```bash
cd frontend-web
cp .env.example .env.local   # completa API_URL y JWT_SECRET (igual al del backend)
npm install
npm run dev                   # http://localhost:3000
```

### Variables de entorno

`backend-inventario/.env` (ver `.env.example` para la lista completa y comentada): `DATABASE_URL`, `JWT_SECRET`, `JWT_EXPIRA`, `RESEND_API_KEY`, `CORREO_REMITENTE`, `APP_URL`, `TRUST_PROXY`.

`frontend-web/.env.local` (ver `.env.example`): `API_URL`, `JWT_SECRET`, `COOKIE_SECURE`.

### Scripts útiles del backend

| Script | Qué hace |
|---|---|
| `npm run db:deploy` | Aplica las migraciones de Prisma |
| `npm run db:seed` | Carga el catálogo base y usuarios de prueba |
| `npm run usuarios:correo -- <usuario> <correo>` | Asigna correo de login a un usuario existente |
| `npm run typecheck` | `tsc --noEmit` |
| `npm test` | Pruebas unitarias (`node:test` vía `tsx`) |

## Producción

Despliegue en una VPS Ubuntu 24.04: PostgreSQL 16 nativo, backend y frontend como servicios systemd, y Caddy como único punto de entrada público — emite HTTPS automático (Let's Encrypt) y sobrescribe `X-Forwarded-For` con la IP real del visitante, de la que dependen los límites de login y de compra.

