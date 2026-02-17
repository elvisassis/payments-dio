# Plataforma de Pagamentos - Demo de Padrões de Projeto

## 1. Visão Geral

Este projeto é uma aplicação Spring Boot desenvolvida com um propósito didático: **demonstrar a aplicação prática de diversos Padrões de Projeto (Design Patterns) e boas práticas de arquitetura de software** em um cenário do mundo real.

A aplicação simula um serviço de pagamentos que se integra a múltiplos provedores (como Stripe e PayPal), oferecendo uma API unificada para processar transações. A complexidade de orquestrar diferentes provedores e reagir a eventos de pagamento serve como um excelente pano de fundo para explorar como os padrões de projeto ajudam a criar um software mais flexível, manutenível e escalável.

## 2. Padrões de Projeto e Arquitetura Aplicados

Este é o coração do projeto. Abaixo estão os principais padrões implementados e onde encontrá-los no código.

### a. Adapter

- **Intenção:** Converter a interface de uma classe para outra interface que o cliente espera. O Adapter permite que classes com interfaces incompatíveis trabalhem juntas.
- **Implementação no Projeto:**
    - A interface `provider/PaymentProvider.java` define o contrato único que o sistema usa para interagir com qualquer provedor de pagamento.
    - As classes no pacote `provider/adapter/` (ex: `StripePaymentAdapter.java`, `PaypalPaymentAdapter.java`) são os **adaptadores**. Cada uma "traduz" a chamada do nosso sistema para a chamada que a API específica do provedor (simulada) esperaria, unificando-as sob a mesma interface `PaymentProvider`.

### b. Facade

- **Intenção:** Fornecer uma interface unificada e simplificada para um conjunto de interfaces em um subsistema. A Facade define uma interface de nível mais alto que torna o subsistema mais fácil de usar.
- **Implementação no Projeto:**
    - A classe `facade/PaymentFacade.java` atua como uma fachada para o complexo subsistema de pagamentos.
    - O `controllers/PaymentController.java` não precisa conhecer todos os componentes internos (use cases, serviços, resolvedores). Ele simplesmente se comunica com a `PaymentFacade`, que orquestra a chamada para os componentes corretos, seja para criar um pagamento (`CreatePaymentUseCase`) ou para consultá-lo (`PaymentService`).

### c. Observer (via Spring Events)

- **Intenção:** Definir uma dependência um-para-muitos entre objetos, de modo que, quando um objeto muda de estado, todos os seus dependentes são notificados e atualizados automaticamente.
- **Implementação no Projeto:**
    - O projeto utiliza a implementação idiomática do Spring para este padrão.
    - **Publicador:** O `useCases/CreatePaymentUseCase.java` publica eventos como `PaymentApprovedEvent` ou `PaymentFailedEvent` após processar um pagamento. Ele não sabe (e não precisa saber) quem irá reagir a esses eventos.
    - **Ouvintes (Listeners):** Classes no pacote `listener/` (ex: `PaymentEmailListener.java`, `PaymentAuditListener.java`) são os observadores. Elas usam a anotação `@EventListener` para se inscreverem nos eventos e executar ações (como enviar um e-mail ou registrar uma auditoria) de forma totalmente desacoplada do fluxo principal.

### d. Strategy / Factory

- **Intenção:** A Factory cria objetos, enquanto a Strategy permite que o algoritmo exato varie independentemente do cliente que o utiliza.
- **Implementação no Projeto:**
    - A classe `resolver/PaymentProviderResolver.java` atua como uma combinação desses padrões. O Spring injeta nela uma lista de todas as *estratégias* de pagamento disponíveis (todos os beans que implementam `PaymentProvider`).
    - O resolver então atua como uma *Factory*, selecionando e retornando a instância correta do provedor com base no nome fornecido na requisição.

### e. Arquitetura Orientada a Use Cases (CQRS)

- **Intenção:** Não é um padrão de projeto clássico, mas um padrão arquitetural. CQRS (Command Query Responsibility Segregation) prega a separação entre operações que alteram o estado do sistema (Commands) e operações que leem o estado (Queries).
- **Implementação no Projeto:**
    - **Comando:** A classe `useCases/CreatePaymentUseCase.java` é um "Manipulador de Comando". Ela encapsula toda a lógica para uma única ação: criar um pagamento.
    - **Consulta:** A classe `service/PaymentService.java` foi simplificada para atuar como um "Manipulador de Consulta", responsável apenas por buscar dados.
    - Essa separação torna o sistema mais claro, fácil de testar e escalável.

## 3. Estrutura do Projeto

```
/src/main/java/br/com/elvisassis/payments/
├───controllers/   # Camada de API (entrada de requisições)
├───facade/        # Padrão Facade: Ponto de entrada simplificado para o sistema
├───useCases/      # Padrão CQRS (Commands): Lógica de negócio para casos de uso de escrita
├───service/       # Padrão CQRS (Queries): Lógica de negócio para casos de uso de leitura
├───provider/      # Padrão Adapter: Contém a interface e os adaptadores para os provedores
│   └───adapter/
├───resolver/      # Padrão Strategy/Factory: Resolve qual provedor de pagamento usar
├───event/         # Padrão Observer: Classes que modelam os eventos de negócio
├───listener/      # Padrão Observer: Classes que reagem aos eventos
├───domain/        # Entidades e objetos de domínio
├───dto/           # Data Transfer Objects (para a camada de API)
└───repository/    # Interfaces de acesso ao banco de dados (Spring Data JPA)
```

## 4. Como Executar

1.  Clone o repositório.
2.  Certifique-se de ter o Java (JDK 17+) e o Maven instalados.
3.  Navegue até a raiz do projeto e execute o seguinte comando no seu terminal:

    ```bash
    ./mvnw spring-boot:run
    ```
4.  A aplicação estará disponível em `http://localhost:8080`.

## 5. API para Demonstração

Para testar o fluxo principal, envie uma requisição `POST` para o endpoint abaixo.

**Endpoint:** `POST /payments/`

**Exemplo de Corpo da Requisição (Body):**

```json
{
  "amount": 100.50,
  "currency": "BRL",
  "method": "CREDIT_CARD",
  "provider": "Stripe"
}
```

- Você pode alterar o `"provider"` para `"Paypal"` para ver o `PaymentProviderResolver` selecionando o outro adaptador.
- No console da aplicação, você verá os logs indicando qual provedor processou o pagamento e os listeners (e-mail, auditoria) reagindo aos eventos de sucesso ou falha.

## 6. Tecnologias

-   Java 25
-   Spring Boot 4
-   Spring Data JPA
-   Maven
-   PostegreSQL