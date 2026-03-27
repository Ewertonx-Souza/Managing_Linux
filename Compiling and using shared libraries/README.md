Laboratório: Ciclo de Vida do Executável Linux (C, GCC e ELF)

Este laboratório documenta o processo de criação de código-fonte em linguagem C, a gestão de dependências e a compilação para um executável no formato ELF (Executable and Linkable Format).

 1. Fundamentos e Arquitetura de Bibliotecas

No ecossistema Linux, um executável resulta da compilação de código-fonte e do seu posterior vínculo (linking) a bibliotecas.

Bibliotecas Compartilhadas (.so): São objetos binários carregados em memória apenas quando necessários. Elas permitem que múltiplos programas utilizem o mesmo código simultaneamente, otimizando o uso de RAM e reduzindo o tamanho dos binários em disco.

Carregador Dinâmico (ld.so): Responsável por localizar e mapear essas bibliotecas no espaço de endereçamento do processo durante a execução.

 2. Estrutura de Diretórios de Bibliotecas

As bibliotecas do sistema seguem o padrão FHS (Filesystem Hierarchy Standard). Para identificar onde residem os objetos compartilhados de 64 bits:

/lib64: Link simbólico para /usr/lib64, contém bibliotecas essenciais para o boot.

/usr/lib/x86_64-linux-gnu: Diretório padrão em sistemas baseados em Debian/Ubuntu para bibliotecas de arquitetura específica.

Comandos de exploração:

cd /usr/lib/x86_64-linux-gnu - Navegar até o diretório de bibliotecas de 64 bits

![lib](../Imagens/shared_libraries/lib64.png)

# Listar arquivos de objetos compartilhados
ls -l | grep ".so"
3. Preparação do Ambiente de Build
Para transformar código em binário, é necessário instalar o conjunto de ferramentas de desenvolvimento (Toolchain).

Bash
sudo apt update
sudo apt install build-essential libgtk-3-dev libwebkit2gtk-4.1-dev pkg-config
Componentes Instalados:
build-essential: Inclui o gcc (compilador), make e as bibliotecas base da libc.

libgtk-3-dev & libwebkit2gtk-4.1-dev: Fornecem os headers (.h) e arquivos de desenvolvimento necessários para chamadas de funções gráficas e de renderização web.

pkg-config: Ferramenta indispensável que automatiza a inserção de flags de compilação e caminhos de bibliotecas para o compilador.

4. Gestão de Permissões
Para garantir que o processo de escrita e compilação ocorra sem conflitos de privilégios no diretório do projeto:

Bash
sudo chown -R $USER:$USER /home/ewerton/projeto
5. Desenvolvimento do Código-Fonte
O arquivo main.c utiliza a biblioteca GTK para a interface de janela e a WebKitGTK para renderizar o motor de busca do Google. O objetivo aqui é observar como o código faz referência a símbolos externos que serão resolvidos na compilação.

6. Compilação e Vinculação (Linking)
A compilação é realizada invocando o gcc e utilizando o pkg-config para resolver as dependências das bibliotecas gráficas.

Bash
gcc main.c -o navegador $(pkg-config --cflags --libs gtk+-3.0 webkit2gtk-4.1)
-o navegador: Define o nome do binário de saída.

--cflags: Inclui os caminhos dos cabeçalhos.

--libs: Indica quais bibliotecas o linker deve associar ao executável.

7. Execução e Análise de Dependências
Para executar aplicações gráficas a partir do terminal puro (TTY), é necessário subir o servidor X ou Wayland:

Bash
sudo systemctl isolate graphical.target
./navegador
Inspeção de Dependências Dinâmicas
Para validar quais bibliotecas o executável "navegador" está solicitando ao sistema em tempo de execução, utilizamos o comando ldd:

Bash
ldd /home/ewerton/projeto/navegador
Este comando exibirá o mapeamento de cada biblioteca .so necessária e o endereço de memória onde o carregador planeja encontrá-la.


