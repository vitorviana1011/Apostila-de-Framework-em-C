# Tutorial 6 - Pintando o Sete

Nesta seção, vamos aprender a "pintar" nossos objetos usando uma imagem, um processo conhecido como mapeamento de textura.

Pense nisso como aplicar um papel de parede a uma forma geométrica. A imagem que usamos para isso é chamada de Textura.

**Antes de Começar: Prepare sua Imagem!**

1. Encontre uma imagem que servirá como a "pele" da nossa figura. Pode ser um desenho simples, um padrão ou qualquer coisa que você queira. O ideal é um arquivo `.png` com fundo transparente.
2. Crie uma pasta chamada `recursos` ao lado do seu arquivo `.c` (se ainda não tiver uma).
3. Salve sua imagem dentro desta pasta com um nome simples, como `textura.png`

***

### Carregando a Textura

Assim como as fontes, as texturas vivem em arquivos no disco e precisam ser carregadas para a memória da placa de vídeo (GPU) para serem usadas. Esta operação, que pode ser um pouco lenta, acontece uma única vez, no nosso Bloco de **Inicializacao**.

```
// --- Inicialização ---
// Configuração da Janela
const int screenWidth = 800;
const int screenHeight = 450;
InitWindow(screenWidth, screenHeight, "Tutorial 5 - Texturas");
SetTargetFPS(60);

// Texturas
Texture2D playerTexture = LoadTexture("recursos/textura.png");

// Posições e Lógica do Jogador
Vector2 playerPosition = { (float)screenWidth / 2.0f, (float)screenHeight / 2.0f };
float playerSpeed = 4.0f;
```

Vamos entender as novidades:

* `Texture2D playerTexture`: Declaramos uma variável do tipo `Texture2D`. Este é o "contêiner" especial da Raylib para guardar uma imagem que está pronta para ser usada pela GPU.
* `LoadTexture("recursos/textura.png")`: Esta função localiza o arquivo de imagem, o processa e o envia para a memória de vídeo. O resultado é armazenado na nossa variável `playerTexture`. Fazemos isso antes do loop para garantir o bom desempenho do jogo.

***

### Aplicando a Textura

Com nossa "tinta" (textura) pronta, podemos aplicá-la ao nosso objeto a cada frame. Para isso, vamos ao Bloco de Desenho e substituímos a antiga chamada `DrawRectangle()` (Ou qualuer outra figura) pela função `DrawTexture()`.

```
// --- Desenho ---
BeginDrawing();
    ClearBackground(RAYWHITE);
    DrawTexture(playerTexture, playerPosition.x, playerPosition.y, WHITE);
EndDrawing();
```

Vamos detalhar os parâmetros da função `DrawTexture()`:

* `playerTexture` (primeiro parâmetro): A variável que contém nossa imagem carregada. É ela que diz à Raylib qual imagem usar como "tinta".
* `playerPosition.x` (segundo parâmetro): A coordenada X (horizontal) onde o canto superior esquerdo da imagem será posicionado.
* `playerPosition.y` (terceiro parâmetro): A coordenada Y (vertical). Como nossa lógica de movimento no Bloco de Atualização continua a mesma, a textura se moverá exatamente como o quadrado se movia.
* `WHITE` (quarto parâmetro): Este é o parâmetro de coloração (`tint`). Pense nele como uma luz colorida que você joga sobre a imagem.
  * Usar `WHITE` é como usar uma luz branca: as cores originais da imagem são preservadas.
  * Se você usasse `RED`, seria como jogar uma luz vermelha sobre a imagem, deixando tudo avermelhado. É uma forma poderosa de criar efeitos.

***

### Liberando a Memoria

Todo recurso que carregamos deve ser liberado ao final do programa. Isso é crucial para a boa saúde do seu aplicativo.

```
// --- Limpeza ---
UnloadTexture(playerTexture); // Libera a textura da memória da GPU
CloseWindow();
```

`UnloadTexture(playerTexture)`: Avisa à placa de vídeo que não precisamos mais dessa imagem, liberando o espaço que ela ocupava para outras tarefas.

***

### Codigo Completo e Resultado

```
#include "raylib.h"

int main(){
    // --- Inicialização ---
    // Configuração da Janela
    const int screenWidth = 800;
    const int screenHeight = 450;
    InitWindow(screenWidth, screenHeight, "Tutorial 5 - Texturas");
    SetTargetFPS(60);

    // Texturas
    Texture2D playerTexture = LoadTexture("recursos/textura.png");

    // Posições e Lógica do Jogador
    Vector2 playerPosition = { (float)screenWidth / 2.0f, (float)screenHeight / 2.0f };
    float playerSpeed = 4.0f;

    while(!WindowShouldClose()){
        //--- Lógica ---
        if (IsKeyDown(KEY_RIGHT) || IsKeyDown(KEY_D)) playerPosition.x += playerSpeed;
        if (IsKeyDown(KEY_LEFT) || IsKeyDown(KEY_A)) playerPosition.x -= playerSpeed;
        if (IsKeyDown(KEY_DOWN) || IsKeyDown(KEY_S)) playerPosition.y += playerSpeed;
        if (IsKeyDown(KEY_UP) || IsKeyDown(KEY_W)) playerPosition.y -= playerSpeed;

        
        //--- Desenho ---
        BeginDrawing();
            ClearBackground(RAYWHITE);
            DrawTexture(playerTexture, playerPosition.x, playerPosition.y, WHITE);
        EndDrawing();
    }

    // --- Limpeza ----
    UnloadTexture(playerTexture); // Libera a textura da memória da GPU
    CloseWindow();

}
```

<figure><img src="../.gitbook/assets/Design sem nome(3).gif" alt=""><figcaption></figcaption></figure>

#### Hora de Experimentar

* Encontre online uma textura de tijolos (`bricks.png`) e use-a para criar uma parede estática na tela com `DrawTexture()`.
* Experimente com o parâmetro de `tint` (a cor). Use `(Color){255, 0, 0, 150}` para deixar seu personagem vermelho e semi-transparente, como se tivesse levado dano.
* Carregue uma textura de grama e use-a para "pintar" o chão da sua tela (desenhando a textura em um grande retângulo no rodapé).
