Laboratório: Desabilitar a interface gráfica Devuan

Esse laboratório descreve o processo e analise de como funciona o SystemV init e seus runlevels. 

 1. Conceitos fundamentais: O systemV é o primeiro processo (PID 1) executado e atráves dele os demais processos e serviços são executados e gerenciados. Quando o Kernel invoca o programa localizado em /sbin/init, o systemV localiza o diretório /etc/inittab e executa os scripts ali descritos. Esse script define qual runlevel será utilizado e seus serviços necessários. Runlevels resumidamente são estados operacionais (iniciando, multusuário, gráfico, etc.) do sistema. Eles agrupam serviços para definir o que carregar, sendo númerados de 0 a 6 e através de links simbólicos, oferecem mais flexibilidade para organizar a inicialização e os serviços.

Para visualizar o contéudo dentro de /etc/inittab utiliza-se o comando:

* sudo nano /etc/inittab ou cat /etc/inittab

 ![nano](../Imagens/Runlevel-DisplayManager/inittab.png)


 2. Análise e entendimento da saída nano/cat. Com base na execução do comando, identificamos as seguintes syntaxes:

id:2:initdefault: - Informa ao systemv qual será o runlevel padrão atual.

si::sysinit:/etc/init.d/rcS - O rcS serve como o primeiro script de controle executado pelo processo init. Sendo responsável por configurar o ambiente básico antes que o usuário possa interagir com o sistema.

12:2:wait:/etc/init.d/rc 2 - rc 2 é o runlevel na qual vai ser executado seus arquivos de serviços que definem o ambiente. 

Quando o systemV olha para o /etc/init.d/rc 2, ele entende que deve ir até o diretório /etc/rc2.d e executar os scripts ali encontrados. Mas essa execução não é deliberada, existe ordens do que "matar" e do que deve iniciar. Com o comando:

* ls -l /etc | grep rc

É possível ver todos os runlevels e dentro deles possuem os arquivos de serviços necessários para ambiente, definido pelo administrador .

![rc.d](../Imagens/Runlevel-DisplayManager/rc.d.png)

rc2.d (Runlevel atual):

![linksym](../Imagens/Runlevel-DisplayManager/rc2.d_linksym.png)

Dentro de rc2.d é possível visualizar todos os arquivos de serviço. Pode-se perceber que o nome dos arquivos começam com K (kill)e depois mudam para S (Start). Isso significa que o systemV identifica primeiro os serviços que NÃO devem ser executados e depois os que devem inicializar. Esses arquivos também segue uma ordem de numeração, qual é do menor para o maior. Por exemplo a ordem do rc2.d começa com 01, 02, 03, 04 e 05. Mas um ponto muito importante desses arquivos é que todos eles são links simbólicos. Esses links simbólicos direcionam o sistema para o arquivo de serviço real, que fica localizado em /etc/init.d/*.

 3. Identificando runlevel atual:

* sudo runlevel

![runlevel](../Imagens/Runlevel-DisplayManager/current_runlevel.png)

A sáida do comando informa que o runlevel atual é o 2, conforme descrito também na syntaxe do /etc/inittab. 

 4. Alterando runlevel atual:

* sudo init 3

![change](../Imagens/Runlevel-DisplayManager/change_runlevel.png)

A saída do comando runlevel mostra 2 e 3. Isso indica que foi feita a mudança de runlevel. No Debian/Devuan, os runlevels 2, 3, 4 e 5 são praticamente iguais por padrão. O motivo da interface gráfica ainda está habilitada é que no Debian/Devuan o display manager (ex: lightdm, gdm, slim etc) é iniciado no runlevel padrão através dos scripts em /etc/rc2.d/. 

 5. Veja qual display manager está instalado:

* ls /etc/init.d | grep -E "gdm|lightdm|slim|xdm"

![change](../Imagens/Runlevel-DisplayManager/check_displaymanager.png)

Conforma a saída do comando executado acima, o display manager nessa distro é o Slim. 

Agora, identificado o display manager, deve ser localizado o os runlevels que executando ele utilizando o comando:

* ls etc/rc2.d | grep slim
* ls etc/rc3.d | grep slim

![change](../Imagens/Runlevel-DisplayManager/verify_displaymanager.png)

Nessa caso, ambos runlevels executando o arquivo de serviço slim, então os dois ambientes incializarão com a interface grafica habilita. 

 6. Desabilitando interface gráfica utilizando o comando:

* sudo update-rc.d slim disable
  
![disable](../Imagens/Runlevel-DisplayManager/disable_displaymanager.png)

Com esse comando, o serviço slim localizado em /etc/init.d/slim será desativado. É possível validar a desativação do serviço identificando como está a letra (S ou K) dentro do diretório do runlevel. Para isso, é utilizado o comando ls novamente:

* ls etc/rc2.d | grep slim
* ls etc/rc3.d | grep slim

!killer](../Imagens/Runlevel-DisplayManager/killer_displaymanager.png)

Como o nome do arquivo inicia com K, é constatado que o serviço não será executado na incialização. 

Após o reboot do sistema, ela inciará como modo texto.

![display](../Imagens/Runlevel-DisplayManager/no_display.png)

Acessado /etc/inittab sem interface gráfica:

![still](../Imagens/Runlevel-DisplayManager/still_runlevel2.png)

Nota técnica: Como o serviço slim em /etc/init.d/slim foi desativado, consequentemente todos os runlevels que possuem link simbólico direcionando para esse caminho, irão incializar sem interface gráfica. 

 7. Destivando a interface gráfica apenas de um runlevel.

Para desativar a interface gráfica apenas de um único runlevel, se utiliza o comando: 

* sudo update-rc.d slim disable - ativar display manager.
* ls etc/rc3.d | grep slim e ls etc/rc2.d | grep slim - Verificar se ambos runlevels estão com o serviço habilitado novamente.
* sudo update-rc.d slim disable 3 - Desabilitar o serviço slim de apenas um runlevel.

![change single](../Imagens/Runlevel-DisplayManager/change_single_runlevel3.png)

Após essas alterações, ao dar reboot na máquina com ela no runlevel 2 (alterar em /etc/inittab), ela iniciará com a interface gráfica. 

![up](../Imagens/Runlevel-DisplayManager/enable_displaymanage_runlevel2.png)

confirmando runlevel:

![up](../Imagens/Runlevel-DisplayManager/display_up_N2.png)

Alterando /etc/inittab novamente para runlevel 3 e reiniciando a máquina, ela inciará em modo texto.

![down](../Imagens/Runlevel-DisplayManager/display_down_N3.png)
