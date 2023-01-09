# GLPI Deploy

O [GLPI](https://glpi-project.org/) é um sistema de código aberto escrito em PHP para Gerenciamento de Ativos de TI, rastreamento de problemas e central de serviços. [GPLI Documentação](https://glpi-project.org/pt-br/documentacao/).

## Sumário

- [GLPI Deploy](#glpi-deploy)
  - [Sumário](#sumário)
  - [Requisitos e Dependências](#requisitos-e-dependências)
  - [Preparação de Ambiente](#preparação-de-ambiente)
    - [Docker:](#docker)
    - [GLPI:](#glpi)
  - [Instalação](#instalação)
    - [Estrutura de Diretório](#estrutura-de-diretório)
      - [Banco de Dados](#banco-de-dados)
      - [Aplicação](#aplicação)
    - [Docker-Compose](#docker-compose)
      - [Portas](#portas)
      - [Variáveis de Ambiente (Environment)](#variáveis-de-ambiente-environment)
      - [Volumes](#volumes)
    - [Executando Docker-Compose](#executando-docker-compose)
    - [Configuração de Permissões](#configuração-de-permissões)
      - [Banco de Dados](#banco-de-dados-1)
      - [Aplicação - GLPI](#aplicação---glpi)
    - [Configurando Proxy Reverso](#configurando-proxy-reverso)
    - [Finalização](#finalização)
      - [Segurança](#segurança)
        - [Pasta de Instalção](#pasta-de-instalção)
- [Intalação com backup (Migração/Restauração)](#intalação-com-backup-migraçãorestauração)
  - [OBSERVAÇÕES](#observações)

## Requisitos e Dependências

- [Docker e Docker-Compose](https://docs.docker.com/)

- [GLPI Website](http://glpi-project.org/) (para baixar versão atual).

- [Repositório no Github](https://github.com/glpi-project/glpi/releases) (Todas as versões - Recomendado).

- Versões Testadas: 10.0.2 (INSEGURA), 10.0.5.

## Preparação de Ambiente

### Docker:

```bash
# Certifique-se que a aplicação está executando.
# Em algumas distribuições Linux.

# Verifique se o Docker está executando.
$ sudo systemctl status docker.service

# Inicie o Docker (caso necessário).
$ sudo systemctl start docker.service

# Para fazer o Docker iniciar junto com o Sistema Operacional.
$ sudo systemctl enable docker.service  
```

### GLPI:

1. Baixe o GLPI através do links disponibilizados.
2. Extraia o arquivo ***glpi-{version}.tgz***. Copie a pasta extraída para ***$(pwd)/glpi-deployer/app***. 

Obs: ***$(pwd)*** simboliza um caminho qualquer na máquina do usuário. Ajuste-o de acordo com suas preferências/necessidades.

## Instalação

### Estrutura de Diretório

#### Banco de Dados

```bash
# Crie os diretórios.

# Dir. Dados
$ mkdir $(pwd)/lib-mysql
```
Sugestão (no Linux):
  - Dir. Dados: */var/lib/mysqlglpi*

#### Aplicação

```bash
# Crie os diretórios.

# Dir. Config
$ mkdir $(pwd)/etc-glpi

# Dir. Dados
$ mkdir -p $(pwd)/lib-glpi/{_cron,_dumps,_graphs,_lock,_pictures,_plugins,_rss,_sessions,_tmp,_uploads,_cache}

# Dir. Log
$ $(pwd)/log-glpi
```

Sugestão (no Linux):
  - Dir. Config: */etc/glpi*
  - Dir. Dados: */var/lib/glpi*
  - Dir. Log: */var/log/glpi*

```bash
# Copie o arquivo "local_define.php" para o diretório "config".

$ cp $(pwd)/glpi-deployer/app/config-app/local_define.php $(pwd)/etc-glpi
```

```
# Comente a linha que ignora a pasta de instalação do GLPI.

# Antes
**/glpi/install

# Depois
# **/glpi/install
```

### Docker-Compose

```
# Comente a linha que ignora a pasta de instalação do GLPI.

# Antes
**/glpi/install

# Depois
# **/glpi/install
```

```yml
# docker-compose.yml (Em networks.glpi-net.ipam)
# Altere os valores caso necessário. 

config:
# Endereço da redeSugestão (no Linux):
  - Dir. Dados: */var/lib/mysqlglpi*
  - subnet: '172.18.0.0/28'
# Gateway da rede
    gateway: 172.18.0.1
```

```yml
# docker-compose.yml (Em services.app.networks)
# Altere os valores caso necessário.

- glpi-net:
# Endereço do container```
# Comente a linha que ignora a pasta de instalação do GLPI.

# Antes
**/glpi/install

# Depois
# **/glpi/install
```
```
Sugestão (no Linux):
  - Dir. Dados: */var/lib/mysqlglpi*
```yml
# docker-compose.yml (Em services.db.networks)
# Altere os valores caso necessário. 

- glpi-net:
# Endereço do container
  ipv4_address: 172.18.0.2
```

#### Portas

```yml
# docker-compose.yml (Em services.app)

# Comente/Descomente (e/ou altere) as portas/serviços que você deseja oferecer.

# Não recomendado.
# Recomendado usar um proxy reverso para expor à internet (com HTTPS).

ports:
# Porta para HTTP.
  - '80:80'
```

```yml
# docker-compose.yml (Em services.db)

# Comente/Descomente (e/ou altere) as portas/serviços que você deseja oferecer.

# Cuida, isso pode expor seu banco para outros hosts.
# Só altere se isso realmente for desejado.

ports:
  - '127.0.0.1:3306:3306'
```

#### Variáveis de Ambiente (Environment)

```yml
# docker-compose.yml (Em services.db)

environment:
# Senha do usuário root
  - MARIADB_ROOT_PASSWORD=rootpass
# Host do root. "localhost" ou "%"(não recomendado)
  - MARIADB_ROOT_HOST=localhost
# Nome do usuário criado 
  - MARIADB_USER=username
# Senha do usuário
  - MARIADB_PASSWORD=userpass
```
novamente
#### Volumes

```yml
# docker-compose.yml (Em services.app)
# Aponte para as pastas criadas anteriormente.

# Antes
volumes:
  - '$(pwd)/etc_glpi:/etc/glpi'
  - '$(pwd)/lib_glpi:/var/lib/glpi'
  - '$(pwd)/log_glpi:/var/log/glpi'

# Depois (exemplo)
volumes:
  - '/etc/glpi:/etc/glpi'
  - '/var/lib/glpi:/var/lib/glpi'
  - '/var/log/glpi:/var/log/glpi'
```

```yml
# docker-compose.yml (Em services.db)
# Aponte para as pastas criadas anteriormente.

# Antes
volumes:
  - '$(pwd)/lib-mysql:/var/lib/mysql'

# Depois (exemplo)
volumes:
  - '/var/lib/mysqlglpi:/var/lib/mysql'
```

### Executando Docker-Compose

```bash
$ docker-compose -f docker-compose.yml up
```

### Configuração de Permissões

#### Banco de Dados

```sql
/* # db/grant.sql */

/*  Linhas 15 e 23 (Todos os locais). */
... TO 'username'@'%' IDENTIFIED BY 'userpass'; 

```

1. Substitua ***username*** pelo valor inserido em ***MARIADB_USER***.
2. Substitua ***userpass*** pelo valor inserido em ***MARIADB_PASSWORD***.
3. Ao configurar a aplicação o banco de deve ser inserido como ***db_glpi***. Caso opte por utilizar outro nome substitua o termo ***db_glpi*** na linha 23 pelo de sua preferência.

Acesse o banco de dados com um utExecutando Docker-Composeilitário gráfico ou via terminal e execute os comandos do arquivos mencionado.

```bash
# Via terminal

# Entre no container
$ sudo docker exec -it glpi-db /bin/bash

# Entre no SGBD (com o usuário root e senha root)
$ mariadb -u root -p

# Agora cole - um por vez - os dois comando.
# Certifique-se de ter feitos as alterações.
```
Executando Docker-Compose
#### Aplicação - GLPI

```bash
# No terminal

# Entre no container
$ sudo docker exec -it glpi-app /bin/bash

# Execute 
$ chown -R www-data:www-data /etc/glpi 
$ chown -R www-data:www-data /var/lib/glpi
$ chown -R www-data:www-data /var/log/glpi
```

### Configurando Proxy Reverso

Para adicionar uma camada a mais de segurança recomendamos usar alguma aplicação de proxy reverso, como por exemplo o Nginx. Isso permite esconder o servidor Apache do usuário final além de tornar mais fácil e escalável configurar coisas como HTTPS, redirecionamentos, etc. [Guia para instalação de um proxy manager](https://github.com/nutecuneal/proxy-manager-deployer/tree/feature).

### Finalização

A partir de seu navegador acesse o domínio/IP e a porta configurada no servidor.

Siga o [*Intall-Wizard GLPI*](https://glpi-install.readthedocs.io/en/latest/install/wizard.html) para concluir o processo.

#### Segurança

##### Pasta de Instalção

Por motivo de segurança recomenda-se remover a pasta de instalação de dentro do código da aplicação. Por isso, faça:

```
# Descomente a linha que ignora a pasta de instalação do GLPI.

# Antes
#**/glpi/install

# Depois
**/glpi/install
```

```bash
# Remova os containers
$ sudo docker rm -f glpi-app glpi-db

# Remova o imagem
$ sudo docker rmi -f glpi-deployer-glpi

# Remova os caches de build
$ docker builder prune -a

# Execute o Docker-Compose novamente
$ sudo docker-compose -f docker-compose.yml up
```



# Intalação com backup (Migração/Restauração)

## OBSERVAÇÕES

Dentro do GLPI:

- Autenticação e destinatário.
- Adicionar Remetente.
- `Geral:Assistência:Permitir abertura de chamados anônimos`
- `Configurar:notifições:notifições`

Ações automáticas:

- mailgate *(Puxa os chamados que vem do email)*
- queuednotification *(Envio da fila de notificação)*

Tarefa automática:

- Periodo de execução: É o horário em que a tarefa pode ser executada, ex: de 0 horas até 24 horas, daquele dia.

TO-DO:

**Verificar o modelo de resposta ao usuário do chamado**.