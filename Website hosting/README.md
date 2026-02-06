Documentação de Homelab: Hospedagem de Site Estático com Apache no Ubuntu Server 

Este projeto descreve a hospedagem de uma aplicação web de consulta climática utilizando o servidor web Apache em ambiente Ubuntu Server. O site é uma aplicação estática (HTML/CSS) que consome dados de clima via API, não demandando banco de dados local. O objetivo é demonstrar a configuração de Virtual Hosts e o fluxo de deploy básico em uma VM.

 1. Instalação e Verificação do Serviço
Caso o Apache não esteja instalado, execute os comandos abaixo para atualizar os repositórios e instalar o pacote:

* sudo apt update
* sudo apt install apache2 -y 

Após a instalação, valide o status do serviço utilizando o utilitário systemctl: 

* sudo systemctl status apache2

 2. Ajuste de Sincronização de Horário (NTP)

Durante a análise inicial, identificou-se que o fuso horário estava configurado como UTC. Para garantir a precisão dos logs e auditorias do serviço, alteramos para o fuso horário brasileiro (America/Sao_Paulo).

* Verificar horário atual: timedatectl

* Listar zonas disponíveis: timedatectl list-timezones

* Configurar fuso horário: sudo timedatectl set-timezone America/Sao_Paulo

* Ativar sincronização NTP: sudo timedatectl set-ntp true

 3. Deploy do Projeto via Git

Com o ambiente preparado, realizamos o clone do repositório público hospedado no GitHub para o diretório de arquivos web do Linux.

* cd /var/www
* sudo git clone https://github.com/EmilyBirschner/Projeto-App-Clima.git

Ajuste de Permissões e Propriedade:
O Apache utiliza o usuário www-data para processar requisições. É necessário ajustar o proprietário e as permissões das pastas para que o servidor consiga ler os arquivos corretamente.

Alterar proprietário (Owner): sudo chown -R www-data:www-data /var/www/Projeto-App-Clima

Alterar permissões (Mode): sudo chmod -R 755 /var/www/Projeto-App-Clima (Garante leitura e execução para o grupo/outros e escrita para o dono).

 4. Configuração do Virtual Host
O Virtual Host permite hospedar múltiplos domínios em um único servidor, direcionando o tráfego para o diretório específico de cada projeto. Criamos um novo arquivo de configuração em /etc/apache2/sites-available/.

* sudo nano /etc/apache2/sites-available/site-clima.conf

Resolução de Erros de Sintaxe:
Ao tentar habilitar o site, o sistema apontou erros de configuração. Para diagnosticar com precisão, utilizou-se o comando: 

* sudo apache2ctl configtest

Erros identificados e corrigidos:

Case Sensitivity: A diretiva DocumentRoot estava escrita com "r" minúsculo.

Tag de fechamento: Faltava a barra de fechamento na tag </VirtualHost>.

Caracteres Inválidos: A palavra "Apache" foi inserida indevidamente no início do arquivo de configuração, fora das tags.

Após as correções, o configtest retornou Syntax OK.

 5. Ativação do Site e Testes
Com a sintaxe corrigida, procedemos com a desativação do site padrão (default) e ativação do novo projeto:

* sudo a2ensite site-clima.conf
* sudo a2dissite 000-default.conf
* sudo systemctl reload apache2

Validação:
Para testar o acesso localmente via terminal: curl http://localhost

Para validação externa, utilizamos uma VM Debian com interface gráfica e navegador Chrome, acessando o IP do servidor:

Também foi realizado o teste de conectividade via shell em outra máquina da rede para confirmar a disponibilidade do serviço.






