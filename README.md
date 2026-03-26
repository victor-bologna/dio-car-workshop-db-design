# Oficina Mecânica – Sistema de Ordens de Serviço (OS)

Modelo de banco de dados para gerenciamento de ordens de serviço em oficina mecânica, com controle de clientes, veículos, equipes, serviços, peças e autorização.

***

## Visão Geral

O sistema permite:

- Cadastro de clientes e seus veículos.  
- Criação de ordens de serviço (OS) vinculadas a veículos.  
- Designação de OS para equipes de mecânicos.  
- Identificação de serviços (com referência de mão-de-obra) e peças.  
- Autorização do cliente para execução.  
- Controle de status e conclusão das OS.

***

## Entidades Principais

### CLIENTE

Representa o cliente da oficina.

- `id` (PK)  
- `nome`, `telefone`, `endereco`  

**Relacionamentos:**
- 1:N `VEICULO`

***

### VEICULO

Veículo do cliente que será atendido.

- `id` (PK)  
- `id_cliente` (FK)  
- `placa`, `modelo`, `ano`  

**Relacionamentos:**
- 1:N `ORDEM_SERVICO`

***

### ORDEM_SERVICO

Ordem de serviço principal.

- `id` (PK)  
- `numero_os`, `data_emissao`, `data_conclusao`  
- `id_veiculo` (FK)  
- `status`, `autorizacao_cliente`, `data_autorizacao`  
- `valor_total` (calculado automaticamente)  

**Relacionamentos:**
- N:N `EQUIPE` (designação da OS)  
- N:N `SERVICO` (serviços executados)  
- N:N `PECA` (peças utilizadas)

***

### MECANICO

Mecânico da oficina.

- `id` (PK)  
- `codigo`, `nome`, `endereco`, `especialidade`  

**Relacionamentos:**
- N:N `EQUIPE`

***

### EQUIPE

Agrupamento de mecânicos para execução da OS.

- `id` (PK)  
- `nome_equipe`  

**Relacionamentos:**
- N:N `MECANICO`  
- N:N `ORDEM_SERVICO`

***

## Serviços e Peças

### SERVICO_REFERENCIA

Tabela de referência de mão-de-obra (preços padrão).

- `id` (PK)  
- `nome_servico`, `valor_hora`  

### ORDEM_SERVICO_HAS_SERVICO

Serviços executados em cada OS.

- `id_os` (FK), `id_servico_ref` (FK)  
- `quantidade_horas`, `valor_unitario`, `valor_total_servico`  

### PECA

Catálogo de peças.

- `id` (PK)  
- `codigo`, `descricao`, `valor_unitario`  

### ORDEM_SERVICO_HAS_PECA

Peças utilizadas em cada OS.

- `id_os` (FK), `id_peca` (FK)  
- `quantidade`, `valor_unitario`, `valor_total_peca`  

***

## Regras de Negócio Implementadas

1. **Cálculo automático**: `ORDEM_SERVICO.valor_total = SUM(valor_total_servico + valor_total_peca)`  
2. **Autorização**: Cliente deve autorizar (`autorizacao_cliente = TRUE`) antes da execução.  
3. **Status workflow**: Emissão → Autorizado → Em andamento → Concluído.  
4. **Equipes**: OS sempre designada a uma equipe de mecânicos.  

***

## Fluxo de uso típico

```
1. Cliente cadastra veículo
2. Cria OS → equipe identifica serviços/pecas
3. Cliente autoriza OS
4. Equipe executa → atualiza status/horas/quantidades
5. OS concluída → fatura total calculado
```

***

## Pontos Fortes do Modelo

- **Flexibilidade**: Suporta múltiplos veículos por cliente, múltiplas OSs por veículo.  
- **Controle de custos**: Separação clara de mão-de-obra e peças.  
- **Responsabilidade**: Equipes claramente designadas por OS.  
- **Auditoria**: Datas de emissão, autorização e conclusão.  
