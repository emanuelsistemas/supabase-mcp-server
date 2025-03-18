# Guia de Configuração do Supabase MCP Server

Este documento contém instruções detalhadas sobre como configurar o Supabase MCP Server, permitindo que Cline e outros clientes MCP se comuniquem de forma segura com bancos de dados Supabase.

## Índice

1. [Pré-requisitos](#pré-requisitos)
2. [Instalação](#instalação)
3. [Configuração](#configuração)
4. [Conexão com Cline](#conexão-com-cline)
5. [Métodos de Verificação](#métodos-de-verificação)
6. [Solução de Problemas](#solução-de-problemas)
7. [Exemplos de Uso](#exemplos-de-uso)

## Pré-requisitos

Antes de começar, certifique-se de ter:

- Python 3.12+ instalado no sistema
- Acesso a um projeto Supabase (seja local ou na nuvem)
- As seguintes informações do seu projeto Supabase:
  - ID de referência do projeto (project_ref)
  - Senha do banco de dados
  - Região onde o projeto está hospedado
  - Token de acesso pessoal Supabase (para API de gerenciamento)
  - Chave de papel de serviço (para operações de autenticação)

## Instalação

Há várias maneiras de instalar o Supabase MCP Server:

### Método 1: Usando pipx (Recomendado)

```bash
# Instalar via pipx (cria ambientes isolados para cada pacote)
pipx install supabase-mcp-server
```

### Método 2: Usando uv

```bash
# Instalar via uv
uv pip install supabase-mcp-server
```

### Método 3: Instalação a partir do código-fonte

```bash
# Clonar o repositório
mkdir -p ~/Cline/MCP
cd ~/Cline/MCP
git clone https://github.com/alexander-zuev/supabase-mcp-server.git
cd supabase-mcp-server

# Instalar com pipx em modo editável
pipx install -e .

# OU criar um ambiente virtual e instalar
uv venv
source .venv/bin/activate  # No Linux/MacOS
.venv\Scripts\activate  # No Windows
uv pip install -e .
```

## Configuração

### Localização do executável

Após a instalação, encontre o caminho completo do executável:

```bash
# No Linux/MacOS:
which supabase-mcp-server
# Saída típica: /home/usuario/.local/bin/supabase-mcp-server

# No Windows:
where supabase-mcp-server
# Saída típica: C:\Users\usuario\.local\bin\supabase-mcp-server.exe
```

### Variáveis de Ambiente Necessárias

O servidor MCP do Supabase requer as seguintes variáveis de ambiente:

| Variável | Descrição | Obrigatório | Exemplo |
|----------|-----------|------------|---------|
| `SUPABASE_PROJECT_REF` | ID de referência do projeto | Sim | `hbgmyrooyrisunhczfna` |
| `SUPABASE_DB_PASSWORD` | Senha do banco de dados | Sim | `sua-senha-segura` |
| `SUPABASE_REGION` | Região AWS do projeto | Sim | `sa-east-1` (São Paulo) |
| `SUPABASE_ACCESS_TOKEN` | Token de acesso para API de gerenciamento | Não | `seu-token-jwt` |
| `SUPABASE_SERVICE_ROLE_KEY` | Chave de serviço para Auth Admin | Não | `sua-chave-jwt` |

### Identificando a Região Correta

É crucial usar a região correta para evitar erros de conexão. Regiões disponíveis:

- `sa-east-1` - América do Sul (São Paulo) 
- `us-east-1` - Leste dos EUA (Virgínia)
- `us-west-1` - Oeste dos EUA (Califórnia)
- `eu-central-1` - Europa Central (Frankfurt)
- `eu-west-1` - Europa Ocidental (Irlanda)
- `ap-southeast-1` - Sudeste Asiático (Singapura)

## Conexão com Cline

Para configurar o servidor MCP no Cline:

1. Localize o arquivo de configuração do Cline MCP:
   ```
   /root/.windsurf-server/data/User/globalStorage/saoudrizwan.claude-dev/settings/cline_mcp_settings.json
   ```

2. Edite o arquivo para incluir a configuração do servidor MCP:
   ```json
   {
     "mcpServers": {
       "github.com/alexander-zuev/supabase-mcp-server": {
         "command": "/caminho/completo/para/supabase-mcp-server",
         "env": {
           "SUPABASE_PROJECT_REF": "seu-project-ref",
           "SUPABASE_DB_PASSWORD": "sua-senha-db",
           "SUPABASE_REGION": "sa-east-1",
           "SUPABASE_ACCESS_TOKEN": "seu-access-token",
           "SUPABASE_SERVICE_ROLE_KEY": "sua-service-role-key"
         },
         "disabled": false,
         "autoApprove": []
       }
     }
   }
   ```

3. Substituir:
   - `/caminho/completo/para/supabase-mcp-server` pelo caminho obtido com `which supabase-mcp-server`
   - `seu-project-ref`, `sua-senha-db`, etc. com seus valores reais

## Métodos de Verificação

Para verificar que o servidor está configurado corretamente:

### 1. Listar Schemas do Banco de Dados

```python
# Com o Cline ou outro cliente MCP, execute:
use_mcp_tool(
  server_name="github.com/alexander-zuev/supabase-mcp-server",
  tool_name="get_schemas",
  arguments={}
)
```

### 2. Teste CRUD Completo

1. Habilitar modo não seguro:
   ```python
   use_mcp_tool(
     server_name="github.com/alexander-zuev/supabase-mcp-server",
     tool_name="live_dangerously",
     arguments={
       "service": "database",
       "enable_unsafe_mode": true
     }
   )
   ```

2. Criar tabela, inserir, atualizar e excluir dados conforme necessário.

## Solução de Problemas

### Problema: Erro de região (Region mismatch)

**Erro:**
```
Error executing tool get_schemas: CONNECTION ERROR: Region mismatch detected!
```

**Solução:**
Atualize a variável `SUPABASE_REGION` no arquivo de configuração do Cline para a região correta do seu projeto.

### Problema: Erro de autenticação

**Erro:**
```
Authentication error: Invalid credentials
```

**Solução:**
Verifique se a senha do banco de dados está correta no arquivo de configuração.

### Problema: Servidor não encontrado

**Erro:**
```
spawn ENOENT
```

**Solução:**
Certifique-se de que o caminho para o executável está completo e correto no arquivo de configuração.

## Exemplos de Uso

### Exemplo 1: Consultar tabelas em um schema

```python
use_mcp_tool(
  server_name="github.com/alexander-zuev/supabase-mcp-server",
  tool_name="get_tables",
  arguments={
    "schema_name": "public"
  }
)
```

### Exemplo 2: Executar uma consulta SQL

```python
use_mcp_tool(
  server_name="github.com/alexander-zuev/supabase-mcp-server",
  tool_name="execute_postgresql",
  arguments={
    "query": "SELECT * FROM public.nome_tabela LIMIT 10;"
  }
)
```

### Exemplo 3: Operação Destrutiva (com confirmação)

Para operações destrutivas, o sistema bloqueia inicialmente e fornece um ID de confirmação:

```python
# 1. Tentativa de execução
use_mcp_tool(
  server_name="github.com/alexander-zuev/supabase-mcp-server",
  tool_name="execute_postgresql",
  arguments={
    "query": "DROP TABLE public.minha_tabela;"
  }
)

# 2. Confirmação da operação com o ID fornecido
use_mcp_tool(
  server_name="github.com/alexander-zuev/supabase-mcp-server",
  tool_name="confirm_destructive_operation",
  arguments={
    "operation_type": "database",
    "confirmation_id": "conf_xxxxxxxxxxxx", # ID fornecido pelo erro
    "user_confirmation": true
  }
)
```

---

Este guia foi criado em 18 de março de 2025 para facilitar a configuração e o uso do Supabase MCP Server com Cline e outros clientes MCP.
