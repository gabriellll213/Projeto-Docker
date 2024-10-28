### Configuração de Infraestrutura AWS com EC2, RDS e EFS para Deploy de WordPress
Este repositório fornece instruções para criar e configurar uma infraestrutura AWS utilizando uma instância EC2, banco de dados RDS, e sistema de arquivos EFS, com suporte a Docker e Docker Compose para hospedar uma aplicação WordPress.

### Índice
- Pré-requisitos
- Configuração do Ambiente
- 1. Criar a Instância EC2
- 2. Configurar o Banco de Dados RDS (MySQL)
- 3. Configurar o Sistema de Arquivos EFS
- Configuração do Script de Inicialização
- Executando o Docker Compose
- Observações Importantes
### Pré-requisitos
Conta AWS com permissões para criação de instâncias EC2, RDS, e EFS.
AWS CLI configurado.
Chave SSH para acesso à instância EC2.
### Configuração do Ambiente
1. Criar a Instância EC2
Acesse o Console da AWS, vá para EC2 > Instances e clique em Launch Instance.
Escolha o AMI: Use uma imagem Ubuntu Server 20.04 LTS.
Selecione o Tipo de Instância: t2.micro (grátis no nível de uso AWS Free Tier).
Configurações de Rede: Escolha a VPC e o grupo de segurança apropriados. Assegure-se de que a porta 80 esteja aberta para acesso HTTP.

User Data Script: No campo User data (opções avançadas), cole o script abaixo para configurar automaticamente a instância.
```
bash
Copiar código
#!/bin/bash

# Atualizar pacotes e instalar Docker e NFS
sudo apt-get update
sudo apt-get install -y docker.io nfs-common

# Iniciar e habilitar o serviço Docker
sudo systemctl start docker
sudo systemctl enable docker

# Configurar montagem do EFS
sudo mkdir /mnt/efs
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport fs-04a8493841bd310bd.efs.us-east-1.amazonaws.com:/ /mnt/efs

# Instalar Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Criar o arquivo Docker Compose para WordPress e MySQL (RDS)
sudo tee /mnt/efs/docker-compose.yml > /dev/null <<EOF 

version: '3.1'

services:
  wordpress:
    image: wordpress
    restart: always
    ports:
      - 80:80
    environment:
      WORDPRESS_DB_HOST: wordpressdocker.c3y6qu8681dn.us-east-1.rds.amazonaws.com
      WORDPRESS_DB_USER: gabriel
      WORDPRESS_DB_PASSWORD: <SUA_SENHA>
      WORDPRESS_DB_NAME: exampledb
    volumes:
      - /mnt/efs:/var/www/html

  db:
    image: mysql:8.0
    restart: always
    environment:
      MYSQL_DATABASE: wordpressdocker
      MYSQL_USER: gabriel
      MYSQL_PASSWORD: <SUA_SENHA>
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
    volumes:
      - db:/var/lib/mysql

volumes:
  wordpress:
  db:
EOF

# Iniciar containers com Docker Compose
cd /mnt/efs
sudo docker-compose up -d
Nota: Substitua <SUA_SENHA> pela senha desejada para o banco de dados RDS.
```

Finalize a Criação: Confirme as configurações e clique em Launch Instance.

### 2. Configurar o Banco de Dados RDS (MySQL)
Acesse o Amazon RDS no Console da AWS e clique em Create Database.
Selecione MySQL como o mecanismo de banco de dados.
Configure:
Nome do banco de dados: exampledb
Usuário: gabriel
Senha: defina uma senha segura
Na seção de conectividade, escolha a mesma VPC da instância EC2.
Conclua a criação do banco de dados.
3. Configurar o Sistema de Arquivos EFS
Acesse o Amazon EFS e clique em Create file system.
Configure o EFS para usar a mesma VPC da instância EC2.
Após a criação, copie o ponto de montagem DNS para usá-lo no script (exemplo: fs-04a8493841bd310bd.efs.us-east-1.amazonaws.com).
Configuração do Script de Inicialização
O script configurará a instância EC2 para:

Atualizar pacotes e instalar Docker e NFS.
Montar o EFS na pasta /mnt/efs.
Baixar e instalar o Docker Compose.
Criar o arquivo docker-compose.yml com as configurações para o WordPress e MySQL.
Iniciar os containers com Docker Compose.
Executando o Docker Compose
Acesse a instância EC2 via SSH e verifique se os containers estão em execução com o comando:

bash
Copiar código
docker ps
Observações Importantes
Segurança: Certifique-se de proteger credenciais sensíveis e restringir o acesso aos recursos.
Portas de Acesso: Garanta que as portas usadas (80 para o WordPress) estejam abertas para permitir o tráfego.
Monitoramento: Use o CloudWatch para monitorar a performance da instância e os logs de erro.
Este setup agora está pronto para execução, permitindo a implementação de um site WordPress com armazenamento persistente no EFS e banco de dados gerenciado no RDS.






