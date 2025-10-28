---
layout:
  width: wide
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: false
  metadata:
    visible: true
---

# Instalação

#### Guia de Instalação Simplificado

Bem-vindo ao nosso ambiente de desenvolvimento com Raylib! Para garantir que todos tenham uma base de projeto idêntica e possam focar no que realmente importa (programar!), seguiremos um método de instalação simplificado. Este guia utiliza um template de projeto pré-configurado que já contém a Raylib e um `Makefile` para automatizar a compilação.

***

**Passo 1: Instalar os Pré-requisitos**

Antes de baixar o projeto, precisamos garantir que seu sistema tenha as ferramentas necessárias para compilar o código.

**Para Linux (Debian/Ubuntu e derivados)**

Abra o terminal e execute o seguinte comando para instalar todas as dependências de vídeo, áudio e desenvolvimento necessárias:

```
sudo apt-get install libasound2-dev libx11-dev libxrandr-dev libxi-dev libgl1-mesa-dev libglu1-mesa-dev libxcursor-dev libxinerama-dev libwayland-dev libxkbcommon-dev
```

**Para Windows**

Para este projeto, é necessário ter um compilador C, como o MinGW-w64, instalado e configurado no PATH do sistema. Se você consegue usar o comando `gcc` no seu terminal, está tudo pronto.

Caso nao tenho o compilador instalado segue um tutorial de intalacao,  utilize o compilador MinGW-w64 (64 bits)

```
https://github.com/skeeto/w64devkit/releases/download/v2.0.0/w64devkit-x64-2.0.0.exe
```

***

**Passo 2: Baixar o Projeto Base**

Vamos clonar o repositório Git que contém nosso template de projeto.

```
git clone https://github.com/murielgodoi/raylibGames.git
cd raylibGames
```

***

**Passo 3: Compilar e Executar com o Makefile**

O `Makefile` dentro do projeto já possui todas as regras necessárias para compilar e executar seu código de forma simples.

**Para Compilar o Projeto:**

No Windows:

```
mingw32-make
```

No Linux:

```
make
```

**Para Executar o Projeto:**

No Windows:

```
mingw32-make run
```

No Linux:

```
make run
```

**Compilação Manual (Alternativa)**

Caso você queira compilar um único arquivo manualmente, sem usar o `Makefile` do projeto, pode usar os seguintes comandos. Lembre-se que você precisa estar na pasta raiz do projeto.

No Linux:

```
gcc seu_jogo.c -lraylib -lGL -lm -lpthread -ldl -lrt -lX11 -o seu_jogo
```

No Windows:

```
gcc seu_jogo.c -o seu_jogo.exe -lraylib -lopengl32 -lgdi32 -lwinmm
```

Observação: Ao compilar manualmente, pode ser necessário adicionar flags para indicar a localização dos arquivos da Raylib, como `-I./src` e `-L./src`, dependendo da estrutura do projeto. O método com `make` é sempre o recomendado.
