Nesse homelab iremos hospedar um site em um servidor web, que é o Apache utilizando o Ubuntu Server. O projeto que será armazenado no Apache se trata de um site climatico, na qual ao inserir a cidade, ele trará as informações do clima da região. O site é estático, utilizando apenas HTML e CSS e não possui banco de dados ou outras aplicações. Dessa forma, apenas uma VM será o suficiente para entender como funciona o serviço Apache. Antes de qualquer coisa, caso Apache não esteja instalado rode os seguintes comamdos:

* sudo apt udpdate
* sudo apt install apache -y

Depois de instalado o serviço web, validar a funcionalidade do serviço com o systemctl, utilitário do systemd:

* sudo systemctl status apache2

(imagem)

Inicialmente já foi identificado o fuso horário diferente, que pode afetar na hora de verificações de logs e analises sobre o serviço. O fuso horário está em UTC, é o padrão de tempo comumente usado para determinar os horários locais em todo o mundo. Dessa forma, vamos fazer os devidos ajustes para ficar no fuso horário do Brasil.

* timedatectl - Horário atual

(imagem)

* timedatectl list-timezones -  Lista zonas disponíveis

(imagem) 

* sudo timedatectl set-timezone America/Sao_Paulo - Fuso horário escolhido
* sudo timedatectl set-ntp true - Ativar NTP
* sudo timedatectl set-ntp false - Desativar NTP, se quiser

(imagem)

Feito os ajustes necessários para dar continuidade na hospedagem do site, agora é preciso trazer o site criado para a máquina local. O arquivo desse site está hospedado no github e é de visualização pública. A principio o git já está instalado no Ubuntu server. Mas se for necessário instalar, rode:

* sudo apt update
* sudo apt install git -y

A partir disso, iremos clonar o arquivo do site do github para o diretório padrão do Linux para armazenar arquivos web. 

* cd /var/www - Diretório padrão para arquivo web
* sudo git clone https://github.com/EmilyBirschner/Projeto-App-Clima.git

(imagem)

O Apache ele roda com um usuário específico, que é o "www-data". E para ele conseguir ler arquivos do site e as vezes escrever o sistema de permissões do Linux precisa permitir.

* sudo chown -R www-data:www-data /var/www/Projeto-App-Clima

chown = change owner
Serve para alterar:

--Usuário dono

--Grupo dono

* sudo chmod -R 755 /var/www/site-amiga

chmod = change mode
Serve para alterar:

--Permissões de leitura, escrita e execução

(imagem)

Tudo que for hospedado no Apache, precisa de um virtual host. Virtual hosta é um recurso que permite hospedar múltiplos sites ou domínios (ex: site1.com, site2.com) em um único servidor físico ou virtual. Essa funcionalidade otimiza recursos (RAM, CPU, IP), permitindo que o Apache direcione cada requisição ao diretório correto do projeto, baseando-se no nome do domínio acessado. Será criado um em /etc/apache2/sites-available/.

* sudo nano etc/apache2/sites/available/site-clima

(imagem)

(imagem)

Após inserir as informações dentro do virtual hosta. O site web deve ser ativado e o site default desativado.

* sudo a2ensite site-amiga.conf - Ativar o arquivo
* sudo a2dissite 000-default.conf - desativar arquivo

Ao executar o primeiro comando, ocorreu algum erros e o Linux apontou para comandos a serem utilizados para encontrar o motivo do erro. 

(imagem)

Com as orientações passadas pelo próprio sistema, foi identificado erro de syntaxe na terceira linha do virtual host. Identificado que a palavra "DocumentRoot" estava com o "R" de root em minúsculo e o VirtualHost faltou um barra, sendo /VirtualHost. O suficiente para o comando de ativação do site não funcionar. outra forma de ser mais acertivo a onde está os erros da syntaxe, ao idenficar que o erro é por causa disso, é utilizando o comando "sudo apache2ctl configtest' que vai deixar explicito as linhas de erro. No caso desse homelab foi a linha 3 e 18. 

(imagem)

Feito as configurações, mas ao rodar "sudo apache2ctl configtest" novamente ele deu um informando que palavra "Apache" antes de começar a syntaxe do virtual host não faz parte do script. 

(imagem)

Retirado a palavra "apache" do incio da syntaxe.

(imagem)

Syntaxe agora funcionando corretamente.

(imagem)

Agora sim, executar os comandos de ativação do site web e desativação do site default.

(imagem)

Testando o acesso do site localmente.

* curl http://localhost

(imagem)






