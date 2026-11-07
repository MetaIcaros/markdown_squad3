
# Contrato de Integração — Squad 3: Agenda e Notificações

## 1. Visão Geral e Identificação
* **Módulo:** Agendamentos e Notificações
* **Squad Fornecedor:** Squad 3
* **Scrum Master:** Wagner
* **Guardião do Contrato:** Ícaro
* **Product Owner (PO):** Cássio
* **Desenvolvedores:** Cássio e Eraldo
* **Ambiente de Desenvolvimento:** Node.js, Express, SQLite, Git/GitHub
* **Infraestrutura:** Exclusivamente **3 Filiais** cadastradas no sistema (`id: 1`, `id: 2` e `id: 3`).
* **Objetivo:** Definir os contratos de API para a interface de atendimento das unidades (consulta de agenda, emissão de Ordem de Serviço e conclusão de atendimentos) e garantir a exportação de dados para relatórios financeiros.

---

## 2. Padrões de Dados, Identificadores e Regras de Negócio (RNs)

### Padrão Geral de Formatação e IDs
* **Formato de Datas:** Padrão brasileiro `DD-MM-AAAA` (Ex: `05-11-2026`).
* **Horários e Estimativas:** Todo agendamento possui um `horario_inicio` e um `horario_estimado_fim` explícitos para evitar conflitos de agenda na filial.
* **IDs de Clientes:** Numéricos sem limite de dígitos iniciando com 2 (Ex: `201`, `2012`).
* **IDs de Pets (Animais):** Numéricos sem limite de dígitos iniciando com 3 (Ex: `301`, `3045`).
* **IDs de Ordem de Serviço (OS / Agendamento):** Numéricos em ordem crescente sem limite, iniciados por zero (Ex: `01`, `02`, `089`, `07899`).
* **IDs de Funcionários:** Numéricos de 2 dígitos iniciando com 1 (Intervalo: `10` a `99`).
* **Porte dos Pets:** Restrito estritamente a 3 opções: `pequeno`, `médio` ou `grande`.
* **Espécies:** Restrito a `Cão` ou `Gato`.

### Catálogo Oficial de Serviços (`servico_id`)
* **`101`:** Banho e Tosa Porte Pequeno — R$ 60,00
* **`102`:** Banho e Tosa Porte Médio — R$ 80,00
* **`103`:** Banho e Tosa Porte Grande — R$ 100,00
* **`104`:** Consulta Veterinária — R$ 150,00

### Distinção de Perfis Profissionais
* **Atendente:** Operador do sistema na unidade, responsável pela consulta da agenda, abertura/emissão de Ordem de Serviço (OS) e marcação de conclusão do atendimento.
* **Banhista - Tosador:** Profissional técnico encarregado tanto do banho quanto da tosa (mesmo profissional para ambas as funções).
* **Veterinário:** Profissional responsável por consultas, exames e procedimentos médicos.

### Regras de Negócio Associadas
* **RN-04 (Isolamento por Filial):** A consulta e a gestão de agendamentos são restritas à filial do atendente logado (`unidade_id` válidos: `1`, `2` ou `3`).
* **RN-08 (Integração Financeira / Gestão):** A conclusão do atendimento expõe os dados consolidados da OS para consumo do Squad 2 (Relatórios de Faturamento/Estoque).
* **RN-09 (Controle de Acesso de Atendimento):** Operações de alteração, abertura de OS e conclusão exigem perfil de **Atendente** associado à respectiva filial.

---

## 3. Matriz de Integração entre Squads

| Squad Origem / Destino | Interface / Módulo | Tipo de Dado Trocado | Método / Rota |
| :--- | :--- | :--- | :--- |
| **Squad 1 $\rightarrow$ Squad 3** | Atendimento / Perfis | IDs de Clientes (`2xx`), Pets (`3xx`), Serviços (`10x`) e Profissionais (`1x`) | `GET /api/clientes`, `GET /api/pets`, `GET /api/servicos` |
| **Squad 3 $\rightarrow$ Squad 3** | Interface de Atendimento | Agenda da filial, Ordens de Serviço (OS) e atualização de status | `GET /api/agendamentos`, `PUT /api/agendamentos/:os_id/concluir` |
| **Squad 3 $\rightarrow$ Squad 2** | Gestão / Relatórios | Dados consolidados de OS concluídas para faturamento | `GET /api/agendamentos/concluidos` |

---

## 4. Definição das APIs e Mocks Oficiais

### 🔵 ROTA 1: Listar Agenda Diária da Unidade (Interface do Atendente)
Permite ao **Atendente** consultar a lista de agendamentos e Ordens de Serviço da filial para um determinado dia.

* **Método HTTP:** `GET`
* **End-point:** `/api/agendamentos`
* **Consumidor:** Interface de Atendimento (Squad 3 / Squad 1)

#### Parâmetros de Entrada (Query Parameters)
| Campo | Tipo | Obrigatório? | Descrição / Valores Permitidos |
| :--- | :--- | :--- | :--- |
| `unidade_id` | Integer | **Sim** | ID da filial (`1`, `2` ou `3`). |
| `data` | String (DD-MM-AAAA) | Não | Data da consulta (Ex: `05-11-2026`). Padrão: data atual. |
| `status` | String | Não | Filtra por estado: `pendente`, `em_andamento`, `concluido`, `cancelado`. |

#### Payload de Resposta de Sucesso (`HTTP 200 OK`)
```json
{
  "sucesso": true,
  "unidade_id": 2,
  "data_consulta": "05-11-2026",
  "total": 1,
  "dados": [
    {
      "cliente": {
        "id": 2012,
        "nome": "Marina Silva"
      },
      "os_id": "089",
      "horario_inicio": "05-11-2026 14:30",
      "horario_estimado_fim": "05-11-2026 15:30",
      "status": "pendente",
      "pet": {
        "id": 3045,
        "nome": "Rex",
        "especie": "Cão",
        "porte": "médio"
      },
      "servico": {
        "id": 102,
        "atividade": "Banho e Tosa Porte Médio",
        "preco": 80.00
      },
      "profissional_executor": {
        "id": 15,
        "nome": "Carlos Silva",
        "perfil": "Banhista - Tosador"
      },
      "atendente_registro": {
        "id": 18,
        "nome": "Ana Souza",
        "perfil": "Atendente"
      }
    }
  ]
}
### 🔵 ROTA 2: Registrar Conclusão de Atendimento / Ordem de Serviço
Permite ao **Atendente** alterar o status da Ordem de Serviço (`os_id`) para concluído ao finalizar o serviço na filial.

- **Método HTTP:** `PUT` (ou `PATCH`)
- **End-point:** `/api/agendamentos/:os_id/concluir`
- **Consumidor:** Interface de Atendimento (Squad 3)

#### Parâmetros de Rota (Path Parameters)
| Parâmetro | Tipo | Obrigatório? | Descrição / Exemplo |
| :--- | :--- | :--- | :--- |
| `:os_id` | String | **Sim** | ID da Ordem de Serviço iniciado por zero (Ex: `"089"`). |

#### Corpo da Requisição (Payload JSON Body)
```json
{
  "status": "concluido",
  "observacoes": "Banho e tosa concluídos sem intercorrências no horário previsto."
}
Exemplo de Resposta de Sucesso (HTTP 200 OK)JSON{
  "sucesso": true,
  "mensagem": "Ordem de serviço concluída com sucesso.",
  "ordem_servico": {
    "cliente": {
      "id": 2012,
      "nome": "Marina Silva"
    },
    "os_id": "089",
    "status": "concluido",
    "unidade_id": 2,
    "data_conclusao": "05-11-2026 15:30",
    "pet": {
      "id": 3045,
      "nome": "Rex",
      "especie": "Cão",
      "porte": "médio"
    },
    "servico": {
      "id": 102,
      "atividade": "Banho e Tosa Porte Médio",
      "preco_praticado": 80.00
    },
    "profissional_executor": {
      "id": 15,
      "nome": "Carlos Silva",
      "perfil": "Banhista - Tosador"
    }
  }
}
Tratamento de Erros RetornadosHTTP 400 Bad Request: Ordem de serviço inválida ou transição de status não permitida.JSON{
  "sucesso": false,
  "erro": "A OS nº 089 já se encontra concluída ou cancelada."
}
HTTP 403 Forbidden: Violação da RN-09 ou RN-04 (Usuário sem permissão de Atendente ou em filial incorreta).JSON{
  "sucesso": false,
  "erro": "Acesso negado: apenas atendentes da filial correspondente podem concluir Ordens de Serviço."
}
🔵 ROTA 3: Exportar Agendamentos Concluídos (Para Gestão/Relatórios)Disponibiliza massivamente ao Squad 2 (Módulo de Gestão e Relatórios) a lista consolidada de todas as Ordens de Serviço que atingiram o status concluido. Permite filtros opcionais por período de datas e filial para fins de fechamento de caixa, faturamento e controle de estoque de insumos.Método HTTP: GETEnd-point: /api/agendamentos/concluidosConsumidor: Squad 2 — Gestão e Relatórios (Guardiã: Beatriz / Bia)Parâmetros de Entrada (Query Parameters)CampoTipoObrigatório?Descrição / Valores Permitidosunidade_idIntegerNãoFiltra o relatório por uma filial específica (1, 2 ou 3). Se omitido, retorna os dados consolidados de todas as filiais.data_inicioString (DD-MM-AAAA)NãoData inicial do período de apuração financeira (Ex: 01-11-2026).data_fimString (DD-MM-AAAA)NãoData final do período de apuração financeira (Ex: 05-11-2026).Exemplo de Resposta de Sucesso (HTTP 200 OK)JSON{
  "sucesso": true,
  "total_registros": 2,
  "filtro_aplicado": {
    "unidade_id": null,
    "periodo": "01-11-2026 a 05-11-2026"
  },
  "dados": [
    {
      "cliente_id": 2012,
      "os_id": "089",
      "unidade_id": 2,
      "data_conclusao": "05-11-2026 15:30",
      "pet_id": 3045,
      "servico_id": 102,
      "preco_praticado": 80.00
    },
    {
      "cliente_id": 2018,
      "os_id": "090",
      "unidade_id": 1,
      "data_conclusao": "05-11-2026 16:15",
      "pet_id": 3102,
      "servico_id": 104,
      "preco_praticado": 150.00
    }
  ]
}
Tratamento de Erros RetornadosHTTP 400 Bad Request: Parâmetros de data enviados em formato inválido diferente de DD-MM-AAAA ou intervalo inconsistente (data_inicio posterior a data_fim).JSON{
  "sucesso": false,
  "erro": "Parâmetros de data inválidos. Utilize estritamente o formato DD-MM-AAAA."
}
HTTP 500 Internal Server Error: Falha crítica na consulta ou na consolidação dos dados junto ao banco de dados SQLite do Squad 3.JSON{
  "sucesso": false,
  "erro": "Erro interno ao consolidar a massa de Ordens de Serviço concluídas para exportação."
}
