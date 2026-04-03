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

ETAPA 1 - COPIANDO A BIBLIOTECA SEM SYMLINK

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

3.3 - Registrar diretório

Agora que a biblioteca foi copiada para um novo diretório, é necessário incluir e atualizar esse caminho em um arquivo de configuração para que o carregador dinâmico consiga localizar esse novo endereço no disco. Esse arquivo de configuração é o /etc/ld.so.conf.d. É nele que é feito a inclusão e atualização de diretórios, a partir disso o ldconfig faz um scan em todos os diretórios que possuem bibliotecas compartilhadas, inclusive no arquivo de configuração /etc/ld.so.conf.d e redefine o /etc/ld.so.cache, um diretório binário na qual o carregador dinâmico acessa para carregar as bibliotecas necessárias para o programa que foi executado. 

* echo "/opt/mylib" > /etc/ld.so.conf.d/mylib.conf

![echo](../Imagens/dynamic_linker/echo.png)

* ldconfig
* ldd /home/ewerton/projeto/navegador

![lddnew](../Imagens/dynamic_linker/new_ldd.png)

 4. Resultado observado

O sistema inicialmente reconhece /usr/lib/x85_64-linux-gnu/libwebkit2gtk-4.1.so.0, Porém ao copiar libwebkit2gtk-4.1.so.0 para /opt/mylib e incluir em /etc/ld.so.conf.d/ um outro caminho a ser seguido para encontrar a biblioteca e utilizar o ldconfig, que é responsável por criar, atualizar e gerenciar links simbólicos e o cache (/etc/ld.so.cache) das bibliotecas compartilhadas (.so) no sistema. Quando é executado novamente o comando ldd, ele trás um outro caminho que agora o carregador dinâmico enxerga em /etc/ld.so.cache.

Nota técnica: O arquivo libwebkit2gtk-4.1.so.0 é um link simbólico que aponta para o binário real (libwebkit2gtk-4.1.so.0.19.9). Ao usar o comando cp sem parâmetros adicionais, ele segue o link e copia o conteúdo do arquivo original. Como resultado, em /opt/mylib, você terá um arquivo real nomeado como libwebkit2gtk-4.1.so.0, em vez de manter a estrutura de link simbólico original.

ETAPA 2 - COPIANDO A BIBLIOTECA COM SYMLINK

 1. Cenário inicial

O ambiente segue sendo o mesmo da etapa anterior e o sistema ainda reconhece /opt/mylib como caminho a ser seguido para encontrar a biblioteca compartilhada que o programa "navegador" precisa. 

 2. Hipótese

Se a biblioteca for movida para /opt com o parâmetro -P e registrada no ld.so.conf.d, o sistema deve priorizar esse caminho.

 3. Procedimento

3.1 - Apagar a biblioteca que está em /opt/mylib

* sudo rm /opt/mylib/*

![rm](../Imagens/dynamic_linker/rm_library.png)

3.2 - Copiar a biblioteca para /opt/mylib (com symlink)

* cp -P /usr/lib/x85_64-linux-gnu/libwebkit2gtk-4.1.so.0 /opt/mylib

![cp-p](../Imagens/dynamic_linker/cp-p.png)

3.3 - Registrar diretório

O diretório /opt/mylib já consta dentro de /etc/ld.so.conf.d/.

* ls -l /etc/ld.so.conf.d/mylib.conf

![ls](../Imagens/dynamic_linker/ls-confd.png)

3.4 Atualizar symlink 

* sudo ldconfig

![ld](../Imagens/dynamic_linker/ldconfig-p.png)

3.5 - Validando busca do carregador dinâmico

* ldd /home/ewerton/projeto/navegador | grep libwebkit

![ldd](../Imagens/dynamic_linker/ldd_navegador.png)

 4. Resultado observado

Ao utilizar o comando cp -P, o link simbólico libwebkit2gtk-4.1.so.0 é copiado para /opt/mylib preservando sua natureza de "atalho". No entanto, isso gera um erro de resolução no gerenciamento de bibliotecas:

--O Problema do Link Órfão: Como o binário real (libwebkit2gtk-4.1.so.0.19.9) não foi copiado junto para /opt/mylib, o link simbólico em seu novo destino aponta para um caminho inexistente naquele diretório.
--Ação do ldconfig: Durante o escaneamento, o ldconfig identifica que o link em /opt/mylib está "quebrado" (aponta para o vazio). Por segurança e integridade do cache, ele ignora esse caminho inválido.
--Persistência do Cache: Como a entrada em /opt/mylib foi descartada, o ldconfig mantém no /etc/ld.so.cache apenas o caminho original e funcional (geralmente em /usr/lib/x86_64-linux-gnu/).
--Resultado: O carregador dinâmico ignora a tentativa de redirecionamento para /opt/mylib e continua utilizando a biblioteca padrão do sistema.

ESCLARECIMENTO TÉCNICO 

O motivo da segunda etapa ocorrer o erro de resolução no gerencimaneto de bibliotecas é porque o symlink não guarda automaticamente o caminho absoluto original. Ele guarda exatamente o texto que foi usado na criação. Se executado o comando:

* ls -l /usr/lib/x86_64-linux-gnu/libwebkit2gtk-4.1.so.0

![ls](../Imagens/dynamic_linker/lib19_9.png)

É possível observar que o libwebkit2gtk-4.1.so.0 aponta para libwebkit2gtk-4.1.so.0.19.9, ou seja:

-NÃO aparece /usr/lib/...
-Só o nome do arquivo

Esse é o ponto crítico para resolução no gerenciamento de bibliotecas da errado. O symlink libwebkit2gtk-4.1.so.0 -> libwebkit2gtk-4.1.so.0.19.9 é um caminho relativo, sem o caminho total desse arquivo real. Dessa forma, o sistema entende que o arquivo real se enocntra no mesmo diretório do symlink, que é /usr/lib/x86_64-linux-gnu/. Quando o symlink alternativo (libwebkit2gtk-4.1.so.0) vai para /opt/mylib e o arquivo real não vai junto, então quando o sistema procura por /opt/mylib/libwebkit2gtk-4.1.so.0.19.9 e não encontra, ele ignora essa entrada e segue o caminho normal e confiável (/usr/lib/x86_64-linux-gnu/). 

Validando informação da biblioteca libwebkit2gtk-4.1.so.0 usando o comando:

* readlink readlink /usr/lib/x86_64-linux-gnu/libwebkit2gtk-4.1.so.0

![echo](../Imagens/dynamic_linker/readlink.png)

A saída do comando é apenas "libwebkit2gtk-4.1.so.0.19.9", isso mostra que o symlink não sabe o caminho absoluto do arquivo real, apenas qual é o arquivo. Por ser um symlink alternativo, o sistema entende que o arquivo real está localizado no mesmo diretório do arquivo symlink. 


