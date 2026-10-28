# Decisões de projeto

Formato curto de ADR (registro de decisão): contexto, decisão, consequências. Cada uma veio de um problema real do sistema em uso, e as datas são as do CHANGELOG privado. Onde há números, eles saem do código ou do CHANGELOG privado, com o comando em [numeros.md](numeros.md); contagens de dados reais da empresa ficam de fora de propósito.

| # | Decisão | Quando |
|---|---|---|
| 1 | [Lixeira de 30 dias em vez de apagar](#adr-001-lixeira-de-30-dias-em-vez-de-apagar) | 14/09/2026 |
| 2 | [Criptografia de campo, só onde nenhuma consulta precisa do texto puro](#adr-002-criptografia-de-campo-só-onde-nenhuma-consulta-precisa-do-texto-puro) | 09/09/2026 |
| 3 | [Perfis de acesso por quadro, checados no servidor](#adr-003-perfis-de-acesso-por-quadro-checados-no-servidor) | 02/09/2026 |
| 4 | [MCP primeiro: o Claude é a porta de entrada](#adr-004-mcp-primeiro-o-claude-é-a-porta-de-entrada) | 02/09 a 10/09/2026 |
| 5 | [Migração sem parar, com contagem antes e depois](#adr-005-migração-sem-parar-com-contagem-antes-e-depois) | 02/09 a 11/09/2026 |
| 6 | [Auditoria retroativa a cada atualização](#adr-006-auditoria-retroativa-a-cada-atualização) | 06/10/2026 |
| 7 | [Deploy só depois de validar no local](#adr-007-deploy-só-depois-de-validar-no-local) | 11/09/2026 |
| 8 | [Construir o essencial em vez de adotar uma ferramenta pronta](#adr-008-construir-o-essencial-em-vez-de-adotar-uma-ferramenta-pronta) | 02/09/2026 |
| 9 | [A regra mora no servidor, não só na tela](#adr-009-a-regra-mora-no-servidor-não-só-na-tela) | contínua |

---

## ADR-001: Lixeira de 30 dias em vez de apagar

**Contexto.** Quem edita um quadro apaga coisa sem querer. O sistema guarda de tudo: pedidos de compra, histórico, anexos. Apagar de verdade era irreversível, e a estrutura tem muita dependência (excluir um quadro leva grupos, colunas, elementos, concessões e histórico).

**Alternativa descartada.** Uma coluna "excluído" em cada tabela. O código da lixeira estima umas 90 consultas que leem quadros, grupos, colunas e elementos (tela, busca, painel e ferramentas do Claude entre elas). Cada uma precisaria do filtro, e toda consulta escrita depois também; um único esquecimento e um item excluído apareceria no painel financeiro.

**Decisão.** Antes de apagar, o sistema monta uma **foto em JSON** das linhas que saem juntas e a guarda numa tabela `lixeira`; só então remove das tabelas de trabalho, como sempre fez. Restaurar é o caminho inverso: as linhas voltam com os ids de antes, e quem apontava para elas volta a apontar (histórico de atividade, pedidos de compra, eventos da agenda). A foto expira em 30 dias, por uma limpeza automática horária. Antes disso, "Apagar de vez" é permitido apenas ao dono da empresa e ao TI. O arquivo morto é um mecanismo à parte: apenas uma marca que deixa o quadro em modo leitura, para obra encerrada.

**Consequências.**

- Quem consulta quadros não precisa saber da lixeira, e nenhuma consulta futura tem de lembrar de filtrar excluídos.
- Custo: o módulo da lixeira precisa conhecer o mapa de dependências entre as tabelas. **Regra de manutenção:** quem criar uma tabela que aponte para área, pasta, quadro, grupo, coluna ou elemento deve incluí-la no mapa; esquecer significa restaurar um quadro faltando pedaços. Já valeu para as fórmulas de célula, as automações e o estado da cobrança de prazos.
- Custo: por até 30 dias o mesmo dado existe duas vezes.
- Um detalhe que ensinou: a restauração só religa as referências declaradas com `ON DELETE SET NULL`. Tabelas que apontam para o quadro por um id solto, sem FK declarada (foi escolha deliberada), não entram na foto; elas ficam intactas enquanto o quadro está na lixeira, e o quadro restaurado reencontra as regras dele no mesmo lugar.

---

## ADR-002: Criptografia de campo, só onde nenhuma consulta precisa do texto puro

**Contexto.** O sistema guarda e-mail e telefone de pessoas, e CNPJ, telefone, e-mail e contato de fornecedores nos pedidos de compra. Quem tiver acesso ao dump do banco não deveria ler isso.

**Decisão.** AES-256-GCM, com prefixo de versão no valor gravado e chave de 32 bytes guardada fora do banco, para os campos estruturados acima. Como o GCM usa vetor de inicialização aleatório, o mesmo e-mail nunca cifra igual duas vezes; então o login, que busca por igualdade, usa um **índice cego** (HMAC do e-mail) com índice único, que também impede e-mail duplicado.

**O que ficou de fora, de propósito.**

- **Alguns campos ficaram sem criptografia de campo por razões funcionais.** Em cada caso, uma consulta, um índice ou uma função de busca do sistema depende do texto puro, e cifrar quebraria essa função. Os campos e os mecanismos não são listados aqui.

A pergunta que virou checklist: *antes de cifrar um campo novo, alguma consulta SQL depende dele em texto puro?* Foi assim que um desses casos foi descoberto a tempo.

**Consequências.**

- Perder a chave é perder esses campos, e o backup do banco sozinho não decifra.
- A migração do dado que já existia é manual, idempotente e tem modo "só contar". Numa instalação nova a ordem importa: sem a migração, o login de quem já existia deixa de achar a conta.
- A API usa uma conta de banco de privilégio mínimo, decidida nesse mesmo pacote.
- Estender a cifra a mais campos seria um projeto à parte, e não uma extensão desta decisão.

---

## ADR-003: Perfis de acesso por quadro, checados no servidor

**Contexto.** O dono da empresa queria quatro papéis e uma exigência que a ferramenta antiga não cumpria: o cliente da obra vê o andamento, mas nunca o quadro de custo e margem da mesma obra.

**Decisão.**

- **Quatro papéis-base:** dono da empresa (todo dado e estrutura; cadastra pessoas e cria e apaga áreas), TI (infra e funcionalidades, mas **sem** acesso automático a dado), funcionário (só o que for concedido; quem edita um quadro edita também a estrutura dele) e dono da obra/cliente (só leitura). Mais uma **conta de serviço** para integrações e uma por agente de IA. Desde a 1.37.0 (28/09/2026) existem também **perfis com nome próprio** (por exemplo, "Projetista"), cada um criado sobre uma das quatro bases; a base é o que decide o que a pessoa alcança.
- **Poder de sistema não é acesso a dado.** Só o dono da empresa chega a todo o dado apenas pelo papel. O TI tem poderes de infraestrutura, mas recebe quadro a quadro, como qualquer um.
- **Para o cliente a concessão é sempre de quadro, nunca de área.** O servidor rejeita conceder uma área a um cliente: toda obra tem um quadro de custo interno, e abrir a obra inteira vazaria esse dado sem que ninguém percebesse.
- **A busca global intersecta com o escopo da pessoa** em vez de ser bloqueada. É a única rota que atravessa quadros e não tem um `board_id` para conferir; a proteção é filtrar.
- **Senha temporária obriga troca, e a checagem vale também ao retomar a sessão:** a API restringe a sessão enquanto houver senha temporária.
- **Nome não é permissão.** O perfil dá o rótulo, e o que a pessoa pode fazer continua vindo da base. Por isso as checagens de acesso ficaram como estavam quando os perfis com nome chegaram, e nenhum perfil novo cria um poder que o servidor não saiba barrar. O "cargo" em texto livre no crachá, que nunca deu permissão, saiu na 1.37.0: o cargo que cada pessoa já tinha foi transformado num perfil real, sem tirar nem dar poder a ninguém.
- **Uma exceção deliberada:** o cronograma de toda obra pode ser aberto e editado por qualquer funcionário, mesmo sem acesso ao quadro. O cliente só consulta, e apenas se tiver o quadro.

**Consequências.**

- Mais concessões para administrar do que um modelo por papel puro. Em troca, a regra "cliente não vê custo" não depende de alguém lembrar.
- Cada agente de IA ganhou conta própria com concessões reais (antes todos usavam a mesma conta de automação, separados só por código). O token de uma pessoa devolve só os quadros dela.
- Tokens de API pessoais e o painel de TI para gerar e revogar.
- Os testes usaram um usuário de cada papel, com os casos de recusa incluídos.

---

## ADR-004: MCP primeiro: o Claude é a porta de entrada

**Contexto.** O dono informou que a maior parte da operação entra pedindo ao Claude, pelo claude.ai ou pelo celular, e não pela tela (estimativa dele; não medi). A tela virou o lugar onde o dado mora, onde se confere e onde o cliente olha em leitura.

**Decisão.** Um servidor MCP remoto (Streamable HTTP) com OAuth 2.1, registro dinâmico de cliente e login com a **mesma conta do sistema**. O token de acesso é a própria sessão da pessoa, então o MCP não tem permissão própria: cada chamada leva o token de quem perguntou e a API filtra pelas concessões dele. O mesmo servidor atende os agentes do WhatsApp, cada um com o token da própria conta.

**Detalhes que importaram.**

- **Sem lista de permissões duplicada.** O protótipo inicial tinha uma lista por agente no servidor MCP. Duas fontes da mesma regra saem de sincronia. Foi apagada.
- **Toda resposta de ferramenta vai marcada como dado**, entre delimitadores aleatórios por chamada, porque o que volta foi digitado por gente. Já havia acontecido, sem ataque nenhum: o título de um item ("TESTE", "NÃO USAR") foi obedecido como instrução por um agente.
- **Ferramentas validam antes de escrever.** No primeiro teste de ponta a ponta, criar um item gravava o item e só depois os valores; um nome de coluna errado deixava o item órfão, e o modelo, ao tentar de novo, duplicava. Agora confere coluna e opção válida contra o schema antes de criar, e recusa com a lista do que aceita.
- **Toda chamada que mira um item leva o quadro junto**, e o servidor rejeita a combinação que não corresponde.
- **Transporte sem estado.** Com sessão em memória, depois de qualquer deploy as conversas abertas mandavam um identificador antigo, tomavam 404 e as ferramentas pareciam ter sumido.
- **Anotações de risco** (só leitura, escreve, destrutiva) em cada ferramenta, para a tela de permissão do cliente.
- **Configurar o painel do dono pelo Claude vale direto em produção** (é dado de configuração, com desfazer), mas o código das ferramentas segue a regra do ADR-007.

**Consequências.**

- O sistema depende do cliente do Claude estar disponível e se comportando; a tela continua sendo o plano B completo.
- A superfície de injeção de prompt aumenta: ver [seguranca.md](seguranca.md).
- 65 ferramentas é muita descrição para o modelo ler. Mantê-las consistentes dá trabalho (cada nova coluna, tipo ou regra precisa chegar ao MCP, senão o assistente recusa com erro de enumeração).

---

## ADR-005: Migração sem parar, com contagem antes e depois

**Contexto.** Sair do serviço antigo sem perder itens, valores de coluna, anexos nem a operação do dia. O serviço antigo ficou ligado durante a migração, e a equipe só trocou depois de validar.

**Decisão.**

- **Exportar e escrever são scripts separados.** Puxar do sistema antigo é lento e limitado por taxa de uso; o resultado fica em cache, e a escrita lê desse cache quantas vezes for preciso.
- **Tudo idempotente e retomável.** A escrita casa área, pasta e quadro por nome e **pula o que já existe**. Foi assim que os primeiros quadros, que já tinham anexo, ficaram protegidos de uma segunda carga.
- **Um script de "deriva" só acrescenta.** O que apareceu no serviço antigo depois da carga entra; o que só existe no novo é **reportado e nunca apagado**.
- **Reconciliação por contagem, quadro a quadro.**
- **Achar o furo antes de rodar em lote.** Antes da carga de anexos, um arquivo sem URL pública dentro de uma célula com vários arquivos era descartado em silêncio e a célula marcada como migrada. Corrigido para abortar a célula inteira e preservar o link original.
- **O sistema novo é dono do dado.** Regra do dono: nada de buscar no serviço antigo em tempo de uso. Por isso os anexos foram todos baixados.

**Verificações feitas.** O resultado vem do diário do projeto; não o recontei, porque o serviço antigo não existe mais. As contagens de dados da empresa foram omitidas de propósito.

| Verificação | Antes | Depois |
|---|---|---|
| Quadros | todos os quadros do serviço antigo | todos no novo; nenhum faltando, nenhum a mais |
| Elementos e subelementos | contagem por quadro no serviço antigo | a mesma contagem por quadro, mais os valores de coluna |
| Quadros com contagem diferente | n/d | só com elementos a mais no novo (itens de teste e cópias); nada foi apagado, ficou como pendência para o dono |
| Cores de status | cores de status e de grupo do serviço antigo | idênticas, sem diferença e sem conflito na tabela de índices |
| Auditoria de tipos no banco inteiro | n/d | nenhum status fora das opções, nenhuma múltipla escolha que não virou lista, nenhum período malformado, nenhum número que não virou número, nenhum quadro sem coluna ou grupo, nenhum elemento sem grupo |
| Células de anexo que dependiam do serviço antigo | todas | todas migradas, menos poucas não migráveis, conferidas à mão (documentos nativos do serviço antigo, que não são arquivos, e um arquivo já apagado lá) |
| Arquivos baixados | n/d | todos, com amostragem de tamanho e assinatura |

**Consequências.**

- Um bug clássico de migração desta natureza apareceu e foi contornado: valores de lista de opções vinham separados por vírgula e quebravam quando a própria opção tinha vírgula no nome. A exportação passou a usar o campo já separado, e o teste foi um quadro real com uma opção desse tipo.
- Os comentários e o log de atividade do serviço antigo não foram migrados (os scripts de exportação e carga tratam de quadros, grupos, colunas, elementos, valores e arquivos); o histórico do sistema novo começa na troca.
- Ficou de fora de propósito: os quadros auxiliares de subelemento (os subelementos entram aninhados no pai), os objetos personalizados do serviço antigo e um documento.
- Medido depois da carga com o dado real: no maior quadro a API responde rápido, mas o front renderiza tudo e demora para refazer a lista a cada tecla da busca. Usável, e é o argumento para virtualizar a lista (ainda não feito).

---

## ADR-006: Auditoria retroativa a cada atualização

**Contexto.** Cada nova regra (cor padrão, nome de status, coluna, permissão, ferramenta do assistente) deixava o dado novo certo e o dado antigo como estava. Em um sistema que o dono vai mudando por pedido, isso vira uma colcha de retalhos em semanas.

**Decisão.** Regra fixa do projeto, desde 06/10/2026: uma atualização só está pronta quando também vale para os dados anteriores.

1. **Mapear:** olhar o sistema inteiro, e não só o quadro que está aberto, para localizar onde a regra nova se aplica.
2. **Auditar:** uma consulta apenas de leitura que conta quantos registros antigos não cumprem a regra.
3. **Ajustar:** migração que pode rodar duas vezes sem efeito colateral (idempotente); toda remoção passa pela lixeira; a foto do estado anterior fica guardada para desfazer.
4. **Provar:** contagem antes e depois no ambiente local; em produção, só com o "pode subir".
5. **Registrar** no CHANGELOG o resultado da auditoria: quantos registros mudaram ou, se o dado antigo já estava certo, uma linha dizendo "nada a ajustar".

**Exemplos reais (do CHANGELOG privado: os três primeiros de 08/10/2026, o último de 27/10/2026; contagens omitidas de propósito).**

| Pedido | Antes | Depois |
|---|---|---|
| Cor padrão de grupo (alternando as duas cores da marca) | grupos antigos fora do padrão, em muitas cores diferentes | todos os grupos na regra |
| "Concluído" no lugar de "Feito" e grupo de concluídos aplicado ao que já estava concluído | colunas e valores com a etiqueta antiga; elementos concluídos fora do grupo de concluídos | nenhuma etiqueta antiga; elementos movidos e grupos criados onde faltavam |
| Uma cor por família de status e um nome só por estado | colunas de status com nomes e cores variados para o mesmo estado | toda etiqueta na cor da sua família; uma segunda execução não altera nada |
| Revisor de texto no diário de obra | textos revisáveis em períodos abertos e em períodos já fechados | os dos períodos abertos entraram na fila (o Claude corrigiu parte; o resto já estava certo); os dos períodos fechados foram deixados como estavam de propósito (documento fechado, com PDF emitido); segunda rodada sem nada a ajustar |

**Consequências.**

- Toda funcionalidade custa mais tempo. A contrapartida é que ninguém precisa perguntar "e o que já existia?".
- "Nada a ajustar" também precisa ser provado: 41 linhas do CHANGELOG citam a auditoria, 31 dizem "nada a ajustar". Há 13 arquivos de backfill e fotos do estado anterior.
- A auditoria sozinha não impede que o dado novo se desvie de novo. Por isso, depois dos casos de etiqueta, o **servidor passou a impor a regra** em todo caminho que grava (ver ADR-009).

---

## ADR-007: Deploy só depois de validar no local

**Contexto.** Era um sistema em uso, com dinheiro e fornecedor dentro. Uma subida errada afeta a operação.

**Decisão.** Sequência fixa para toda mudança de código:

1. Alterar o arquivo apenas na máquina de desenvolvimento.
2. Validar de verdade: reconstruir o serviço afetado, usar no navegador, ler o log.
3. Apresentar a mudança e o que foi validado, e **aguardar o "pode subir" explícito do dono**, inclusive no conserto de um bug que ele mesmo apontou.
4. Dump do banco e um arquivo de rollback antes; copiar **só os arquivos do pedido**; reconstruir só o serviço afetado.
5. Conferir (hash local contra servidor, status das páginas, log sem erro) e atualizar `VERSION` e CHANGELOG.

**Exceções, escritas de antemão:** apagar incêndio que já está quebrado em produção (diagnostica, corrige e conta depois), e **configuração** do painel do dono pelo Claude, que é dado com desfazer. O código das ferramentas continua exigindo o "pode subir".

**Consequências e lições vividas.**

- Mais lento, e depende de uma aprovação humana. Para um sistema financeiro, o trade-off foi aceito.
- Várias sessões do Claude Code podem mexer no mesmo diretório local. Uma vez, uma sessão subiu o arquivo de versão errado porque outra tinha incrementado a numeração. Foi corrigido na hora; a regra virou: conferir a versão logo antes de copiar, e copiar só a entrada do CHANGELOG da própria versão. Por isso o deploy é por lista de arquivos e compara hash local e remoto antes de assumir o ponto de partida.
- `CREATE TABLE IF NOT EXISTS` **não atualiza** uma constraint de tabela que já existe. Em produção, uma lista de valores aceitos ficou presa nos quatro primeiros e derrubou uma tela com erro 500, enquanto o banco de desenvolvimento, criado depois, funcionava. Regra: toda mudança de constraint tem um `ALTER TABLE` explícito no fim do schema, e antes de subir comparam-se as colunas e constraints de produção e local.
- Cada deploy tem número de versão, entrada no CHANGELOG com o que foi provado e o que **não** foi provado (por exemplo, "não provado: iPhone real").

---

## ADR-008: Construir o essencial em vez de adotar uma ferramenta pronta

**Contexto.** O motivo da troca era o preço por usuário do serviço antigo, que escalava mal. Antes de construir, o projeto testou uma ferramenta de banco de dados com interface de planilha, com uma migração de teste de uma amostra de quadros reais.

**Decisão.** Rejeitada. Tecnicamente funcionou, mas perdia a hierarquia **grupo colorido, elemento, subelemento** que a equipe usa todo dia, e "parecia um banco de dados" para quem não é técnico, com risco de alguém mexer na estrutura sem querer. Construiu-se só o essencial mapeado nos quadros reais, com o motor de automação ficando de fora (o n8n já fazia; a decisão valeu para o primeiro corte, e a 1.11.0, cinco dias depois do lançamento, trouxe um motor próprio de automações, com tela e assistente; ver [linha-do-tempo.md](linha-do-tempo.md)) e sem aplicativo nativo para celular (a tela de campo veio depois, como web).

**Consequências.** Mais trabalho e mais manutenção para a equipe. Em troca, o modelo de permissão (ADR-003) e o conector do Claude (ADR-004) existem exatamente como eram necessários.

---

## ADR-009: A regra mora no servidor, não só na tela

**Contexto.** A mesma informação é gravada por vários caminhos: a tela, o conector do Claude, os agentes, o n8n, scripts.

**Decisão.** Toda regra de negócio e de permissão é imposta no servidor.

- **Permissão:** a tela só esconde botão; quem barra é o servidor (tentar por URL leva 403).
- **Padrão de etiquetas de status:** nomes e cores por estado são aplicados em todo caminho que grava opções, inclusive no valor gravado na célula (quem manda "Done" acaba com "Concluído"). Duas variantes enviadas juntas viram uma só.
- **Fórmulas:** calculadas no servidor, na leitura, para valer igual na tela, no MCP e no relatório.
- **Cor padrão de grupo novo:** decidida pelo servidor; nenhuma ferramenta precisa mandar cor.
- **Valores válidos de coluna** recusados antes de gravar.

**Consequências.** Há um lugar só para mudar cada regra, e nenhum caminho consegue contorná-la. O custo é que uma regra nova precisa de teste por todos os caminhos (por isso a auditoria retroativa, ADR-006, e a lista de ferramentas MCP que precisa acompanhar).
