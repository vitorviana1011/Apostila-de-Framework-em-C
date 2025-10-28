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

# Tutorial 9 - Um Lugar Silencioso

Nesta seção, vamos aprender a carregar e tocar Efeitos Sonoros (arquivos curtos, como `.wav` ou `.ogg`) usando o módulo `raudio` da Raylib. O processo é muito parecido com o de texturas: carregar, usar e descarregar.

Para conseguirmos focar no som, usaremos de base nosso Jogo do Clique do Tutorial 6.

### Antes do Código

* Encontre um arquivo de som curto que você goste para o "acerto". Pode ser um "blip", "click", "coin", etc. Sites como [freesound.org](https://freesound.org/) são ótimos para isso.
* O formato `.wav` é o mais recomendado para efeitos sonoros curtos, pois não precisa de decodificação complexa.
* Salve o arquivo na sua pasta `recursos` com um nome simples, como `acerto.wav`.

***

### Preparando o Som

Assim como a janela e as texturas, o dispositivo de áudio do seu computador precisa ser inicializado, e o arquivo de som precisa ser carregado do disco para a memória RAM.

Adicione estas novas linhas no seu Bloco de **Inicizalição:**

```c
#include "raylib.h"

int main(void)
{
    // --- Inicizalização ---
    const int screenWidth = 800;
    const int screenHeight = 450;
    InitWindow(screenWidth, screenHeight, "Tutorial 8 - Efeitos Sonoros");
    
    // Inicializa o dispositivo de áudio
    InitAudioDevice();

    SetTargetFPS(60);

    // Posições e Lógica
    Vector2 targetPosition = { (float)screenWidth / 2, (float)screenHeight / 2 };
    float targetRadius = 30.0f;

    // UI / Lógica
    int score = 0;
    
    // Carrega o efeito sonoro
    Sound hitSound = LoadSound("recursos/acerto.wav");

    // ... (resto do código)
```

Vamos entender as novidades:

* `InitAudioDevice()`:
  * O quê: Esta função "liga" o sistema de áudio da Raylib.
  * Por quê: Ela prepara seu computador para tocar sons. Assim como a `InitWindow()`, ela só precisa ser chamada uma vez no início do programa.
* `Sound hitSound = LoadSound(...)`:
  * O quê: Carrega nosso arquivo `acerto.wav` para uma variável do tipo `Sound`.
  * Por quê: `Sound` é o tipo de dado da Raylib para efeitos sonoros curtos. Eles são carregados inteiramente na memória RAM (diferente de músicas, que veremos depois). Fazer isso antes do loop permite que o som seja tocado instantaneamente quando precisarmos, sem atrasos.

***

### No Tempo Certo

Agora, o coração da lógica. Queremos que o som toque no exato momento em que o jogador acerta o alvo. O lugar perfeito para isso é dentro do Bloco de **Lógica**, logo após a colisão ser detectada.

Encontre o `if` que verifica a colisão e adicione apenas uma linha:

```c
// --- Lógica ---
if (IsMouseButtonPressed(MOUSE_BUTTON_LEFT)){
    Vector2 mousePosition = GetMousePosition();

    if (CheckCollisionPointCircle(mousePosition, targetPosition, targetRadius)){
        score += 100;
        targetPosition.x = GetRandomValue(targetRadius, screenWidth - targetRadius);
        targetPosition.y = GetRandomValue(targetRadius, screenHeight - targetRadius);
        
        // Toca o som do acerto
        PlaySound(hitSound);
    }
}
```

Vamos analisar esta linha:

* `PlaySound(hitSound)`:
  * O quê: Esta função simplesmente toca o som que está armazenado na variável `hitSound`.
  * Por quê: Colocamos ela dentro do `if` de colisão bem-sucedida. Assim, o som é disparado no exato frame em que o clique, a colisão, o aumento de pontos e a mudança de posição do alvo acontecem. É o feedback perfeito!

***

### Limpando a Mémoria

Assim como carregamos, precisamos descarregar. Todo `Load` precisa de um `Unload`, e todo `Init` precisa de um `Close`.

Adicione estas novas linhas no seu Bloco de Limpeza, ao final do programa:

```c
// --- Limpeza ---
UnloadSound(hitSound); // Libera o som da RAM
CloseAudioDevice();    // Desliga o dispositivo de áudio

CloseWindow();
```

***

### Código Final e Resultados

```c
#include "raylib.h"

int main(){
    // --- Inicialização
    const int screenWidth = 800;
    const int screenHeight = 450;
    InitWindow(screenWidth, screenHeight, "Tutorial 6 - Jogo do Clique");
    SetTargetFPS(60);

    InitAudioDevice();

    Vector2 targetPosition = { (float)screenWidth / 2, (float)screenHeight / 2 };
    float targetRadius = 30.0f;

    int score = 0;
    
    Sound hitSound = LoadSound("recursos/acerto.wav");

    while(!WindowShouldClose()){
        // --- Lógica ---
        if (IsMouseButtonPressed(MOUSE_BUTTON_LEFT)){
            Vector2 mousePosition = GetMousePosition();
            if (CheckCollisionPointCircle(mousePosition, targetPosition, targetRadius)){
                score += 100;
                targetPosition.x = GetRandomValue(targetRadius, screenWidth - targetRadius);
                targetPosition.y = GetRandomValue(targetRadius, screenHeight - targetRadius);

                PlaySound(hitSound);
            }
        }

        // --- Bloco de Desenho ---
        BeginDrawing();
            ClearBackground(RAYWHITE);
            DrawText(TextFormat("Pontuação: %04i", score), 20, 20, 30, GRAY);
            DrawCircleV(targetPosition, targetRadius, MAROON);
        EndDrawing();
    }

    // --- Limpeza ---
    UnloadSound(hitSound); 
    CloseAudioDevice();  

    CloseWindow();
    return 0;
}
```

{% file src="../.gitbook/assets/Design sem nome.mp4" %}

#### Hora de Experimentar!

* Carregue um segundo som, `miss.wav`, e toque-o no `else` da verificação de colisão (ou seja, quando o jogador clicar, mas errar o alvo).
* Tente usar `SetSoundVolume(hitSound, 0.5f);` (logo após o `LoadSound`) para deixar o som mais baixo. O volume vai de `0.0` (mudo) a `1.0` (máximo).
* (Avançado) Use `GetRandomValue()` para escolher entre 3 sons de acerto diferentes (`hit1.wav`, `hit2.wav`, `hit3.wav`) para que o jogo tenha mais variedade.
