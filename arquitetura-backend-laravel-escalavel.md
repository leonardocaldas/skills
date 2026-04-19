# Skill: Arquitetura Backend Laravel Escalável (Padrão CQRS / Clean Code)

## Objetivo
Você atua como um Engenheiro de Software Sênior especialista em Laravel e Arquitetura de Software. Sua missão é projetar, refatorar ou implementar funcionalidades no backend utilizando uma arquitetura escalável orientada a **CQRS (Command Query Responsibility Segregation)**, seguindo rigorosamente as diretrizes abaixo.

Sempre que o usuário pedir para criar um novo módulo, endpoint, ou refatorar algo, **DEVE** aplicar esta estrutura de forma obrigatória.

---

## Padrões Arquiteturais Obrigatórios

### 1. Roteamento via Atributos (Spatie Route Attributes)
*   **Regra:** Não utilize os arquivos de rota tradicionais (`routes/api.php`, `routes/web.php`). As rotas devem ser autodeclaradas nos próprios Controllers utilizando os atributos da biblioteca Spatie.
*   **Aplicação:** 
    *   No Controller: `#[Middleware("internal_api")]`, `#[Prefix("/recurso")]`
    *   Nos Métodos: `#[Get("/")]`, `#[Post("/")]`, `#[Put("/{id}")]`, `#[Delete("/{id}")]`.

### 2. Form Requests como Camada de Tradução
*   **Regra:** O Controller não deve possuir regras de validação nem instanciar DTOs ou Commands manualmente com dezenas de `request->input(...)`.
*   **Aplicação:** Todo input complexo (POST, PUT) deve usar uma classe `FormRequest` dedicada (Ex: `CreateResourceFormRequest`). 
*   **Responsabilidade extra:** O FormRequest é responsável por mapear os dados validados para a classe de Command correspondente através de métodos dedicativos.
    *   Exemplo prático no controller: `$handler->handle($request->toCreateCommand());` ou `$request->toUpdateCommand($id);`

### 3. Separação Estrita de Responsabilidade (CQRS Lite)

A regra de ouro do Controller é atuar **apenas** como um orquestrador.

#### A. Command Handlers (Mutação de Estado)
*   Para toda ação que modifique o estado do banco de dados (Create, Update, Delete, Sync, Approve, etc.), crie um `Command` (um DTO puro com campos tipados) e um `CommandHandler` associado.
*   **Convenção de nomenclatura:** `Create[Entity]Command` e `Create[Entity]CommandHandler`.
*   Passe dependências no construtor do Handler, passe dados de entrada exclusivamentamente através do objeto Command.

#### B. Query Handlers (Consultas Complexas)
*   Quando a consulta exigir JOINs, agregações, exportação (Excel/PDF), regras de negócio antes de responder ou processamentos em múltiplos passos, utilize um `QueryHandler`.
*   **Padrão de uso:** Criar `GetPaginated[Entity]QueryHandler` ou `FindAggregated[Entity]ByIdQueryHandler`. Recebem um objeto `Query` se necessário e retornam um objeto `Response` tipado (ex: `FindAggregatedByIdResponse`) ou um construtor de listagem (`QueryBuilderResult`).

#### C. Repositories (Consultas Simples)
*   Se a consulta for apenas um find simples ou listagem direta na tabela sem agregação ou lógica extra: injete o `Repository` diretamente no Controller.
*   Exemplo no controller: `return $this->repository->findById($id);` (Não é necessário Query Handler para isso).

### 4. DTOs (Commands / Queries / Responses)
*   Sempre utilize objetos tipados para passar dados entre Camadas. Nunca passe vetores genéricos (`array $data`) pelo Handler.
*   Atributos do DTO garantem tipagem forte, previsibilidade (usando `readonly` por exemplo) e facilitam refatorações futuras.

### 5. SOLID & Clean Code nos Handlers
A essência das regras de negócio vive agora nos Handlers. Portanto:
*   Mantenha os Handlers coesos (Single Responsibility). Eles não formatam HTTP e também não mandam e-mail sozinhos (acionam um Mailer/Event se necessário).
*   Prefira múltiplos métodos privados no Handler com nomes altamente descritivos para dividir os passos da alteração.
*   Siga as práticas de código limpo (early returns, evite else/elseif prolongados, injete TODAS dependências no construtor).

### 6. Padrões de Migration (Performance FIRST)
Mude a sua forma padrão de criar relacionamentos e tabelas no Laravel:
*   **SEM Foreign Key Constraints:** Não utilize `$table->foreignId('...')->constrained()`. Nós omitimos FKs a nível de banco para favorecer alta escalabilidade, particionamento e escritas rápidas em tabelas volumosas.
*   **USE Índices Fortes:** Apesar da ausência de restrições de chave estrangeira, você DEVE declarar explicitamente os campos de ID (ex: `$table->unsignedBigInteger('user_id')`) e obrigatoriamente acoplar um index (`$table->index('user_id')`) para otimização de consultas e joins.

---

## Exemplo Típico de um Controller Escalável

```php
#[Middleware("internal_api")]
#[Prefix("/orders")]
class OrderController
{
    public function __construct(
        private OrderRepository $orderRepository,
        private CreateOrderCommandHandler $createOrderCommandHandler,
        private FindAggregatedOrderByIdQueryHandler $findAggregatedQueryHandler,
    ) {}

    // Apenas listagem simples -> Usamos algo genérico ou um Handler específico.
    #[Get("/{id}")]
    public function findById(int $id): Order
    {
        return $this->orderRepository->findById($id);
    }

    // Consulta Complexa -> Delega para QueryHander (Query/Response Pattern)
    #[Get("/{id}/aggregated")]
    public function findAggregatedById(int $id): FindAggregatedByIdResponse
    {
        return $this->findAggregatedQueryHandler->handle(new FindAggregatedOrderByIdQuery($id));
    }

    // Mutante -> FormRequest traduz para um CommandDTo -> Envia para CommandHandler
    #[Post("/")]
    public function create(CreateOrderFormRequest $request): Order
    {
        return $this->createOrderCommandHandler->handle($request->toCreateCommand());
    }
}
```

## Como a IA deve operar sob esta Skill
Se o Usuário disser: *"Crie os endpoints e regras para o cadastro e fluxo de aprovação de 'Invoices'"*
1. **Você criará o InvoicesController contendo as rotas via Atributos.**
2. **Criará o InvoiceRepository.**
3. **Criará métodos HTTP apontando para Handlers (Ex: ApproveInvoiceCommandHandler) invés de programar dentro do controller.**
4. **Validará inputs no FormRequest e o usará como factory de DTO (Commands).**
5. **Escreverá as migrations focadas em indexes rápidos, sem constraints de restrição impeditiva.**
