# Configuração

O `config.yml` fica em `plugins/AuroraLogin/`. Toda opção já vem com uma explicação no próprio arquivo, então dá para entender o que cada campo faz sem precisar abrir documentação.

Se você prefere clicar em vez de editar texto, `/auroraauth panel` faz a mesma coisa por dentro do jogo: cada ajuste salvo ali vai para o arquivo na hora, sem reiniciar o servidor. E como o painel reescreve apenas a linha do valor que mudou, os comentários continuam todos lá depois.

Um servidor que já estava rodando com uma configuração antiga é atualizado sozinho na primeira inicialização depois da troca de versão: o plugin reaproveita os seus valores, grava o arquivo anterior como `config.yml.bak` e escreve a versão nova, comentada. Os comentários são escritos sem acento de propósito, para o arquivo abrir certo em qualquer editor, inclusive os que salvam em ANSI. O texto que o jogador vê, esse sim acentuado, fica nos arquivos da pasta `lang/`.

## O arquivo inteiro

```yaml
# ============================================================
#  AuroraLogin - arquivo de configuracao
# ============================================================
#
#  Tudo que o plugin faz no seu servidor esta aqui. Da para
#  editar na mao ou abrir o painel no jogo com /auroraauth panel
#  e ir clicando nas opcoes.
#
#  Editou este arquivo? Rode /auroraauth reload. Nao precisa
#  reiniciar o servidor.
#
#  Quando o painel muda uma opcao, o plugin reescreve apenas a
#  linha daquele valor. Entao os comentarios e a ordem das secoes
#  continuam aqui depois.
#
#  Os comentarios estao sem acento de proposito: assim o arquivo
#  abre certo em qualquer editor, inclusive os que salvam em
#  ANSI. O texto das mensagens do jogo, esses sim acentuados,
#  fica nos arquivos da pasta lang/.
#
#  Site, suporte e explicacao detalhada de cada opcao:
#  https://auroraplugins.duckdns.org
# ============================================================

# Versao deste formato. Serve para o plugin perceber, depois de
# uma atualizacao, que o arquivo antigo precisa ganhar os
# comentarios novos. Deixe como esta.
config-version: 1

# ------------------------------------------------------------
#  Login e conta
# ------------------------------------------------------------
settings:
  # Idioma do servidor inteiro: mensagens, painel e ate os nomes
  # dos comandos de login. Ao trocar para en, /registrar vira
  # /register e /entrar vira /login. Opcoes: pt_br, en, ru.
  default-language: pt_br

  # Conta original do Minecraft entra sozinha, sem registrar e
  # sem digitar senha. Desligue se quiser senha para todo mundo,
  # inclusive para quem tem conta paga.
  premium-auto-login: true

  # Com isso ligado o plugin confere a conta na Mojang durante o
  # login, e e isso que faz a entrada automatica funcionar em
  # servidor com online-mode=false. Normal nem precisar mexer:
  # com proxy fazendo a checagem, ele se desliga sozinho.
  premium-verification: true

  # So tem efeito com o Floodgate instalado. Com ele, o jogador
  # de celular, console e Windows entra sem senha. Sem Floodgate,
  # esta opcao fica paradinha e o bedrock registra com senha.
  floodgate-auto-login: true

  # Sessao por IP: quem ja logou volta pelo mesmo IP e entra
  # direto, sem senha de novo. Otimo para o jogador, ruim se o
  # seu servidor roda em lan house, onde todo mundo divide o
  # mesmo IP.
  session-enabled: true

  # Quanto tempo a sessao vale, em minutos. Passou, pede a senha
  # de novo.
  session-minutes: 30

  # Tempo, em segundos, que o jogador tem para se autenticar
  # depois de entrar. Estourou, ele e desconectado.
  timeout-seconds: 60

  # Tamanho da senha, valendo para o registro e para a troca.
  password-min-length: 6
  password-max-length: 64

  # Nao deixa usar o proprio nome como senha, que e o primeiro
  # chute de quem tenta entrar na conta alheia.
  password-deny-name: true

  # Na troca de senha, nao deixa voltar para a senha que estava
  # la antes.
  password-reuse: true

  # O jogador andou antes de logar? Ele volta para o ponto onde
  # entrou. Desligue se preferir deixar ele parado onde esta.
  teleport-on-move: true

  # Antes de autenticar, o jogador nao abre inventario, nao
  # quebra nem coloca bloco, nao usa item nem entidade. E a
  # protecao para quem entra numa conta alheia e ja sair levando
  # o baul.
  block-interactions: true

  # Efeitos em quem ainda nao se autenticou, para a pessoa
  # perceber que esta esperando login e nao achar que o servidor
  # travou. Cada um liga e desliga sozinho.
  pre-login-effects:
    # Chave geral: desligando isto, nenhum efeito e aplicado.
    enabled: true
    # Cega o jogador.
    blindness: true
    # Deixa ele andando devagar.
    slowness: true
    # Nivel da lentidao, de 1 a 5.
    slowness-level: 2
    # Demora mais para quebrar bloco.
    mining-fatigue: true
    # Nivel da fadiga de mineracao, de 1 a 5.
    mining-fatigue-level: 1

# ------------------------------------------------------------
#  Seguranca
# ------------------------------------------------------------
security:
  # Protecao contra quem fica chutando senha uma por uma. O
  # bloqueio e por IP e cresce a cada vez que o limite estoura:
  # primeira vez leva um kick curto, segunda um kick mais longo,
  # terceira banimento de IP.
  brute-force:
    # Chave geral da protecao.
    enabled: true
    # Quantas senhas erradas o IP pode mandar, dentro da janela
    # abaixo, antes de levar o primeiro strike.
    max-attempts: 5
    # Janela de contagem, em minutos.
    window-minutes: 5
    # Tempo do kick no primeiro excesso, em minutos.
    block-minutes: 5
    # Tempo do kick no segundo excesso, em minutos.
    second-kick-minutes: 15
    # Duracao do ban de IP no terceiro excesso, em horas. O ban
    # cai sozinho nesse prazo, sem ninguem precisar desbanir.
    ban-hours: 24
    # Passando deste tempo sem nenhum excesso, os strikes do IP
    # voltam a zero. Em horas (168 = uma semana).
    strike-reset-hours: 168
    # Desligado, a terceira vez vira apenas mais um kick com o
    # segundo tempo, em vez de banir o IP.
    ip-ban: true
    # Link do convite do Discord, mostrado na tela de banimento
    # para o jogador poder recorrer. Deixe vazio ('') para nao
    # mostrar botao nenhum.
    discord-invite: ''

# ------------------------------------------------------------
#  Interface
# ------------------------------------------------------------
visual:
  # Titulo na tela quando o jogador entra e quando autentica.
  titles: true
  # Som nas acoes de registro, login e troca de senha.
  sounds: true
  # Barra de acao com o tempo que falta para autenticar.
  action-bar: true
  # De quanto em quanto tempo a barra e atualizada, em ticks
  # (20 ticks = 1 segundo). 40 significa a cada 2 segundos.
  action-bar-interval-ticks: 40

# ------------------------------------------------------------
#  Painel
# ------------------------------------------------------------
panel:
  # Som de clique ao navegar pelo /auroraauth panel.
  sounds: true

# ------------------------------------------------------------
#  Administracao
# ------------------------------------------------------------
admin:
  # Grava em audit.log o que a administracao fez: quem redefiniu
  # senha, removeu conta, desconectou ou desbanou um IP. Senha
  # nunca aparece ali.
  audit: true

# ------------------------------------------------------------
#  Armazenamento
# ------------------------------------------------------------
storage:
  # Banco SQLite dentro de plugins/AuroraLogin/, onde ficam as
  # contas, as sessoes e os IPs banidos. Para fazer backup, copie
  # este arquivo com o servidor parado.
  file: accounts.db
```

## Login e conta

`default-language` escolhe o idioma do servidor inteiro: mensagens, painel e também os nomes dos comandos de autenticação. Ao trocar para inglês, `/registrar` vira `/register` e `/entrar` vira `/login`; os nomes antigos deixam de existir na hora.

`premium-auto-login` é a entrada automática de conta original do Minecraft. O jogador chega, o plugin confere a conta e abre a porta sem pedir registro nem senha. `premium-verification` é a checagem em si, a que faz isso funcionar em servidor com `online-mode=false`; se você usa Velocity ou BungeeCord fazendo essa verificação, o plugin percebe e se desliga sozinho, mas dá para desligar por aí também. `floodgate-auto-login` é a mesma ideia para quem entra pelo Bedrock, e só vale quando o Floodgate está instalado.

`session-enabled` com `session-minutes` deixa o jogador voltar pelo mesmo IP sem digitar a senha de novo até a sessão expirar. `timeout-seconds` é quanto tempo ele tem para se autenticar depois de entrar antes de ser desconectado.

Os quatro campos de senha (`password-min-length`, `password-max-length`, `password-deny-name`, `password-reuse`) montam a política: tamanho, proibir a senha igual ao nome e não deixar voltar para a senha anterior na troca.

`teleport-on-move` devolve o jogador para o ponto de entrada se ele andar antes de autenticar. `block-interactions` é o que impede mexer em inventário, quebrar bloco ou usar item nesse intervalo. E `pre-login-effects` aplica cegueira, lentidão e fadiga de mineração em quem ainda está esperando login, para a pessoa entender que está bloqueada e não achar que o servidor travou; os níveis vão de 1 a 5 e cada efeito liga ou desliga por conta própria.

## Segurança

A proteção contra força bruta funciona por IP e é progressiva. `max-attempts` é quantas senhas erradas cabem dentro de `window-minutes`; estourou, o IP leva o primeiro strike e é expulso por `block-minutes`. O segundo strike expulsa por `second-kick-minutes`. O terceiro aplica o ban de IP por `ban-hours`, desde que `ip-ban` esteja ligado; desligado, a terceira vez é só mais um kick com o segundo tempo. `strike-reset-hours` é quanto tempo o IP precisa ficar sem passar do limite para os strikes voltarem a zero.

O ban expira sozinho no prazo de `ban-hours`, e ele não apaga conta nem senha: a pessoa volta a poder jogar quando o tempo passar ou quando você desbanir. Para desbanir antes, o caminho é **Segurança → IPs banidos** no painel, onde aparece a lista com strikes, data do ban, expiração, tempo restante e o motivo de cada IP.

`discord-invite` recebe o link do seu Discord. Ele é mostrado na tela de banimento, junto com o IP e o tempo restante, para o jogador ter por onde recorrer. Deixe `''` para não mostrar botão nenhum.

## Interface, painel e administração

`visual.titles`, `visual.sounds` e `visual.action-bar` controlam o que aparece na tela: o título ao entrar e ao autenticar, os sons das ações de login e a barra de ação com o tempo que falta. `action-bar-interval-ticks` é a frequência dessa barra, em ticks, sendo 20 ticks um segundo.

`panel.sounds` liga ou desliga o clique ao navegar pelo painel. `admin.audit` grava em `plugins/AuroraLogin/audit.log` as ações da administração: quem redefiniu senha, removeu conta, desconectou jogador ou desbanou um IP. Senha nunca é escrita ali.

## Armazenamento

`storage.file` é o banco SQLite dentro de `plugins/AuroraLogin/`, onde ficam as contas, as sessões e os IPs banidos. Para backup, basta copiar esse arquivo com o servidor parado.

## Arquivos de idioma

As mensagens ficam em `plugins/AuroraLogin/lang/`, um arquivo por idioma: `pt_br.yml`, `en.yml` e `ru.yml`. Dá para editar o texto à vontade, inclusive os nomes dos comandos de autenticação, que ficam nas chaves `command-register`, `command-login`, `command-logout` e `command-changepassword`. Depois de editar, use `/auroraauth reload` para aplicar.
