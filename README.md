
# Refatoração em escala com VS Code + GitHub Copilot Agents
## Do `swagger-typescript-api` para `openapi-typescript-codegen` (ex.: 80+ arquivos, com segurança)

Migrar um cliente de API pode parecer assustador — especialmente quando impacta dezenas de arquivos. A boa notícia: com um **plano enxuto**, **padrões claros** e um **prompt bem escrito** para o GitHub Copilot Agents, dá para transformar uma maratona de retrabalho em um processo **repetível, auditável e rápido**.

> **Objetivo**: migrar o cliente de API gerado por `swagger-typescript-api` para o cliente gerado por `openapi-typescript-codegen` em um projeto TypeScript (React/Node), mantendo comportamento e melhorando tipagem, ergonomia e confiabilidade.

---

## Índice
- [Por que migrar](#por-que-migrar)
- [Ferramental](#ferramental)
- [Estrutura alvo do cliente](#estrutura-alvo-do-cliente)
- [Prompt de Refatoração (Copilot Agents)](#prompt-de-refatoração-copilot-agents)
- [Fluxo de trabalho recomendado](#fluxo-de-trabalho-recomendado)
- [Regras e padrões (o que mudar)](#regras-e-padrões-o-que-mudar)
- [Exemplos antes/depois](#exemplos-antesdepois)
- [Checklist por arquivo](#checklist-por-arquivo)
- [Validações (lint, tipos, testes)](#validações-lint-tipos-testes)
- [Estratégia de commits e PR](#estratégia-de-commits-e-pr)
- [Métricas de progresso](#métricas-de-progresso)
- [FAQ](#faq)

---

## Por que migrar
- **Consistência de chamadas**: serviços e métodos com nomenclatura previsível.
- **Tipagem forte e atualizada**: menos `any`, mais feedback do TypeScript.
- **Ergonomia de DX**: assinaturas orientadas a objeto (`{ id, ... }`) e runtime padronizado.
- **Base para automação**: estrutura por `services/models/core` facilita buscas e refactors em lote.

---

## Ferramental
- **VS Code** (com ESLint/Prettier/TypeScript).
- **GitHub Copilot Chat/Agents**.
- **Geradores**:
  ```bash
  # Cliente legado (exemplo)
  npx swagger-typescript-api -p ./openapi.json -o ./src/api-client-legacy -n index.ts

  # Novo cliente
  npx openapi-typescript-codegen --input ./openapi.json --output ./src/api-client
  ```
- **CI** para validar build, lint e testes.

> **Dica**: mantenha o cliente legado temporariamente para comparar comportamentos até concluir a migração.

---

## Estrutura alvo do cliente
```
src/
  api-client/
    services/   # 1 service por recurso
    models/     # tipos gerados
    core/       # runtime/config
    index.ts    # exportações principais
```
Essa organização facilita navegar por recursos e localizar métodos equivalentes no novo cliente.

---

## Prompt de Refatoração (Copilot Agents)
Use um prompt como “fonte da verdade” para alinhar o Copilot e aplicar sempre os mesmos padrões. Você pode baixar um exemplo pronto aqui:

- 👉 **[Copilot Agent — Refatoração (Markdown)](https://github.com/jefersonmlopes/copilot-refactor-prompt/copilot-refactor-prompt.md)**

Salve-o no repositório (ex.: `docs/copilot-refactor-prompt.md`) e consulte-o durante a migração.

> Se preferir, também há um arquivo enviado originalmente: `prompt-refactor.md`. Ajuste o conteúdo conforme suas preferências antes de usar.

---

## Fluxo de trabalho recomendado
1. **Crie uma branch de migração** (ex.: `feat/api-client-migration`).  
2. **Gere o novo cliente** com `openapi-typescript-codegen` em `src/api-client/`.  
3. **Padronize autenticação** em `ApiConfig` com `withAuth(fn)` e `withoutAuth(fn)`.  
4. **Abra o Copilot Chat** no VS Code e cole o **Prompt de Refatoração**.  
5. **Refatore por domínios** (Orders, Users, Companies...), em **lotes pequenos** e revisáveis.  
6. **Valide**: `tsc --noEmit`, `eslint`, testes.  
7. **Commits atômicos** por domínio; **PRs** com descrição clara e plano de rollback.

> Em um caso real, esse processo cobriu **80+ arquivos** com previsibilidade e poucas correções manuais.

---

## Regras e padrões (o que mudar)

### Imports
- Remova imports do cliente legado.
- Adicione imports do novo cliente a partir de `src/api-client` (ex.: `import { OrderService } from "src/api-client";`).
- Importe `ApiConfig` (ex.: `import { ApiConfig } from "src/services/api-config";`).

### Assíncrono e erros
- Converta `.then/.catch` para `async/await` com `try/catch`.
- Centralize autenticação com `ApiConfig.withAuth(() => Service.method(params))`.
- Chamadas **públicas** usam `ApiConfig.withoutAuth(() => Service.method(params))`.
- Padronize logs de erro (`console.error("Erro ao carregar X:", error)`).
- **Não** misture `.then/.catch` dentro de `withAuth/withoutAuth`.

### Assinaturas e tipos
- Adapte parâmetros para objetos `{ id, ... }`.
- Preserve e propague **tipos gerados** (preferir os de `models/`).
- Evite `any` e coerções desnecessárias.

### Efeitos colaterais
- Não altere regras de negócio.
- Não mude estado ou props fora do necessário.
- Mantenha logs úteis, remova ruído.

---

## Exemplos antes/depois

### Autenticadas
**Antes (legado)**
```ts
// clienteAntigo.getOrderById(id, { headers: authHeaders() })
clienteAntigo.getOrderById(id, { headers: authHeaders() })
  .then(r => setCurrentOrder(r.data))
  .catch(e => console.error("Error", e));
```

**Depois (novo)**
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

### Públicas
**Antes (legado)**
```ts
clienteAntigo.listPublicOffers()
  .then(r => setOffers(r.data))
  .catch(e => console.error("Error", e));
```

**Depois (novo)**
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

---

## Checklist por arquivo
- [ ] Remover imports do cliente legado  
- [ ] Importar novo `Service` e `ApiConfig`  
- [ ] Migrar chamadas **autenticadas** → `withAuth`  
- [ ] Migrar chamadas **públicas** → `withoutAuth`  
- [ ] Converter para `async/await` + `try/catch`  
- [ ] Validar tipos de entrada/saída  
- [ ] Executar testes manuais/automáticos  

---

## Validações (lint, tipos, testes)
```bash
# Tipos
pnpm tsc --noEmit    # ou yarn tsc --noEmit / npm run tsc -- --noEmit

# Lint/format
pnpm eslint .
pnpm prettier --check .

# Testes
pnpm test
```
> Adapte para o seu gerenciador de pacotes: `npm`, `yarn` ou `pnpm`.

---

## Estratégia de commits e PR
- **Commits pequenos por domínio** (ex.: `feat(api): migrate order calls to new client`).
- **Descrição do PR**:
  - escopo (quais services/rotas),
  - mudanças de assinatura relevantes,
  - impactos esperados,
  - plano de **rollback**,
  - instruções de **testes manuais**.
- Habilite **proteções** (CI obrigatória, revisões, squash/rebase conforme política).

---

## Métricas de progresso
- % de arquivos migrados por domínio.
- Build e testes verdes no CI.
- Erros de tipo (antes vs. depois).
- Tempo médio de revisão por PR.

---

## FAQ
**Posso manter partes do cliente antigo?**  
Sim, durante a transição. Evite “misturar” padrões no mesmo arquivo.

**Por que `withAuth/withoutAuth`?**  
Centraliza headers, tokens e tratamento de erros; reduz duplicação e bugs.

**E se o método mudou assinatura?**  
Adapte para o formato `{ ... }` e confira tipos gerados em `models/`.

---

> **Licença & Uso**: este guia é genérico e pode ser adaptado livremente. Remova ou ajuste seções conforme a realidade do seu projeto.

---

### Créditos
Este README foi pensado para ser usado tanto como **documentação interna** do repositório quanto como base para um **artigo público** (ex.: LinkedIn). Ajuste o tom conforme o canal.
