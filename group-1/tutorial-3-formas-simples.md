# Tutorial 3 - Formas Simples

Nesta seção, vamos aprender a usar as funções do módulo `rshapes` para criar e colorir as formas geométricas básicas. Cada uma delas funciona de uma maneira um pouco diferente, e entender seus parâmetros é a chave para desenhar qualquer coisa que você imaginar. Todo o código que escreveremos a seguir pertence ao Bloco de Desenho, entre as funções `BeginDrawing()` e `EndDrawing()`.

***

### Passo 1 - Retangulo

```
DrawRectangle(50, 70, 100, 60, BLUE);
```

Vamos entender cada um dos seus 5 parâmetros:

* `50` (posição X): A distância em pixels da borda esquerda da tela até o canto esquerdo do retângulo.
* `70` (posição Y): A distância em pixels do topo da tela até o canto superior do retângulo. Esses dois primeiros valores definem o ponto de partida do desenho.
* `100` (largura): A largura do retângulo em pixels, a partir do ponto inicial.
* `60` (altura): A altura do retângulo em pixels, a partir do ponto inicial.
* `BLUE`: A cor de preenchimento. A Raylib tem várias cores prontas, como `RED`, `GOLD`, `SKYBLUE`, etc.

***

### Passo 2 - Circulo

Diferente do retângulo, o círculo não é definido por um canto, mas sim pelo seu centro e pela distância até sua borda.

```
DrawCircle(GetScreenWidth() / 2, GetScreenHeight() / 2, 50, MAROON);
```

Analisando os 4 parâmetros:

* `GetScreenWidth() / 2` (centro X): Esta função pega a largura total da tela e a divide por 2, nos dando a coordenada X exata do meio da tela.
* `GetScreenHeight() / 2` (centro Y): O mesmo para a altura, nos dando a coordenada Y do meio da tela. Usar essas funções garante que o círculo sempre estará centralizado, mesmo que você mude o tamanho da janela.
* `50` (raio): O raio do círculo em pixels. Lembre-se que o raio é a distância do centro até qualquer ponto da borda. O diâmetro (largura total) do círculo será o dobro disso (100 pixels).
* `MAROON`: A cor de preenchimento.

***

### Passo 3 - Triangulo

O triângulo é a forma mais fundamental da computação gráfica. Ele é definido pelas coordenadas exatas do seus tres vertices.

```
DrawTriangle((Vector2){400, 350}, (Vector2){300, 400}, (Vector2){500, 400}, GREEN);
```

Analisando os 4 parâmetros:

* `(Vector2){400, 350}` (Ponto 1): Este é o primeiro vértice. Usamos uma `struct` `Vector2` para agrupar as coordenadas X e Y de um ponto. Este é o vértice de cima do nosso triângulo.
* `(Vector2){300, 400}` (Ponto 2): O segundo vértice, que ficará na parte inferior esquerda.
* `(Vector2){500, 400}` (Ponto 3): O terceiro vértice, na parte inferior direita. A Raylib se encarrega de "conectar os pontos" e preencher a área entre eles.
* `GREEN`: A cor de preenchimento.

***

### Passo 4 - Linha

Para desenhar uma linha, precisamos apenas de um ponto inicial e um ponto final.

```
DrawLineEx((Vector2){550, 50}, (Vector2){750, 400}, 4.0f, GOLD);
```

Analisando os 4 parâmetros:

* `(Vector2){550, 50}` (Ponto Inicial): As coordenadas X e Y de onde a linha começa.
* `(Vector2){750, 400}` (Ponto Final): As coordenadas X e Y de onde a linha termina.
* `4.0f` (Espessura): A espessura da linha em pixels. Este parâmetro só está disponível na função `DrawLineEx()`. A `DrawLine()` normal tem sempre 1 pixel de espessura.
* `GOLD`: A cor da linha.

***

### Codigo Final

```
#include "raylib.h"

int main()
{
    const int screenWidth = 800;
    const int screenHeight = 450;
    InitWindow(screenWidth, screenHeight, "Desenhando Formas Simples");
    SetTargetFPS(60);

    while (!WindowShouldClose())
    {

        BeginDrawing();

            ClearBackground(RAYWHITE);

            DrawRectangle(50, 50, 100, 60, BLUE); // Desenha um retângulo azul
            DrawCircle(GetScreenWidth() / 2, GetScreenHeight() / 2, 50, RED); // Desenha um círculo vermelho
            DrawTriangle((Vector2){400, 350}, (Vector2){300, 400}, (Vector2){500, 400}, GREEN); // Desenha um triângulo verde
            DrawLineEx((Vector2){550, 50}, (Vector2){750, 400}, 4.0f, GOLD); // Desenha uma linha dourada

        EndDrawing();
    }

    CloseWindow();
    return 0;
}
```

Ao compilar e rodar, você verá sua primeira cena estática, com todas as formas que programamos aparecendo na tela!

<figure><img src="../.gitbook/assets/Captura de tela de 2025-10-09 10-57-19.png" alt=""><figcaption></figcaption></figure>

**Hora de Experimentar!**

O segredo para dominar o desenho é praticar. Tente modificar o código para:

* Mudar as cores, tamanhos e posições das formas existentes.
* Usar outras funções como `DrawEllipse()` (uma elipse) ou `DrawRing()` (um anel/círculo vazado).
* Tentar desenhar apenas o contorno das formas, usando funções como `DrawRectangleLines()`, `DrawCircleLines()` ou `DrawTriangleLines()`.
* Desafio: Tente criar um desenho simples combinando as formas. Que tal um boneco de palito, um carro ou uma casa?
