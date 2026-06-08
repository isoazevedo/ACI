# 📮 GUIA POSTMAN - Asterisk API

## 📥 IMPORTAÇÃO

### 1. Importar Collection
1. Abra o Postman
2. Clique em **Import** (canto superior esquerdo)
3. Selecione o arquivo: `Asterisk-API-Collection.json`
4. Clique em **Import**

### 2. Importar Environment
1. Clique em **Import** novamente
2. Selecione: `Asterisk-API-Environment-Local.json`
3. (Opcional) Importe também: `Asterisk-API-Environment-Production.json`

## ⚙️ CONFIGURAÇÃO

### 1. Selecionar Environment
No canto superior direito do Postman:
- Clique no dropdown de Environments
- Selecione: **Asterisk API - Local**

### 2. Configurar Variáveis
Clique no ícone de olho 👁️ ao lado do Environment e configure:

```
api_url      = http://localhost/api
api_token    = seu_token_gerado_pela_api
company_id   = empresa1 (ou deixe vazio se não usar multi-empresa)
```

**Como obter o token:**
- Após instalar a API, verifique o arquivo: `/var/www/html/api/config/config.php`
- Ou veja o output do script `install.sh`

## 🚀 USO BÁSICO

### Testar Conexão
1. Abra a pasta **System**
2. Execute: **Health Check**
3. Você deve receber:
```json
{
  "success": true,
  "data": {
    "status": "ok",
    "timestamp": "2025-01-08 10:30:00"
  }
}
```

### Criar um Ramal
1. Abra a pasta **Extensions (Ramais)**
2. Selecione: **Create Extension WebRTC**
3. Clique em **Send**
4. Verifique a resposta

### Listar Ramais
1. Ainda em **Extensions (Ramais)**
2. Selecione: **List All Extensions**
3. Clique em **Send**

## 🏢 USO COM MULTI-EMPRESA

### Método 1: Usar Header X-Company-ID (Recomendado)

1. Em qualquer requisição, vá na aba **Headers**
2. Ative o header: `X-Company-ID`
3. Altere o valor conforme necessário:
   - `empresa1`
   - `empresa2`
   - etc.

### Método 2: Usar Variável de Environment

1. Configure `company_id` no Environment
2. O header `X-Company-ID` já está configurado para usar: `{{company_id}}`
3. Ative o header em cada requisição

### Método 3: Pasta Multi-Company Examples

A collection inclui uma pasta **Multi-Company Examples** com exemplos prontos:
- Create Extension - Company 1
- Create Extension - Company 2
- Create Queue - Company 1
- List Extensions - Company 1

## 📁 ESTRUTURA DA COLLECTION

```
Asterisk API - Complete Collection
│
├── 📁 System
│   └── Health Check
│
├── 📁 Extensions (Ramais)
│   ├── List All Extensions
│   ├── Get Extension
│   ├── Create Extension WebRTC
│   ├── Update Extension
│   └── Delete Extension
│
├── 📁 Trunks (Troncos)
│   ├── List All Trunks
│   ├── Get Trunk
│   ├── Create Trunk (Registration)
│   ├── Create Trunk (Static IP)
│   ├── Update Trunk
│   └── Delete Trunk
│
├── 📁 Queues (Filas)
│   ├── List All Queues
│   ├── Get Queue
│   ├── Create Queue
│   ├── Update Queue
│   └── Delete Queue
│
├── 📁 Queue Members (Membros)
│   ├── Add Member to Queue
│   ├── Remove Member from Queue
│   ├── Pause Member in Queue
│   └── Unpause Member in Queue
│
└── 📁 Multi-Company Examples
    ├── Create Extension - Company 1
    ├── Create Extension - Company 2
    ├── Create Queue - Company 1
    └── List Extensions - Company 1
```

## 🎯 FLUXO DE TESTE COMPLETO

### 1. Teste Básico
```
1. Health Check
2. List All Extensions (deve estar vazio)
3. Create Extension WebRTC
4. List All Extensions (deve mostrar o ramal criado)
5. Get Extension (detalhe do ramal)
```

### 2. Teste de Fila
```
1. Create Queue
2. List All Queues
3. Add Member to Queue
4. Get Queue (verifica membro adicionado)
5. Pause Member in Queue
6. Unpause Member in Queue
7. Remove Member from Queue
```

### 3. Teste Multi-Empresa
```
1. Ative X-Company-ID: empresa1
2. Create Extension (ramal 2000)
3. Ative X-Company-ID: empresa2
4. Create Extension (ramal 3000)
5. Ative X-Company-ID: empresa1
6. List Extensions (só mostra ramais da empresa1)
```

## 💡 DICAS

### 1. Duplicar Requisições
- Clique com botão direito em uma requisição
- **Duplicate**
- Modifique conforme necessário

### 2. Salvar Respostas
- Após executar uma requisição
- Clique em **Save Response**
- Útil para documentação

### 3. Usar Pre-request Scripts
Adicione no Pre-request Script da collection:
```javascript
// Gerar senha aleatória
pm.environment.set("random_password", Math.random().toString(36).slice(-8));

// Gerar número de ramal aleatório
pm.environment.set("random_extension", Math.floor(Math.random() * 9000) + 1000);
```

Depois use nas requisições:
```json
{
    "extension": "{{random_extension}}",
    "password": "{{random_password}}"
}
```

### 4. Usar Tests
Adicione na aba Tests de uma requisição:
```javascript
// Verifica se a resposta foi sucesso
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

pm.test("Response has success field", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.success).to.eql(true);
});

// Salva dados para próximas requisições
var jsonData = pm.response.json();
if (jsonData.data && jsonData.data.extension) {
    pm.environment.set("last_extension", jsonData.data.extension);
}
```

### 5. Coleções de Testes Automatizados
Use o **Collection Runner**:
1. Clique nos três pontos da collection
2. **Run collection**
3. Selecione as requisições desejadas
4. Clique em **Run Asterisk API**

## 🔒 SEGURANÇA

⚠️ **IMPORTANTE:**
- Nunca commite arquivos com tokens reais
- Use variáveis de ambiente para dados sensíveis
- No Postman, marque variáveis sensíveis como **secret**
- Em produção, sempre use HTTPS

## 🐛 TROUBLESHOOTING

### Erro 401 Unauthorized
- Verifique se o token está correto
- Certifique-se que está no formato: `Bearer seu_token`
- O header Authorization deve estar ativo

### Erro 403 Forbidden
- Seu IP não está na lista de permitidos
- Edite `config/config.php` e adicione seu IP
- Ou deixe o array vazio para permitir todos

### Erro 400 Bad Request
- No modo multi-empresa, verifique se está enviando `company_id`
- Verifique a sintaxe JSON do body
- Confira se todos os campos obrigatórios estão presentes

### Erro 404 Not Found
- Verifique se a URL da API está correta
- Certifique-se que o Apache está rodando
- Verifique se o mod_rewrite está ativo

### Erro 500 Internal Server Error
- Verifique os logs da API: `/var/log/asterisk-api/`
- Verifique os logs do Apache: `/var/log/apache2/error.log`
- Confira as permissões dos arquivos

## 📊 VARIÁVEIS DISPONÍVEIS

### Variables da Collection
```
{{api_url}}      - URL base da API
{{api_token}}    - Token de autenticação
{{company_id}}   - ID da empresa (multi-empresa)
```

### Como Usar
Nas requisições, você pode usar variáveis assim:
```
URL: {{api_url}}/extension/{{last_extension}}
Header: Authorization: Bearer {{api_token}}
Header: X-Company-ID: {{company_id}}
```

## 📝 EXEMPLOS DE BODY

### Criar Ramal Completo
```json
{
    "extension": "1000",
    "password": "senha_segura_123",
    "caller_id_name": "João Silva",
    "caller_id_num": "1000",
    "context": "default",
    "max_contacts": 2
}
```

### Criar Tronco com Registro
```json
{
    "trunk": "trunk_principal",
    "type": "registration",
    "host": "sip.provedor.com",
    "username": "usuario123",
    "password": "senha456",
    "context": "from-trunk"
}
```

### Criar Fila Completa
```json
{
    "queue": "suporte",
    "strategy": "rrmemory",
    "timeout": 30,
    "retry": 5,
    "wrapuptime": 15,
    "maxlen": 20,
    "announce_frequency": 60,
    "musicclass": "default",
    "members": ["1000", "1001", "1002"]
}
```

## 🎓 RECURSOS AVANÇADOS

### 1. Workflows
Encadeie requisições usando scripts:
```javascript
// Na aba Tests da requisição "Create Extension"
var jsonData = pm.response.json();
pm.environment.set("new_extension", jsonData.data.extension);

// Na próxima requisição "Add to Queue", use:
{
    "extension": "{{new_extension}}"
}
```

### 2. Data Files
Execute testes em massa:
1. Crie um CSV com dados:
```csv
extension,password,name
1000,pass1,João
1001,pass2,Maria
1002,pass3,Pedro
```

2. No Collection Runner:
   - Selecione o CSV
   - Execute "Create Extension" múltiplas vezes

### 3. Monitores
Configure monitores para verificar health da API:
1. Collection → três pontos → **Monitor collection**
2. Configure intervalo (ex: a cada 5 minutos)
3. Receba alertas se a API ficar offline

## 📞 SUPORTE

Problemas com a collection?
- Verifique se está usando Postman atualizado
- Reimporte a collection
- Confira as variáveis de environment
- Consulte os logs da API

---

**Collection Version:** 1.0
**Last Update:** Janeiro 2025
**Compatibility:** Postman v10+
