# Especificación Técnica — Sistema Digital CFI
## Certificado de Fomento Industrial — MICI Panamá

**Versión:** 1.0  
**Fecha:** Junio 2025  
**Institución:** Ministerio de Comercio e Industrias (MICI)  
**Unidad responsable:** Dirección General de Industrias — Departamento de Evaluación Industrial  
**Base legal:** Texto Único de la Ley N°76 del 23 de noviembre de 2009 · Decreto Ejecutivo N°37 del 10 de abril de 2018

---

## Tabla de contenidos

1. [Resumen ejecutivo](#1-resumen-ejecutivo)
2. [Actores y roles del sistema](#2-actores-y-roles-del-sistema)
3. [Arquitectura técnica](#3-arquitectura-técnica)
4. [Modelo de datos](#4-modelo-de-datos)
5. [API REST — Especificación de endpoints](#5-api-rest--especificación-de-endpoints)
6. [Máquina de estados del expediente](#6-máquina-de-estados-del-expediente)
7. [Reglas de negocio](#7-reglas-de-negocio)
8. [Módulo de votación CONAPI](#8-módulo-de-votación-conapi)
9. [Integraciones externas](#9-integraciones-externas)
10. [Seguridad y autenticación](#10-seguridad-y-autenticación)
11. [Almacenamiento de documentos](#11-almacenamiento-de-documentos)
12. [Notificaciones](#12-notificaciones)
13. [Criterios de aceptación por historia de usuario](#13-criterios-de-aceptación-por-historia-de-usuario)
14. [Plan de fases de implementación](#14-plan-de-fases-de-implementación)
15. [Requisitos no funcionales](#15-requisitos-no-funcionales)
16. [Glosario](#16-glosario)

---

## 1. Resumen ejecutivo

### Problema que resuelve

El proceso actual de solicitud del CFI es enteramente físico. El ciudadano debe presentarse al MICI con 11+ documentos en papel. El expediente pasa físicamente por 7 actores internos. El CONAPI recibe información por correo electrónico informal. El solicitante no tiene visibilidad del estado de su trámite en ningún momento. El tiempo estimado supera los 60 días hábiles.

### Solución propuesta

Sistema web de tres portales interconectados que digitalizan el flujo de punta a punta:

| Portal | Audiencia | URL propuesta |
|---|---|---|
| Portal ciudadano | Empresas solicitantes | `solicitud-cfi.mici.gob.pa` |
| Panel interno MICI | Funcionarios de la DGI | `funcionarios.cfi.mici.gob.pa` |
| Portal CONAPI | Miembros del Consejo | `conapi.cfi.mici.gob.pa` |

### Impacto esperado

| Métrica | Situación actual | Objetivo |
|---|---|---|
| Tiempo de resolución | 60+ días hábiles | 25–30 días hábiles |
| Visitas presenciales al MICI | 2–3 | 0 (solo visita técnica a empresa) |
| Traspasos físicos de expediente | 7+ | 0 |
| Visibilidad del ciudadano | Ninguna | Tiempo real |
| Documentos en papel | 15+ por expediente | 0 |

---

## 2. Actores y roles del sistema

### 2.1 Ciudadano / Empresa solicitante

Accede al portal ciudadano. Puede crear una cuenta, iniciar una solicitud, cargar documentos, completar todos los formularios de la Ley 76, firmar electrónicamente y consultar el estado de su expediente en tiempo real.

**Permisos:**
- Crear y editar su propia solicitud antes de envío
- Cargar y reemplazar documentos
- Consultar el estado y el historial de su expediente
- Recibir notificaciones por email
- Descargar el CFI digital cuando sea emitido

### 2.2 Secretaria CFI

Primera funcionaria del MICI que interactúa con el expediente. Valida la documentación recibida, folia digitalmente el expediente, asigna el analista industrial y el CPA, y gestiona las comunicaciones con el solicitante cuando hay documentos incompletos.

**Permisos sobre expedientes:** estado `recibido` y `observado`

### 2.3 Analista Industrial

Evalúa técnicamente el expediente. Agenda y realiza la visita técnica a la empresa. Redacta el Informe de Visita Técnica y el Informe Técnico con el concepto favorable o desfavorable. Elabora el borrador de la Resolución tras la aprobación del CONAPI.

**Permisos sobre expedientes:** estado `evaluacion` y `visita`

### 2.4 Contador Público Autorizado (CPA)

Revisa el Detalle de Inversiones factura por factura. Marca cada ítem como Conforme (C) o No Conforme (NC) con su observación. Emite el informe de CPA con el monto aprobado.

**Permisos sobre expedientes:** estado `evaluacion` y `visita`

### 2.5 Jefe del Departamento de Evaluación Industrial

Supervisa el Informe Técnico y el Detalle de Inversiones. Verifica que el Acta del CONAPI esté completa. Aprueba el envío al CONAPI o devuelve al analista. Recibe el borrador de Resolución del analista y lo remite al Asesor Legal.

**Permisos sobre expedientes:** estado `visita`, `conapi`, `resolucion`

### 2.6 Asesor Legal

Revisa la viabilidad jurídica del borrador de Resolución. Verifica que cite correctamente la base legal, que el monto no exceda el 40%, y que el número de Resolución sea correlativo. Emite su concepto legal antes de la firma del Director.

**Permisos sobre expedientes:** estado `resolucion`

### 2.7 Director General de Industrias

Firma digitalmente la Resolución que otorga el CFI. Su firma activa la generación del CFI digital con código QR y la notificación al solicitante.

**Permisos sobre expedientes:** estado `resolucion` y `emitido`

### 2.8 Secretaria CONAPI

Arma la agenda de cada sesión, distribuye los informes técnicos a los miembros, abre y cierra la sala de votación, y gestiona las firmas del Acta.

**Permisos:** gestión completa del módulo CONAPI

### 2.9 Miembro del CONAPI

Accede al portal del CONAPI para revisar los informes técnicos y emitir su voto (Favorable / Desfavorable / Abstención) con su PIN de firma digital.

**Permisos:** lectura de informes + emisión de voto en sesiones activas

---

## 3. Arquitectura técnica

### 3.1 Stack tecnológico recomendado

**Frontend:**
- Framework: Next.js 14+ (React) con App Router
- Estilos: Tailwind CSS + tokens de diseño institucional panameño
- Fuente: DM Sans (Google Fonts)
- Colores primarios: Navy `#0D2B55`, Teal `#0E7C6E`
- Estado cliente: Zustand o React Query
- Firma electrónica: integración con SDK del proveedor PKI nacional

**Backend:**
- Runtime: Node.js 20+ con TypeScript
- Framework: Fastify o NestJS
- ORM: Prisma (PostgreSQL)
- Cola de trabajos: BullMQ (Redis)
- Autenticación: JWT + refresh tokens, sesiones en Redis

**Base de datos:**
- Principal: PostgreSQL 15+ con cifrado AES-256 en reposo
- Caché y sesiones: Redis 7+
- Búsqueda: PostgreSQL full-text search (o Meilisearch si se requiere)

**Infraestructura:**
- Contenedores: Docker + Docker Compose (desarrollo), Kubernetes (producción)
- Object Storage: MinIO (self-hosted) o AWS S3 compatible
- CDN: Cloudflare o equivalente nacional
- CI/CD: GitHub Actions
- Monitoreo: Prometheus + Grafana
- Logs: Loki o ELK Stack

### 3.2 Diagrama de capas

```
┌─────────────────────────────────────────────────────────┐
│  CAPA DE PRESENTACIÓN (Next.js — GitHub Pages / CDN)    │
│  Portal ciudadano | Panel MICI | Portal CONAPI           │
└────────────────────────┬────────────────────────────────┘
                         │ HTTPS / REST API
┌────────────────────────▼────────────────────────────────┐
│  CAPA DE LÓGICA DE NEGOCIO (Node.js / Fastify)          │
│  Gestión expedientes | Motor de flujo | Votación | CFI  │
└────────────────────────┬────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────┐
│  SERVICIOS DE SOPORTE                                    │
│  Notificaciones | Storage S3 | Auditoría | PKI/Firma    │
└────────────────────────┬────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────┐
│  CAPA DE DATOS                                           │
│  PostgreSQL | Redis | Object Storage | Backup diario    │
└────────────────────────┬────────────────────────────────┘
                         │ APIs externas (solo lectura)
┌────────────────────────▼────────────────────────────────┐
│  INTEGRACIONES EXTERNAS                                  │
│  Registro Público | DGI-MEF | CSS | ATTT/Municipio      │
└─────────────────────────────────────────────────────────┘
```

### 3.3 Variables de entorno requeridas

```env
# Base de datos
DATABASE_URL=postgresql://user:password@host:5432/cfi_db
REDIS_URL=redis://host:6379

# JWT
JWT_SECRET=<256-bit-random-string>
JWT_EXPIRES_IN=8h
JWT_REFRESH_EXPIRES_IN=7d

# Object Storage
S3_ENDPOINT=https://storage.mici.gob.pa
S3_ACCESS_KEY=<key>
S3_SECRET_KEY=<secret>
S3_BUCKET_DOCUMENTOS=cfi-documentos
S3_BUCKET_CFI=cfi-certificados

# Notificaciones
SMTP_HOST=smtp.mici.gob.pa
SMTP_PORT=587
SMTP_USER=noreply@mici.gob.pa
SMTP_PASSWORD=<password>
SMS_PROVIDER_URL=<url>
SMS_API_KEY=<key>

# Integraciones
REGISTRO_PUBLICO_API_URL=https://api.registro.gob.pa/v1
REGISTRO_PUBLICO_API_KEY=<key>
DGI_PAZ_SALVO_API_URL=https://api.dgi.gob.pa/paz-salvo
CSS_PAZ_SALVO_API_URL=https://api.css.gob.pa/paz-salvo

# PKI / Firma electrónica
PKI_PROVIDER_URL=https://firma.gob.pa/api
PKI_ENTITY_CERT=<base64-cert>
PKI_ENTITY_KEY=<base64-private-key>

# App
NODE_ENV=production
PORT=3001
CORS_ORIGINS=https://solicitud-cfi.mici.gob.pa,https://funcionarios.cfi.mici.gob.pa,https://conapi.cfi.mici.gob.pa
```

---

## 4. Modelo de datos

### 4.1 Esquema completo (PostgreSQL / Prisma)

```prisma
// schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// ── USUARIOS Y ROLES ──────────────────────────────────────

enum RolUsuario {
  CIUDADANO
  SECRETARIA_CFI
  ANALISTA_INDUSTRIAL
  CONTADOR_CPA
  JEFE_DEPARTAMENTO
  ASESOR_LEGAL
  DIRECTOR_DGI
  SECRETARIA_CONAPI
  MIEMBRO_CONAPI
  ADMIN_SISTEMA
}

model Usuario {
  id           String      @id @default(uuid())
  email        String      @unique
  nombre       String
  apellidos    String
  cedula       String?     @unique
  rol          RolUsuario
  institucion  String?
  activo       Boolean     @default(true)
  passwordHash String
  createdAt    DateTime    @default(now())
  updatedAt    DateTime    @updatedAt

  expedientesGestionados Expediente[] @relation("AnalistaExpedientes")
  expedientesCPA         Expediente[] @relation("CPAExpedientes")
  informesTecnicos       InformeTecnico[]
  votos                  Voto[]
  auditoria              RegistroAuditoria[]

  @@map("usuarios")
}

// ── EMPRESA ───────────────────────────────────────────────

enum TipoPersona {
  JURIDICA
  NATURAL
}

model Empresa {
  id                    String      @id @default(uuid())
  tipo                  TipoPersona
  // Persona jurídica
  nombreComercial       String
  razonSocial           String?
  tomo                  String?
  folio                 String?
  asiento               String?
  ficha                 String?
  rollo                 String?
  imagen                String?
  // Persona natural
  nombres               String?
  apellidos             String?
  // Común
  cedula                String?
  pasaporte             String?
  nacionalidad          String?
  representanteLegal    String?
  cedulaRL              String?
  avisoOperacion        String?
  provincia             String
  distrito              String?
  corregimiento         String?
  zona                  String?
  calle                 String?
  telefono              String?
  email                 String
  actividadIndustrial   String?
  diasPorSemana         Int?
  turnosPorSemana       Int?
  horasLaborables       Int?
  createdAt             DateTime    @default(now())
  updatedAt             DateTime    @updatedAt

  usuario               Usuario?    @relation(fields: [usuarioId], references: [id])
  usuarioId             String?
  expedientes           Expediente[]

  @@map("empresas")
}

// ── EXPEDIENTE ────────────────────────────────────────────

enum EstadoExpediente {
  RECIBIDO
  EN_EVALUACION
  VISITA_TECNICA
  EN_CONAPI
  EN_RESOLUCION
  CFI_EMITIDO
  OBSERVADO
  RECHAZADO
  ARCHIVADO
}

enum Modalidad {
  INVESTIGACION_DESARROLLO
  GESTION_CALIDAD
  GESTION_AMBIENTAL
  INVERSION_REINVERSION
  CAPACITACION_RRHH
  INCREMENTO_EMPLEO
}

model Expediente {
  id                    String           @id @default(uuid())
  numero                String           @unique  // CFI-2025-XXXX
  empresa               Empresa          @relation(fields: [empresaId], references: [id])
  empresaId             String
  analista              Usuario?         @relation("AnalistaExpedientes", fields: [analistaId], references: [id])
  analistaId            String?
  cpa                   Usuario?         @relation("CPAExpedientes", fields: [cpaId], references: [id])
  cpaId                 String?
  estado                EstadoExpediente @default(RECIBIDO)
  modalidades           Modalidad[]
  montoInversion        Decimal?         @db.Decimal(15, 2)
  montoCFIEstimado      Decimal?         @db.Decimal(15, 2)  // 40% de montoInversion
  montoCFIAprobado      Decimal?         @db.Decimal(15, 2)  // Definido por CPA + CONAPI
  numFolios             Int?
  destinoNacional       Int?             // porcentaje 0-100
  destinoExportParcial  Int?
  destinoExportTotal    Int?
  incentiviosPrevios    Boolean          @default(false)
  contratoNacionNum     String?
  contratoNacionVence   DateTime?
  roinNum               String?
  ultimoCAT             DateTime?
  procesoProduccion     String?
  observacionesGen      String?
  fechaSolicitud        DateTime         @default(now())
  fechaAsignacion       DateTime?
  fechaVisita           DateTime?
  fechaConapi           DateTime?
  fechaResolucion       DateTime?
  fechaEmision          DateTime?
  createdAt             DateTime         @default(now())
  updatedAt             DateTime         @updatedAt

  documentos            Documento[]
  inversiones           Inversion[]
  productos             Producto[]
  maquinarias           Maquinaria[]
  materiasPrimas        MateriaPrima[]
  empaques              MaterialEmpaque[]
  visitaTecnica         VisitaTecnica?
  informeTecnico        InformeTecnico?
  sesionConapi          SesionConapi?    @relation(fields: [sesionConapiId], references: [id])
  sesionConapiId        String?
  resolucion            Resolucion?
  cfiDigital            CFIDigital?
  historial             HistorialEstado[]
  auditoria             RegistroAuditoria[]
  votos                 Voto[]
  empleos               EmpleoDetalle[]
  inversionesFijas      InversionFija[]

  @@map("expedientes")
}

// ── DOCUMENTOS ────────────────────────────────────────────

enum TipoDocumento {
  FORMULARIO_CFI
  REGISTRO_PUBLICO
  CEDULA_RL
  ESTADOS_FINANCIEROS
  PAZ_SALVO_MUNICIPAL
  PAZ_SALVO_CSS
  PAZ_SALVO_NACIONAL
  AVISO_OPERACION
  DECLARACION_NOTARIAL
  INFORME_INVERSION_CPA
  DOCUMENTOS_MODALIDAD
  INFORME_VISITA
  INFORME_TECNICO
  ACTA_CONAPI
  RESOLUCION
  CFI_DIGITAL
  OTRO
}

model Documento {
  id             String         @id @default(uuid())
  expediente     Expediente     @relation(fields: [expedienteId], references: [id])
  expedienteId   String
  tipo           TipoDocumento
  nombreArchivo  String
  urlStorage     String         // URL firmada S3
  hashSHA256     String         // Integridad del archivo
  tamanoBytes    Int
  mimeType       String
  validado       Boolean        @default(false)
  validadoPor    String?        // usuarioId del validador
  observacion    String?
  version        Int            @default(1)
  createdAt      DateTime       @default(now())

  @@map("documentos")
}

// ── DETALLE DE INVERSIONES ────────────────────────────────

enum TipoDocInversion {
  FF   // Factura Fiscal
  FC   // Factura Comercial
  FE   // Factura por Compra al Exterior
  DA   // Declaración de Aduanas
  BP   // Boleta de Pago
  TB   // Transferencia Bancaria
  EC   // Estado de Cuenta Bancario
  RE   // Retención Remesas al Exterior
  RI   // Retención ITBMS
}

model Inversion {
  id             String           @id @default(uuid())
  expediente     Expediente       @relation(fields: [expedienteId], references: [id])
  expedienteId   String
  folio          String?          // Asignado por el analista
  fecha          DateTime
  tipoDoc        TipoDocInversion
  numDocumento   String
  proveedor      String
  descripcion    String
  monto          Decimal          @db.Decimal(15, 2)
  itbms          Decimal?         @db.Decimal(15, 2)
  conforme       Boolean?         // null = sin revisar
  noConforme     Boolean?
  observacion    String?
  revisadoPor    String?          // usuarioId CPA o analista
  revisadoEn     DateTime?

  @@map("inversiones")
}

// ── VISITA TÉCNICA ────────────────────────────────────────

model VisitaTecnica {
  id                   String     @id @default(uuid())
  expediente           Expediente @relation(fields: [expedienteId], references: [id])
  expedienteId         String     @unique
  fechaProgramada      DateTime
  horaProgramada       String
  analistasParticipan  String
  notificadoCiudadano  Boolean    @default(false)
  fechaRealizada       DateTime?
  activosVerificados   Boolean?
  personalPresente     String?
  observaciones        String?
  firmadoPor           String?    // usuarioId analista
  firmadoEn            DateTime?
  createdAt            DateTime   @default(now())

  @@map("visitas_tecnicas")
}

// ── INFORME TÉCNICO ───────────────────────────────────────

enum ConceptoInforme {
  FAVORABLE
  FAVORABLE_CON_OBSERVACIONES
  DESFAVORABLE
}

model InformeTecnico {
  id                String          @id @default(uuid())
  expediente        Expediente      @relation(fields: [expedienteId], references: [id])
  expedienteId      String          @unique
  analista          Usuario         @relation(fields: [analistaId], references: [id])
  analistaId        String
  montoVerificado   Decimal         @db.Decimal(15, 2)
  montoCFIRec       Decimal         @db.Decimal(15, 2)
  concepto          ConceptoInforme
  sustento          String
  aprobadoJefe      Boolean         @default(false)
  aprobadoJefeEn    DateTime?
  firmaDigital      String?         // hash de firma PKI
  firmadoEn         DateTime?
  createdAt         DateTime        @default(now())

  @@map("informes_tecnicos")
}

// ── SESIÓN CONAPI ─────────────────────────────────────────

enum EstadoSesion {
  PROGRAMADA
  EN_CURSO
  CERRADA
}

model SesionConapi {
  id             String       @id @default(uuid())
  numeroSesion   String       @unique  // "09/2025"
  fecha          DateTime
  hora           String
  modalidad      String       // "Presencial", "Virtual", "Mixta"
  quorum         Int          @default(0)
  estado         EstadoSesion @default(PROGRAMADA)
  createdAt      DateTime     @default(now())

  expedientes    Expediente[]
  votos          Voto[]
  acta           ActaSesion?

  @@map("sesiones_conapi")
}

enum ResultadoVoto {
  FAVORABLE
  DESFAVORABLE
  ABSTENCION
}

model Voto {
  id             String        @id @default(uuid())
  sesion         SesionConapi  @relation(fields: [sesionId], references: [id])
  sesionId       String
  expediente     Expediente    @relation(fields: [expedienteId], references: [id])
  expedienteId   String
  miembro        Usuario       @relation(fields: [miembroId], references: [id])
  miembroId      String
  resultado      ResultadoVoto
  pinFirmaHash   String        // hash del PIN de firma
  emitidoEn      DateTime      @default(now())

  @@unique([sesionId, expedienteId, miembroId])
  @@map("votos")
}

model ActaSesion {
  id             String       @id @default(uuid())
  sesion         SesionConapi @relation(fields: [sesionId], references: [id])
  sesionId       String       @unique
  contenidoHTML  String
  urlStorage     String?
  hashSHA256     String?
  firmada        Boolean      @default(false)
  firmadaEn      DateTime?
  createdAt      DateTime     @default(now())

  @@map("actas_sesion")
}

// ── RESOLUCIÓN Y CFI DIGITAL ──────────────────────────────

model Resolucion {
  id              String     @id @default(uuid())
  expediente      Expediente @relation(fields: [expedienteId], references: [id])
  expedienteId    String     @unique
  numero          String     @unique
  fecha           DateTime
  montoAprobado   Decimal    @db.Decimal(15, 2)
  contenidoHTML   String
  viabiLidadLegal Boolean    @default(false)
  viabiLidadEn    DateTime?
  firmaDirector   String?    // hash de firma PKI
  firmadaEn       DateTime?
  urlStorage      String?
  createdAt       DateTime   @default(now())

  cfiDigital      CFIDigital?

  @@map("resoluciones")
}

model CFIDigital {
  id             String     @id @default(uuid())
  resolucion     Resolucion @relation(fields: [resolucionId], references: [id])
  resolucionId   String     @unique
  expediente     Expediente @relation(fields: [expedienteId], references: [id])
  expedienteId   String     @unique
  codigoQR       String     @unique  // UUID verificable
  urlDescarga    String
  hashSHA256     String
  emitidoEn      DateTime   @default(now())
  notificado     Boolean    @default(false)
  notificadoEn   DateTime?

  @@map("cfi_digitales")
}

// ── AUDITORÍA ─────────────────────────────────────────────

model HistorialEstado {
  id             String           @id @default(uuid())
  expediente     Expediente       @relation(fields: [expedienteId], references: [id])
  expedienteId   String
  estadoAnterior EstadoExpediente?
  estadoNuevo    EstadoExpediente
  usuarioId      String
  comentario     String?
  createdAt      DateTime         @default(now())

  @@map("historial_estados")
}

model RegistroAuditoria {
  id           String     @id @default(uuid())
  usuario      Usuario    @relation(fields: [usuarioId], references: [id])
  usuarioId    String
  expediente   Expediente? @relation(fields: [expedienteId], references: [id])
  expedienteId String?
  accion       String
  entidad      String
  entidadId    String?
  detalle      Json?
  ip           String?
  userAgent    String?
  createdAt    DateTime   @default(now())

  @@map("registro_auditoria")
}
```

---

## 5. API REST — Especificación de endpoints

**Base URL:** `https://api.cfi.mici.gob.pa/v1`  
**Autenticación:** `Authorization: Bearer <JWT>`  
**Content-Type:** `application/json`

### 5.1 Autenticación

#### `POST /auth/login`
Inicia sesión y retorna JWT.

**Request:**
```json
{
  "email": "empresa@ejemplo.com",
  "password": "contraseña"
}
```

**Response 200:**
```json
{
  "accessToken": "eyJ...",
  "refreshToken": "eyJ...",
  "expiresIn": 28800,
  "usuario": {
    "id": "uuid",
    "nombre": "Carlos Méndez",
    "rol": "CIUDADANO",
    "email": "empresa@ejemplo.com"
  }
}
```

#### `POST /auth/refresh`
Renueva el accessToken con el refreshToken.

#### `POST /auth/logout`
Invalida el refreshToken en Redis.

#### `POST /auth/registro`
Registro de nuevas empresas (solo rol CIUDADANO).

---

### 5.2 Expedientes

#### `POST /expedientes`
Crea un nuevo expediente. Solo ciudadanos autenticados.

**Request:**
```json
{
  "empresa": {
    "tipo": "JURIDICA",
    "nombreComercial": "Industrias Panamá S.A.",
    "razonSocial": "Industrias Panamá S.A.",
    "tomo": "100",
    "folio": "50",
    "asiento": "1",
    "representanteLegal": "Carlos Méndez",
    "cedulaRL": "8-123-4567",
    "provincia": "Panamá",
    "email": "empresa@ejemplo.com",
    "avisoOperacion": "AO-2024-001"
  },
  "modalidades": ["INVERSION_REINVERSION"],
  "destinoNacional": 70,
  "destinoExportParcial": 30,
  "procesoProduccion": "Descripción del proceso..."
}
```

**Response 201:**
```json
{
  "id": "uuid",
  "numero": "CFI-2025-0413",
  "estado": "RECIBIDO",
  "fechaSolicitud": "2025-06-02T10:00:00Z"
}
```

#### `GET /expedientes`
Lista expedientes. Filtrado automático por rol:
- CIUDADANO: solo sus propios expedientes
- SECRETARIA_CFI: todos en estado RECIBIDO y OBSERVADO
- ANALISTA_INDUSTRIAL: asignados a él en EVALUACION y VISITA_TECNICA
- etc.

**Query params:** `?estado=RECIBIDO&page=1&limit=20&sort=fechaSolicitud&order=desc`

**Response 200:**
```json
{
  "data": [
    {
      "id": "uuid",
      "numero": "CFI-2025-0412",
      "empresa": { "nombreComercial": "Industrias Tropicales S.A." },
      "estado": "RECIBIDO",
      "modalidades": ["INVERSION_REINVERSION"],
      "montoInversion": 285000,
      "montoCFIEstimado": 114000,
      "diasEnTramite": 2,
      "fechaSolicitud": "2025-05-28T08:00:00Z"
    }
  ],
  "total": 24,
  "page": 1,
  "limit": 20
}
```

#### `GET /expedientes/:id`
Retorna el expediente completo con todos sus sub-recursos.

#### `PATCH /expedientes/:id/estado`
Transiciona el estado del expediente. Solo roles autorizados para cada transición.

**Request:**
```json
{
  "estadoNuevo": "EN_EVALUACION",
  "comentario": "Documentación completa. Asignado a Luis Ramos.",
  "analistaId": "uuid-analista",
  "cpaId": "uuid-cpa",
  "numFolios": 42
}
```

**Validaciones del backend:**
- Verifica que la transición sea válida según la máquina de estados (ver sección 6)
- Verifica que el usuario tenga rol autorizado para esa transición
- Registra el cambio en `historial_estados` y `registro_auditoria`
- Dispara las notificaciones correspondientes

**Response 200:**
```json
{
  "id": "uuid",
  "estadoAnterior": "RECIBIDO",
  "estadoNuevo": "EN_EVALUACION",
  "updatedAt": "2025-06-02T11:30:00Z"
}
```

---

### 5.3 Documentos

#### `POST /expedientes/:id/documentos`
Sube un documento al expediente.

**Request:** `multipart/form-data`
```
tipo: "PAZ_SALVO_MUNICIPAL"
archivo: <File PDF>
```

**Proceso backend:**
1. Validar tipo MIME (solo `application/pdf`)
2. Validar tamaño máximo (10 MB)
3. Calcular SHA-256 del archivo
4. Subir a Object Storage con clave `expedientes/{expedienteId}/documentos/{tipo}/{uuid}.pdf`
5. Generar URL firmada con TTL de 24h
6. Insertar registro en tabla `documentos`

**Response 201:**
```json
{
  "id": "uuid",
  "tipo": "PAZ_SALVO_MUNICIPAL",
  "nombreArchivo": "paz-salvo-municipal.pdf",
  "urlDescarga": "https://storage.../...",
  "hashSHA256": "abc123...",
  "tamanoBytes": 245678
}
```

#### `GET /expedientes/:id/documentos`
Lista todos los documentos del expediente.

#### `PATCH /expedientes/:id/documentos/:docId/validar`
Marca un documento como validado. Solo SECRETARIA_CFI, ANALISTA o CPA.

```json
{
  "validado": true,
  "observacion": "Documento legible y vigente."
}
```

---

### 5.4 Inversiones (Detalle de inversiones)

#### `POST /expedientes/:id/inversiones`
Agrega una fila al detalle de inversiones.

**Request:**
```json
{
  "fecha": "2024-03-15",
  "tipoDoc": "FF",
  "numDocumento": "00123",
  "proveedor": "Equipos Tech S.A.",
  "descripcion": "Mezcladora industrial 500L",
  "monto": 45000.00,
  "itbms": null
}
```

#### `POST /expedientes/:id/inversiones/bulk`
Importa múltiples filas desde el Excel MICI (parseo del .xls del formulario oficial).

**Request:** `multipart/form-data`
```
archivo: <File XLS>
```

#### `PATCH /expedientes/:id/inversiones/:invId/revision`
CPA o analista marcan C/NC en una fila.

```json
{
  "conforme": false,
  "noConforme": true,
  "observacion": "Falta Declaración de Aduanas para compra al exterior.",
  "folio": "003"
}
```

#### `GET /expedientes/:id/inversiones/resumen`
Retorna el total declarado, total conforme, total no conforme y CFI estimado.

```json
{
  "totalDeclarado": 135000.00,
  "totalConforme": 70000.00,
  "totalNoConforme": 65000.00,
  "cfiEstimado": 54000.00,
  "cfiConforme": 28000.00,
  "itemsTotal": 4,
  "itemsConformes": 3,
  "itemsNoConformes": 1
}
```

---

### 5.5 Visita técnica

#### `POST /expedientes/:id/visita`
Agenda la visita técnica.

```json
{
  "fechaProgramada": "2025-06-10",
  "horaProgramada": "09:00",
  "analistasParticipan": "Luis Ramos, Pedro Vásquez"
}
```

El backend envía automáticamente el email de notificación al Representante Legal.

#### `PATCH /expedientes/:id/visita`
Completa el informe de visita técnica.

```json
{
  "fechaRealizada": "2025-06-10",
  "activosVerificados": true,
  "personalPresente": "Gerente de planta: Mario García",
  "observaciones": "Todos los activos declarados están instalados y operando..."
}
```

---

### 5.6 Informe técnico

#### `POST /expedientes/:id/informe-tecnico`
El analista crea el informe técnico.

```json
{
  "montoVerificado": 145000.00,
  "montoCFIRec": 58000.00,
  "concepto": "FAVORABLE",
  "sustento": "La empresa cumple con los requisitos del Art. 15 del D.E. N°37 de 2018..."
}
```

#### `PATCH /expedientes/:id/informe-tecnico/aprobar-jefe`
El Jefe de Departamento aprueba el informe.

```json
{
  "aprobado": true,
  "comentarios": "Informe completo y bien fundamentado."
}
```

---

### 5.7 CONAPI

#### `GET /conapi/sesiones`
Lista sesiones del CONAPI.

#### `POST /conapi/sesiones`
Crea una nueva sesión. Solo SECRETARIA_CONAPI.

```json
{
  "numeroSesion": "09/2025",
  "fecha": "2025-06-02",
  "hora": "10:00",
  "modalidad": "Presencial/Virtual mixta",
  "expedientesIds": ["uuid-1", "uuid-2", "uuid-3"]
}
```

#### `POST /conapi/sesiones/:id/votos`
Emite un voto. Solo MIEMBRO_CONAPI en sesión activa.

```json
{
  "expedienteId": "uuid",
  "resultado": "FAVORABLE",
  "pinFirma": "123456"
}
```

**Validaciones:**
- La sesión debe estar en estado EN_CURSO
- El miembro no puede haber votado ya este expediente en esta sesión
- El PIN de firma es validado contra el proveedor PKI
- Registro inmutable en base de datos con timestamp

#### `GET /conapi/sesiones/:id/resultado`
Retorna el resultado de la votación para un expediente.

```json
{
  "expedienteId": "uuid",
  "totalMiembros": 6,
  "votosEmitidos": 6,
  "favorables": 5,
  "desfavorables": 0,
  "abstenciones": 1,
  "resultado": "FAVORABLE",
  "quorumAlcanzado": true
}
```

#### `POST /conapi/sesiones/:id/acta`
Genera el Acta automáticamente.

#### `POST /conapi/sesiones/:id/acta/firmar`
Un miembro firma digitalmente el Acta con su PIN.

---

### 5.8 Resolución y CFI

#### `POST /expedientes/:id/resolucion`
El analista crea el borrador de resolución.

#### `PATCH /expedientes/:id/resolucion/viabilidad-legal`
Asesor Legal emite su concepto.

```json
{
  "viable": true,
  "observaciones": "El expediente cumple con todos los requisitos legales..."
}
```

#### `PATCH /expedientes/:id/resolucion/firmar`
Director DGI firma la Resolución.

```json
{
  "numero": "45",
  "fecha": "2025-06-02",
  "pinFirma": "123456"
}
```

El backend, al recibir la firma del Director:
1. Valida el PIN contra el proveedor PKI
2. Aplica la firma digital al PDF de la Resolución
3. Genera el CFI digital con código QR único
4. Sube ambos documentos a Object Storage
5. Transiciona el expediente a estado `CFI_EMITIDO`
6. Envía notificación por email y SMS al solicitante

#### `GET /cfi/:codigoQR/verificar`
Endpoint público para verificar la autenticidad de un CFI digital (accesible sin autenticación).

```json
{
  "valido": true,
  "empresa": "Industrias Panamá S.A.",
  "numeroResolucion": "45",
  "fecha": "2025-06-02",
  "monto": 114000.00,
  "modalidad": "INVERSION_REINVERSION"
}
```

---

### 5.9 Integraciones externas

#### `GET /integraciones/registro-publico/:ruc`
Consulta la existencia de la empresa en Registro Público.

#### `GET /integraciones/paz-salvo/municipal/:ruc`
Consulta el Paz y Salvo Municipal.

#### `GET /integraciones/paz-salvo/css/:ruc`
Consulta el Paz y Salvo de la CSS.

#### `GET /integraciones/paz-salvo/dgi/:ruc`
Consulta el Paz y Salvo Nacional (DGI-MEF).

Todos estos endpoints hacen caché en Redis por 24 horas para reducir la carga en los sistemas externos.

---

## 6. Máquina de estados del expediente

### 6.1 Estados válidos

| Estado | Descripción | Actor responsable |
|---|---|---|
| `RECIBIDO` | Solicitud enviada por el ciudadano | Secretaria CFI |
| `EN_EVALUACION` | Documentos validados, analista y CPA asignados | Analista / CPA |
| `VISITA_TECNICA` | Visita técnica agendada o en proceso | Analista |
| `EN_CONAPI` | Informe técnico enviado a sesión del CONAPI | CONAPI |
| `EN_RESOLUCION` | CONAPI aprobó, se redacta la Resolución | Analista / Legal / Director |
| `CFI_EMITIDO` | Director firmó, CFI digital generado | Sistema |
| `OBSERVADO` | Documentos incompletos, pendiente del ciudadano | Secretaria CFI |
| `RECHAZADO` | CONAPI denegó la solicitud | Sistema |
| `ARCHIVADO` | Expediente cerrado por inactividad | Admin |

### 6.2 Transiciones válidas

```
RECIBIDO ──────────────────────► EN_EVALUACION   (Secretaria CFI valida docs + asigna)
RECIBIDO ──────────────────────► OBSERVADO        (Secretaria CFI detecta docs faltantes)
OBSERVADO ─────────────────────► RECIBIDO         (Ciudadano envía documentos faltantes)
EN_EVALUACION ─────────────────► VISITA_TECNICA   (Analista agenda visita)
EN_EVALUACION ─────────────────► OBSERVADO        (CPA detecta faltantes)
VISITA_TECNICA ────────────────► EN_CONAPI        (Jefe aprueba informe técnico)
EN_CONAPI ─────────────────────► EN_RESOLUCION    (CONAPI vota favorable)
EN_CONAPI ─────────────────────► RECHAZADO        (CONAPI vota desfavorable)
EN_RESOLUCION ─────────────────► CFI_EMITIDO      (Director firma Resolución)
CFI_EMITIDO ───────────────────► ARCHIVADO        (Admin archiva tras 5 años)
RECHAZADO ─────────────────────► ARCHIVADO        (Admin archiva)
```

### 6.3 Validación de transiciones en el backend

```typescript
const TRANSICIONES_VALIDAS: Record<EstadoExpediente, EstadoExpediente[]> = {
  RECIBIDO:        ['EN_EVALUACION', 'OBSERVADO'],
  OBSERVADO:       ['RECIBIDO'],
  EN_EVALUACION:   ['VISITA_TECNICA', 'OBSERVADO'],
  VISITA_TECNICA:  ['EN_CONAPI'],
  EN_CONAPI:       ['EN_RESOLUCION', 'RECHAZADO'],
  EN_RESOLUCION:   ['CFI_EMITIDO'],
  CFI_EMITIDO:     ['ARCHIVADO'],
  RECHAZADO:       ['ARCHIVADO'],
  ARCHIVADO:       [],
};

const ROLES_POR_TRANSICION: Record<string, RolUsuario[]> = {
  'RECIBIDO->EN_EVALUACION':        ['SECRETARIA_CFI'],
  'RECIBIDO->OBSERVADO':            ['SECRETARIA_CFI'],
  'OBSERVADO->RECIBIDO':            ['CIUDADANO'],
  'EN_EVALUACION->VISITA_TECNICA':  ['ANALISTA_INDUSTRIAL'],
  'EN_EVALUACION->OBSERVADO':       ['CONTADOR_CPA', 'ANALISTA_INDUSTRIAL'],
  'VISITA_TECNICA->EN_CONAPI':      ['JEFE_DEPARTAMENTO'],
  'EN_CONAPI->EN_RESOLUCION':       ['SECRETARIA_CONAPI'],
  'EN_CONAPI->RECHAZADO':           ['SECRETARIA_CONAPI'],
  'EN_RESOLUCION->CFI_EMITIDO':     ['DIRECTOR_DGI'],
  'CFI_EMITIDO->ARCHIVADO':         ['ADMIN_SISTEMA'],
  'RECHAZADO->ARCHIVADO':           ['ADMIN_SISTEMA'],
};

async function transicionarEstado(
  expedienteId: string,
  estadoNuevo: EstadoExpediente,
  usuarioId: string,
  rol: RolUsuario,
  comentario?: string
): Promise<void> {
  const expediente = await prisma.expediente.findUniqueOrThrow({
    where: { id: expedienteId }
  });

  const estadoActual = expediente.estado;
  const transicionKey = `${estadoActual}->${estadoNuevo}`;

  if (!TRANSICIONES_VALIDAS[estadoActual].includes(estadoNuevo)) {
    throw new Error(`Transición inválida: ${transicionKey}`);
  }

  if (!ROLES_POR_TRANSICION[transicionKey]?.includes(rol)) {
    throw new Error(`Rol ${rol} no autorizado para transición ${transicionKey}`);
  }

  await prisma.$transaction([
    prisma.expediente.update({
      where: { id: expedienteId },
      data: { estado: estadoNuevo }
    }),
    prisma.historialEstado.create({
      data: {
        expedienteId,
        estadoAnterior: estadoActual,
        estadoNuevo,
        usuarioId,
        comentario
      }
    }),
    prisma.registroAuditoria.create({
      data: {
        usuarioId,
        expedienteId,
        accion: 'TRANSICION_ESTADO',
        entidad: 'Expediente',
        entidadId: expedienteId,
        detalle: { estadoAnterior: estadoActual, estadoNuevo, comentario }
      }
    })
  ]);

  await enviarNotificacionTransicion(expedienteId, estadoNuevo);
}
```

---

## 7. Reglas de negocio

### 7.1 Cálculo del CFI

```
CFI_ESTIMADO = MONTO_INVERSION_DECLARADO × 0.40
CFI_APROBADO = MONTO_CONFORME_CPA × 0.40

El CFI aprobado por el CONAPI no puede exceder el CFI_ESTIMADO.
El CFI aprobado no puede exceder el límite establecido en el Art. 10 de la Ley N°76.
```

### 7.2 Validación de documentos requeridos

Antes de permitir la transición `RECIBIDO → EN_EVALUACION`, el sistema valida que existan documentos cargados para TODOS los tipos obligatorios:

```typescript
const DOCUMENTOS_OBLIGATORIOS = [
  'FORMULARIO_CFI',
  'REGISTRO_PUBLICO',
  'CEDULA_RL',
  'ESTADOS_FINANCIEROS',
  'PAZ_SALVO_MUNICIPAL',
  'PAZ_SALVO_CSS',
  'PAZ_SALVO_NACIONAL',
  'AVISO_OPERACION',
  'DECLARACION_NOTARIAL',
  'INFORME_INVERSION_CPA',
];
```

### 7.3 Exclusión de beneficios previos

Al recibir una solicitud, el sistema consulta automáticamente la base de datos interna del MICI para verificar que la empresa no tenga ningún beneficio fiscal activo. Si lo tiene, la solicitud es bloqueada antes del envío con un mensaje explicativo.

### 7.4 Quórum del CONAPI

- El quórum mínimo para sesionar es de 4 de 6 miembros.
- El concepto es FAVORABLE si más de la mitad de los miembros presentes votan favorablemente.
- Las abstenciones no cuentan como votos en contra.
- Una sesión con quórum insuficiente no puede abrirse.

### 7.5 Número correlativo de Resolución

El sistema genera automáticamente el número de Resolución como correlativo dentro del año fiscal. El Asesor Legal verifica que el número sea correcto antes de dar viabilidad.

### 7.6 Vigencia del CFI

El CFI emitido tiene una vigencia de 3 años fiscales contados desde la fecha de la Resolución, conforme al Texto Único de la Ley N°76.

---

## 8. Módulo de votación CONAPI

### 8.1 Flujo completo

```
1. Secretaria CONAPI crea la sesión con la lista de expedientes
2. Sistema notifica a los 6 miembros con enlace de acceso seguro
3. Secretaria CONAPI abre la sala de votación (estado: EN_CURSO)
4. Cada miembro accede con sus credenciales
5. Cada miembro revisa el Informe Técnico de cada caso
6. Cada miembro emite su voto con su PIN de firma
7. El sistema registra el voto de forma inmutable con timestamp
8. Cuando todos los miembros han votado, el sistema calcula el resultado
9. Secretaria CONAPI cierra la sesión
10. Sistema genera el Acta automáticamente
11. Cada miembro firma digitalmente el Acta con su PIN
12. Secretaria CONAPI remite el Acta al expediente
13. Sistema transiciona el expediente según el resultado
```

### 8.2 Seguridad del voto

- Los votos son inmutables una vez emitidos (sin UPDATE, sin DELETE en la tabla `votos`)
- Cada voto incluye el timestamp exacto del servidor (no del cliente)
- El PIN de firma es validado contra el proveedor PKI nacional antes de registrar el voto
- El hash del PIN se almacena, nunca el PIN en texto plano
- El resultado es calculado por el backend, nunca por el frontend

---

## 9. Integraciones externas

### 9.1 Registro Público de Panamá

**Propósito:** Verificar la existencia legal y estado de la sociedad  
**Datos obtenidos:** Nombre de la sociedad, estado (activa/cancelada), representante legal, fecha de inscripción  
**Modo:** API REST (solo lectura)  
**Caché:** 24 horas en Redis  
**Fallback:** Si la API no responde, se permite continuar y el analista valida manualmente el certificado en PDF

### 9.2 Dirección General de Ingresos (DGI — MEF)

**Propósito:** Verificar Paz y Salvo Nacional  
**Datos obtenidos:** Estado del paz y salvo, fecha de vencimiento  
**Modo:** API REST (solo lectura)  
**Caché:** 6 horas en Redis

### 9.3 Caja de Seguro Social (CSS)

**Propósito:** Verificar Paz y Salvo de la CSS  
**Modo:** API REST (solo lectura)  
**Caché:** 6 horas en Redis

### 9.4 ATTT / Municipio de Panamá

**Propósito:** Verificar Paz y Salvo Municipal  
**Nota:** La integración varía por municipio. Inicialmente se implementa para el Municipio de Panamá. Para otras provincias, el ciudadano carga el documento PDF.

### 9.5 Proveedor PKI (Firma electrónica)

**Propósito:** Validar PINs de firma y aplicar firmas digitales a documentos  
**Documentos firmados:** Informe Técnico, Acta del CONAPI, Resolución, CFI Digital  
**Proveedor recomendado:** FirmadorGob o equivalente certificado por la AIG

---

## 10. Seguridad y autenticación

### 10.1 Autenticación

- JWT con expiración de 8 horas para accessToken
- Refresh token de 7 días almacenado en Redis (permite revocación)
- Contraseñas hasheadas con bcrypt (costo 12)
- Rate limiting: máximo 5 intentos de login fallidos por IP en 15 minutos

### 10.2 Autorización

- RBAC (Role-Based Access Control) implementado a nivel de middleware
- Cada endpoint especifica los roles autorizados
- Los datos están filtrados por rol en todas las consultas (ciudadano solo ve sus expedientes)

### 10.3 Cifrado

- HTTPS/TLS 1.3 en todas las comunicaciones
- Datos en reposo cifrados con AES-256 en PostgreSQL
- Documentos en Object Storage cifrados con SSE-S3
- Variables de entorno gestionadas con secretos del sistema (no en código)

### 10.4 Auditoría

Toda acción sobre un expediente queda registrada en `registro_auditoria` con:
- Usuario que realizó la acción
- Timestamp del servidor
- IP de origen
- Acción ejecutada
- Estado anterior y nuevo (para transiciones)
- Payload relevante en JSON

Los registros de auditoría son inmutables (solo INSERT, sin UPDATE ni DELETE).

### 10.5 Headers de seguridad

```
Content-Security-Policy: default-src 'self'
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Strict-Transport-Security: max-age=31536000; includeSubDomains
Referrer-Policy: strict-origin-when-cross-origin
```

---

## 11. Almacenamiento de documentos

### 11.1 Estructura de claves en Object Storage

```
cfi-documentos/
  expedientes/
    {expedienteId}/
      documentos/
        {tipoDocumento}/
          {uuid}_{version}.pdf
      inversiones/
        detalle-inversiones.xlsx
  informes/
    {expedienteId}/
      informe-tecnico-{timestamp}.pdf
      acta-conapi-{sesionId}.pdf
  resoluciones/
    {expedienteId}/
      resolucion-{numeroResolucion}.pdf

cfi-certificados/
  {expedienteId}/
    CFI-{numeroResolucion}-{codigoQR}.pdf
```

### 11.2 URLs firmadas

Los documentos no son accesibles directamente. Se generan URLs firmadas con TTL de 1 hora para descarga y de 15 minutos para visualización. Esto garantiza que solo usuarios autenticados y autorizados accedan a los documentos.

### 11.3 Integridad

Cada documento tiene su SHA-256 calculado en el backend al momento de la carga. Cualquier descarga puede verificarse contra este hash. Los documentos firmados digitalmente también incluyen el certificado de firma incrustado.

---

## 12. Notificaciones

### 12.1 Eventos que disparan notificaciones

| Evento | Canal | Destinatario |
|---|---|---|
| Solicitud recibida | Email | Representante Legal, Secretaria CFI |
| Documentos incompletos | Email + SMS | Representante Legal |
| Visita técnica agendada | Email + SMS | Representante Legal |
| Expediente enviado al CONAPI | Email | Jefe de Depto., Secretaria CONAPI |
| Sesión CONAPI programada | Email | Todos los miembros del CONAPI |
| Resultado de votación | Email | Jefe de Depto., Director |
| Resolución pendiente de firma | Email | Director DGI |
| CFI emitido | Email + SMS | Representante Legal |
| Solicitud rechazada | Email | Representante Legal |

### 12.2 Plantillas de email (ejemplo)

```html
<!-- CFI Emitido -->
Asunto: Su Certificado de Fomento Industrial ha sido emitido — {{numero}}

Estimado/a {{nombreRL}},

Nos complace informarle que el Certificado de Fomento Industrial
correspondiente a la empresa {{nombreEmpresa}} ha sido emitido.

RESOLUCIÓN N° {{numeroResolucion}}
MONTO APROBADO: B/. {{montoAprobado}}
CÓDIGO DE VERIFICACIÓN: {{codigoQR}}

Puede descargar su CFI digital desde el siguiente enlace:
{{urlDescarga}}

Para verificar la autenticidad de su certificado visite:
https://api.cfi.mici.gob.pa/v1/cfi/{{codigoQR}}/verificar
```

---

## 13. Criterios de aceptación por historia de usuario

### HU-01: Ciudadano envía solicitud de CFI

**Como** representante legal de una empresa,  
**quiero** poder completar y enviar mi solicitud del CFI en línea,  
**para** evitar desplazarme al MICI.

**Criterios de aceptación:**
- [ ] El ciudadano puede registrar una cuenta con email y contraseña
- [ ] El formulario incluye todas las secciones del formulario oficial (Secciones I–VIII de la Ley 76)
- [ ] El sistema permite cargar los 11 tipos de documentos requeridos en PDF
- [ ] El sistema calcula automáticamente el CFI estimado (40% de la inversión declarada)
- [ ] El sistema valida que todos los campos obligatorios estén completos antes de permitir el envío
- [ ] Al enviar, el sistema genera un número de expediente único (CFI-YYYY-XXXX)
- [ ] El ciudadano recibe un email de confirmación con el número de expediente en menos de 2 minutos
- [ ] El ciudadano puede consultar el estado de su expediente en tiempo real
- [ ] Si la empresa ya tiene beneficios fiscales activos, el sistema lo indica antes de enviar

### HU-02: Secretaria CFI valida documentación

**Como** Secretaria CFI del MICI,  
**quiero** revisar y validar los documentos de cada solicitud recibida,  
**para** asegurar que estén completos antes de asignar el expediente.

**Criterios de aceptación:**
- [ ] La secretaria ve una bandeja con todos los expedientes en estado RECIBIDO
- [ ] Puede abrir cada documento en el navegador sin descargarlo
- [ ] Puede marcar cada documento como "Completo" o "Incompleto" con una observación
- [ ] Puede asignar un analista industrial y un CPA de la lista de funcionarios disponibles
- [ ] Puede registrar el número de folios digitalmente
- [ ] Si hay documentos incompletos, puede devolver la solicitud al ciudadano con una nota explicativa
- [ ] Al confirmar la recepción, el sistema notifica automáticamente al analista y al CPA
- [ ] Toda acción queda registrada en el historial del expediente con timestamp

### HU-03: Analista realiza visita técnica

**Como** analista industrial,  
**quiero** agendar y documentar la visita técnica a la empresa,  
**para** verificar que los activos fijos estén instalados y operando.

**Criterios de aceptación:**
- [ ] El analista puede seleccionar fecha, hora y analistas participantes en un formulario de agenda
- [ ] Al confirmar la agenda, el sistema envía una notificación automática al Representante Legal
- [ ] El formulario de informe de visita incluye: fecha realizada, verificación de activos, personal presente, observaciones
- [ ] El analista puede completar el Informe Técnico con monto verificado, CFI recomendado y concepto
- [ ] El concepto es: Favorable, Favorable con observaciones o Desfavorable
- [ ] El informe es enviado al Jefe de Departamento para su aprobación
- [ ] El Jefe puede aprobar o devolver el informe con comentarios
- [ ] Al aprobar, el expediente pasa automáticamente a EN_CONAPI

### HU-04: CPA revisa el detalle de inversiones

**Como** CPA asignado al expediente,  
**quiero** revisar cada factura del detalle de inversiones,  
**para** determinar cuáles cumplen con los requisitos de la Ley 76.

**Criterios de aceptación:**
- [ ] El CPA ve el detalle de inversiones en una tabla con todos los campos del formato MICI
- [ ] Puede marcar cada ítem como Conforme (C) o No Conforme (NC) con una observación
- [ ] El sistema calcula automáticamente el total conforme y el CFI sobre el monto conforme
- [ ] Puede solicitar documentos adicionales al solicitante desde el sistema
- [ ] El informe del CPA incluye el monto aprobado y las observaciones generales
- [ ] Su firma digital queda registrada en el informe

### HU-05: CONAPI vota digitalmente

**Como** miembro del CONAPI,  
**quiero** revisar los informes técnicos y votar en línea,  
**para** poder participar en las sesiones sin necesidad de asistir físicamente.

**Criterios de aceptación:**
- [ ] Recibo una notificación por email con el enlace a la sala de votación al menos 48 horas antes
- [ ] Puedo ver el Informe Técnico completo de cada caso en el portal
- [ ] Puedo emitir mi voto (Favorable / Desfavorable / Abstención) con mi PIN de firma digital
- [ ] Una vez emitido, mi voto no puede ser modificado
- [ ] Puedo ver el resultado acumulado de la votación en tiempo real
- [ ] El Acta se genera automáticamente con todos los votos y el resultado
- [ ] Puedo firmar digitalmente el Acta con mi PIN

### HU-06: Director firma la Resolución

**Como** Director General de Industrias,  
**quiero** revisar y firmar digitalmente la Resolución del CFI,  
**para** darle validez legal al certificado.

**Criterios de aceptación:**
- [ ] Recibo una notificación cuando el Asesor Legal ha dado viabilidad
- [ ] Puedo ver la vista previa completa de la Resolución antes de firmar
- [ ] La Resolución incluye: número correlativo, fecha, empresa, monto aprobado, base legal
- [ ] Puedo firmar digitalmente con mi PIN de firma PKI
- [ ] Al firmar, el sistema genera automáticamente el CFI digital con código QR verificable
- [ ] El ciudadano es notificado por email y SMS en menos de 5 minutos
- [ ] El CFI digital queda disponible para descarga en el portal ciudadano

### HU-07: Ciudadano descarga el CFI digital

**Como** representante legal de la empresa,  
**quiero** descargar mi CFI digital,  
**para** utilizarlo ante las entidades del Estado.

**Criterios de aceptación:**
- [ ] Recibo una notificación por email y SMS cuando el CFI es emitido
- [ ] Puedo descargar el CFI en PDF desde el portal ciudadano
- [ ] El CFI incluye: datos de la empresa, monto aprobado, número de Resolución, firma digital del Director, código QR de verificación
- [ ] Cualquier persona puede verificar la autenticidad del CFI escaneando el código QR
- [ ] El CFI tiene validez legal equivalente al documento en papel sello frío

---

## 14. Plan de fases de implementación

### Fase 1 — Portal ciudadano (Mes 1–3)

**Entregables:**
- Sistema de autenticación (registro, login, recuperación de contraseña)
- Formulario de solicitud completo (Secciones I–VIII)
- Módulo de carga de documentos con validación
- Tabla de inversiones con cálculo automático del CFI
- Generación de número de expediente
- Notificaciones por email (confirmación de recepción)
- Vista de estado del expediente para el ciudadano

**Criterio de completación:** Un ciudadano puede completar y enviar una solicitud completa sin contactar al MICI.

### Fase 2 — Panel interno MICI (Mes 2–6)

**Entregables:**
- Sistema de roles y permisos (6 roles internos)
- Bandeja de trabajo filtrada por rol
- Módulo de Secretaria CFI (validación de docs + asignación)
- Módulo del Analista (agenda de visita + informes)
- Módulo del CPA (revisión de inversiones con C/NC)
- Módulo del Jefe de Departamento (aprobación de informes)
- Módulo del Asesor Legal (viabilidad jurídica)
- Módulo del Director DGI (firma de Resolución)
- Motor de flujo de estados con historial completo
- Registro de auditoría completo

**Criterio de completación:** Un expediente puede pasar de RECIBIDO a EN_CONAPI sin ningún intercambio de papel.

### Fase 3 — Portal CONAPI (Mes 5–7)

**Entregables:**
- Portal independiente para miembros del CONAPI
- Módulo de agenda de sesiones
- Sala de votación digital con firma por PIN
- Generación automática del Acta
- Firma digital del Acta por todos los miembros
- Integración del resultado de votación con el panel MICI

**Criterio de completación:** El CONAPI puede sesionar, votar y generar el Acta sin usar correo electrónico informal.

### Fase 4 — Resolución y CFI digital (Mes 7–9)

**Entregables:**
- Generación automática del borrador de Resolución
- Firma digital del Director con PKI
- Generación del CFI digital con código QR
- Endpoint público de verificación de autenticidad
- Notificación al ciudadano y descarga del CFI
- Integración con integraciones externas (Registro Público, DGI, CSS)

**Criterio de completación:** El proceso completo puede ejecutarse de punta a punta sin papel, desde la solicitud ciudadana hasta el CFI digital verificable.

---

## 15. Requisitos no funcionales

### 15.1 Rendimiento

| Métrica | Objetivo |
|---|---|
| Tiempo de respuesta API (p95) | < 500 ms |
| Tiempo de carga inicial portal | < 3 segundos |
| Carga de documentos PDF (10 MB) | < 10 segundos |
| Disponibilidad del sistema | 99.5% mensual |
| Tiempo de recuperación ante falla | < 4 horas |

### 15.2 Escalabilidad

- El sistema debe soportar 500 usuarios concurrentes en horario pico
- El Object Storage debe soportar hasta 1 TB de documentos por año
- La base de datos debe mantener su rendimiento con hasta 10,000 expedientes anuales

### 15.3 Compatibilidad

- Navegadores: Chrome 100+, Firefox 100+, Safari 15+, Edge 100+
- Dispositivos: Desktop (prioridad), Tablet, Mobile (portal ciudadano)
- No requiere instalación de plugins o software adicional

### 15.4 Accesibilidad

- Cumplimiento WCAG 2.1 nivel AA
- Contraste de color mínimo 4.5:1 en texto normal
- Navegación completa por teclado
- Compatibilidad con lectores de pantalla

### 15.5 Respaldo y recuperación

- Backup automático de PostgreSQL: diario completo + WAL continuo
- Backup de Object Storage: replicación en zona geográfica distinta
- RPO (Recovery Point Objective): máximo 1 hora
- RTO (Recovery Time Objective): máximo 4 horas

---

## 16. Glosario

| Término | Definición |
|---|---|
| **CFI** | Certificado de Fomento Industrial. Crédito fiscal no transferible equivalente al 40% de la inversión calificada. |
| **MICI** | Ministerio de Comercio e Industrias de la República de Panamá. |
| **DGI** | Dirección General de Industrias, unidad del MICI responsable del CFI. |
| **CONAPI** | Consejo Nacional de Política Industrial. Órgano colegiado que aprueba o deniega los CFI. |
| **CPA** | Contador Público Autorizado. Profesional que certifica el Detalle de Inversiones y emite el informe contable. |
| **PKI** | Public Key Infrastructure. Infraestructura de clave pública para firmas electrónicas. |
| **Expediente** | Conjunto de documentos, formularios y actuaciones que conforman una solicitud de CFI. |
| **Modalidad** | Tipo de actividad que da derecho al CFI (Inversión, Capacitación, I+D, etc.). |
| **Informe Técnico** | Documento elaborado por el analista que sustenta el concepto del MICI ante el CONAPI. |
| **Resolución** | Acto administrativo firmado por el Director DGI que otorga formalmente el CFI. |
| **Quórum** | Número mínimo de miembros del CONAPI requeridos para sesionar válidamente (4 de 6). |
| **Folio** | Número de página asignado a cada documento del expediente físico. En el sistema digital es un identificador secuencial. |
| **C / NC** | Conforme / No Conforme. Calificación del CPA para cada ítem del Detalle de Inversiones. |
| **SHA-256** | Algoritmo de hash utilizado para verificar la integridad de los documentos cargados. |
| **SSE-S3** | Server-Side Encryption. Cifrado en reposo aplicado a los documentos en Object Storage. |
| **JWT** | JSON Web Token. Estándar de autenticación sin estado utilizado en la API. |
| **RBAC** | Role-Based Access Control. Control de acceso basado en roles de usuario. |
| **WAL** | Write-Ahead Log. Registro de transacciones de PostgreSQL usado para respaldos continuos. |
| **RPO** | Recovery Point Objective. Máxima pérdida de datos aceptable ante una falla. |
| **RTO** | Recovery Time Objective. Tiempo máximo aceptable para restaurar el sistema. |

---

*Fin del documento — Versión 1.0 — Junio 2025*  
*Ministerio de Comercio e Industrias — Dirección General de Industrias*  
*República de Panamá*
