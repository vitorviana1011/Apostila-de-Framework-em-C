---
description: Nesse tutorial, voce vai aprender a criar uma janela e escrever nela.
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

# Tutorial 1 - Criando uma janela.

### Criando a janela

Todo programa Raylib segue uma estrutura muito parecida. Vamos começar com o código mínimo necessário para criar uma janela e mantê-la aberta.

```
#include "raylib.h" // Adiciona a biblioteca raylib ao codigo
int main(){
    // Configuracao
    const int screenWidth = 800; // Largura da janela
    const int screenHeight = 450; // Altura da janela
    
    InitWindow(screenWidth, screenHeight, "Primeira Janela"); // Inicializa a janela
    SetTargetFPS(60); // Define a taxa de quadros
    
    // Loop principal
    while (!WindowShouldClose()){
        // Acoes e desenhos acontecem aqui dentro
    }
    
    // Finalizacao
    CloseWindow(); // Fecha a janela
    return 0;
}
```

Vamos destrinchar o codigo parte por parte:

* `#include "raylib.h"`: Importa a biblioteca raylib e todas as suas funcoes e capacitades para o nosso codigo. Sem ela todo o resto simplesmente nao funciona.
* `int main()`: Inicializa a main e com ele o nosso programa.
* `const int screenWidth` e `const int screenHeight`: Constantes que definem o tamanho da nossa janela.
* `InitWindow(...)`: Esta função "cria" a janela. Ela recebe a largura, a altura e um título.
* `SetTargetFPS(...)`: Define nossa "taxa de quadros por segundo" (Frames Per Second). Essencialmente, estamos pedindo para a Raylib tentar executar e redesenhar tudo dentro do nosso loop 60 vezes por segundo, criando uma animação fluida.
* `while (!WindowShouldClose())`: Este é o Loop Principal. Enquanto a janela estiver aberta (ou seja, enquanto o usuário não clicar no 'X' ou pressionar ESC), tudo que estiver dentro deste `while` será executado repetidamente, 60 vezes por segundo.
* `CloseWindow()`: Quando o loop termina, esta função é chamada para fechar a janela de forma segura e liberar os recursos do sistema.

Se você compilar e rodar este código, verá uma janela preta que não faz nada, mas que já responde ao comando de fechar. Algo parecido com isso:

<figure><img src="../.gitbook/assets/Captura de tela de 2025-09-16 13-25-38.png" alt=""><figcaption></figcaption></figure>

Uma janela vazia não tem graça, entao vamos imcrementar algo, vamos escrever um pequeno texto!

### Escrevendo na janela

Para isso temos que definir 3 fatores importantes:

1. Qual sera o nosso texto?
2. Onde ele vai ficar?
3. Qual tamanho ele vai ter?

Vamos adicionar um texto.

Primeiro, prepare as variáveis (antes do loop `while`):

```
// ... depois de SetTargetFPS(60);

// Variáveis para o nosso texto
const char* text = "Hello World!"; // Qual o nosso texto
int fontSize = 40; // O tamanho da fonte
Vector2 textPosition = { 250, 200 }; // Posição X e Y na tela
```

`Vector2`: É uma estrutura da Raylib que armazena duas variáveis (`x` e `y`), perfeita para representar posições ou direções em um espaço 2D.

Agora, vamos desenhar (dentro do loop `while`):

```
BeginDrawing(); // Inicializa o desenho
ClearBackground(RAYWHITE); // Pinta o fundo de branco
Color textColor = BLACK; // Define a cor preto para o texto 
DrawText(text, textPosition.x, textPosition.y, fontSize, textColor); // Efetivamente escreve o texto
EndDrawing(); // Termina o desenho da janela

```

* `BeginDrawing()`: Avisa à Raylib: "Ok, vou começar a desenhar coisas para este quadro!".
* `ClearBackground()`: Limpa a tela inteira com uma cor. Se não fizermos isso, os desenhos do quadro anterior continuarão na tela, criando um efeito de "rastro". `RAYWHITE` é uma cor pré-definida pela Raylib.
* `DrawText()`: A função que efetivamente desenha o texto. Ela precisa saber: o quê (`text`), onde (`textPosition.x`, `textPosition.y`), o tamanho (`fontSize`) e a cor (`BLACK`).
* `EndDrawing()`: Avisa à Raylib: "Terminei de desenhar por agora, pode mostrar o resultado na tela!".

Com tudo isso, seu codigo deve ficar assim:

```
#include "raylib.h"

int main() {
    // Configuração
    const int screenWidth = 800;
    const int screenHeight = 450;
    InitWindow(screenWidth, screenHeight, "Primeira Janela");
    SetTargetFPS(60);

    // Variáveis para o nosso texto
    const char* text = "Hello World!";
    int fontSize = 40;
    Vector2 textPosition = { 250, 200 }; // Posição X e Y na tela

    // Loop Principal
    while (!WindowShouldClose()) {
        // Desenho
        BeginDrawing();
            ClearBackground(RAYWHITE);
            DrawText(text, textPosition.x, textPosition.y, fontSize, BLACK);
        EndDrawing();
    }

    // Finalização
    CloseWindow();
    return 0;
}
```

E ao compilar e rodar o codigo, voce criou essa janela:

<figure><img src="../.gitbook/assets/Captura de tela de 2025-09-16 13-50-22.png" alt=""><figcaption></figcaption></figure>

### Hora de Experimentar!

Este tutorial é apenas uma orientação. O segredo para aprender é a prática! Pegue o código final e brinque com ele. Aqui ficam algumas ideias:

* Mude o conteúdo do texto.
* Mude o tamanho da janela atraves das variaveis `const int screenWidth` e `const int screenHeight.`
* Altere a cor do fundo e a cor do texto (tente `RED`, `BLUE`, `GREEN` ou crie sua própria cor com `(Color){R, G, B, A}`).
* Mude a posição do texto na tela alterando os valores do `Vector2`.
* Adicione um segundo `DrawText()` para escrever outra coisa em um lugar diferente!

Quanto mais você se desafiar e "quebrar" o código, mais rápido você vai aprender!
