💳 Especificação Técnica: API de Checkout com Split de Pagamento
Versão: 2.1 (Enterprise) Contexto: Transações onde o valor pago pelo cliente é dividido entre o Marketplace (comissão) e o Vendedor (seller).

1. Regras de Negócio de Fluxo (Business Rules)
RN01 - Cálculo de Split: O Marketplace retém uma comissão fixa de 15% sobre o valor de cada item.

RN02 - Validação de Estoque: O checkout deve validar a disponibilidade via microsserviço de inventário antes de processar o pagamento.

RN03 - Idempotência: Toda requisição deve conter o header X-Idempotency-Key para evitar cobranças duplicadas em caso de retentativa.

RN04 - Anti-Fraude: Transações acima de R$ 5.000,00 devem ser enviadas com o status PENDING_REVIEW.

2. Documentação da API (Endpoints)
POST /v2/orders
Autenticação: Bearer Token (JWT)

Payload (Exemplo):

JSON
{
  "order_ref": "ORD-99821",
  "customer": {
    "id": "cust_001",
    "ip_address": "192.168.1.1"
  },
  "cart": {
    "currency": "BRL",
    "items": [
      {
        "product_id": "prod_abc",
        "seller_id": "seller_xpto",
        "amount": 1000.00,
        "quantity": 1
      }
    ]
  },
  "payment_method": {
    "type": "credit_card",
    "token": "card_tok_882193"
  }
}
Respostas Esperadas:

201 Created: Transação processada.

422 Unprocessable Entity: Saldo insuficiente ou erro no cálculo de split.

409 Conflict: Idempotency Key já utilizada.

3. Arquitetura de Dados (Database Schema)
Table orders: id (UUID), status (enum), total_amount (decimal), idempotency_key (unique).

Table split_rules: id (PK), order_id (FK), recipient_id (string), share_amount (decimal).

Table inventory_hold: product_id (PK), quantity_reserved (int).

4. Webhooks (Notificações Assíncronas)
Após o processamento adquirente, o sistema envia um POST para a URL do vendedor: { "event": "payment.succeeded", "order_id": "uuid", "split_confirmed": true }