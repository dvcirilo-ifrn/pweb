---
theme: default
size: 4:3
marp: true
paginate: true
_paginate: false
title: Aula 07: Bootstrap
author: Diego Cirilo

---
<style>
img {
  display: block;
  margin: 0 auto;
}
</style>

# <!-- fit --> Programação de Aplicação Web

### Prof. Diego Cirilo

**Aula 07**: Bootstrap

---
# O que é Bootstrap?

![bg right:35%](../img/bootstrap-logo.png)

- Framework CSS de código aberto;
- Criado pelo Twitter em 2011;
- Fornece componentes prontos e um sistema de grid responsivo;
- Objetivo: criar interfaces consistentes e responsivas com menos código CSS.

---
# Por que usar Bootstrap?
- Responsividade automática (adapta ao tamanho da tela);
- Componentes prontos: botões, formulários, cards, menus, etc.;
- Documentação extensa e exemplos copiáveis;
- Amplamente utilizado no mercado;
- Integra facilmente com templates Django.

---
# Pontos negativos do Bootstrap
- Sites ficam com aparência parecida ("cara de Bootstrap");
- CSS gerado é grande — carrega classes que não serão usadas;
- Customizar além do básico exige sobrescrever muitas regras;
- HTML fica verboso com muitas classes em cada elemento;
- Pode incentivar o uso sem entender o CSS subjacente.

---
# Designer vs. Programador
- Em equipes reais, design e desenvolvimento são **funções separadas**;
- Designer: identidade visual, cores, tipografia;
- Programador: lógica, banco de dados, integração;
- Nessa disciplina o foco é o **back-end**;
- Bootstrap evita perder tempo no front e entrega interfaces **funcionais**;
- Componentes padronizados que os usuários **já conhecem e sabem usar**.

---
# Frameworks Alternativos
| Framework | Característica |
|-----------|----------------|
| **Tailwind CSS** | Utilitários atômicos, altamente customizável, sem componentes prontos |
| **Bulma** | Baseado em Flexbox, sem JavaScript, sintaxe limpa |
| **Foundation** | Robusto, voltado para projetos corporativos |
| **Materialize** | Baseado no Material Design do Google |
| **PicoCSS** | Minimalista, estiliza tags HTML sem classes |

---
# Como incluir o Bootstrap
- Via CDN (recomendado para projetos simples):
```html
<!-- No <head> do HTML -->
<link rel="stylesheet"
  href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css">

<!-- Antes do </body> -->
<script
  src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js">
</script>
```
- Sempre consulte a versão atual em [getbootstrap.com](https://getbootstrap.com)

---
# Estrutura base de um template
```html
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet"
      href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css">
    <title>Meu Site</title>
</head>
<body>
    <!-- conteúdo aqui -->
    <script
      src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js">
    </script>
</body>
</html>
```

---
# Classes Utilitárias
- Bootstrap funciona principalmente através de **classes CSS**;
- Não é necessário escrever CSS próprio para a maioria dos casos;
- As classes seguem convenções previsíveis.

---
# Exemplos de Classes Utilitárias
```html
<!-- Texto -->
<p class="text-center">Texto centralizado</p>
<p class="text-danger fw-bold">Texto vermelho e negrito</p>

<!-- Fundo e padding -->
<div class="bg-primary text-white p-3">
    Fundo azul, texto branco, padding 3
</div>

<!-- Margem -->
<div class="mt-4 mb-2">Margem top 4, bottom 2</div>
```

---
# Espaçamento - Convenção
- Propriedade: `m` (margin) ou `p` (padding);
- Lado: `t` (top), `b` (bottom), `s` (start), `e` (end), `x` (horizontal), `y` (vertical);
- Tamanho: `0` a `5` (ou `auto`);
- Exemplos:
    - `mt-3`: margin-top nível 3;
    - `px-4`: padding horizontal nível 4;
    - `m-0`: sem margem em todos os lados.

---
# Cores no Bootstrap
- Cores semânticas disponíveis como classes:
    - `primary` - azul (ação principal)
    - `secondary` - cinza
    - `success` - verde
    - `danger` - vermelho
    - `warning` - amarelo
    - `info` - ciano
    - `light` / `dark`
- Usadas como sufixo: `text-danger`, `bg-success`, `btn-warning`

---
# Sistema de Grid

![bg right:45%](../img/bootstrap-grid.png)

- O grid é o coração do Bootstrap;
- Divide a página em **12 colunas**;
- Adapta o layout automaticamente para diferentes tamanhos de tela;
- Necessita de 3 elementos: `container` → `row` → `col`.

---
# Container, Row e Col
```html
<div class="container">      <!-- limita a largura -->
    <div class="row">        <!-- linha do grid -->
        <div class="col">    <!-- coluna automática -->
            Coluna 1
        </div>
        <div class="col">
            Coluna 2
        </div>
        <div class="col">
            Coluna 3
        </div>
    </div>
</div>
```
- 3 colunas de largura igual (4 + 4 + 4 = 12)

---
# Colunas com Tamanho Definido
```html
<div class="container">
    <div class="row">
        <div class="col-8">
            Coluna maior (8/12)
        </div>
        <div class="col-4">
            Coluna menor (4/12)
        </div>
    </div>
</div>
```
- Os números sempre somam 12;
- `col-8` + `col-4` = 12

---
# Breakpoints (Responsividade)
| Prefixo | Tela | Dispositivo |
|---------|------|-------------|
| (nenhum) | < 576px | Extra pequeno |
| `sm` | ≥ 576px | Pequeno |
| `md` | ≥ 768px | Médio |
| `lg` | ≥ 992px | Grande |
| `xl` | ≥ 1200px | Extra grande |

---
# Grid Responsivo
```html
<div class="container">
    <div class="row">
        <!-- Mobile: 12 cols (full), Tablet: 6, Desktop: 4 -->
        <div class="col-12 col-md-6 col-lg-4">Card 1</div>
        <div class="col-12 col-md-6 col-lg-4">Card 2</div>
        <div class="col-12 col-md-6 col-lg-4">Card 3</div>
    </div>
</div>
```
- Em mobile: cada card ocupa a linha inteira;
- Em tablet: dois cards por linha;
- Em desktop: três cards por linha.

---
<style scoped>section { font-size: 22px; }</style>
# Revisão: Flexbox no CSS

![bg right:38%](../img/flexbox-axes.png)

- Modelo de layout CSS para distribuir elementos em uma dimensão;
- `display: flex` ativa o modo flex em um elemento;
- **Flex container**: o elemento que recebe `display: flex`;
    - Controla como os filhos são organizados;
- **Flex items**: os filhos diretos do container;
    - São posicionados automaticamente pelo container;
- Dois eixos controlam o layout:
    - **Eixo principal** (*main axis*): direção dos itens (`row` ou `column`);
    - **Eixo cruzado** (*cross axis*): perpendicular ao principal.

---
# Revisão: Propriedades do Flexbox
- Aplicadas no **container**:
    - `flex-direction` — direção dos itens (`row`, `column`);
    - `justify-content` — alinhamento no eixo principal;
    - `align-items` — alinhamento no eixo cruzado;
    - `gap` — espaçamento entre os itens;
    - `flex-wrap` — permite quebrar linha se não couber.
- Aplicadas nos **items**:
    - `flex-grow` — quanto o item cresce para preencher espaço;
    - `align-self` — sobrescreve o `align-items` para um item só.

---
# Flexbox com Bootstrap
- Bootstrap oferece classes utilitárias para Flexbox;
- Útil para alinhar elementos dentro de um container;
- Complementa o grid para alinhamentos finos;
- Documentação: *Utilities → Flex*.

---
# d-flex
- `d-flex` ativa o `display: flex` no elemento;
- Os filhos diretos passam a ser os *flex items*;
```html
<div class="d-flex">
    <div>Item 1</div>
    <div>Item 2</div>
    <div>Item 3</div>
</div>
```
- Sem mais classes: itens ficam em linha, alinhados à esquerda.

---
# justify-content — Eixo horizontal
```html
<!-- Itens à esquerda (padrão) -->
<div class="d-flex justify-content-start"> ... </div>

<!-- Itens ao centro -->
<div class="d-flex justify-content-center"> ... </div>

<!-- Itens à direita -->
<div class="d-flex justify-content-end"> ... </div>

<!-- Espaço igual entre os itens -->
<div class="d-flex justify-content-between"> ... </div>

<!-- Espaço ao redor de cada item -->
<div class="d-flex justify-content-around"> ... </div>
```

---
# align-items — Eixo vertical
```html
<!-- Alinha ao topo (padrão) -->
<div class="d-flex align-items-start"> ... </div>

<!-- Alinha ao centro vertical -->
<div class="d-flex align-items-center"> ... </div>

<!-- Alinha à base -->
<div class="d-flex align-items-end"> ... </div>
```
- Muito usado para centralizar ícone + texto lado a lado.

---
# flex-direction e gap
```html
<!-- Itens em coluna (empilhados) -->
<div class="d-flex flex-column"> ... </div>

<!-- Espaçamento entre itens -->
<div class="d-flex gap-3"> ... </div>

<!-- Combinando tudo -->
<div class="d-flex justify-content-between align-items-center gap-2">
    <span>Título</span>
    <button class="btn btn-sm btn-primary">Ação</button>
</div>
```

---
# Exemplo Prático com d-flex
```html
<!-- Rodapé com texto à esquerda e link à direita -->
<div class="d-flex justify-content-between align-items-center p-3 bg-dark text-white">
    <span class="fw-bold">Meu Site</span>
    <span class="text-white-50 small">© 2024</span>
</div>

<!-- Card com badge posicionado -->
<div class="card-header d-flex justify-content-between align-items-center">
    <h5 class="mb-0">Pedidos Recentes</h5>
    <span class="badge bg-danger">5 pendentes</span>
</div>
```

---
# Usando a Documentação
- A documentação do Bootstrap é o principal recurso;
- Acesse: **getbootstrap.com/docs**
- Cada componente tem:
    - Descrição do componente;
    - Exemplos visuais interativos;
    - Código HTML pronto para copiar.

---
# Workflow com a Documentação
1. Identifique o que você precisa (ex.: "menu de navegação");
2. Acesse a documentação e procure o componente ("Navbar");
3. Visualize os exemplos disponíveis;
4. Copie o código HTML do exemplo que mais se adequa;
5. Cole no seu template e adapte o conteúdo.

---
# Componente: Navbar
- Menu de navegação responsivo;
- Na documentação: *Components → Navbar*;
```html
<nav class="navbar navbar-expand-lg bg-body-tertiary">
  <div class="container-fluid">
    <a class="navbar-brand" href="#">Meu Site</a>
    <div class="navbar-nav">
      <a class="nav-link active" href="#">Home</a>
      <a class="nav-link" href="#">Sobre</a>
    </div>
  </div>
</nav>
```

---
# Componente: Card
- Caixa de conteúdo com cabeçalho, corpo e rodapé;
- Na documentação: *Components → Card*;
```html
<div class="card" style="width: 18rem;">
  <div class="card-body">
    <h5 class="card-title">Título do Card</h5>
    <p class="card-text">Texto do conteúdo do card.</p>
    <a href="#" class="btn btn-primary">Ver mais</a>
  </div>
</div>
```

---
# Componente: Button
- Botões com estilos semânticos;
- Na documentação: *Components → Buttons*;
```html
<button class="btn btn-primary">Primário</button>
<button class="btn btn-secondary">Secundário</button>
<button class="btn btn-success">Sucesso</button>
<button class="btn btn-danger">Perigo</button>
<button class="btn btn-outline-primary">Contorno</button>
```

---
# Componente: Alert
- Mensagens de feedback ao usuário;
- Na documentação: *Components → Alerts*;
```html
<div class="alert alert-success" role="alert">
  Operação realizada com sucesso!
</div>

<div class="alert alert-danger" role="alert">
  Erro! Verifique os dados informados.
</div>
```
- Integra bem com as mensagens do Django (`django.contrib.messages`)

---
# Componente: Table
- Tabelas estilizadas;
- Na documentação: *Content → Tables*;
```html
<table class="table table-striped table-hover">
  <thead class="table-dark">
    <tr>
      <th>Nome</th>
      <th>Preço</th>
    </tr>
  </thead>
  <tbody>
    <tr><td>Produto A</td><td>R$ 10,00</td></tr>
  </tbody>
</table>
```

---
# Formulários com Bootstrap
- Na documentação: *Forms → Overview*;
```html
<form>
  <div class="mb-3">
    <label for="nome" class="form-label">Nome</label>
    <input type="text" class="form-control" id="nome">
  </div>
  <div class="mb-3">
    <label for="email" class="form-label">E-mail</label>
    <input type="email" class="form-control" id="email">
  </div>
  <button type="submit" class="btn btn-primary">Enviar</button>
</form>
```

---
# Bootstrap + Django Templates
- O Bootstrap é usado diretamente nos templates HTML do Django;
- Carregue o CSS no template base (`base.html`) e os outros templates herdam;
```django
<!-- templates/base.html -->
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/
    bootstrap@5.3.3/dist/css/bootstrap.min.css">
    <title>{% block title %}Meu Site{% endblock %}</title>
</head>
<body>
{% block content %}{% endblock %}
<script src="https://cdn.jsdelivr.net/npm/
bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

---
# Integrando Mensagens Django + Alert Bootstrap
```django
{% if messages %}
    {% for message in messages %}
        <div class="alert alert-{{ message.tags }} alert-dismissible fade show">
            {{ message }}
            <button type="button" class="btn-close" data-bs-dismiss="alert">
            </button>
        </div>
    {% endfor %}
{% endif %}
```
- `message.tags` retorna `success`, `error`, `warning`, `info`

---
# Customizando a Paleta de Cores
- O Bootstrap 5 usa **variáveis CSS** (`--bs-*`) internamente;
- É possível sobrescrevê-las sem modificar o Bootstrap;
- Basta criar um arquivo CSS próprio carregado **após** o Bootstrap.

---
# Variáveis CSS do Bootstrap
- Cada cor semântica tem uma variável correspondente:
```css
:root {
    --bs-primary:     #0d6efd;  /* azul padrão */
    --bs-secondary:   #6c757d;
    --bs-success:     #198754;
    --bs-danger:      #dc3545;
    --bs-warning:     #ffc107;
    --bs-info:        #0dcaf0;
}
```
- Sobrescrever no `:root` do seu CSS muda a cor em todo o site.

---
# Sobrescrevendo Variáveis CSS
- Crie um arquivo `custom.css` carregado **após** o Bootstrap:
```html
<link rel="stylesheet" href=".../bootstrap.min.css">
<link rel="stylesheet" href="custom.css">  <!-- depois! -->
```
- No `custom.css`:
```css
:root {
    --bs-primary: #6f42c1;        /* roxo */
    --bs-primary-rgb: 111, 66, 193;
    --bs-success: #20c997;        /* verde-água */
    --bs-danger:  #e63946;        /* vermelho vivo */
}
```
- `btn-primary`, `bg-primary`, `text-primary` etc. passam a usar a nova cor.

---
# Bootstrap Icons
- Biblioteca de ícones oficial do Bootstrap;
- Mais de 2.000 ícones SVG gratuitos;
- Biblioteca **separada** — precisa ser incluída além do Bootstrap;
```html
<!-- No <head>, após o CSS do Bootstrap -->
<link rel="stylesheet"
  href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.min.css">
```
- Uso: tag `<i>` com a classe `bi bi-nome-do-icone`;
- Catálogo completo em **icons.getbootstrap.com**.

---
# Bootstrap Icons — Exemplo
```html
<!-- Ícones simples -->
<i class="bi bi-house"></i>
<i class="bi bi-envelope"></i>
<i class="bi bi-person-circle"></i>

<!-- Tamanho e cor via classes utilitárias -->
<i class="bi bi-star-fill text-warning fs-3"></i>

<!-- Ícone dentro de botão -->
<button class="btn btn-primary">
    <i class="bi bi-plus-circle me-1"></i> Novo item
</button>

<!-- Ícone em alerta -->
<div class="alert alert-danger">
    <i class="bi bi-exclamation-triangle-fill me-2"></i>
    Atenção: verifique os dados antes de continuar.
</div>
```

---
# Onde Encontrar Mais
- **Documentação oficial**: getbootstrap.com/docs/5.3
- **Componentes**: getbootstrap.com/docs/5.3/components
- **Utilitários**: getbootstrap.com/docs/5.3/utilities
- **Exemplos completos**: getbootstrap.com/docs/5.3/examples
- **Ícones**: icons.getbootstrap.com

---
# Sugestões de Projeto
1. **Página de cardápio de restaurante**
    - Grid responsivo com cards para cada prato (imagem, nome, preço, descrição);
    - Navbar com o nome do restaurante e categorias;
    - Rodapé com contato e localização.

---
# Sugestões de Projeto
2. **Painel de controle simples (dashboard)**
    - Navbar com nome do sistema e botão de logout;
    - Cards com estatísticas (total de produtos, pedidos, clientes);
    - Tabela paginada com listagem de registros;
    - Formulário modal para adicionar novos itens.

---
# Sugestões de Projeto
3. **Blog/portfólio pessoal**
    - Página inicial com grid de posts em cards;
    - Página de detalhe do post com layout de duas colunas (conteúdo + sidebar);
    - Formulário de contato estilizado;
    - Navbar fixa com links de seções.

---
# Referências
- https://getbootstrap.com/docs/5.3/
- https://getbootstrap.com/docs/5.3/layout/grid/
- https://getbootstrap.com/docs/5.3/utilities/flex/
- https://getbootstrap.com/docs/5.3/components/
- https://getbootstrap.com/docs/5.3/examples/
- https://getbootstrap.com/docs/5.3/customize/css-variables/

---
# <!--fit--> Dúvidas? 🤔
