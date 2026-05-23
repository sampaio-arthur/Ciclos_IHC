# Changelog · v1 → v2 (Ciclo 2 → Ciclo 3)

> **Projeto:** Protótipo de Onboarding do Goodreads (PT-BR)  
> **Disciplina:** Interface Humano-Computador (IHC) · UFSC Araranguá  
> **Professor:** Matheus Cataneo  
> **Autores:** Arthur Sampaio · Leonardo Boteon  
> **Base:** feedbacks recebidos na avaliação por pares do Ciclo 2

---

## Resumo

O Ciclo 2 (`v1/index.html`) resolveu os problemas P1 (tradução para PT-BR) e P2 (categorização de gêneros). A avaliação por pares apontou seis novos feedbacks, consolidados em nove lacunas (L1–L9). O Ciclo 3 (`v2/index.html`) implementa sete prioridades de correção que endereçam todas as lacunas críticas.

---

## P1 — Fluxo principal interativo

**Lacuna resolvida:** L1  
**Feedbacks:** F2, F3, F5  
**Heurísticas:** H7 · Critério 1 Feedback · Critério 5 Erros · WCAG Operável

### v1
- Botão Gmail: clicável visualmente, sem nenhum retorno ao clicar.
- Link "ou envie seu próprio convite": href="#", não faz nada.
- Link "Não vê seus gêneros favoritos aqui?": href="#", não faz nada.
- Botão "Continuar com N gêneros": desabilitado com `disabled`; quando habilitado, também não fazia nada.

### v2
| Elemento | Comportamento novo |
|---|---|
| **Logo "goodreads"** | Clicar no logo redireciona para a Etapa 1 (Tela A) de qualquer ponto do protótipo |
| **Botão Gmail** | Abre modal informando que a conexão é simulada; CTA "Continuar" avança para Tela B + toast de confirmação |
| **"ou envie seu próprio convite"** | Revela painel inline com input de e-mail, validação de formato em JS; após envio bem-sucedido, exibe toast e redireciona automaticamente para a Tela B (1.2s delay) |
| **"Não vê seus gêneros favoritos aqui?"** | Revela painel inline com input de texto livre; gêneros adicionados entram na seleção como chips removíveis; ao clicar "Cancelar", todos os gêneros customizados são removidos da contagem e o painel é limpo |
| **Botão "Continuar"** | Com seleção válida (3–10 gêneros): abre modal de confirmação listando os gêneros escolhidos antes de concluir |
| **Escape / clique fora dos modais** | Fecha todos os modais |

---

## P2 — Faixa de seleção 3–10 com prevenção ativa

**Lacuna resolvida:** L2 (P3 do Ciclo 1 permanecia em aberto)  
**Feedbacks:** F2, F3  
**Heurísticas:** H5 · Critério 5 Proteção · Critério 2 Carga

### v1
- `toggleChip()` aceitava seleção ilimitada (todos os 40 gêneros podiam ser marcados).
- Sem teto de seleção, sem instrução de faixa, sem bloqueio de chips.
- Botão habilitava com qualquer `n ≥ 1`.

### v2
- Constantes centralizadas: `MIN_SELECT = 3`, `MAX_SELECT = 10`.
- **Microcopy fixo** acima da grade antes de qualquer interação: *"Escolha entre 3 e 10 gêneros para recebermos boas recomendações."*
- **Estados do contador:**
  - `0`: neutro
  - `1–2`: neutro + *"— faltam X para o mínimo"*
  - `3–9`: verde (`.counter-ok`)
  - `10`: laranja (`.counter-warn`) + *"máximo atingido"*
- **Tentativa de selecionar o 11º:** chip alvo não é marcado, recebe animação `shake` (280ms), toast vermelho *"Limite atingido (10 gêneros). Desmarque um para escolher outro."*
- **Chips bloqueados:** ao atingir 10, chips não selecionados recebem `data-blocked="true"` → `opacity: 0.38; cursor: not-allowed`.
- Botão Continuar habilitado apenas quando `3 ≤ n ≤ 10`.

---

## P3 — Renomear "Avaliar livros" e refatorar o stepper

**Lacunas resolvidas:** L3 (rótulo inconsistente) + L4 (artefato visual)  
**Feedbacks:** F2, F3  
**Heurísticas:** H2 · H4 · H8 · Critério 6 · Critério 7

### v1
- Step 3 rotulado como *"Avaliar livros"* — inconsistente com a tela de seleção de gêneros (quebra de modelo mental).
- Stepper implementado com `transform: skewX(-22deg)` em pseudoelementos `::before`/`::after`, gerando um artefato visual (recorte diagonal sem função) entre os steps 3 e 4 com fundo escuro.

### v2
- Step 3 renomeado para **"Escolher gêneros"** — alinhado ao conteúdo da tela.
- Estrutura do stepper completamente substituída: **pílulas arredondadas independentes** (`.step-pill`) separadas por `›` tipográfico puro (`.step-sep`). Zero sobreposição, zero artefato.
- `aria-current="step"` atualizado dinamicamente na navegação entre telas.
- CSS do stepper reduzido: sem `position: relative`, sem pseudoelementos de forma, sem `skewX`.

```diff
- <div id="step-3" class="step pending">
-   <span class="step__num">3.</span> <span class="step__label">Avaliar livros</span>
- </div>
+ <div id="step-3" class="step-pill pending">
+   <span class="step-pill__num">3</span><span class="step-pill__label">. Escolher gêneros</span>
+ </div>
```

---

## P4 — Busca/filtro textual em tempo real

**Lacuna resolvida:** L7  
**Feedback:** F6  
**Heurísticas:** H7 · Critério 4 Flexibilidade · UI Pattern Busca

### v1
- Tela B não possuía nenhum mecanismo de busca.
- Usuário era forçado a varrer visualmente todos os 6 cards e 40 chips.

### v2
- **Input de busca** (`type="text"`) com ícone lupa alinhado, placeholder *"Buscar gênero (ex.: fantasia, biografia…)"*, posicionado acima da grade.
- **Filtragem em tempo real** com debounce de 150ms; busca case-insensitive por substring.
- Chips sem match recebem `opacity: 0.2; pointer-events: none` (`.search-hidden`).
- Cards sem nenhum chip visível recebem `opacity: 0.45` + badge *"0 visíveis"* (`.search-empty`) e são **colapsados automaticamente**.
- **Comportamento da busca:** ao digitar, **todas** as categorias são colapsadas; apenas as que contêm gêneros correspondentes são expandidas. Ao limpar a busca, todas expandem de volta.
- **Botão `×` único** dentro do input para limpar busca (sem duplicação com o clear nativo do browser).
- **Atalho `/`** foca o campo — funciona apenas quando a Tela B está ativa e nenhum input está focado.

---

## P5 — Diagnóstico explícito ao tentar avançar sem seleção suficiente

**Lacuna resolvida:** L5  
**Feedback:** F5  
**Heurísticas:** H9 · Critério 5 Mensagens · Critério 1 Feedback

### v1
- Botão Continuar com atributo `disabled` nativo quando `n === 0`.
- Quando desabilitado, o botão é mudo: o usuário sabe que algo está errado, mas não sabe o quê nem como resolver.

### v2
- Botão Continuar **sem `disabled` nativo** — sempre clicável (preserva foco para leitores de tela, alinhado a WCAG).
- Estado visual de desabilitado via classe `.visually-disabled` (cor cinza, sem `box-shadow`).
- Ao clicar com `n < 3`:
  - Animação `shake` no botão.
  - `<div role="alert" aria-live="assertive">` exibe mensagem no formato **O QUE + COMO**: *"Você selecionou apenas 2 gêneros. Selecione mais 1 para continuar (mínimo: 3)."*
  - Mensagem desaparece após 4s ou ao próximo clique em chip.

---

## P6 — Substituição do ícone 🐉 na categoria Ficção

**Lacuna resolvida:** L8  
**Feedback:** F2  
**Heurísticas:** H6 · Critério 7 Significado dos Códigos · WCAG Compreensível

### v1
- Categoria *Ficção* usava `🐉` (dragão) — marcador de Fantasia, não de Ficção como categoria genérica. Criava ambiguidade de código visual.

### v2
- `🐉` substituído por `🌌` (universo/imaginação) — representa abrangência sem se prender a um subgênero específico.
- Demais ícones auditados e mantidos: `📐` Formato, `📖` Não-Ficção, `🎭` Temas, `🎨` Assunto, `📚` Literatura.

```diff
- icon: '🐉',  // marcador de Fantasia — ambíguo para Ficção
+ icon: '🌌',  // universo/imaginação — representa Ficção como categoria ampla
```

---

## P7 — Revelação progressiva: colapso/expansão por categoria

**Lacuna resolvida:** L6  
**Feedback:** F1  
**Heurísticas:** H8 · H5 · Critério 2 Densidade · WCAG Operável

### v1
- Todos os 40 chips de todas as 6 categorias exibidos permanentemente.
- Sem possibilidade de recolher categorias já decididas, mantendo alta densidade informacional.

### v2
- Cabeçalho de cada card convertido em `<button>` com:
  - `aria-expanded="true"` (estado inicial expandido — preserva a affordance que já funcionava)
  - `aria-controls` apontando para o corpo colapsável
  - Ícone chevron com rotação CSS (`rotate(-90deg)`) quando colapsado
- Colapso implementado com `display: none` (classe `.collapsed`) em vez de `max-height` — resolve completamente o problema de altura das linhas no CSS Grid, onde cards vizinhos impediam o colapso visual.
- Ao colapsar, o contador exibe *"X de N selecionados"* — mantém visibilidade do status mesmo com card fechado.
- Operável por teclado (Enter/Espaço nativos em `<button>`), com foco visível customizado.

---

## P8 — Fracionamento do formulário (adiado)

**Lacuna:** L9  
**Feedback:** F4  
**Decisão:** não implementado no Ciclo 3.

**Justificativa:** as Prioridades 2 (limite 3–10), 4 (busca) e 7 (colapso opcional) já reduzem substancialmente a carga cognitiva da tela sem adicionar fricção de cliques. O input livre de gênero personalizado está disponível via link "Não vê seus gêneros favoritos aqui?" (P1), endereçando a segunda parte do feedback F4.

Parquear para possível Ciclo 4 mediante evidência empírica de teste de usabilidade.

---

## Correções pós-implementação (bugfixes)

Após a implementação inicial das Prioridades 1–7, os seguintes bugs foram identificados e corrigidos:

| # | Bug | Correção |
|:-:|---|---|
| B1 | Clicar no logo "goodreads" não fazia nada | Logo agora redireciona para Etapa 1 (Tela A) via `showScreen('a')` |
| B2 | Enviar convite por e-mail não redirecionava para a próxima etapa | Após validação bem-sucedida, aguarda 1.2s (para o usuário ler o feedback `✓`) e redireciona para Tela B + toast |
| B3 | Barra de busca exibia **dois ícones ×** (clear nativo do `type="search"` + botão `search-clear`) | Input alterado de `type="search"` para `type="text"`, eliminando o clear nativo do browser |
| B4 | Ícone lupa desalinhado verticalmente na barra de busca | Posicionamento alterado de `top: 50%; transform: translateY(-50%)` para `top: 13px` fixo |
| B5 | Categorias não colapsavam corretamente em grid 3 colunas (a linha do grid mantinha a altura do vizinho expandido) | Mecanismo de colapso trocado de `max-height` + `opacity` para **`display: none`** — remove o corpo do card do layout, fazendo a linha do grid encolher imediatamente |
| B6 | Botão × da busca desalinhado verticalmente | Posicionamento alterado de `top: 50%; transform: translateY(-50%)` para `top: 12px` fixo |
| B7 | Atalho `/` não funcionava na prática | Adicionada verificação de que a Tela B está ativa e comparação genérica com `tagName !== 'INPUT'` |
| B8 | Clicar "Cancelar" no painel de gênero personalizado não excluía os gêneros da contagem | Ao cancelar, todos os gêneros customizados são removidos do `Set selected` e `updateUI()` é chamado |
| B9 | Busca expandia cards com resultados mas não colapsava os sem resultados | Busca agora **colapsa todas** as categorias e **expande apenas** as que contêm resultados; ao limpar, todas expandem |
| B10 | Texto "Dica: pressione `/` para focar a busca rapidamente" desnecessário | Removido |

---

## Verificação dos critérios de aceitação

| # | Critério | v1 | v2 |
|:-:|---|:-:|:-:|
| 1 | Todos os controles visíveis respondem ao clique | ❌ | ✅ |
| 2 | Impossível selecionar > 10 gêneros | ❌ | ✅ |
| 3 | Impossível avançar com < 3 sem diagnóstico | ❌ | ✅ |
| 4 | Step 3 = "Escolher gêneros" | ❌ | ✅ |
| 5 | Sem artefatos visuais no stepper | ❌ | ✅ |
| 6 | Campo de busca filtra chips em tempo real | ❌ | ✅ |
| 7 | Nenhum ícone confundível com subgênero | ❌ | ✅ |
| 8 | Categorias colapsáveis/expansíveis por teclado | ❌ | ✅ |
| 9 | Caminho para adicionar gênero não listado | ❌ | ✅ |
| 10 | Mensagens no formato "O QUE + COMO" | ❌ | ✅ |
| 11 | `aria-live`, `role="alert"`, `aria-expanded` presentes | ⚠️ parcial | ✅ |

**v1: 0/11 critérios do Ciclo 3 cumpridos · v2: 11/11 ✅**

> ⚠️ O v1 tinha `aria-live="polite"` no contador, mas sem `role="alert"` e sem `aria-expanded`.
