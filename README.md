# Exemplo de configuração docker-compose para Docker Registry, Registry UI, Jenkins, Kubernetes e pipeline CI/CD
#### 🚀 **Os serviços estarão disponíveis nos endereços:**
***Jenkins***: http://localhost:8085

***Docker Registry UI***: http://localhost:8086

Para implementar a solução, basta seguir as instruções abaixo.
## Geração de certificado, chave privada e arquivo para autenticação do Registry
- Criar os diretórios para o certificado e chave, e arquivo para autenticação:
	```sh
	mkdir auth certs
	```
- Gerar arquivo **htpasswd** com usuário e senha:
	```sh
	docker run --rm -ti xmartlabs/htpasswd <username> <password> > auth/htpasswd
	```
- Gerar o certificado e chave privada:
	```sh
	openssl req \
	-x509 -newkey rsa:4096 -days 365 -config openssl.conf \
	-keyout certs/domain.key -out certs/domain.crt
	```
## Iniciar Kubernetes (minikube)
- Testes realizados com a versão 1.20.2 - Iniciar o minikube com o comando abaixo:
	```sh
	minikube start --embed-certs --insecure-registry "192.168.1.0/24" --kubernetes-version v1.20.2
	```
## Configuração Kubernetes
- Criar Secret (usuário e senha informados na geração do arquivo htpasswd, será utilizado para autenticar acesso ao Registry):
	```sh
	kubectl create secret docker-registry registrycredentials --docker-server 192.168.1.92:5000 --docker-username <username> --docker-password <password>
	```
## Iniciar os containers
- Utilizando Docker-compose, inicie os containers com o comando:
	```sh
	docker-compose up
	```
## Configuração Jenkins
- Instalar plugins Docker, Docker Pipeline, Kubernetes e Kubernetes Continuous Deploy;
>Gerenciar Jenkins – Gerenciar plugins
- Criar as credencias para acesso ao Github, Registry e Kubernetes;
>Gerenciar Jenkins – Manage Credentials – Jenkins - Global credentials (unrestricted)
#### Github:
- Add Credentials – Username with password – informar Username e Password – *ID* Github – *Description* Github
#### Registry:
- Add Credentials – Username with password – informar Username e Password – *ID* Registry – *Description* Registry
#### Kubernetes (Secret File para uso na configuração do Kubernetes no Jenkins):
- Add Credentials – Secret file – escolher arquivo config do cluster Kubernetes – *ID* Kube – *Description* Kube
#### Kubernetes (conteúdo do arquivo config do cluster, será referenciado na pipeline Jenkinsfile):
- Add Credentials – Kubernetes configuration (kubeconfig) – *ID* kubeconfig – *Description* kubeconfig, *Enter directly* e colar o conteúdo do arquivo config.
### Configuração Kubernetes no Jenkins
> Gerenciar Jenkins - Configurar o sistema – Cloud (The cloud configuration has moved to a separate configuration page)
- Adicionar uma nova nuvem – Kubernetes – Kubernetes Cloud details…;
- Em *Credentials*, selecionar config (Kube). Teste a conexão;
- Jenkins URL - informar a url do Jenkins;
- Pod Templates… – Add Pod Template – *Name* (kube);
- Pod Template details… – *Labels* (kubepods);
- Containers – Add Container – Container Template;
- *Name* (jnlp) – *Docker image* (jenkins/jnlp-slave:latest) – *Working directory* (/home/jenkins) – marcar Allocate pseudo-TTY;
-  Save;
- Em Gerenciar Jenkins – Configurar segurança global – Agents - TCP port for inbound agents – selecionar Randômico;
- Salvar.
## *Pipeline* simples para deploy no Kubernetes
- No Jenkins, selecionar Novo job – Pipeline;
- Em *Build Triggers*, marcar *GitHub hook trigger for GITScm polling*;
- Em *Pipeline*, selecionar *Pipeline script from SCM* - *SCM Git* - *Repository URL* - informar o endereço do repositório no GitHub;
- Em *Credentials*, selecionar a credencial referente ao GitHub;
- Em *Branches to build* - *Branch Specifier (blank for 'any')*, informar a branche do repositório;
- Salvar
 ### *Webhook* no GitHub
 - No repositório do GitHub, ir em *Settings* - *Webhooks* - *Add webhook* - *Payload URL* - informar o endereço externo do servidor Jenkins (ex: http://ip-externo:8085/github-webhook/);
 - *Content Type* - selecionar application/json;
 - Add webhook..
