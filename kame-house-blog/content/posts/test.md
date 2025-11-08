+++
date = '2025-11-08T16:08:12-03:00'
draft = true
title = 'Test'
+++

# Campanha na Barra de Busca - Documentação Técnica

  

## Introdução

  

Esta é a documentação técnica da funcionalidade de **Campanha na Barra de Busca** (também conhecida como Banner AutoComplete). O objetivo deste documento é fornecer informações técnicas detalhadas sobre a arquitetura, funcionamento interno, regras de negócio, integrações e troubleshooting da feature.

  

> **🚨 IMPORTANTE**: Este documento contém informações técnicas de infraestrutura e arquitetura. Para documentação de uso e guia do usuário, consulte a documentação não técnica disponível no portal administrativo.

  

**Público-alvo**: Este documento é destinado a técnicos de suporte, desenvolvedores e profissionais que necessitem compreender o funcionamento técnico da funcionalidade, incluindo estrutura de banco de dados, APIs e fluxos de dados.

  

---

  

## Contexto

  

O sistema de campanhas de busca opera em dois ambientes distintos:

  

### Admin Novo (Painel Administrativo)

Portal utilizado pelos administradores para gerenciar as campanhas de busca. Localizado na seção **Comunicação → Home logada → Campanha de busca**.

  

**Responsabilidades**:

- Cadastro de novas campanhas

- Edição de campanhas existentes

- Ativação e desativação de campanhas

- Configuração de regras de segmentação

- Upload de imagens e definição de metadados

  

### Loja Virtual (WebCustomer)

Interface voltada ao cliente final onde as campanhas são exibidas na barra de busca.

  

**Responsabilidades**:

- Exibição da campanha ativa para shoppers autenticados

- Aplicação de regras de segmentação baseadas no perfil do cliente

- Renderização visual da campanha com imagem e call-to-action

  

### Área Específica: AutoComplete

  

As campanhas de busca são tecnicamente classificadas como banners da área **AutoComplete**, diferenciando-se de outras áreas como:

- **Home**: Banners carrossel da página inicial

- **TopBar**: Barra superior fixa

- **Fixed**: Banners fixos na página

  

---

  

## Fluxo da Funcionalidade

  

### Painel Administrativo

  

Os administradores acessam a funcionalidade através do menu **Admin novo → Comunicação → Home logada → Campanha de busca → Gerenciar**.

  

**Operações disponíveis**:

  

1. **Criar nova campanha**

- Preencher formulário com título, subtítulo, call to action, texto do botão, tag

- Upload de imagem (obrigatório: 603x349px, PNG ou JPEG)

- Definir período de vigência (datas inicial e final)

- Configurar regras de segmentação (obrigatório: pelo menos uma regra)

  

2. **Editar campanha existente**

- Localizar campanha na listagem

- Acessar via ícone lateral

- Modificar campos necessários

- Salvar alterações

  

3. **Ativar/Desativar campanha**

- Localizar campanha na listagem

- Utilizar botão na coluna Status

- **Restrição**: Apenas uma campanha pode estar ativa por vez

  

4. **Excluir campanha**

- Remover campanha do sistema permanentemente

  

### Loja Virtual

  

Durante a navegação do shopper autenticado na Loja Virtual:

  

1. Shopper faz login na Loja Virtual

2. Shopper acessa qualquer página que contenha a barra de busca

3. Front-end da Loja Virtual faz requisição ao Marketing API para carregar a campanha

4. API verifica se existe campanha ativa e elegível para aquele CNPJ

5. Sistema aplica regras de segmentação baseadas no perfil do cliente

6. Campanha é renderizada na interface da barra de busca

7. Shopper pode interagir clicando no call-to-action configurado

  

---

  

## Documentação Relacionada

  

Para informações sobre como utilizar a funcionalidade do ponto de vista do usuário administrador, consulte o guia não técnico da funcionalidade disponível no portal de documentação.

  

**Tópicos abordados na documentação de usuário**:

- Passo a passo para criar campanhas

- Limitações de caracteres por campo

- Especificações de imagem

- Como configurar regras de exibição

- Como ativar e desativar campanhas

  

---

  

## Contexto Técnico - Gerenciamento

  

### APIs de Gerenciamento

  

O gerenciamento de campanhas utiliza a versão 2 da API de Marketing, especificamente os endpoints do `BannerController`.

  

#### Listar Campanhas Filtradas

  

**Endpoint**: `GET /marketing/v2/banner/getbannerfiltered`

  

**Descrição**: Retorna lista paginada de campanhas com suporte a múltiplos filtros.

  

**Query Parameters**:

- `pageSize` (int): Quantidade de registros por página (padrão: 20)

- `pageIndex` (int): Número da página (padrão: 1)

- `area` (string): Área do banner - deve ser **"AutoComplete"** para campanhas de busca

- `sellerId` (Guid): Filtrar por seller específico (opcional)

- `search` (string): Busca textual em título, área e autor (opcional)

- `status` (int): Filtrar por status (0 = inativo, 1 = ativo, 2 = todos) (opcional)

- `creationDate` (string): Filtrar por data de criação (opcional)

- `initialDate` (string): Filtrar por data de início de vigência (opcional)

- `finishDate` (string): Filtrar por data de término de vigência (opcional)

  

**Autorização**: Requer role `JSMAdmin` via JWT token.

  

**Lógica de Filtro**:

- Busca textual utiliza operador LIKE em múltiplos campos

- Filtros são combinados com operador AND

- Ordenação: Status (DESC) → Order (ASC)

- Inclui regras de segmentação (BannersRules) automaticamente

  

#### Criar Nova Campanha

  

**Endpoint**: `POST /marketing/v2/banner/post`

  

**Descrição**: Cadastra nova campanha de busca no sistema.

  

**Content-Type**: `multipart/form-data`

  

**Campos do FormData**:

- `sellerId` (Guid): Identificador do seller - usar `00000000-0000-0000-0000-000000000000` para campanha default

- `title` (string): Título da campanha (obrigatório, max 50 caracteres)

- `area` (string): Deve ser **"AutoComplete"** (obrigatório)

- `callToAction` (string): Caminho relativo após `/l/` na URL (obrigatório, max 500 caracteres)

- `subTitle` (string): Subtítulo da campanha (opcional, max 68 caracteres)

- `tag` (string): Tag/categoria da campanha (opcional, max 32 caracteres)

- `titleButton` (string): Texto do botão de ação (obrigatório, max 20 caracteres)

- `effectiveInitialDate` (DateTime): Data/hora de início de vigência (obrigatório)

- `effectiveFinalDate` (DateTime): Data/hora de término de vigência (opcional para AutoComplete)

- `desktopImage` (File): Arquivo de imagem PNG ou JPEG (obrigatório, recomendado 603x349px)

- `mobileImage` (File): Arquivo de imagem para mobile (opcional)

- `author` (string): Nome do autor/criador (obrigatório, max 50 caracteres)

- `status` (int): Status inicial (0 = inativo, 1 = ativo)

- `order` (int): Ordem de exibição (padrão: 0)

  

**Autorização**: Requer role `JSMAdmin` via JWT token.

  

**Regras de Validação**:

- Apenas uma campanha AutoComplete pode estar ativa simultaneamente

- Se `status = 1`, sistema valida se já existe outra campanha ativa

- Segmentação deve ser configurada por regras OU por planilha de CNPJs (não ambos)

- Se usar regras de segmentação, `sellerId` não pode ser vazio

- Se usar planilha de CNPJs, `sellerId` deve ser vazio (00000000-0000-0000-0000-000000000000)

  

#### Atualizar Campanha

  

**Endpoint**: `PUT /marketing/v2/banner/put`

  

**Descrição**: Atualiza uma campanha existente.

  

**Content-Type**: `multipart/form-data`

  

**Campos**: Mesmos campos do POST, incluindo `id` (Guid) da campanha a ser atualizada.

  

**Autorização**: Requer role `JSMAdmin` via JWT token.

  

#### Atualizar Status da Campanha

  

**Endpoint**: `PATCH /marketing/v2/banner/updatestatus`

  

**Descrição**: Ativa ou desativa uma campanha.

  

**Content-Type**: `application/json`

  

**Body**:

```json

{

"id": "uuid-da-campanha",

"status": 1

}

```

  

**Autorização**: Requer role `JSMAdmin` via JWT token.

  

**Validação**: Antes de ativar, verifica se já existe outra campanha AutoComplete ativa.

  

#### Excluir Campanha

  

**Endpoint**: `DELETE /marketing/v2/banner/{id}`

  

**Descrição**: Remove permanentemente uma campanha do sistema.

  

**Path Parameter**:

- `id` (Guid): Identificador único da campanha

  

**Autorização**: Requer role `JSMAdmin` via JWT token.

  

**Efeito Colateral**: Remove todas as regras de segmentação e relacionamentos associados.

  

#### Atualizar Planilha de Segmentação

  

**Endpoint**: `PATCH /marketing/v2/banner/sheetfile`

  

**Descrição**: Atualiza ou adiciona planilha CSV com lista de CNPJs autorizados a visualizar a campanha.

  

**Content-Type**: `multipart/form-data`

  

**Campos**:

- `bannerId` (Guid): ID da campanha

- `sheetFile` (File): Arquivo CSV com coluna de CNPJs

  

**Autorização**: Requer role `JSMAdmin` via JWT token.

  

**Processamento**:

- Sistema faz parse do arquivo CSV

- Extrai CNPJs da coluna especificada

- Popula tabela `Cnpjs` e relacionamento `BannersCnpjs`

- Invalida cache Redis para forçar atualização

  

### Regras de Negócio - Gerenciamento

  

1. **Unicidade de Banner Ativo**: Apenas uma campanha AutoComplete pode ter `status = 1` por vez

2. **Segmentação Excludente**: Campanha pode ser segmentada por regras (ParameterId) OU por planilha de CNPJs, nunca ambos

3. **Regra de Exibição Obrigatória**: Ao criar campanha, deve-se definir pelo menos uma regra de segmentação

4. **Imagem Desktop Obrigatória**: Campo `desktopImage` é obrigatório no cadastro

5. **Data Final Opcional**: Para área AutoComplete, `effectiveFinalDate` é opcional

6. **Invalidação de Cache**: Ao criar, atualizar ou deletar campanha, cache Redis é limpo usando padrão `banner-autocomplete-*`

  

---

  

## Contexto Técnico - Visualização

  

### API de Visualização

  

#### Recuperar Campanha por CNPJ

  

**Endpoint**: `GET /marketing/v2/banner/campaignautocomplete/{cnpj}`

  

**Descrição**: Retorna a campanha ativa elegível para visualização pelo shopper identificado pelo CNPJ.

  

**Path Parameter**:

- `cnpj` (string): CNPJ do cliente (14 dígitos sem formatação)

  

**Autorização**: Requer autenticação via JWT token (qualquer usuário autenticado).

  

**Response**: Objeto `BannerWebCustomerDto` contendo:

- `imagePath` (string): URL completa da imagem desktop no CDN

- `imageMobilePath` (string): URL completa da imagem mobile no CDN

- `callToAction` (string): Caminho para redirecionamento

- `order` (int): Ordem de exibição

- `title` (string): Título da campanha

- `tag` (string): Tag/categoria

- `subTitle` (string): Subtítulo

- `titleButton` (string): Texto do botão

  

**Status Code**:

- `200 OK`: Campanha encontrada e retornada

- `404 Not Found`: Nenhuma campanha elegível para o CNPJ

  

### Lógica de Seleção de Campanha

  

O processo de seleção segue uma hierarquia de prioridade:

  

#### Passo 1: Verificar Cache Redis

  

- **Chave Redis**: `banner-autocomplete-{cnpj}`

- **TTL**: 12 horas

- Se encontrado no cache, retorna imediatamente sem consultar banco de dados

  

#### Passo 2: Buscar Campanhas Segmentadas por CNPJ

  

Se não encontrado no cache, sistema busca campanhas com segmentação específica por CNPJ:

  

**Filtros aplicados**:

- `status = 1` (ativo)

- `area = "AutoComplete"`

- `effectiveInitialDate <= dataAtual`

- `effectiveFinalDate >= dataAtual`

- Existe relacionamento em `BannersCnpjs` com o CNPJ solicitado

  

Se encontrar, essa campanha tem **prioridade máxima** e é retornada.

  

#### Passo 3: Recuperar Perfil do Cliente

  

Sistema extrai do JWT token ou consulta banco de dados:

- Lista de `sellersIds`: sellers aos quais o cliente tem acesso

- Lista de `parametersIds`: parâmetros de configuração do cliente

  

#### Passo 4: Buscar Campanhas com Regras de Segmentação

  

**Filtros aplicados**:

- `status = 1` (ativo)

- `area = "AutoComplete"`

- `effectiveInitialDate <= dataAtual`

- `effectiveFinalDate >= dataAtual`

- `bannerCnpjBanners.Count = 0` (não tem segmentação por CNPJ)

- **E uma das condições**:

- `sellerId = 00000000-0000-0000-0000-000000000000` (default, sem seller específico)

- **OU** (`sellerId` está na lista de sellers do cliente **E** existe regra com `parameterId` na lista de parâmetros do cliente)

  

#### Passo 5: Aplicar Prioridade de Seleção

  

Entre as campanhas elegíveis, sistema aplica prioridade:

  

1. **Campanhas com regras**: Prioriza campanhas que possuem `Rules.Count > 0`

2. **Campanhas sem regras**: Se não houver campanhas com regras, considera campanhas default

3. **Seleção final para AutoComplete**: Ordena por `CreationDate DESC` e seleciona a **mais recente**

  

> **💡 NOTA**: Para área AutoComplete especificamente, diferente de outras áreas, a lógica seleciona sempre a campanha mais recente entre as elegíveis.

  

#### Passo 6: Armazenar em Cache

  

Campanha selecionada é serializada e armazenada no Redis:

- **Chave**: `banner-autocomplete-{cnpj}`

- **Valor**: JSON serializado do objeto `BannerWebCustomerDto`

- **Expiração**: 12 horas (TimeSpan.FromHours(12))

  

### Regras de Negócio - Visualização

  

1. **Autenticação Obrigatória**: Apenas usuários autenticados (com JWT válido) podem visualizar campanhas

2. **Segmentação por CNPJ Prioritária**: Campanhas segmentadas por CNPJ têm precedência sobre campanhas com regras

3. **Validação de Vigência**: Campanhas fora do período de vigência são ignoradas

4. **Status Ativo Obrigatório**: Apenas campanhas com `status = 1` são consideradas

5. **Seleção por Recência**: Para AutoComplete, quando múltiplas campanhas são elegíveis, a mais recente (CreationDate) é exibida

6. **Cache por Cliente**: Cache é específico por CNPJ, não há cache global

7. **Invalidação de Cache**: Cache é limpo automaticamente quando:

- Nova campanha AutoComplete é criada

- Campanha existente é atualizada

- Campanha é deletada

- Status de campanha é alterado

  

### Performance e Otimização

  

**Cache Redis**:

- Reduz carga no banco de dados SQL Server

- Tempo de resposta: ~5-10ms (cache hit) vs ~50-150ms (cache miss)

- TTL de 12 horas balanceia entre atualização e performance

  

**Prefetch de Dados**:

- Consulta ao banco inclui `.Include(Rules)` para evitar N+1 queries

- Operação é otimizada com índices em `Status`, `Area` e datas de vigência

  

**Fallback Gracioso**:

- Se nenhuma campanha é elegível, retorna 404 sem erro

- Front-end renderiza interface normal sem campanha

  

---

  

## Requisições

  

### Resumo das APIs

  

| Método | Endpoint | Autenticação | Uso |

|--------|----------|--------------|-----|

| GET | `/marketing/v2/banner/getbannerfiltered` | JSMAdmin | Listar campanhas no admin com filtros |

| POST | `/marketing/v2/banner/post` | JSMAdmin | Criar nova campanha |

| PUT | `/marketing/v2/banner/put` | JSMAdmin | Atualizar campanha existente |

| PATCH | `/marketing/v2/banner/updatestatus` | JSMAdmin | Ativar/desativar campanha |

| PATCH | `/marketing/v2/banner/sheetfile` | JSMAdmin | Atualizar planilha de segmentação |

| DELETE | `/marketing/v2/banner/{id}` | JSMAdmin | Excluir campanha |

| GET | `/marketing/v2/banner/campaignautocomplete/{cnpj}` | Autenticado | Visualizar campanha na Loja Virtual |

  

### Detalhamento de Campos

  

#### Campos de Entrada (POST/PUT)

  

| Campo | Tipo | Obrigatório | Tamanho Máx | Descrição |

|-------|------|-------------|-------------|-----------|

| `id` | Guid | PUT apenas | - | Identificador único da campanha |

| `sellerId` | Guid | Sim | - | ID do seller ou 00000000-0000-0000-0000-000000000000 para default |

| `title` | String | Sim | 50 | Título principal da campanha |

| `subTitle` | String | Não | 68 | Subtítulo complementar |

| `tag` | String | Não | 32 | Tag/categoria para classificação |

| `titleButton` | String | Sim | 20 | Texto exibido no botão de ação |

| `callToAction` | String | Sim | 500 | Caminho relativo após `/l/` para redirecionamento |

| `area` | String | Sim | 20 | Deve ser "AutoComplete" |

| `effectiveInitialDate` | DateTime | Sim | - | Data/hora de início da vigência |

| `effectiveFinalDate` | DateTime | Não* | - | Data/hora de término da vigência (*opcional para AutoComplete) |

| `desktopImage` | File | Sim | - | Imagem PNG/JPEG (recomendado 603x349px) |

| `mobileImage` | File | Não | - | Imagem PNG/JPEG para dispositivos móveis |

| `sheetFile` | File | Não | - | Arquivo CSV com CNPJs para segmentação |

| `author` | String | Sim | 50 | Nome do criador da campanha |

| `status` | Integer | Sim | - | 0 = Inativo, 1 = Ativo |

| `order` | Integer | Sim | - | Ordem de exibição (padrão: 0) |

  

#### Campos de Saída (GET)

  

**Objeto `GetBannerDto` (Admin)**:

```

{

"id": "uuid",

"sellerId": "uuid",

"title": "string",

"subTitle": "string",

"tag": "string",

"titleButton": "string",

"area": "AutoComplete",

"order": 0,

"callToAction": "string",

"imagePath": "url-cdn",

"imageMobilePath": "url-cdn",

"sheetPath": "url-cdn",

"effectiveInitialDate": "2025-01-01T00:00:00Z",

"effectiveFinalDate": "2025-12-31T23:59:59Z",

"author": "string",

"status": 1,

"creationDate": "2025-01-01T10:00:00Z",

"inactivationDate": null,

"rules": [

{

"id": "uuid",

"parameterId": "uuid",

"status": 1,

"creationDate": "2025-01-01T10:00:00Z"

}

]

}

```

  

**Objeto `BannerWebCustomerDto` (Loja Virtual)**:

```

{

"imagePath": "https://cdn.exemplo.com/banner.jpg",

"imageMobilePath": "https://cdn.exemplo.com/banner-mobile.jpg",

"callToAction": "/produtos/busca?sellerName=Example",

"order": 0,

"title": "Campanha Exemplo",

"tag": "Novidades",

"subTitle": "Confira os produtos em destaque",

"titleButton": "Ver produtos"

}

```

  

### Exemplos de Requisições

  

#### Exemplo 1: Listar Campanhas AutoComplete no Admin

  

**Request**:

```http

GET /marketing/v2/banner/getbannerfiltered?pageSize=10&pageIndex=1&area=AutoComplete&status=1

Authorization: Bearer {jwt-token-admin}

```

  

**Response 200 OK**:

```json

{

"pageIndex": 1,

"pageSize": 10,

"totalPages": 1,

"totalCount": 3,

"count": 3,

"items": [

{

"id": "a1b2c3d4-...",

"title": "Campanha Vedacit",

"status": 1,

"area": "AutoComplete",

...

}

]

}

```

  

#### Exemplo 2: Criar Campanha de Busca

  

**Request**:

```http

POST /marketing/v2/banner/post

Content-Type: multipart/form-data

Authorization: Bearer {jwt-token-admin}

  

------WebKitFormBoundary...

Content-Disposition: form-data; name="sellerId"

  

00000000-0000-0000-0000-000000000000

------WebKitFormBoundary...

Content-Disposition: form-data; name="title"

  

Promoção Verão 2025

------WebKitFormBoundary...

Content-Disposition: form-data; name="area"

  

AutoComplete

------WebKitFormBoundary...

Content-Disposition: form-data; name="callToAction"

  

/produtos/busca?category=verao

------WebKitFormBoundary...

Content-Disposition: form-data; name="titleButton"

  

Ver produtos

------WebKitFormBoundary...

Content-Disposition: form-data; name="desktopImage"; filename="banner.jpg"

Content-Type: image/jpeg

  

{binary-data}

------WebKitFormBoundary...

```

  

**Response 201 Created**:

```json

{

"id": "new-uuid-created",

"title": "Promoção Verão 2025",

...

}

```

  

#### Exemplo 3: Visualizar Campanha na Loja (Cliente)

  

**Request**:

```http

GET /marketing/v2/banner/campaignautocomplete/12345678000190

Authorization: Bearer {jwt-token-cliente}

```

  

**Response 200 OK**:

```json

{

"imagePath": "https://cdn.juntossomosmaisi.com.br/marketing/banners/abc123.jpg",

"imageMobilePath": "https://cdn.juntossomosmaisi.com.br/marketing/banners/abc123-mobile.jpg",

"callToAction": "/produtos/busca?category=verao",

"order": 0,

"title": "Promoção Verão 2025",

"tag": "Promoção",

"subTitle": "Aproveite as ofertas de verão",

"titleButton": "Ver produtos"

}

```

  

**Response 404 Not Found** (nenhuma campanha elegível):

```json

{

"success": false,

"errors": [

"Não foi possível encontrar banners da área AutoComplete para o CNPJ 12345678000190."

]

}

```

  

---

  

## Contexto do Banco de Dados

  

### Diagrama de Relacionamento

  

```

┌─────────────────┐ ┌──────────────────┐

│ Banners │────1:N──│ BannersRules │

│ │ │ │

│ Id (PK) │ │ Id (PK) │

│ SellerId │ │ BannerId (FK) │

│ Title │ │ ParameterId │

│ Area │ │ Status │

│ ... │ │ CreationDate │

└─────────────────┘ └──────────────────┘

│

│ N:M

│

┌───────▼──────────────────────────┐

│ BannersCnpjs (Junção) │

│ │

│ BannerId (PK, FK) │

│ BannerCnpjId (PK, FK) │

└───────┬──────────────────────────┘

│

│ N:1

│

┌───────▼──────────┐

│ Cnpjs │

│ │

│ Id (PK) │

│ Cnpj │

└──────────────────┘

```

  

### Tabela: Banners

  

**Schema**: `dbo.Banners`

  

**Descrição**: Tabela principal que armazena todas as campanhas/banners do sistema, incluindo campanhas de busca (área AutoComplete).

  

| Campo | Tipo | Nulo | Default | Descrição |

|-------|------|------|---------|-----------|

| `Id` | UNIQUEIDENTIFIER | Não | - | Identificador único da campanha (chave primária) |

| `SellerId` | UNIQUEIDENTIFIER | Não | - | ID do seller associado. Usar `00000000-0000-0000-0000-000000000000` para campanha default |

| `Title` | NVARCHAR(50) | Não | - | Título da campanha exibido na interface |

| `SubTitle` | NVARCHAR(80) | Sim | '' | Subtítulo complementar da campanha |

| `Tag` | NVARCHAR(32) | Sim | '' | Tag/categoria para classificação da campanha |

| `TitleButton` | NVARCHAR(20) | Sim | '' | Texto exibido no botão de call-to-action |

| `Area` | NVARCHAR(20) | Não | '' | Área de exibição. Para campanhas de busca: **"AutoComplete"** |

| `Order` | INT | Não | 0 | Ordem de exibição (menor valor = maior prioridade) |

| `CallToAction` | NVARCHAR(500) | Não | '' | URL relativa para redirecionamento ao clicar na campanha |

| `ImagePath` | NVARCHAR(900) | Não | - | Caminho completo da imagem desktop no CDN |

| `ImageMobilePath` | NVARCHAR(900) | Não | - | Caminho completo da imagem mobile no CDN |

| `SheetPath` | NVARCHAR(900) | Sim | '' | Caminho para planilha CSV de segmentação por CNPJ |

| `EffectiveInitialDate` | DATETIME | Não | - | Data/hora de início da vigência da campanha |

| `EffectiveFinalDate` | DATETIME | Não | - | Data/hora de término da vigência da campanha |

| `Author` | NVARCHAR(50) | Não | - | Nome do usuário que criou a campanha |

| `Status` | INT | Não | - | Status da campanha: **0** = Inativo, **1** = Ativo |

| `CreationDate` | DATETIME | Não | GETDATE() | Data/hora de criação do registro |

| `InactivationDate` | DATETIME | Sim | NULL | Data/hora de inativação (quando status mudou para 0) |

  

**Índices**:

- Primary Key: `PK_Banners` em `Id`

- Índice não clusterizado recomendado em: `(Area, Status, EffectiveInitialDate, EffectiveFinalDate)` para otimizar consultas de visualização

  

**Constraints**:

- Validação de negócio: Apenas uma campanha com `Area = 'AutoComplete'` pode ter `Status = 1` simultaneamente (validado na aplicação)

  

### Tabela: BannersRules

  

**Schema**: `dbo.BannersRules`

  

**Descrição**: Tabela de relacionamento que armazena as regras de segmentação das campanhas. Cada regra define um parâmetro que deve estar presente no perfil do cliente para que a campanha seja exibida.

  

| Campo | Tipo | Nulo | Default | Descrição |

|-------|------|------|---------|-----------|

| `Id` | UNIQUEIDENTIFIER | Não | NEWID() | Identificador único da regra (chave primária) |

| `BannerId` | UNIQUEIDENTIFIER | Não | - | ID da campanha (chave estrangeira para `Banners.Id`) |

| `ParameterId` | UNIQUEIDENTIFIER | Não | - | ID do parâmetro de cliente que deve estar presente |

| `Status` | INT | Não | - | Status da regra: **0** = Inativa, **1** = Ativa |

| `CreationDate` | DATETIME | Não | - | Data/hora de criação da regra |

  

**Índices**:

- Primary Key: `PK_BannersRules` em `Id`

- Foreign Key: `FK_BannersRules_Banners` em `BannerId` → `Banners.Id`

- Índice recomendado em: `(BannerId, ParameterId, Status)` para otimizar joins

  

**Relacionamentos**:

- N:1 com `Banners`: Uma campanha pode ter múltiplas regras

  

**Lógica de Uso**:

- Cliente visualiza campanha SE:

- `SellerId` da campanha está nos sellers do cliente **E**

- **Pelo menos uma** regra tem `ParameterId` presente nos parâmetros do cliente **E**

- `Status` da regra = 1

  

### Tabela: Cnpjs

  

**Schema**: `dbo.Cnpjs`

  

**Descrição**: Tabela que armazena CNPJs utilizados para segmentação específica de campanhas.

  

| Campo | Tipo | Nulo | Default | Descrição |

|-------|------|------|---------|-----------|

| `Id` | UNIQUEIDENTIFIER | Não | - | Identificador único (chave primária) |

| `Cnpj` | NVARCHAR(14) | Não | - | CNPJ sem formatação (apenas números) |

  

**Índices**:

- Primary Key: `PK_Cnpjs` em `Id`

- Índice único recomendado em: `Cnpj` para garantir unicidade e otimizar buscas

  

**Observações**:

- CNPJs são extraídos de planilhas CSV enviadas via `PATCH /sheetfile`

- Formato esperado: 14 dígitos numéricos sem pontuação

  

### Tabela: BannersCnpjs

  

**Schema**: `dbo.BannersCnpjs`

  

**Descrição**: Tabela de junção muitos-para-muitos entre campanhas e CNPJs. Estabelece segmentação específica por CNPJ.

  

| Campo | Tipo | Nulo | Default | Descrição |

|-------|------|------|---------|-----------|

| `BannerId` | UNIQUEIDENTIFIER | Não | - | ID da campanha (chave primária composta, FK para `Banners.Id`) |

| `BannerCnpjId` | UNIQUEIDENTIFIER | Não | - | ID do registro de CNPJ (chave primária composta, FK para `Cnpjs.Id`) |

  

**Índices**:

- Primary Key: `PK_BannersCnpjs` em `(BannerId, BannerCnpjId)`

- Foreign Key: `FK_BannersCnpjs_Banners` em `BannerId` → `Banners.Id`

- Foreign Key: `FK_BannersCnpjs_Cnpjs` em `BannerCnpjId` → `Cnpjs.Id`

  

**Relacionamentos**:

- N:1 com `Banners`: Uma campanha pode ser segmentada para múltiplos CNPJs

- N:1 com `Cnpjs`: Um CNPJ pode ter acesso a múltiplas campanhas

  

**Prioridade de Segmentação**:

- Segmentação por CNPJ (via esta tabela) tem **prioridade máxima** sobre segmentação por regras

- Se uma campanha tem registros em `BannersCnpjs`, ela é exibida **apenas** para os CNPJs listados

- Se uma campanha **não** tem registros em `BannersCnpjs`, ela utiliza segmentação por regras ou é default (sem segmentação)

  

### Queries Comuns

  

#### Query 1: Listar campanhas AutoComplete ativas

  

```sql

SELECT

b.Id,

b.Title,

b.Area,

b.Status,

b.EffectiveInitialDate,

b.EffectiveFinalDate,

b.CreationDate

FROM dbo.Banners b

WHERE

b.Area = 'AutoComplete'

AND b.Status = 1

AND b.EffectiveInitialDate <= GETDATE()

AND (b.EffectiveFinalDate >= GETDATE() OR b.EffectiveFinalDate IS NULL)

ORDER BY b.CreationDate DESC;

```

  

#### Query 2: Verificar regras de segmentação de uma campanha

  

```sql

SELECT

br.Id,

br.ParameterId,

br.Status,

br.CreationDate

FROM dbo.BannersRules br

WHERE

br.BannerId = 'uuid-da-campanha'

AND br.Status = 1;

```

  

#### Query 3: Listar CNPJs autorizados para uma campanha

  

```sql

SELECT

c.Cnpj

FROM dbo.BannersCnpjs bc

INNER JOIN dbo.Cnpjs c ON bc.BannerCnpjId = c.Id

WHERE bc.BannerId = 'uuid-da-campanha';

```

  

#### Query 4: Identificar campanhas conflitantes (mais de 1 ativa)

  

```sql

SELECT

b.Id,

b.Title,

b.Status,

b.CreationDate

FROM dbo.Banners b

WHERE

b.Area = 'AutoComplete'

AND b.Status = 1

AND b.EffectiveInitialDate <= GETDATE()

AND (b.EffectiveFinalDate >= GETDATE() OR b.EffectiveFinalDate IS NULL)

ORDER BY b.CreationDate DESC;

  

-- Se retornar mais de 1 registro, há conflito

```

  

### Troubleshooting - Banco de Dados

  

#### Problema: Campanha não aparece para nenhum cliente

  

**Verificações**:

1. Confirmar `Status = 1` na tabela `Banners`

2. Verificar se `Area = 'AutoComplete'`

3. Validar período de vigência (EffectiveInitialDate e EffectiveFinalDate)

4. Checar se existe outra campanha ativa conflitante

5. Se segmentada por CNPJ, verificar se há registros em `BannersCnpjs`

6. Se segmentada por regras, verificar se há regras ativas em `BannersRules`

  

#### Problema: Campanha aparece para clientes errados

  

**Verificações**:

1. Verificar registros em `BannersCnpjs` para segmentação específica

2. Analisar `BannersRules` e comparar com parâmetros dos clientes

3. Confirmar `SellerId` da campanha versus sellers dos clientes

4. Verificar cache Redis: limpar chave `banner-autocomplete-{cnpj}` do cliente afetado

  

#### Problema: Alteração não reflete imediatamente

  

**Causa**: Cache Redis com TTL de 12 horas

  

**Solução**:

1. Conectar ao Redis

2. Executar: `DEL banner-autocomplete-{cnpj}` para cliente específico

3. Ou executar: `KEYS banner-autocomplete-*` e deletar todas as chaves para limpeza geral

  

#### Problema: Performance lenta na visualização

  

**Verificações**:

1. Verificar taxa de hit do cache Redis (deve ser > 90%)

2. Analisar query plan das consultas de seleção de banner

3. Confirmar existência de índices recomendados

4. Verificar se há muitos registros em `BannersRules` (JOIN pesado)

  

### Manutenção e Monitoramento

  

#### Métricas Recomendadas

  

1. **Taxa de Hit de Cache**: Percentual de requisições atendidas pelo Redis

2. **Tempo de Resposta**: P50, P95, P99 das APIs de visualização

3. **Campanhas Ativas**: Alertar se > 1 campanha AutoComplete ativa

4. **Taxa de 404**: Percentual de clientes sem campanha elegível

  

#### Scripts de Manutenção

  

**Limpar campanhas expiradas há mais de 6 meses**:

```sql

DELETE FROM dbo.BannersRules

WHERE BannerId IN (

SELECT Id FROM dbo.Banners

WHERE Area = 'AutoComplete'

AND Status = 0

AND EffectiveFinalDate < DATEADD(MONTH, -6, GETDATE())

);

  

DELETE FROM dbo.BannersCnpjs

WHERE BannerId IN (

SELECT Id FROM dbo.Banners

WHERE Area = 'AutoComplete'

AND Status = 0

AND EffectiveFinalDate < DATEADD(MONTH, -6, GETDATE())

);

  

DELETE FROM dbo.Banners

WHERE Area = 'AutoComplete'

AND Status = 0

AND EffectiveFinalDate < DATEADD(MONTH, -6, GETDATE());

```

  

**Auditoria de campanhas ativas**:

```sql

SELECT

b.Id,

b.Title,

b.Author,

b.CreationDate,

b.EffectiveInitialDate,

b.EffectiveFinalDate,

(SELECT COUNT(*) FROM dbo.BannersRules WHERE BannerId = b.Id) AS RulesCount,

(SELECT COUNT(*) FROM dbo.BannersCnpjs WHERE BannerId = b.Id) AS CnpjsCount

FROM dbo.Banners b

WHERE b.Area = 'AutoComplete' AND b.Status = 1

ORDER BY b.CreationDate DESC;

```

  

---

  

## Glossário Técnico

  

- **AutoComplete**: Área específica de banner correspondente à barra de busca da Loja Virtual

- **Banner**: Termo técnico para campanha promocional visual no sistema

- **Call to Action (CTA)**: Ação/redirecionamento configurado quando usuário interage com a campanha

- **CNPJ**: Cadastro Nacional da Pessoa Jurídica, identificador único de empresas no Brasil

- **JWT**: JSON Web Token, método de autenticação utilizado nas APIs

- **ParameterId**: Identificador de parâmetro de configuração de cliente utilizado para segmentação

- **Redis**: Sistema de cache em memória utilizado para otimização de performance

- **SellerId**: Identificador único de vendedor/fornecedor no sistema

- **Segmentação**: Processo de definir quais clientes visualizam determinada campanha

- **TTL (Time To Live)**: Tempo de expiração de um registro em cache

- **Vigência**: Período de validade de uma campanha (entre EffectiveInitialDate e EffectiveFinalDate)

  

---

  

## Suporte e Contato

  

Para dúvidas técnicas, problemas ou sugestões relacionadas a esta funcionalidade, entre em contato com:

  

- **Equipe de Desenvolvimento**: Time de Marketing/Comunicação

- **Documentação Técnica**: Este documento

- **Documentação de Usuário**: Portal Admin (guia não técnico)

  

---

  

**Última Atualização**: Janeiro 2025

**Versão da API**: v2

**Versão do Documento**: 1.0
