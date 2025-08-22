# Como Refatorar 80+ Arquivos de Forma Segura com VS Code e GitHub Copilot 🚀

Você já enfrentou aquela situação onde precisa migrar um cliente de API e isso impacta dezenas de arquivos? 😰 Compartilho aqui uma estratégia que transformou uma tarefa que poderia levar semanas em um processo **organizado, repetível e auditável**.

## 🎯 O Desafio

Migrar de `swagger-typescript-api` para `openapi-typescript-codegen` em um projeto TypeScript React, atualizando **mais de 80 arquivos** sem quebrar funcionalidades existentes.

## 🛠️ A Estratégia Vencedora

### 1. **Planejamento é Tudo**

Antes de qualquer linha de código, definimos:

- ✅ Estrutura clara do novo cliente (services/models/core)
- ✅ Padrões de nomenclatura e assinaturas
- ✅ Estratégia de autenticação centralizada
- ✅ Checklist de validação por arquivo

### 2. **Prompt Engineering para GitHub Copilot**

O segredo foi criar um **arquivo de prompt estruturado** como "fonte da verdade":

```markdown
# Prompt de Refatoração - GitHub Copilot Agent

## Regras de Migração:
1. Converter .then/.catch para async/await
2. Centralizar autenticação com ApiConfig.withAuth()
3. Adaptar assinaturas para objetos { id, ... }
4. Preservar tipos e comportamentos
5. Padronizar tratamento de erros

## Exemplos Antes/Depois:
[Exemplos específicos da migração]
```

📋 **Prompt completo disponível em:** [GitHub - Copilot Refactor Prompt](https://github.com/jefersonmlopes/copilot-refactor-prompt/blob/main/copilot-refactor-prompt.md)

### 3. **Fluxo de Trabalho Repetível**

```text
📋 Para cada domínio (Orders, Users, etc.):
1. Abrir VS Code Copilot Chat
2. Colar o prompt de refatoração
3. Solicitar migração específica do arquivo
4. Revisar e validar mudanças
5. Executar testes (tsc, eslint, unit tests)
6. Commit atômico por domínio
```

## 🎯 Resultados Práticos

### ✅ **O que funcionou muito bem:**

- **Consistência**: Mesmos padrões aplicados em todos os arquivos
- **Velocidade**: 40-50 arquivos por hora vs. 5-10 manualmente
- **Qualidade**: Menos erros de tipagem e padrões mais limpos
- **Auditabilidade**: Commits pequenos e fáceis de revisar

### 📊 **Métricas do Processo:**

- 🕐 **Tempo**: 3 horas vs. estimativa inicial de 1 semanas
- 🐛 **Bugs pós-deploy**: Zero bugs relacionados à migração
- 📝 **Code Review**: PRs menores e mais focados
- 🔄 **Rollback**: Estratégia clara definida desde o início

## 💡 **Lições Aprendidas**

### 1. **Prompt bem estruturado = Resultado previsível**

O GitHub Copilot é poderoso, mas precisa de contexto claro. Um prompt detalhado com exemplos e regras específicas fez toda a diferença.

### 2. **Validação automatizada é essencial**

```bash
# Pipeline de validação
pnpm tsc --noEmit    # Verificação de tipos
pnpm eslint .        # Linting
pnpm test           # Testes unitários
```

### 3. **Commits pequenos salvam vidas**

Em vez de um PR gigante, fizemos commits por domínio. Facilita review e rollback se necessário.

## 🔧 **Ferramentas que Fizeram a Diferença**

- **VS Code** com extensões TypeScript
- **GitHub Copilot Chat** para refatoração assistida
- **ESLint + Prettier** para consistência de código
- **TypeScript strict mode** para detecção de problemas

## 🎯 **Principais Takeaways**

1. **Invista tempo no planejamento**: 20% do tempo planejando economiza 80% da execução
2. **Documente o processo**: Prompt reutilizável para futuras migrações
3. **Automatize validações**: Deixe as ferramentas encontrarem os problemas
4. **Commits atômicos**: Facilita review e possibilita rollback granular
5. **GitHub Copilot é um multiplicador**: Quando bem orientado, acelera drasticamente o processo


## ✨ **Solicitação para o Copilot (use exatamente o texto abaixo)**

```text
Tarefa: Refatore este(s) arquivo(s) do cliente legado para o novo cliente gerado por `openapi-typescript-codegen` seguindo as **Regras de Refatoração** deste documento.

Instruções:
1) Altere imports para `src/api-client` e `ApiConfig`.  
2) Converta `.then/.catch` para `async/await` com `try/catch`.  
3) Use `ApiConfig.withAuth` para chamadas autenticadas e `ApiConfig.withoutAuth` para públicas.  
4) Adapte as assinaturas para objetos `{ ... }`.  
5) Preserve comportamento e tipos.  
6) Mostre o diff proposto (ou o bloco final) e explique mudanças.

Saída esperada: Código final refatorado + resumo das mudanças.
```


## 🚀 **Próximos Passos**

Criei um template público com todos os recursos necessários:

**🔗 Repositório completo:** [GitHub - Copilot Refactor Prompt](https://github.com/jefersonmlopes/copilot-refactor-prompt)

**📋 Inclui:**

- ✅ Estrutura de prompt para GitHub Copilot
- ✅ Checklist de migração
- ✅ Scripts de validação
- ✅ Estratégias de teste

## 💭 **E você?**

Já enfrentou refatorações em grande escala? Quais estratégias funcionaram no seu contexto? Compartilhe sua experiência nos comentários! 👇

---

**Hashtags:** #Programação #TypeScript #VSCode #GitHubCopilot #Refatoração #DevExperience #API #React #NodeJS #Produtividade

---

**Quer o template completo?** Acesse diretamente o [repositório no GitHub](https://github.com/jefersonmlopes/copilot-refactor-prompt) com todos os arquivos de prompt e checklists! 📋
