# Imperium Moda Social Masculina — Design Spec

## Visão Geral

Site institucional premium para a loja **Imperium Moda Social Masculina** (Taquaritinga - SP). Composto por uma **landing page** focada em conversão via WhatsApp e uma **página de catálogo** com produtos gerenciados pelo painel admin existente (`cardapio-admin`).

## Dados da Empresa

- **Nome:** Imperium Moda Social Masculina
- **Endereço:** Rua dos Domingues, 534 - Centro, Taquaritinga - SP, 15900-023
- **WhatsApp:** (16) 99161-1681
- **Instagram:** @imperium.modasocial

## Decisões de Design

| Decisão | Escolha |
|---|---|
| Cor de destaque | Dourado fosco (#d4af37) |
| Paleta base | Preto, grafite, cinza escuro, branco |
| Hero | Full-screen imersivo (overlay escuro, texto centralizado) |
| Catálogo | Página separada, grid de cards com filtros por categoria |
| Layout catálogo | Grid 3 colunas (desktop), 2 (tablet), 1 (mobile) |
| Dados de contato | Firestore (businessInfo), editável pelo admin |
| Fontes | Playfair Display (títulos) + Inter (corpo) |
| Abordagem técnica | HTML estático (padrão Marieta) |
| Preços | Numérico ou "Sob consulta" (preco: null) |

## Arquitetura

### Estrutura de Arquivos (Frontend)

```
imperium-moda-social-masculina/
├── index.html          # Landing page
├── catalogo.html       # Página de catálogo
├── css/
│   └── style.css       # Estilos compartilhados
├── js/
│   └── app.js          # Lógica compartilhada (Firestore, UI)
└── imagens/            # Assets locais (logo, ícones, fallback)
```

### Fluxo de Dados

```
Admin (cardapio-admin) → Firestore → REST API → Frontend (index.html / catalogo.html)
```

O frontend busca dados via Firestore REST API (`restaurants/imperium-moda-social/data/roupas` e `restaurants/imperium-moda-social/data/businessInfo`). Fallback para dados locais hardcoded quando offline ou em protocolo `file://`.

### Firestore — Coleções

**Slug do cliente:** `imperium-moda-social`

#### `restaurants/imperium-moda-social/data/roupas`

```json
{
  "content": [
    {
      "id": "camisa-social-azul-001",
      "nome": "Camisa Social Azul Marinho levemente Acetinada",
      "descricao": "Super fácil de passar, não amassa durante o uso...",
      "preco": 229.90,
      "categoria": "camisas-sociais",
      "destaque": false,
      "observacoes": "",
      "images": [
        { "url": "https://...", "storagePath": "restaurants/imperium-moda-social/roupas/..." }
      ]
    }
  ]
}
```

Campos:
- `id` (string) — identificador slug gerado automaticamente
- `nome` (string) — nome do produto
- `descricao` (string) — descrição completa
- `preco` (number | null) — preço em reais; `null` = "Sob consulta"
- `categoria` (string) — slug da categoria para filtro
- `destaque` (boolean) — reservado para uso futuro (ex: badges, ordenação prioritária no catálogo)
- `observacoes` (string) — texto auxiliar (ex: "Consulte disponibilidade")
- `images` (array) — até 6 objetos `{ url, storagePath }`; vazio quando sem foto

#### `restaurants/imperium-moda-social/data/businessInfo`

```json
{
  "content": {
    "name": "Imperium Moda Social Masculina",
    "city": "Taquaritinga",
    "slogan": "Elegância masculina para quem quer presença, estilo e sofisticação.",
    "tagline": "Moda Social Masculina Premium",
    "whatsapp": "(16) 99161-1681",
    "whatsappNumber": "5516991611681",
    "phone": "(16) 99161-1681",
    "address": "Rua dos Domingues, 534",
    "neighborhood": "Centro",
    "cityState": "Taquaritinga - SP",
    "cep": "15900-023",
    "hours": {
      "weekdays": "09:00 - 18:00",
      "saturday": "09:00 - 13:00"
    },
    "instagram": "@imperium.modasocial",
    "facebook": "",
    "googleMapsEmbed": "",
    "googleMapsLink": ""
  }
}
```

### Categorias de Produto

| Slug | Label |
|---|---|
| `camisas-sociais` | Camisas Sociais |
| `camisetas-polos` | Camisetas & Polos |
| `calcas-bermudas` | Calças & Bermudas |
| `ternos-costumes` | Ternos & Costumes |
| `acessorios` | Acessórios |
| `kits-combinacoes` | Kits & Combinações |
| `jaquetas-sueteres` | Jaquetas & Suéteres |
| `perfumes` | Perfumes |

## Landing Page — Seções

### 1. Header / Navbar

- Logo "IMPERIUM" em Playfair Display, dourado, à esquerda
- Menu: Início | A Marca | Estilos | Diferenciais | Avaliações | Catálogo | Contato
- Botão dourado "WhatsApp" à direita
- Fixo no topo, fundo transparente no hero → fundo escuro sólido ao scrollar
- Mobile: hamburguer menu

### 2. Hero Full-Screen

- Altura: 100vh
- Fundo: imagem de moda masculina com overlay gradiente escuro (`rgba(0,0,0,0.6)`)
- Conteúdo centralizado:
  - Label superior: `IMPERIUM MODA SOCIAL MASCULINA` (dourado, espaçado, uppercase, 12px)
  - Headline (H1): **"Elegância masculina para quem quer presença, estilo e sofisticação."**
  - Subtítulo: *"A referência em moda social masculina em Taquaritinga. Peças selecionadas para o homem que valoriza qualidade, conforto e uma imagem impecável."*
  - CTA primário: `Fale pelo WhatsApp` (fundo dourado, texto escuro)
  - CTA secundário: `Ver Localização` (outline dourado)

### 3. Seção Institucional — "A Marca"

- Fundo: grafite escuro (#1a1a1a)
- Detalhe: linha vertical dourada à esquerda do texto
- Título (H2): **"Mais do que roupas. Uma experiência de estilo."**
- Copy: "A Imperium existe para transformar a maneira como o homem se apresenta. Cada peça é escolhida para oferecer não apenas qualidade e conforto, mas confiança. Referência em moda masculina em Taquaritinga, a loja reúne o que há de melhor em tecidos, caimento e acabamento para quem entende que a imagem pessoal é o primeiro passo para conquistar respeito e admiração."

### 4. Seção "Para quem é a Imperium"

- Fundo: preto (#0a0a0a)
- Título (H2): **"Para o homem que não abre mão de presença."**
- 5 cards (grid responsivo) com ícone dourado + texto branco:
  1. "Quer se vestir melhor no dia a dia sem perder o conforto"
  2. "Busca peças para trabalho, reuniões e eventos com elegância"
  3. "Valoriza qualidade e durabilidade em cada detalhe"
  4. "Quer elevar sua imagem pessoal e autoestima"
  5. "Procura moda masculina com bom gosto e sofisticação"

### 5. Seção de Estilos / Linhas

- Fundo: grafite (#1a1a1a)
- Título (H2): **"Estilos para cada momento."**
- Grid de 6 cards visuais (3x2 desktop, 2x3 tablet, 1x6 mobile):
  - **Moda Social** — "Presença e autoridade em cada ocasião formal."
  - **Ternos & Costumes** — "Corte impecável para momentos que exigem o melhor."
  - **Esporte Fino** — "O equilíbrio perfeito entre elegância e descontração."
  - **Casual Sofisticado** — "Conforto com classe para o dia a dia."
  - **Acessórios** — "Os detalhes que completam o look perfeito."
  - **Kits & Combinações** — "Looks completos pensados para facilitar sua escolha."
- Cada card: espaço para imagem (240px) + nome + descrição
- Os cards de estilo são conceituais/aspiracionais, não mapeiam 1:1 com categorias do catálogo. Link de cada card aponta para `/catalogo.html` (catálogo geral)
- Hover: leve scale + borda dourada

### 6. Seção de Diferenciais

- Fundo: preto (#0a0a0a)
- Título (H2): **"Por que escolher a Imperium?"**
- Grid 3x2 (desktop), 2x3 (tablet), 1x6 (mobile):
  1. **Atendimento Personalizado** — "Ajudamos você a encontrar a peça ideal para cada ocasião."
  2. **Qualidade Premium** — "Tecidos selecionados com acabamento impecável."
  3. **Especialistas em Moda Masculina** — "Foco exclusivo no homem que valoriza elegância."
  4. **Variedade de Estilos** — "Do social ao casual, para todas as ocasiões."
  5. **Localização Central** — "No coração de Taquaritinga, fácil de encontrar."
  6. **Preço Justo** — "Sofisticação acessível, sem abrir mão da qualidade."
- Cada item: ícone SVG dourado (32px) + título branco + descrição cinza

### 7. Seção de Prova Social

- Fundo: grafite (#1a1a1a)
- Título (H2): **"O que nossos clientes dizem."**
- 4 cards de depoimento com aspas douradas:
  1. *"Atendimento nota 10. Me ajudaram a montar o look completo para minha formatura."* — R.S.
  2. *"Melhor loja de moda masculina da região. Qualidade incrível."* — M.A.
  3. *"Sempre encontro o que preciso. Variedade e bom gosto."* — L.F.
  4. *"As camisas são espetaculares. Não amassam e o caimento é perfeito."* — P.H.
- Estrelas douradas (5/5) em cada card

### 8. Seção CTA Intermediária

- Fundo: gradiente dourado escuro (linear-gradient de #1a1510 para #2a2018) com textura sutil
- Título (H2): **"Encontre a peça perfeita para o seu estilo."**
- Subtítulo: *"Fale com a equipe Imperium e descubra peças selecionadas para você."*
- Botão grande: `Chamar no WhatsApp` (fundo dourado, texto escuro, hover com brilho)

### 9. Seção Instagram

- Fundo: preto (#0a0a0a)
- Título (H2): **"Acompanhe no Instagram"**
- Subtítulo: *"Novidades, looks e inspirações para o homem moderno."*
- Grid de 6 placeholders quadrados para fotos (imagens estáticas, não API do Instagram)
- Handle: `@imperium.modasocial`
- Botão: `Seguir no Instagram` (link externo)

### 10. Seção Contato e Localização

- Fundo: grafite (#1a1a1a)
- Título (H2): **"Visite a Imperium."**
- Layout: dois blocos lado a lado (desktop), empilhados (mobile)
  - Bloco esquerdo: dados de contato (endereço, WhatsApp, horário) + botões "Chamar no WhatsApp" e "Como Chegar"
  - Bloco direito: mapa embed (Google Maps) ou placeholder
- Dados carregados do Firestore (businessInfo)
- CTA final: **"Vista sua melhor versão. A Imperium espera por você."**

### 11. Footer

- Fundo: preto (#050505)
- Logo "IMPERIUM" pequena em dourado
- Dados resumidos: endereço, WhatsApp, Instagram
- Links do menu
- Copyright: "© 2026 Imperium Moda Social Masculina. Todos os direitos reservados."

### Elemento Fixo

- Botão flutuante WhatsApp (canto inferior direito)
- Ícone WhatsApp branco sobre fundo verde (#25D366)
- Borda-raio circular, sombra sutil
- Link: `https://wa.me/5516991611681?text=Olá! Gostaria de conhecer as peças da Imperium.`

## Página de Catálogo

### Estrutura

1. **Header** — mesmo navbar da landing page
2. **Banner compacto** — título "Catálogo" + subtítulo, fundo escuro, altura ~200px
3. **Filtros** — pills/tags horizontais com as 8 categorias + "Todos" (ativo por padrão)
4. **Grid de produtos** — cards renderizados dinamicamente a partir do Firestore
5. **Modal de produto** — detalhes ao clicar em um card
6. **Footer** — mesmo da landing page

### Card de Produto

- Imagem principal (ou placeholder cinza escuro com ícone de camisa se sem foto)
- Nome do produto (max 2 linhas, truncado)
- Preço em dourado ou "Sob consulta" em itálico dourado
- Hover: leve elevação + borda dourada sutil
- Click: abre modal

### Modal de Produto

- Overlay escuro semi-transparente
- Container centralizado com fundo grafite
- Galeria de fotos (carrossel se múltiplas, imagem única se 1, placeholder se 0)
- Nome do produto (H3)
- Descrição completa
- Preço ou "Sob consulta"
- Observações (se houver, em texto menor)
- Botão: `Perguntar pelo WhatsApp` → abre `https://wa.me/5516991611681?text=Olá! Vi o produto "{nome}" no site e gostaria de mais informações.`
- Botão fechar (X) no canto superior direito
- Fecha ao clicar fora do container ou pressionar ESC

### Filtro por Categoria

- URL param: `catalogo.html?cat=camisas-sociais`
- Ao carregar, se `?cat` presente, ativa o filtro correspondente
- Filtro "Todos" mostra todos os produtos
- Animação suave de transição ao filtrar (fade ou slide)

## Alterações no cardapio-admin

### Arquivos a modificar

1. **`src/pages/AdminRestaurantes.jsx`**
   - Adicionar opção `roupas` no radio de tipo ao criar novo cliente
   - Inicializar docs `roupas` (content: []) e `businessInfo` (com defaults para loja de roupas)

2. **`src/pages/Dashboard.jsx`**
   - Quando `type === 'roupas'`: exibir botões "Catálogo" e "Informações"

3. **`src/App.jsx`**
   - Adicionar rota: `/restaurante/:slug/roupas` → RoupasEditor

4. **Nova página: `src/pages/RoupasEditor.jsx`**
   - Lista de produtos com drag-and-drop (@dnd-kit) para reordenar
   - Formulário por produto:
     - `nome` (text input)
     - `descricao` (textarea)
     - `preco` (number input) + checkbox "Sob consulta" (quando marcado, preco = null)
     - `categoria` (select com as 8 categorias)
     - `destaque` (toggle)
     - `observacoes` (text input)
     - Upload de imagens (até 6) — mesmo padrão do VeiculosEditor (Firebase Storage)
   - Botão adicionar produto
   - Botão remover produto (com confirmação)
   - Salva em `restaurants/{slug}/data/roupas`

### Script de Seed

Script Node.js para inserir os 33 produtos iniciais no Firestore:
- Lê os dados do catálogo (hardcoded no script, baseado na planilha)
- Gera IDs slug a partir do nome
- Define categoria com base no mapeamento manual
- `preco`: valor numérico ou `null` para "Sob consulta"
- `images`: array vazio (fotos adicionadas manualmente depois)
- Executa via `node seed-imperium.js`

## SEO

### index.html

- **Title:** `Imperium Moda Social Masculina | Loja de Moda Masculina em Taquaritinga - SP`
- **Meta description:** `Moda social masculina premium em Taquaritinga. Camisas, ternos, calças alfaiataria e acessórios com qualidade e elegância. Visite a loja no centro ou fale pelo WhatsApp.`
- **H1:** headline do hero
- **H2:** um por seção
- **Schema.org:** JSON-LD tipo LocalBusiness com nome, endereço, telefone, coordenadas
- Termos naturais no conteúdo: "moda masculina em Taquaritinga", "loja de roupa masculina em Taquaritinga", "moda social masculina em Taquaritinga"

### catalogo.html

- **Title:** `Catálogo | Imperium Moda Social Masculina - Taquaritinga SP`
- **Meta description:** `Confira o catálogo completo da Imperium: camisas sociais, ternos, calças alfaiataria, acessórios e mais. Moda masculina premium em Taquaritinga.`

## Direção Visual Detalhada

### Paleta

| Uso | Cor | Hex |
|---|---|---|
| Fundo principal | Preto | #0a0a0a |
| Fundo alternado | Grafite | #1a1a1a |
| Fundo cards | Cinza escuro | #111111 |
| Texto principal | Branco | #ffffff |
| Texto secundário | Cinza | #999999 |
| Destaque / CTAs | Dourado fosco | #d4af37 |
| Dourado escuro | Hover/variação | #b8960c |
| Dourado claro | Detalhes sutis | #c9a227 |
| WhatsApp fixo | Verde WhatsApp | #25D366 |

### Tipografia

- **Títulos (H1, H2):** Playfair Display, peso 700, cor branca
- **Subtítulos / labels:** Playfair Display, peso 400, cor dourada ou cinza
- **Corpo:** Inter, peso 300-400, cor branca ou cinza
- **Preços:** Inter, peso 600, cor dourada
- **CTAs / botões:** Inter, peso 600, uppercase, letter-spacing 1px

### Componentes Visuais

- **Botões primários:** fundo dourado (#d4af37), texto preto (#111), border-radius 6px, hover brilho
- **Botões secundários:** border 1px dourado, texto dourado, fundo transparente
- **Cards:** fundo #111 ou #1a1a1a, border 1px #222, border-radius 10-12px, hover com border dourado sutil
- **Separadores:** linhas douradas gradiente (de dourado para transparente)
- **Ícones:** SVG inline, cor dourada, 28-32px

### Animações

- Scroll reveal: elementos aparecem com fade-in + translate-y suave (IntersectionObserver)
- Hover em cards: translateY(-4px) + sombra + border dourado
- Navbar: transição de transparente para fundo sólido ao scrollar
- Modal: fade-in do overlay + scale do container
- Filtros do catálogo: transição suave ao filtrar produtos

### Responsividade

- **Desktop (>1024px):** grids de 3 colunas, layout lado a lado no contato
- **Tablet (768-1024px):** grids de 2 colunas, navbar compacta
- **Mobile (<768px):** 1 coluna, hamburguer menu, botões full-width, modal fullscreen

## Stack Técnica

| Componente | Tecnologia |
|---|---|
| Frontend | HTML + CSS + JS vanilla |
| Dados | Firestore REST API + fallback local |
| Admin | cardapio-admin (React + Firebase) — novo tipo "roupas" |
| Fontes | Google Fonts (Playfair Display + Inter) |
| Ícones | SVG inline (Lucide Icons) |
| Ícones | SVG inline (Lucide ou similar) |
| Imagens | Firebase Storage (gerenciadas pelo admin) |
| Deploy | Hosting estático |

## Catálogo — 33 Produtos Iniciais

### Camisas Sociais

| # | Nome | Preço |
|---|---|---|
| 1 | Camisa Social Azul Marinho levemente Acetinada | R$ 229,90 |
| 2 | Camisa Social Slim Branca (Fibra Natural de Bambu) | R$ 229,90 |
| 3 | Camisa Social sem bolso Branca (Passa fácil e não Amassa) | R$ 229,90 |
| 4 | Camisas Social Slim Fit Vivacci (passa fácil e não amarrota) | R$ 229,90 |
| 10 | Camisa social tecnológica Elastic Sibra | R$ 249,90 |

### Camisetas & Polos

| # | Nome | Preço |
|---|---|---|
| 13 | T-shirts básicas diversas cores | Sob consulta |
| 16 | Camisas Polo Vivacci P ao G1 | R$ 149,90 |
| 22 | Camisetas Polos diversos modelos, cores e tamanhos | Sob consulta |

### Calças & Bermudas

| # | Nome | Preço |
|---|---|---|
| 8 | Bermudas alfaiataria | R$ 169,90 |
| 9 | Calças Jeans (Disponível do 38 ao 48) | R$ 179,90 |
| 12 | Calças Alfaiataria com regulagem | Sob consulta |
| 18 | Calças alfaiataria (disponível do 38 ao 52) | Sob consulta |
| 29 | Calças Sarjas Alfaiataria | Sob consulta |

### Ternos & Costumes

| # | Nome | Preço |
|---|---|---|
| 7 | Costume Two Way com elastano | R$ 599,90 |
| 17 | Ternos Poliviscose Slim Italiano | R$ 799,90 |
| 23 | Ternos e costumes (Variedade em cores e tecidos) | Sob consulta |
| 27 | Ternos Slim Microfibra Italiano | R$ 499,90 |
| 28 | Ternos Fio Indiano Slim Italiano | R$ 599,90 |

### Acessórios

| # | Nome | Preço |
|---|---|---|
| 5 | Cinto dupla face (Preto/Marrom) | R$ 49,90 |
| 6 | Cintos trava automática Premium | R$ 129,90 |
| 19 | Variedade em Gravatas | Sob consulta |
| 21 | Cintos em vários modelos, tamanhos e cores | Sob consulta |
| 24 | Gravatas Slim Xadrez 1.200 Fios | R$ 29,90 |
| 25 | Cinto Couro Nobre Fasolo | R$ 99,90 |
| 30 | Gravatas Importadas Slim (Vários modelos) | Sob consulta |
| 33 | Cinto Elástico com trava | Sob consulta |

### Kits & Combinações

| # | Nome | Preço |
|---|---|---|
| 20 | Looks, Combinações e Conjuntos | Sob consulta |
| 26 | Kit Perfeito Calça Alfaiataria + Camisa Polo Importada Vivacci | Sob consulta |
| 31 | Kit 2 Camisa Social + Calça Esporte fino | Sob consulta |
| 32 | Kit 1 Camisa social + Gravata + Prendedor de Gravata | Sob consulta |

### Jaquetas & Suéteres

| # | Nome | Preço |
|---|---|---|
| 11 | Jaqueta Sarja Premium | Sob consulta |
| 14 | Suéter Premium | Sob consulta |

### Perfumes

| # | Nome | Preço |
|---|---|---|
| 15 | Perfumes a pronta entrega | R$ 159,90 |

## O que NÃO está no escopo

- E-commerce com carrinho de compras / checkout
- Integração com gateway de pagamento
- Sistema de login para clientes
- Blog ou seção de notícias
- API do Instagram (fotos são estáticas)
- Chat ao vivo (apenas WhatsApp)
- Multi-idioma
