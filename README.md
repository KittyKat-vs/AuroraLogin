**Português** · [English](README.en.md) · [Русский](README.ru.md)

# AuroraLogin

Autenticação de jogador para Paper. Registro, login, sessão por IP, proteção de conta e um painel dentro do jogo para ajustar tudo.

Um jar só. Sem dependência, sem site externo, sem banco remoto.

**Versão:** 1.10.2  
**Servidor:** Paper 1.20.1 ou mais novo  
**Java:** 17, ou o que a sua versão do servidor já pedir  
**Dependências:** nenhuma  
**Custo:** gratuito

## Por que ele existe

Servidor com `online-mode=false` tem um problema óbvio: qualquer um entra com o nome de qualquer um. O AuroraLogin fecha isso com registro e senha.

O que eu não queria era te obrigar a montar site, subir banco remoto e instalar meia dúzia de plugin antes de ter login funcionando. Então a conta fica num SQLite dentro da pasta do plugin, a senha vira hash PBKDF2-SHA256, e depois de configurar uma vez você provavelmente nem abre o `config.yml` de novo. Tudo tem no painel do jogo.

Tem uma parte que costuma dar trabalho e ele já resolve: **conta original do Minecraft entra sozinha**, sem registrar, mesmo em servidor offline. Isso fica dentro do jar, sem ProtocolLib e sem plugin de verificação.

## Resumo rápido

- Registro, login, troca de senha e logout, com sessão por IP.
- Conta original entra sozinha. Em servidor offline o plugin confere na Mojang durante a entrada, sem prender a conexão, e quem não tem nome comprado nem é incomodado: entra de primeira.
- Bedrock entra sozinho se o Floodgate estiver instalado.
- Painel em `/auroraauth panel`: configuração, lista de jogadores e de IPs com strike.
- Strike progressivo contra força bruta: kick, kick maior, ban de IP.
- Português, inglês e russo, inclusive no nome dos comandos.
- Configuração comentada, e o comentário não some quando o painel grava.

## Como é entrar no servidor

O jogador chega e fica retido. Não abre inventário, não quebra bloco, não usa item. Andar antes de logar não adianta, ele volta pro ponto onde entrou. Os efeitos de pré-login (cegueira, lentidão, fadiga) estão aí para ele entender que precisa se autenticar.

No chat aparece o botão com o comando certo, já no idioma do servidor. Na tab, a lista de comandos fica reduzida aos de autenticação. Ele registra com confirmação de senha, ou entra com a que já tem. Autenticou: os efeitos saem, a lista volta, ele joga.

Voltou dentro do período da sessão? Entra direto, sem digitar nada. Conta original entra sem nada disso, e Bedrock com Floodgate também. No fim é isto: dono da conta entra, quem não é fica do lado de fora.

## Contas originais

`premium-auto-login` vem ligado. Com ele, conta original entra sem registro e sem senha.

- Servidor com online-mode: a identidade já chega validada pela Mojang e o plugin usa direto.
- Servidor offline sem proxy: o plugin decide antes de desafiar. Primeiro ele pergunta ao catálogo oficial de nomes da Mojang se aquele nome pertence a uma conta paga. Nome que ninguém comprou não recebe pedido de criptografia nenhum e vai direto para o registro, na primeira entrada. Nome de conta paga é desafiado, o jogador entra assim que aceita a criptografia e a sessão é conferida no servidor de sessões da Mojang em paralelo, sem ele esperar na porta. Nome que a Mojang já confirmou uma vez fica anotado no banco do plugin, e na volta é desafiado na hora, sem espera de rede. Tudo dentro do jar.
- Velocity ou BungeeCord com forwarding: o plugin percebe e não briga com o proxy. Deixa a verificação para lá e segue o fluxo normal de login.
- Deu errado? A Mojang não confirmou, o catálogo ficou mudo, respondeu tarde ou marcou limite de requisições, ou a estrutura de uma versão nova não foi reconhecida. O jogador cai no fluxo normal de registro e senha. Nenhuma dessas falhas empurra alguém para fora e nenhuma atrasa o login: o catálogo tem 4,5 segundos para responder no login (3 para conectar, 4 para a requisição) e sem resposta o plugin não desafia; a conferência de sessão tem 5 segundos para responder e, sem resposta, o prompt de senha aparece e a pessoa registra normal. Dá para desligar tudo em `premium-verification`. Uma falha fica lembrada 5 segundos e a espera global é de 3, então um erro não empurra os próximos jogadores para o fluxo de senha. No start e a cada `/auroraauth reload` o plugin abre a conexão com a Mojang usando um nome de teste, para o primeiro jogador não pagar o custo de uma conexão fria, e o aviso no console sai uma vez só. `/auroraauth reload` também limpa o que o catálogo respondeu. Se um nome de conta paga foi usado por quem não tem a conta, veja [Cliente sem sessão Mojang](#cliente-sem-sessão-mojang-o-pino).
- Na primeira entrada o plugin cria a conta interna com uma senha aleatória que ninguém nunca vê, só para o banco ficar consistente. Se já existe conta com aquele nome, ela é vinculada ao jogador em vez de duplicada.
- O UUID não muda. Mundo, permissão e dado que você já tem continuam valendo.
- A conta fica marcada como verificada no banco, e o perfil dela no painel mostra isso. **Selo no chat e na tab não tem**: o nome aparece normal, que é o que a maioria quer.
- E enquanto a sessão não responde? O jogador já está no mundo, mas retido igual qualquer um sem sessão: efeitos de pré-login, sem comando e sem chat liberado, e nenhuma permissão de autenticado. Respondeu a favor, ele é liberado sem digitar nada; respondeu contra, o prompt de senha chega no idioma dele. Na prática é uma fração de segundo, e ninguém vira jogador autenticado antes da Mojang confirmar.

### Cliente sem sessão Mojang (o pino)

Nome de conta paga usado por quem não tem a conta ainda é desafiado, e é de propósito: é isso que impede alguém de entrar no servidor com a identidade do dono. O launcher desse jogador desiste ali, com aquele aviso de *Invalid session*, ainda no cliente, antes de qualquer senha. Mas uma vez só.

O plugin registra o par nome e IP que morreu no handshake (ou ficou mudo por 4 segundos) e para de desafiar. Da próxima entrada ele nem é consultado no catálogo: vai reto para o fluxo de registro e senha, como se a checagem nunca tivesse existido para ele. Quem não tem nome comprado nunca passa por aqui, porque nunca foi desafiado.

- `/auroraauth premium` lista os clientes registrados, com nome e IP.
- `/auroraauth premium reset <nome|ip>` limpa. Depois do reset o cliente volta a ser desafiado, e se cair de novo volta para o pino.
- O pino fica no `accounts.db` e na memória. Reiniciar o servidor não apaga nada.
- Cliente que responde ao desafio nunca é pinado: virou pino significa que ele não respondeu.
- `premium-verification: false` desliga a checagem inteira, e aí nenhum cliente é desafiado nem pinado.

Cada pino gravado aparece no console com nome e IP, para você conferir quando quiser. Não é preciso ficar olhando: o plugin se vira sozinho.

## Bedrock (Floodgate/Geyser)

Com `floodgate-auto-login` ligado e o Floodgate instalado, quem entra pelo Bedrock com identidade confirmada pelo Floodgate entra direto, sem senha.

A identificação usa a API do próprio Floodgate (`isFloodgatePlayer`), nunca o prefixo no nome. Cliente Java não consegue se passar por jogador de celular. Sem Floodgate o bypass fica desligado, e o jogador Bedrock registra com senha como qualquer outro, sem erro e sem aviso no console. O UUID do Floodgate é preservado e a sessão por IP vale igual.

As duas entradas automáticas você liga e desliga sem abrir arquivo: `/autologin off` desliga tudo, `/autologin on` religa, e sem argumento o comando mostra como está agora. Para controlar uma de cada vez, tem o painel em **Geral**. O nome do comando segue o idioma (`/autologin` em pt-br e inglês, `/автовход` em russo).

## Proteção contra força bruta

O bloqueio é por IP e sobe o tom aos poucos, em vez de banir no primeiro erro. Se o IP errar `max-attempts` senhas dentro da janela configurada:

1. primeira vez: expulsão por 5 minutos;
2. segunda: expulsão por 15 minutos;
3. terceira: ban de IP por 24 horas.

Os três tempos você ajusta no painel. O ban expira sozinho no prazo configurado, e o strike zera depois de 7 dias sem novo excesso.

Enquanto o IP está bloqueado ou banido, a conexão nem chega a entrar. A tela de desconexão mostra o IP, o tempo restante e, no banimento, o motivo e o convite do Discord que você configurar. É o jogador tendo por onde recorrer em vez de abrir tópico no seu fórum.

Precisa olhar o que está travando alguém antes do prazo? **Segurança → IPs e strikes**. A lista traz todo IP que encostou no servidor — bloqueado, banido, expirado, liberado ou sem nenhuma strike — com o nome da última tentativa de entrada, a strike e o status, do mais recente para o mais antigo. A busca aceita parte do IP, o botão de atualizar recarrega sem fechar o menu, e cada IP abre uma tela com a cabeça de quem tentou entrar por último e duas ações separadas: **Remover bloqueio** libera o tempo na hora e mantém as strikes, **Apagar registro** tira o IP do banco e zera as strikes. Nenhuma das duas encosta em conta ou senha. Um IP para de aparecer quando fica `strike-reset-hours` sem tentar entrar.

E antes de autenticar não é só a senha que está travada. O chat fica fechado, os comandos são restringidos e o mundo também: inventário, drop, coleta, troca de mão, bloco e entidade, portal, balde, consumo de item e tosquia.

## Idiomas

Três idiomas completos: português (pt-br), inglês e russo. Trocar o idioma muda o servidor inteiro, inclusive o nome dos comandos de autenticação. Os antigos deixam de existir e os novos valem na hora.

| Idioma | Registrar | Entrar | Sair | Trocar senha | Entrada automática |
| --- | --- | --- | --- | --- | --- |
| pt-br | `/registrar` | `/entrar` | `/sair` | `/trocarsenha` | `/autologin` |
| en | `/register` | `/login` | `/logout` | `/changepassword` | `/autologin` |
| ru | `/регистрация` | `/вход` | `/выход` | `/сменитьпароль` | `/автовход` |

O comando administrativo `/auroraauth` é o mesmo em qualquer idioma. O texto de cada um fica solto em `lang/`, edite como quiser.

Uma coisa que incomoda, e por isso foi tratada à parte: enquanto o jogador digita a senha em `/registrar`, `/entrar` ou `/trocarsenha`, a tab não sugere nome de ninguém. Senha sendo digitada não é hora de vazar a lista de jogadores.

## Painel dentro do jogo

`/auroraauth panel` abre o menu. Cada clique grava em `config.yml` na hora, sem reiniciar e sem bagunçar o arquivo.

| Categoria | O que você ajusta |
| --- | --- |
| Geral | Sessão, entrada automática de conta original, verificação da Mojang no login, tempo para autenticar, tamanho de senha, senha igual ao nome, reutilização, teletransporte e bloqueio de interações |
| Segurança | Força bruta, tentativas, janela, os três tempos de strike, ban de IP e a lista de IPs com strike, com busca, atualização e duas ações por IP |
| Visual | Efeitos de pré-login, títulos, sons e barra de ação |
| Jogadores | Lista de contas, busca por nome, perfil e ações administrativas |
| Idioma | Troca o idioma do servidor inteiro |
| Sistema | Auditoria, sons do painel e recarregar a configuração |

Nos ajustes numéricos: clique esquerdo aumenta, direito diminui, com shift o passo é maior.

### Jogadores

A aba **Jogadores** lista todas as contas, inclusive as de quem está offline, paginadas, cada uma com a cabeça do jogador.

- A busca funciona digitando o nome no chat depois de clicar em Buscar. Cancela com `cancelar`, `cancel` ou `отмена`.
- O perfil mostra UUID, criação, último login, último IP com o status e a strike dele, se está autenticado, tempo de bloqueio e se a conta foi verificada pela Mojang.
- Redefinir senha gera uma temporária de 12 caracteres, mostrada só para quem executou, com clique para copiar. O jogador não é expulso: continua jogando, a sessão por IP dele acaba, e a senha nova vale no próximo login.
- Dá também para desconectar, remover o bloqueio e remover a conta. O remover bloqueio limpa as duas travas de uma vez — o tempo gravado no IP e o contador da sessão — e mantém a strike para o próximo excesso valer como segunda vez. Remover a conta tem tela de confirmação antes.

O perfil de um jogador específico abre direto com `/auroraauth panel <jogador>`.

## Comandos

### De jogador

| Comando | O que faz |
| --- | --- |
| `/registrar <senha> <senha>` | Cria a conta |
| `/entrar <senha>` | Autentica na conta |
| `/sair` | Encerra a sessão |
| `/trocarsenha <atual> <nova> <nova>` | Troca a senha |

Os nomes mudam com o idioma, como na tabela de cima.

### De administração

Todo subcomando tem um atalho curto, para você não precisar digitar `/auroraauth` na frente. Os dois formatos usam a mesma permissão e respondem as mesmas mensagens.

| Comando completo | Atalho | O que faz |
| --- | --- | --- |
| `/auroraauth info <jogador>` | `/info <jogador>` | Mostra os dados da conta |
| `/auroraauth unregister <jogador>` | `/unregister <jogador>` | Pede confirmação para remover a conta |
| `/auroraauth unregister <jogador> confirm` | `/unregister <jogador> confirm` | Remove a conta |
| `/auroraauth resetpassword <jogador>` | `/resetpassword <jogador>` | Gera senha temporária e redefine |
| `/auroraauth language <pt-br\|en\|ru>` | `/language <pt-br\|en\|ru>` | Troca o idioma do servidor |
| `/auroraauth panel [jogador]` | `/panel [jogador]` | Abre o painel ou o perfil (só no jogo) |
| `/auroraauth autologin` | `/autologin` | Mostra o estado da entrada automática |
| `/auroraauth autologin <on\|off>` | `/autologin <on\|off>` | Liga ou desliga as entradas automáticas |
| `/auroraauth premium` | — | Lista os clientes sem sessão Mojang (o pino) |
| `/auroraauth premium reset <nome\|ip>` | — | Tira um cliente do pino |
| `/auroraauth reload` | `/reload` | Recarrega configuração e mensagens |

Nome com namespace (`/auroralogin:reload`) funciona sempre. O `/reload` curto recarrega este plugin; para os datapacks do servidor, `/minecraft:reload` continua no lugar dele. E se outro plugin já tiver tomado algum atalho, o AuroraLogin desiste do nome, avisa no console e segue funcionando pelo `/auroraauth`.

## Permissões

`auroralogin.admin` liga tudo e já inclui as demais. Por padrão, só quem é op tem.

| Permissão | O que libera | Padrão |
| --- | --- | --- |
| `auroralogin.admin` | Acesso completo | op |
| `auroralogin.admin.info` | Consultar contas | op |
| `auroralogin.admin.unregister` | Remover contas | op |
| `auroralogin.admin.resetpassword` | Redefinir senhas | op |
| `auroralogin.admin.language` | Trocar o idioma do servidor | op |
| `auroralogin.admin.panel` | Abrir o painel | op |
| `auroralogin.admin.autologin` | Ligar e desligar a entrada automática | op |
| `auroralogin.admin.premium` | Ver e limpar o registro de clientes sem sessão Mojang | op |
| `auroralogin.admin.reload` | Recarregar configuração e mensagens | op |
| `auroralogin.admin.kick` | Desconectar jogador pelo painel | op |
| `auroralogin.admin.unblock` | Remover bloqueio de tentativas, do jogador e do IP | op |
| `auroralogin.admin.unban` | Abrir a lista de IPs e apagar registro | op |

## Instalação

1. Baixe o jar mais recente na aba **Releases**.
2. Jogue o arquivo em `plugins/` do servidor Paper.
3. Inicie o servidor. A pasta `plugins/AuroraLogin/` aparece com a configuração e os idiomas.
4. Ajuste pelo painel com `/auroraauth panel`, ou edite o `config.yml` e rode `/auroraauth reload`.

Pré-requisito: Paper 1.20.1 ou mais novo e Java 17. Nada além disso, porque a verificação de conta original em servidor offline já vem dentro do jar. Quer que o pessoal do Bedrock entre sem senha? Aí sim você precisa do Floodgate (com o Geyser). O plugin detecta ele sozinho, e sem ele tudo continua funcionando, só com senha para o bedrock.

## Configuração

O `config.yml` vem com uma explicação por opção. E o painel preserva isso: cada clique reescreve só a linha do valor, então o comentário continua lá.

Servidor que já tinha configuração antiga é atualizado sozinho na primeira inicialização, reaproveitando seus valores e guardando o arquivo anterior como `config.yml.bak`.

A lista completa, com cada opção explicada e o arquivo de exemplo, está em [docs/configuracao.md](docs/configuracao.md) — também em [inglês](docs/configuracao.en.md) e [russo](docs/configuracao.ru.md). As mensagens ficam em `plugins/AuroraLogin/lang/`, um arquivo por idioma (`pt_br.yml`, `en.yml`, `ru.yml`).

## Sobre os dados do seu servidor

- Senha vira hash PBKDF2-SHA256 com 600.000 iterações. Não fica em texto puro no banco, não fica em log, e não existe comando que mostre a senha de alguém.
- A redefinição administrativa cria uma temporária de 12 caracteres, mostrada só para quem executou a ação, com clique para copiar. O jogador continua online.
- As contas ficam em `plugins/AuroraLogin/accounts.db`, SQLite local. Backup é copiar o arquivo com o servidor parado.
- A mesma base guarda a tela de IPs: para cada IP, a data da última tentativa de entrada, o nome que foi usado nela e o UUID daquele cliente. É o que permite ver quem tentou entrar por um IP antes de qualquer strike. Isso não sai do seu servidor, e a linha some sozinha quando o IP fica `strike-reset-hours` sem aparecer.
- A auditoria em `plugins/AuroraLogin/audit.log` registra o que a administração fez, sem tocar em senha.
- A checagem de conta original conversa só com dois endereços oficiais da Mojang, durante o login: o catálogo de nomes e o servidor de sessões. São consultas de leitura, e nada do seu servidor vai junto: sem token, sem chave, sem lista de jogadores. O catálogo responde pelo nome, a sessão confirma o cliente, e a confirmação vale apenas para aquela conexão. A resposta do catálogo fica guardada um tempo justamente para não martelar a Mojang a cada entrada, e os nomes que a Mojang confirmou como pagos ficam anotados no banco do plugin — só o nome e a data, para a volta do dono não depender de rede. Não tem opção para apontar nenhuma dessas checagens para outro endereço, e não vai ter: se existisse campo para isso, alguém apontaria para uma máquina que responde "sim" para qualquer nome.
- Nada do que acontece no seu servidor sai para fora, exceto essa consulta pontual à Mojang. Não tem site, telemetria ou serviço de terceiro no meio.

## Compatibilidade

Testado em Paper 1.20.1 e Paper 1.21.1, sem nenhum outro plugin instalado.

A verificação de conta original em servidor offline acompanha o pipeline de rede do próprio servidor. É o que dispensa o ProtocolLib e é também o calcanhar de aquiles: se uma versão futura mudar demais a estrutura interna e o plugin não reconhecer, ele avisa no console, deixa a verificação desligada e o login por senha continua exatamente como está. Ninguém fica travado do lado de fora por causa disso. Nesse caso é esperar a atualização do plugin.

## Changelog

**1.10.2** — Conta original não fica mais presa na tela de conexão. O plugin esperava o `hasJoined` da Mojang responder antes de liberar o login, e isso era custo por entrada, não exceção: na minha medição, ~420 ms em toda entrada de conta paga, mesmo com o nome já anotado no catálogo. Agora o login é liberado assim que o cliente aceita a criptografia, a conexão fica marcada como pendente e a Mojang é consultada do lado. Confirmou, entra sozinho, sem prompt e sem digitar nada. Não confirmou, cai no fluxo de senha do mesmo jeito, só que com o aviso chegando depois do veredito. Até a resposta chegar o jogador está no mundo e retido: efeitos de pré-login, sem comando, sem chat, sem nenhuma permissão de autenticado. No teste, com a resposta forçada a demorar 1,5 s, ele pisou no mundo 0,6 s antes do veredito. Quem sai antes da resposta não gera exceção nem deixa pendência no banco, e o tempo limite de verificação que segurava essa espera saiu junto com a espera.

Pirata entra de primeira. Todo cliente recebia o pedido de criptografia no login, e o launcher de quem não tem conta desistia com o aviso de *Invalid session* sem nunca chegar na senha. Agora, antes de desafiar, o plugin pergunta ao catálogo oficial de nomes da Mojang se aquele nome é de conta paga: nome que ninguém comprou não recebe pedido de criptografia nenhum e vai direto para o registro, na primeira entrada. Nome de conta paga continua sendo desafiado, porque é isso que impede alguém de entrar com a identidade do dono, e quem usa esse nome sem ter a conta é anotado por nome e IP depois do erro único e para de ser desafiado — o login seguinte vai reto para o fluxo de senha. `/auroraauth premium` lista os anotados e `/auroraauth premium reset <nome|ip>` limpa um. Permissão nova: `auroralogin.admin.premium`. Catálogo fora do ar, lento ou marcando limite de requisições também não desafia e não atrasa ninguém.

Conta original voltou a entrar sozinha no servidor recém-ligado. O primeiro login depois do start pagava uma conexão fria com a Mojang (DNS e TLS do zero) e, com o teto antigo, a resposta certa chegava depois do prazo: o nome ia para o fluxo de senha mesmo sendo pago. Agora o plugin abre a conexão no start e em cada `/auroraauth reload` com um nome de teste, e o orçamento é de 3 segundos para conectar e 4 para a requisição, com 4,5 segundos de espera no login; uma falha fica lembrada 5 segundos e a espera global é de 3. Nome que a Mojang confirmou como pago fica anotado no `accounts.db` (tabela `premium_present`, só nome e data) e na volta é desafiado na hora, sem rede nenhuma: com o catálogo fora do ar, o dono entra sozinho do mesmo jeito. A resposta do catálogo fica em cache 15 minutos para nome pago e 30 para nome livre, e o reload limpa as duas coisas.

No painel, o botão de remover bloqueio passou a mexer no bloqueio de verdade. Existiam duas travas, uma na memória por jogador e outra no banco por IP, e o perfil só limpava a primeira: o admin clicava, recebia a confirmação e o jogador continuava caindo na tela de bloqueio. Agora o perfil limpa as duas e mostra o status e a strike do último IP da conta. E a tela de IPs virou o registro de entrada, não só a lista de problema. Antes uma linha nascia no primeiro excesso, então quem só encostou em um servidor limpo não aparecia; agora toda conexão que chega ao login grava o IP, o nome usado na tentativa, o UUID do cliente e a hora, sem chave nova no config e sem opção para desligar. Strike continua sendo strike: a tentativa não zera, não empurra e não apaga bloqueio nenhum. A lista traz todo IP do banco — bloqueado, banido, expirado ou já liberado — ordenada pela atividade mais recente, com busca por parte do IP e botão de atualizar, e cada IP abre a tela com a cabeça de quem tentou entrar por último, a strike, o status e duas ações: **Remover bloqueio** libera o tempo na hora e mantém as strikes, **Apagar registro** tira a linha e zera as strikes. É assim que se vê o impostor antes de ele autenticar: nome pago sendo tentado por outra pessoa aparece na lista sem que ele tenha errado senha uma vez.

**1.10.1** — Comentários do `config.yml` reescritos para explicar o que cada opção faz na prática, inclusive o que muda se você desligar. Documentação de configuração separada do README.

**1.10.0** — Configuração reorganizada em seções, com cabeçalho e uma explicação por opção, que sobrevive a qualquer gravação feita pelo painel ou pelos comandos. Migração automática de configuração antiga, com backup em `config.yml.bak`. O endereço do servidor de sessões da Mojang saiu do arquivo: a checagem sempre consulta o servidor oficial e não há como redirecionar.

**1.9.2** — Folga nas duas laterais da grade de opções do painel.

**1.9.1** — Alinhamento da primeira linha de opções de cada categoria.

**1.9.0** — Strike progressivo por IP: kick de 5 minutos, kick de 15 minutos e ban de IP de 24 horas, todos ajustáveis. Tela de banimento mostrando o IP, o tempo restante, o motivo e o convite do Discord. Lista de IPs banidos com desbanimento pelo painel e expiração automática.

**1.8.3** — Atalhos para todos os subcomandos administrativos, sem `/auroraauth` na frente. O comando da entrada automática também segue o idioma. Ao digitar a senha, a tab parou de sugerir nome de jogador.

**1.8.2** — `/autologin on|off` controla as duas entradas automáticas de uma vez e, sem argumento, mostra o estado atual.

**1.8.0** — Entrada automática para Bedrock com Floodgate, identificando pela API do Floodgate e não pelo prefixo do nome.

**1.7.3** — Conta original identificada apenas no banco e no perfil do painel, sem selo no chat nem na tab.

**1.7.2** — Senha temporária da redefinição com clique para copiar, e o jogador redefinido continua online.

**1.7.0** — Verificação da conta original dentro do próprio plugin, em jar único, sem protocolo auxiliar, funcionando em servidor offline.

**1.5.5** — Entrada automática de conta original, vinculando a conta que já existia em vez de duplicar.

**1.5.0** — Comandos de autenticação no idioma do servidor, com os nomes antigos deixando de existir ao trocar.

**1.4.0** — Aba **Jogadores**: lista paginada de contas online e offline, busca por nome no chat, perfil completo e ações administrativas. Aba **Sistema** com auditoria, sons do painel e recarregar.

**1.3.0** — Painel gráfico dentro do jogo, por categorias, com toggles e ajustes numéricos salvando na hora.

**1.2.0** — Português, inglês e russo, efeitos de pré-login configuráveis, lista de comandos reduzida até autenticar, botões clicáveis no chat e contagem regressiva na barra de ação.

**1.1.0** — Sessão por IP, proteção contra força bruta, política de senha, bloqueio de interações antes do login, permissões administrativas separadas, remoção de conta com confirmação e auditoria.

**1.0.0** — Registro, login, troca de senha, logout e administração básica.

## Suporte

Achou um bug, tem uma sugestão, ou viu um comportamento estranho e quer entender antes de assumir que a culpa é do plugin? Abra uma issue neste repositório. Ajudam muito a versão do Paper, a versão do plugin e o trecho do console onde deu problema.

Site: https://auroraplugins.duckdns.org

Feito por Kat. Uma pessoa, sem equipe.
