# Tutorial 2 - Aprofundando no Texto

### Introducao

Agora voce sabe como abrir uma janela e escrever um texto. Nessa secao vamos aprofundar um pouco nessa segunda parte: escrever um texto

***

### Deixando a sua cara - Trocando a fonte

Crie uma pasta chamada `recursos` ao lado do seu arquivo `.c` e coloque um arquivo de fonte (`.ttf` ou `.otf`) dentro dela. Para este exemplo, vou assumir que o nome do arquivo é arial`.ttf` (uma fonte classica, presente em todos os lugares).

Vamos começar com a estrutura básica: criar a janela e carregar nossa fonte. Por enquanto, só vamos escrever um título no canto da tela.

```
#include "raylib.h"

int main(void)
{
    // Configuração da Janela
    const int screenWidth = 800;
    const int screenHeight = 450;
    InitWindow(screenWidth, screenHeight, "Tutorial de Texto Avançado");
    SetTargetFPS(60);

    // Carregando a fonte personalizada
    Font customFont = LoadFont("recursos/ARIAL.TTF");

    // Loop Principal do Jogo
    while (!WindowShouldClose())
    {
         BeginDrawing();

            ClearBackground(DARKBLUE); // Um fundo azul escuro para dar clima

            // Desenhando o título com a nova fonte
            DrawTextEx(customFont, "Texto em Arial!", (Vector2){ 20, 20 }, 60, 2, WHITE);

        EndDrawing();
    }

    // Descarregando a fonte da memória
    UnloadFont(customFont);

    CloseWindow();
    return 0;
}
```

Vamos entender as novidades neste código:

* `Font customFont = LoadFont(...)`: Esta linha, posicionada antes do loop principal, carrega o arquivo da fonte para uma variável especial do tipo `Font`. Fazemos isso apenas uma vez para que a fonte fique pronta na memória para ser usada rapidamente a qualquer momento.
* `DrawTextEx(...)`: Dentro do loop, usamos esta nova função para desenhar. A diferença fundamental é que o primeiro argumento que passamos é a nossa variável `customFont`, dizendo à Raylib para usar a fonte que carregamos, e não a padrão.
* `UnloadFont(...)`: Depois que o loop termina, nós liberamos a memória que a fonte estava usando. É uma boa prática que mantém seu programa limpo e eficiente.



Com isso possuimos uma janela assim:

<figure><img src="../.gitbook/assets/Captura de tela de 2025-10-07 13-52-51.png" alt=""><figcaption></figcaption></figure>

Uma janela com o fundo azul e nosso texto em branco, e o principal, com fonte Arial!

***

### Texto em Tempo Real - Informacao Dinamica

As vezes um programa precisa mostrar informações que se atualizam, como uma pontuação. Para isso, vamos misturar texto com o valor de variáveis.

Primeiro, prepare as variáveis (antes do loop `while`):

Precisamos de uma variável para guardar o placar e outra para nos ajudar a controlar o tempo.

```
int score = 0;
double lastUpdateTime = 0.0;
```

Agora, a lógica e o desenho (dentro do loop `while`):

Vamos adicionar a lógica para aumentar o placar em intervalos regulares e o comando para desenhar esse valor na tela.

```
#include <time.h> // Nova biblioteca para usar a funcao GetTime

// Lógica de atualização (colocar antes de BeginDrawing)
if (GetTime() - lastUpdateTime >= 1.0)
{
    score += 100;
    lastUpdateTime = GetTime();
}

// Desenho (colocar dentro de BeginDrawing/EndDrawing)
DrawText(TextFormat("SCORE: %05i", score), 20, 100, 30, GOLD);
```

Vamos entender o que esse novo trecho faz:

* `if (GetTime() ...)`: O loop do jogo roda 60 vezes por segundo. Essa condição `if` age como um filtro, garantindo que o placar só aumente uma vez por segundo, tornando a atualização visível e controlada.
* `TextFormat("SCORE: %05i", score)`: Esta é a ferramenta mágica da Raylib para misturar texto com variáveis. O código `"%05i"` é uma "receita" que diz: "neste lugar, coloque um número inteiro (`i`), formatado para ter 5 dígitos, preenchendo com zeros (`05`) à esquerda".

Com isso possuimos uma janela assim:

<figure><img src="../.gitbook/assets/Captura de tela de 2025-10-07 14-06-27.png" alt=""><figcaption></figcaption></figure>

Temos a mesma janela do exemplo anterior, porem agora, logo abaixo, temos um placar dinamica, ou seja, atualiza seu valor a cada segundo que passada.

***

### Alinhamento Perfeito e Toques Finais

Até agora, posicionamos os textos "no olho". Para uma interface profissional, precisamos de precisão matemática.

A Ferramenta Essencial: `MeasureTextEx()`

Para alinhar algo, primeiro precisamos saber seu tamanho. Adicione esta linha antes do loop `while`:

```
const char *titleText = "Texto em Arial!;
Vector2 titleSize = MeasureTextEx(customFont, titleText, 60, 2);
```

* O quê: A função `MeasureTextEx()` age como uma fita métrica virtual. Ela calcula a largura e altura exatas que o texto terá, mas sem desenhá-lo.
* Por quê: Precisamos saber a largura do título com antecedência para podermos calcular onde ele deve ficar para ser centralizado. Como o título não muda, medimos apenas uma vez.

#### A Lógica e a Aplicação no Desenho

Agora, dentro do loop, vamos usar essa medida para calcular a posição e desenhar. Substitua o `DrawTextEx` do título e adicione os outros textos:

```
// Cálculo e desenho (dentro de BeginDrawing/EndDrawing)

// 1. Calculamos a posição X do título para centralizá-lo
float titlePosX = (GetScreenWidth() / 2.0f) - (titleSize.x / 2.0f);
DrawTextEx(customFont, titleText, (Vector2){ titlePosX, 60 }, 60, 2, WHITE);

// 2. Desenhamos o placar no rodapé esquerdo
DrawText(TextFormat("SCORE: %05i", score), 20, GetScreenHeight() - 40, 30, GOLD);

// 3. Desenhamos um texto alinhado à direita
const char *highScoreText = "High Score: 12500";
float highScoreTextWidth = MeasureText(highScoreText, 20);
DrawText(highScoreText, GetScreenWidth() - highScoreTextWidth - 20, 20, 20, LIGHTGRAY);
```

A fórmula `(tela / 2) - (texto / 2)` é o segredo para a centralização. Pense em pendurar um quadro: você acha o meio da parede e recua metade da largura do quadro. A lógica é a mesma!

Com isso possuimos uma janela assim:

<figure><img src="../.gitbook/assets/Captura de tela de 2025-10-07 14-10-45.png" alt=""><figcaption></figcaption></figure>

Agora, centralizamos nosso texto principal no centro da janela, no canto inferior esquerdo o nosso score, e no canto superior direito adicionamos o maior score.

***

**Código Final e Resultado**

Juntando todas as partes, seu programa completo ficará assim:

```
#include "raylib.h"
#include <time.h>

int main(void)
{
    // Configuração da Janela
    const int screenWidth = 800;
    const int screenHeight = 450;
    InitWindow(screenWidth, screenHeight, "Tutorial de Texto Avançado");
    SetTargetFPS(60);

    Font customFont = LoadFont("recursos/ARIAL.TTF");

    int score = 0;
    double lastUpdateTime = 0.0;
    
    // Textos estáticos que vamos alinhar
    const char *titleText = "Texto em Arial!";
    const char *scoreText = TextFormat("SCORE: %05i", score); // Apenas para medir inicialmente

    // Medindo os textos ANTES do loop para calcular posições
    Vector2 titleSize = MeasureTextEx(customFont, titleText, 60, 2);
    
    // Loop Principal do Jogo
    while (!WindowShouldClose())
    {
        // Lógica de atualização
        if (GetTime() - lastUpdateTime >= 1.0)
        {
            score += 100;
            lastUpdateTime = GetTime();
        }

        // Desenho
        BeginDrawing();
        
            ClearBackground(DARKBLUE);

            // Calculando a posição X do título para centralizá-lo
            float titlePosX = (screenWidth / 2.0f) - (titleSize.x / 2.0f);
            DrawTextEx(customFont, titleText, (Vector2){ titlePosX, 60 }, 60, 2, WHITE);

            // Formatando o placar a cada frame
            scoreText = TextFormat("SCORE: %05i", score);
            DrawText(scoreText, 20, screenHeight - 40, 30, GOLD); // Posicionado no rodapé esquerdo
            
            // Desenhando um texto alinhado à direita
            DrawText("High Score: 12500", screenWidth - MeasureText("High Score: 12500", 20) - 20, 20, 20, LIGHTGRAY);

        EndDrawing();
    }

    UnloadFont(customFont);
    CloseWindow();
    return 0;
}
```

### Hora de Experimentar!

Agora que você tem essas novas ferramentas, o desafio é seu! Tente modificar o código para:

* Fazer o "High Score" ser uma variável e não um texto fixo.
* Adicionar um cronômetro que conta de forma regressiva usando `TextFormat()` com `%f`.
* Mudar a cor do placar quando ele passar de 10000 pontos.
