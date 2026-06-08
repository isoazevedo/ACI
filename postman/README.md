# 📮 Postman Collection - Asterisk API

Collection completa para testar todos os endpoints da API Asterisk.

## 📦 Conteúdo

- **Asterisk-API-Collection.json** - Collection principal com todos os endpoints
- **Asterisk-API-Environment-Local.json** - Environment para desenvolvimento local
- **Asterisk-API-Environment-Production.json** - Environment para produção
- **GUIA-POSTMAN.md** - Guia completo de uso

## 🚀 Quick Start

1. **Importe a Collection:**
   - Abra Postman
   - Import → Selecione `Asterisk-API-Collection.json`

2. **Importe o Environment:**
   - Import → Selecione `Asterisk-API-Environment-Local.json`

3. **Configure as variáveis:**
   ```
   api_url    = http://localhost/api
   api_token  = seu_token_aqui
   company_id = empresa1 (opcional)
   ```

4. **Teste:**
   - Execute: System → Health Check
   - Se retornar 200 OK, está funcionando!

## 📁 Estrutura da Collection

```
✅ System (1 endpoint)
   - Health Check

✅ Extensions - Ramais (5 endpoints)
   - List, Get, Create, Update, Delete

✅ Trunks - Troncos (6 endpoints)
   - List, Get, Create (2 tipos), Update, Delete

✅ Queues - Filas (5 endpoints)
   - List, Get, Create, Update, Delete

✅ Queue Members - Membros (4 endpoints)
   - Add, Remove, Pause, Unpause

✅ Multi-Company Examples (4 exemplos)
   - Exemplos práticos de uso multi-empresa
```

**Total: 25 requisições prontas para uso!**

## 🏢 Multi-Empresa

Para usar com múltiplas empresas:

1. Ative o header `X-Company-ID` nas requisições
2. Ou configure `company_id` no Environment

Exemplo:
```
X-Company-ID: empresa1
```

## 💡 Recursos

- ✅ Todas as requisições documentadas
- ✅ Exemplos de body prontos
- ✅ Suporte multi-empresa
- ✅ Variáveis configuráveis
- ✅ Headers pré-configurados
- ✅ Environments separados (dev/prod)

## 📖 Documentação Completa

Consulte **GUIA-POSTMAN.md** para:
- Configuração detalhada
- Fluxos de teste
- Dicas avançadas
- Troubleshooting
- Exemplos de scripts

## 🎯 Endpoints Principais

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | /health | Health check |
| GET | /extensions | Lista ramais |
| POST | /extension | Cria ramal |
| GET | /trunks | Lista troncos |
| POST | /trunk | Cria tronco |
| GET | /queues | Lista filas |
| POST | /queue | Cria fila |
| POST | /queue/{id}/add-member | Adiciona à fila |
| POST | /queue/{id}/pause-member | Pausa membro |

## 🔐 Autenticação

Todas as requisições usam Bearer Token:
```
Authorization: Bearer seu_token_aqui
```

O token é configurado automaticamente via variável `{{api_token}}`.

## 📞 Suporte

- Problemas? Veja GUIA-POSTMAN.md
- Collection não funciona? Reimporte
- Erros 401? Verifique o token
- Erros 400? Confira o body JSON

---

Desenvolvido para Asterisk API v1.0
