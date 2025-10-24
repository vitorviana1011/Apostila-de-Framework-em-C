# Tutorial 8 -Animando as Coisas

Até agora, quando usamos uma textura, desenhamos a imagem inteira. Mas e se nossa imagem contiver várias pequenas imagens (chamadas de sprites) que representam os diferentes quadros de uma animação, ou até mesmo vários objetos diferentes do jogo?

Isso é chamado de Folha de Sprite (Sprite Sheet). Ela otimiza o uso de memória e torna o gerenciamento de assets mais eficiente. Neste tutorial, vamos aprender a "recortar" e animar um personagem usando uma folha de sprite.

***

### Antes do Código

Encontre uma folha de sprite simples para um personagem, idealmente com quadros de animação de tamanho fixo (ex: 32x32 pixels, 64x64 pixels).

* Exemplo: Procure por "walk cycle sprite sheet" ou "character sprite sheet". Salve uma imagem como `player_sheet.png`.
* Importante: Conheça as dimensões de cada "quadro" (frame) da sua folha de sprite. Para este tutorial, vamos assumir que cada sprite tem 32x32 pixels.
* Estrutura: Imagine que sua imagem tem 4 quadros de animação de caminhada, um ao lado do outro, todos de 32x32 pixels. A imagem total seria 128x32 pixels (4 \* 32 = 128 de largura).
* Se não encontrar: Pode criar uma simples no Paint/GIMP: faça uma imagem de 128x32 pixels e desenhe um quadrado azul no primeiro 32x32, um verde no segundo, um vermelho no terceiro e um amarelo no quarto.

***

### Carregando o Arquivo e Definindo a Animação

No nosso Bloco de **Inicialização**, vamos carregar a folha de sprite e criar variáveis para controlar qual quadro de animação está sendo exibido.

```
// --- Inicialização ---
// Configuração da Janela
const int screenWidth = 800;
const int screenHeight = 450;
InitWindow(screenWidth, screenHeight, "Tutorial 7 - Sprites e Animação");
SetTargetFPS(60);

// Texturas/Sprites
Texture2D playerSheet = LoadTexture("recursos/player_sheet.png"); // Nossa folha de sprite

// Animação
const int frameWidth = 32;  // Largura de um único quadro de sprite
const int frameHeight = 32; // Altura de um único quadro de sprite
int currentFrame = 0;       // Qual quadro da animação estamos mostrando (0 = primeiro)
float frameTime = 0.0f;     // Tempo acumulado para mudar o frame
float frameSpeed = 0.1f;    // Velocidade da animação (tempo para mudar de frame)

// Posições e Lógica do Jogador
Vector2 playerPosition = { (float)screenWidth / 2.0f, (float)screenHeight / 2.0f };
float playerSpeed = 4.0f;
```

Vamos entender as novas variáveis:

* `Texture2D playerSheet`: Variável que guarda a folha de sprite inteira.
* `frameWidth`, `frameHeight`: Constantes que definem o tamanho de cada sprite individual dentro da folha. É crucial que esses valores correspondam ao tamanho real dos seus sprites.
* `currentFrame`: Um número inteiro que indica qual sprite (quadro) da nossa animação está ativo no momento. Começamos no quadro 0 (o primeiro).
* `frameTime`: Acumula o tempo que passou desde a última troca de frame. Usaremos isso para controlar a velocidade da animação.
* `frameSpeed`: Define a cada quantos segundos o `currentFrame` deve mudar. `0.1f` significa 100 milissegundos.

***

### Lógica da Animação

A cada frame, precisamos atualizar o `frameTime` e decidir se é hora de avançar para o próximo quadro da animação. Esta lógica vai no Bloco de **Lógica**.

```
// --- Lógica ---
// Lógica da Animação
frameTime += GetFrameTime(); // Acumula o tempo que o último frame levou para ser desenhado

if (frameTime >= frameSpeed){
    frameTime = 0.0f; // Reinicia o contador de tempo

    currentFrame++; // Avança para o próximo quadro

    // Se o quadro atual ultrapassou o número de quadros na nossa animação, volta para o primeiro
    if (currentFrame * frameWidth >= playerSheet.width) // Se for uma linha de sprites
    // if (currentFrame >= TOTAL_FRAMES){ // Se souber o número total de frames
    
        currentFrame = 0; // Volta para o primeiro quadro
    }
}
```

Vamos analisar a lógica da animação:

* `frameTime += GetFrameTime();`: `GetFrameTime()` retorna o tempo, em segundos, que o frame anterior levou para ser processado. Somamos isso ao `frameTime`, acumulando o tempo.
* `if (frameTime >= frameSpeed)`: Quando o tempo acumulado (`frameTime`) atinge ou excede a nossa velocidade (`frameSpeed`), significa que é hora de mudar o sprite.
* `currentFrame++`: Avançamos para o próximo quadro.
* `if (currentFrame * frameWidth >= playerSheet.width)`: Esta é uma forma simples de verificar se já mostramos todos os quadros de uma linha de sprites. Se a posição X do nosso `currentFrame` atual exceder a largura total da `playerSheet`, voltamos para o `currentFrame = 0`.
  * Alternativa: Se você sabe exatamente quantos frames sua animação tem (ex: 4 frames), pode usar `if (currentFrame >= 4) { currentFrame = 0; }`.

***

### Desenhando o Sprite

Agora a parte mais importante: dizer à `DrawTexturePro()` qual parte da `playerSheet` ela deve desenhar. É aqui que o `sourceRec` se torna dinâmico.

<pre><code>// --- Desenho ---
<strong>BeginDrawing();
</strong><strong>    ClearBackground(RAYWHITE);
</strong>
    // Definindo os Retângulos para DrawTexturePro
    // A moldura de ORIGEM: O "recorte" da nossa folha de sprite
    // A posição X do recorte é calculada: currentFrame * frameWidth
    Rectangle sourceRec = { (float)currentFrame * frameWidth, 0, (float)frameWidth, (float)frameHeight };

    // A moldura de DESTINO: Onde na tela e com que tamanho o sprite será desenhado
    // Aqui estamos desenhando o sprite em 64x64 pixels (o dobro do tamanho original)
    Rectangle destRec = { playerPosition.x, playerPosition.y, 64, 64 };

    // Desenha o sprite animado
    DrawTexturePro(playerSheet, sourceRec, destRec, (Vector2){ 0, 0 }, 0.0f, WHITE);

EndDrawing();
</code></pre>

Vamos focar no `sourceRec` alterado:

* `Rectangle sourceRec = { (float)currentFrame * frameWidth, 0, (float)frameWidth, (float)frameHeight };`:
  * `(float)currentFrame * frameWidth`: Este é o segredo! Ele calcula a posição X do canto superior esquerdo do sprite que queremos recortar. Se `currentFrame` for 0, começa em `0 * 32 = 0`. Se for 1, começa em `1 * 32 = 32`, e assim por diante.
  * `0`: A posição Y do recorte (assumimos que nossos sprites estão na primeira linha).
  * `(float)frameWidth`, `(float)frameHeight`: A largura e altura do sprite que estamos recortando.

***

### Liberando a Mémoria

Não esqueça de liberar a textura quando o jogo terminar.

<pre><code>// --- Limpeza --- 
// (Depois do Loop)
<strong>UnloadTexture(playerSheet); // Libera a folha de sprite da memória
</strong>CloseWindow();
</code></pre>

### Código Final

```
#include "raylib.h"

int main(void)
{
    // --- Inicialização ---
    const int screenWidth = 800;
    const int screenHeight = 450;
    InitWindow(screenWidth, screenHeight, "Tutorial 7 - Animação Parada");
    SetTargetFPS(60);

    Texture2D playerSheet = LoadTexture("recursos/AnimationSheet.png");

    const int FRAME_WIDTH = 16;
    const int FRAME_HEIGHT = 24;
    const int TOTAL_FRAMES_PER_ROW = 9;

    int currentFrame = 0;
    float frameTime = 0.0f;
    float frameSpeed = 0.1f;

    Vector2 playerPosition = { (float)screenWidth / 2.0f - (FRAME_WIDTH * 4.0f) / 2.0f, 
                               (float)screenHeight / 2.0f - (FRAME_HEIGHT * 4.0f) / 2.0f };

    while (!WindowShouldClose())
    {
        // --- Lógica ---
        frameTime += GetFrameTime();
        if (frameTime >= frameSpeed)
        {
            frameTime = 0.0f;
            currentFrame++;
            if (currentFrame >= TOTAL_FRAMES_PER_ROW)
            {
                currentFrame = 0;
            }
        }

        //--- Desenho ---
        BeginDrawing();
            ClearBackground(BLACK);

            float sourceX = (float)currentFrame * FRAME_WIDTH;
            float sourceY = 0 * FRAME_HEIGHT; // Usando a primeira linha (índice 0)
            
            Rectangle sourceRec = { sourceX, sourceY, (float)FRAME_WIDTH, (float)FRAME_HEIGHT };
            
            float escala = 4.0f;
            Rectangle destRec = { playerPosition.x, playerPosition.y, FRAME_WIDTH * escala, FRAME_HEIGHT * escala };

            DrawTexturePro(playerSheet, sourceRec, destRec, (Vector2){ 0, 0 }, 0.0f, WHITE);
        EndDrawing();
    }
    // --- Limpeza ---
    UnloadTexture(playerSheet);
    CloseWindow();
    return 0;
}
```

#### Hora de Experimentar

* Mude o `frameSpeed` para `0.05f` ou `0.2f` e veja como a velocidade da animação muda.
* Tente fazer com que a animação só aconteça quando o personagem estiver se movendo. Se ele estiver parado, `currentFrame` deve ser travado em `0`.
* Se sua folha de sprite tiver várias linhas de animação (ex: caminhada para cima, para baixo, para os lados), você pode criar variáveis `currentAnimationRow` e `currentFrameInRow` e ajustar o `sourceRec.y` para selecionar a linha correta.
