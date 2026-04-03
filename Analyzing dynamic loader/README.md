Laboratório: Analisando o funcionamento do carregador dinâmico Linux
 
Neste laboratório iremos analisar como o carregador dinâmico resolve bibliotecas compartilhadas em tempo de execução, com foco em symlinks e diretórios alternativos.

Fundamentos:

O carregador dinâmico no Linux (ld.so ou ld-linux.so) é o componente do sistema operacional responsável por preparar um programa para execução, carregando as bibliotecas compartilhadas (arquivos .so - Shared Objects) de que ele precisa na memória. Diferente dos programas vinculados estaticamente, que contêm todo o código necessário, a maioria dos binários Linux utiliza vinculação dinâmica para otimizar espaço em disco e memória, dependendo do carregador para resolver essas dependências no momento em que o programa é iniciado.

Ambiente:

Sistema: 
* Linux (Debian)
* Arquitetura: x86_64
* Biblioteca analisada: libwebkit2gtk-4.1.so.0

Ferramentas:
* ldd
* ldconfig
* readlink
* ls
* cp

ETAPA COPIANDO A BIBLIOTECA SEM SYMLINK

 1. Cenário inicial

Foi criado um código em C e compilado para se tornar um executável ELF, conforme documentado em "Compiling and using shared libraries", após toda a compilação, esse código fonte, que agora é um executável, passou a ter dependencias de bibliotecas compartilhadas disponíveis no sistema operacional. Das várias bibliotecas das quais o binário precisa, uma delas é a libwebkit2gtk-4.1.so.0. 

A estrutura original dessa biblioteca é:

/usr/lib/x86_64-linux-gnu/ - path

libwebkit2gtk-4.1.so.0 -> libwebkit2gtk-4.1.so.0.x.y (arquivo real) 

![ldd](../Imagens/dynamic_linker/ldd_navegador.png)

 2. Hipótese

Se a biblioteca for movida para /opt e registrada no ld.so.conf.d, o sistema deve priorizar esse caminho.

 3. Procedimento

3.1 - Criar um diretório em /opt

* sudo mkdir /opt/mylib

![mkdir](../Imagens/dynamic_linker/mkdir.png)

3.2 - Copiar a biblioteca para /opt/mylib (sem symlink)

* cp /usr/lib/x85_64-linux-gnu/libwebkit2gtk-4.1.so.0 /opt/mylib

![cp](../Imagens/dynamic_linker/cp_lib.png)

3.3 Registrar diretório

Agora que a biblioteca foi copiada para um novo diretório, é necessário incluir e atualizar esse caminho em um arquivo de configuração para que o carregador dinâmico consiga localizar esse novo endereço no disco. Esse arquivo de configuração é o /etc/ld.so.conf.d. É nele que é feito a inclusão e atualização de diretórios, a partir disso o ldconfig faz um scan em todos os diretórios que possuem bibliotecas compartilhadas, inclusive no arquivo de configuração /etc/ld.so.conf.d e redefine o /etc/ld.so.cache, um diretório binário na qual o carregador dinâmico acessa para carregar as bibliotecas necessárias para o programa que foi executado. 

* echo "/opt/mylib" > /etc/ld.so.conf.d/mylib.conf

![echo](../Imagens/dynamic_linker/echo.png)

* ldconfig
* ldd /home/ewerton/projeto/navegador

![lddnew](../Imagens/dynamic_linker/new_ldd.png)

 4. Resultado observado

O sistema inicialmente reconhece /usr/lib/x85_64-linux-gnu/libwebkit2gtk-4.1.so.0, Pórem ao copiar libwebkit2gtk-4.1.so.0 para /opt/mylib e incluir em /etc/ld.so.conf.d/ um outro caminho a ser seguido para encontrar a biblioteca e utilizar o ldconfig, que é responsável por criar, atualizar e gerenciar links simbólicos e o cache (/etc/ld.so.cache) das bibliotecas compartilhadas (.so) no sistema. Quando é executado novamente o comando ldd, ele trás um outro caminho que agora o carregador dinâmico enxerga em /etc/ld.so.cache.







