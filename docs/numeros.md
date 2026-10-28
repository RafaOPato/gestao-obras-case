# Números

Cada número do estudo de caso, com o comando que o gerou. Tudo foi recontado em **28/10/2026**, com o Claude Code, no Windows 10 com Git Bash e Node 24.19.0, sobre o repositório privado do Canteiro.

Dois cuidados com os comandos:

- **Os nomes de pasta foram generalizados.** `backend/`, `frontend/`, `mcp/`, `agentes/`, `cronograma/` e `excel/` são as pastas de cada serviço (no repositório real elas têm outros nomes). O script de teste do modo fornecedor aparece como `teste_modo_fornecedor.mjs`, nome generalizado; o padrão `teste_*.mjs` corresponde a esse único arquivo. Do mesmo modo, o teste puro da cobrança de compromissos aparece como `compromissos`.
- **Os padrões de busca são genéricos**, sem nome de função interna: a rota é "um identificador, ponto, o verbo HTTP e um texto que começa com barra" e a ferramenta MCP é "uma chamada que abre numa linha sozinha".

Só entram aqui números que saem de um comando sobre o código ou o CHANGELOG privados; contagens de dados reais da empresa não entram, de propósito. Os comandos rodam a partir da **raiz desse repositório** (`<raiz>`); como o código não é público, quem não tiver acesso não consegue repeti-los. Por isso a saída de cada um está escrita aqui.

> Código-fonte não é público (propriedade da empresa); demo com dados fictícios sob pedido.

Dois avisos:

- **O repositório privado continua mudando** (há mais de uma sessão de trabalho editando). Os totais abaixo são uma fotografia do momento da medição; o conjunto inteiro foi rodado de uma vez só para os números não ficarem misturados.
- **Contagem não é qualidade.** Linhas de código incluem comentários e linhas em branco; "rotas" são declarações no código, não endereços únicos.

## 1. Versões (CHANGELOG)

| Métrica | Valor | Comando em `<raiz>` | Nota |
|---|---|---|---|
| Entradas de versão | **115** | `grep -c '^## \[' CHANGELOG.md` | |
| Versões distintas | **114** | `grep -oE '^## \[[0-9]+\.[0-9]+\.[0-9]+\]' CHANGELOG.md \| sort -u \| wc -l` | A 1.36.0 tem duas entradas (24/09 e 25/09); não registrei o motivo. |
| Última e primeira entrada | 1.61.0 em 28/10/2026; 1.0.0 em 10/09/2026 | `grep -oE '^## \[[0-9.]+\] — [0-9-]{10}' CHANGELOG.md \| sed -n '1p;$p'` | O CHANGELOG tem a mais nova no topo, por isso a saída traz a 1.61.0 primeiro. 48 dias entre as duas datas. |
| Dias com pelo menos uma entrada | **24** | `grep -oE '^## \[[0-9.]+\] — [0-9-]{10}' CHANGELOG.md \| awk '{print $NF}' \| sort -u \| wc -l` | |
| Dia com mais entradas | 13 (em 18/09/2026) | `grep -oE '^## \[[0-9.]+\] — [0-9-]{10}' CHANGELOG.md \| awk '{print $NF}' \| sort \| uniq -c \| sort -rn \| head -5` | Seguem 22/09, 17/09 e 14/09, com 11 cada. |

## 2. API e MCP

| Métrica | Valor | Comando em `<raiz>` | Nota |
|---|---|---|---|
| Declarações de rota HTTP no backend | **292** | `grep -hoE "\b[A-Za-z_]+\.(get\|post\|put\|patch\|delete)\(['\"]/" backend/src/*.js \| wc -l` | Declarações no código, não endereços únicos. |
| Por verbo | GET 112, POST 110, PUT 22, PATCH 17, DELETE 31 | o mesmo comando com um verbo só, por exemplo `grep -hoE "\b[A-Za-z_]+\.get\(['\"]/" backend/src/*.js \| wc -l` | 112 + 110 + 22 + 17 + 31 = 292 |
| Ferramentas MCP gerais | **58** | `grep -cE "^[[:space:]]*[a-z]+\($" mcp/tools.js` | Cada ferramenta é registrada por uma chamada que abre numa linha sozinha. |
| Ferramentas MCP de automação | **7** | `grep -cE "^[[:space:]]*[a-z]+\($" mcp/automacoes.js` | Só aparecem para quem tem a permissão de automatizar. |
| Total de ferramentas MCP | **65** | 58 + 7 | |

## 3. Banco de dados

| Métrica | Valor | Comando em `<raiz>` | Nota |
|---|---|---|---|
| Tabelas no schema | **71** | `grep -ciE '^[[:space:]]*CREATE TABLE IF NOT EXISTS' backend/src/schema.sql` | Tabelas criadas pelo schema do sistema; o banco das cotações dos agentes é outro. |
| Linhas de índice | **68** | `grep -ciE '^[[:space:]]*CREATE (UNIQUE )?INDEX' backend/src/schema.sql` | |
| Linhas do `schema.sql` | **1.702** | `wc -l < backend/src/schema.sql` | |

## 4. Tamanho do código

| Área | Linhas | Arquivos | Comando em `<raiz>` |
|---|---|---|---|
| Backend, JavaScript (sem testes) | **23.508** | 51 | `ls backend/src/*.js \| grep -v '\.test\.js' \| xargs cat \| wc -l` (arquivos: `ls backend/src/*.js \| grep -v '\.test\.js' \| wc -l`) |
| Backend, testes em JavaScript | 1.320 | 4 | `cat backend/src/*.test.js \| wc -l` |
| Backend, testes de integração em Python | 533 | 7 | `cat backend/test_*.py \| wc -l` |
| Frontend, componentes (`.jsx`) | **28.830** | **56** (50 de mesa e 6 do celular) | `find frontend/src -name '*.jsx' \| xargs cat \| wc -l` (celular: `find frontend/src/mobile -name '*.jsx' \| wc -l`) |
| Frontend, módulos `.js` | 658 | 8 | `find frontend/src -name '*.js' \| xargs cat \| wc -l` |
| Frontend, CSS | 9.595 | 32 | `find frontend/src -name '*.css' \| xargs cat \| wc -l` |
| Pasta do celular (jsx, js e css) | 5.127 | n/d | `find frontend/src/mobile -type f \( -name '*.jsx' -o -name '*.js' -o -name '*.css' \) \| xargs cat \| wc -l` |
| Servidor MCP | 2.120 | 4 | `cat mcp/*.js \| wc -l` |
| Serviço de agentes | 1.881 | n/d | `cat agentes/*.js agentes/*.mjs \| wc -l` (inclui o script de teste do modo fornecedor, `teste_modo_fornecedor.mjs`, nome generalizado, com 45 linhas; sem ele são 1.836: `cat agentes/*.js \| wc -l`) |
| Serviço de cronograma (Python) | 335 | 1 | `cat cronograma/*.py \| wc -l` |
| Serviço de Excel (Python) | 414 | 1 | `cat excel/*.py \| wc -l` |

Não entram: `node_modules`, arquivos de dados, workflows do n8n e os documentos do projeto.

## 5. Infraestrutura

| Métrica | Valor | Comando em `<raiz>` | Nota |
|---|---|---|---|
| Serviços no Compose | **16** | `awk '/^services:/{f=1;next} /^[a-z]/{f=0} f && /^  [a-zA-Z0-9_-]+:[[:space:]]*$/' docker-compose.yml \| wc -l` | Alguns só sobem com perfil (proxy reverso, front de produção, túnel de demonstração). |
| Serviços no arquivo de produção | 2 | o mesmo comando em `docker-compose.prod.yml` | É uma sobreposição; só altera dois serviços. |
| Workflows do n8n exportados | 14 | `ls workflows/*.json \| wc -l` | Os arquivos não estão neste repositório. |

## 6. Testes

| Métrica | Valor | Como | Nota |
|---|---|---|---|
| Arquivos de teste puros em JavaScript | 4 (`compromissos`, `centro_custo_auto`, `rdo`, `rdo_clima`) | `ls backend/src/*.test.js \| wc -l` | |
| Esses testes **foram rodados (com o Claude Code) e passaram** | 4 de 4 | Os arquivos `backend/src/*.js` foram copiados para uma pasta temporária **fora** do repositório, onde se fez `npm init -y` e `npm install express pg` e se rodou `node <nome>.test.js` (de dentro da pasta `src/` copiada) para cada um. Cada um terminou com "ok" e saída 0. Repetido em 28/10/2026. | Sem banco e sem rede de verdade; o teste do clima registra falhas simuladas de rede, de propósito. |
| Linhas com `assert` nos 4 arquivos | 554 | `grep -c assert backend/src/*.test.js` (9 + 306 + 203 + 36) | Conta linhas, não casos. |
| Scripts de integração em Python | 7 | `ls backend/test_*.py \| wc -l` | **Não executados:** exigem o sistema completo no ar. |
| Script do modo fornecedor (JavaScript) | 1 (`teste_modo_fornecedor.mjs`, nome generalizado; 45 linhas) | `ls agentes/teste_*.mjs \| wc -l` | **Não executado:** usa um WAHA falso, mas roda dentro do contêiner do serviço de agentes e chama o modelo. |

## 7. Processo

| Métrica | Valor | Comando em `<raiz>` | Nota |
|---|---|---|---|
| Linhas do CHANGELOG que citam a auditoria retroativa | **41** | `grep -ciE "auditoria retroativa" CHANGELOG.md` | Conta linhas. |
| Entradas do CHANGELOG que citam a auditoria retroativa | **40** | `awk '/^## \[/{e=$0;next} tolower($0) ~ /auditoria retroativa/{s[e]=1} END{for(k in s)n++;print n}' CHANGELOG.md` | Conta entradas distintas, de um total de 115. |
| Linhas que dizem "nada a ajustar" | **31** | `grep -ciE "nada a ajustar" CHANGELOG.md` | |
| Linhas que mencionam o "pode subir" | 12 | `grep -ciE "pode subir" CHANGELOG.md` | Em 9 entradas distintas (mesmo `awk` da linha de entradas, trocando o padrão por `/pode subir/`). Em 6 das 12 linhas a aprovação já foi dada; as outras citam a regra ou uma versão que ainda espera. |
| Linhas com "pedido do" seguido de nome | 40 | `grep -cE "pedido d[oa] [A-Z][a-z]+" CHANGELOG.md` | Todas com o mesmo nome (o dono da empresa): `grep -oE "pedido d[oa] [A-Z][a-z]+" CHANGELOG.md \| sort \| uniq -c \| wc -l` devolve 1. |
| Entradas com "pedido do" seguido de nome | **36** | `awk '/^## \[/{e=$0;next} /pedido d[oa] [A-Z][a-z]+/{s[e]=1} END{for(k in s)n++;print n}' CHANGELOG.md` | Conta entradas distintas, de um total de 115. |
| Arquivos na pasta de memória do projeto (notas entre sessões) | 80 (79 notas e 1 arquivo de índice) | `ls <pasta de memória do projeto> \| wc -l` (saída 80); sem o índice: `ls <pasta de memória do projeto> \| grep -vc '^MEMORY.md$'` (saída 79) | A pasta fica fora do repositório do sistema, na configuração do Claude Code, e é a memória do ambiente de trabalho inteiro: mistura notas de outros assuntos, então só parte delas trata do Canteiro. |
| Arquivos de backfill (scripts e fotos do antes) | 13 | `ls docs/backfills \| wc -l` | |
| Commits no git | **15**, de 25/09 a 09/10/2026 | `git rev-list --count HEAD` e `git log --reverse --format=%ad --date=short \| sed -n '1p;$p'` | O git só entrou no projeto em 25/09, e a maior parte do trabalho recente não foi commitada. O registro confiável é o CHANGELOG. |

## 8. Números de negócio (em branco de propósito)

Não tenho medidos, e não vou estimar: usuários ativos, horas economizadas, tempo de fechamento de pedido, pedidos por mês, custo anual do serviço antigo contra a hospedagem atual, % da operação que entra pelo Claude e agentes e injeções barradas em produção.

## 9. Validação dos diagramas

Os 7 blocos Mermaid do README e de `docs/arquitetura.md` (4 fluxogramas, 1 diagrama entidade-relacionamento e 2 de sequência) foram validados com `mermaid.parse()`, versão **11.4.1**, rodando em Node com `jsdom@25`, numa pasta temporária fora do repositório: 7 blocos, 0 erros. Para ter certeza de que a checagem pega erro de verdade, passei um bloco quebrado de propósito, e ele falhou com erro de parsing. Isso valida a **sintaxe**, não o visual: a renderização final depende do GitHub.

Para repetir, numa pasta temporária fora do repositório: `npm i mermaid@11.4.1 jsdom@25` e um `check.mjs` com o código abaixo (o Mermaid precisa de um DOM, por isso o `jsdom` e as globais copiadas dele; sem elas o `mermaid.parse` falha com "Element is not defined").

```js
// check.mjs: valida a sintaxe dos blocos Mermaid de arquivos .md.
// Uso: node check.mjs <caminho>/README.md <caminho>/docs/arquitetura.md
import { JSDOM } from 'jsdom';
import { readFileSync } from 'node:fs';

const dom = new JSDOM('<!doctype html><html><body></body></html>');
for (const k of ['window', 'document', 'Element', 'HTMLElement', 'SVGElement', 'Node',
  'DocumentFragment', 'NodeFilter', 'DOMParser', 'MutationObserver']) {
  Object.defineProperty(globalThis, k, { value: dom.window[k], configurable: true, writable: true });
}
const { default: mermaid } = await import('mermaid');

let blocos = 0, erros = 0;
for (const arquivo of process.argv.slice(2)) {
  const md = readFileSync(arquivo, 'utf8');
  for (const m of md.matchAll(/^```mermaid\r?\n([\s\S]*?)^```/gm)) {
    blocos++;
    try { await mermaid.parse(m[1]); }
    catch (e) { erros++; console.log('ERRO em', arquivo, '-', String(e.message).split('\n')[0]); }
  }
}
console.log(`${blocos} blocos, ${erros} erros`);
process.exit(erros ? 1 : 0);
```

Comando, a partir da pasta temporária: `node check.mjs <caminho-do-repositório>/README.md <caminho-do-repositório>/docs/arquitetura.md` (a saída esperada é `7 blocos, 0 erros`; o README tem 1 bloco e a arquitetura, 6). Esta versão do script foi repetida em 28/10/2026, depois das últimas edições dos documentos: 7 blocos, 0 erros; e um arquivo com um bloco quebrado de propósito termina com `1 blocos, 1 erros` e saída 1.

Os outros arquivos (`decisoes.md`, `seguranca.md`, `linha-do-tempo.md`, este) não têm diagramas.
