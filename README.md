**Português** · [English](README.en.md) · [Русский](README.ru.md)

# AuroraLogin

Autenticação de jogadores para Paper: registro, login, sessão por IP, proteção de conta e um painel dentro do jogo para ajustar tudo. Um jar só, sem dependência nenhuma, sem site externo e sem banco de dados remoto.

**Versão:** 1.10.1  
**Servidor:** Paper 1.20.1 ou mais novo  
**Java:** 17 ou o que a sua versão do servidor já exige  
**Dependências:** nenhuma  
**Custo:** gratuito

## Por que ele existe

Todo servidor que não usa online-mode tem o mesmo problema: qualquer pessoa entra com o nome de qualquer pessoa e pronto. O AuroraLogin fecha isso com registro e senha, mas sem te obrigar a montar site, banco remoto ou instalar meia dúzia de plugins junto. As contas ficam num SQLite dentro da própria pasta do plugin, a senha vira hash PBKDF2-SHA256 e, depois de configurar uma vez, você normalmente nem abre o arquivo de configuração de novo, porque tudo tem no painel do jogo.

Ele também resolve uma parte que costuma dar trabalho: **quem tem conta original do Minecraft entra sozinho**, sem precisar registrar, mesmo em servidor com `online-mode=false`. Isso funciona dentro do próprio jar, sem ProtocolLib e sem plugin de verificação.

## Resumo rápido

- Registro, login, troca de senha e logout, com sessão por IP.
- Conta original entra sozinha; em servidor offline o plugin confere na Mojang durante o login.
- Bedrock entra sozinho quando o Floodgate está instalado.
- Painel `/auroraauth panel` com as configurações, lista de jogadores e IPs banidos.
- Strike progressivo contra força bruta: kick, kick maior e ban de IP.
- Português, inglês e russo, inclusive no nome dos comandos.
- Configuração comentada, e os comentários não somem quando o painel grava.

## Como é entrar no servidor

O jogador chega e fica retido: não abre inventário, não quebra bloco, não usa item, e ainda leva os efeitos de pré-login (cegueira, lentidão, fadiga) para perceber que precisa se autenticar. Se ele andar antes de logar, volta para o ponto onde entrou.

No chat aparece o botão com o comando certo, já no idioma do servidor, e a lista de comandos da tab fica reduzida aos de autenticação. Ele registra com confirmação de senha ou entra com a senha que já tem. Autenticou, os efeitos saem, a lista de comandos volta e ele joga normalmente.

Quem voltar dentro do período da sessão entra direto, sem digitar de novo. Conta original entra sem nada disso, e jogador Bedrock com Floodgate também. No fim, é isto: quem é dono da conta entra, quem não é fica do lado de fora.

## Contas originais

Com `premium-auto-login` ligado (vem ligado), a conta original do Minecraft entra sem registro e sem senha.

- Em servidor com online-mode, a identidade já chega validada pela Mojang e o plugin usa direto.
- Em servidor offline sem proxy, o plugin faz a checagem durante o próprio login: conversa com o cliente, confirma a sessão no servidor de sessões da Mojang e só então libera. Isso fica dentro do jar.
- Se você usa Velocity ou BungeeCord com forwarding, o plugin percebe e não compete com o proxy: ele deixa a verificação para lá e segue o fluxo normal de login.
- Se a Mojang não confirmar, o servidor de sessões estiver fora do ar, ou a estrutura interna de uma versão nova não for reconhecida, o jogador cai no fluxo normal de registro e senha. Nenhuma entrada é bloqueada por causa dessa checagem, e ela pode ser desligada em `premium-verification`.
- Na primeira entrada o plugin cria a conta interna com uma senha aleatória que ninguém nunca vê, só para manter o banco consistente. Se já existir conta com aquele nome, ela é vinculada ao jogador em vez de duplicada.
- O UUID não muda, então mundo, permissões e dados que você já tem continuam valendo.
- A conta fica marcada como verificada no banco e o perfil dela no painel mostra isso. **Não tem selo no chat nem na tab**: o nome aparece normal, como você provavelmente prefere.

## Bedrock (Floodgate/Geyser)

Com `floodgate-auto-login` ligado e o Floodgate instalado, quem entra pelo Bedrock com identidade confirmada pelo Floodgate entra direto, sem senha.

A identificação usa a API do próprio Floodgate (`isFloodgatePlayer`), nunca o prefixo no nome, então um cliente Java não consegue se passar por jogador de celular. Sem o Floodgate instalado, o bypass fica desligado e o jogador Bedrock registra com senha como qualquer outro, sem erro nem aviso no console. O UUID do Floodgate é preservado e a sessão por IP vale igual.

Dá para ligar e desligar as duas entradas automáticas de uma vez sem abrir arquivo: `/autologin off` desliga tudo, `/autologin on` liga de novo, e sem argumento o comando mostra como está agora. Para controlar cada uma separado, use o painel em **Geral**. O nome do comando também segue o idioma (`/autologin` em pt-br e inglês, `/автовход` em russo).

## Proteção contra força bruta

O bloqueio é por IP e sobe o tom aos poucos, em vez de banir no primeiro erro. Se o IP errar `max-attempts` senhas dentro da janela configurada:

1. primeiro excesso: expulsão por 5 minutos;
2. segundo excesso: expulsão por 15 minutos;
3. terceiro excesso: ban de IP por 24 horas.

Os três tempos são ajustáveis no painel, o ban expira sozinho no prazo configurado e os strikes zeram depois de 7 dias sem novo excesso. Enquanto o IP está bloqueado ou banido, a conexão nem chega a entrar: a tela de desconexão mostra o IP, o tempo restante e, no banimento, o motivo e o convite do Discord que você configurar. É o jogador tendo por onde recorrer em vez de abrir tópico no seu fórum.

Para tirar um ban antes do prazo, **Segurança → IPs banidos** no painel lista cada IP com strikes, data do ban, expiração, tempo restante e motivo, e desbanir ali não apaga conta nem senha, libera só o IP.

Além disso, o chat fica travado até autenticar, os comandos são restringidos e as interações do mundo ficam bloqueadas (inventário, drop, coleta, troca de mão, blocos e entidades, portais, baldes, consumo de item e tosquia).

## Idiomas

Três idiomas completos: português (pt-br), inglês e russo. Trocar o idioma muda o servidor inteiro, incluindo o nome dos comandos de autenticação; os antigos deixam de existir e os novos valem na hora.

| Idioma | Registrar | Entrar | Sair | Trocar senha | Entrada automática |
| --- | --- | --- | --- | --- | --- |
| pt-br | `/registrar` | `/entrar` | `/sair` | `/trocarsenha` | `/autologin` |
| en | `/register` | `/login` | `/logout` | `/changepassword` | `/autologin` |
| ru | `/регистрация` | `/вход` | `/выход` | `/сменитьпароль` | `/автовход` |

O comando administrativo `/auroraauth` é o mesmo em qualquer idioma, e o texto de cada idioma fica solto em `lang/` para você editar como quiser.

Uma coisa que costuma incomodar e por isso foi tratada à parte: quando o jogador está digitando a senha em `/registrar`, `/entrar` ou `/trocarsenha`, a tab não sugere nome de ninguém. Nada de vazar a lista de jogadores enquanto alguém escreve a senha.

## Painel dentro do jogo

`/auroraauth panel` abre o menu. Cada clique grava em `config.yml` na hora, sem reiniciar e sem bagunçar o arquivo.

| Categoria | O que você ajusta |
| --- | --- |
| Geral | Sessão, entrada automática de conta original, verificação da Mojang no login, tempo para autenticar, tamanho de senha, senha igual ao nome, reutilização, teletransporte e bloqueio de interações |
| Segurança | Força bruta, tentativas, janela, os três tempos de strike, ban de IP e a lista de IPs banidos |
| Visual | Efeitos de pré-login, títulos, sons e barra de ação |
| Jogadores | Lista de contas, busca por nome, perfil e ações administrativas |
| Idioma | Troca o idioma do servidor inteiro |
| Sistema | Auditoria, sons do painel e recarregar a configuração |

Nos ajustes numéricos, clique esquerdo aumenta, direito diminui e com shift o passo é maior.

### Jogadores

A aba **Jogadores** lista todas as contas, inclusive as de quem está offline, paginadas, cada uma com a cabeça do jogador.

- A busca funciona digitando o nome no chat depois de clicar em Buscar (cancela com `cancelar`, `cancel` ou `отмена`).
- O perfil mostra UUID, criação, último login, último IP, se está autenticado, tempo de bloqueio e se a conta foi verificada pela Mojang.
- Redefinir senha gera uma temporária de 12 caracteres, mostrada só para quem executou, com clique para copiar. O jogador não é expulso: ele continua jogando, a sessão por IP dele acaba, e a senha nova vale no próximo login.
- Também dá para desconectar, remover o bloqueio de tentativas e remover a conta, esta com tela de confirmação antes.

O perfil de um jogador específico abre direto com `/auroraauth panel <jogador>`.

## Comandos

### De jogador

| Comando | O que faz |
| --- | --- |
| `/registrar <senha> <senha>` | Cria a conta |
| `/entrar <senha>` | Autentica na conta |
| `/sair` | Encerra a sessão |
| `/trocarsenha <atual> <nova> <nova>` | Troca a senha |

Os nomes mudam com o idioma, como na tabela acima.

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
| `/auroraauth reload` | `/reload` | Recarrega configuração e mensagens |

O nome com namespace (`/auroralogin:reload`) funciona sempre. O `/reload` curto recarrega este plugin; para os datapacks do servidor, `/minecraft:reload` continua no lugar dele. Se outro plugin já tiver tomado algum desses atalhos, o AuroraLogin desiste do nome, avisa no console e segue funcionando pelo `/auroraauth`.

## Permissões

Tudo é ligado por `auroralogin.admin`, que já inclui as demais. Por padrão, só quem é op tem.

| Permissão | O que libera | Padrão |
| --- | --- | --- |
| `auroralogin.admin` | Acesso completo | op |
| `auroralogin.admin.info` | Consultar contas | op |
| `auroralogin.admin.unregister` | Remover contas | op |
| `auroralogin.admin.resetpassword` | Redefinir senhas | op |
| `auroralogin.admin.language` | Trocar o idioma do servidor | op |
| `auroralogin.admin.panel` | Abrir o painel | op |
| `auroralogin.admin.autologin` | Ligar e desligar a entrada automática | op |
| `auroralogin.admin.reload` | Recarregar configuração e mensagens | op |
| `auroralogin.admin.kick` | Desconectar jogador pelo painel | op |
| `auroralogin.admin.unblock` | Remover bloqueio de tentativas | op |
| `auroralogin.admin.unban` | Desbanir IP pela lista do painel | op |

## Instalação

1. Baixe o jar mais recente na aba **Releases**.
2. Jogue o arquivo em `plugins/` do servidor Paper.
3. Inicie o servidor. A pasta `plugins/AuroraLogin/` aparece com a configuração e os idiomas.
4. Ajuste pelo painel com `/auroraauth panel`, ou edite o `config.yml` e rode `/auroraauth reload`.

Pré-requisitos: Paper 1.20.1 ou mais novo e Java 17 ou mais novo. Nada além disso, porque a verificação de conta original em servidor offline já vem dentro do jar. Se você quer que o pessoal do Bedrock entre sem senha, aí sim precisa do Floodgate (com o Geyser); o plugin detecta ele sozinho, e sem ele tudo continua funcionando, só com senha para o bedrock.

## Configuração

O `config.yml` vem com uma explicação por opção, e o painel preserva isso: como cada clique reescreve apenas a linha do valor, os comentários continuam lá. Servidor que já tinha configuração antiga é atualizado sozinho na primeira inicialização, reaproveitando seus valores e guardando o arquivo anterior como `config.yml.bak`.

A lista completa, com cada opção explicada e o arquivo de exemplo, está em [docs/configuracao.md](docs/configuracao.md). As mensagens ficam em `plugins/AuroraLogin/lang/`, um arquivo por idioma (`pt_br.yml`, `en.yml`, `ru.yml`).

## Sobre os dados do seu servidor

- Senha vira hash PBKDF2-SHA256 com 600.000 iterações. Ela nunca fica em texto puro nem no banco, nem em log, e não existe comando que mostre a senha de alguém.
- A redefinição administrativa cria uma temporária de 12 caracteres, mostrada só para quem executou a ação, com clique para copiar. O jogador continua online.
- As contas ficam em `plugins/AuroraLogin/accounts.db`, SQLite local. Backup é copiar o arquivo com o servidor parado.
- A auditoria em `plugins/AuroraLogin/audit.log` registra o que a administração fez, sem tocar em senha.
- A checagem de conta original conversa só com o servidor de sessões oficial da Mojang, durante o login, e não guarda token nem chave; a confirmação vale apenas para aquela conexão. Não tem opção para redirecionar essa checagem para outro endereço.
- Nada do que acontece no seu servidor sai para fora, exceto essa consulta pontual à Mojang. Não há site, telemetria ou serviço de terceiros.

## Compatibilidade

Testado em Paper 1.20.1 e Paper 1.21.1, sem nenhum outro plugin instalado.

A verificação de conta original em servidor offline acompanha o pipeline de rede do próprio servidor, o que a torna sensível à estrutura interna da versão. Se uma versão futura mudar demais e o plugin não reconhecer, ele avisa no console, deixa a verificação desligada e o login por senha continua exatamente como está, sem travar a entrada de ninguém. Nesse caso, é esperar a atualização do plugin.

## Changelog

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

Achou um bug, tem uma sugestão ou quer entender um comportamento estranho antes de assumir que é o plugin? Abra uma issue neste repositório, de preferência com a versão do Paper, a versão do plugin e o trecho do console onde deu problema.

Site: https://auroraplugins.duckdns.org

Feito por Kat.
