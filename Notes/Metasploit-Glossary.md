# Glossário Completo do Metasploit Framework

Este documento serve como uma referência técnica detalhada dos comandos, arquitetura e fluxos de trabalho essenciais dentro do Metasploit Framework (MSF) para testes de intrusão e segurança ofensiva.

---

## 1. Comandos de Navegação e Exploração Básica

Estes são os comandos fundamentais que usas para te mover dentro da consola (`msfconsole`) e preparar o ambiente de ataque.

### Procurar Módulos (`search`)
* **Comando Exemplo:** `search eternalblue` ou `search type:exploit platform:windows`
* **O que faz:** Varre a base de dados local do Metasploit à procura de exploits, scanners ou ferramentas auxiliares relacionadas com a palavra-chave que escreveste. Podes filtrar por tipo, plataforma ou data.
* **Quando é utilizado:** É sempre o teu primeiro passo quando descobres um serviço vulnerável no Nmap (ex: Apache, SMB) e precisas de ver se o Metasploit já tem um exploit pronto para essa falha.

### Carregar um Módulo (`use`)
* **Comando Exemplo:** `use exploit/windows/smb/ms17_010_eternalblue`
* **O que faz:** Carrega o exploit ou scanner selecionado para a memória ativa do terminal, mudando o prompt do Metasploit (que fica vermelho) e abrindo o contexto para esse ataque.
* **Quando é utilizado:** Usas logo após teres encontrado o módulo certo com o comando `search` e estares pronto para o configurar.

### Ver Configurações e Requisitos (`show options`)
* **Comando Exemplo:** `show options`
* **O que faz:** Mostra todas as variáveis que o exploit precisa para funcionar (como IPs, portas, caminhos) e indica quais são obrigatórias (`Required: true`).
* **Quando é utilizado:** Obrigatório usar assim que entras num módulo. Serve para saberes exatamente que informações tens de preencher antes de disparar o ataque, evitando erros.

### Definir Variáveis (`set` / `setg`)
* **Comando Exemplo:** `set RHOSTS 10.10.10.10` ou `setg LHOST 10.10.14.2`
* **O que faz:** O `set` define o valor de uma variável específica para o módulo atual. O `setg` (Set Global) define o valor para todos os módulos que carregares a seguir durante essa sessão.
* **Quando é utilizado:** Usas para preencher os dados que viste no `show options`. O `RHOSTS` é quase sempre o IP da tua vítima (alvo) e o `LHOST` é o teu próprio IP de atacante (o IP da tua máquina do TryHackMe ou VPN).

### Disparar o Ataque (`exploit` / `run`)
* **Comando Exemplo:** `exploit` ou `run -j`
* **O que faz:** Executa o código do módulo. Se for um exploit, lança o ataque contra o alvo; se for um módulo auxiliar/scanner, inicia o varrimento. A flag `-j` corre o ataque em segundo plano (*job*).
* **Quando é utilizado:** É o passo final, executado apenas quando tens a certeza absoluta de que todas as opções obrigatórias do `show options` foram bem preenchidas.

---

## 2. Tipos de Módulos do Metasploit

O Metasploit está dividido em categorias de ferramentas. É crucial saber a diferença entre elas para não tentares disparar um scanner como se fosse um exploit.

* **Exploits:** Módulos que aproveitam uma vulnerabilidade de software para executar código não autorizado na máquina alvo. É o que te dá o acesso inicial.
* **Payloads:** O código que corre na máquina alvo *depois* que o exploit funciona (ex: abrir uma consola, criar um utilizador). É a "carga útil".
* **Auxiliary:** Scanners de portas, sniffers, ferramentas de enumeração e descobridores de vulnerabilidades. Não te dão uma consola de comando, mas dão-te informação vital.
* **Post:** Módulos de pós-exploração. Usados depois de já estares dentro da máquina para roubar passwords, recolher ficheiros ou saltar para outras máquinas da rede (*pivoting*).
* **Nops / Encoders:** Usados para modificar a assinatura do payload e tentar fazer com que o antivírus do alvo não detete o ataque.

---

## 3. Gestão de Payloads (Staged vs Unstaged)

Perceber a diferença entre a estrutura dos payloads é uma das perguntas favoritas dos engenheiros seniores em entrevistas de Red Team.

### Payloads Staged (Em Fases)
* **Como identificar no Metasploit:** Têm uma barra fixa no nome, ex: `windows/x64/meterpreter/reverse_tcp`
* **O que faz:** O exploit envia primeiro um código minúsculo (*stager*) apenas para abrir uma ligação na memória do alvo. Uma vez aberta, esse código descarrega o resto do payload pesado (*Meterpreter*) da tua máquina.
* **Quando é utilizado:** Usas quando o espaço de memória para explorar a vulnerabilidade é muito pequeno e não consegues injetar o código todo de uma vez.

### Payloads Unstaged (Inline / Passo Único)
* **Como identificar no Metasploit:** Têm um underscore no nome, ex: `windows/x64/meterpreter_reverse_tcp`
* **O que faz:** O exploit envia o payload completo de uma só vez para o alvo. Não há segundas descargas pela rede.
* **Quando é utilizado:** É mais estável e ideal para redes onde firewalls mais rígidas podem bloquear a segunda fase de download de um payload *staged*.

---

## 4. Comandos Essenciais do Meterpreter

O Meterpreter é o payload avançado do Metasploit que corre na memória do alvo, sendo muito difícil de detetar por antivírus comuns.

* `sysinfo` -> Mostra o sistema operativo, arquitetura (x64/x86) e nome da máquina da vítima.
* `getuid` -> Diz-te com que utilizador estás a correr os comandos (ex: se és um utilizador normal ou se já és Administrador/`NT AUTHORITY\SYSTEM`).
* `hashdump` -> Arranca com a extração das passwords encriptadas (hashes) do sistema Windows (ficheiro SAM).
* `upload` / `download` -> Permite mover ferramentas para dentro da máquina alvo ou roubar ficheiros confidenciais para a tua máquina de ataque.
* `shell` -> Transforma a sessão numa consola de comandos normal do sistema operativo alvo (CMD/Bash).
* `background` -> Coloca a sessão atual em segundo plano sem a fechar, permitindo-te voltar à consola principal do Metasploit para usar outras ferramentas.
