Markdown Squad_3 
# Contrato de Integração — Squad 3: Agenda e Notificações

## 1. Visão Geral e Identificação
* **Módulo:** Agendamentos e Notificações
* **Squad Fornecedor:** Squad 3
* **Scrum Master:** Wagner
* **Guardião do Contrato:** Ícaro
* **Product Owner (PO):** Cássio
* **Desenvolvedores:** Cássio e Eraldo
* **Ambiente de Desenvolvimento:** Node.js, Express, SQLite, Git/GitHub
* **Infraestrutura:** Únidades 1, 2,3 cadastradas no sistema.
* **Objetivo:** Definir os contratos de API para a interface de atendimento das unidades (consulta de agenda e conclusão de serviços) e garantir a exportação contínua de dados de atendimentos concluídos para relatórios financeiros.

---

## 2. Perfis de Acesso e Regras de Negócio (RNs)

### Distinção de Perfis Profissionais
* **Atendente:** Responsável por operar a interface da unidade, consultar a agenda diária e registrar a conclusão dos serviços prestados.
* **Tosador / Prestador de Serviço:** Profissional técnico associado à execução direta do agendamento (ex: banho, tosa, consulta). Não opera a finalização financeira na interface, apenas recebe os agendamentos atribuídos.
* **Funcionário Geral:** Perfil genérico da equipe com permissões restritas de visualização interna.

### Regras de Negócio Associadas
* **RN-04 (Isolamento por Unidade):** A consulta e a gestão de agendamentos são estritamente filtradas pela unidade do atendente logado. O sistema possui no máximo **3 unidades** (`unidade_id` válidos: `1`, `2` ou `3`).
* **RN-08 (Integração Financeira / Gestão):** A marcação de um atendimento como `concluido` expõe automaticamente os dados consolidados do agendamento para consumo do Squad 2 (Relatórios de Faturamento/Estoque).
* **RN-09 (Controle de Acesso de Atendimento):** Operações de alteração e conclusão de atendimento exigem credenciais válidas do perfil **Atendente** vinculado à respectiva unidade (`1`, `2` ou `3`).

---

## 3. Matriz de Integração entre Squads

| Squad Origem / Destino | Interface / Módulo | Tipo de Dado Trocado | Método / Rota |
| :--- | :--- | :--- | :--- |
| **Squad 1 $\rightarrow$ Squad 3** | Atendimento / Perfis | IDs e nomes de Clientes, Pets, Serviços e Tosadores/Profissionais | `GET /api/clientes`, `GET /api/pets`, `GET /api/servicos` |
| **Squad 3 $\rightarrow$ Squad 3** | Interface de Atendimento | Grade diária da unidade e atualização de status | `GET /api/agendamentos`, `PUT /api/agendamentos/:id/concluir` |
| **Squad 3 $\rightarrow$ Squad 2** | Gestão / Relatórios | Massa de atendimentos concluídos para faturamento | `GET /api/agendamentos/concluidos` |

---

## 4. Definição das APIs e Mocks Oficiais

### ROTA 1: Listar Agenda Diária da Unidade (Interface do Atendente)
Permite ao **Atendente** visualizar a lista de agendamentos filtrados por unidade e data.

* **Método HTTP:** `GET`
* **End-point:** `/api/agendamentos`
* **Consumidor:** Interface de Atendimento (Squad 3 / Squad 1)

#### Parâmetros de Entrada (Query Parameters)
| Campo | Tipo | Obrigatório? | Descrição / Valores Permitidos |
| :--- | :--- | :--- | :--- |
| `unidade_id` | Integer | **Sim** | ID da unidade em atendimento (Permitidos: `1`, `2` ou `3`). |
| `data` | String (YYYY-MM-DD) | Não | Data da consulta. Se omitido, assume a data atual. |
| `status` | String | Não | Filtra por estado: `pendente`, `concluido`, `cancelado`. |

#### Payload de Resposta de Sucesso (`HTTP 200 OK`)
```json
{
  "sucesso": true,
  "unidade_id": 2,
  "data_consulta": "2026-11-05",
  "total": 1,
  "dados": [
    {
      "agendamento_id": 45,
      "horario_inicio": "2026-11-05T14:30:00Z",
      "duracao_minutos": 60,
      "status": "pendente",
      "cliente": {
        "id": 12,
        "nome": "Marina Silva"
      },
      "pet": {
        "id": 3,
        "nome": "Rex",
        "especie": "Cachorro"
      },
      "servico": {
        "id": 1,
        "nome": "Banho e Tosa Porte Médio",
        "preco": 80.00
      },
      "tosador": {
        "id": 5,
        "nome": "Carlos Tosador",
        "perfil": "Tosador"
      },
      "atendente_registro": {
        "id": 8,
        "nome": "Ana Atendente",
        "perfil": "Atendente"
      }
    }
  ]
}