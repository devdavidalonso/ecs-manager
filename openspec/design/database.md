# Modelagem de Dados NoSQL (Cloud Firestore)
## Projeto: Evangelização Cairbar Schutel (ecs-manager)

Este documento especifica a modelagem de dados NoSQL utilizando o **Cloud Firestore** para o sistema de gestão da Evangelização Cairbar Schutel. Ele descreve a estrutura das coleções principais, os tipos de dados, as regras de negócio para preenchimento de campos e exemplos práticos de documentos JSON.

---

## 1. Decisões de Arquitetura NoSQL

Para o desenvolvimento deste modelo, foram aplicadas as seguintes premissas do Cloud Firestore:
* **Desnormalização (Nesting):** Optou-se por aninhar os dados dos responsáveis dentro do documento do aluno (objeto `responsavel`). Como a relação responsável-aluno na ficha de inscrição é tipicamente direta e não exige consultas complexas isoladas dos responsáveis (eles sempre são visualizados no contexto do aluno), o aninhamento reduz leituras no banco de dados e simplifica a escrita em um único documento.
* **Tipos de Dados Nativos:** Utilização de `timestamp` para datas, garantindo suporte nativo a ordenações cronológicas e filtros por período.
* **Campos de Gestão Interna:** Separação clara entre campos preenchidos pelo usuário (público) e campos controlados pela administração (privados), preparando a estrutura para regras de segurança (Firestore Security Rules) robustas.

---

## 2. Coleção `alunos`

A coleção `alunos` armazena o cadastro das crianças e adolescentes inscritos na Evangelização, englobando as informações capturadas no formulário físico (e posteriormente na interface de inscrição pública) e o controle administrativo de alocação de turmas e status.

### 2.1. Tabela de Estrutura do Documento

| Campo | Tipo | Origem | Descrição | Exemplo / Valores Permitidos |
| :--- | :--- | :--- | :--- | :--- |
| `id` | `string` | Auto | ID único do documento gerado pelo Firestore. | `3hF9aK2dJs8L1oPq` |
| `nome` | `string` | Formulário | Nome completo do aluno (criança/adolescente). | `"Pedro Henrique de Souza"` |
| `dataNascimento` | `timestamp` | Formulário | Data de nascimento para cálculo de idade e turma. | `2018-05-15T00:00:00Z` |
| `comoConheceu` | `string` | Formulário | Canal ou meio pelo qual conheceu a Evangelização. | `"Indicação de amigos"`, `"Redes sociais"` |
| `quantidadeFamiliares` | `number` | Formulário | Quantidade de membros da família que pretendem participar (ex: Sala da Família). | `2` |
| **`responsavel`** | **`map`** | **Formulário** | **Objeto aninhado contendo os dados do responsável legal.** | |
| ↳ `nome` | `string` | Formulário | Nome completo do responsável. | `"Marisa de Souza"` |
| ↳ `telefone` | `string` | Formulário | Telefone / WhatsApp de contato (padrão E.164). | `"+5511998765432"` |
| ↳ `email` | `string` | Formulário | Endereço de e-mail principal. | `"marisa.souza@email.com"` |
| `status` | `string` | Admin | Status atual da inscrição do aluno no sistema. | `"pre_inscrito"`, `"confirmado"`, `"cancelado"`, `"lista_espera"` |
| `turma` | `string` | Admin | Turma alocada com base na faixa etária. | `"maternal"`, `"jardim"`, `"ciclo_1"`, `"ciclo_2"`, `"ciclo_3"`, `"mocidade"` |
| `dataCadastro` | `timestamp` | Auto | Data e hora em que a pré-inscrição foi realizada. | `2026-06-07T18:00:00Z` |
| `dataAtualizacao` | `timestamp` | Auto | Última modificação realizada no documento. | `2026-06-07T19:30:00Z` |

> [!NOTE]
> **Regra de Negócio para Alocação de Turma (Sugestão de Faixa Etária):**
> * **Maternal:** 0 a 3 anos
> * **Jardim:** 4 a 6 anos
> * **Ciclo 1:** 7 e 8 anos
> * **Ciclo 2:** 9 e 10 anos
> * **Ciclo 3:** 11 e 12 anos
> * **Mocidade:** 13 anos ou mais

### 2.2. Exemplo JSON Realista (`alunos`)

```json
{
  "nome": "Guilherme Santos Oliveira",
  "dataNascimento": {
    "_seconds": 1528934400,
    "_nanoseconds": 0
  },
  "comoConheceu": "Panfleto entregue na palestra pública de sábado",
  "quantidadeFamiliares": 3,
  "responsavel": {
    "nome": "Roberto Carlos Oliveira",
    "telefone": "+5511987654321",
    "email": "roberto.carlos@provedor.com"
  },
  "status": "confirmado",
  "turma": "ciclo_1",
  "dataCadastro": {
    "_seconds": 1780862400,
    "_nanoseconds": 0
  },
  "dataAtualizacao": {
    "_seconds": 1780869600,
    "_nanoseconds": 0
  }
}
```

---

## 3. Coleção `voluntarios`

A coleção `voluntarios` centraliza a listagem de colaboradores da Evangelização, auxiliando na escala de facilitadores, organizadores da recepção, coordenadores de turmas e palestrantes da Sala da Família.

### 3.1. Tabela de Estrutura do Documento

| Campo | Tipo | Origem | Descrição | Exemplo / Valores Permitidos |
| :--- | :--- | :--- | :--- | :--- |
| `id` | `string` | Auto | ID único do documento gerado pelo Firestore. | `v8Jd2K9sL1pQ4aBc` |
| `nome` | `string` | Form / Admin | Nome completo do voluntário. | `"Francisco Cândido Xavier"` |
| `telefone` | `string` | Form / Admin | Telefone de contato rápido (WhatsApp) (padrão E.164). | `"+5534999991910"` |
| `email` | `string` | Form / Admin | Endereço de e-mail para comunicação oficial. | `"chico.xavier@mensageiros.org"` |
| `areasInteresse` | `array (string)` | Form / Admin | Áreas nas quais o voluntário deseja atuar. | `["evangelizador", "recepcao", "sala_da_familia"]` |
| `status` | `string` | Admin | Status atual de atividade do voluntário na instituição. | `"ativo"`, `"inativo"` |
| `dataCadastro` | `timestamp` | Auto | Data de entrada do voluntário no sistema. | `2026-05-10T14:00:00Z` |
| `dataAtualizacao` | `timestamp` | Auto | Última modificação no registro do voluntário. | `2026-06-07T18:50:00Z` |

> [!TIP]
> **Valores comuns para `areasInteresse`:**
> * `evangelizador`: Atuação direta em sala de aula com crianças/jovens.
> * `recepcao`: Recepção e acolhimento de famílias e controle de presença na entrada.
> * `sala_da_familia`: Condução de palestras e atividades de acolhimento para os pais/responsáveis.
> * `apoio_geral`: Auxílio na preparação do lanche, organização de eventos e logística interna.

### 3.2. Exemplo JSON Realista (`voluntarios`)

```json
{
  "nome": "Carla Mendonça Vieira",
  "telefone": "+5511977665544",
  "email": "carla.mendonca@gmail.com",
  "areasInteresse": [
    "evangelizador",
    "sala_da_familia"
  ],
  "status": "ativo",
  "dataCadastro": {
    "_seconds": 1778421600,
    "_nanoseconds": 0
  },
  "dataAtualizacao": {
    "_seconds": 1780867800,
    "_nanoseconds": 0
  }
}
```

---

## 4. Planejamento de Índices e Consultas

Para as telas administrativas do `ecs-manager`, serão necessárias consultas avançadas que exigem a criação de **Índices Compostos** no Firestore. Abaixo estão os cenários previstos:

1. **Filtrar alunos por status e ordenar por data de cadastro (Ex: Fila de espera mais antiga primeiro):**
   * Coleção: `alunos`
   * Campos indexados: `status` (Ascendente) ➔ `dataCadastro` (Ascendente)
2. **Listar alunos de uma determinada turma ordenados por nome alfabético:**
   * Coleção: `alunos`
   * Campos indexados: `turma` (Ascendente) ➔ `nome` (Ascendente)
3. **Filtrar voluntários ativos por área de interesse específica:**
   * Coleção: `voluntarios`
   * Campos indexados: `status` (Ascendente) ➔ `areasInteresse` (Array-Contains)

---

## 5. Diretrizes de Segurança (Esboço de Security Rules)

Para proteger a integridade dos dados e a privacidade (em conformidade com a LGPD, visto que há coleta de dados de menores de idade), as seguintes permissões devem ser implementadas no Firestore:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Regras para Alunos
    match /alunos/{alunoId} {
      // Qualquer usuário não autenticado pode enviar formulário de pré-inscrição (criar)
      allow create: if request.resource.data.status == "pre_inscrito";
      
      // Apenas administradores autenticados podem ler, atualizar ou excluir registros de alunos
      allow read, update, delete: if request.auth != null && request.auth.token.role == "admin";
    }

    // Regras para Voluntários
    match /voluntarios/{voluntarioId} {
      // Qualquer um pode se candidatar como voluntário (criar)
      allow create: if true;
      
      // Apenas administradores autenticados podem ler e gerenciar os dados dos voluntários
      allow read, update, delete: if request.auth != null && request.auth.token.role == "admin";
    }
  }
}
```
