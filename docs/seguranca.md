# Segurança

O que foi feito para proteger o Canteiro, um sistema que guarda contrato, nota fiscal, comprovante de pagamento, dados de fornecedor e finanças da empresa. Este documento descreve decisões e resultados de teste. **Não traz configuração real:** nenhum domínio, IP, porta, chave, nome de variável de ambiente ou id do ambiente de produção.

A origem de cada afirmação é o plano de segurança, o CHANGELOG e as notas de trabalho do projeto privado. Onde algo foi testado, está dito como.

## 1. Como o trabalho foi feito

1. **Revisão testada, não deduzida.** Cada achado tinha um comando que o comprovava. O que não foi reproduzido não entrou na lista.
2. **Remover antes de acrescentar.** Quando o achado era algo desnecessário, a correção foi apagar, e não somar uma camada.
3. **Registrar as exceções decididas de propósito e o motivo**, como o que não é cifrado ([ADR-002](decisoes.md)).

## 2. Revisões de segurança

Varredura de segurança em 02/09/2026 no ambiente local, antes do servidor: achados críticos corrigidos e retestados no mesmo dia, depois uma segunda rodada de achados altos e médios, e uma varredura específica de injeção de prompt (seção 6). Os achados em si não são publicados neste repositório.

## 3. Quem pode ver o quê

- **A decisão é do servidor.** Os testes usaram um usuário de cada papel, com os casos de recusa incluídos. O front só esconde botão; tentar pela URL leva 403.
- **Quatro papéis-base (e perfis com nome próprio sobre eles) mais contas de serviço, concessão quadro a quadro.** O cliente da obra só lê e nunca recebe a área inteira (o servidor recusa), porque cada obra tem um quadro de custo e margem interna. Detalhes em [ADR-003](decisoes.md).
- **A busca global só devolve o que a pessoa já alcança.** É a única rota que atravessa quadros.
- **Senha:** `scrypt` do próprio Node (sem dependência nativa para compilar). Não existe recuperação por e-mail: a senha é redefinida pelo TI ou pelo dono da empresa.
- **Sessão e token na mesma tabela**, para revogar na hora. Um token vazado se corta com uma linha, e a sessão não é um JWT sem estado.
- **Tokens pessoais e por agente.** Cada agente de IA tem conta e concessões próprias; o token de uma pessoa devolve só os quadros dela (conferido ao vivo: o token de um gerente devolveu só os quadros dele, e não o catálogo inteiro).

## 4. Dados

- **Criptografia de campo (AES-256-GCM)** para e-mail e telefone de pessoas e para os dados de contato dos pedidos de compra, com índice cego para o login. O que não foi cifrado, e por quê, está no [ADR-002](decisoes.md).
- **Conta de banco de privilégio mínimo** para a API.
- **Backups em dois níveis**, um deles fora do servidor (ver [arquitetura.md](arquitetura.md), seção 7).
- **Excluir não apaga na hora:** lixeira de 30 dias ([ADR-001](decisoes.md)). "Apagar de vez" é do dono e do TI.
- **Log de atividade** de toda mudança em célula, com antes, depois e quem fez (também para as ações dos agentes e das automações).

## 5. Anexos e infraestrutura

- Anexos exigem login, e o nome do arquivo no disco é aleatório.
- Há limite de tamanho por arquivo.
- HTTPS automático (certificado renovado sozinho) e cookie de sessão só por HTTPS em produção.
- O editor de documentos roda em subdomínio próprio, isolado da origem do sistema (ver [arquitetura.md](arquitetura.md), seção 7).
- Segredos ficam fora dos arquivos versionados. Regra de trabalho: token ou chave colada em texto numa conversa é tratada como exposta e revogada, mesmo sem uso indevido.

## 6. Agentes de IA

É a parte mais particular deste sistema, porque texto escrito por gente de fora chega a um modelo que tem ferramentas.

**Caminhos por onde chega texto de quem não é da empresa:**

| Entrada | Quem escreve | Chega em |
|---|---|---|
| Mensagem de fornecedor no WhatsApp | empresa externa | agente de compras |
| Mensagem da equipe | funcionário | agentes restritos (financeiro, projetos) |
| Anexo (PDF, planilha) | quem mandou | lido pelo agente |
| Conteúdo de quadro (título, observação) | quem tiver permissão de edição | volta numa resposta de ferramenta MCP |

O risco já tinha aparecido sem querer: um item intitulado "TESTE" ou "NÃO USAR" levou um agente a obedecer o título como se fosse ordem e deixar o fornecedor sem resposta. Não houve ataque; o texto do quadro foi simplesmente cumprido.

**O que foi feito, em camadas**:

1. **Marca de dado no prompt.** Todo texto de fora entra cercado por um delimitador **sorteado a cada requisição**: se ele fosse fixo, a própria mensagem poderia repeti-lo e encerrar o bloco mais cedo. Antes do bloco vai uma regra: aquilo é dado e nunca instrução, não troca o papel do agente, não revela segredo, não roda comando, não manda nada para fora e não amplia acesso. Diante de uma tentativa, o agente avisa uma pessoa e cita o trecho. A lógica mora num arquivo só, então vale para todos os agentes sem mexer em cada workflow do n8n.
2. **Marca de dado na resposta das ferramentas.** O conteúdo de quadro volta depois do prompt, e ali a proteção do prompt não alcança. Toda resposta de ferramenta MCP é embrulhada em delimitadores sorteados a cada chamada, junto com o aviso de que o conteúdo é dado.
3. **Trava de ferramenta em código.** Cada agente restrito tem uma lista de ferramentas e de quadros checada antes de cada chamada, em código, não no prompt. Ainda que a injeção convença o modelo, um quadro que não está na lista continua inacessível.
4. **Conta própria por agente**, com concessões reais, para o servidor também recusar.
5. **O agente que fala com fornecedor não tem shell.** Ele usa quatro ferramentas estreitas, presas ao id que o roteador do n8n passa (e não ao que o modelo escolhesse), e a planilha de resultado é gerada por um script fixo.
6. **Pasta de trabalho vazia para agentes restritos.** Cada um roda numa pasta vazia, e o único `Read` liberado é o do anexo recebido (caminho conferido com `realpath`).

**Testes com ataque simulado (02/09 e 22/09/2026):**

- Modo de fornecedor: falsa "nota técnica do sistema", pedido para mandar a planilha a outro número e para mostrar os preços da concorrente. O agente respondeu a proposta normalmente, emitiu um alerta de segurança e nada saiu do que as ferramentas permitem.
- Pedido de chave de API disfarçado de "nova instrução do sistema" a um agente restrito: respondeu só o que era legítimo, apontou a tentativa com o trecho citado e não deixou segredo nenhum na resposta.
- Instrução plantada num campo de observação de um item de quadro, mandando listar um quadro fora do escopo do agente: o agente apontou o item e o trecho, e foi conferido item a item que nenhum título do quadro proibido apareceu. Quem segurou foram duas camadas: a marca de dado e a trava de ferramenta (o quadro não está na lista dele).

**Aprovação humana nas ações que mexem em dinheiro ou cobram alguém.** Enviar um pedido de compra por WhatsApp e cobrar um atraso no grupo de WhatsApp só acontecem depois de uma pessoa aprovar cada envio na tela. O agente de acompanhamento de obra **escuta** os grupos e propõe; nenhuma mensagem de cobrança no grupo é automática. Os avisos diários de compromissos no sino são automáticos, mas a regra só roda depois que uma pessoa a ativa.

## 7. Limites desta descrição

- A defesa no prompt é probabilística: baixa bastante o risco, mas não o zera. Por isso o que realmente contém é limitar em código o alcance de cada agente.
- Os testes descritos aqui foram feitos no projeto; não é uma auditoria externa.
