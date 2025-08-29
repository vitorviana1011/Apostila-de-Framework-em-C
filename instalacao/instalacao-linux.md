---
description: >-
  Este tutorial mostra como instalar a Raylib em um ambiente Linux (baseado em
  Debian/Ubuntu) compilando-a diretamente do código-fonte. Isso garante que você
  tenha a versão mais recente e um controle to
---

# Instalacao Linux

***

### Passo 1: Instalar Dependências do Sistema

Primeiro, vamos instalar todas as ferramentas e bibliotecas necessárias para compilar a Raylib.

Abra o terminal e execute o comando:

```
sudo apt update && sudo apt install -y build-essential git cmake libglfw3-dev libgl1-mesa-dev libxcursor-dev libxinerama-dev libxrandr-dev libxi-dev libasound2-dev
```

### Passo 2: Baixar o Código-Fonte da Raylib

Agora, vamos baixar a versão mais recente da Raylib diretamente do GitHub.

```
git clone https://github.com/raysan5/raylib.git
```

### Passo 3: Compilar a Raylib

Com o código-fonte baixado, vamos entrar na pasta e compilar a biblioteca. Usaremos o método `make` tradicional, que é mais simples.

```
cd raylib/src && make
```

Ao final deste processo, você terá o arquivo `libraylib.a` dentro da pasta `src`, que é a nossa biblioteca compilada.

Com isso voce ja deve ter o suficiente para comecar a utilizar do raylib, porem recomendo testar com o codigo exemplo a seguir. Se voce nao tiver interesse, aconselho pular ao passo 6, onde criaremos um makefile para facilitar a compilacao.

### Passo 4: Exemplo (Opcional)

Vamos voltar para a pasta anterior (onde a pasta `raylib` está) e criar um arquivo de teste.

```
cd ..
```

Agora, crie um arquivo chamado `exemplo.c`

```
#include "raylib.h"

int main(void)
{
    // Inicialização
    const int screenWidth = 800;
    const int screenHeight = 450;
    InitWindow(screenWidth, screenHeight, "Olá, Raylib no Linux!");

    SetTargetFPS(60); // Define o nosso jogo para rodar a 60 frames por segundo

    // Loop principal do jogo
    while (!WindowShouldClose())    // Detecta se o usuário fechou a janela
    {
        // Desenhar
        BeginDrawing();
            ClearBackground(RAYWHITE);
            DrawText("Olá, Raylib no Linux!", 240, 200, 20, DARKGRAY);
            DrawCircle(400, 300, 50, MAROON);
            DrawRectangle(600, 150, 80, 80, DARKBLUE);
        EndDrawing();
    }

    // Finalização
    CloseWindow(); // Fecha a janela e o contexto OpenGL

    return 0;
}
```

### Passo 5: Compilar e Testar

Com o nosso código de exemplo pronto, vamos compilá-lo, fazendo a "ligação" (link) com a biblioteca Raylib que geramos.

```
gcc exemplo.c -o exemplo_simples -I./raylib/src -L./raylib/src -lraylib -lm -lpthread -ldl -lrt -lX11
```

Se tudo correu bem, nenhum erro aparecerá. Agora, vamos executar nosso programa!

```
./exemplo_simples
```

O programa está rodando! Uma janela deve ter aparecido na sua tela mostrando o texto "Olá, Raylib no Linux!" junto com um círculo e um retângulo.

### Passo 6: Criando um Makefile

Digitar aquele comando `gcc` gigante toda vez é cansativo. Vamos criar um `Makefile` genérico para automatizar a compilação de qualquer projeto futuro.

Crie um arquivo chamado `Makefile.projeto` e cole o seguinte conteúdo nele:

```
# Define o compilador
CC = gcc

# Nome do arquivo de saída (executável)
# Use: make NOME=meu_jogo
NOME ?= programa

# Diretório da Raylib
RAYLIB_DIR = ./raylib

# Flags de compilação
# -I diz ao compilador onde procurar os arquivos de cabeçalho (.h)
# -L diz ao vinculador onde procurar as bibliotecas (.a)
# -l são as bibliotecas a serem vinculadas
CFLAGS = -I$(RAYLIB_DIR)/src -L$(RAYLIB_DIR)/src -lraylib -lm -lpthread -ldl -lrt -lX11

# Regra principal: compila o programa
all:
	$(CC) $(NOME).c -o $(NOME) $(CFLAGS)

# Regra para limpar os arquivos compilados
clean:
	rm -f $(NOME)
```

#### Compilando:

Agora podemos compilar usando dois metodos. Com o comando completo:

```
gcc meu_programa.c -I./raylib/src -L./raylib/src -lraylib -lm -lpthread -ldl -lrt -lX11 -o meu_programa
```

Ou usando o make file que criamos:

```
# Copie o nosso modelo de Makefile para o nome padrão "Makefile"
cp Makefile.projeto Makefile

# Compile passando o nome do seu arquivo (sem o .c)
make NOME=meu_programa

# Execute
./meu_programa
```

**Estrutura Final de Arquivos:**

Sua pasta de projetos deverá ter esta aparência:

```
.
├── raylib/                 # Pasta com o código-fonte da Raylib
│   └── src/
│       ├── raylib.h        # Cabeçalho principal
│       └── libraylib.a     # Biblioteca compilada
│
├── meu_programa.c       # Seu código de exemplo
├── meu_programa         # Executável do exemplo
└── Makefile.projeto        # Nosso Makefile modelo para projetos
```

