# redmine-compose
Docker compose setup para host Redmine.

O principal problema que este repositório está tentando resolver é a instalação (automatizada) de plugins.

O `docker-compose.yml` é quase exatamente o documentado no
[imagem oficial do Redmine docker] (https://hub.docker.com/_redrede), com a adição de um estágio de configuração do plugin.

## O problema

Normalmente instalar plugins no Redmine é muito fácil (embora infelizmente não seja gerenciado pelo built-in
interface de administração) - você `cd` no diretório` plugins`, faça o download do plugin, execute qualquer configuração
é necessário, então reinicie o Redmine.

Mas no Docker, não há uma maneira fácil de lidar com essa reinicialização - exceto a execução do `docker restart redmine_redmine_1`
Manualy, que não é exatamente algo que você pode fazer em uma configuração automatizada.

A inicialização do redmine na imagem oficial do Docker espera que o banco de dados esteja pronto e possa concluir
etapa de inicialização do banco de dados, mesmo que o banco de dados demore para iniciar (o que ele faz para o MariaDB / MySQL, pelo menos), mas
não nos espera instalar plugins.

## A solução

Eu adicionei um novo serviço Docker que está executando uma configuração única para os plugins em um volume compartilhado e termina por
escrevendo um arquivo de notificação "plugins are ready".

Um novo script de wrapper é configurado como o ponto de entrada do Redmine, que aguarda o arquivo "plugins are ready" antes de executar
o ponto de entrada original. Em que ponto a inicialização padrão do Redmine ocorre e pode registrar o
plugins.
  
## Uso

1. Atualize o arquivo `install-plugins.sh` para instalar os plugins que você precisa. O script atual é instalado
`redmine_omniauth_google` como exemplo (e também porque é útil para o meu caso de uso.
1. Se mais dependências forem necessárias para a instalação do plugin, atualize o Dockerfile com as novas dependências.
1. Execute o `docker-compose up`.
1. Aguarde até que os serviços concluam a inicialização.