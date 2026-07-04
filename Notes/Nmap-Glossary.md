# 📖 Glossário Completo do Nmap

Este documento serve como uma referência técnica detalhada de todas as flags e parâmetros essenciais do Nmap utilizados em auditorias de segurança e testes de intrusão (Red Team).

---

## 1. Tipos de Varrimento (Scans) Técnicos

Esta secção cobre a forma como o Nmap comunica com as portas do sistema alvo ao nível dos protocolos de rede.

###  TCP SYN Scan (Varrimento Síncrono / Furtivo)
* **Flag:** `-sS`
* **O que faz:** É o scan padrão do Nmap. Envia um pacote com a flag SYN (pedido de ligação). Se a porta estiver aberta, o alvo responde com SYN/ACK. O Nmap responde imediatamente com um pacote RST (reset) para derrubar a ligação antes que ela se conclua. 
* **Vantagem:** É muito rápido e não conclui o "Three-Way Handshake" do TCP, o que faz com que não seja registado nos logs de serviços/aplicações do alvo.

###  TCP Connect Scan (Varrimento de Ligação Completa)
* **Flag:** `-sT`
* **O que faz:** Conclui a ligação TCP de 3 vias por completo (SYN -> SYN/ACK -> ACK) e fecha a ligação de seguida.
* **Vantagem:** É o método utilizado obrigatoriamente quando o utilizador que corre o Nmap não tem privilégios de Administrador (Root) na máquina atacante, ou quando a rede não suporta pacotes raw. É muito ruidoso nos logs do alvo.

###  UDP Scan (Varrimento de Protocolo UDP)
* **Flag:** `-sU`
* **O que faz:** Envia pacotes UDP para as portas do alvo. Se não houver resposta, o Nmap assume que a porta está aberta ou filtrada. Se receber uma mensagem ICMP de "Port Unreachable", a porta está fechada.
* **Vantagem:** Permite descobrir serviços críticos que não usam TCP, como DNS (porta 53), SNMP (porta 161) ou DHCP. É um scan extremamente lento porque depende de timeouts.

---

## 2. Técnicas Avançadas de Evasão (Manipulação de Flags)

Estes scans manipulam as regras do protocolo TCP para tentar contornar firewalls e Sistemas de Deteção de Intrusão (IDS) que bloqueiam os scans tradicionais.

###  Null Scan (Varrimento Nulo)
* **Flag:** `-sN`
* **O que faz:** Envia um pacote TCP completamente limpo, ou seja, com todas as flags (SYN, ACK, FIN, etc.) desativadas (a zero).
* **Vantagem:** Segundo as regras do protocolo TCP (RFC 793), se a porta estiver fechada, o sistema deve responder com um RST. Se estiver aberta, o pacote é ignorado. Ajuda a passar por firewalls simples que só monitorizam ligações iniciadas por pacotes SYN.

###  FIN Scan (Varrimento de Finalização)
* **Flag:** `-sF`
* **O que faz:** Envia um pacote com a flag FIN ativada (usada normalmente para encerrar uma ligação legítima).
* **Vantagem:** Funciona da mesma forma que o Null Scan: portas fechadas respondem com RST e portas abertas ignoram. Útil quando a firewall bloqueia pacotes SYN e Null.

###  Xmas Scan (Varrimento Árvore de Natal)
* **Flag:** `-sX`
* **O que faz:** Envia um pacote TCP com as flags FIN, PSH (Push) e URG (Urgent) ativas em simultâneo. O nome vem do facto de o pacote parecer "iluminado como uma árvore de Natal" no analisador de tráfego.
* **Vantagem:** Tenta explorar o comportamento de sistemas baseados em Unix/Linux para contornar regras rígidas de firewalls comerciais.

###  ACK Scan (Mapeamento de Firewall)
* **Flag:** `-sA`
* **O que faz:** Envia pacotes com a flag ACK activa. Ao contrário dos outros, este scan nunca diz se uma porta está aberta ou fechada.
* **Vantagem:** Serve puramente para analisar a firewall do alvo. Se a resposta for um RST, significa que a porta não está a ser filtrada pela firewall. Se o pacote for bloqueado ou ignorado, a porta está "Filtrada".

---

## 3. Controlo de Tempo e Desempenho (Timing)

Flags que controlam a velocidade do envio de pacotes. Ajustar o timing é essencial para balancear eficácia, estabilidade e ocultação.

* `-T0` (Paranoico) -> Envia um pacote de cada vez com intervalos de até 5 minutos. Usado exclusivamente para evitar alarmes em sistemas IDS/IPS altamente sensíveis.
* `-T1` (Furtivo) -> Envia pacotes com intervalos de 15 segundos. Muito lento, usado para evadir deteções de varrimento automático.
* `-T2` (Polido) -> Reduz a velocidade para consumir menos largura de banda e evitar deitar abaixo sistemas extremamente antigos ou frágeis do cliente.
* `-T3` (Normal) -> O perfil padrão do Nmap. Equilibrado e seguro para a maioria das redes.
* `-T4` (Agressivo) -> Acelera o scan assumindo que estás numa rede moderna, estável e rápida. É o padrão utilizado em laboratórios do TryHackMe e no dia a dia da indústria para poupar tempo.
* `-T5` (Insano) -> Velocidade máxima absoluta. Sacrifica a precisão do scan (pode perder portas abertas por causa de timeouts de rede) e faz disparar todos os alarmes de segurança.

---

## 4. Seleção e Filtragem de Alvos (Portas e IPs)

Flags utilizadas para definir o alcance exato do varrimento, otimizando o tempo de execução.

* `-Pn` -> Ignora a verificação de Ping (ICMP) inicial. O Nmap assume que o alvo está ativo e faz o scan diretamente. Essencial quando o alvo está atrás de uma firewall que descarta pings.
* `-p-` -> Ordena ao Nmap para fazer o varrimento de todas as 65.535 portas TCP existentes, em vez de focar apenas nas 1000 portas mais comuns (padrão).
* `-p [portas]` -> Define um alvo estrito. Pode ser uma porta única (`-p 80`), uma lista separada por vírgulas (`-p 22,80,443`) ou um intervalo (`-p 1-1024`).
* `-F` (Fast) -> Varre apenas as 100 portas estatisticamente mais comuns da internet. Reduz o tempo do comando para escassos segundos.
* `--top-ports [número]` -> Permite-te escolher exatamente quantas portas mais comuns queres varrer (ex: `--top-ports 500`).

---

## 5. Inspeção de Serviços e Inteligência (Deteção)

Flags usadas para recolher informações profundas sobre o que está a correr nas portas encontradas.

* `-sV` -> Deteção de Versão. Interroga a porta aberta para extrair o banner e a versão exata do software (ex: Apache httpd 2.4.41). Crucial para pesquisar exploits públicos conhecidos (CVEs).
* `-sC` -> Executa os scripts padrão do NSE (Nmap Script Engine). Faz testes automáticos e seguros de vulnerabilidades conhecidas, configurações erradas ou recolha extra de metadados.
* `-O` -> Deteção de Sistema Operativo. Analisa a forma como a pilha de rede do alvo responde a pacotes específicos e compara com a base de dados do Nmap para adivinhar o SO (Windows, Linux, macOS, etc.).
* `-A´` (Modo Agressivo) -> Ativa o `-sV`, `-sC`, `-O` e o comando `traceroute` tudo ao mesmo tempo com uma única flag.

---

## 6. Formatos de Exportação (Output)

Flags para guardar os resultados do scan em ficheiros, algo obrigatório para a criação de relatórios em ambiente de trabalho real.

* `-oN [nome_ficheiro]` -> Formato Normal. Guarda o resultado exatamente como ele aparece no ecrã do teu terminal.
* `-oG [nome_ficheiro]` -> Formato Grepable. Guarda o output estruturado por linhas. Permite usar comandos de Linux (`grep`, `awk`, `cut`) para filtrar rapidamente listas gigantes de IPs e portas.
* `-oX [nome_ficheiro]` -> Formato XML. Exporta os dados estruturados para serem facilmente importados em ferramentas de terceiros (como a base de dados do Metasploit ou geradores de relatórios automáticos).
* `-oA [nome_ficheiro]` -> Exportação Total. Gera três ficheiros em simultâneo com o mesmo nome, um em cada formato acima (.nmap, .gnmap, e .xml).
