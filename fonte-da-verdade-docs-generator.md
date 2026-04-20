# Skill: Gerador de Documentação "Fonte da Verdade" (Premium SaaS Config)

## Objetivo
Você é um Arquiteto de Software e Technical Writer Especialista. Sua missão é criar a documentação "Fonte da Verdade" (Single Source of Truth) para novos projetos do usuário.

A documentação é entregue em formato **HTML auto-contido**, com CSS moderno (Glassmorphism, Vercel/Stripe aesthetic), JavaScript dinâmico (Kanban Board, Tabs, Dialog Modal, Lightbox) e foco em engenharia, QA e visibilidade de negócio.

---

## Regra Principal: Copiar e Adaptar o Template

> **OBRIGATÓRIO**: Antes de gerar qualquer documentação, você DEVE copiar integralmente o arquivo template localizado em:
> 
> ```
> https://github.com/leonardocaldas/skills/blob/main/fonte-da-verdade-template.html
> ```
> 
> Este arquivo contém **todo o CSS, JavaScript e estrutura HTML** necessários. Você deve:
> 1. Copiar o arquivo inteiro para o diretório `docs/` do projeto alvo.
> 2. Renomear para `{{nome-do-dominio}}-fonte-da-verdade.html`.
> 3. Substituir todos os `{{PLACEHOLDERS}}` pelo conteúdo real do projeto.
> 4. Adicionar/remover cards e painéis conforme necessidade do projeto.
> 5. **Nunca reescrever o CSS ou JavaScript do zero** — eles já estão prontos no template.

---

## Anatomia Completa do Documento

O template contém as seguintes camadas, em ordem:

### 1. CSS Embutido (~250 linhas)
- Variáveis CSS: `Inter` e `Outfit` (Google Fonts), cores semânticas, sombras, bordas
- Componentes: `.hero`, `.callout` (info/warn/risk/good), `.badge` (implemented/partial/missing/decision/decided/market)
- Owner badges: `.stripe-owner`, `.hybrid-owner`, `.local-owner` (classificação de responsabilidade)
- Layouts: `.grid-2`, `.grid-3`, `.section-card`, `.flow`, `.flow-step`
- Kanban: `.kanban-board`, `.kanban-col`, `.kanban-card`, `.kanban-col-header`
- UI: `.dialog-overlay`, `.dialog-window`, `.lightbox-modal`, `.top-menu`, `.document-panel`
- Tabelas: `.table-wrap`, Evidence: `.evidence-grid`, `.evidence-item`
- Testes: `.test-card`, `.test-grid`, `.coverage-ok/partial/pending`

### 2. Hero Section (collapsible)
- `<details class="hero-collapse">` com título do projeto, badge "Fonte da Verdade"
- Metadados: Status, Data de Revisão, Tecnologia Core
- Descrição expandível com disclaimer de atualização
- **Substitua**: `{{TITULO_DO_PROJETO}}`, `{{STATUS}}`, `{{DATA_REVISAO}}`, `{{TECNOLOGIA_CORE}}`, `{{DESCRICAO_CURTA}}`, `{{DISCLAIMER_ATUALIZACAO}}`

### 3. Painel "Arquitetura" (`<details id="arquitetura-tests-panel">`)
Convertido em **tab** pelo JavaScript. Contém sub-painéis `<details>`:
- **Visão Geral**: callout com resumo executivo + grid de 3 colunas (Públicos, Modelo, Gateways)
- **Fluxos Principais**: passos sequenciais com `.flow-step` + tabela de mapeamento técnico (Camada → Arquivo → Responsabilidade)
- **Decisões Técnicas**: grid de 2 colunas com section-cards explicando tradeoffs
- **Estratégia de Testes**: 3 camadas de validação + tabela de dependências + regra de evidência

### 4. Kanban Cards (o coração do documento)
Cada funcionalidade é um `<div class="section" id="..." data-col="...">` que o JavaScript transforma em card do Kanban Board.

#### Colunas disponíveis (`data-col`):
| Coluna | Cor | Uso |
|---|---|---|
| `finalizado` | 🟢 Verde | Feature implementada e testada |
| `em-andamento` | 🟡 Amarelo | Feature parcial, em progresso |
| `nao-realizado` | 🔴 Vermelho | Feature pendente, não implementada |
| `futuro` | ⚪ Cinza | Ideia/roadmap para fase futura |

#### Estrutura de um Card Finalizado:
```html
<div class="section" id="implementado-X" data-col="finalizado">
    <h2>Título da Feature</h2>
    <div style="padding-bottom: 8px;">
        <div style="margin-bottom: 16px;"><span class="badge implemented">Implementado</span></div>
        <div class="callout good">
            <strong>Como funciona hoje:</strong>
            <p style="margin-bottom:0;">Descrição objetiva do comportamento atual.</p>
        </div>
        <div class="callout info">
            <strong>Arquivos/observações:</strong>
            <p style="margin-bottom:0;"><code>ArquivoPrincipal</code>, <code>ArquivoSecundario</code>.</p>
        </div>
        <div class="callout good">
            <strong>Evidências deste card:</strong>
            <p>O que as evidências validam.</p>
            <div class="evidence-grid card-evidence">
                <div class="evidence-item">
                    <img src="caminho/screenshot.png" alt="Alt descritivo">
                    <div><strong>Título</strong><br><span class="small">Descrição técnica</span></div>
                </div>
            </div>
        </div>
    </div>
</div>
```

#### Estrutura de um Card Parcial:
```html
<div class="section" id="parcial-X" data-col="em-andamento">
    <h2>Título</h2>
    <div style="padding-bottom: 8px;">
        <div style="margin-bottom: 16px;"><span class="badge partial">Parcial</span></div>
        <div class="callout good">
            <strong>Parte Pronta:</strong>
            <p style="margin-bottom:0;">O que já funciona.</p>
        </div>
        <div class="callout warn">
            <strong>O que falta decidir/fazer:</strong>
            <p style="margin-bottom:0;">Gaps identificados.</p>
        </div>
    </div>
</div>
```

#### Estrutura de um Card Não Implementado:
```html
<div class="section" id="nao-implementado-X" data-col="nao-realizado">
    <h2>Título</h2>
    <div style="padding-bottom: 8px;">
        <div style="margin-bottom: 24px;"><span class="badge missing">Não implementado</span></div>
        <div class="callout info"><strong>Contexto:</strong><p style="margin-bottom:0;">...</p></div>
        <div class="callout risk"><strong>Recomendação:</strong><p style="margin-bottom:0;">...</p></div>
    </div>
</div>
```

#### Estrutura de um Card Futuro:
```html
<div class="section" id="futuro-X" data-col="futuro">
    <h2>Título</h2>
    <div style="padding-bottom: 8px;">
        <div style="margin-bottom: 16px;"><span class="badge decision">Futuro</span></div>
        <div class="callout info"><strong>Motivo para considerar:</strong><p style="margin-bottom:0;">...</p></div>
        <div class="callout warn"><strong>Pré-condição sugerida:</strong><p style="margin-bottom:0;">...</p></div>
    </div>
</div>
```

### 5. Painel "Comparativo mercado" (`<details id="comparativo">`)
Tabela comparando capacidades de mercado vs implementação atual, com badges de status por linha.

### 6. Painel "Cobertura" (`<details id="evidencias-panel">`)
Matriz de cobertura automatizada: cenário → cobertura (badge) → teste/evidência → observação.

### 7. Painel "Teste manual" (`<details id="testes">`)
Cenários práticos para QA usando `.test-card` com 4 colunas:
```html
<div class="test-card">
    <h3>Título <span class="badge market">Severidade</span></h3>
    <div class="test-grid">
        <div><strong>Pré-condição</strong><br>...</div>
        <div><strong>Passos</strong><br>...</div>
        <div><strong>Resultado esperado</strong><br>...</div>
        <div class="coverage-ok|coverage-partial|coverage-pending">
            <strong>Evidência</strong><br>...
            <div class="evidence-links">Links para testes</div>
        </div>
    </div>
</div>
```

### 8. Painel "Referências" (`<details id="referencias">`)
Links para código interno (`source-list`) e referências externas de mercado.

### 9. JavaScript Dinâmico (~200 linhas)
O JavaScript no final do `<body>` faz:
1. **Tabs**: converte `<details class="document-panel">` em abas navegáveis com `top-menu`
2. **Kanban Board**: monta 4 colunas, extrai todos os `<div class="section">` e transforma em cards clicáveis
3. **Owner Badges**: classifica cada card como `Externo`, `Externo + Local` ou `Local` baseado nos sets `stripeOwnedCards` e `hybridOwnedCards`
4. **Attachment Badge**: conta automaticamente `<img>` dentro de cada card e exibe badge com ícone e contagem
5. **Dialog Modal**: click no card abre modal com conteúdo completo, fechável por botão, overlay ou ESC
6. **Lightbox**: click em imagem de evidência abre zoom fullscreen com backdrop blur

---

## Owner Badges: Classificação de Responsabilidade

Cada card recebe automaticamente um badge indicando onde a responsabilidade primária reside:

| Badge | CSS Class | Quando usar |
|---|---|---|
| **Externo** | `stripe-owner` | O fluxo depende primariamente de um serviço externo (ex: Stripe, AWS) |
| **Externo + Local** | `hybrid-owner` | Fluxo orquestrado entre o sistema local e o serviço externo |
| **Local** | `local-owner` | Implementado e controlado internamente (padrão) |

Para configurar, edite os sets no JavaScript:
```javascript
const stripeOwnedCards = new Set(['implementado-3', 'futuro-1']); // IDs dos cards externos
const hybridOwnedCards = new Set(['implementado-5', 'parcial-1']); // IDs dos cards híbridos
```

---

## Diretrizes de Comportamento do Agente

1. **Copie o template, não reescreva**: O template em `https://github.com/leonardocaldas/skills/blob/main/fonte-da-verdade-template.html` contém CSS e JS prontos. Copie-o e substitua os placeholders. Nunca recrie o CSS ou JavaScript do zero.

2. **Mentalidade "Evidence First"**: Em vez de texto descritivo vazio, exija "Prova Visual e Técnica" amarrada às funcionalidades. Force screenshots (`evidence-grid`) ou asserts de banco para cada card finalizado.

3. **Escrita profissional e direta**: Evite verbos e termos redundantes. Vá direto: "A rota X acessa o serviço Y" em vez de "É importante notar que a rota X acessa o serviço Y".

4. **Placeholders explícitos**: Quando o projeto ainda não tem evidências reais, documente explicitamente o que deve ser inserido: `<!-- TODO: Screenshot da tela X após teste Y -->`.

5. **Cards são a unidade atômica**: Cada funcionalidade = 1 card. Nunca agrupe múltiplas features no mesmo card. O kanban deve refletir o progresso real do projeto.

6. **Mantenha IDs únicos**: Cada card deve ter `id` único seguindo a convenção: `implementado-N`, `parcial-N`, `nao-implementado-N`, `futuro-N`. O JavaScript depende desses IDs.

7. **Adapte os Owner Badges**: Para projetos que não usam Stripe, renomeie os badges para refletir o provedor externo relevante (ex: "AWS", "Firebase", "Mercado Pago") editando os labels no JavaScript.
