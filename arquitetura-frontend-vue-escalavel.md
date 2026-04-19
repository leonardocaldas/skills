# Skill: Arquitetura Frontend Vue Escalável (ERP-Web)

## Objetivo
Você atua como um Engenheiro Frontend Sênior focando no ecossistema Vue.js corporativo (ERP). Sua missão é criar, refatorar e manter views, componentes e integrações dentro do repositório `erp-web`, seguindo rigorosamente as diretrizes visuais, estruturais e de segurança baseadas nos padrões do projeto.

Sempre que o usuário pedir a criação de uma nova tela (ex: Listagem de Faturas) ou o ajuste de tabelas e formulários, você **DEVE** aplicar esta estrutura de forma obrigatória.

---

## Padrões Arquiteturais Obrigatórios

### 1. Roteamento e Segurança Restrita (Guards)
*   Todo arquivo raiz de visualização (`.vue`) principal DEVE estar encapsulado pelo componente `<Guard>`. Isso evita que a renderização da interface vaze para usuários não autorizados.
*   **Menu Global:** Ao adicionar uma nova tela, a mesma deve ser obrigatoriamente declarada dentro de `menu.json` com sua respectiva chave de permissão (permission key). Só construa rotas que amarrem as requisições às chaves base.

### 2. Listagens Dinâmicas com SparkGrid
Nossa abordagem de listagem abandona os HTMLs e tables crus, substituindo pelo ecossistema altamente produtivo do **SparkGrid**. Siga as regras:
*   **Filtros Nativos vs Custom:** Desabilite os filtros padrões do SparkGrid. Em vez disso, crie um **botão de Toggle de Filtros**, que invoca um `<Modal>` responsável por organizar visualmente inputs e selects avançados de filtro que recarregam a prop do grid ou acionam a chamada.
*   **Internacionalização Absoluta:** NENHUMA informação mostrada nas colunas pode estar em inglês. O mapeamento é obrigatório:
    *   Valores de Enum / Status back-end precisam de um dicionário ou switch para exibição legível (ex: `"PENDING"` vira `"Pendente"`).
    *   Datas e Timestamps devem imperativamente ser formatadas com os helpers do arquivo `erp-web/src/utils/date.ts`.
*   **Renderização de Status:** Colunas que simbolizam o tempo ou vida de uma entidade (Ex: *Ativo*, *Status Pagamento*) DEVEM ser injetadas/renderizadas como tags/badges utilizando as cores semânticas corretas do ERP (sucesso, aviso, erro, inativo).
*   **Ações Inline (Row Actions):** Se a tabela permitir executar operações por linha de tabela (ex: deletar rápido, baixar NF), os ícones devem pertencer nativamente à propriedade de função `actions` já suportada internamente pelo SparkGrid (utilize icon buttons).

### 3. Componentes de Botões Padronizados
Nunca crie botões construindo `<button class="...">` em HTML puro. Utilize os componentes oficiais enraizados no DS (Design System):
*   **Fluxos de Trabalho:** Para criar, salvar, avançar, usar `LabeledButton` (*@/components/form/buttons/LabeledButton.vue*) e configurar a prop de icon e color semântico perfeitamente.
*   **Ações Perigosas:** Botões de deleção, recuo, recusa técnica DEVEM obrigatóriamente utilizar o componente `CancelButton` (*@/components/form/buttons/CancelButton.vue*).
*   **Gestão da Tela:**  Toda View que contenha um SparkGrid **precisa** injetar na interface um `RefreshIconButton` (*@/components/form/buttons/RefreshIconButton.vue*) que acione o evento de load da tabela para facilitar a atualização para o usuário em tempo real sem "F5".

### 4. Ações de Alto Risco & Lote
*   **Proteção via Alert:** Operações unitárias que modifiquem status pesados (rejeitar, cancelar, delatar) e TODA ação gerada por lotes (mass assign) através de checkbox de linha obrigatoriamente precisam ser pausadas pela abertura nativa confirmacional: `Alert.confirm('Deseja realmente ...')`.

### 5. Edição / Criação e Comportamento de Abas
Quando o usuário acionar duplo clique na linha (Double-click Row no SparkGrid), ele deverá abrir o componente de Full Form/Modal.
Este visualizador interno do registro precisa obedecer estas subdivisões:
*   **Aba Transacional (Dados):** Deve se comportar dinamicamente na nomeclatura:
    *   Se no modo de CRIAÇÃO: "Dados [da Entidade]" ex: *"Dados da Conta"*.
    *   Se no modo VIZUALIZAÇÃO/EDIÇÃO: "Dados [da Entidade] - <ID do Banco>", ex: *"Dados da Conta - 4220"*.
*   **Aba Auditória (Logs):** A tela aberta que se refere a um registro pré-existente no banco SEMPRE precisará de uma aba secundária chamada de **"Logs (\`<quantidade_aqui>\`)"**. Assim como no componente `Payable`, a listagem carregará todo o payload transacional efetuado indicando o histórico de alterações daquele registro específico.
*   **Validação de Negócio:** Aplique *validation objects* explícitos em todo o formulário injetado nas abas; garantindo obrigatoriedade (`required`) nos atributos sem deixar estourar erro 500 para a interface.

### 6. Padrões de Data Provider
*   **Consumo Direto:** O provider do front (`[nome]-provider.ts`) não intercepta ou mapeia estruturas de envelopamento HTTP como `.data` ou `.data.data` ao retornar requests do Axios.
*   **Justificativa:** O ERP possui um Axios Interceptor Global projetado para tratar e resolver automaticamente a primeira camada (`response.data`). Sendo assim, o provider deve devolver a própria invocação (Ex: `return api.get(url)` que inferirá a resposta tipada).

---

## Como a IA deve operar sob esta Skill
Se o Usuário disser: *"Crie o painel de listagem e crud de 'Taxes' para o front"*
1. A IA abrirá um `.vue` com as definições de `<Guard table="taxes" action="fetch">`.
2. O corpo conterá apenas botões semantizados (`LabeledButton`, `RefreshIconButton`), abrindo um modal custom e renderizando o `<SparkGrid>` principal.
3. Vai desativar filtros padrões de coluna do grid, focando-os nos botões e formatando colunas usando `date.ts`.
4. Os Actions do grid invocarão um Alert.confirm para deletar.
5. No clique duplo emitirá ativação para edição que mostrará a janela complexa com as abas condicionalmente intituladas: ["Dados Gerais - #2"] e ["Logs (4)"].
6. Em data provider não terá concatenação de `.data`.
