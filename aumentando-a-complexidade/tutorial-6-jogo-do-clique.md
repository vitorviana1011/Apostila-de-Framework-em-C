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

# Tutorial 6 - Jogo do Clique

Bem-vindo ao nosso primeiro minijogo! Vamos criar um "treinador de mira" simples, onde um alvo (uma bolinha) aparece em locais aleatórios na tela. O objetivo é clicar nele o mais rápido possível para ganhar pontos.

Este tutorial vai nos ensinar a:

1. Ler cliques do mouse.
2. Verificar se o clique do mouse acertou um objeto (colisão ponto-círculo).
3. Usar números aleatórios para fazer o alvo "pular" para um novo local.
4. Exibir uma pontuação que se atualiza.

***

### Preparando as Variaveis

Como sempre, começamos no Bloco de **Inicialização** para configurar nossas variáveis. Precisamos de um círculo para ser o alvo e uma variável para guardar a pontuação.

```c
// --- Inicialização
const int screenWidth = 800;
const int screenHeight = 450;
InitWindow(screenWidth, screenHeight, "Tutorial 6 - Jogo do Clique");
SetTargetFPS(60);

// Posições e Lógica do Jogo
Vector2 targetPosition = { (float)screenWidth / 2, (float)screenHeight / 2 }; // Posição inicial do alvo
float targetRadius = 30.0f;

// UI / Lógica do Jogo
int score = 0;
```

Vamos entender as variáveis:

* `Vector2 targetPosition`: Armazena as coordenadas X e Y do centro do nosso alvo. Começamos no meio da tela.
* `float targetRadius`: Define o raio (tamanho) do nosso alvo.
* `int score`: Uma variável para guardar a pontuação do jogador, começando em 0.

***

### Desenhando a Cena

No Bloco de **Desenho**, vamos renderizar nosso alvo (um círculo vermelho) e a pontuação.

```c
// --- Bloco de Desenho ---
BeginDrawing();
    ClearBackground(RAYWHITE);

    // Desenha a pontuação no canto da tela
    DrawText(TextFormat("Pontuação: %04i", score), 20, 20, 30, GRAY);

    // Desenha o nosso alvo (círculo)
    DrawCircleV(targetPosition, targetRadius, MAROON);

EndDrawing();
```

* `DrawText(TextFormat(...))`: Usamos o `TextFormat` que aprendemos para exibir a variável `score`. O `%04i` garante que ela sempre tenha 4 dígitos (ex: "0000", "0100").
* `DrawCircleV(...)`: Esta é uma variação útil da `DrawCircle`. Em vez de pedir `x` e `y` separados, ela aceita um `Vector2` diretamente para a posição do centro, o que deixa o código mais limpo.

***

### Lógica: Detectando o Clique e Colisão

Agora, o coração do jogo. No Bloco de **Lógica**, precisamos verificar duas coisas a cada frame:

1. O jogador clicou com o mouse?
2. Se sim, o clique foi dentro do alvo?

Adicione o seguinte código ao seu Bloco de Atualização:

```c
// --- Lógica ---

// Verifica se o botão esquerdo do mouse foi pressionado NESTE frame
if (IsMouseButtonPressed(MOUSE_BUTTON_LEFT)){
    // Pega a posição do mouse no momento do clique
    Vector2 mousePosition = GetMousePosition();

    // Verifica a colisão entre o ponto do mouse e o nosso círculo
    if (CheckCollisionPointCircle(mousePosition, targetPosition, targetRadius)){
        // ACERTOU!
        score += 100; // Aumenta a pontuação
        
        // Move o alvo para uma nova posição aleatória
        targetPosition.x = GetRandomValue(targetRadius, screenWidth - targetRadius);
        targetPosition.y = GetRandomValue(targetRadius, screenHeight - targetRadius);
    }
}
```

Vamos dissecar essa lógica:

* `if (IsMouseButtonPressed(MOUSE_BUTTON_LEFT))`:
  * O quê: Verifica se o botão esquerdo do mouse foi _pressionado_.
  * Por quê: Usamos `IsMouseButtonPressed()` (pressionado) em vez de `IsMouseButtonDown()` (segurado). `Pressed` é `true` apenas no único frame em que o clique acontece, o que impede o jogador de ganhar centenas de pontos apenas segurando o botão em cima do alvo.
* `Vector2 mousePosition = GetMousePosition();`: Se o clique aconteceu, esta função nos dá as coordenadas X e Y exatas do cursor do mouse.
* `if (CheckCollisionPointCircle(...))`:
  * O quê: Esta é a função de colisão perfeita da Raylib para este caso. Ela retorna `true` se um ponto (`mousePosition`) estiver dentro de um círculo (definido por `targetPosition` e `targetRadius`).
  * Por quê: É a forma mais simples e eficiente de verificar se o "tiro" acertou o "alvo".
* `score += 100;`: Se a colisão for verdadeira, aumentamos a pontuação.
* `targetPosition.x = GetRandomValue(min, max);`:
  * O quê: Esta é a grande novidade! `GetRandomValue()` retorna um número inteiro aleatório entre um `min` e um `max`.
  * Por quê: Nós a usamos para definir uma nova coordenada X e Y para o nosso alvo. Note os limites que usamos: `targetRadius` e `screenWidth - targetRadius`. Isso garante que o alvo nunca apareça "cortado" nas bordas da tela, ele sempre aparecerá inteiramente dentro da janela.

***

### Código Final e Resultado

```c
#include "raylib.h"

int main(){
    // --- Inicialização
    const int screenWidth = 800;
    const int screenHeight = 450;
    InitWindow(screenWidth, screenHeight, "Tutorial 6 - Jogo do Clique");
    SetTargetFPS(60);

    Vector2 targetPosition = { (float)screenWidth / 2, (float)screenHeight / 2 };
    float targetRadius = 30.0f;

    int score = 0;

    while(!WindowShouldClose()){
        // --- Lógica ---
        if (IsMouseButtonPressed(MOUSE_BUTTON_LEFT)){
            Vector2 mousePosition = GetMousePosition();
            if (CheckCollisionPointCircle(mousePosition, targetPosition, targetRadius)){
                score += 100;
                targetPosition.x = GetRandomValue(targetRadius, screenWidth - targetRadius);
                targetPosition.y = GetRandomValue(targetRadius, screenHeight - targetRadius);
            }
        }

        // --- Bloco de Desenho ---
        BeginDrawing();
            ClearBackground(RAYWHITE);
            DrawText(TextFormat("Pontuação: %04i", score), 20, 20, 30, GRAY);
            DrawCircleV(targetPosition, targetRadius, MAROON);
        EndDrawing();
    }
}
```

<figure><img src="../.gitbook/assets/Design sem nome(4).gif" alt=""><figcaption></figcaption></figure>

#### Hora de Experimentar!

* Tente diminuir o `targetRadius` para `15.0f` para deixar o jogo mais difícil.
* O que acontece se você errar o clique? Adicione um `else` na verificação de colisão (`if (CheckCollisionPointCircle...)`) que _diminui_ a pontuação (`score -= 50;`).
* Faça o alvo mudar de cor aleatoriamente toda vez que ele pular de lugar.
