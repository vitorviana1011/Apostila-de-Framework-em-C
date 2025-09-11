---
layout:
  width: wide
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: false
  metadata:
    visible: true
---

# Instalacao Windows

### Passo 1: Baixando a Biblioteca

Acesse o link: [https://github.com/raysan5/raylib/releases](https://github.com/raysan5/raylib/releases)\
Procure a versao mais recente para o seu Windows. (Atencao para se seu sistema 'e 32 ou 64 bits)

### Passo 2: Instalando a Biblioteca

Execute o .exe que baixou, ele vai pedir um diretorio para baixar a biblioteca. 'E receomendado baixa-la na pasta `C:/raylib/`

### Passo 3: Path

1. Adicionando o Compilador ao PATH do Sistema:
   * Pressione a tecla Windows e digite `variáveis de ambiente`.
   * Clique na opção "Editar as variáveis de ambiente do sistema".
   * Na janela que abrir, clique no botão "Variáveis de Ambiente...".
   * Na seção de baixo ("Variáveis do sistema"), encontre a variável chamada `Path` e clique duas vezes sobre ela.
   * Na nova janela, clique em "Novo".
   * Cole o seguinte caminho exatamente como está: `C:\raylib\mingw\bin`
   * Clique em OK em todas as janelas para salvar as alterações.
2. Verificando a Configuração:
   * Para que a mudança no PATH tenha efeito, você precisa abrir um novo terminal.
   * Abra o menu Iniciar, digite `cmd` ou `powershell` e abra um novo prompt de comando.
   * Digite o seguinte comando e pressione Enter: `gcc --version`
   * Resultado esperado (Sucesso): O terminal deve exibir a versão do compilador `gcc`, algo como `gcc.exe (RevX, Built by MSYS2 project) X.X.X`.
   * Resultado de erro: Se você receber uma mensagem como `'gcc' não é reconhecido como um comando interno ou externo...`, algo deu errado na configuração do PATH. Verifique o caminho que você adicionou e reinicie o terminal.

### Passo 4: Criando e Compilando um Exemplo no Windows

Com o ambiente configurado, vamos criar, compilar e rodar nosso primeiro programa diretamente pelo terminal.

### 1. Crie o Arquivo de Exemplo:

Abra a pasta `C:\raylib` no VS Code. Crie um novo arquivo chamado `exemplo.c` e cole o seguinte código. Note que alteramos o título da janela para refletir o sistema operacional.

```
#include "raylib.h"

int main(void)
{
    // Inicialização
    const int screenWidth = 800;
    const int screenHeight = 450;
    InitWindow(screenWidth, screenHeight, "Olá, Raylib no Windows!"); // Título alterado

    SetTargetFPS(60); // Define o nosso jogo para rodar a 60 frames por segundo

    // Loop principal do jogo
    while (!WindowShouldClose())    // Detecta se o usuário fechou a janela
    {
        // Desenhar
        BeginDrawing();
            ClearBackground(RAYWHITE);
            DrawText("Olá, Raylib no Windows!", 240, 200, 20, DARKGRAY);
            DrawCircle(400, 300, 50, MAROON);
            DrawRectangle(600, 150, 80, 80, DARKBLUE);
        EndDrawing();
    }

    // Finalização
    CloseWindow(); // Fecha a janela e o contexto OpenGL

    return 0;
}
```

Abra o terminal. Com nosso código de exemplo pronto, vamos compilá-lo, fazendo a "ligação" (link) com a biblioteca Raylib.

```
gcc exemplo.c -o exemplo.exe -I C:\raylib\raylib\src -L C:\raylib\raylib\src -lraylib -lopengl32 -lgdi32 -lwinmm
```

Se tudo correu bem, nenhum erro aparecerá. Agora, vamos executar nosso programa! Lembre-se que no Windows, programas executáveis terminam com `.exe`.

```
./exemplo.exe
```

O programa está rodando! Uma janela deve ter aparecido na sua tela mostrando o texto "Olá, Raylib no Windows!" junto com um círculo e um retângulo.

## Passo 5: Criando um `Makefile` para Windows

Digitar aquele comando `gcc` gigante toda vez é cansativo. Vamos criar um `Makefile` para automatizar a compilação. A ferramenta `make` funciona porque ela veio junto com o pacote MinGW que instalamos!

Crie um arquivo chamado `Makefile.win` e cole o seguinte conteúdo nele:

```
# Define o compilador que estamos usando
CC = gcc

# Nome do arquivo de saída (executável)
# Use: make NOME=meu_jogo
# O padrão será "programa.exe"
NOME ?= programa

# Diretório da Raylib (caminho absoluto para evitar erros)
RAYLIB_PATH = C:/raylib/raylib/src

# Flags de compilação para Windows
# As bibliotecas -lopengl32, -lgdi32, -lwinmm são essenciais para Windows
CFLAGS = -I$(RAYLIB_PATH) -L$(RAYLIB_PATH) -lraylib -lopengl32 -lgdi32 -lwinmm

# Regra principal: compila o programa, adicionando .exe ao final
all:
	$(CC) $(NOME).c -o $(NOME).exe $(CFLAGS)

# Regra para limpar os arquivos compilados (usa o comando 'del' do Windows)
clean:
	del /F /Q $(NOME).exe
```

#### Compilando de Forma Simples

Agora você pode compilar seus projetos de duas maneiras:

1\. Com o Comando Completo:

```
gcc meu_programa.c -o meu_programa.exe -I C:\raylib\raylib\src -L C:\raylib\raylib\src -lraylib -lopengl32 -lgdi32 -lwinmm
```

2\. Usando o `Makefile` que criamos (Recomendado):

Supondo que seu arquivo se chame `meu_programa.c`.

```
# Copie nosso modelo de Makefile para o nome padrão "Makefile"
# No Windows, o comando para copiar é 'copy'
copy Makefile.win Makefile

# Compile passando o nome do seu arquivo (sem o .c)
# O Makefile já adicionará o .exe automaticamente
make NOME=meu_programa

# Execute
./meu_programa.exe
```
