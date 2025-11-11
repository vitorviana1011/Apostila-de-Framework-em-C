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

# Tutorial 10 - Nasce a Orquestra

Todo bom jogo é acompanhado por uma boa trilha sonora. Essa sessão a tocar arquivos de música longos (como `.mp3` ou `.ogg`) que ficam tocando em _looping_ no fundo do nosso jogo, usando o módulo `raudio`. Usaremos o soco de clique do tutorial anterior como base ainda.

### Antes do Código

* Encontre um arquivo de música que você goste. O formato `.mp3` ou `.ogg` é o ideal para _streaming_.
* Salve o arquivo na sua pasta `recursos` com um nome simples, como `musica_fundo.mp3`.

***

### Carregando a música

O processo é muito parecido com o de `Sound`, mas usamos funções específicas para `Music`. A inicialização do áudio (`InitAudioDevice`) já deve estar lá do tutorial anterior.

```c
    InitAudioDevice();

    // Carrega a música como um "Stream"
    Music backgroundMusic = LoadMusicStream("recursos/musica_fundo.mp3");
    
    // Começa a tocar a música
    PlayMusicStream(backgroundMusic);
```

Vamos entender as novidades:

* `Music backgroundMusic = LoadMusicStream(...)`:
  * O quê: Carregamos nosso arquivo de música para uma variável do tipo `Music`.
  * Por quê: Usamos `LoadMusicStream` em vez de `LoadSound`. Isso diz à Raylib: "Não carregue esse arquivo inteiro para a RAM! Apenas prepare-se para lê-lo aos pedaços (streaming) do disco."
* `PlayMusicStream(backgroundMusic)`:
  * O quê: Damos o "play" na música.
  * Por quê: Fazemos isso uma vez antes do loop principal, para que a música comece a tocar assim que o jogo abrir.

***

### A Lógica do Streaming

Esta é a parte mais importante e diferente de usar `Music`. Como a música não está inteira na memória, precisamos dizer à Raylib a cada frame: "Ei, continue lendo o arquivo e mandando os próximos pedaços de áudio para o alto-falante".

Adicione esta linha no seu Bloco de **Lógica**. O melhor lugar é logo no início dele.

```c
UpdateMusicStream(backgroundMusic);
```

Vamos analisar esta linha:

* `UpdateMusicStream(backgroundMusic)`:
  * O quê: Esta função "alimenta" a placa de som com o próximo pedaço da música.
  * Por quê: Você deve chamar esta função a cada frame (dentro do loop `while`). Se você esquecer, `LoadMusicStream` não fará nada, e sua música tocará por alguns segundos e parará (ou nem começará).

***

### Hora da Faxina

Assim como carregamos, precisamos descarregar.

Adicione esta nova linha no seu Bloco de **Limpeza**, ao final do programa:

```c
// --- Limpeza ---
UnloadMusicStream(backgroundMusic); // Libera o stream da música
UnloadSound(hitSound);
CloseAudioDevice();
CloseWindow();
```

`UnloadMusicStream(backgroundMusic)`: Esta é a função par do `LoadMusicStream`. Ela fecha o arquivo e libera os recursos de _streaming_.

***

### Código Final

```
#include "raylib.h"

int main(){
    // --- Inicialização ---
    const int screenWidth = 800;
    const int screenHeight = 450;
    InitWindow(screenWidth, screenHeight, "Tutorial 6 - Jogo do Clique");
    SetTargetFPS(60);

    InitAudioDevice();

    // Carrega a música como um "Stream"
    Music backgroundMusic = LoadMusicStream("recursos/musica.mp3");
    
    // Começa a tocar a música
    PlayMusicStream(backgroundMusic);

    Vector2 targetPosition = { (float)screenWidth / 2, (float)screenHeight / 2 };
    float targetRadius = 30.0f;

    int score = 0;
    
    Sound hitSound = LoadSound("recursos/acerto.wav");

    while(!WindowShouldClose()){
        // --- Lógica ---

        UpdateMusicStream(backgroundMusic);

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
    UnloadMusicStream(backgroundMusic); // Libera o stream da música
    UnloadSound(hitSound);
    CloseAudioDevice();
    CloseWindow();

    return 0;
}
```

#### Hora de Experimentar

* Tente usar `SetMusicVolume(backgroundMusic, 0.5f);` (logo após o `LoadMusicStream`) para deixar a música mais baixa. O volume vai de `0.0` (mudo) a `1.0` (máximo).
* Adicione uma lógica no Bloco de Atualização para pausar a música: `if (IsKeyPressed(KEY_P)) PauseMusicStream(backgroundMusic);` e `if (IsKeyPressed(KEY_O)) ResumeMusicStream(backgroundMusic);`.
