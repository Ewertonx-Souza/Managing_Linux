Laboratório: Desabilitar a interface gráfica Devuan

Esse laboratório descreve o processo e analise de como funciona o SystemV init e seus runlevels. 

 1. Conceitos fundamentais: O systemV é o primiero processo (PID 1) executado e atráves dele os demais processos e serviços são executados e gerenciados. Quando o Kernel invoca o programa localizado em /sbin/init, o systemV localiza o diretório /etc/inittab e executa os scripts ali descritos. Esse script define qual runlevel será utilizado e seus serviços necessários. Runlevels resumidamente são estados operacionais (iniciando, multusuário, gráfico, etc.) do sistema. Eles agrupam serviços para definir o que carrega, sendo númerados de 0 a 6 e através de links simbólicos, oferecem mais flexibilidade para organizar a inicialização e os serviços.

Para visualizar o contéudo dentro de /etc/inittab utiliza-se o comando:

* sudo nano /etc/inittab ou cat /etc/inittab

 ![nano](../Imagens/Runlevel-DisplayManager/inittab.png)


 2. Análise da Saída nano/cat. Com base na execução do comando, identificamos as seguintes syntaxe:

id:2:initdefault: - Informa ao systemv quando será o runlevel padrão atual.

si::sysinit:/etc/init.d/rc2 - O rcS serve como o primeiro script de controle executado pelo processo init. Sendo responsável por configurar o ambiente básico antes que o usuário possa interagir com o sistema.

12:2:wait:/etc/init/rc 2 - rc 2 é o runlevel na qual vai ser executado seus arquivos de serviços que definem o ambiente. 

Quando o systemV executa o rc 2, ele entende que deve ir até o diretório /etc/rc2.d e executar os scripts ali encontrados. Mas essa execução não é deliberada, existe ordens do que "matar" e do iniciar. Com o comando:

* ls -l /etc | grep rc

É possível ver todos os runlevels e dentro deles possuem os arquivos de serviços necessários para ambiente, definido pelo administrador .

![preparando imagem](../Imagens/Runlevel-isplayManager/inittab.png)


