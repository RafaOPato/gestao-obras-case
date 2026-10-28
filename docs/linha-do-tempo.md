# Linha do tempo

As versões do Canteiro, de 10/09/2026 a 28/10/2026, escritas a partir do CHANGELOG privado e **generalizadas**: sem nome de cliente, obra, fornecedor ou funcionário.

Como ler:

- O CHANGELOG tem 115 entradas e **114 versões distintas** (a 1.36.0 aparece duas vezes, em 24/09 e 25/09; não registrei o motivo). Comando em [numeros.md](numeros.md).
- Foram 48 dias entre a 1.0.0 (10/09) e a 1.61.0 (28/10), com lançamentos em 24 deles. Os dias com mais entradas foram 18/09 (13), 14/09, 17/09 e 22/09 (11 cada).
- "Só local" quer dizer que, na data em que li o CHANGELOG (28/10/2026), a versão estava pronta e validada no ambiente de desenvolvimento, mas ainda não tinha sido liberada para produção. Isso vale para a 1.53.0 e a 1.55.0.
- A numeração segue `MAJOR.MINOR.PATCH`: `MINOR` para funcionalidade nova, `PATCH` para correção.
- Muitas entradas nasceram de um pedido do dono da empresa durante o uso: 36 das 115 entradas têm "pedido do" seguido do nome dele (40 linhas no total; comandos em numeros.md). Não contei a origem das demais.

## Fase 1: no ar em produção (10 a 11/09)

| Versão | Dia | O que mudou |
|---|---|---|
| 1.0.0 | 10 a 11/09 | Primeiro lançamento: sistema em servidor próprio com HTTPS automático; conector remoto do Claude com login OAuth por pessoa; autorização por quadro unificada (o conector herda a permissão da pessoa); link temporário de upload para anexar arquivo; backup diário do banco; playbook de produção; limite de anexo de 25 para 50 MB. |
| 1.1.0 | 11/09 | Reordenar grupos arrastando; "adicionar grupo" vai para o topo. |
| 1.2.0 | 11/09 | Ferramenta para criar quadro, com cópia opcional da estrutura de outro; anotações de risco (só leitura, destrutiva) nas ferramentas do conector. |
| 1.2.1 | 11/09 | Correção feita direto em produção: o botão de nova coluna não criava coluna alguma e o menu abria em cada grupo ao mesmo tempo. A mesma correção chegou ao código local antes da subida seguinte. |

## Fase 2: o que o uso real pediu (14 a 15/09)

| Versão | Dia | O que mudou |
|---|---|---|
| 1.3.0 | 14/09 | Agenda no cabeçalho, no estilo de calendário. |
| 1.4.0 | 14/09 | Travar horário na agenda; cargo no crachá da pessoa; datas em dd/mm/aaaa; colunas de número que não são dinheiro. |
| 1.4.1 | 14/09 | Barra de cima e lateral esquerda fixas ao rolar. |
| 1.5.0 | 14/09 | **Lixeira de 30 dias e arquivo morto**; menu de botão direito em área e elemento; duplicar área. |
| 1.5.1 a 1.5.3 | 14/09 | Botão direito em quadros e pastas; correções (alinhamento do nome, duplicar quadro para quem edita só o quadro, foto de perfil, adicionar pasta para quem não é dono). |
| 1.6.0 | 14/09 | Largura da barra lateral ajustável; reordenar áreas por arraste; áreas favoritas. |
| 1.7.0 | 14/09 | Mover e duplicar grupos e elementos entre quadros e áreas. |
| 1.7.1 | 14/09 | Busca no menu de opções de status e múltipla escolha. |
| 1.7.2 | 14/09 | Agenda: "Área / obra" lista todas as áreas. |
| 1.8.0 | 15/09 | Automação de conclusão em todos os quadros (o concluído vai para um grupo próprio). |
| 1.8.1 a 1.8.5 | 15/09 | Recarregar volta ao mesmo quadro; "voltar" do navegador não sai do app; elemento novo entra em primeiro; galeria e prévia de anexos; correção de quadro com muitos anexos que travava ao carregar. |
| 1.9.0 | 15/09 | **Planilha do Excel editada dentro do sistema** (.xlsx, .xlsm; .xls abre convertido). |
| 1.10.0 | 15/09 | Pastas nos subelementos; ordenar por previsão em trimestre ou ano. |
| 1.11.0 | 15/09 | **Motor de automações** (gatilhos e ações) com tela própria e aba por quadro; assistente de automações (quem tem permissão); as mesmas ferramentas chegam ao conector do Claude. |

## Fase 3: visual novo e ferramentas de obra (16 a 18/09)

| Versão | Dia | O que mudou |
|---|---|---|
| 1.12.0 | 16/09 | Visual novo do sistema inteiro; troca da foto da tela de entrada e da capa. |
| 1.13.0 | 17/09 | Enviar muitos arquivos de uma vez, com painel de envio e anel de progresso. |
| 1.14.0 e 1.14.1 | 17/09 | Cor das etiquetas; ícone da aba do navegador. |
| 1.15.0 | 17/09 | **Assistente dentro do app que executa**, não só propõe; agentes no chat por menção; aba Painel por quadro (soma por trimestre, mês ou ano); ferramenta de soma feita no servidor. |
| 1.16.0 a 1.16.2 | 17/09 | Arrumar a barra lateral arrastando; troca do arraste nativo, que travava, por arraste com ponteiro. |
| 1.17.0 | 17/09 | Perfil das pessoas: painel com setor, cidade ou obra base, aniversário. |
| 1.18.0 | 17/09 | **Cronograma de obra dentro do sistema:** ler arquivo do MS Project, editar na tela, gerar o arquivo, "cronograma vivo" (a data de status é hoje) e bolinhas de status calculadas. |
| 1.18.1 e 1.18.2 | 17/09 | "Criar cronograma em branco" dava erro 500 em produção (restrição de banco antiga, ver ADR-007); cronograma criado no app nasce com o horário de trabalho da empresa. |
| 1.19.0 e 1.19.1 | 18/09 | **Curva S** e carimbo editável do cronograma; correção de curva e carimbo que mostravam números diferentes. |
| 1.20.0 e 1.20.1 | 18/09 | Medições dos arquivos de cronograma entram na curva S; frases legíveis no histórico. |
| 1.21.0 e 1.21.1 | 18/09 | **Fórmulas estilo Excel** nos quadros; correção do erro de uma aba aberta antes de um deploy, que pedia um pedaço do código com nome antigo (agora o app recarrega sozinho, uma vez). |
| 1.22.0 | 18/09 | Mover quadro para outra área; "obras concluídas"; "arquivo morto" passa a se chamar "Arquivados". |
| 1.23.0 e 1.23.1 | 18/09 | Cor da área ou obra, refletida na agenda. |
| 1.24.0 a 1.24.2 | 18/09 | **Word, PowerPoint e PDF** no editor, no esquema da planilha; arraste mais estável. |
| 1.25.0 | 18/09 | **Fórmula dentro de uma célula**, com referência por clique. |

## Fase 4: agentes e assistentes (21 a 25/09)

| Versão | Dia | O que mudou |
|---|---|---|
| 1.26.0 | 21/09 | Assistente "espelho" para o dono e o TI: tudo o que o Claude deles faz, o assistente do sistema faz também. |
| 1.27.0 a 1.27.2 | 22/09 | Foto baixada em JPEG; foto de iPhone (HEIC) vira AVIF como as outras; quadro lembra os grupos que você fechou; nome de anexo com acento embaralhado corrigido (código e carga); agentes travados numa conversa que não existia mais. |
| 1.28.0 | 22/09 | **Os agentes do WhatsApp passam a trabalhar no sistema próprio**, com concessões nas contas dos agentes e uma lista fixa de quadros no código (a lista passou a vir das concessões na 1.31.0); ferramentas para anexar e enviar arquivo do WhatsApp. |
| 1.29.0 | 22/09 | **Agente de compras em modo fornecedor, com ferramentas estreitas** (ver seguranca.md); mapa de concorrência gerado por script fixo. |
| 1.30.0 | 22/09 | Previsão do tempo na agenda; cidade da obra com base de cidades própria; **nova obra num passo só**; selecionar grupos em lote. |
| 1.31.0 | 22/09 | O agente de projetos enxerga obra nova sozinho. |
| 1.32.0 a 1.34.0 | 22/09 | Editar área num modal; confirmações e avisos com a cara do sistema; agenda filtra por obra. |
| 1.35.0 | 22/09 | Correções do painel financeiro: obra nova nascia zerada, seletor oferecia planilha que o painel não lia, "R$ 0" sem explicação. |
| 1.36.0 | 24 e 25/09 | **Gerente de obra** (aba da obra que escuta os grupos de WhatsApp); assistente com memória da conversa; ferramenta de editar coluna; triagem de pedido ("fazer agora" ou "registrar"). |
| 1.36.1 | 25/09 | Gerente de obra ligado de verdade; transcrição de áudio feita por um fluxo do n8n, sem chave própria no sistema. |

## Fase 5: perfis, texto e acabamento (28/09 a 01/10)

| Versão | Dia | O que mudou |
|---|---|---|
| 1.37.0 | 28/09 | Perfis de acesso com nome próprio, entre eles o "Projetista". |
| 1.38.0 e 1.38.1 | 28/09 | Nove correções do assistente espelho; anexo baixa com o nome que aparece na tela; área sem quadro vivo não some mais do assistente. |
| 1.39.0 | 29/09 | **Corretor de texto automático** (LanguageTool, no servidor, só no que vem da tela). |
| 1.40.0 e 1.41.0 | 29/09 | Cor personalizada em toda paleta; cores salvas da empresa. |
| 1.42.0 | 30/09 | **Revisor de texto com o Claude** e memória de correções da equipe (uma lista de fatos enviada a cada revisão; não é treino de modelo). |
| 1.43.0 e 1.43.1 | 30/09 | Cor padrão de grupo novo (duas cores da marca, alternando); agenda, cronograma e gerente só aparecem em área que é obra. |
| 1.44.0 | 30/09 | Convidado sugere outro dia e horário para o evento; o organizador aceita ou recusa. |
| 1.44.1 | 01/10 | Tela de entrada mais limpa. |

## Fase 6: padronização retroativa e celular (08 a 09/10)

| Versão | Dia | O que mudou |
|---|---|---|
| 1.44.2 | 08/10 | Cores dos grupos antigos padronizadas, **com auditoria retroativa**. |
| 1.44.3 | 08/10 | "Feito" é renomeado para "Concluído" no sistema inteiro, e a regra do grupo de concluídos passa a valer também para o que já estava concluído. |
| 1.44.4 | 08/10 | Etiquetas de status unificadas: cada família de estado ganha uma cor, e cada estado, um único nome. |
| 1.45.0 | 08/10 | **O servidor passa a impor o padrão das etiquetas**; escala de prioridade com cor fixa. |
| 1.46.0 e 1.46.1 | 08/10 | **Versão para celular** (tela de campo), escolhida pelo aparelho no endereço normal, sem botão de troca (validada no navegador; iPhone real não provado). |
| 1.47.0 | 08/10 | O resto do app no celular: pedidos de compra, agenda, avisos, perfil, pastas na obra, subelementos; sem barra embaixo, ações em um "+" no topo. |
| 1.47.1 | 09/10 | Varredura 360 do celular: celular real abria a tela de mesa, datas em UTC, "visita hoje" nunca acendia, cliente via ações que o servidor recusava, entre outros. |
| 1.47.2 a 1.47.4 | 09/10 | Registros de fotos organizados como os outros quadros; "voltar" em laço; área que não é obra; cor e ícone da área; ícone para o Safari. |
| 1.47.5 | 09/10 | O assistente passa a falar com o banco de imagens da empresa (produto à parte). |
| 1.48.0 e 1.48.1 | 09/10 | Todo quadro novo e vazio vem com as colunas-padrão da empresa, definidas depois de olhar os quadros que já existiam; pastas começam fechadas no celular. |

## Fase 7: aplicativos, painel do dono e diário de obra (13 a 28/10)

| Versão | Dia | O que mudou |
|---|---|---|
| 1.49.0 | 13/10 | **CAD (DWG e DXF)** aberto no CAD nativo do PC da pessoa, com volta da versão salva para a célula (salvar de volta não provado em PC real); menu **Aplicativos**; documento e cronograma em branco. |
| 1.50.0 | 20/10 | **Painel do dono montado por widgets e customizável pelo Claude**, com 11 ferramentas MCP e desfazer. |
| 1.50.1 a 1.50.6 | 20/10 | Correções; pedidos por voz entendidos pelo servidor; edição concorrente do layout; cores e acessibilidade dos gráficos; histórico de versões do painel; painel no celular e no iPhone. |
| 1.51.0 a 1.51.2 | 22 e 23/10 | Aba **Gerente de obra** com combinados, possíveis combinados e cobrança no grupo **só com aprovação**; regra "cliente não é cobrado"; correção de navegação. |
| 1.52.0 | 23/10 | Coluna "Centro de custo", com etiqueta puxada da lista do painel. |
| 1.53.0 | 23/10 | **(Só local)** Ação de automação que descobre o centro de custo a partir do nome do elemento e o preenche. |
| 1.54.0 | 26/10 | **Diário de Obra (RDO)**: manhã e tarde, mão de obra por empresa, modelo editável, clima automático e gratuito, PDF. |
| 1.54.1 | 26/10 | Liberação por horário de Brasília; temperatura automática só leitura; noite anterior por observação de aeroporto. |
| 1.55.0 | 26/10 | **(Só local)** O gatilho que cria o pedido de compra passa a ser configurável na tela; PDF do pedido feito pela IA sobre o modelo vigente. |
| 1.56.0 a 1.56.4 | 26 e 27/10 | Correções do teste de usabilidade com 5 perfis de gerente de obra; ajustes dos testes de ponta a ponta (aviso do sino, painel geral, toque duplo); câmera em rajada no diário; ligar grupo do WhatsApp a uma obra. |
| 1.57.0 | 27/10 | **Diário simples (um por dia) ou completo (manhã e tarde)**, por obra. |
| 1.58.0 | 27/10 | Aviso em cartão no canto da tela; sub-aba de combinados de ata, triangulada com o que o gerente já conhece. |
| 1.59.0 | 27/10 | Revisor de texto do Claude também no diário de obra. |
| 1.60.0 | 27/10 | A câmera passa a ficar aberta para tirar várias fotos em sequência, em toda tela do celular que fotografa na hora (iPhone real não provado). |
| 1.61.0 | 28/10 | Cobrança de compromissos de entrega: nível recalculado todo dia pela distância até o prazo, um resumo por pessoa no sino. Subiu em produção em 28/10, e a primeira regra de cobrança foi ligada no mesmo dia, como piloto em uma obra. |

## O que o conjunto mostra

- **Ritmo.** Muitas versões pequenas, cada uma com entrada no CHANGELOG, o que foi provado e o que não foi.
- **Ordem.** Primeiro o que o uso real exigiu (lixeira, mover, anexos); depois o que diferencia o sistema (cronograma, fórmulas, editor de documentos, agentes); depois a consolidação (padrões retroativos, celular); por fim os módulos de campo (CAD, painel, diário).
- **Retrabalho visível.** Há correções que aparecem logo depois de uma funcionalidade (1.18.1 depois da 1.18.0, 1.19.1 depois da 1.19.0, 1.47.1 depois da 1.47.0). É o custo de validar com o uso e de subir pequeno.
