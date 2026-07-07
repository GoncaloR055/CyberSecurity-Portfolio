# OWASP Top 10 Cheatsheet & Glossary (TryHackMe Edition)

Este repositório serve como um guia rápido, conciso e prático para as 10 principais vulnerabilidades Web do **OWASP Top 10**, com foco em técnicas de teste (pentesting) e remediação baseadas nos laboratórios do TryHackMe.

---

## 📌 Tabela de Conteúdos
1. [Broken Access Control](#1-broken-access-control)
2. [Cryptographic Failures](#2-cryptographic-failures)
3. [Injection](#3-injection)
4. [Insecure Design](#4-insecure-design)
5. [Security Misconfiguration](#5-security-misconfiguration)
6. [Vulnerable and Outdated Components](#6-vulnerable-and-outdated-components)
7. [Identification and Authentication Failures](#7-identification-and-authentication-failures)
8. [Software and Data Integrity Failures](#8-software-and-data-integrity-failures)
9. [Security Logging and Monitoring Failures](#9-security-logging-and-monitoring-failures)
10. [Server-Side Request Forgery (SSRF)](#10-server-side-request-forgery-ssrf)

---

### 1. Broken Access Control
* **O que é:** O sistema falha em validar se o utilizador autenticado tem permissão real para ver um dado recurso ou executar uma ação.
* **Como Testar:**
  * Altere IDs numéricos ou lógicos em URLs ou parâmetros (ex: mudar `://site.com/api/user?id=10` para `id=11`).
  * Tente aceder a caminhos administrativos sem privilégios (ex: `/admin`, `/panel`).
  * Force a alteração de métodos HTTP (ex: mudar pedido `GET` para `POST` ou `DELETE`).
* **Como Prevenir:**
  * Adote o princípio do menor privilégio por padrão.
  * Valide as permissões de acesso no servidor em **todas** as requisições, nunca confiando apenas no lado do cliente.

### 2. Cryptographic Failures
* **O que é:** Proteção inadequada de dados sensíveis em trânsito ou em repouso, expondo segredos através de falta de cifragem ou algoritmos fracos.
* **Como Testar:**
  * Intercepte o tráfego HTTP para validar a transmissão de credenciais em texto limpo.
  * Identifique o uso de hashes obsoletos (ex: MD5, SHA-1) em cookies ou bases de dados vazadas.
  * Verifique certificados TLS inválidos, expirados ou configurados com cifras antigas.
* **Como Prevenir:**
  * Force o uso de HTTPS com TLS atualizado através de políticas HSTS.
  * Use algoritmos de cifragem robustos (ex: AES-256) e funções de hashing seguras para passwords (ex: bcrypt, Argon2).

### 3. Injection (SQLi, Command Injection, XSS)
* **O que é:** Dados fornecidos pelo utilizador são interpretados pelo interpretador do sistema como comandos de código ou consultas estruturadas.
* **Como Testar:**
  * Insira caracteres especiais como `'`, `"`, `--`, ou `#` em campos de texto para quebrar consultas SQL.
  * Injete operadores de sistema (ex: `;`, `&&`, `|`) seguidos de comandos em funções de upload/pesquisa (ex: `; whoami`).
  * Submeta scripts maliciosos (ex: `<script>alert(1)</script>`) em inputs para testar Cross-Site Scripting.
* **Como Prevenir:**
  * Utilize consultas parametrizadas (Prepared Statements) para interações com bases de dados.
  * Implemente validação estrita baseada em listas de permissão (allowlists) e sanitize todos os dados de entrada.

### 4. Insecure Design
* **O que é:** Falhas originadas na arquitetura e no planeamento do sistema, onde o fluxo lógico do negócio foi desenhado de forma inerentemente insegura.
* **Como Testar:**
  * Explore o fluxo de submissão de formulários (ex: saltar a etapa de pagamento e ir direto para a página de sucesso).
  * Abuse de perguntas de recuperação de password cujas respostas podem ser facilmente adivinhadas ou automatizadas.
* **Como Prevenir:**
  * Implemente a Modelação de Ameaças (Threat Modeling) nas fases iniciais do ciclo de desenvolvimento (SDLC).
  * Use padrões de design seguros estabelecidos e bibliotecas de componentes de autenticação validadas.

### 5. Security Misconfiguration
* **O que é:** Falta de endurecimento (hardening) do servidor ou da aplicação, mantendo definições padrão, serviços desnecessários ativos ou erros detalhados expostos.
* **Como Testar:**
  * Tente aceder a painéis com credenciais de fábrica (ex: `admin:admin`, `root:root`).
  * Procure por listagem de diretórios ativa (ver os ficheiros do servidor no navegador) ou mensagens de erro com stack traces detalhados.
* **Como Prevenir:**
  * Desative funcionalidades, portas e serviços não utilizados.
  * Remova credenciais de fábrica e configure mensagens de erro genéricas para o utilizador final.

### 6. Vulnerable and Outdated Components
* **O que é:** Utilização de bibliotecas, frameworks, plugins ou sistemas operativos antigos que possuem vulnerabilidades públicas conhecidas (CVEs).
* **Como Testar:**
  * Analise o cabeçalho HTTP `Server` ou use ferramentas como Wappalyzer e Nmap para identificar versões de software.
  * Cruze as versões encontradas com bases de dados públicas de exploits (ex: Exploit-DB, CVE Details).
* **Como Prevenir:**
  * Mantenha um inventário contínuo e automatizado de dependências (ex: Software Bill of Materials - SBOM).
  * Aplique patches e atualizações de segurança de forma regular e programada.

### 7. Identification and Authentication Failures
* **O que é:** Fragilidades nos mecanismos de autenticação e gestão de sessões que permitem ataques de força bruta, personificação ou fixação de sessão.
* **Como Testar:**
  * Submeta ataques de dicionário (Brute Force) contra o painel de login sem sofrer bloqueio de IP ou conta.
  * Teste se o sistema aceita passwords extremamente fracas durante o registo (ex: `password123`).
* **Como Prevenir:**
  * Implemente obrigatoriamente a Autenticação Multi-Fator (MFA).
  * Adote políticas de bloqueio temporário de conta (rate limiting) e requisitos rígidos de complexidade para passwords.

### 8. Software and Data Integrity Failures
* **O que é:** Aceitação de código, atualizações ou objetos serializados vindos de fontes externas sem que a sua origem e integridade sejam validadas.
* **Como Testar:**
  * Modifique o conteúdo de objetos serializados guardados em cookies para tentar escalar privilégios (Insecure Deserialization).
  * Altere ficheiros de configuração ou atualizações a meio do trânsito para verificar se o sistema os executa cegamente.
* **Como Prevenir:**
  * Use assinaturas digitais ou hashes criptográficos para validar a integridade de ficheiros e atualizações antes da execução.
  * Evite passar objetos serializados diretamente do cliente para o servidor; use formatos de dados puros como JSON estruturado.

### 9. Security Logging and Monitoring Failures
* **O que é:** Ausência ou ineficácia no registo de eventos críticos de segurança, impedindo a deteção precoce, triagem e resposta a incidentes e ataques em curso.
* **Como Testar:**
  * Execute múltiplos ataques deliberados (ex: 100 logins falhados ou injeções SQL visíveis) e verifique se as ações são omitidas nos logs do servidor.
  * Teste se atividades anómalas geram alertas imediatos para a equipa de segurança.
* **Como Prevenir:**
  * Registe de forma centralizada todas as falhas de autenticação, acessos negados e erros graves de validação.
  * Garanta que os logs são armazenados num formato legível por sistemas SIEM para monitorização e alertas em tempo real.

### 10. Server-Side Request Forgery (SSRF)
* **O que é:** O servidor Web aceita um URL fornecido pelo utilizador e faz um pedido HTTP em seu nome, permitindo ao atacante interagir com a rede interna do servidor.
* **Como Testar:**
  * Modifique parâmetros que aceitam links (ex: `?image=http://...`) para apontar para a interface local (`http://127.0.0.1:80`) ou para portas internas.
  * Tente aceder aos endpoints de metadados de ambientes Cloud (ex: AWS/GCP: `http://169.254.169.254`).
* **Como Prevenir:**
  * Implemente uma lista estrita de domínios permitidos (allowlist) para requisições externas.
  * Bloqueie a nível de rede a capacidade do servidor Web efetuar pedidos direcionados à rede interna ou a endereços de loopback.
