# Itaú Orchestrator Suite

Central inteligente de orquestração e integração de comunicações Itaú.

## ⚡ Otimizações Implementadas

- **Performance**: Regex patterns pré-compilados para processamento mais rápido
- **Frontend**: Debounce em filtros (300ms) para reduzir renderizações desnecessárias
- **Segurança**: Tokens movidos para variáveis de ambiente (`.env`)
- **Manutenção**: Limpeza automática de jobs antigos (configurável, padrão 30 dias)
- **Upload**: Validação de tamanho máximo (configurável, padrão 100MB) A plataforma importa arquivos legados PF, aplica as regras herdadas (normalização de canal, validações de CPF/CNPJ, montagem de cabeçalho e nomenclatura de arquivo) e exporta lotes prontos para SMS e e-mail. Dados sensíveis são mascarados nas visualizações para atender à LGPD.

Da planilha bruta à execução automatizada — a suíte acompanha o fluxo completo em três etapas, exibe métricas e libera os downloads assim que o job é finalizado.

## Requisitos

- Python 3.11 ou superior
- Pip (ou equivalente)

## Configuração do ambiente

```bash
python -m venv .venv
.\.venv\Scripts\activate
pip install -r requirements.txt
```

### Variáveis de Ambiente (Opcional)

Crie um arquivo `.env` na raiz do projeto (use `.env.example` como referência):

```bash
# Tokens de segurança (altere em produção)
ITAU_EMAIL_TOKEN=seu_token_email_aqui
ITAU_SMS_TOKEN=seu_token_sms_aqui

# Tamanho máximo de upload em MB
MAX_UPLOAD_SIZE_MB=100

# Dias para manter jobs antigos
JOB_CLEANUP_DAYS=30

# Número de workers para processamento
ORCHESTRATOR_MAX_WORKERS=4
```

## Execução do servidor

Execute a partir da raiz do projeto:

```bash
python -m uvicorn backend.main:app --reload --app-dir app
```

A interface web ficará disponível em `http://localhost:8000/`.

## Autenticação

- Primeiro acesso: login `matheus_feitosa`, senha `Heitor99$`.
- A interface exibe uma tela de login com opção de cadastro. Escolha um login único, defina a senha e confirme para criar a conta.
- Após autenticado, todas as requisições usam um token de sessão (`Authorization: Bearer ...`). Caso a sessão expire, basta entrar novamente.
- Toda ação (uploads, cadastro de templates etc.) é registrada automaticamente usando o login autenticado, sem necessidade de informar operador.
- O botão `Sair` encerra a sessão atual e limpa as credenciais armazenadas no navegador.

## Layouts

O layout atualmente suportado é **ITAÚ PF** (19 colunas). O mapeamento de campos e as regras de interpretação de datas ficam em `app/backend/layouts.py`. Novos layouts podem ser adicionados reutilizando a mesma estrutura. O cabeçalho do CSV é validado rigorosamente: se qualquer coluna obrigatória estiver ausente, o processamento é interrompido com uma mensagem indicando quais campos faltam.

## Estrutura de jobs

Cada upload cria a pasta `app/data/jobs/<job_id>/` com a estrutura:

- `upload/`: arquivo original recebido.
- `work/`: espaço reservado para artefatos intermediários.
- `export/`: arquivos finais por canal, além de um `export.zip` opcional.
- `progress/`: status, marcadores de etapa e logs de ajustes/geração.
- `logs/`: log estruturado do processamento (sem dados sensíveis).

Os arquivos exportados por canal trazem a primeira linha com o cabeçalho exigido e a segunda linha com a estrutura de colunas tratadas (por exemplo `CPFCNPJFormatado`, `Celular`, `Email`, `Campanha`, `Variavel`).

## Configurações principais

- `app/backend/config.py`: tokens, carteiras, fuso horário, defaults e layout padrão.
- `app/backend/templates_email_validos.json`: whitelist de templates permitidos para e-mail.

## Testes

```bash
python -m pytest
```

Os testes cobrem as regras de normalização, o pipeline completo do layout ITAÚ PF (incluindo masking de amostras) e os fluxos de autenticação e templates.

