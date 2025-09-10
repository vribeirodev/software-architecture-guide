# software-architecture-guide

Guia prático de sugestão de arquitetura ( NÃO SÃO MANDAMENTOS ESCRITOS EM PEDRA )

## Índice

1. [Análise inicial](#1-análise-inicial---o-que-estou-construindo)
2. [Estruturas por complexidade](#2-estrutura-por-complexidade)
3. [Decisão: quando criar cada camada](#3-decisão-quando-criar-cada-camada)
4. [Framework de integrações](#4-framework-de-integrações)
5. [Templates por cenário](#5-templates-por-cenário)
6. [Framework de nomenclatura](#6-framework-de-nomenclatura)
7. [Exemplos práticos](#7-exemplos-práticos)
8. [Checklist de validação](#8-checklist-de-validação)
9. [Patterns específicos](#9-patterns-específicos)
10. [Regras finais](#10-regras-finais)

    

## 1. ANÁLISE INICIAL - "O QUE ESTOU CONSTRUINDO?"

### Passo 1A: Identifique o Tipo de Operação
- [ ] É uma consulta (query)? → Caminho mais simples
- [ ] É uma operação (command)? → Caminho mais complexo  
- [ ] É um processamento assíncrono? → Mensageria
- [ ] É integração externa? → Gateway/Adapter pattern

### Passo 1B: Identifique a Complexidade
- **SIMPLES** = 1 fonte de dados + validação básica + sem side effects
- **MÉDIA** = 2-3 fontes + validações + 1-2 side effects  
- **COMPLEXA** = múltiplas fontes + orquestração + múltiplos side effects
- **CRÍTICA** = tudo acima + compensação + auditoria + compliance

```mermaid
---
config:
  theme: mc
  look: classic
  layout: dagre
---
flowchart TD
    A["Nova Funcionalidade"] --> B{"É uma consulta<br>ou operação?"}
    B L_B_C_0@-- Consulta --> C["Caminho Simples"]
    B L_B_D_0@-- Operação --> D{"Quantas fontes<br>de dados?"}
    D L_D_E_0@-- 1 fonte --> E{"Tem validação<br>básica apenas?"}
    D L_D_F_0@-- "2-3 fontes" --> F["Complexidade MÉDIA"]
    D L_D_G_0@-- Múltiplas --> G["Complexidade COMPLEXA"]
    E L_E_H_0@-- Sim --> H{"Sem side<br>effects?"}
    E L_E_F_0@-- Não --> F
    H L_H_I_0@-- Sim --> I["Complexidade SIMPLES<br>Rota → UseCase → Repository"]
    H L_H_F_0@-- Não --> F
    F --> J["Rota → UseCase → Repository + Services<br>2-3 fontes + 1-2 side effects"]
    G L_G_K_0@--> K{"Precisa compensação<br>+ auditoria?"}
    K L_K_L_0@-- Não --> L["Múltiplas fontes + orquestração<br>+ múltiplos side effects"]
    K L_K_M_0@-- Sim --> M["Complexidade CRÍTICA<br>+ compensação + compliance"]
    C L_C_N_0@--> N{"É processamento<br>assíncrono?"}
    N L_N_O_0@-- Sim --> O["Usar Mensageria ou corrotinas"]
    N L_N_P_0@-- Não --> P{"É integração<br>externa?"}
    P L_P_Q_0@-- Sim --> Q["Gateway/Adapter Pattern"]
    P L_P_I_0@-- Não --> I
     A:::Sky
     B:::Peach
     C:::Sky
     D:::Peach
     E:::Peach
     F:::Sky
     G:::Sky
     H:::Peach
     I:::Aqua
     J:::Aqua
     K:::Peach
     L:::Aqua
     M:::Aqua
     N:::Peach
     O:::Pine
     O:::Aqua
     P:::Peach
     Q:::Aqua
    classDef Peach stroke-width:1px, stroke-dasharray:none, stroke:#FBB35A, fill:#FFEFDB, color:#8F632D
    classDef Pine stroke-width:1px, stroke-dasharray:none, stroke:#254336, fill:#27654A, color:#FFFFFF
    classDef Sky stroke-width:1px, stroke-dasharray:none, stroke:#374D7C, fill:#E2EBFF, color:#374D7C
    classDef Aqua stroke-width:1px, stroke-dasharray:none, stroke:#46EDC8, fill:#DEFFF8, color:#378E7A
    style A stroke-width:4px,stroke-dasharray: 0,stroke:#000000,fill:#E1BEE7,color:#000000
    style B stroke-width:4px,stroke-dasharray: 0,color:#000000
    style C stroke-width:4px,stroke-dasharray: 0,color:#000000
    style D stroke-width:4px,stroke-dasharray: 0,color:#000000
    style E stroke-width:4px,stroke-dasharray: 0,color:#000000
    style F color:#000000,stroke-width:4px,stroke-dasharray: 0
    style G stroke-width:4px,stroke-dasharray: 0,color:#000000
    style H color:#000000,stroke-width:4px,stroke-dasharray: 0
    style I color:#000000,stroke-width:4px,stroke-dasharray: 0
    style J stroke-width:4px,stroke-dasharray: 0,color:#000000
    style K stroke-width:4px,stroke-dasharray: 0,color:#000000
    style L stroke-width:4px,stroke-dasharray: 0,color:#000000
    style M stroke-width:4px,stroke-dasharray: 0,color:#000000
    style N stroke-width:4px,stroke-dasharray: 0,color:#000000
    style O stroke-width:4px,stroke-dasharray: 0,color:#000000
    style P stroke-width:4px,stroke-dasharray: 0,color:#000000
    style Q stroke-width:4px,stroke-dasharray: 0,color:#000000
    L_B_C_0@{ animation: slow } 
    L_B_D_0@{ animation: slow } 
    L_D_E_0@{ animation: slow } 
    L_D_F_0@{ animation: slow } 
    L_D_G_0@{ animation: slow } 
    L_E_H_0@{ animation: slow } 
    L_E_F_0@{ animation: slow } 
    L_H_I_0@{ animation: slow } 
    L_H_F_0@{ animation: slow } 
    L_G_K_0@{ animation: slow } 
    L_K_L_0@{ animation: slow } 
    L_K_M_0@{ animation: slow } 
    L_C_N_0@{ animation: slow } 
    L_N_O_0@{ animation: slow } 
    L_N_P_0@{ animation: slow } 
    L_P_Q_0@{ animation: slow } 
    L_P_I_0@{ animation: slow }
```

## 2. ESTRUTURA POR COMPLEXIDADE

### Nível SIMPLES
```python
# Estrutura: Rota → UseCase → Repository
GET /accounts/{document}

class GetCustomerAccountsUseCase:
    def __init__(self, account_repo: AccountRepository):
        self._account_repo = account_repo
    
    async def execute(self, document: str) -> List[Account]:
        return await self._account_repo.find_by_document(document)

# SEM services intermediários - direto pro repository
```

### Nível MÉDIA
```python
# Estrutura: Rota → UseCase → Repository + Cache(se aplicavel)
POST /instant-payment/simulate

class SimulateInstantLoanUseCase:
    def __init__(self, 
                 account_repo: AccountRepository,
                 credit_params_repo: CreditParametersRepository,
                 loan_calculator: LoanCalculator):
    
    async def execute(self, cmd: SimulateCommand) -> SimulationResult:
        # Busca dados
        account = await self._account_repo.find_by_id(cmd.account_id)
        params = await self._credit_params_repo.get_by_account_id(cmd.account_id)
        
        # Processa (service/domain)
        return self._loan_calculator.calculate(account, params, cmd.amount)

# Use case orquestra, mas sem services intermediários
```

### Nível COMPLEXA
```python
# Estrutura: Rota → UseCase → Multiple Services → Multiple Repos + External
POST /instant-payment/execute

class ExecuteInstantPaymentUseCase:
    def __init__(self, 
                 data_service: PaymentDataPreparationService,
                 validation_service: PaymentValidationService, 
                 execution_service: PaymentExecutionService):
                 
    async def execute(self, cmd: InstantPaymentCommand):
        data = await self._data_service.prepare(cmd)
        await self._validation_service.validate(cmd, data)  
        return await self._execution_service.process(cmd, data)

# Use case coordena services, services coordenam repositories
```

## 3. DECISÃO: QUANDO CRIAR CADA CAMADA

### Service Layer - Quando Criar
**✅ CRIE Service quando:**
- [ ] Precisa coordenar 2+ repositories
- [ ] Tem lógica de orquestração media ou complexa  
- [ ] Precisa fazer calls externos + internos
- [ ] Tem transações distribuídas
- [ ] Precisa de retry/circuit breaker

**❌ NÃO crie Service quando:**
- [ ] É só buscar dados e retornar
- [ ] É validação simples
- [ ] É cálculo direto sem side effects

### Repository - Quando Separar
**✅ SEPARE Repository quando:**
- [ ] Agregados de domínio diferentes
- [ ] Ciclos de vida independentes
- [ ] Fontes de dados diferentes
- [ ] Padrões de acesso diferentes

**✅ MANTENHA JUNTO quando:**
- [ ] Sempre acessados juntos
- [ ] Forte acoplamento de dados  
- [ ] Mesma transação sempre
- [ ] Mesmo bounded context

## 4. FRAMEWORK DE INTEGRAÇÕES

### APIs Internas (Microserviços)
```python
# Padrão: Gateway + Adapter
class IPaymentGateway:  # Interface
    async def create_payment_order() -> PaymentResult

class BankPaymentGateway(IPaymentGateway):  # Implementação
    def __init__(self, http_client: HttpClient, config: BankConfig)
    
# Use Case usa Gateway (abstração)
# Não conhece implementação específica
```

### APIs de Terceiros
```python
# Padrão: External Service + Circuit Breaker
class InstantPaymentKeyService:
    def __init__(self, 
                 central_bank_client: CentralBankClient,
                 circuit_breaker: CircuitBreaker,
                 cache: RedisClient):
                 
    async def get_payment_key_info(self, key: str):
        # Circuit breaker + cache(se aplicavel) + retry
        return await self._central_bank_client.get_key_info(key)
```

### Redis (Cache/Session)
```python
# Decisão por tipo de dado:
CACHE_LOOKUP     = dados que não mudam frequentemente
SESSION_DATA     = dados da sessão do usuário  
DISTRIBUTED_LOCK = coordenação entre instâncias
RATE_LIMITING    = controle de taxa

class PaymentKeyLookupCache:  # Cache específico
    async def get_cached_key_info(self, key: str) -> Optional[PaymentKeyInfo]
    async def cache_key_info(self, key: str, info: PaymentKeyInfo, ttl: int)
```

### Mensageria (Eventos/Comandos)
```python
# Padrão por tipo:
EVENTO       = algo aconteceu (PaymentCompleted)
COMANDO      = faça algo (ProcessPaymentWebhook) 
NOTIFICAÇÃO  = informe algo/alguém (NotifyPaymentStatus)

class InstantPaymentEventPublisher:
    async def publish_payment_completed(self, event: PaymentCompletedEvent)
    
class WebhookEventProcessor:
    async def handle_payment_webhook(self, event: PaymentWebhookEvent)
```

## 5. TEMPLATES POR CENÁRIO

### Template: Consulta Simples
```
Controller → UseCase → Repository
                     ↓
                    (Cache opcional se aplicavel)
```

### Template: Operação Média
```  
Controller → UseCase → Repository
                     → Validation Service 
                     → Domain Service
                     → Cache/External API (opcional se aplicavel)
```

### Template: Operação Complexa
```
Controller → UseCase → Data Preparation Service → Multiple Repos
                     → Validation Service → External APIs  
                     → Execution Service → Cache/Message Queue
                     → Compensation Service (se falhar)
```

## 6. FRAMEWORK DE NOMENCLATURA

### Testes de Validação de Nomes

#### Teste da Explicação para leigos
**Pergunta:** "Como explicaria essa classe para outra pessoa?"

```python
# ❌ Ruim:
PaymentDataService  → "É um... eh... serviço que... prepara dados de pagamento"

# ✅ Bom:  
InstantLoanDataPreparationService → "Pega todas as informações necessárias para fazer um empréstimo instantâneo"
```
**Regra:** Se você hesita ou usa palavras vagas ("dados", "manager", "handler"), o nome está ruim.

#### Teste da Conversa Telefônica
**Pergunta:** "Se eu ligar para alguém e falar só o nome da classe, a pessoa entende o que ela faz?"

```python
# ❌ Confuso:
"Tem um bug no PaymentService"  → "Qual PaymentService? O que ele faz?"

# ✅ Claro:
"Tem um bug no InstantLoanExecutionService" → "Ah, o que executa empréstimos instantâneos"
```

### Templates de Nomenclatura

#### Para APIs/Microserviços
```python
# Use Cases
[Action][BusinessConcept]UseCase
# Exemplos: ExecuteInstantPaymentUseCase, ProcessWebhookNotificationUseCase

# Services  
[BusinessConcept][TechnicalOperation]Service
# Exemplos: PaymentDataPreparationService, KYCValidationService

# Repositories
[DomainAggregate]Repository
# Exemplos: InstantPaymentRepository, CustomerAccountRepository

# Controllers
[BusinessConcept]Controller
# Exemplos: InstantPaymentController, OpenBankingController

# DTOs
[BusinessConcept][Direction]DTO
# Exemplos: InstantPaymentRequestDTO, AccountBalanceResponseDTO
```

#### Para Webhooks/Eventos
```python
# Handlers
[EventType][BusinessConcept]Handler
# Exemplos: PaymentWebhookHandler, TransactionNotificationHandler

# Processors  
[EventType][BusinessConcept]Processor
# Exemplos: PaymentWebhookProcessor, RefundProcessor

# Publishers
[BusinessConcept]EventPublisher
# Exemplos: TransferEventPublisher, CustomerAccountEventPublisher
```

### Hierarquia de Especificidade
Sempre do mais específico para o mais genérico:

```python
# Ordem de preferência:
1. InstantLoanPaymentExecutionService     # ultra específico
2. InstantLoanExecutionService           # específico  
3. InstantPaymentService                 # genérico
4. PaymentService                        # muito genérico ❌
5. FinancialService                      # genérico demais ❌
```
**Regra:** Pare no primeiro nível onde não fica redundante demais.

### Tips and tricks para Nomenclatura

1. **Prefira verbosidade a ambiguidade**
   - `LoanDataPreparationService` > `DataService`

2. **Use o vocabulário do negócio**
   - `InterestCalculator` > `FinancialMathService`

3. **Seja consistente no projeto**
   - Se usa `Service` para orquestração, sempre use `Service`

4. **Evite abreviações**
   - `LoanParametersRepository` > `LoanParamsRepo`

5. **Una responsabilidade ao nome**
   - `CustomerValidationService` faz só validação
   - `CustomerDataService` é vago demais

## 7. EXEMPLOS PRÁTICOS

### Cenário: Consulta de Saldo (SIMPLES)
```python
GET /accounts/{account_id}/balance

class GetAccountBalanceUseCase:
    def __init__(self, account_repo: AccountRepository):
        self._account_repo = account_repo
    
    async def execute(self, account_id: str) -> AccountBalance:
        return await self._account_repo.get_balance(account_id)
```

### Cenário: Simulação de Empréstimo (MÉDIA)
```python
POST /loans/simulate

class SimulateLoanUseCase:
    def __init__(self, 
                 customer_repo: CustomerRepository,
                 credit_score_service: CreditScoreService,
                 loan_calculator: LoanCalculator):
    
    async def execute(self, request: LoanSimulationRequest) -> LoanSimulation:
        customer = await self._customer_repo.find_by_id(request.customer_id)
        credit_score = await self._credit_score_service.get_score(customer.document)
        return self._loan_calculator.simulate(customer, credit_score, request.amount)
```

### Cenário: Pagamento Instantâneo (COMPLEXA)
```python
POST /instant-payments/execute

class ExecuteInstantPaymentUseCase:
    def __init__(self, 
                 payment_data_service: PaymentDataPreparationService,
                 anti_fraud_service: AntiFraudValidationService,
                 payment_execution_service: PaymentExecutionService,
                 notification_service: PaymentNotificationService):
                 
    async def execute(self, command: InstantPaymentCommand) -> PaymentResult:
        # Preparar dados
        payment_data = await self._payment_data_service.prepare(command)
        
        # Validar fraude
        await self._anti_fraud_service.validate(payment_data)
        
        # Executar pagamento
        result = await self._payment_execution_service.process(payment_data)
        
        # Notificar (assíncrono)
        await self._notification_service.notify_payment_executed(result)
        
        return result
```

### Cenário: Open Banking - Consentimento (COMPLEXA)
```python
POST /open-banking/consents

class CreateConsentUseCase:
    def __init__(self,
                 consent_validation_service: ConsentValidationService,
                 customer_data_service: CustomerDataPreparationService,
                 consent_storage_service: ConsentStorageService,
                 audit_service: ComplianceAuditService):
                 
    async def execute(self, request: ConsentRequest) -> ConsentResponse:
        # Validar requisição
        await self._consent_validation_service.validate(request)
        
        # Preparar dados do cliente
        customer_data = await self._customer_data_service.prepare(request.customer_id)
        
        # Armazenar consentimento
        consent = await self._consent_storage_service.store(request, customer_data)
        
        # Auditoria para compliance
        await self._audit_service.log_consent_creation(consent)
        
        return ConsentResponse.from_consent(consent)
```

## 8. CHECKLIST DE VALIDAÇÃO

### Para Cada Nova Feature
- [ ] Classifiquei corretamente a complexidade?
- [ ] Usei a estrutura mínima necessária?
- [ ] Cada service tem responsabilidade única?
- [ ] Abstraí corretamente as dependências externas?
- [ ] Considerei falhas e compensação?
- [ ] Nomes seguem o framework de nomenclatura?
- [ ] Posso testar cada camada isoladamente?
- [ ] Considerei aspectos de compliance/auditoria?

### Red Flags - Sinais de Alerta
- [ ] Use Case com MUITAS dependências
- [ ] Service que chama outro service diretamente  
- [ ] Repository que faz algo alem de persistir ou consultar dados
- [ ] Controller com lógica de negócio
- [ ] Nomes genéricos (DataService, Helper, Manager)
- [ ] Operações criticas sem auditoria
- [ ] APIs externas sem circuit breaker ou retry

## 9. PATTERNS ESPECÍFICOS

### Anti-Fraude
```python
class AntiFraudValidationService:
    async def validate_transaction(self, transaction: Transaction) -> FraudResult:
        # Múltiplas validações em paralelo ou concorrente no caso de Python ou semelhantes 
        results = await asyncio.gather(
            self._device_validation.validate(transaction.device_info),
            self._behavior_analysis.analyze(transaction.customer_id),
            self._amount_validation.validate(transaction.amount),
            self._velocity_check.check(transaction.customer_id)
        )
        return self._fraud_scorer.calculate_risk(results)
```

### Conciliação Bancária
```python
class BankReconciliationService:
    async def reconcile_statements(self, date: date) -> ReconciliationReport:
        internal_transactions = await self._transaction_repo.get_by_date(date)
        bank_statements = await self._bank_gateway.get_statements(date)
        
        return self._reconciler.match_transactions(internal_transactions, bank_statements)
```

### KYC/AML
```python
class KYCValidationService:
    async def validate_customer(self, customer: Customer) -> KYCResult:
        # Validações obrigatórias por regulamentação
        cpf_validation = await self._cpf_validator.validate(customer.document)
        pep_check = await self._pep_service.check(customer.document)
        sanctions_check = await self._sanctions_service.check(customer.document)
        
        return KYCResult(cpf_validation, pep_check, sanctions_check)
```

## 10. REGRAS FINAIS

1. **Comece Simples → Evolua para Complexo**
   - Primeira implementação: mínimo necessário, faça funcionar primeiro
   - Reorganize ou refatore enquanto a complexidade aumenta

2. **Uma Responsabilidade por Classe**
   - Se não consegue explicar em uma frase, está fazendo demais

3. **Abstraia External Dependencies**
   - Use case nunca deve conhecer implementação externa

4. **Falhas são Parte do Design**
   - Sempre pense no que pode falhar e como compensar

5. **Teste da Substituição**
   - Se mudar implementação externa, quantas classes preciso alterar?

6. **Compliance é Requisito, Não Opção**
   - Auditoria e logs devem ser considerados desde o início

7. **Segurança em Camadas**
   - Validações em múltiplos pontos, especialmente para operações criticas
