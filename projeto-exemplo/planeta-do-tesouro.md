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
  tags:
    visible: true
---

# Planeta do Tesouro

## Introdução

O projeto Planeta do Tesouro foi desenvolvido como um estudo de caso avançado sobre a utilização da biblioteca Raylib em linguagem C. O objetivo principal foi transcender exemplos básicos, implementando uma arquitetura de software robusta para gerenciar mecânicas complexas de um jogo de aventura 2D. No controle de um pirata espacial, o jogador deve navegar por labirintos modulares, gerenciar recursos de vida, coletar tesouros e interagir com sistemas de teletransporte, tudo isso enquanto lida com uma inteligência artificial de inimigos em tempo real.

## Arquitetura do Projeto

A arquitetura do Planeta do Tesouro foi projetada seguindo o princípio da Separação de Responsabilidades. O código foi dividido em módulos independentes, o que facilitou o desenvolvimento paralelo, a manutenção e a depuração de erros. Cada módulo é composto por um par de arquivos (`.c` e `.h`), garantindo que a lógica e as interfaces estejam bem definidas.

Abaixo, detalhamos a responsabilidade de cada componente:

* `planetadotesouro.c` (Núcleo e Orquestração): Atua como o ponto de entrada da aplicação. Contém o loop principal (`Main Loop`) e gerencia a Máquina de Estados do jogo (Alternando entre Menu, Gameplay e Game Over), coordenando as chamadas das funções de alto nível.
* `logicaJogo.c` (Mecânicas de Gameplay): Contém as regras de negócio do jogo. É responsável pelo processamento da física simples, movimentação do jogador, detecção de colisões, lógica dos portais de teletransporte e o controle de progressão entre as fases.
* `manipulaArquivos.c` (Persistência e I/O): Gerencia a comunicação com o disco. Este módulo é responsável por ler os arquivos de texto que definem o layout dos mapas e por persistir os dados do sistema de ranking, garantindo que o progresso e os recordes sejam salvos.
* `inimigo.c` (Gestão de Entidades): Responsável pelo ciclo de vida dos inimigos. Gerencia desde a alocação das instâncias até a lógica de controle e movimentação desses perigos, isolando o comportamento da IA das demais lógicas do jogo.
* `desenhos.c` (Sistema de Renderização): Concentra toda a parte visual e de interface (UI/UX). Gerencia o carregamento de texturas, a renderização dos sprites na tela, a criação de botões interativos e a montagem das diferentes telas do sistema.
* `audio.c` (Motor de Áudio): Centraliza o gerenciamento de som do projeto. Responsável pelo carregamento e execução dos efeitos sonoros (SFX) disparados por eventos e pela reprodução contínua da trilha sonora (BGM).

## Sistema de Mapas e Level Design

A construção dos níveis em Planeta do Tesouro utiliza um sistema de Tile-based Rendering (renderização baseada em blocos). Em vez de desenhar um mapa estático, o jogo constrói o cenário dinamicamente a cada execução, lendo as instruções de arquivos de texto externos.

**1. O Formato dos Dados (.txt)**

Os mapas são armazenados na pasta `mapas/` como arquivos `.txt` simples (ex: `mapa1.txt`). Cada caractere dentro do arquivo representa um elemento específico no mundo do jogo:

* `#` : Representa uma Parede (obstáculo intransponível).
* `.` : Representa o Chão (área de livre circulação).
* `@` : Ponto de início do Jogador.
* `T` : Localização de um Tesouro.
* `I` : Ponto de spawn de um Inimigo.
* `C`: Adiciona uma vida ao Jogador

```
// Exemplo de como o jogo vê o mapa internamente:
13 22
######################
#@........##........T#
#.######..##..######.#
#.#....#..I...#....#.#
#.#.T#.#..1...#.#T.#.#
#.#..#.########.#..#.#
#....#.....C....#....#
#.#..#.########.#..#.#
#.#I.#.#..1...#.#.I#.#
#.#....#......#....#.#
#.######..##..######.#
#T........##........T#
######################
```

**2. O Processo de Parsing (Módulo `manipulaArquivos.c`)**

A inteligência do sistema reside na função de carregamento. O módulo percorre o arquivo de texto linha por linha e coluna por coluna:

1. Leitura: O código abre o arquivo correspondente à fase atual.
2. Mapeamento: Para cada caractere lido, o sistema calcula uma coordenada no mundo real (multiplicando a posição do caractere pelo tamanho do _tile_, ex: 32x32 pixels).
3. Instanciação: Os objetos são carregados na memória: paredes são adicionadas à lista de colisões e itens são posicionados para interação.

A função central para transformar o texto em jogo é a `carregaMapa`. Ela utiliza funções de entrada e saída padrão de C para alocar a memória necessária e ler os caracteres.

```c
// Trecho simplificado de manipulaArquivos.c
Mapa carregaMapa(int fase, Inimigo **inimigos) {
    Mapa mapa = {0};
    char caminho[50];
    snprintf(caminho, sizeof(caminho), "mapas/mapa%d.txt", fase); // Gera o caminho do arquivo

    FILE *arq = fopen(caminho, "r");
    if (arq != NULL) {
        // Captura as dimensões da primeira linha (Ex: 10 15)
        if(tamanhoMapa(arq, &mapa.linhas, &mapa.colunas)){
            // Alocação dinâmica da matriz que guardará os dados
            mapa.dados = malloc(mapa.linhas * sizeof(char*));
            for(int i = 0; i < mapa.linhas; i++){
                mapa.dados[i] = malloc((mapa.colunas + 1) * sizeof(char));
                fscanf(arq, "%s", mapa.dados[i]); // Lê a linha do mapa
            }
        }
        fclose(arq);
    }
    return mapa;
}
```

O uso de `malloc` permite que o jogo suporte mapas de qualquer tamanho (dentro dos limites de memória), tornando o sistema flexível. Cada linha do arquivo é lida como uma _string_ e armazenada na matriz `mapa.dados`.

**3. Renderização e Escala**

Após carregar a matriz de caracteres, o jogo percorre os dados para identificar onde estão os tesouros, inimigos e portais. Isso é feito na função `contarElementosMapa`.

```c
// Trecho de manipulaArquivos.c
void contarElementosMapa(Mapa *mapa, int *totalTesouros, int *totalInimigos, Inimigo **inimigos) {
    for (int i = 0; i < mapa->linhas; i++) {
        for (int j = 0; j < mapa->colunas; j++) {
            if (mapa->dados[i][j] == 'T') {
                (*totalTesouros)++; // Identifica um Tesouro
            } else if (mapa->dados[i][j] == 'I') {
                (*totalInimigos)++; // Identifica e instancia um Inimigo
                // ... lógica de alocação de memória para o inimigo ...
            }
        }
    }
    processarPortais(mapa); // Identifica os pares de teleporte
}
```

Esta varredura é essencial para a Lógica de Jogo. Ela não apenas conta os itens, mas também inicializa as coordenadas de entidades dinâmicas (como inimigos) que se moverão de forma independente pelo mapa durante o loop principal.

No módulo `desenhos.c`, o jogo utiliza as texturas de `parede.png`, `chao.png` e `portal.png` para preencher esses espaços. A grande vantagem desta técnica é a eficiência de memória, já que o jogo carrega apenas uma pequena imagem de parede e a repete centenas de vezes, em vez de carregar uma imagem gigante de todo o labirinto.

**4. Progressão de Fases**

A lógica de progressão foi implementada de forma modular no arquivo `logicaJogo.c`. Quando o sistema detecta uma colisão entre o jogador e o objeto 'Portal', o jogo:

1. Limpa os dados da fase atual.
2. Incrementa o contador de nível.
3. Solicita ao `manipulaArquivos.c` que carregue o próximo ficheiro (ex: `mapa2.txt`).

Para garantir que o jogo não consuma memória desnecessária ao trocar de fase, implementamos uma função de limpeza que libera a matriz dinâmica.

```c
// Trecho de manipulaArquivos.c
void liberaMapa(Mapa *mapa){
    for(int i = 0; i < mapa->linhas; i++){
        free(mapa->dados[i]); // Libera cada linha
    }
    free(mapa->dados); // Libera o ponteiro principal
    mapa->dados = NULL;
}
```

***

#### Vantagens Técnicas desta Abordagem

* Facilidade de Edição: É possível criar novas fases ou alterar labirintos existentes apenas editando um bloco de notas, sem a necessidade de alterar uma única linha de código C ou recompilar o projeto.
* Escalabilidade: O sistema está preparado para suportar dezenas de níveis diferentes apenas adicionando novos arquivos `.txt` à pasta de mapas.
* Desempenho: O uso de mapas baseados em caracteres é extremamente leve, permitindo tempos de carregamento quase instantâneos.

***

## Gameplay

O núcleo de gameplay foi projetado para ser uma mistura de exploração estratégica e reflexos rápidos. O jogador assume o papel de um pirata espacial navegando por labirintos desconhecidos, onde cada movimento deve ser calculado para otimizar o tempo e preservar a vida.

**1. Movimentação e Resposta Tátil**

A movimentação é o pilar principal da interação. O jogador controla o pirata utilizando as setas do teclado ou as teclas WASD.

* Sistema de Grid: Embora a movimentação pareça fluida, ela é governada pelas restrições do mapa baseado em blocos (_tiles_).
* Detecção de Colisões: Antes de cada passo, o jogo verifica se a coordenada de destino na matriz do mapa contém uma parede (`#`). Se houver um obstáculo, o movimento é bloqueado, garantindo que o jogador permaneça dentro dos limites do labirinto.

A movimentação é processada verificando a entrada do teclado e validando a posição de destino contra obstáculos no mapa.

```c
// Trecho de logicaJogo.c
if (IsKeyPressed(KEY_UP)) novoY = jogador->y - 1;
else if (IsKeyPressed(KEY_DOWN)) novoY = jogador->y + 1;
else if (IsKeyPressed(KEY_LEFT)) novoX = jogador->x - 1;
else if (IsKeyPressed(KEY_RIGHT)) novoX = jogador->x + 1;

// Verificação de colisão com paredes
if (mapa->dados[novoY][novoX] == '#') return; 
```

O sistema utiliza `IsKeyPressed` para capturar o movimento por "casas". Antes de atualizar a posição do jogador, o código acessa a matriz `mapa->dados` para garantir que o destino não seja uma parede (`#`).

**2. Sobrevivência e Gestão de Vida**

O pirata inicia sua jornada com um número limitado de vidas (geralmente três).

* Perigos (Inimigos): Entidades hostis patrulham o mapa. Ao entrar em contato com um inimigo, o jogador sofre dano imediato, perdendo uma vida.
* Itens de Cura: Espalhados estrategicamente, o caractere `C` no mapa representa suprimentos médicos que restauram a saúde do jogador, incentivando a exploração de áreas arriscadas.
* Game Over: Se o contador de vidas chegar a zero, o estado do jogo muda para `GAME_OVER`, forçando o jogador a decidir entre reiniciar a jornada ou retornar ao menu.

O jogo gerencia a saúde do jogador através de colisões com inimigos e itens de cura (`C`).

```c
// Trecho de logicaJogo.c
void verificaColisaoComInimigos(Jogador *jogador, Inimigo *inimigos, Mapa *mapa, int *statusJogo) {
    if (GetTime() > jogador->tempoInvencibilidade) {
        for (int i = 0; i < mapa->totalInimigos; i++) {
            if (jogador->x == inimigos[i].x && jogador->y == inimigos[i].y) {
                jogador->vidas--; // Reduz vida
                jogador->tempoInvencibilidade = GetTime() + 2.0; // Invencibilidade temporária
                verificaGameOver(jogador, statusJogo);
            }
        }
    }
}
```

Se a coordenada do jogador coincidir com a de um inimigo, o contador de vidas diminui. Um temporizador de invencibilidade impede que o jogador perca todas as vidas instantaneamente no mesmo encontro.

**3. Mecânica de Coleta e Pontuação**

O objetivo principal em cada fase é a riqueza.

* Tesouros (`T`): Cada tesouro coletado aumenta a pontuação do jogador.
* Progressão Obrigatória: Diferente de jogos de exploração livre, em algumas fases pode ser necessário coletar uma quantidade específica de tesouros para que finalize a fase.

A progressão depende da coleta total de tesouros (`T`) presentes no nível.

```c
// Trecho de logicaJogo.c
if (confereTesouro(mapa, novoX, novoY)) {
    (*tesouroColetados)++;
    if (*tesouroColetados >= mapa->totalTesouros) {
        *statusJogo = ENTRE_FASES; // Condição de vitória da fase
    }
}
```

A cada tesouro coletado, o contador `tesouroColetados` aumenta. O jogo compara este valor com o `totalTesouros` (calculado durante o carregamento do mapa) para mudar o estado para `ENTRE_FASES`.

**4. Sistema de Teletransporte (Portais Numéricos)**

Uma das mecânicas mais complexas do projeto é o sistema de portais numerados (de 1 a 9).

* Pares Conectados: Os portais funcionam em duplas. Ao entrar em um bloco marcado com o número `1`, o jogador é instantaneamente transportado para a coordenada do outro bloco `1` no mapa.
* Estratégia de Navegação: Essa mecânica permite criar labirintos não-lineares, onde o caminho mais curto entre dois pontos pode envolver um salto espacial através de um portal.

Os portais numerados permitem deslocamento instantâneo entre pares de coordenadas.

```c
// Trecho de logicaJogo.c
void verificaPortalNumerado(Jogador *jogador, Mapa *mapa, int y, int x) {
    for (int i = 0; i < mapa->totalPortais; i++) {
        if (mapa->portais[i].x1 == x && mapa->portais[i].y1 == y) {
            jogador->x = mapa->portais[i].x2; // Vai para a posição 2
            jogador->y = mapa->portais[i].y2;
            return;
        }
        // ... lógica inversa para retornar do portal 2 para o 1 ...
    }
}
```

O sistema percorre a lista de portais pré-mapeados. Ao detectar que o jogador entrou em um ponto (ex: `x1, y1`), ele redefine as coordenadas do jogador para a saída correspondente (`x2, y2`).

**5. Condição de Vitória e Transição de Fase**

Quando o status do jogo muda para `ENTRE_FASES` (após a coleta total), o jogador visualiza uma tela de transição.

* Próximo Desafio: Ao pressionar a tecla `ENTER`, o sistema libera a memória do mapa atual, incrementa o nível e carrega o arquivo `.txt` da próxima fase.

Ao avançar, o jogo libera os recursos da fase antiga e carrega os novos dados.

```c
// Trecho de logicaJogo.c
void proximaFase(Mapa *mapa, Jogador *jogador, int *statusJogo, int *tesouroColetados, int *fase, Inimigo **inimigo, Cronometro *cronometro) {
    finalizarFase(cronometro); // Registra tempo
    int novaFase = (*fase) + 1;
    Mapa novoMapa = carregaMapa(novaFase, inimigo); // Tenta carregar novo .txt
    
    if (novoMapa.dados != NULL) {
        liberaMapa(mapa); // Limpa memória do mapa anterior
        *mapa = novoMapa;
        *statusJogo = JOGANDO;
    } else {
        *statusJogo = TELA_RESULTADOS; // Fim de todas as fases
    }
}
```

A função `proximaFase` orquestra a transição: encerra o tempo da fase atual, tenta carregar o próximo arquivo de mapa e, caso não existam mais fases, encaminha o jogador para os resultados finais.

**6. O Elemento Speedrun (Cronômetro)**

Para adicionar uma camada de competitividade, o jogo integra um sistema de cronometragem precisa.

* Tempo Total vs. Tempo por Fase: O jogo registra quanto tempo o jogador leva para concluir cada mapa individualmente e o tempo total da "run".
* Ranking: Ao final do jogo, o tempo total é utilizado para classificar o jogador no sistema de recordes, incentivando o aprimoramento das rotas e a agilidade.

O tempo é monitorado para alimentar o sistema de ranking e estatísticas.

```c
// Trecho de manipulaArquivos.c
void finalizarFase(Cronometro *cronometro) {
    cronometro->fimFase = GetTime();
    cronometro->tempoFase = cronometro->fimFase - cronometro->inicioFase;
    cronometro->temposPorFase[cronometro->faseAtual - 1] = cronometro->tempoFase;
}
```

O módulo de arquivos gerencia o tempo capturando o `GetTime()` da Raylib. A diferença entre o início e o fim da fase é armazenada em um array, permitindo gerar relatórios detalhados de desempenho ao fim da partida.

***

## Visuais

O aspecto visual do jogo foi desenvolvido para transformar uma matriz abstrata de caracteres em uma experiência imersiva com estética _pixel art_. A renderização utiliza o pipeline gráfico da Raylib, onde cada elemento é desenhado em camadas específicas durante o loop principal para garantir a profundidade visual correta.

**1. Gerenciamento de Texturas (Assets)**

O jogo não utiliza formas geométricas básicas para os elementos principais; em vez disso, ele carrega arquivos externos de imagem (`.png`) que definem a personalidade do pirata espacial e de seu universo.

* Sprites de Cenário: Imagens separadas para paredes, chão e o fundo do espaço (`background.png`), permitindo que o mapa pareça um ambiente coeso e não apenas um labirinto genérico.
* Entidades Dinâmicas: Sprites exclusivos para o Jogador, Inimigos e Itens (Tesouros, Vidas e Portais), cada um com dimensões que respeitam o sistema de _tiles_ do mapa.
* Elementos de Interface: Uso de logotipos (`logo.png`) e ícones de status (`vida.png`) para compor a parte informativa da tela.

O jogo utiliza estruturas `Texture2D` para armazenar todos os elementos gráficos em memória RAM de vídeo, carregando-os a partir de arquivos externos.

```c
// Trecho de desenhos.c
static Texture2D texJogador = {0};
static Texture2D texInimigo = {0};
static Texture2D texTesouro = {0};

void carregarRecursos(void) {
    // Carregamento de arquivos PNG do diretório de recursos
    texJogador = LoadTexture("recursos/sprites/jogador.png");
    texInimigo = LoadTexture("recursos/sprites/inimigo.png");
    texTesouro = LoadTexture("recursos/sprites/tesouro.png");
    // ... carregamento de outros recursos ...
}
```

A função `carregarRecursos` centraliza a importação dos arquivos `.png`, transformando-os em objetos de textura que a Raylib pode desenhar na tela com alta performance.

**2. Sistema de Camadas de Desenho**

Para evitar que elementos se sobreponham de forma errada, o módulo de desenhos segue uma hierarquia rigorosa dentro de cada frame:

* Fundo e Mapa: Primeiro, a tela é limpa e o cenário estático (chão e paredes) é renderizado.
* Objetos de Interação: Portais e itens são desenhados sobre o chão.
* Entidades Ativas: Inimigos são renderizados em movimento.
* Jogador (Topo): O pirata é desenhado por último, muitas vezes com efeitos visuais (como transparência ou brilho) para indicar estados especiais, como a invencibilidade temporária após sofrer dano.

O cenário é desenhado percorrendo a matriz de dados do mapa e substituindo cada caractere pelo seu sprite correspondente.

```c
// Trecho de desenhos.c
void desenhaMapa(Mapa mapa) {
    for (int i = 0; i < mapa.linhas; i++) {
        for (int j = 0; j < mapa.colunas; j++) {
            char tile = mapa.dados[i][j];
            // ... cálculo de coordenadas x, y baseado no tileSize ...
            switch (tile) {
                case '#': // Desenha a parede
                    DrawTexturePro(texParede, srcParede, dest, origin, 0.0f, WHITE);
                    break;
                case 'T': // Desenha o tesouro sobre o chão
                    DrawTexturePro(texChao, srcChao, dest, origin, 0.0f, WHITE);
                    DrawTexturePro(texTesouro, src, dest, origin, 0.0f, WHITE);
                    break;
            }
        }
    }
}
```

O uso do `switch` permite identificar rapidamente o tipo de bloco e renderizar a textura correta. Note que itens como tesouros desenham primeiro o chão por baixo, garantindo a coesão visual do cenário.

Para elementos como portais e o estado de invencibilidade do jogador, o jogo utiliza funções de tempo para criar dinamismo.

```c
// Trecho de desenhos.c
// Animação do portal baseada no tempo
float tempo = GetTime();
int frameAtual = (int)(tempo * 4.0f) % 3; // 3 frames de animação
Rectangle srcPortal = { frameAtual * 32.0f, 0.0f, 32.0f, 32.0f };
DrawTexturePro(texPortal, srcPortal, dest, origin, 0.0f, WHITE);

// Efeito de invencibilidade do jogador (piscar)
if (jogador.tempoInvencibilidade > GetTime()) {
    if (((int)(GetTime() * 10)) % 2 == 0) {
        DrawTexturePro(texJogador, src, dest, origin, 0.0f, WHITE);
    } // Alterna entre visível e invisível/transparente
}
```

A função `GetTime()` é usada para calcular qual frame da folha de sprite do portal deve ser exibido, criando uma animação de 3 quadros. O mesmo princípio é usado para fazer o jogador "piscar" após sofrer dano.

**3. Interface de Usuário (UI) e Telas de Estado**

O jogo possui um sistema completo de telas que gerenciam a navegação do usuário:

* Menus e Tutoriais: Telas dedicadas com botões interativos que utilizam detecção de clique do mouse para iniciar o jogo ou visualizar instruções.
* HUD (Heads-Up Display): Durante a partida, uma interface persistente exibe o nível atual, o contador de tesouros, as vidas restantes e o cronômetro em tempo real.
* Telas de Feedback: Transições entre fases, telas de Game Over e o sistema de entrada de nome para o ranking, que utiliza renderização dinâmica de texto conforme o jogador digita.

***

## Efeitos Sonoros e Musicas

O áudio no Planeta do Tesouro atua como a camada de feedback imediato, utilizando o módulo `raudio` da Raylib para processar sons simultâneos e músicas contínuas. O sistema é dividido entre trilhas de longa duração e efeitos disparados por eventos de gameplay.

**1. Música de Fundo (Streaming)**

O jogo utiliza _streaming_ de áudio para trilhas longas, o que economiza memória ao não carregar o arquivo inteiro de uma vez para a RAM.

* Carregamento: No módulo de áudio, as trilhas são preparadas como fluxos (_streams_).

```c
// Trecho de audio.c
musicaPrincipal = LoadMusicStream("recursos/sons/musicaPrincipal.mp3");
musicaMenu = LoadMusicStream("recursos/sons/musicaMenu.mp3");
SetMusicVolume(musicaPrincipal, 0.3f); // Volume controlado para não abafar os efeitos
```

* Processamento em Tempo Real: Diferente dos sons curtos, a música precisa ser atualizada a cada frame do loop principal para continuar a reprodução.

```c
// Trecho de planetadotesouro.c (Loop Principal)
while(!WindowShouldClose()){
    atualizarMusica(); // Internamente executa UpdateMusicStream() para manter o áudio fluindo
    // ...
}
```

**2. Efeitos Sonoros (SFX)**

Sons curtos são carregados integralmente na memória para garantir que sejam disparados instantaneamente por eventos da lógica do jogo.

* Implementação de Disparo: Funções específicas encapsulam o comando de reprodução para facilitar a chamada em outros módulos.

```c
// Trecho de audio.c
void tocarSomTesouro(void) {
    if (sistemaAudioPronto()) PlaySound(somTesouro); // Reprodução instantânea
}
```

* Exemplo de Feedback Positivo: Disparado no módulo de lógica quando o jogador coleta um item.

```c
// Trecho de logicaJogo.c
if (confereTesouro(mapa, novoX, novoY)) {
    tocarSomTesouro(); // Som metálico de recompensa
    (*tesouroColetados)++;
    // ...
}
```

**3. Orquestração por Estados de Jogo**

O sistema de áudio é orquestrado de forma que, ao mudar o estado do jogo (ex: de "Jogando" para "Game Over"), as músicas de fundo são trocadas instantaneamente, mantendo a sincronia com o visual.

```c
// Trecho de planetadotesouro.c
if (statusJogo != statusJogoAnterior) {
    switch (statusJogo) {
        case MENU:
            pararMusicaPrincipal();
            iniciarMusicaMenu();
            break;
        case JOGANDO:
            pararMusicaMenu();
            iniciarMusicaPrincipal();
            break;
        case GAME_OVER:
            pararMusicaPrincipal();
            tocarSomGameOver(); // Som melancólico de derrota
            iniciarMusicaMenu();
            break;
    }
}
```

A máquina de estados no `main` garante que apenas uma trilha de fundo esteja ativa por vez, gerenciando o início e a parada dos _streams_ conforme o jogador navega entre os menus e as fases.

## Conclusão

O desenvolvimento de Planeta do Tesouro cumpriu com êxito o desafio de criar uma aplicação robusta e modularizada, indo além de conceitos básicos de programação. O projeto serviu como uma vitrine para a implementação de sistemas essenciais em um motor de jogo, como o carregamento dinâmico de mapas via arquivos externos, a gestão de estados de jogo e o processamento de áudio em tempo real.

A utilização da linguagem C permitiu um controle rigoroso sobre a memória, especialmente na alocação dinâmica das matrizes de níveis e na gestão de entidades como inimigos e portais. Aliada a isso, a biblioteca Raylib proveu as ferramentas necessárias para uma renderização eficiente de sprites e uma interface de usuário responsiva.

***

#### Repositório do Projeto

O código-fonte completo, incluindo todos os módulos, arquivos de cabeçalho e recursos gráficos/sonoros, está disponível para consulta e contribuição no GitHub:

[🔗 Acesse o Repositório no GitHub](https://github.com/vitorviana1011/PlanetaDoTesouro.git)
