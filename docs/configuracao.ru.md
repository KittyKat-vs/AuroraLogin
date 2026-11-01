[Português](configuracao.md) · [English](configuracao.en.md) · **Русский**

# Конфигурация

`config.yml` лежит в `plugins/AuroraLogin/`. У каждой опции уже есть пояснение в самом файле, так что можно понять, за что отвечает поле, не открывая документацию.

Если вы предпочитаете нажимать, а не править текст, `/auroraauth panel` делает то же самое прямо в игре: любая настройка, сохранённая там, попадает в файл сразу — без перезапуска сервера. А поскольку меню переписывает только строку изменившегося значения, комментарии после этого остаются на месте.

Сервер, который уже работал со старым конфигом, обновляется сам при первом запуске после смены версии: плагин переносит ваши значения, сохраняет прежний файл как `config.yml.bak` и пишет новый, прокомментированный. Комментарии намеренно идут на португальском и без диакритики — так файл корректно открывается в любом редакторе, включая те, что сохраняют в ANSI. А вот текст, который видит игрок, как раз с ударениями, и он лежит в файлах папки `lang/`.

## Файл целиком

Одно замечание: блок ниже — это ровно тот файл, который плагин кладёт в папку, вместе с комментариями. Они остаются на португальском для всех языков, потому что шаблон конфига в jar один. Всё, что идёт после блока, — уже по-русски.

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

## Вход и аккаунт

`default-language` выбирает язык всего сервера: сообщения, меню и даже названия команд авторизации. При переходе на английский `/registrar` становится `/register`, а `/entrar` — `/login`; старые названия тут же перестают существовать.

`premium-auto-login` — это авто-вход лицензионных аккаунтов Minecraft: игрок заходит, плагин проверяет аккаунт и открывает дверь без регистрации и пароля. `premium-verification` — сама проверка, именно она делает авто-вход рабочим на сервере с `online-mode=false`; если у вас Velocity или BungeeCord проверяет сам, плагин это замечает и отключает проверку, но можно выключить её и руками. `floodgate-auto-login` — то же самое для заходящих через Bedrock, и действует только при установленном Floodgate.

`session-enabled` вместе с `session-minutes` разрешают игроку возвращаться по тому же IP, не вводя пароль заново, пока сессия не истекла. `timeout-seconds` — сколько времени у него на авторизацию после захода, прежде чем его отключат.

Четыре поля пароля (`password-min-length`, `password-max-length`, `password-deny-name`, `password-reuse`) задают политику: длина, запрет пароля совпадающего с ником и запрет возвращать прежний пароль при смене.

`teleport-on-move` возвращает игрока на точку захода, если он пошёл ходить до авторизации. `block-interactions` — это то, что не даёт трогать инвентарь, ломать блоки и использовать предметы в этот момент. А `pre-login-effects` накладывает слепоту, замедление и усталость шахты на того, кто ещё ждёт входа, чтобы человек понял, что его держат, а не что сервер завис; уровни от 1 до 5, и каждый эффект включается отдельно.

## Безопасность

Защита от подбора работает по IP и усиливается шагами. `max-attempts` — сколько неверных паролей помещается внутри `window-minutes`; как только лимит превышен, IP получает первый страйк и его кикает на `block-minutes`. Второй страйк кикает на `second-kick-minutes`. Третий даёт бан IP на `ban-hours`, пока включён `ip-ban`; если его выключить, третий раз превращается в очередной кик со вторым временем. `strike-reset-hours` — сколько времени IP должен пробыть без превышений, чтобы страйки обнулились.

Бан снимается сам по истечении `ban-hours`, и он не удаляет ни аккаунты, ни пароли: человек снова сможет играть, когда пройдёт время или когда вы его разбаните. Разбанить раньше — путь через **Безопасность → Забаненные IP** в меню: там список с числом попыток, датой бана, сроком, остатком и причиной по каждому IP.

`discord-invite` принимает ссылку на ваш Discord. Она показывается на экране бана рядом с IP и оставшимся временем, чтобы игроку было куда пойти. Оставьте `''`, чтобы не показывать кнопку вовсе.

## Интерфейс, меню и администрирование

`visual.titles`, `visual.sounds` и `visual.action-bar` отвечают за то, что появляется на экране: титр при заходе и при авторизации, звуки действий входа и экшн-бар с оставшимся временем. `action-bar-interval-ticks` — как часто обновляется эта строка, в тиках, где 20 тиков — одна секунда.

`panel.sounds` включает и выключает щелчок при навигации по меню. `admin.audit` пишет действия администраторов в `plugins/AuroraLogin/audit.log`: кто сбросил пароль, удалил аккаунт, отключил игрока или разбанил IP. Пароли туда не попадают никогда.

## Хранилище

`storage.file` — база SQLite внутри `plugins/AuroraLogin/`, где лежат аккаунты, сессии и забаненные IP. Для резервной копии достаточно скопировать этот файл на остановленном сервере.

## Файлы языков

Тексты сообщений — в `plugins/AuroraLogin/lang/`, по файлу на язык: `pt_br.yml`, `en.yml` и `ru.yml`. Правьте их как хотите, включая названия команд авторизации — они живут в ключах `command-register`, `command-login`, `command-logout` и `command-changepassword`. После правки выполните `/auroraauth reload`, чтобы применить.
