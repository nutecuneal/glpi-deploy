# GLPI Deploy

## Sumário

- [GLPI Deploy](#glpi-deploy)
  - [Sumário](#sumário)
  - [Pré-instalação (Requisitos)](#pré-instalação-requisitos)
    - [Downloads](#downloads)
  - [Sobre](#sobre)
    - [Docker:](#docker)
    - [Glpi:](#glpi)
  - [Preparação](#preparação)
    - [Docker:](#docker-1)
    - [Criação de Diretórios](#criação-de-diretórios)
  - [Construção dos Containers Docker](#construção-dos-containers-docker)
    - [Banco de Dados](#banco-de-dados)
    - [Docker](#docker-2)
    - [Configurando a Aplicação](#configurando-a-aplicação)
      - [GLPI - Banco de Dados](#glpi---banco-de-dados)
      - [GLPI - Aplicação](#glpi---aplicação)
- [To-do ( Instalação com backup (Migração/Restauração) )](#to-do--instalação-com-backup-migraçãorestauração-)
  - [OBSERVAÇÕES](#observações)
- [GLPI-Agent](#glpi-agent)
  - [Instalação Windows](#instalação-windows)
    - [Enviar inventário](#enviar-inventário)
  - [Inventário da rede via SNMP](#inventário-da-rede-via-snmp)
    - [Procurar dispositivos na rede](#procurar-dispositivos-na-rede)
## Pré-instalação (Requisitos)
<br>

### Downloads

- Git instalado na máquina. [Site Git](https://git-scm.com/)
  
- ```bash 
  #Clone o repositório do projeto.
  #1. Ou clicando na botão download no site do Github.
  #2. Ou Executando o comando abaixo (Necessário autenticação).

  $ git clone https://github.com/nutecuneal/glpi-deploy.git
    ```

## Sobre
<br>

### Docker:
  - Docker é um plataforma que usa virtualização a nível de aplicação/"Sistema Operacional" para entregar softwares empacotados, chamados de containers.
  - [Docker: Guia de Uso e instalação](https://docs.docker.com/desktop/).

### Glpi:
  - Software de gerenciamento de serviços.
  - [GLPI Website](http://glpi-project.org/) (para baixar versão atual).
  - [Repositório do Github](https://github.com/glpi-project/glpi/releases ) (Todas as versões - Recomendado).
  - Versões Testadas: 10.0.2.

## Preparação
<br>

### Docker:
- Certifique-se que a aplicação está executando.
- ```bash
  #Em algumas distribuições Linux.

  #Para verificar se o Docker está executando.
  $ sudo systemctl status docker.service
  
  #Para iniciar o Docker (caso necessário).
  $ sudo systemctl start docker.service

  #Para fazer o Docker iniciar junto com o Sistema Operacional.
  $ sudo systemctl enable docker.service  
  ```

### GLPI:
- Extraia o arquivo *glpi-{version}.tgz*. Copie a pasta extraída para dentro de "*$path1*/glpi-deploy/main". Onde *\$path1* é o caminho para pasta *glpi-deploy* clonada na seção [Projeto GLPI-Deploy](####Projeto-GLPI-Deploy).

## Primeira Instalação
<br>

```bash
# Entre na pasta glpi-deploy

$ cd $path1/glpi-deploy
```

### Criação de Diretórios

```bash
$ mkdir -p $path2/lib/glpi/{config,data,database}
$ mkdir -p $path2/lib/glpi/data/{_cron,_dumps,_graphs,_lock,_pictures,_plugins,_rss,_sessions,_tmp,_uploads,_cache}
$ mkdir -p $path3/log/glpi

# Copiando arquivo de configuração de diretórios da aplicação
$ cp main/configs/php/local_define.php $path2/lib/glpi/config
```
**Um possível valor para *\$path2* e *\$path3* é "/var".**

## Construção dos Containers Docker
<br>

### Banco de Dados

- Altere os seguintes arquivos (pasta "database") inserindo os valores preterido seguindo os modelos.
  
```bash
# Em ".env"

# Senha do usuário root
MARIADB_ROOT_PASSWORD

# Hosts permitidos para acesso ao usuário root. Pode ser ou "%" ou "localhost" (sem aspas). Padrão "localhost".  
MARIADB_ROOT_HOST

# Nome do usuário que será utilizado na aplicação do "GLPI". 
MARIADB_USER

# Senha do usuário
MARIADB_PASSWORD
```

```bash
# Em ".grant.sql"

#Linhas 15 e 23
... TO 'test'@'%' IDENTIFIED BY 'test'; 

## Substitua 'test' em ...['test'@'%']... pelo valor inserido em MARIADB_USER

## Substitua 'test' em ...[IDENTIFIED BY 'test']... pelo valor inserido em MARIADB_PASSWORD

## Ao configurar a aplicação o banco de deve ser inserido como "db_glpi". Caso queira utilizar outro nome substitua o termo "db_glpi" na linha 23 pelo de sua preferência.
```

### Docker

```dockerfile
#  Em "docker-compose.yml"

# Service: glpi

## Altere o valor "5000" pela porta escolhida para a aplicação GLPI.
ports:
  - "5000:80" 

## Altere o valor "~/glpi-storage/" pelos caminhos definidos no tópico "Criação de Diretórios"
volumes:
  - ~/glpi-storage/lib/glpi/config:/etc/glpi
  - ~/glpi-storage/lib/glpi/data/:/var/lib/glpi
  - ~/glpi-storage/log/glpi:/var/log/glpi

## Descomente a linha (remova o "#" no início da linha)
## (Antes)
"# - ./main/glpi/install:/var/www/html/install"
## (Depois)
"- ./main/glpi/install:/var/www/html/install"


# Service: glpi_db
## Faça as mesmas alterações.
volumes:
  - ~/glpi-storage/lib/glpi/database:/var/lib/mysql
```

```bash
# No terminal construa a imagem

$ sudo docker-compose -f docker-compose.yml up
```

### Configurando a Aplicação

```bash
# Para verificar se os containers estão executando
$ sudo docker container ls
```

#### GLPI - Banco de Dados

```bash
# Acesse o container através do comando
$ sudo docker exec -it glpi-db /bin/bash

# Entre no SGBD (com o usuário root e sua senha)
$ mariadb -u root -p

# Agora, no terminal cole - um por vez - os dois comando do arquivo "grant.sql". Certifique-se de ter feitos as alterações mencionadas no início do tutorial.
```

#### GLPI - Aplicação
```bash
# Entre no container através do comando
$ sudo docker exec -it glpi /bin/bash

# Execute
$ chown -R www-data:www-data /etc/glpi 
$ chown -R www-data:www-data /var/lib/glpi
$ chown -R www-data:www-data /var/log/glpi
```

Informações para acesso local:
  - GLPI: 172.16.1.3:80 ou "localhost:portaGLPI" ou "ipHost:portaGLPI"
  - GLPI Banco de Dados: 172.16.1.2:3306 ou "localhost:3306"

Agora, em seu navegador acesse a aplicação GLPI. Faça a configuração seguindo o [manual de instalação](https://glpi-install.readthedocs.io/en/latest/install/wizard.html).

<br>

Por motivo de segurança recomenda-se remover a pasta de instalação de dentro do código da aplicação. Por isso, faça:


```dockerfile
#  Em "docker-compose.yml"
# Service: glpi
## Comente a linha (inserindo "#"
## (Antes)
"- ./main/glpi/install:/var/www/html/install"
## (Depois)
"# - ./main/glpi/install:/var/www/html/install"
```
```bash
# Remova os containers
$ sudo docker rm -f glpi glpi-db

# Remova o imagem
$ sudo docker rmi -f glpi-deploy-glpi

# Execute os containers novamente
$ sudo docker-compose -f docker-compose.yml up
```

# To-do ( Instalação com backup (Migração/Restauração) )

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

<br>

# GLPI-Agent

Glpi-agent:

  - É um serviço que permite gerar o inventário da máquina e enviar seus resultados para um servidor de confiança.
  - Ele também permite indentificar máquinas na rede e gerar seus inventários, utilizando o protocolo SNMP.
  - Funciona tanto no Windows como no Linux.

Caso tenha algum problema ou dúvida durante a instalação e uso do agente, acesse a documentação:   (https://glpi-agent.readthedocs.io/en/latest/)

## Instalação Windows
<br>

Para realizar a instalação no windows, você deve baixar o instalador do agente no site ou GitHub do glpi.

- Download agent: (https://github.com/glpi-project/glpi-agent/releases)

Ao terminar, execute o .exe:

  - Aceite o contrato e avance.
  - Pode deixar o caminho de instalação no *"Program Files/"* mesmo.
  - No tipo de instalação temos 3 opções:
    - Typical: Instala somente as ferramentas necessárias para a obtenção do inventário. **(Recomendado para as máquinas dos usuários).**
     - Complete: Instala todas as ferramentas para inventário, descobrimento de máquinas na rede via SNMP e inventário remoto. **(Recomendado para a máquina que vai utilizar os recursos de SNMP, pois assim ela consegue enviar essas informações ao servidor do GLPI).**   
     - Custom: Permite selecionar quais ferramentas serão instaladas nessa máquina.
- Após selecinado, devemos inserir a url do nosso servidor em Remote target, pode ser: DNS https://example.com.br/front/inventory.php  ou IP do servidor.
- Local target pode deixar vazio.
- Agora é  avançar e aguardar a instalação.

### Enviar inventário
<br>

Ao terminar a instalação, o agente já vai ser iniciado e enviará o inventário automáticamente para o servidor configurado de tempos em tempos, mas podemos adiantar o primeiro envio e ver se tudo esta funcionando. Para isso:

- Acesse o caminho: "*C:Program Files/GLPI Agent/*"
- Ao entrar na pasta procure pelo bat glpi-agent e o execute.
- Depois procure o bat glpi-inventory e o execute. Ele pode demorar um pouco para ser executado pois vai colher as informações da sua máquina.
- Quando ele encerrar, abra um navegador e digite: [localhost:62354](localhost:62354)
- Vai abrir uma página simples, onde mostra o que o agente está fazendo, qual a próxima hora de envio do inventário ao servidor e uma opção para forçar o envio imediato.
- Clique em Force Inventory e pronto, o inventário foi enviado para o servidor do GLPI.
- Abra o servidor e verifique se as informações da máquina chegaram. 

**Obs: O caminho *"C:Program Files/GLPI Agent/"* só exite se você deixou ele como padrão na hora da instalação, caso tenha alterado, procure a pasta do Glpi Agent no caminho informado por você.**



## Inventário da rede via SNMP
<br>

Para realizar o inventário da rede, a máquina que realizará isso deve ter o agente instalado com a opção complete.

Diferente do inventário da máquina que é realizado de forma automática e sempre atualiza as informações, o inventário de rede é feito de forma manual e não atualiza as informações no servidor automáticamente. Assim, caso algum dispositivo mude de ip ou seja adicionados mais dispositivos na rede, o processo manual deve ser realizado novamente.

**Obs: Antes de enviar um inventário manual para o Glpi, verifique se a regra de import denied está inativa:**

1. Acesse o sistema do Glpi
2. Na aba Administração clique em inventário
3. Procure: Regras para importação e vínculo de equipamentos e clique
4. Selecione Dispositivo de rede 
5. Clique em NetworkEquipment import denied
6. Caso esteja ativa, desative e clique em salvar


### Procurar dispositivos na rede
<br>

Os comandos que podem ser utilizados são:

- glpi-netdiscovery -> Faz a identificação dos dispositivos na rede.
- glpi-netinventory -> Gera o inventário dos dispositivos.
- glpi-injector -> Faz o envio do inventário de rede para o servidor.
  

**Obs: Antes de procurar um dispositivo, verifique se ele possui o protocolo SNMP ativo, alguns por padrão vem desativado.**
  
Para todos os procedimentos  devemos utilizar o cmd do Windows.


- ```cmd
  REM/ -> Local de comentário

   REM/ Acesse a pasta do Glpi-agent

  cd C:Program Files/Glpi Agent/

  REM/ utilize o comando glpi-netdiscovery e seus parâmetros para procurar os dispositivos

  glpi-netdiscovery --first 192.168.1.1 --last 192.168.1.254 --port 161 --community public -i -s  dispostivos\

  REM/ -i -> Já gera o inventário da máquina também, sem precisar usar o comando glpi-netinventory depois.

  REM/ -s -> Informe em que pasta você quer salvar os arquivos de inventário, se a pasta estiver fora da Glpi Agent, informe o caminho completo, ex.: C:Users\Documentos\inventario
  ```

Execute o comando ajustando os parâmetros a sua necessidade. Algumas explicações sobre os parâmetros usados:

- port -> É a porta que o dispositivo usa para o SNMP, por padrão é a 161.
- community -> É o nome da comunidade no dispositivo, por padão é public.
- --v1, --v2c -> Pode ser necessário especificar caso o dispositivo não utilize a v1 que é a padrão.
- --host -> Caso queira somente identificar 1 dispositivo na rede utilize ao invés de --first e --last.

Ex.:

```cmd 
glpi-netdiscovery --host 192.168.1.20 --port 161 --v2c --community public -i -s dispositivos\
```

Após o comando executar e caso não informe erros, serão criadas sub-pastas dentro de dispositivos:

- netdiscovery
- netinventory

A pasta que interessa para adicionar os dispositivos no servidor é a *netinvetory*. Agora devemos fazer o envio dos arquivos gerados para o servidor.

```cmd
Rem/ Dentro da pasta do Glpi Agent execute

glpi-injector -v -f dispositivos\netinventory\192.168.1.20.xml --url https://login@example.com.br/front/inventory.php ou https://login@ip-servidor
```

Nesse caso o envio será de um arquivo apenas, caso seja uma pasta inteira utilize:

```cmd
glpi-injector -v -R -d dispositivos\netinventory --url https://login@example.com.br/front/
```

Explicação dos parâmetros:

- -v -> Vai dando um feedback para o usuário sobre o que o comando esta fazendo.
- -f -> Lê um arquivo.
- -R -> Recursivo, ou seja, tudo que estiver na pasta.
- -d -> Lê o diretório.
- --url -> É o caminho do servidor, nele é preciso informar o nome do usuário de login e o endereço do servidor.
  

Acesse o sistema do Glpi e verifique se os dispositivos foram adicionados corretamente.

Também é possivel fazer o envio do inventário de rede pelo sistema do Glpi, porém a desvantagem é que ele só aceita 1 arquivo por vez. Para isso:

1. Acesse o sistema do Glpi
2. Na aba Administração clique em inventário
3. Selecione Importar do arquivo
4. Clique em Escolher arquivo
5. Escolha o arquivo
6. Clique em Upload
7. Volte e verifique se foi adicionado corretamente










