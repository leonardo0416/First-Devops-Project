📋 Índice
Visão Geral
Pré-requisitos
Arquitetura do Projeto
Fase 1: Preparação do Ambiente Local
Fase 2: Containerização com Docker
Fase 3: Teste Local do Container
Fase 4: Configuração do Amazon ECR
Fase 5: Push da Imagem para o ECR
Fase 6: Provisionamento da Instância EC2
Fase 7: Deploy na EC2
Verificação e Testes
Troubleshooting
Limpeza de Recursos
🎯 Visão Geral
O que vamos construir?
Neste laboratório, você aprenderá a containerizar um website estático (HTML, CSS e JavaScript) usando Docker e implantá-lo manualmente em uma instância EC2 na AWS, utilizando o Amazon ECR (Elastic Container Registry) para gerenciamento de imagens.

Por que isso é importante?
Portabilidade: Seu site funcionará da mesma forma em qualquer ambiente
Isolamento: Elimina problemas de "funciona na minha máquina"
Escalabilidade: Base para futuras implementações mais complexas
Padrão da Indústria: Docker é amplamente utilizado no mercado
Tempo estimado: 2-3 horas
🔧 Pré-requisitos
Ferramentas Necessárias
1. Docker Desktop
Windows/Mac: Baixe em docker.com/products/docker-desktop
Linux: Instale via terminal:
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
Para verificar a instalação:

docker --version
[Espaço para print: Resultado do comando docker --version]

2. AWS CLI
Instale seguindo a documentação oficial

Para verificar:

aws --version
[Espaço para print: Resultado do comando aws --version]

3. Conta AWS
Crie uma conta gratuita em aws.amazon.com
⚠️ Importante: Alguns recursos podem gerar custos. Use o Free Tier quando possível
4. Editor de Código
Recomendado: Visual Studio Code
Extensões úteis: Docker, AWS Toolkit
Estrutura do Projeto
meu-projeto/
├── website/
│   ├── index.html
│   ├── styles.css
│   ├── script.js
│   └── assets/
│       └── (imagens, fontes, etc.)
└── Dockerfile (vamos criar)
🏗️ Arquitetura do Projeto
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Código Local   │────▶│   Docker Image  │────▶│    Amazon ECR   │
│  (HTML/CSS/JS)  │     │   (Container)   │     │   (Registry)    │
└─────────────────┘     └─────────────────┘     └─────────────────┘
                                                          │
                                                          ▼
                        ┌─────────────────┐     ┌─────────────────┐
                        │    Browser      │◀────│    Amazon EC2   │
                        │  (User Access)  │     │   (Container)   │
                        └─────────────────┘     └─────────────────┘
📦 Fase 1: Preparação do Ambiente Local
Passo 1.1: Verificar estrutura do projeto
Navegue até o diretório do seu projeto:

cd caminho/para/seu/projeto
ls -la
Você deve ver a pasta website/ com seus arquivos:

ls -la website/
[Espaço para print: Estrutura de arquivos do projeto]

Passo 1.2: Testar o website localmente (opcional)
Você pode abrir o index.html diretamente no navegador para verificar se está funcionando:

# No Mac
open website/index.html

# No Linux
xdg-open website/index.html

# No Windows (PowerShell)
start website/index.html
[Espaço para print: Website funcionando no navegador]

🐳 Fase 2: Containerização com Docker
Passo 2.1: Criar o Dockerfile
Na raiz do projeto (mesmo nível da pasta website/), crie um arquivo chamado Dockerfile:

touch Dockerfile
Passo 2.2: Escrever o Dockerfile
Abra o Dockerfile no seu editor e adicione:

# Imagem base - Nginx Alpine (leve e eficiente)
FROM nginx:alpine

# Copia os arquivos do website para o diretório do Nginx
COPY website/ /usr/share/nginx/html/

# Expõe a porta 80 (documentação - não abre a porta realmente)
EXPOSE 80

# Comando padrão quando o container iniciar
CMD ["nginx", "-g", "daemon off;"]
🎓 Entendendo cada linha:
FROM nginx:alpine: Define a imagem base. Alpine é uma versão Linux super leve
COPY: Copia arquivos do host para dentro da imagem
EXPOSE: Documenta qual porta o container usa
CMD: Define o comando padrão ao iniciar o container
[Espaço para print: Dockerfile criado no editor]

Passo 2.3: Construir a imagem Docker
No terminal, na raiz do projeto, execute:

docker build -t meu-website:v1.0 .
🎓 Entendendo o comando:
docker build: Comando para construir uma imagem
-t meu-website:v1.0: Tag (nome:versão) da imagem
.: Contexto de build (diretório atual)
Você verá a saída do processo de build:

[+] Building 10.5s (8/8) FINISHED
 => [internal] load build definition from Dockerfile
 => transferring dockerfile: 370B
 => [internal] load .dockerignore
 => [internal] load metadata for docker.io/library/nginx:alpine
 => [1/3] FROM docker.io/library/nginx:alpine
 => [2/3] RUN rm -rf /usr/share/nginx/html/*
 => [3/3] COPY website/ /usr/share/nginx/html/
 => exporting to image
 => naming to docker.io/library/meu-website:v1.0
[Espaço para print: Processo de build do Docker]

Passo 2.4: Verificar a imagem criada
docker images
Você deve ver sua imagem listada:

REPOSITORY     TAG       IMAGE ID       CREATED          SIZE
meu-website    v1.0      abc123def456   30 seconds ago   23.5MB
[Espaço para print: Lista de imagens Docker]

🧪 Fase 3: Teste Local do Container
Passo 3.1: Executar o container localmente
docker run -d -p 8080:80 --name meu-website-container meu-website:v1.0
🎓 Entendendo o comando:
docker run: Cria e executa um container
-d: Executa em background (detached)
-p 8080:80: Mapeia porta 8080 do host para porta 80 do container
--name: Nome do container
meu-website:v1.0: Imagem a ser usada
Passo 3.2: Verificar se o container está rodando
docker ps
Você verá algo como:

CONTAINER ID   IMAGE              COMMAND                  CREATED         STATUS         PORTS                  NAMES
xyz789abc123   meu-website:v1.0   "nginx -g 'daemon..."   10 seconds ago  Up 9 seconds   0.0.0.0:8080->80/tcp   meu-website-container
[Espaço para print: Container em execução]

Passo 3.3: Testar no navegador
Abra seu navegador e acesse:

http://localhost:8080
Você deve ver seu website funcionando! 🎉

[Espaço para print: Website rodando via Docker no localhost:8080]

Passo 3.4: Verificar logs do container (opcional)
docker logs meu-website-container
Passo 3.5: Parar e remover o container de teste
# Parar o container
docker stop meu-website-container

# Remover o container
docker rm meu-website-container
☁️ Fase 4: Configuração do Amazon ECR
Passo 4.1: Acessar o Console AWS
Acesse console.aws.amazon.com
Faça login com suas credenciais
[Espaço para print: Console AWS]

Passo 4.2: Navegar para o ECR
Na barra de busca superior, digite "ECR"
Clique em "Elastic Container Registry"
[Espaço para print: Busca pelo ECR]

Passo 4.3: Criar um repositório
Clique em "Create repository"
Configure:
Visibility settings: Private
Repository name: meu-website
Tag immutability: Disabled (padrão)
Scan on push: Enabled (recomendado para segurança)
Clique em "Create repository"
[Espaço para print: Formulário de criação do repositório]

Passo 4.4: Anotar a URI do repositório
Após criar, você verá algo como:

123456789012.dkr.ecr.us-east-1.amazonaws.com/meu-website
⚠️ Importante: Copie e guarde esta URI, você precisará dela!

[Espaço para print: Repositório criado com a URI visível]

📤 Fase 5: Push da Imagem para o ECR
Passo 5.1: Configurar AWS CLI
Se ainda não configurou, execute:

aws configure
Você precisará fornecer:

AWS Access Key ID: Obtida no IAM
AWS Secret Access Key: Obtida no IAM
Default region: ex: us-east-1
Default output format: json
Passo 5.2: Autenticar Docker com ECR
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com
⚠️ Substitua:

us-east-1 pela sua região
123456789012 pelo seu Account ID
Você deve ver:

Login Succeeded
[Espaço para print: Login bem-sucedido no ECR]

Passo 5.3: Tagar a imagem para o ECR
docker tag meu-website:v1.0 123456789012.dkr.ecr.us-east-1.amazonaws.com/meu-website:v1.0
Passo 5.4: Push da imagem
docker push 123456789012.dkr.ecr.us-east-1.amazonaws.com/meu-website:v1.0
Você verá o progresso do upload:

The push refers to repository [123456789012.dkr.ecr.us-east-1.amazonaws.com/meu-website]
abc123: Pushed
def456: Pushed
v1.0: digest: sha256:xyz789... size: 1234
[Espaço para print: Push concluído]

Passo 5.5: Verificar no Console AWS
Volte ao ECR no console AWS
Clique no seu repositório
Você deve ver a imagem com a tag v1.0
[Espaço para print: Imagem no ECR]

🖥️ Fase 6: Provisionamento da Instância EC2
Passo 6.1: Navegar para EC2
No console AWS, busque por "EC2"
Clique em "EC2"
Passo 6.2: Lançar instância
Clique em "Launch Instance"
Configure:
Nome e tags
Name: meu-website-server
Imagem de aplicação e sistema operacional
AMI: Amazon Linux 2023 (Free tier eligible)
[Espaço para print: Seleção da AMI]

Tipo de instância
Instance type: t2.micro (Free tier eligible)
[Espaço para print: Seleção do tipo de instância]

Par de chaves
Clique em "Create new key pair"
Key pair name: meu-website-key
Key pair type: RSA
Private key file format: .pem (Linux/Mac) ou .ppk (Windows/PuTTY)
Clique em "Create key pair" e salve o arquivo
⚠️ IMPORTANTE: Guarde este arquivo com segurança! Você precisará dele para acessar a EC2.

[Espaço para print: Criação do key pair]

Configurações de rede
VPC: Default
Subnet: No preference
Auto-assign public IP: Enable
Firewall (security groups): Create security group
Security group name: meu-website-sg
Description: Security group for website
Regras do Security Group
Adicione as seguintes regras:

Type	Protocol	Port Range	Source
SSH	TCP	22	My IP
HTTP	TCP	80	0.0.0.0/0
[Espaço para print: Configuração do Security Group]

Configurar armazenamento
Volume: 8 GiB gp3 (padrão)
Passo 6.3: Configurar IAM Role (Permissões para ECR)
Criar IAM Role
Em "Advanced details", encontre "IAM instance profile"
Clique em "Create new IAM profile"
Ou vá para IAM Console e:
Clique em "Roles" → "Create role"
Trusted entity: AWS service
Use case: EC2
Permissions: Adicione AmazonEC2ContainerRegistryReadOnly
Role name: EC2-ECR-Role
[Espaço para print: Criação do IAM Role]

Volte para a configuração da EC2 e selecione o role criado
Passo 6.4: Revisar e lançar
Revise todas as configurações
Clique em "Launch instance"
Aguarde a instância inicializar (status: running)
[Espaço para print: Instância EC2 rodando]

Passo 6.5: Anotar informações importantes
Anote:

Public IP: Ex: 54.123.45.67
Instance ID: Ex: i-0abc123def456789
🚀 Fase 7: Deploy na EC2
Passo 7.1: Conectar à instância EC2
No Linux/Mac:
# Ajustar permissões da chave
chmod 400 meu-website-key.pem

# Conectar via SSH
ssh -i meu-website-key.pem ec2-user@54.123.45.67
No Windows (usando PuTTY):
Converta a chave .pem para .ppk usando PuTTYgen
Use PuTTY para conectar com a chave .ppk
Você verá:

   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|
  ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
   ~~       V~' '->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/'
[ec2-user@ip-172-31-xx-xx ~]$
[Espaço para print: Conexão SSH estabelecida]

Passo 7.2: Instalar Docker na EC2
# Atualizar pacotes
sudo yum update -y

# Instalar Docker
sudo yum install docker -y

# Iniciar serviço Docker
sudo systemctl start docker

# Habilitar Docker no boot
sudo systemctl enable docker

# Adicionar ec2-user ao grupo docker
sudo usermod -a -G docker ec2-user

# Verificar instalação
docker --version
Passo 7.3: Fazer logout e login novamente
# Sair
exit

# Conectar novamente
ssh -i meu-website-key.pem ec2-user@54.123.45.67
Passo 7.4: Autenticar Docker com ECR na EC2
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com
[Espaço para print: Login ECR na EC2]

Passo 7.5: Pull da imagem do ECR
docker pull 123456789012.dkr.ecr.us-east-1.amazonaws.com/meu-website:v1.0
Você verá:

v1.0: Pulling from meu-website
Status: Downloaded newer image for 123456789012.dkr.ecr.us-east-1.amazonaws.com/meu-website:v1.0
[Espaço para print: Pull concluído]

Passo 7.6: Executar o container
docker run -d -p 80:80 --name meu-website-prod --restart always 123456789012.dkr.ecr.us-east-1.amazonaws.com/meu-website:v1.0
🎓 Parâmetros importantes:
--restart always: Reinicia o container se a EC2 reiniciar
-p 80:80: Mapeia porta 80 (padrão HTTP)
Passo 7.7: Verificar se está rodando
# Verificar container
docker ps

# Verificar logs
docker logs meu-website-prod
[Espaço para print: Container rodando na EC2]

✅ Verificação e Testes
Teste 1: Acessar pelo navegador
Abra seu navegador
Digite o IP público da EC2: http://54.123.45.67
Seu website deve aparecer! 🎉
[Espaço para print: Website funcionando na AWS]

Teste 2: Verificar logs na EC2
# Logs do container
docker logs -f meu-website-prod

# Status do container
docker stats meu-website-prod
Teste 3: Testar reinicialização
# Parar o container
docker stop meu-website-prod

# Verificar se parou
docker ps

# Iniciar novamente
docker start meu-website-prod

# Verificar se voltou
docker ps
🔧 Troubleshooting
Problema 1: "Cannot connect to the Docker daemon"
Solução:

sudo systemctl start docker
sudo usermod -a -G docker $USER
# Fazer logout e login novamente
Problema 2: Site não abre no navegador
Verificações:

Security Group tem porta 80 aberta?
Container está rodando? (docker ps)
IP público está correto?
Teste com curl na EC2: curl localhost
Problema 3: "No basic auth credentials" no pull do ECR
Solução:

# Re-autenticar
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin [ECR_URI]
Problema 4: Permissão negada no Docker
Solução:

# Adicionar usuário ao grupo docker
sudo usermod -a -G docker ec2-user
# Logout e login
exit
ssh -i key.pem ec2-user@IP
🧹 Limpeza de Recursos
⚠️ IMPORTANTE: Para evitar custos, limpe os recursos após o laboratório!

Passo 1: Parar e remover container na EC2
docker stop meu-website-prod
docker rm meu-website-prod
docker rmi 123456789012.dkr.ecr.us-east-1.amazonaws.com/meu-website:v1.0
Passo 2: Terminar instância EC2
Console AWS → EC2
Selecione sua instância
Actions → Instance State → Terminate
[Espaço para print: Confirmação de terminate]

Passo 3: Deletar imagem do ECR
Console AWS → ECR
Selecione o repositório
Selecione a imagem
Delete
Passo 4: Deletar repositório ECR (opcional)
Selecione o repositório
Delete
Passo 5: Deletar Security Group
EC2 → Security Groups
Selecione meu-website-sg
Actions → Delete
Passo 6: Deletar IAM Role (opcional)
IAM → Roles
Selecione EC2-ECR-Role
Delete
🎓 Conceitos Aprendidos
✅ Containerização: Empacotamento de aplicações com suas dependências

✅ Docker: Plataforma para criar e executar containers

✅ Dockerfile: Arquivo de configuração para construir imagens

✅ ECR: Registro privado de imagens Docker na AWS

✅ EC2: Máquinas virtuais na nuvem AWS

✅ Security Groups: Firewall virtual para EC2

✅ IAM Roles: Gerenciamento de permissões na AWS