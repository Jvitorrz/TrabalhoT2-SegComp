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

Iniciar o ambiente vulnerável: 
Foi definida a porta 3000 no hospedeiro.

Bash
docker run --rm -d --name juiceshop_t2 -p 3000:3000 bkimminich/juice-shop

Acesso à Aplicação: 

URL: http://localhost:3000

Porta: 3000

Credenciais: A aplicação não possui credenciais padrão expostas intencionalmente.

Como parar o ambiente vulnerável: 

Bash
docker stop juiceshop_t2

2. Descrição Completa dos Experimentos Realizados 

Ferramentas de Pentesting Utilizadas 


Proxy de Interceptação: Burp Suite Community Edition.

Navegador: Chromium (Embutido nativamente no Burp Suite).

Configurações Específicas 

O navegador embutido do Burp Suite foi utilizado para rotear o tráfego automaticamente através do proxy local 127.0.0.1:8080 , permitindo a interceptação, análise e modificação das requisições HTTP antes de serem enviadas ao servidor.
+1

Procedimento Experimental Geral 

Iniciar o Docker Desktop e o contêiner do Juice Shop.

Iniciar o Burp Suite Community e abrir a aba "Proxy" -> "Intercept".

Clicar em "Open Browser" dentro do Burp para abrir o navegador com o proxy já configurado.

Navegar até a URL da aplicação local.

3. Exploração de Vulnerabilidades 

3.1. 
Vulnerabilidade 1: SQL Injection (Authentication Bypass) 

Objetivo do ataque: Burlar o mecanismo de autenticação para acessar a conta do usuário administrador sem possuir a senha.


Passos realizados: 

No navegador do Burp, acessar a rota de login (/#/login).

Ativar a interceptação no Burp Suite (Intercept is on).

Preencher o campo de e-mail com o payload: ' or 1=1-- e qualquer caractere na senha.

Clicar em "Log in" e analisar a requisição retida no Burp.

Liberar a requisição (Forward) e observar o login bem-sucedido na interface.


Requisição Enviada (Interceptada): 

HTTP
POST /rest/user/login HTTP/1.1
Host: localhost:3001
Content-Type: application/json

{"email":"' or 1=1--","password":"123"}

Resposta Obtida: 

HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"authentication":{"umail":"admin@juice-sh.op","id":1,"token":"eyJhbGciOiJIUzI1..."}}

Evidência Visual: 

[COLE SEU PRINT AQUI - TELA DO BURP COM O PAYLOAD OU TELA LOGADA COMO ADMIN]


Interpretação do Resultado: 
O banco de dados recebeu a string diretamente na query. O caractere ' fechou a instrução esperada, o or 1=1 forçou uma condição verdadeira que retorna o primeiro usuário do banco (o admin), e o -- comentou o restante da query que validaria a senha.


Discussão sobre Mitigação: 
A vulnerabilidade pode ser evitada utilizando Prepared Statements (Consultas Parametrizadas) ou frameworks ORM modernos. Isso garante que o banco de dados trate a entrada do usuário estritamente como um dado, e não como código executável.
+1

3.2. 
Vulnerabilidade 2: DOM Cross-Site Scripting (XSS) 

Objetivo do ataque: Injetar e executar um script malicioso no navegador da vítima através da barra de pesquisa.


Passos realizados: 

Na interface principal, clicar no ícone de lupa (Search).

Inserir o payload de iframe no campo de busca: <iframe src="javascript:alert('XSS_T2.2')">

Pressionar Enter.


Requisição Enviada: 

HTTP
GET /rest/products/search?q=%3Ciframe%20src%3D%22javascript%3Aalert%BN%27XSS_T2.2%27%29%22%3E HTTP/1.1
Host: localhost:3001

Resposta Obtida: 

HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"status":"success","data":[]}

Evidência Visual: 

[COLE SEU PRINT AQUI - TELA DO NAVEGADOR MOSTRANDO A CAIXA DE ALERTA COM O SCRIPT EXECUTADO]


Interpretação do Resultado: 
O frontend da aplicação recebe o parâmetro de busca e o renderiza dinamicamente na página HTML sem realizar a devida codificação de saída. Isso faz com que o navegador interprete a tag <iframe> como um elemento HTML legítimo e execute o JavaScript.


Discussão sobre Mitigação: Deve-se implementar Output Encoding transformando caracteres especiais em entidades HTML seguras antes de renderizar dados fornecidos pelo usuário no navegador. Adicionalmente, implementar uma rigorosa Content Security Policy (CSP) mitigaria a execução de scripts não autorizados.
+1
