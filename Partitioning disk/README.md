Nesse homelab, será criado uma partição para o /boot, pois na instalação do sistema operacional Linux, não foi feito isso. A não criação foi proposital para que seja utilizado métodos necessários para a criação já com o SO em execução. Para isso, foi necessário a utilização do LIVECD do Linux, que é o sistema sendo executado na memória, mas não está no dispositivo de armazenamento. O virtualizador utilizado para criar a VM foi o Proxmox.



Abaixo temos a informação do sistema operacional principal e suas partições:



!\[lsblk1](../Imagens/lsblk\_vm1.png)



sda1 - Bios\_boot

sda2 - Root filesystem



Pode-se perceber que não existe partição /boot. /boot é apenas um diretório comum dentro do diretório raiz /.



Por que criar uma partição para o /boot?



* Flexibilidade

Caso a partição raiz (/) for criptografada e o boot estiver dentro dele, o sistema não consegue carregar o kernel antes da descriptografia. Agora, se o /boot estiver separado e sem criptografia, GRUB (bootloader) pode carregar os arquivos essenciais.



* Segurança

Se ocorrer alguma corrupção do root filesystem, o /boot separado permite iniciar o sistema em modo recuperação.



* Gerenciamento de Kernels

Permite ter vários kernels instalados sem conflitos.



* Facilidade em Dual-Boot

Em situações com outros sistemas ou partições complexas, igual ZFS em LVM, o /boot compacto e simples (ext4) previne problemas de compatibilidade do bootloader.



* Diminuir complicação

Mantém o processo de inicialização mais simples e otimizado, com tabelas de arquivos menores, diminuindo a pressão sobre a RAM inicial.



Por que durante a instalação eu consigo particionar, mas depois preciso de LIVECD?



* Durante a instalação está montada como sistema ativo
* O disco não está em uso
* O instalador roda em um ambiente live (RAM)



Ou seja, na pratica, o instalador já é um LIVECD automatizado com o disco totalmente livre para ser alterado. Agora, quando o Linux está instalado e rodando:



* A partição raiz (/) está montada
* O sistema de arquivo está em uso
* Binários, bibliotecas e processos estão acessando o disco
* O kernel bloqueia operações destrutivas em partições montadas



Exemplo:



Não é possível dimensionar, mover ou recriar:

* /
* /boot (se existir)
* Partições ativas



Isso causaria corrupção imediata do filesystem.



Por que especificamente o /boot exige LIVECD?



O /boot contém:



• Kernel (vmlinuz)

• Initramfs

• GRUB

• Mapas de boot



Durante o boot normal:



• O kernel foi carregado a partir do /boot

• O GRUB depende desses arquivos

• O filesystem precisa estar 100% consistente



Então:



Criar /boot separado

Redimensionar partição raiz

Mover blocos no disco



Somente com o sistema desligado, via LIVECD.



Agora, já esclarecido as questões sobre o /boot e filesystem, seguiremos o processo de criação do /boot. Utilizando o Proxmox, foi necessário criar uma outra VM com a imagem ISO do Ubuntu server, porém sem adicionar um disco para ela.



!\[rescue](../Imagens/vm\_rescue\_no\_disk.png)



VM criada sem disco, pois será anexado nela a imagem de disco da VM principal, na qual queremos criar a partição SDA3. Com a VM rescue criada e sem disco, será utilizado o shell do node que as VM's estão hospedadas.



Os comandos usado para isso foi os seguintes:



* qm set 101 -scsi1 local-lvm:vm-100-disk-0 - Anexar o disco
* qm set 101 -delete scsi1 - Remover disco, se necessário
* qm config 101 - Ver as configurações



!\[anexo](../Imagens/anexo\_VM1xVM2.png)



Dessa forma, com a VM rescue criada e anexado o disco a VM principal, a VM deve ser iniciada em modo LIVECD  e executado os comandos necessários. Ela será iniciada normalmente com um instalador, clicar em "Try or Install Ubuntu" e quando aparecer a opção de idioma, pressionar Alt+Ctrl+f2 e a máquina entrará em modo LIVECD. A partir disso, pode começar utilizar os comandos.



* lsblk -f - visualizar o disco e partições



!\[rescuelsblk](../Imagens/lsblk\_rescue.png)



Na imagem, é possível ver que  os mesmo discos e partições da VM rescue e o da VM principal.

