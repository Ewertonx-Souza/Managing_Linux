Laboratório: Desabilitar a interface gráfica Devuan

Esse laboratório descreve o processo e analise de como funciona o System V init e seus runlevels. 

 1. Conceitos fundamentais: O system V é o primiero processo (PID 1) executado e atráves dele os demais processos e serviços são gerenciados. No system V, quando o Kernel invoca o programa localizado em /sbin/init, o programa localiza o diretório /etc/inittab e executa os scripts ali descritos.

Para visualizar o contéudo dentro de /etc/inittab utiliza-se o comando:

* sudo nano /etc/inittab ou cat /etc/inittab

 ![nano](../Imagens/Runlevel-displayManager/inittab.png)
