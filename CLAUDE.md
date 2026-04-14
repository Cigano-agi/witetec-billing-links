# WIA-272: Billing Links

Stack: NestJS (Node) + .NET 8 + React 18 + PostgreSQL 15 + Redis 7

## Comandos essenciais
- Testes Node: `cd node-api && npm test`
- Testes .NET: `cd dotnet-service-tests && dotnet test`
- Testes Frontend: `cd frontend && npm test`
- Subir infra: `docker compose up -d`

## Regras criticas
- name/cpf NUNCA em logs — PiiSanitizer obrigatorio
- seller_id sempre do JWT
- Idempotency-Key obrigatorio no endpoint publico
