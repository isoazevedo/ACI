#  Asterisk REST API - Documentação Completa


## Índice

1. [Visão Geral](#visão-geral)
2. [Autenticação](#autenticação)
3. [Estrutura de Resposta](#estrutura-de-resposta)
4. [System](#system)
5. [Companies (Empresas)](#companies-empresas)
6. [DIDs (Mapeamento de Entrada)](#dids-mapeamento-de-entrada)
7. [DID Schedules (Horarios Comerciais)](#did-schedules-horarios-comerciais)
8. [Extensions (Ramais)](#extensions-ramais)
8. [Trunks (Troncos)](#trunks-troncos)
9. [Queues (Filas)](#queues-filas)
10. [Queue Members](#queue-members)
11. [IVRs (URAs)](#ivrs-uras)
12. [IVR Options (Opções)](#ivr-options-opções)
13. [Códigos de Erro](#códigos-de-erro)
14. [Exemplos Completos](#exemplos-completos)

---

## Visão Geral

API REST para gerenciamento completo de PBX Asterisk multi-tenant com suporte a:

- ✅ **Multi-empresa**: Isolamento completo por company_id
- ✅ **Ramais PJSIP**: Gerenciamento completo de extensões
- ✅ **Troncos**: SIP registration e static
- ✅ **Filas**: Criação e gerenciamento de queues
- ✅ **DIDs**: Roteamento de chamadas de entrada
- ✅ **IVRs (URAs)**: Menus interativos com DTMF e sub-menus
- ✅ **AGI Integration**: Roteamento automático via AGI

**Base URL**: `http://seu-servidor`

**Versão**: 1.1.0

---

## Autenticação

Todos os endpoints (exceto `/health`) requerem autenticação via Bearer Token.

### Header de Autenticação

```http
Authorization: Bearer seu_token_aqui
```

### Company ID

Endpoints de recursos (extensions, trunks, queues, dids) requerem identificação da empresa:

**Opção 1: Header (recomendado)**
```http
X-Company-ID: empresa1
```

**Opção 2: Query Parameter**
```http
GET /extensions?company_id=empresa1
```

### Exemplo Completo

```bash
curl -X GET "http://localhost/extensions" \
  -H "Authorization: Bearer meu_token_secreto" \
  -H "X-Company-ID: empresa1"
```

---

## Estrutura de Resposta

### Sucesso

```json
{
  "success": true,
  "data": {
    "id": 1,
    "extension": "1000",
    "caller_id_name": "João Silva"
  },
  "timestamp": "2026-01-14 10:30:00"
}
```

### Erro

```json
{
  "success": false,
  "error": "Extension not found",
  "timestamp": "2026-01-14 10:30:00"
}
```

### Lista

```json
{
  "success": true,
  "data": [
    {"id": 1, "extension": "1000"},
    {"id": 2, "extension": "1001"}
  ],
  "timestamp": "2026-01-14 10:30:00"
}
```

---

## System

### Health Check

Verifica status da API e banco de dados.

**Endpoint**: `GET /health`

**Autenticação**: ❌ Não requer

```bash
curl -X GET "http://localhost/health"
```

**Resposta**:
```json
{
  "success": true,
  "data": {
    "status": "healthy",
    "database": "connected",
    "timestamp": "2026-01-14 10:30:00",
    "version": "1.1.0"
  }
}
```

---

## Companies (Empresas)

Gerenciamento de empresas (multi-tenant).

### Listar Empresas

**Endpoint**: `GET /companies`

```bash
curl -X GET "http://localhost/companies" \
  -H "Authorization: Bearer token"
```

**Resposta**:
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "company_id": "empresa1",
      "name": "Empresa 1 LTDA",
      "active": 1,
      "created_at": "2026-01-14 10:00:00"
    }
  ]
}
```

---

### Listar com Estatísticas

**Endpoint**: `GET /companies/with-stats`

```bash
curl -X GET "http://localhost/companies/with-stats" \
  -H "Authorization: Bearer token"
```

**Resposta**:
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "company_id": "empresa1",
      "name": "Empresa 1 LTDA",
      "total_extensions": 10,
      "total_trunks": 2,
      "total_queues": 3
    }
  ]
}
```

---

### Obter Empresa

**Endpoint**: `GET /companies/{id}`

```bash
curl -X GET "http://localhost/companies/1" \
  -H "Authorization: Bearer token"
```

---

### Obter Estatísticas da Empresa

**Endpoint**: `GET /companies/{id}/stats`

```bash
curl -X GET "http://localhost/companies/1/stats" \
  -H "Authorization: Bearer token"
```

**Resposta**:
```json
{
  "success": true,
  "data": {
    "company_id": "empresa1",
    "extensions": 10,
    "trunks": 2,
    "queues": 3,
    "queue_members": 15,
    "dids": 5
  }
}
```

---

### Criar Empresa

**Endpoint**: `POST /companies`

**Campos obrigatórios:**
- `company_id` (string, 3-50 chars, alfanumérico + _ -)
- `name` (string)

**Campos opcionais:**
- `contact_email` (email)
- `contact_phone` (string)
- `address` (string)

```bash
curl -X POST "http://localhost/companies" \
  -H "Authorization: Bearer token" \
  -H "Content-Type: application/json" \
  -d '{
    "company_id": "acme",
    "name": "ACME Corporation",
    "contact_email": "contato@acme.com.br",
    "contact_phone": "61999887766"
  }'
```

**Resposta**:
```json
{
  "success": true,
  "data": {
    "id": 2,
    "company_id": "acme",
    "name": "ACME Corporation",
    "created_at": "2026-01-14 11:00:00"
  }
}
```

---

### Validar Company ID

Valida se um company_id está disponível antes de criar.

**Endpoint**: `POST /companies/validate`

```bash
curl -X POST "http://localhost/companies/validate" \
  -H "Authorization: Bearer token" \
  -H "Content-Type: application/json" \
  -d '{
    "company_id": "acme"
  }'
```

**Resposta (disponível)**:
```json
{
  "success": true,
  "data": {
    "available": true,
    "company_id": "acme"
  }
}
```

**Resposta (indisponível)**:
```json
{
  "success": true,
  "data": {
    "available": false,
    "company_id": "acme",
    "reason": "Company ID already exists"
  }
}
```

---

### Atualizar Empresa

**Endpoint**: `PUT /companies/{id}`

**Campos editáveis:**
- `name`
- `contact_email`
- `contact_phone`
- `address`

**⚠️ Nota**: `company_id` não pode ser alterado.

```bash
curl -X PUT "http://localhost/companies/1" \
  -H "Authorization: Bearer token" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Empresa 1 - Atualizada",
    "contact_phone": "61988776655"
  }'
```

---

### Ativar Empresa

**Endpoint**: `POST /companies/{id}/activate`

```bash
curl -X POST "http://localhost/companies/1/activate" \
  -H "Authorization: Bearer token"
```

---

### Desativar Empresa

**Endpoint**: `POST /companies/{id}/deactivate`

```bash
curl -X POST "http://localhost/companies/1/deactivate" \
  -H "Authorization: Bearer token"
```

---

### Deletar Empresa

**Endpoint**: `DELETE /companies/{id}`

**⚠️ Nota**: Só permite deletar se não houver recursos (ramais, troncos, filas).

```bash
curl -X DELETE "http://localhost/companies/1" \
  -H "Authorization: Bearer token"
```

---

### Deletar Empresa (Forçado)

Deleta empresa e todos os recursos vinculados.

**Endpoint**: `DELETE /companies/{id}/force-delete`

**⚠️ CUIDADO**: Ação irreversível!

```bash
curl -X DELETE "http://localhost/companies/1/force-delete" \
  -H "Authorization: Bearer token"
```

---

## DIDs (Mapeamento de Entrada)

Gerenciamento de números DID e roteamento de chamadas de entrada.

### Tipos de Destino

| Tipo | Descrição |
|------|-----------|
| `extension` | Ramal direto |
| `queue` | Fila de atendimento |
| `ivr` | Menu IVR/URA |
| `voicemail` | Correio de voz |
| `conference` | Sala de conferência |

---

### Listar DIDs

**Endpoint**: `GET /dids`

```bash
curl -X GET "http://localhost/dids" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1"
```

**Resposta**:
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "did_number": "06132334455",
      "description": "Recepção Principal",
      "destination_type": "extension",
      "destination": "1000",
      "timeout": 30,
      "active": 1
    }
  ]
}
```

---

### Listar DIDs com Detalhes

Inclui nome do ramal/fila de destino.

**Endpoint**: `GET /dids/with-details`

```bash
curl -X GET "http://localhost/dids/with-details" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1"
```

**Resposta**:
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "did_number": "06132334455",
      "destination_type": "extension",
      "destination": "1000",
      "destination_label": "João Silva"
    }
  ]
}
```

---

### Obter DID

**Endpoint**: `GET /dids/{id}`

```bash
curl -X GET "http://localhost/dids/1" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1"
```

---

### Estatísticas de DIDs

**Endpoint**: `GET /dids/stats`

```bash
curl -X GET "http://localhost/dids/stats" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1"
```

**Resposta**:
```json
{
  "success": true,
  "data": [
    {
      "destination_type": "extension",
      "total_dids": 3,
      "dids": "06132334455, 06132334456, 06132334457"
    }
  ]
}
```

---

### Buscar DIDs por Tipo

**Endpoint**: `GET /dids/by-type?type={type}`

**Tipos**: `extension`, `queue`, `ivr`, `voicemail`, `conference`

```bash
curl -X GET "http://localhost/dids/by-type?type=extension" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1"
```

---

### Criar DID

**Endpoint**: `POST /dids`

**Campos obrigatórios:**
- `did_number` (string, apenas números)
- `destination_type` (enum)
- `destination` (string)

**Campos opcionais:**
- `description` (string)
- `timeout` (int, padrão: 30)
- `business_hours_enabled` (boolean, padrão: 0)
- `business_hours_start` (time, padrão: 08:00:00)
- `business_hours_end` (time, padrão: 18:00:00)
- `business_days` (string, padrão: mon,tue,wed,thu,fri)
- `after_hours_destination_type` (enum)
- `after_hours_destination` (string)
- `caller_id_override` (string)

**Exemplo 1: DID para Ramal**
```bash
curl -X POST "http://localhost/dids" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1" \
  -H "Content-Type: application/json" \
  -d '{
    "did_number": "06132334455",
    "description": "Recepção Principal",
    "destination_type": "extension",
    "destination": "1000"
  }'
```

**Exemplo 2: DID para Fila com Horário**
```bash
curl -X POST "http://localhost/dids" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1" \
  -H "Content-Type: application/json" \
  -d '{
    "did_number": "06132334466",
    "description": "Suporte - Horário Comercial",
    "destination_type": "queue",
    "destination": "suporte",
    "business_hours_enabled": 1,
    "business_hours_start": "08:00:00",
    "business_hours_end": "18:00:00",
    "business_days": "mon,tue,wed,thu,fri",
    "after_hours_destination_type": "voicemail",
    "after_hours_destination": "suporte"
  }'
```

---

### Atualizar DID

**Endpoint**: `PUT /dids/{id}`

```bash
curl -X PUT "http://localhost/dids/1" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1" \
  -H "Content-Type: application/json" \
  -d '{
    "description": "Recepção - Atualizada",
    "destination": "2000"
  }'
```

---

### Clonar DID

Duplica configuração para um novo número DID.

**Endpoint**: `POST /dids/{id}/clone`

```bash
curl -X POST "http://localhost/dids/1/clone" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1" \
  -H "Content-Type: application/json" \
  -d '{
    "new_did_number": "06132334499"
  }'
```

---

### Testar Horário de Atendimento

**Endpoint**: `GET /dids/{id}/test-hours`

```bash
curl -X GET "http://localhost/dids/1/test-hours" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1"
```

**Resposta**:
```json
{
  "success": true,
  "data": {
    "is_business_hours": true,
    "current_time": "2026-01-14 14:30:00",
    "day_of_week": "Tuesday",
    "schedules_count": 3,
    "schedules": [
      {
        "id": 1,
        "label": "Dias uteis",
        "days": "mon,tue,wed,thu,fri",
        "time_start": "08:00:00",
        "time_end": "22:00:00",
        "priority": 10
      }
    ]
  }
}
```

---

### Deletar DID

**Endpoint**: `DELETE /dids/{id}`

```bash
curl -X DELETE "http://localhost/dids/1" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1"
```

---

## DID Schedules (Horarios Comerciais)

Gerenciamento de multiplos horarios comerciais por DID. Permite configurar faixas de horario diferentes para cada dia da semana.

> **Compatibilidade:** Se `business_hours_enabled = 1` e nenhum schedule existir, o sistema usa as colunas legadas (`business_hours_start`, `business_hours_end`, `business_days`).

### Listar Schedules

**Endpoint**: `GET /dids/{id}/schedules`

```bash
curl -X GET "http://localhost/dids/1/schedules" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1"
```

**Resposta**:
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "did_mapping_id": 1,
      "label": "Dias uteis",
      "days": "mon,tue,wed,thu,fri",
      "time_start": "08:00:00",
      "time_end": "22:00:00",
      "priority": 10,
      "active": 1
    },
    {
      "id": 2,
      "did_mapping_id": 1,
      "label": "Sabado",
      "days": "sat",
      "time_start": "08:00:00",
      "time_end": "18:00:00",
      "priority": 20,
      "active": 1
    },
    {
      "id": 3,
      "did_mapping_id": 1,
      "label": "Domingo",
      "days": "sun",
      "time_start": "08:00:00",
      "time_end": "12:00:00",
      "priority": 30,
      "active": 1
    }
  ]
}
```

---

### Obter Schedule

**Endpoint**: `GET /dids/{id}/schedules/{scheduleId}`

```bash
curl -X GET "http://localhost/dids/1/schedules/1" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1"
```

---

### Criar Schedule

**Endpoint**: `POST /dids/{id}/schedules`

**Campos obrigatorios:**
- `days` (string) - Dias separados por virgula: mon,tue,wed,thu,fri,sat,sun

**Campos opcionais:**
- `label` (string) - Nome/rotulo do horario
- `time_start` (time, padrao: 08:00:00)
- `time_end` (time, padrao: 18:00:00)
- `priority` (int, 0-100, padrao: 10) - Menor = maior prioridade

```bash
curl -X POST "http://localhost/dids/1/schedules" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1" \
  -H "Content-Type: application/json" \
  -d '{
    "label": "Dias uteis",
    "days": "mon,tue,wed,thu,fri",
    "time_start": "08:00:00",
    "time_end": "22:00:00",
    "priority": 10
  }'
```

---

### Atualizar Schedule

**Endpoint**: `PUT /dids/{id}/schedules/{scheduleId}`

```bash
curl -X PUT "http://localhost/dids/1/schedules/1" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1" \
  -H "Content-Type: application/json" \
  -d '{
    "time_end": "20:00:00"
  }'
```

---

### Deletar Schedule

**Endpoint**: `DELETE /dids/{id}/schedules/{scheduleId}`

```bash
curl -X DELETE "http://localhost/dids/1/schedules/1" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1"
```

---

### Deletar Todos os Schedules

**Endpoint**: `DELETE /dids/{id}/schedules`

```bash
curl -X DELETE "http://localhost/dids/1/schedules" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1"
```

---

## Extensions (Ramais)

Gerenciamento de ramais PJSIP com perfis otimizados.

### Perfis Disponíveis

| Perfil | Descrição | Uso |
|--------|-----------|-----|
| `standard` | Telefones IP tradicionais | Grandstream, Yealink, softphones |
| `webrtc` | Navegadores web | Chrome, Firefox, Safari |

---

### Listar Ramais

**Endpoint**: `GET /extensions`

```bash
curl -X GET "http://localhost/extensions" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1"
```

**Resposta**:
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "extension": "1000",
      "profile": "standard",
      "caller_id_name": "João Silva",
      "caller_id_num": "1000",
      "transport": "transport-udp",
      "webrtc": 0,
      "active": 1
    },
    {
      "id": 2,
      "extension": "2000",
      "profile": "webrtc",
      "caller_id_name": "Maria Santos",
      "caller_id_num": "2000",
      "transport": "transport-wss",
      "webrtc": 1,
      "active": 1
    }
  ]
}
```

---

### Obter Ramal

**Endpoint**: `GET /extension/{extension_number}`

```bash
curl -X GET "http://localhost/extension/1000" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1"
```

---

### Criar Ramal Standard

**Endpoint**: `POST /extension`

Perfil para **telefones IP tradicionais** (Grandstream, Yealink, softphones).

**Campos obrigatórios:**
- `extension` (string, numérico)
- `password` (string, mín. 6 chars)
- `profile` (enum: "standard")

**Campos opcionais:**
- `caller_id_name` (string)
- `caller_id_num` (string, padrão: extension)
- `transport` (string, padrão: transport-udp)
- `codecs` (array, padrão: ["ulaw", "alaw", "gsm", "g722"])
- `max_contacts` (int, 1-10, padrão: 2)
- `qualify_frequency` (int, 0-300, padrão: 60)

**Configurações automáticas para Standard:**
```json
{
  "transport": "transport-udp",
  "webrtc": 0,
  "media_encryption": "no",
  "ice_support": 0,
  "dtls_auto_generate_cert": 0,
  "rtcp_mux": 0,
  "use_avpf": 0
}
```

**Exemplo:**
```bash
curl -X POST "http://localhost/extension" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1" \
  -H "Content-Type: application/json" \
  -d '{
    "extension": "1000",
    "profile": "standard",
    "password": "senhaSegura123",
    "caller_id_name": "João Silva"
  }'
```

**Resposta:**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "extension": "1000",
    "profile": "standard",
    "caller_id_name": "João Silva",
    "transport": "transport-udp",
    "webrtc": 0,
    "created_at": "2026-01-15 10:00:00"
  }
}
```

---

### Criar Ramal WebRTC

**Endpoint**: `POST /extension`

Perfil para **navegadores web** (Chrome, Firefox, Safari).

**Campos obrigatórios:**
- `extension` (string, numérico)
- `password` (string, mín. 6 chars)
- `profile` (enum: "webrtc")

**Campos opcionais:**
- `caller_id_name` (string)
- `caller_id_num` (string, padrão: extension)
- `codecs` (array, padrão: ["opus", "ulaw", "alaw"])
- `max_contacts` (int, 1-10, padrão: 5)
- `qualify_frequency` (int, 0-300, padrão: 30)

**Configurações automáticas para WebRTC:**
```json
{
  "transport": "transport-wss",
  "webrtc": 1,
  "media_encryption": "dtls",
  "ice_support": 1,
  "dtls_auto_generate_cert": 1,
  "rtcp_mux": 1,
  "use_avpf": 1
}
```

**Exemplo:**
```bash
curl -X POST "http://localhost/extension" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1" \
  -H "Content-Type: application/json" \
  -d '{
    "extension": "2000",
    "profile": "webrtc",
    "password": "senhaWebRTC456",
    "caller_id_name": "Maria Santos"
  }'
```

**Resposta:**
```json
{
  "success": true,
  "data": {
    "id": 2,
    "extension": "2000",
    "profile": "webrtc",
    "caller_id_name": "Maria Santos",
    "transport": "transport-wss",
    "webrtc": 1,
    "media_encryption": "dtls",
    "created_at": "2026-01-15 10:05:00"
  }
}
```

---

### Personalizar Configurações

Você pode sobrescrever as configurações padrão de qualquer perfil:

**Standard Customizado:**
```bash
curl -X POST "http://localhost/extension" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1" \
  -H "Content-Type: application/json" \
  -d '{
    "extension": "3000",
    "profile": "standard",
    "password": "senha789",
    "caller_id_name": "Pedro Costa",
    "codecs": ["g722", "ulaw"],
    "max_contacts": 3,
    "qualify_frequency": 30
  }'
```

**WebRTC Customizado:**
```bash
curl -X POST "http://localhost/extension" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1" \
  -H "Content-Type: application/json" \
  -d '{
    "extension": "4000",
    "profile": "webrtc",
    "password": "webrtc999",
    "caller_id_name": "Ana Lima",
    "codecs": ["opus"],
    "max_contacts": 10
  }'
```

---

### Atualizar Ramal

**Endpoint**: `PUT /extension/{extension_number}`

**⚠️ Nota**: `extension` e `profile` não podem ser alterados.

```bash
curl -X PUT "http://localhost/extension/1000" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1" \
  -H "Content-Type: application/json" \
  -d '{
    "caller_id_name": "João Silva - Atualizado",
    "password": "novaSenha789"
  }'
```

---

### Deletar Ramal

**Endpoint**: `DELETE /extension/{extension_number}`

```bash
curl -X DELETE "http://localhost/extension/1000" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1"
```

---

### Comparação de Perfis

| Característica | Standard | WebRTC |
|----------------|----------|---------|
| **Uso** | Telefones IP físicos | Navegadores web |
| **Transport** | UDP (porta 5060) | WebSocket WSS (porta 8089) |
| **Codecs** | ulaw, alaw, gsm, g722 | opus, ulaw, alaw |
| **Encryption** | Não (opcional) | DTLS obrigatório |
| **ICE/STUN** | Não necessário | Obrigatório |
| **NAT** | Simples | Complexo (ICE) |
| **Max Contacts** | 2 | 5 |
| **Qualify** | 60s | 30s |
| **Casos de Uso** | Escritórios, telefones fixos | Home office, atendimento remoto |

---

## Trunks (Troncos)

Gerenciamento de troncos SIP.

### Tipos de Tronco

| Tipo | Descrição |
|------|----------|
| `registration` | Registra no servidor do operador |
| `static` | Recebe chamadas sem registro |

---

### Listar Troncos

**Endpoint**: `GET /trunks`

```bash
curl -X GET "http://localhost/trunks" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1"
```

**Resposta**:
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "trunk_name": "trunk_operadora",
      "trunk_type": "registration",
      "host": "sip.operadora.com.br",
      "username": "user123",
      "active": 1
    }
  ]
}
```

---

### Obter Tronco

**Endpoint**: `GET /trunk/{trunk_name}`

```bash
curl -X GET "http://localhost/trunk/trunk_operadora" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1"
```

---

### Listar por Tipo

**Endpoint**: `GET /trunks/type/{type}`

**Tipos**: `registration`, `static`

```bash
curl -X GET "http://localhost/trunks/type/registration" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1"
```

---

### Criar Tronco Registration

**Endpoint**: `POST /trunk`

**Campos obrigatórios:**
- `trunk_name` (string, único)
- `trunk_type` (enum: registration ou static)
- `host` (string)

**Para registration, também obrigatório:**
- `username` (string)
- `password` (string)

**Campos opcionais:**
- `server_uri` (string)
- `client_uri` (string)
- `contact_user` (string)
- `retry_interval` (int, 10-3600, padrão: 60)
- `max_retries` (int, 0-100, padrão: 10)
- `codecs` (array, padrão: ["ulaw", "alaw", "gsm"])
- `context` (string, padrão: from-trunk)

```bash
curl -X POST "http://localhost/trunk" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1" \
  -H "Content-Type: application/json" \
  -d '{
    "trunk_name": "trunk_operadora",
    "trunk_type": "registration",
    "host": "sip.operadora.com.br",
    "username": "user123",
    "password": "senha456",
    "server_uri": "sip:sip.operadora.com.br",
    "client_uri": "sip:user123@sip.operadora.com.br"
  }'
```

---

### Criar Tronco Static

```bash
curl -X POST "http://localhost/trunk" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1" \
  -H "Content-Type: application/json" \
  -d '{
    "trunk_name": "trunk_static",
    "trunk_type": "static",
    "host": "10.0.0.100",
    "context": "from-trunk"
  }'
```

---

### Atualizar Tronco

**Endpoint**: `PUT /trunk/{trunk_name}`

**⚠️ Nota**: `trunk_name` não pode ser alterado.

```bash
curl -X PUT "http://localhost/trunk/trunk_operadora" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1" \
  -H "Content-Type: application/json" \
  -d '{
    "password": "novaSenha789",
    "retry_interval": 120
  }'
```

---

### Deletar Tronco

**Endpoint**: `DELETE /trunk/{trunk_name}`

```bash
curl -X DELETE "http://localhost/trunk/trunk_operadora" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1"
```

---

## Queues (Filas)

Gerenciamento de filas de atendimento.

### Estratégias de Distribuição

| Estratégia | Descrição |
|------------|-----------|
| `ringall` | Toca em todos os ramais |
| `leastrecent` | Ramal que atendeu há mais tempo |
| `fewestcalls` | Ramal com menos chamadas |
| `random` | Aleatório |
| `rrmemory` | Round-robin com memória |

---

### Listar Filas

**Endpoint**: `GET /queues`

```bash
curl -X GET "http://localhost/queues" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1"
```

**Resposta**:
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "queue_name": "suporte",
      "strategy": "ringall",
      "timeout": 30,
      "retry": 5,
      "member_count": 3,
      "active": 1
    }
  ]
}
```

---

### Obter Fila

**Endpoint**: `GET /queue/{queue_name}`

```bash
curl -X GET "http://localhost/queue/suporte" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1"
```

---

### Criar Fila

**Endpoint**: `POST /queue`

**Campos obrigatórios:**
- `queue_name` (string, único)

**Campos opcionais:**
- `strategy` (enum, padrão: ringall)
- `timeout` (int, 1-300, padrão: 30)
- `retry` (int, 0-60, padrão: 5)
- `wrapuptime` (int, 0-600, padrão: 0)
- `maxlen` (int, 0-999, padrão: 0)
- `announce_frequency` (int, padrão: 0)
- `announce_holdtime` (enum: yes/no, padrão: no)
- `musicclass` (string, padrão: default)
- `context` (string)

```bash
curl -X POST "http://localhost/queue" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1" \
  -H "Content-Type: application/json" \
  -d '{
    "queue_name": "suporte",
    "strategy": "ringall",
    "timeout": 30,
    "retry": 5,
    "wrapuptime": 10,
    "announce_holdtime": "yes"
  }'
```

---

### Atualizar Fila

**Endpoint**: `PUT /queue/{queue_name}`

```bash
curl -X PUT "http://localhost/queue/suporte" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1" \
  -H "Content-Type: application/json" \
  -d '{
    "strategy": "leastrecent",
    "timeout": 45
  }'
```

---

### Deletar Fila

**Endpoint**: `DELETE /queue/{queue_name}`

```bash
curl -X DELETE "http://localhost/queue/suporte" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1"
```

---

## Queue Members

Gerenciamento de membros (ramais) nas filas.

### Listar Membros

**Endpoint**: `GET /queue/{queue_name}/members`

```bash
curl -X GET "http://localhost/queue/suporte/members" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1"
```

**Resposta**:
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "extension": "1000",
      "member_name": "João Silva",
      "penalty": 0,
      "paused": 0
    }
  ]
}
```

---

### Adicionar Membro

**Endpoint**: `POST /queue/{queue_name}/member`

**Campos obrigatórios:**
- `extension` (string)

**Campos opcionais:**
- `penalty` (int, padrão: 0)
- `member_name` (string)

```bash
curl -X POST "http://localhost/queue/suporte/member" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1" \
  -H "Content-Type: application/json" \
  -d '{
    "extension": "1000",
    "penalty": 0
  }'
```

---

### Remover Membro

**Endpoint**: `DELETE /queue/{queue_name}/member/{extension}`

```bash
curl -X DELETE "http://localhost/queue/suporte/member/1000" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1"
```

---

### Pausar Membro

**Endpoint**: `POST /queue/{queue_name}/member/{extension}/pause`

**Campos opcionais:**
- `reason` (string) - Motivo da pausa

```bash
curl -X POST "http://localhost/queue/suporte/member/1000/pause" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1" \
  -H "Content-Type: application/json" \
  -d '{
    "reason": "Almoço"
  }'
```

---

### Despausar Membro

**Endpoint**: `POST /queue/{queue_name}/member/{extension}/unpause`

```bash
curl -X POST "http://localhost/queue/suporte/member/1000/unpause" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1"
```

---

## IVRs (URAs)

Gerenciamento de IVRs (Interactive Voice Response) - Unidades de Resposta Audível.

Uma IVR é um menu interativo que permite aos chamadores navegar por opções usando o teclado do telefone (DTMF).

### Fluxo de Funcionamento

```
1. Chamada entra no DID mapeado para IVR
2. Áudio de boas-vindas é reproduzido
3. Sistema aguarda dígito DTMF (timeout configurável)
4. Dígito é processado e chamada roteada para destino
5. Se inválido/timeout, comportamento configurável (repetir, hangup, outro destino)
```

### Tipos de Ação (action_type)

| Tipo | Descrição |
|------|-----------|
| `queue` | Direciona para fila de atendimento |
| `extension` | Direciona para ramal direto |
| `ivr` | Direciona para outra IVR (sub-menu) |
| `hangup` | Desliga a chamada |

### Tipos de Destino para Timeout/Invalid

| Tipo | Descrição |
|------|-----------|
| `hangup` | Desliga a chamada |
| `repeat` | Repete o menu (apenas para invalid) |
| `queue` | Direciona para fila |
| `extension` | Direciona para ramal |
| `ivr` | Direciona para outra IVR |

---

### Listar IVRs

**Endpoint**: `GET /ivrs`

```bash
curl -X GET "http://localhost/ivrs" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1"
```

**Resposta**:
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "company_id": "empresa1",
      "ivr_name": "menu_principal",
      "description": "Menu principal de atendimento",
      "welcome_audio_url": "http://example.com/audios/bemvindo.wav",
      "timeout": 5,
      "max_retries": 3,
      "timeout_destination_type": "queue",
      "timeout_destination": "suporte",
      "invalid_destination_type": "repeat",
      "active": 1,
      "created_at": "2026-02-04 10:00:00"
    }
  ]
}
```

---

### Obter IVR

**Endpoint**: `GET /ivrs/{id}`

Retorna IVR com todas as opções configuradas.

```bash
curl -X GET "http://localhost/ivrs/1" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1"
```

**Resposta**:
```json
{
  "success": true,
  "data": {
    "id": 1,
    "company_id": "empresa1",
    "ivr_name": "menu_principal",
    "description": "Menu principal de atendimento",
    "welcome_audio_url": "http://example.com/audios/bemvindo.wav",
    "timeout": 5,
    "max_retries": 3,
    "timeout_destination_type": "queue",
    "timeout_destination": "suporte",
    "invalid_destination_type": "repeat",
    "invalid_destination": null,
    "active": 1,
    "options": [
      {
        "id": 1,
        "digit": "1",
        "action_type": "queue",
        "destination": "comercial",
        "audio_url": null,
        "description": "Setor Comercial"
      },
      {
        "id": 2,
        "digit": "2",
        "action_type": "queue",
        "destination": "suporte",
        "audio_url": null,
        "description": "Suporte Técnico"
      },
      {
        "id": 3,
        "digit": "0",
        "action_type": "extension",
        "destination": "1000",
        "audio_url": null,
        "description": "Falar com atendente"
      }
    ]
  }
}
```

---

### Criar IVR

**Endpoint**: `POST /ivrs`

**Campos obrigatórios:**
- `ivr_name` (string, alfanumérico + _ -)

**Campos opcionais:**
- `description` (string)
- `welcome_audio_url` (string, URL do áudio de boas-vindas)
- `timeout` (int, 1-60, padrão: 5) - segundos para aguardar DTMF
- `max_retries` (int, 1-10, padrão: 3) - tentativas antes de destino final
- `timeout_destination_type` (enum, padrão: hangup)
- `timeout_destination` (string)
- `invalid_destination_type` (enum, padrão: repeat)
- `invalid_destination` (string)
- `options` (array) - opções do menu

**Exemplo Completo:**
```bash
curl -X POST "http://localhost/ivrs" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1" \
  -H "Content-Type: application/json" \
  -d '{
    "ivr_name": "menu_principal",
    "description": "Menu principal de atendimento",
    "welcome_audio_url": "http://example.com/audios/bemvindo.wav",
    "timeout": 5,
    "max_retries": 3,
    "timeout_destination_type": "queue",
    "timeout_destination": "suporte",
    "invalid_destination_type": "repeat",
    "options": [
      {
        "digit": "1",
        "action_type": "queue",
        "destination": "comercial",
        "description": "Setor Comercial"
      },
      {
        "digit": "2",
        "action_type": "queue",
        "destination": "suporte",
        "description": "Suporte Técnico"
      },
      {
        "digit": "3",
        "action_type": "extension",
        "destination": "1000",
        "description": "Falar com atendente"
      },
      {
        "digit": "9",
        "action_type": "hangup",
        "description": "Encerrar chamada"
      }
    ]
  }'
```

**Exemplo Simples:**
```bash
curl -X POST "http://localhost/ivrs" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1" \
  -H "Content-Type: application/json" \
  -d '{
    "ivr_name": "menu_simples",
    "welcome_audio_url": "http://example.com/audios/menu.wav",
    "options": [
      {"digit": "1", "action_type": "queue", "destination": "vendas"},
      {"digit": "2", "action_type": "queue", "destination": "suporte"},
      {"digit": "9", "action_type": "hangup"}
    ]
  }'
```

---

### Atualizar IVR

**Endpoint**: `PUT /ivrs/{id}`

**⚠️ Nota**: `ivr_name` não pode ser alterado.

**Atualizar apenas configurações (sem alterar opções):**
```bash
curl -X PUT "http://localhost/ivrs/1" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1" \
  -H "Content-Type: application/json" \
  -d '{
    "description": "Menu atualizado",
    "timeout": 10,
    "max_retries": 5
  }'
```

**Atualizar substituindo todas as opções:**
```bash
curl -X PUT "http://localhost/ivrs/1" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1" \
  -H "Content-Type: application/json" \
  -d '{
    "description": "Menu com novas opções",
    "options": [
      {"digit": "1", "action_type": "queue", "destination": "vendas"},
      {"digit": "2", "action_type": "queue", "destination": "financeiro"},
      {"digit": "3", "action_type": "queue", "destination": "suporte"},
      {"digit": "0", "action_type": "extension", "destination": "1000"}
    ]
  }'
```

---

### Deletar IVR

**Endpoint**: `DELETE /ivrs/{id}`

```bash
curl -X DELETE "http://localhost/ivrs/1" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1"
```

---

## IVR Options (Opções)

Gerenciamento individual de opções (dígitos DTMF) de uma IVR.

### Dígitos Válidos

`0`, `1`, `2`, `3`, `4`, `5`, `6`, `7`, `8`, `9`, `*`, `#`

---

### Listar Opções da IVR

**Endpoint**: `GET /ivrs/{id}/options`

```bash
curl -X GET "http://localhost/ivrs/1/options" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1"
```

**Resposta**:
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "ivr_id": 1,
      "digit": "1",
      "action_type": "queue",
      "destination": "comercial",
      "audio_url": null,
      "description": "Setor Comercial",
      "active": 1
    },
    {
      "id": 2,
      "ivr_id": 1,
      "digit": "2",
      "action_type": "queue",
      "destination": "suporte",
      "audio_url": null,
      "description": "Suporte Técnico",
      "active": 1
    }
  ]
}
```

---

### Adicionar Opção

**Endpoint**: `POST /ivrs/{id}/options`

**Campos obrigatórios:**
- `digit` (char, 0-9, *, #)
- `action_type` (enum: queue, extension, ivr, hangup)
- `destination` (string, obrigatório exceto para hangup)

**Campos opcionais:**
- `description` (string)
- `audio_url` (string, áudio a reproduzir antes de rotear)

```bash
curl -X POST "http://localhost/ivrs/1/options" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1" \
  -H "Content-Type: application/json" \
  -d '{
    "digit": "5",
    "action_type": "queue",
    "destination": "financeiro",
    "description": "Setor Financeiro",
    "audio_url": "http://example.com/audios/financeiro.wav"
  }'
```

---

### Obter Opção por Dígito

**Endpoint**: `GET /ivrs/{id}/{digit}`

```bash
curl -X GET "http://localhost/ivrs/1/1" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1"
```

**Resposta**:
```json
{
  "success": true,
  "data": {
    "id": 1,
    "ivr_id": 1,
    "digit": "1",
    "action_type": "queue",
    "destination": "comercial",
    "audio_url": null,
    "description": "Setor Comercial",
    "active": 1
  }
}
```

---

### Atualizar Opção

**Endpoint**: `PUT /ivrs/{id}/{digit}`

```bash
curl -X PUT "http://localhost/ivrs/1/1" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1" \
  -H "Content-Type: application/json" \
  -d '{
    "action_type": "extension",
    "destination": "1001",
    "description": "Atendente VIP"
  }'
```

---

### Remover Opção

**Endpoint**: `DELETE /ivrs/{id}/{digit}`

```bash
curl -X DELETE "http://localhost/ivrs/1/5" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1"
```

---

### Remover Todas as Opções

**Endpoint**: `DELETE /ivrs/{id}/options`

```bash
curl -X DELETE "http://localhost/ivrs/1/options" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1"
```

---

### Mapear DID para IVR

Para que chamadas sejam direcionadas para uma IVR, é necessário mapear um DID.

**Endpoint**: `POST /dids`

```bash
curl -X POST "http://localhost/dids" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1" \
  -H "Content-Type: application/json" \
  -d '{
    "did_number": "06132334455",
    "destination_type": "ivr",
    "destination": "menu_principal",
    "description": "DID principal com IVR"
  }'
```

---

### Exemplo Completo: Setup de IVR com Sub-menus

```bash
# 1. Criar IVR principal
curl -X POST "http://localhost/ivrs" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1" \
  -H "Content-Type: application/json" \
  -d '{
    "ivr_name": "menu_principal",
    "welcome_audio_url": "http://example.com/audios/bemvindo.wav",
    "timeout": 5,
    "max_retries": 3,
    "timeout_destination_type": "queue",
    "timeout_destination": "recepcao",
    "options": [
      {"digit": "1", "action_type": "ivr", "destination": "menu_comercial", "description": "Comercial"},
      {"digit": "2", "action_type": "ivr", "destination": "menu_suporte", "description": "Suporte"},
      {"digit": "0", "action_type": "extension", "destination": "1000", "description": "Atendente"}
    ]
  }'

# 2. Criar sub-menu Comercial
curl -X POST "http://localhost/ivrs" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1" \
  -H "Content-Type: application/json" \
  -d '{
    "ivr_name": "menu_comercial",
    "welcome_audio_url": "http://example.com/audios/comercial.wav",
    "options": [
      {"digit": "1", "action_type": "queue", "destination": "vendas", "description": "Vendas"},
      {"digit": "2", "action_type": "queue", "destination": "orcamentos", "description": "Orçamentos"},
      {"digit": "9", "action_type": "ivr", "destination": "menu_principal", "description": "Voltar"}
    ]
  }'

# 3. Criar sub-menu Suporte
curl -X POST "http://localhost/ivrs" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1" \
  -H "Content-Type: application/json" \
  -d '{
    "ivr_name": "menu_suporte",
    "welcome_audio_url": "http://example.com/audios/suporte.wav",
    "options": [
      {"digit": "1", "action_type": "queue", "destination": "suporte_tecnico", "description": "Técnico"},
      {"digit": "2", "action_type": "queue", "destination": "suporte_financeiro", "description": "Financeiro"},
      {"digit": "9", "action_type": "ivr", "destination": "menu_principal", "description": "Voltar"}
    ]
  }'

# 4. Mapear DID para IVR principal
curl -X POST "http://localhost/dids" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1" \
  -H "Content-Type: application/json" \
  -d '{
    "did_number": "06132334455",
    "destination_type": "ivr",
    "destination": "menu_principal"
  }'
```

---

## Códigos de Erro

| Código | Descrição |
|--------|-----------|
| 200 | Sucesso |
| 400 | Requisição inválida |
| 401 | Não autorizado |
| 404 | Recurso não encontrado |
| 405 | Método não permitido |
| 409 | Conflito (duplicado) |
| 500 | Erro interno do servidor |

### Exemplos de Erros

**401 - Token Inválido**
```json
{
  "success": false,
  "error": "Invalid or missing token"
}
```

**404 - Não Encontrado**
```json
{
  "success": false,
  "error": "Extension not found"
}
```

**409 - Duplicado**
```json
{
  "success": false,
  "error": "Extension 1000 already exists"
}
```

---

## Exemplos Completos

### Exemplo 1: Setup Inicial de Empresa

```bash
# 1. Criar empresa
curl -X POST "http://localhost/companies" \
  -H "Authorization: Bearer token" \
  -d '{"company_id":"acme","name":"ACME Corp"}'

# 2. Criar ramal standard (telefone IP)
curl -X POST "http://localhost/extension" \
  -H "X-Company-ID: acme" \
  -d '{"extension":"1000","profile":"standard","password":"senha123","caller_id_name":"Recepção"}'

# 3. Criar ramal webrtc (navegador)
curl -X POST "http://localhost/extension" \
  -H "X-Company-ID: acme" \
  -d '{"extension":"2000","profile":"webrtc","password":"webrtc123","caller_id_name":"Atendente Remoto"}'

# 4. Criar tronco
curl -X POST "http://localhost/trunk" \
  -H "X-Company-ID: acme" \
  -d '{"trunk_name":"trunk_op","trunk_type":"registration","host":"sip.op.com.br","username":"user","password":"pass"}'

# 5. Criar DID
curl -X POST "http://localhost/dids" \
  -H "X-Company-ID: acme" \
  -d '{"did_number":"06132334455","destination_type":"extension","destination":"1000"}'
```

---

### Exemplo 2: Setup de Call Center

```bash
# 1. Criar fila
curl -X POST "http://localhost/queue" \
  -H "X-Company-ID: empresa1" \
  -d '{"queue_name":"vendas","strategy":"ringall"}'

# 2. Criar ramais standard (escritório)
curl -X POST "http://localhost/extension" \
  -H "X-Company-ID: empresa1" \
  -d '{"extension":"1001","profile":"standard","password":"senha1001","caller_id_name":"Vendedor 1"}'

curl -X POST "http://localhost/extension" \
  -H "X-Company-ID: empresa1" \
  -d '{"extension":"1002","profile":"standard","password":"senha1002","caller_id_name":"Vendedor 2"}'

# 3. Criar ramais webrtc (home office)
curl -X POST "http://localhost/extension" \
  -H "X-Company-ID: empresa1" \
  -d '{"extension":"2001","profile":"webrtc","password":"webrtc2001","caller_id_name":"Vendedor Remoto 1"}'

curl -X POST "http://localhost/extension" \
  -H "X-Company-ID: empresa1" \
  -d '{"extension":"2002","profile":"webrtc","password":"webrtc2002","caller_id_name":"Vendedor Remoto 2"}'

# 4. Adicionar à fila
curl -X POST "http://localhost/queue/vendas/member" \
  -H "X-Company-ID: empresa1" \
  -d '{"extension":"1001"}'

curl -X POST "http://localhost/queue/vendas/member" \
  -H "X-Company-ID: empresa1" \
  -d '{"extension":"1002"}'

curl -X POST "http://localhost/queue/vendas/member" \
  -H "X-Company-ID: empresa1" \
  -d '{"extension":"2001"}'

curl -X POST "http://localhost/queue/vendas/member" \
  -H "X-Company-ID: empresa1" \
  -d '{"extension":"2002"}'

# 5. Mapear DID para fila
curl -X POST "http://localhost/dids" \
  -H "X-Company-ID: empresa1" \
  -d '{"did_number":"06132334466","destination_type":"queue","destination":"vendas"}'
```

---

### Exemplo 3: Sistema Híbrido (Escritório + Home Office)

```bash
# Escritório - 10 ramais standard (1000-1009)
for i in {1000..1009}; do
  curl -X POST "http://localhost/extension" \
    -H "Authorization: Bearer token" \
    -H "X-Company-ID: empresa1" \
    -H "Content-Type: application/json" \
    -d "{
      \"extension\": \"$i\",
      \"profile\": \"standard\",
      \"password\": \"senha${i}\",
      \"caller_id_name\": \"Escritório ${i}\"
    }"
done

# Home Office - 10 ramais webrtc (2000-2009)
for i in {2000..2009}; do
  curl -X POST "http://localhost/extension" \
    -H "Authorization: Bearer token" \
    -H "X-Company-ID: empresa1" \
    -H "Content-Type: application/json" \
    -d "{
      \"extension\": \"$i\",
      \"profile\": \"webrtc\",
      \"password\": \"webrtc${i}\",
      \"caller_id_name\": \"Remoto ${i}\"
    }"
done

# Criar fila com todos
curl -X POST "http://localhost/queue" \
  -H "X-Company-ID: empresa1" \
  -d '{"queue_name":"atendimento","strategy":"leastrecent"}'

# Adicionar todos à fila (script)
for i in {1000..1009} {2000..2009}; do
  curl -X POST "http://localhost/queue/atendimento/member" \
    -H "X-Company-ID: empresa1" \
    -d "{\"extension\":\"$i\"}"
done
```

---

### Exemplo 4: Horário de Atendimento

```bash
# DID com horário comercial
curl -X POST "http://localhost/dids" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: empresa1" \
  -H "Content-Type: application/json" \
  -d '{
    "did_number": "06132334477",
    "destination_type": "queue",
    "destination": "suporte",
    "business_hours_enabled": 1,
    "business_hours_start": "08:00:00",
    "business_hours_end": "18:00:00",
    "business_days": "mon,tue,wed,thu,fri",
    "after_hours_destination_type": "voicemail",
    "after_hours_destination": "suporte"
  }'
```

---

### Exemplo 5: Setup Completo com IVR

```bash
# 1. Criar empresa
curl -X POST "http://localhost/companies" \
  -H "Authorization: Bearer token" \
  -H "Content-Type: application/json" \
  -d '{"company_id":"clinica","name":"Clinica Saúde Total"}'

# 2. Criar ramais
curl -X POST "http://localhost/extension" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: clinica" \
  -H "Content-Type: application/json" \
  -d '{"extension":"1000","profile":"standard","password":"senha1000","caller_id_name":"Recepção"}'

# 3. Criar filas
curl -X POST "http://localhost/queue" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: clinica" \
  -H "Content-Type: application/json" \
  -d '{"queue_name":"agendamentos","strategy":"ringall"}'

curl -X POST "http://localhost/queue" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: clinica" \
  -H "Content-Type: application/json" \
  -d '{"queue_name":"laboratorio","strategy":"rrmemory"}'

# 4. Criar IVR principal
curl -X POST "http://localhost/ivrs" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: clinica" \
  -H "Content-Type: application/json" \
  -d '{
    "ivr_name": "menu_clinica",
    "description": "Menu principal da clínica",
    "welcome_audio_url": "http://example.com/audios/clinica_bemvindo.wav",
    "timeout": 8,
    "max_retries": 3,
    "timeout_destination_type": "extension",
    "timeout_destination": "1000",
    "invalid_destination_type": "repeat",
    "options": [
      {"digit": "1", "action_type": "queue", "destination": "agendamentos", "description": "Agendar consulta"},
      {"digit": "2", "action_type": "queue", "destination": "laboratorio", "description": "Resultados de exames"},
      {"digit": "3", "action_type": "extension", "destination": "1000", "description": "Falar com recepção"},
      {"digit": "9", "action_type": "hangup", "description": "Encerrar"}
    ]
  }'

# 5. Criar tronco
curl -X POST "http://localhost/trunk" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: clinica" \
  -H "Content-Type: application/json" \
  -d '{
    "trunk_name": "trunk_clinica",
    "trunk_type": "registration",
    "host": "sip.operadora.com.br",
    "username": "clinica_user",
    "password": "clinica_pass"
  }'

# 6. Mapear DID para IVR (chamadas entram na URA)
curl -X POST "http://localhost/dids" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: clinica" \
  -H "Content-Type: application/json" \
  -d '{
    "did_number": "06133445566",
    "destination_type": "ivr",
    "destination": "menu_clinica",
    "description": "Telefone principal - IVR",
    "business_hours_enabled": 1,
    "business_hours_start": "07:00:00",
    "business_hours_end": "19:00:00",
    "business_days": "mon,tue,wed,thu,fri,sat",
    "after_hours_destination_type": "hangup"
  }'

# 7. Verificar tudo
curl -X GET "http://localhost/ivrs" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: clinica"

curl -X GET "http://localhost/dids/by-type?type=ivr" \
  -H "Authorization: Bearer token" \
  -H "X-Company-ID: clinica"
```

---


**Versão**: 1.5.0
**Última atualização**: 2026-02-05
**By: Israel Azevedo**
