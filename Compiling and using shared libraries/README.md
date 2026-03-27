Laboratório: Craindo código fonte em C e compilando

Esse labortário descreve o funcionamento da criação de um código fonte e como ele é tranformado em um executável ELF.

1. Conceitos fundamentais: Todo executável no Linux é um código fonte que foi compilado e linkado a bibliotecas compartilhadas ou estáticas. Uma biblioteca compartilha é um conjunto de código pré-compilado reutilizável, carregado na memória e utilizado por múltiplos programas simultaneamente durante a execução. Dessa forma, o executável não fica pesado, por não ter muitas bibliotecas introduzidas nele e o sistema tem uma otimização de recursos. Essas bibliotecas compartilhadas ficam disponíveis em diretórios específicos para que o carregador dinâmico (ld.so ou ld-linux.so) possa localizar e carregá-las na memória.

 2. IDENTIFICANDO BIBLIOTECAS COMPARTILHADAS 

Para visualizar essas bibliotecas compartilhadas, pode-se olhar nos seguintes diretórios:

/lib - Link simbólico para /usr/lib.
/usr/lib - bibliotecas específicas do sistema.
/lib64 - Link simbólico para /usr/lib64.
/usr/lib64 - possui um link simbólico para o carregador dinâmico de 64 bits. 
 
* cd /usr/lib/x86_64-linux-gnu - Navedar no diretório que possui arquivos .so de 64 gits

(imagem)

* ls -l - Visualizar os arquivos .so

(imagem) 

 3. PREPARANDO AMBIENTE BUILD
 
Agora que foi identificado a localização das bibliotecas compartilhadas, será instalado no sistema ferramentas necessárias para um ambiente de build. 

* sudo apt install build-essential libgtk-3-dev libwebkit2gtk-4.1-dev

- build-essential
---gcc (compilador C)
---make
---libc dev
Necessário para compilar qualquer programa em C.

- libgtk-3-dev
---Headers (.h)
---Bibliotecas de desenvolvimento
Necessário para include.

- libwebkit2gtk-4.1-dev
---Headers (.h)
---Bibliotecas de desenvolvimento
Necessário também para include.

- pkg-config
Ferramenta que resolve:
includes
libs
flags de compilação

(imagem)

 4. PERMISSÃO DE USÚARIO 

Foi dada as permissões necessárias para o usuário, pois o caminho /home/ewerton/projeto é pertencente ao root.

* sudo chown -R ewerton:ewerton /home/ewerton/projeto.

(imagem)  

 5. CRIAÇÃO DO CÓDIGO FONTE

Após as instalações dos pacotes necessários para a compilação e dados as permissões necessárias para escrever no diretório, será criado um código fonte simples utilizando GTK (biblioteca gráfica) e webkitGTK (gera o conteúdo web dentro de GTK). O local na qual ficou armazenado o código fonte, foi em /home/ewerton/projeto/main.c. Esse código fonte consiste em criar a interface gráfica e dentro dela ser exibido o google.com. Esse site foi desenvolvido de forma simples apenas para fins de estudos, pois o foco é a compilação e vinculação com as bibliotecas compartilhadas. 

(imagem)

Código fonte criado:

(imagem)

 6. COMPILANDO CÓDIGO FONTE

Criado arquivo .c com código fonte, dado as permissões necessárias para o diretório, é só executar o comando de compilação do script.

* gcc main.c -o navegador $(pkg-config --cflags --libs gtk+-3.0 webkit2gtk-4.1)

(imagem)

Após a execução, dentro do mesmo diretório terá um binário com o nome de "navegador". Esse é o executável criado após a instalção. Para testar o executável, é necessário acessar a interface gráfica do sistema. Dessa forma, será alterado a target do sistema com:

* sudo systemctl isolate graphical.target

E executar o programa e o resultado é o google aparecendo sem a necessidade de utilizar o browser:

(imagem)

ainda no shell linux, é possivel utilizar um comando para poder visualizar todas as bibliotecas que agora o executável faz referências para que ele funcione?

* ldd /home/ewerton/projeto/navegador

(imagem)


