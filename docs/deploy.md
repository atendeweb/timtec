> **ATENÇÃO 0:** Essa é a documentação básica de instalação (deploy) timtec. Para informações sobre requisitos e dependências, veja o [README.md](https://github.com/hacklabr/timtec/blob/master/README.md).

> **ATENÇÃO 1:** execute os comandos na sequencia em que são apresentados aqui.

> **ATENÇÃO 2:** quando o comando precisar ser executado como **root** ele virá precedido desta indicação **root@server#**. Quando o comando precisar ser executado como **usuário da aplicação** ele virá precedido desta indicação **user@server$**. 

> **ATENÇÃO 3:** se tiver dúvidas, assista ao vídeo-log do processo de instalação aqui: https://youtu.be/Xg_YwDmCwVs

## Prepare o servidor

Você precisa de um servidor com algum dos seguintes sistemas operacionais:

* Ubuntu 16.04 
* Debian 9.0

> **ATENÇÃO 4:** É possível instalar em outras distribuições, no entanto não fizemos os testes de 
deploy que recobrem esses procedimentos. Você pode fazer estes testes e construir uma documentação com base em outros servidores. Se fizer isso, mande um pullrequest para a gente que incluiremos a documentação neste repositório oficial. 

> **ATENÇÃO 5:** Certifique-se de ter a senha ssh deste servidor para começar o processo e de ter permissão de sudo(root) na máquina em questão. 

## Dependências

* Atualize o **ÍNDICE DE PACOTES** de seu servidor

```
root@server# apt-get update
```

* Instale a ferramenta **GIT**

```
root@server# apt-get install git
```

* Instale os pacotes do **PYTHON**

```
root@server# apt-get install build-essential python-dev gettext python-virtualenv
```

* Instale algumas **BIBLIOTECAS DE PROCESSAMENTO DE IMAGENS**

```
root@server# apt-get install libpq-dev libjpeg-dev libpng-dev 
```

* Instale o **NODEJS**

> **ATENÇÃO 6:**  Você pode instalar o nodejs no Debian ou no Ubuntu, mas o procedimento é diferente em cada um deles. Veja abaixo e escolha de acordo com sua distribuição. 

* Nodejs no **Ubuntu**
Diferente de todas as outras distribuições, o ubuntu usa o comando node para o nodejs por padrão, então vamos fazer isso.

```
root@server#  apt-get install nodejs npm
root@server# update-alternatives --install /usr/bin/node node /usr/bin/nodejs 10
```

* Nodejs no **DEBIAN**

```
root@server# apt-get install curl
root@server# curl -sL https://deb.nodesource.com/setup_6.x | bash -
root@server# apt-get install nodejs
```
* Instalando o **POSTGRES**

Recomendamos o postgreSQL, mas o django suporta outros bancos de dados relacionais.

```
root@server# apt-get install postgresql
```

* Instale o servidor web **nginx** e o servidor de aplicação **UWSGI**

```
root@server# apt-get install nginx uwsgi uwsgi-plugin-python
```

## Crie o usuário da aplicação no servidor

Sugerimos que o usuário usado para fazer a instalação não seja o root. Aqui, o usuário que está fazendo a instalação é o **timtec-production**. Assim, nos comandos abaixo, substitua o nome do usuário pelo que você estiver usando. Os scripts possuem uma variável para definir o usuário que executará o proxy (wsgi). Caso use um usuário diferente, olhe o arquivo Makefile e faça as alterações necessárias.

Este tutorial pressupões que você esteja logado com o usuário timtec-production, que ele esteja no grupo sudo.

É possível verificar se o usuário timtec-production existe através do comando:
```
root@server# grep timtec-production /etc/passwd
```

Caso o usuário exista, o comando acima deve retornar uma linha semelhante a seguinte:
```
timtec-production:x:999:999::/home/timtec-production:
```
Caso não exista é possível criá-lo com:

```
root@server# useradd --groups sudo --create-home timtec-production
```
Depois mude a senha caso não se lembre:

```
root@server# passwd timtec-production
```



## Obtendo o código

* Com usuário da aplicação - no nosso caso **timtec-production** - faça a clonagem do repositório:

```
timtec-production@server$ git clone https://github.com/institutotim/timtec.git
```

* Faça checkout para versão atual (4.1.10)

```
timtec-production@server$ cd timtec
timtec-production@server$ git checkout 4.1.10
```

## Criando Banco de dados

* Como root, Crie um usuário para o banco de dados:

```
root@server# sudo su - postgres -c "createuser -d timtec-production"
```

* Como usuário da aplicação - logado como **timtec-production** - crie a base de dados

```
timtec-production@server$ createdb --encoding "UTF-8" timtec-production
```

## Instalação

* Copie o arquivo de configuração modelo da aplicação. Esse arquivo fica dentro de ```/timtec/timtec/settings_local.py.template``` e deve ser copiado para ```/HOME-USER-DA-APLICACAO//timtec/settings_local.py```

Veja exemplo com usuário ```timtec-production```:
```
timtec-production@server$ cp /home/timtec-production/timtec/timtec/settings_local.py.template /home/timtec-production/timtec/timtec/settings_local.py
```

Depois de copiar, entre nesse arquivo e faça as adaptações necessárias. Por padrão ele vem pronto para uma instalação nova, seguindo os parametros passados acima, mas você pode mudar conforme melhor se adequar a sua situação. Nesse arquivo você deve encontrar as informações de banco de dados, url, entre outras coisas importantes. 

* Faça a edição nesse arquivo e esteja atento aos pontos abaixo:

```
timtec-production@server$ nano /home/timtec-production/timtec/timtec/settings_local.py

```

Edite estes trechos caso precise:

```
# Acrescente a url principal que deseja usar
DOMAIN_NAME = 'suaurl.com.br'

# Aqui devemos deixar dessa forma para que ele possa pegar a url da variável acima
ALLOWED_HOSTS = [
    DOMAIN_NAME,
    'wwww.' + DOMAIN_NAME
]

# Mude isto caso queira alterar o nome da base de dados
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'timtec-production',
        'USER': 'timtec-production',
    }
}

# Aqui é o padrão do endereço de armazenamento dos files da aplicação
MEDIA_ROOT = "/home/timtec-production/webfiles/media/"
STATIC_ROOT = "/home/timtec-production/webfiles/static/"

```

> **ATENÇÃO 7:** Caso queira extender os temas-padrão para fazer modificações e evitar que sejam sobreescritas ao atualizar a aplicação ou mesmo usar um tema próprio com interface diferente, você pode seguir estes passos descritos em [configurando novo tema](https://github.com/institutotim/timtec/blob/master/docs/instalacao_e_configuracao/config_theme.md)


* Utilize o make para rodar todos os comandos necessários para a instalação da aplicação propriamente dita

```
timtec-production@server$ cd ~/timtec
timtec-production@server$ make install
```


> **ATENÇÃO 7:** Se ocorrer algum erro, tente rodar o comando make novamente, pois falhas podem ocorrer devido a problemas com a internet.


## Servidor web e de aplicação

* Na pasta timtec/scripts/conf temos exemplo de arquivos de configuração. Copie os mesmos para seus locais e edite-os confore sua necessidade:

```
root@server# cp /home/timtec-production/timtec/scripts/conf/timtec-production.ini /etc/uwsgi/apps-available
```

* Nas configurações do UWSGI, crie um link simbólico de apps-available para apps-enable

```
root@server# ln -s /etc/uwsgi/apps-available/timtec-production.ini /etc/uwsgi/apps-enabled/timtec-production.ini
```

* Inicie servidor de aplicação

```
root@server# service uwsgi start
```

* Copie os scritps de configuração da instância timtec-production para os sites available do nginx

```
root@server# cp /home/timtec-production/timtec/scripts/conf/nginx-timtec-production /etc/nginx/sites-available/timtec-production

```
* Edite o arquivo de configuração do nginx para colocar seu domínio
```
root@server# vi /etc/nginx/sites-available/timtec-production

EDITE O QUE PRECISAR NESTE ARQUIVO
```

* Crie link simbólico do projeto de sites-available para sites-enable

```
root@server# ln -s /etc/nginx/sites-available/timtec-production /etc/nginx/sites-enabled/timtec-production
```

* Remova o arquivo de configuração padrão do nginx para não haver conflito

```
root@server# rm /etc/nginx/sites-enabled/default
```

* Faça um reload do nginx para se certificar que ele está rodando corretamente

```
root@server# nginx -s reload
```
* Se o nginx não estiver rodando, execute:

```
root@server# service nginx start
```


A instalação não terminou ainda! Precisamos criar o usuário inicial, configurar o domínio do django, o envio de email e a API do youtube.

Proceda a página de [Configurações](config.md) para continuar.
