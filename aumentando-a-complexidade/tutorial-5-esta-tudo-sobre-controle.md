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

# Tutorial 5 - Esta Tudo Sobre Controle

Por enquanto fizemos apenas movimentos automaticos em nossas figuras, nesse tutorial quem vai assumir o comando 'e a gente. Mas tambem., todo bom heroi precisa de um nome. Entao vamos dar um nome e controlar nosso figura/personagem nesse tutorial.

***

### Estrutura De Estados e Novas Variaveis

Para fazer isso, vamos separar nosso programa em duas "telas" ou estados: uma para digitar o nome e outra para o jogo em si. Isso nos ensinará a gerenciar diferentes partes de um jogo e a capturar a digitação do teclado letra por letra.

Para gerenciar os estados, primeiro precisamos defini-los. A melhor forma de fazer isso em C é com um `enum`. Adicionalmente, precisaremos de variáveis para armazenar o nome que está sendo digitado.

```c
// --- Inicialização ---
// ...

// Definindo os estados do nosso jogo
typedef enum {
    STATE_TYPING_NAME,
    STATE_GAMEPLAY
} GameScreen;

// Estado do Jogo
GameScreen currentScreen = STATE_TYPING_NAME; // Começamos na tela de digitação

//  UI / Variáveis de Nome
char name[10]; // Um array para guardar até 9 letras + caractere nulo
int letterCount = 0; // Um contador para as letras digitadas

//  Variáveis do Jogador
Vector2 playerPosition = { (float)screenWidth / 2 - 25, (float)screenHeight / 2 - 25 };
float playerSpeed = 4.0f;
```

Vamos entender as novidades:

* `typedef enum { ... } GameScreen;`: Um `enum` é uma forma de criar um tipo de variável com nomes legíveis em vez de números. Agora, em vez de usarmos `0` e `1`, podemos usar `STATE_TYPING_NAME` e `STATE_GAMEPLAY`. É muito mais organizado!
* `GameScreen currentScreen = ...`: Esta é a variável mais importante. Ela vai guardar qual é o estado/tela atual do nosso jogo. Começamos na tela de digitação.
* `char name[10]`: É um "vetor" ou uma "caixa" com 10 espaços para guardar caracteres. Isso nos permite ter um nome de até 9 letras (o último espaço é reservado para um caractere especial `\0` que marca o fim do texto).
* `int letterCount`: Uma variável simples para sabermos quantas letras já foram digitadas.

***

### Como fica a Logica do jogo?

Agora, a grande mudança. O Bloco de **L'ogica** precisa saber qual lógica executar: a de digitar o nome ou a de mover o jogador. A ferramenta perfeita para isso é o `switch`.

Substitua seu Bloco de Atualização por esta estrutura:

```c
// --- Lógica ---
switch(currentScreen)
{
    case STATE_TYPING_NAME:
    {
        // LÓGICA DA TELA DE DIGITAÇÃO
    } break;
    case STATE_GAMEPLAY:
    {
        // LÓGICA DA TELA DE JOGO
    } break;
    default: break;
}
```

O `switch` verifica o valor de `currentScreen` e executa apenas o código dentro do `case` correspondente.

Agora, vamos preencher a lógica de cada estado:

* `case STATE_TYPING_NAME`:
  * Vamos capturar as teclas, adicionar letras ao nosso `name` e verificar se o jogador pressionou `ENTER` para finalizar.

```c
// Dentro de 'case STATE_TYPING_NAME:'
int key = GetCharPressed(); // Captura a última tecla pressionada como um caractere

// Adiciona o caractere ao nome se for uma tecla válida e houver espaço
if ((key >= 32) && (key <= 125) && (letterCount < 9)){
    name[letterCount] = (char)key;
    name[letterCount+1] = '\0'; // Adiciona o terminador nulo
    letterCount++;
}

// Lógica para apagar com BACKSPACE
if (IsKeyPressed(KEY_BACKSPACE)){
    if (letterCount > 0){
        letterCount--;
        name[letterCount] = '\0';
    }
}

// Mudar para a tela de jogo
if (IsKeyPressed(KEY_ENTER)){
    currentScreen = STATE_GAMEPLAY;
}
```

`case STATE_GAMEPLAY`:

* Agora vem a parte principal. Precisamos "escutar" o teclado a cada frame e, se uma tecla de movimento estiver pressionada, devemos atualizar a `playerPosition`. Toda essa lógica pertence exclusivamente ao Bloco de **Logica**. Adicione o seguinte código dentro do seu Bloco:

```c
// Dentro de 'case STATE_GAMEPLAY:'
if (IsKeyDown(KEY_RIGHT) || IsKeyDown(KEY_D)) playerPosition.x += playerSpeed;
if (IsKeyDown(KEY_LEFT) || IsKeyDown(KEY_A)) playerPosition.x -= playerSpeed;
if (IsKeyDown(KEY_DOWN) || IsKeyDown(KEY_S)) playerPosition.y += playerSpeed;
if (IsKeyDown(KEY_UP) || IsKeyDown(KEY_W)) playerPosition.y -= playerSpeed;
```

Vamos detalhar o que cada linha faz:

* `if (IsKeyDown(KEY_RIGHT) ...)`: A função `IsKeyDown()` é a principal ferramenta para inputs contínuos. Ela pergunta à Raylib: "A tecla SETA DIREITA está sendo segurada neste exato momento?". Se a resposta for sim, o código dentro do `if` é executado.
* `|| IsKeyDown(KEY_D)`: O símbolo `||` significa OU. Ele nos permite criar controles alternativos. Essa linha lê: "Se a SETA DIREITA OU a tecla 'D' estiverem pressionadas...". É uma forma simples de dar suporte tanto às setas quanto ao padrão WASD.
* `playerPosition.x += playerSpeed;`: Se a condição for verdadeira, nós executamos a ação: pegamos a coordenada X atual do jogador e adicionamos o valor da `playerSpeed`. Isso move o quadrado para a direita. O mesmo princípio se aplica às outras três direções, subtraindo de `x` para ir para a esquerda, e somando ou subtraindo de `y` para mover na vertical.

***

#### Desenhando O Estado Correto

O Bloco de Desenho também precisa saber o que desenhar: a tela de digitação ou a tela de jogo. Usaremos a mesma estrutura `switch`.

Substitua seu Bloco de Desenho por esta estrutura:

```c
// --- Desenho ---
    switch(currentScreen){
        case STATE_TYPING_NAME:{
            // DESENHO DA TELA DE DIGITAÇÃO
            DrawText("DIGITE O NOME DO SEU HEROI:", 170, 140, 20, GRAY);
            DrawText(name, 310, 200, 40, MAROON);
            DrawRectangle(300, 250, 200, 2, MAROON); // Uma linha sob o nome
            DrawText("Pressione ENTER para continuar", 220, 300, 20, DARKGRAY);
        } break;
        case STATE_GAMEPLAY:{
            // DESENHO DA TELA DE JOGO
            DrawRectangle(playerPosition.x, playerPosition.y, 50, 50, BLUE);
            // Mostra o nome do herói acima dele
            DrawText(TextFormat("Herói: %s", name), playerPosition.x - 20, playerPosition.y - 30, 20, BLACK);
        } break;
        default: break;
    }

EndDrawing();
```

* `case STATE_TYPING_NAME`: Desenhamos as instruções e, o mais importante, a variável `name`, que será atualizada a cada letra digitada.
* `case STATE_GAMEPLAY`: Desenhamos nosso quadrado e, como bônus, usamos `TextFormat()` para exibir o nome que o jogador digitou logo acima do personagem!

***

### Codigo Final e Resultados

```c
#include "raylib.h"

typedef enum {
    STATE_TYPING_NAME,
    STATE_GAMEPLAY
} GameScreen;


int main(){
    // --- Inicialização ---
    const int screenWidth = 800;
    const int screenHeight = 450;
    InitWindow(screenWidth, screenHeight, "Tutorial 5 - Nome e Controle");
    SetTargetFPS(60);

    GameScreen currentScreen = STATE_TYPING_NAME;
    char name[10]; 
    int letterCount = 0; 

    Vector2 playerPosition = { (float)screenWidth / 2 - 25, (float)screenHeight / 2 - 25 };
    float playerSpeed = 4.0f;

    while (!WindowShouldClose())
    {
        // --- Lógica ---
        switch(currentScreen){
            case STATE_TYPING_NAME:{
                int key = GetCharPressed();

                if ((key >= 32) && (key <= 125) && (letterCount < 9)){
                    name[letterCount] = (char)key;
                    name[letterCount+1] = '\0';
                    letterCount++;
                }

                if (IsKeyPressed(KEY_BACKSPACE)){
                    if (letterCount > 0){
                        letterCount--;
                        name[letterCount] = '\0';
                    }
                }

                if (IsKeyPressed(KEY_ENTER)){
                    currentScreen = STATE_GAMEPLAY;
                }

            } break;
            case STATE_GAMEPLAY:{
                if (IsKeyDown(KEY_RIGHT) || IsKeyDown(KEY_D)) playerPosition.x += playerSpeed;
                if (IsKeyDown(KEY_LEFT) || IsKeyDown(KEY_A)) playerPosition.x -= playerSpeed;
                if (IsKeyDown(KEY_DOWN) || IsKeyDown(KEY_S)) playerPosition.y += playerSpeed;
                if (IsKeyDown(KEY_UP) || IsKeyDown(KEY_W)) playerPosition.y -= playerSpeed;

            } break;
            default: break;
        }
        // --- Desenho ---
        switch(currentScreen){
            case STATE_TYPING_NAME:{
                DrawText("DIGITE O NOME DO SEU HEROI:", 170, 140, 20, BLACK);
                DrawText(name, 310, 200, 40, MAROON);
                DrawRectangle(300, 250, 200, 2, MAROON);
                DrawText("Pressione ENTER para continuar", 220, 300, 20, DARKGRAY);
            } break;
            case STATE_GAMEPLAY:{
                DrawRectangle(playerPosition.x, playerPosition.y, 50, 50, BLUE);
                DrawText(TextFormat("Herói: %s", name), playerPosition.x - 20, playerPosition.y - 30, 20, BLACK);
            } break;
            default: break;
        }

    EndDrawing();
    }
    
    CloseWindow();
    return 0;
}
```

<figure><img src="../.gitbook/assets/Design sem nome(2).gif" alt=""><figcaption></figcaption></figure>

#### Hora de Experimentar!

* Mude o valor de `playerSpeed` para `1.0f` ou `10.0f` e veja como a "sensibilidade" do movimento muda.
* Tente usar a função `IsKeyPressed()`. Ela só retorna `true` no único frame em que a tecla é pressionada. Como o movimento fica diferente? (Parecerá que o quadrado "dá um pulinho" a cada toque).
* Adicione a lógica de colisão com as paredes para impedir que o jogador saia da tela.
* Crie um terceiro estado, `STATE_PAUSE`, que é ativado ao pressionar 'P' durante o `STATE_GAMEPLAY` e exibe um texto "JOGO PAUSADO".
* Impida que o jogador avance com um nome vazio (Dica: no `if (IsKeyPressed(KEY_ENTER))`, adicione a condição `&& (letterCount > 0)`).
