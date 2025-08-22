
# GitHub Copilot Agent — Prompt de Refatoração
> Objetivo: migrar o cliente de API gerado por **swagger-typescript-api** para o cliente gerado por **openapi-typescript-codegen**, atualizando ~80+ arquivos de forma segura e consistente em um projeto TypeScript/React/Node.

## Contexto
- Cliente **origem**: swagger-typescript-api (padrões legados, chamadas com headers manuais e uso frequente de `.then/.catch`).
- Cliente **alvo**: openapi-typescript-codegen com estrutura padrão:
  - `src/api-client/`  
    - `services/` — um `Service` por recurso
    - `models/` — tipos gerados automaticamente
    - `core/` — runtime/config
    - `index.ts` — exportações
- Camada de configuração/autenticação: `ApiConfig` com `withAuth(fn)` e `withoutAuth(fn)` para padronizar headers, tokens e tratamento de erros.

## Regras de Refatoração (faça sempre)
1. **Imports**  
   - Remova imports do cliente legado.  
   - Adicione imports do novo cliente a partir de `src/api-client` (ex.: `import { OrderService } from "src/api-client";`).  
   - Importe `ApiConfig` (ex.: `import { ApiConfig } from "src/services/api-config";`), se aplicável.

2. **Assíncrono e Erros**  
   - Converta `.then/.catch` para `async/await` com `try/catch`.  
   - Centralize autenticação usando `ApiConfig.withAuth(() => Service.method(params))`.  
   - Chamadas **públicas** usam `ApiConfig.withoutAuth(() => Service.method(params))`.  
   - Padronize mensagens de erro (ex.: `console.error("Erro ao carregar X:", error)`).
   - **Não** misture `.then/.catch` dentro de `withAuth/withoutAuth`.

3. **Assinaturas e Tipos**  
   - Adapte parâmetros para a nova assinatura `{ id, ... }`.  
   - Preserve e propague tipos retornados; preferir tipos gerados em `models/`.  
   - Evite `any` e coerções desnecessárias.

4. **Efeitos Colaterais**  
   - Não altere regras de negócio.  
   - Não mude nomes/props do estado fora do necessário.  
   - Mantenha logs úteis, remova logs ruidosos.

5. **Checklist por arquivo**  
   - Remover imports legados ✔  
   - Importar novo `Service` e `ApiConfig` ✔  
   - Migrar chamadas autenticadas → `withAuth` ✔  
   - Migrar chamadas públicas → `withoutAuth` ✔  
   - `async/await` + `try/catch` ✔  
   - Validar tipos de entrada/saída ✔  
   - Executar testes manuais/automáticos ✔

## Exemplos (genéricos)

### Consumo de APIs Autenticadas
**Antes (legado):**
```ts
// clienteAntigo.getOrderById(id, { headers: authHeaders() })
clienteAntigo.getOrderById(id, { headers: authHeaders() })
  .then(r => setCurrentOrder(r.data))
  .catch(e => console.error("Error", e));
```

**Depois (novo):**
```ts
import { ApiConfig } from "src/services/api-config";
import { OrderService } from "src/api-client";

try {
  const response = await ApiConfig.withAuth(() =>
    OrderService.getOrderById({ id })
  );
  setCurrentOrder(response);
} catch (error) {
  console.error("Erro ao carregar pedido:", error);
  if ((error as any)?.status === 404) {
    // tratar 404
  }
}
```

### Consumo de APIs Publicas
**Antes (legado):**
```ts
clienteAntigo.listPublicOffers()
  .then(r => setOffers(r.data))
  .catch(e => console.error("Error", e));
```

**Depois (novo):**
```ts
import { ApiConfig } from "src/services/api-config";
import { OfferService } from "src/api-client";

try {
  const response = await ApiConfig.withoutAuth(() =>
    OfferService.listPublicOffers({ page, pageSize })
  );
  setOffers(response);
} catch (error) {
  console.error("Erro ao carregar ofertas públicas:", error);
}
```

## Padrões a Evitar
- ❌ Misturar padrões antigos e novos (ex.: `Service.method(..., { headers })` no novo cliente).
- ❌ Usar `.then/.catch` dentro de `withAuth/withoutAuth`.
- ❌ Introduzir `any` arbitrariamente.

## Dicas de Localização de Métodos
1. Identifique o recurso (Order, Company, User...).  
2. Abra `services/<Recurso>Service.ts`.  
3. Busque o método equivalente e ajuste parâmetros para `{ ... }`.

## Solicitação para o Copilot (use exatamente o texto abaixo)
> **Tarefa**: Refatore este(s) arquivo(s) do cliente legado para o novo cliente gerado por `openapi-typescript-codegen` seguindo as **Regras de Refatoração** deste documento.
>
> **Instruções**:
> 1) Altere imports para `src/api-client` e `ApiConfig`.  
> 2) Converta `.then/.catch` para `async/await` com `try/catch`.  
> 3) Use `ApiConfig.withAuth` para chamadas autenticadas e `ApiConfig.withoutAuth` para públicas.  
> 4) Adapte as assinaturas para objetos `{ ... }`.  
> 5) Preserve comportamento e tipos.  
> 6) Mostre o diff proposto (ou o bloco final) e explique mudanças.
>
> **Saída esperada**: Código final refatorado + resumo das mudanças.

## Pós-Refatoração
- Rode lint/format (`eslint`, `prettier`), testes e uma checagem de tipos (`tsc --noEmit`).  
- Faça commits pequenos por domínio (ex.: `feat(api): migrate order calls to new client`).  
- Abra PRs com nota de migração e plano de rollback.