# Ferramentas do Supabase MCP Server

Este documento detalha todas as ferramentas disponíveis no Supabase MCP Server, organizadas por categoria.

## Ferramentas de Banco de Dados

### get_schemas
**Descrição:** Lista todos os schemas do banco de dados com seus tamanhos e contagem de tabelas  
**Uso:** Não requer parâmetros  
**Exemplo:**
```json
{
  "results": [{
    "rows": [
      {
        "schema_name": "public",
        "total_size": "32 kB",
        "table_count": 2
      }
    ]
  }]
}
```

### get_tables
**Descrição:** Lista todas as tabelas, tabelas estrangeiras e views em um schema com tamanhos, contagem de linhas e metadados  
**Parâmetros:** 
- schema_name (ex: 'public', 'auth')  

**Exemplo:**
```json
{
  "results": [{
    "rows": [
      {
        "table_name": "dv_restricao_user",
        "table_type": "BASE TABLE",
        "size_bytes": 16384,
        "row_count": 0,
        "column_count": 3
      }
    ]
  }]
}
```

### get_table_schema
**Descrição:** Obtém a estrutura detalhada da tabela, incluindo colunas, chaves e relacionamentos  
**Parâmetros:**
- schema_name (schema da tabela)
- table (nome da tabela)

**Exemplo:**
```json
{
  "results": [{
    "rows": [
      {
        "column_name": "id",
        "data_type": "bigint",
        "is_nullable": "NO",
        "is_primary_key": true
      }
    ]
  }]
}
```

### execute_postgresql
**Descrição:** Executa comandos SQL PostgreSQL contra o banco de dados Supabase  
**Parâmetros:**
- query (consulta SQL)
- migration_name (opcional - nome da migração)

**Exemplo:**
```sql
SELECT * FROM public.dv_restricao_user;
```

### retrieve_migrations
**Descrição:** Recupera histórico de migrações do banco de dados  
**Parâmetros:**
- limit (opcional - número máximo de migrações a retornar)

## Ferramentas de Segurança

### live_dangerously
**Descrição:** Habilita o modo não seguro para permitir operações de escrita ou potencialmente destrutivas  
**Parâmetros:**
- service ("api" ou "database")
- enable_unsafe_mode (booleano)

### confirm_destructive_operation
**Descrição:** Confirma uma operação destrutiva após receber um ID de confirmação  
**Parâmetros:**
- operation_type ("api" ou "database")
- confirmation_id
- user_confirmation (booleano)

## Ferramentas de API de Gerenciamento

### send_management_api_request
**Descrição:** Envia requisições para a API de gerenciamento do Supabase  
**Parâmetros:**
- method (método HTTP)
- path (caminho da API)
- path_params (parâmetros do caminho)
- request_params (parâmetros da requisição)
- request_body (corpo da requisição)

### get_management_api_spec
**Descrição:** Obtém a especificação da API de gerenciamento do Supabase  
**Parâmetros:**
- params (opcional - parâmetros adicionais)
  - domain (opcional - domínio específico da API)
  - path (opcional - caminho específico)
  - method (opcional - método HTTP específico)

## Ferramentas de Administração de Autenticação

### get_auth_admin_methods_spec
**Descrição:** Obtém a especificação dos métodos disponíveis no SDK de Admin de Autenticação  
**Uso:** Não requer parâmetros

### call_auth_admin_method
**Descrição:** Chama um método do SDK de Admin de Autenticação do Supabase  
**Parâmetros:**
- method (nome do método)
- params (parâmetros para o método)

## Ferramentas de Logs e Analytics

### retrieve_logs
**Descrição:** Recupera logs do sistema Supabase  
**Parâmetros:**
- collection (coleção de logs a recuperar)
- limit (número máximo de logs a retornar)
- hours_ago (recuperar logs das últimas N horas)
- filters (lista de filtros)
- search (texto para buscar nos logs)

## Coleções de Logs Disponíveis

- postgres: Logs do servidor de banco de dados
- api_gateway: Requisições da API gateway
- auth: Eventos de autenticação
- postgrest: Logs do serviço de API RESTful
- pooler: Logs de pooling de conexões
- storage: Operações de armazenamento
- realtime: Logs de subscrições WebSocket
- edge_functions: Execuções de funções serverless
- cron: Logs de jobs agendados
- pgbouncer: Logs do pooler de conexões

## Níveis de Risco das Operações

O Supabase MCP Server implementa um sistema de segurança com três níveis:

1. **Baixo Risco (Permitido por padrão)**
   - Operações de leitura (SELECT)
   - Requisições GET à API

2. **Médio Risco (Requer modo unsafe)**
   - Operações de escrita (INSERT, UPDATE, DELETE)
   - Maioria das requisições POST/PUT à API

3. **Alto Risco (Requer confirmação)**
   - Operações destrutivas (DROP, TRUNCATE)
   - Endpoints DELETE da API

4. **Risco Extremo (Bloqueado)**
   - Operações que podem causar perda de dados irreversível
   - Deleção de projetos

## Uso do Sistema de Segurança

1. Para operações de leitura:
   - Use as ferramentas normalmente, não é necessária configuração especial

2. Para operações de escrita:
   ```javascript
   // Primeiro, habilite o modo unsafe
   live_dangerously({
     service: "database",
     enable_unsafe_mode: true
   })
   
   // Agora você pode executar operações de escrita
   execute_postgresql({
     query: "INSERT INTO ..."
   })
   ```

3. Para operações destrutivas:
   ```javascript
   // Além do modo unsafe, você precisará confirmar a operação
   confirm_destructive_operation({
     operation_type: "database",
     confirmation_id: "id-recebido",
     user_confirmation: true
   })
