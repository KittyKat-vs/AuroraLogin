[Português](configuracao.md) · **English** · [Русский](configuracao.ru.md)

# Configuration

`config.yml` sits in `plugins/AuroraLogin/`. Every option already carries an explanation inside the file itself, so you can work out what a field does without opening any documentation.

If you would rather click than edit text, `/auroraauth panel` does the same thing from inside the game: anything you save there goes into the file immediately, without restarting the server. And because the panel rewrites only the line holding the value that changed, all the comments are still there afterwards.

A server that was already running with an old configuration is updated on its own the first time it boots after you change version: the plugin reuses your values, writes the previous file as `config.yml.bak` and puts the new commented one in place. The comments are written without accents on purpose, so the file opens correctly in any editor, including the ones that save as ANSI. The text the player actually sees, which does carry accents, lives in the files under `lang/`.

## The whole file

One thing this page cannot do for you: the block below is exactly the file the plugin ships, comments included, and those stay in Portuguese for every language, because there is a single config template inside the jar. Everything after the block is in English.

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

## Login and account

`default-language` picks the language of the whole server: messages, panel and even the names of the authentication commands. Switching to English turns `/registrar` into `/register` and `/entrar` into `/login`; the old names stop existing immediately.

`premium-auto-login` is the automatic entry for paid Minecraft accounts. The player arrives, the plugin checks the account and opens the door without asking for a registration or a password. `premium-verification` is the check itself, the thing that makes that work on a server with `online-mode=false`; if you run Velocity or BungeeCord doing that verification, the plugin notices and switches itself off, though you can turn it off by hand too. `floodgate-auto-login` is the same idea for anyone joining over Bedrock, and it only counts when Floodgate is installed.

`session-enabled` together with `session-minutes` lets a player come back over the same IP without retyping the password until the session expires. `timeout-seconds` is how long they have to authenticate after joining before they get disconnected.

The four password fields (`password-min-length`, `password-max-length`, `password-deny-name`, `password-reuse`) build the policy: length, not allowing a password equal to the name, and not letting someone go back to the password they had before when changing it.

`teleport-on-move` returns the player to the spot where they joined if they walk before authenticating. `block-interactions` is what stops inventory, breaking blocks and item use during that window. And `pre-login-effects` applies blindness, slowness and mining fatigue to anyone still waiting on login, so they understand they are being held rather than thinking the server froze; the levels run from 1 to 5, and each effect turns on and off on its own.

## Security

The brute-force protection works per IP and escalates. `max-attempts` is how many wrong passwords fit inside `window-minutes`; once it overflows, the IP takes its first strike and is kicked for `block-minutes`. The second strike kicks for `second-kick-minutes`. The third applies the IP ban for `ban-hours`, as long as `ip-ban` is on; with it off, the third time is just another kick at the second duration. `strike-reset-hours` is how long an IP has to stay under the limit for its strikes to return to zero.

The ban expires on its own after `ban-hours`, and it deletes neither accounts nor passwords: the person can play again when the time passes or when you unban them. To unban early, the path is **Security → Banned IPs** in the panel, where the list shows strikes, ban date, expiry, time left and the reason for each IP.

`discord-invite` takes the link to your Discord. It is shown on the ban screen, next to the IP and the time left, so the player has somewhere to go. Leave it `''` to show no button at all.

## Interface, panel and administration

`visual.titles`, `visual.sounds` and `visual.action-bar` control what appears on screen: the title on join and on authenticating, the sounds for the login actions, and the action bar with the time remaining. `action-bar-interval-ticks` is how often that bar is refreshed, in ticks, 20 ticks being one second.

`panel.sounds` turns the click on or off when navigating the panel. `admin.audit` writes the administration's actions to `plugins/AuroraLogin/audit.log`: who reset a password, removed an account, disconnected a player or unbanned an IP. A password is never written there.

## Storage

`storage.file` is the SQLite database inside `plugins/AuroraLogin/`, holding the accounts, the sessions and the banned IPs. For a backup, copy that file with the server stopped.

## Language files

The messages live in `plugins/AuroraLogin/lang/`, one file per language: `pt_br.yml`, `en.yml` and `ru.yml`. You can edit the text as much as you like, including the authentication command names, which sit in the keys `command-register`, `command-login`, `command-logout` and `command-changepassword`. After editing, run `/auroraauth reload` to apply.
