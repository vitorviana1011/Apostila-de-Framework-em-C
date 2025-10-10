# Tutorial 4 - Movendo e Colidindo

Vamos dar um passo além das formas estáticas. Vamos criar um objeto que se move sozinho e ensiná-lo a interagir com o ambiente, primeiro ricocheteando nas paredes e depois detectando a presença de outros objetos.

***

### Passo 1 - Ele esta vivo!

Nosso primeiro objetivo é fazer um circulo se mover de um lado para o outro da tela, sem intervenção do jogador.

#### Preparando as variaveis

Antes de qualquer coisa, temos que declarar as variaveis que vamos usar. A partir de agora, conforme nossos programas crescem em tamanho vamos separa-los em blocos, esse primeiro bloco sera o bloco de **Inicializacao,** que acontece antes do nosso loop principa&#x6C;**.**

```
// --- Inicializacao ---
// Configuração da Janela
const int screenWidth = 800;
const int screenHeight = 450;
InitWindow(screenWidth, screenHeight, "Tutorial 3 - Movimento Automático e Colisões");
SetTargetFPS(60);

// Posições e Lógica da Caixa
Vector2 boxPosition = { 100, 200 }; // Posição inicial
Vector2 boxSpeed = { 4.0f, 0.0f }; // Velocidade: 4 pixels/frame no eixo X, 0 no eixo Y
```

Vamos analisar as novidades:

* `Vector2 boxPosition`: Armazena a posição atual X e Y da nossa caixa.
* `Vector2 boxSpeed`: Esta é a grande mudança. Em vez de um único número (`float`), estamos usando um `Vector2` para a velocidade. Isso nos permite controlar a velocidade nos eixos X (`boxSpeed.x`) e Y (`boxSpeed.y`) de forma independente. No momento, a velocidade Y é zero, então o movimento será puramente horizontal.

#### Atualizando a posicao

Fazer o bloco se mover sozinho e bem simples: a cada frame, simplesmente aplicamos a velocidade à posição. Esse passo vai ficar em um bloco dentro do nosso loop principal chamado **Atualizacao da L'ogica.**

```
// --- Atualização da Lógica ---
boxPosition.x += boxSpeed.x; // Movimento da Caixa
```

* `boxPosition.x += boxSpeed.x`: A cada frame, a posição X da caixa é atualizada, somando-se o valor da sua velocidade X. Como o loop roda 60 vezes por segundo, a caixa se moverá 4 pixels para a direita, 60 vezes por segundo.

#### Desenhando o Movimento

O Bloco de **Desenho** apenas desenha a caixa na sua nova posição, que foi calculada no Bloco de Atualização.

```
// --- Desenho ---
BeginDrawing();
            ClearBackground(RAYWHITE);
            DrawCircle(boxPosition.x, boxPosition.y, 50, RED); // Desenha a caixa
EndDrawing();
```

Com isso voce vai ter algo assim:

<figure><img src="../.gitbook/assets/tutorail4.gif" alt=""><figcaption></figcaption></figure>

***

