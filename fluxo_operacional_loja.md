# Fluxo Operacional da Loja

## Objetivo

Documentar o fluxo operacional da loja dentro da nova arquitetura do CardápioFast, considerando a separação entre:

* conta administrativa (`cf_members`)
* loja (`cf_stores`)
* operação da loja (`cf_store_*`)

---

## Visão Geral

A operação da loja começa quando um `cf_member` acessa uma `cf_store` na qual ele possui vínculo em `cf_store_members`.

A partir daí, todo o gerenciamento operacional acontece no contexto da loja ativa.

---

## Fluxo Principal

```mermaid
flowchart TD
    A[cf_member faz login] --> B[Seleciona ou acessa uma loja]
    B --> C[Valida vínculo em cf_store_members]
    C --> D[Carrega contexto da cf_store]

    D --> E[Gerencia catálogo]
    D --> F[Gerencia configurações]
    D --> G[Gerencia pedidos]
    D --> H[Gerencia clientes]

    E --> E1[cf_store_categories]
    E --> E2[cf_store_products]
    E --> E3[cf_store_product_extras]
    E --> E4[cf_store_additionals]
    E --> E5[cf_store_additional_items]
    E --> E6[cf_store_category_additionals]
    E --> E7[cf_store_product_additionals]

    F --> F1[cf_store_addresses]
    F --> F2[cf_store_seo]
    F --> F3[cf_store_hours]
    F --> F4[cf_store_coupons]
    F --> F5[cf_store_cloud_messages]

    G --> G1[cf_store_orders]
    G1 --> G2[cf_store_order_items]

    H --> H1[clientes do pedido / histórico]
```

---

## Fluxo de Acesso

```mermaid
flowchart LR
    Member[cf_member] --> Access[cf_store_members]
    Access --> Store[cf_store]

    Access --> Role{role}
    Role --> Owner[owner]
    Role --> Admin[admin]
    Role --> Editor[editor]
    Role --> Viewer[viewer]
```

### Regras

* `owner`: acesso total à loja
* `admin`: gerencia operação, pedidos, catálogo e configurações
* `editor`: gerencia catálogo e partes operacionais permitidas
* `viewer`: apenas visualização

---

## Fluxo do Catálogo

```mermaid
flowchart TD
    A[cf_store] --> B[cf_store_categories]
    B --> C[cf_store_products]
    C --> D[cf_store_product_extras]

    A --> E[cf_store_additionals]
    E --> F[cf_store_additional_items]

    B --> G[cf_store_category_additionals]
    C --> H[cf_store_product_additionals]
```

### Explicação

* a loja possui categorias
* as categorias agrupam produtos
* os produtos podem ter extras próprios
* a loja também pode ter grupos de adicionais
* esses grupos podem ser vinculados a categorias ou produtos

---

## Fluxo de Configuração da Loja

```mermaid
flowchart TD
    A[cf_store] --> B[cf_store_addresses]
    A --> C[cf_store_seo]
    A --> D[cf_store_hours]
    A --> E[cf_store_coupons]
    A --> F[cf_store_cloud_messages]
```

### Responsabilidades

* `cf_store_addresses`: endereço físico/comercial e dados de entrega
* `cf_store_seo`: dados públicos da loja, SEO, branding e configurações visíveis
* `cf_store_hours`: horários de funcionamento
* `cf_store_coupons`: regras promocionais da loja
* `cf_store_cloud_messages`: integrações de push/notificação

---

## Fluxo do Pedido

```mermaid
flowchart TD
    A[Cliente realiza pedido] --> B[cf_store_orders]
    B --> C[cf_store_order_items]
    B --> D[status do pedido]
    B --> E[dados financeiros]
    B --> F[dados do cliente]
```

### Estrutura esperada

#### `cf_store_orders`

Concentra:

* loja
* identificação do pedido
* status
* totais
* dados do cliente
* dados de entrega
* forma de pagamento

#### `cf_store_order_items`

Concentra:

* produto do pedido
* quantidade
* preço unitário
* subtotal do item
* observações
* adicionais serializados ou normalizados futuramente

---

## Fluxo Operacional Diário

```mermaid
flowchart TD
    A[Login do membro] --> B[Seleciona loja]
    B --> C[Abre dashboard da loja]

    C --> D[Atualiza catálogo]
    C --> E[Abre ou fecha loja]
    C --> F[Configura cupom ou horário]
    C --> G[Acompanha pedidos]
    C --> H[Consulta métricas]

    G --> I[Recebe pedido]
    I --> J[Confirma pedido]
    J --> K[Prepara pedido]
    K --> L[Sai para entrega ou retirada]
    L --> M[Finaliza pedido]
```

---

## Separação de Responsabilidades

### Pertence ao `cf_member`

* login
* senha
* conta administrativa
* plano
* pagamento de assinatura
* afiliado

### Pertence ao `cf_store`

* catálogo
* endereço da loja
* SEO
* horário
* pedido
* cupom
* notificações
* operação comercial

### Pertence ao vínculo `cf_store_members`

* permissão do membro dentro da loja
* papel de acesso
* status do vínculo

---

## Regra Central do Sistema

> Tudo que representa operação da loja deve nascer de `store_id`.

Ou seja:

* categoria pertence à loja
* produto pertence à loja
* pedido pertence à loja
* cupom pertence à loja
* configuração pertence à loja

O `cf_member` não é mais a raiz do catálogo e da operação.

---

## Resumo Final

O fluxo operacional correto é:

1. o membro autentica
2. entra em uma loja à qual possui acesso
3. o sistema valida o papel dele em `cf_store_members`
4. toda a operação passa a ocorrer dentro do contexto de `cf_store`
5. catálogo, pedidos e configurações sempre dependem da loja

Isso permite:

* múltiplas lojas por usuário
* múltiplos usuários por loja
* permissões por loja
* escalabilidade real do sistema
