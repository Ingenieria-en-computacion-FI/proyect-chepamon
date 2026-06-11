#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <time.h>
#include <SDL2/SDL.h>
#include <SDL2/SDL_image.h>

#define FILAS 15
#define COLUMNAS 21
#define TAM 32 // Tamaño en píxeles de cada casilla del mapa
#define WIDTH (COLUMNAS * TAM) // Ancho total de la ventana de juego en píxeles
#define HEIGHT (FILAS * TAM) // Alto total de la ventana de juego en píxeles
#define MAX_FANTASMAS 4
#define TOTAL_NIVELES 3

char mapa[FILAS][COLUMNAS + 1]; // Matriz que guarda el diseño del laberinto
int jugadorFila, jugadorCol, inicioJugadorFila, inicioJugadorCol; // Coordenadas actuales e iniciales de Pikachu
int velFila = 0; // Dirección de movimiento
int velCol = 0;
int fantasmaFila[MAX_FANTASMAS], fantasmaCol[MAX_FANTASMAS]; // Arreglos para controlar la posición de cada Gengar individualmente
int inicioFantasmaFila[MAX_FANTASMAS], inicioFantasmaCol[MAX_FANTASMAS];
int fantasmaDir[MAX_FANTASMAS], totalFantasmas; // Guarda hacia dónde miran los fantasmas para su sprite
int dirFantasma[MAX_FANTASMAS];
int puntos = 0, vidas = 3, pellets = 0, nivel = 1;

int dirJugador = 0;
int framePikachu = 0; // Columna actual de la animación de caminata de Pikachu
int frameGengar = 0;

Uint32 ultimoFrame = 0;

bool jugando = true, victoria = false;
SDL_Texture *pikachuTexture = NULL;
SDL_Texture *gengarTexture = NULL;

char mapasDefault[TOTAL_NIVELES][FILAS][COLUMNAS + 1] = {
    {
        "#####################",
        "#P........#........F#",
        "#.###.###.#.###.###.#",
        "#...................#",
        "#.###.#.#####.#.###.#",
        "#.....#...#...#.....#",
        "#####.### # ###.#####",
        "#.........F.........#",
        "#####.### # ###.#####",
        "#.....#...#...#.....#",
        "#.###.#.#####.#.###.#",
        "#...................#",
        "#.###.###.#.###.###.#",
        "#F........#........F#",
        "#####################"
    },
    {
        "#####################",
        "#P....#.......#....F#",
        "#.##.#.#.###.#.#.##.#",
        "#....#.........#....#",
        "####.###.#.#.###.####",
        "#......#.#.#.#......#",
        "#.####.#.....#.####.#",
        "#........F..........#",
        "#.####.#.....#.####.#",
        "#......#.#.#.#......#",
        "####.###.#.#.###.####",
        "#....#.........#....#",
        "#.##.#.#.###.#.#.##.#",
        "#F....#.......#....F#",
        "#####################"
    },
    {
        "#####################",
        "#P..#.......#......F#",
        "#.#.#.#####.#.####..#",
        "#.#...............#.#",
        "#.#####.###.#####.#.#",
        "#.....#..F..#.......#",
        "#####.#.###.#.#######",
        "#...................#",
        "#######.#.###.#.#####",
        "#.......#.....#.....#",
        "#.#.#####.###.#####.#",
        "#.#...............#.#",
        "#..####.#.#####.#.#.#",
        "#F......#.......#...#",
        "#####################"
    }
};
//Funciones 
void cargarMapa(void);
void buscarElementos(void);
void reiniciarPosiciones(void);
void moverJugador(int df, int dc);
void moverFantasmas(void);
void revisarColisiones(void);
void guardarPuntaje(void);
void dibujarCirculo(SDL_Renderer *renderer, int cx, int cy, int radio);
void renderizar(SDL_Renderer *renderer);
void actualizarTitulo(SDL_Window *window);

void cargarMapa(void)
{
    FILE *archivo;
    char nombre[20], linea[100];
    int f, c;
    //Armo el nombre del archivo según el nivel actual
    sprintf(nombre, "nivel%d.txt", nivel);
    // Intento abrir el archivo en modo lectura ("r")
    archivo = fopen(nombre, "r");
    //Recorro el mapa fila por fila para llenarlo con los caracteres correctos
    for (f = 0; f < FILAS; f++) {
        if (archivo != NULL && fgets(linea, sizeof(linea), archivo) != NULL) {
            for (c = 0; c < COLUMNAS; c++) { // Reviso cada columna de esa línea
                // Si encuentro un salto de línea o el final del texto, lo cambio por un espacio vacío
                if (linea[c] == '\n' || linea[c] == '\r' || linea[c] == '\0') {
                    mapa[f][c] = ' ';
                } else {
                    // Si es un carácter válido ('#', '.', 'P', 'F'), lo guardo en la matriz
                    mapa[f][c] = linea[c];
                }
            }
        } else { //ASi no existe el archivo .txt, uso los mapas que guardé en el código por defecto
            for (c = 0; c < COLUMNAS; c++) {
                mapa[f][c] = mapasDefault[nivel - 1][f][c];
            }
        }
        mapa[f][COLUMNAS] = '\0';
    }

    if (archivo != NULL) fclose(archivo);
}

void buscarElementos(void)
{
    int f, c;
    // Reseteo los contadores antes de empezar a escanear la matriz
    totalFantasmas = 0;
    pellets = 0;

    for (f = 0; f < FILAS; f++) {
        for (c = 0; c < COLUMNAS; c++) {
            // Si encuentro la 'P', guardo la posición actual y de inicio para Pikachu
            if (mapa[f][c] == 'P') {
                jugadorFila = inicioJugadorFila = f;
                jugadorCol = inicioJugadorCol = c;
                mapa[f][c] = ' '; // Borro la P para que no se quede estancada en el mapa
            } else if (mapa[f][c] == 'F' && totalFantasmas < MAX_FANTASMAS) {
                fantasmaFila[totalFantasmas] = inicioFantasmaFila[totalFantasmas] = f;
                fantasmaCol[totalFantasmas] = inicioFantasmaCol[totalFantasmas] = c;
                fantasmaDir[totalFantasmas] = rand() % 4;
                dirFantasma[totalFantasmas] = 0;
                totalFantasmas++;
                mapa[f][c] = ' ';
            // Si encuentro un pellet, lo sumo al contador de comida para saber cuándo ganar
            } else if (mapa[f][c] == '.') {
                pellets++;
            }
        }
    }
}

void reiniciarPosiciones(void)
{
    int i;
    // Regreso a Pikachu a sus coordenadas de inicio de fábrica
    jugadorFila = inicioJugadorFila;
    jugadorCol = inicioJugadorCol;
    // Recorro la lista de Gengars activos para mandar a cada uno a su esquina
    for (i = 0; i < totalFantasmas; i++) {
        fantasmaFila[i] = inicioFantasmaFila[i];
        fantasmaCol[i] = inicioFantasmaCol[i];
    }
}

void moverJugador(int df, int dc)
{
    // Calculo la posición a la que intenta moverse Pikachu nf=nueva fila, nc=nueva columna
    int nf = jugadorFila + df;
    int nc = jugadorCol + dc;
    // Si la nueva posición se sale de la pantalla, cancelo el movimiento
    if (nf < 0 || nf >= FILAS || nc < 0 || nc >= COLUMNAS) return;
    // Si en el destino hay una pared (#), no lo dejo pasar
    if (mapa[nf][nc] == '#') return;
    // Cambiar la dirección del sprite de Pikachu según hacia dónde camina
    if(df == -1) dirJugador = 3; // Mirar arriba
    if(df == 1)  dirJugador = 0; // Mirar abajo
    if(dc == -1) dirJugador = 1; // Mirar izquierda
    if(dc == 1)  dirJugador = 2; // Mirar derecha
    // Actualizo la posición real del jugador
    jugadorFila = nf;
    jugadorCol = nc;
    // Si la nueva casilla tiene un puntito, Pikachu se lo come
    if (mapa[nf][nc] == '.') {
        puntos += 10;
        pellets--;
        mapa[nf][nc] = ' ';
    }
}

void moverFantasmas(void)
{
    int i, intento;
    // Recorro la lista de Gengars uno por uno para moverlos
    for (i = 0; i < totalFantasmas; i++) {
        // 70% de probabilidad de que persigan a Pikachu
        if (rand() % 10 < 7) {
            if (jugadorCol < fantasmaCol[i]) fantasmaDir[i] = 2; // Ir a la izquierda
            else if (jugadorCol > fantasmaCol[i]) fantasmaDir[i] = 3; // Ir a la derecha
            else if (jugadorFila < fantasmaFila[i]) fantasmaDir[i] = 0; // Ir arriba
            else if (jugadorFila > fantasmaFila[i]) fantasmaDir[i] = 1; // Ir abajo
        } else {
            // El otro 30% de las veces, el Gengar camina al azar para despistar
            fantasmaDir[i] = rand() % 4;
        }
        // ANTI-PAREDES: Tiene hasta 4 intentos para encontrar camino libre
        for (intento = 0; intento < 4; intento++) {
            int nf = fantasmaFila[i];
            int nc = fantasmaCol[i];
            // Aplico el movimiento en el mapa según la dirección deseada
            if (fantasmaDir[i] == 0) nf--;
            if (fantasmaDir[i] == 1) nf++;
            if (fantasmaDir[i] == 2) nc--;
            if (fantasmaDir[i] == 3) nc++;
            // Cambio la orientación del sprite del Gengar para que mire a donde va
            if (fantasmaDir[i] == 0) dirFantasma[i] = 3;
            if (fantasmaDir[i] == 1) dirFantasma[i] = 0;
            if (fantasmaDir[i] == 2) dirFantasma[i] = 1;
            if (fantasmaDir[i] == 3) dirFantasma[i] = 2;
            // Si la casilla destino es segura y no es un muro (#), se mueve oficialmente
            if (nf >= 0 && nf < FILAS && nc >= 0 && nc < COLUMNAS && mapa[nf][nc] != '#') {
                fantasmaFila[i] = nf;
                fantasmaCol[i] = nc;
                break;  // Rompo el ciclo de intentos porque ya pudimos mover al Gengar
            }
            // Si el intento falló (chocó con pared #), elige otra dirección al azar y repite
            fantasmaDir[i] = rand() % 4;
        }
    }
}

void revisarColisiones(void)
{
    int i;
    // Reviso a todos los Gengars uno por uno para ver si tocaron a Pikachu
    for (i = 0; i < totalFantasmas; i++) {
        // Si las coordenadas de Pikachu y las del Gengar actual son iguales
        if (jugadorFila == fantasmaFila[i] && jugadorCol == fantasmaCol[i]) {
            vidas--;
            reiniciarPosiciones();
            SDL_Delay(400); // Pauso el juego 400 MILISEGUNDOS antes de reiniciar
            return;
        }
    }
}

void guardarPuntaje(void)
{   // Abro el archivo para añadir texto al final sin borrar lo de antes
    FILE *archivo = fopen("puntajes.txt", "a");
    // Verifico si el archivo se pudo crear o abrir bien
    if (archivo != NULL) {
        // Guardo los datos finales del jugador en una línea del bloc de notas
        fprintf(archivo, "Puntaje: %d, nivel alcanzado: %d, vidas: %d\n", puntos, nivel, vidas);
        fclose(archivo);
    }
}

void dibujarCirculo(SDL_Renderer *renderer, int cx, int cy, int radio)
{
    int x, y;

    for (y = -radio; y <= radio; y++) {
        for (x = -radio; x <= radio; x++) {
            if (x * x + y * y <= radio * radio) {
                SDL_RenderDrawPoint(renderer, cx + x, cy + y);
            }
        }
    }
}

void renderizar(SDL_Renderer *renderer)
{
    int f, c, i;
    // 1. Limpio la pantalla rellenándola de color negro (RGB: 0, 0, 0)
    SDL_SetRenderDrawColor(renderer, 0, 0, 0, 255);
    SDL_RenderClear(renderer);
    // 2. Dibujo el laberinto recorriendo la matriz casilla por casilla
    for (f = 0; f < FILAS; f++) {
        for (c = 0; c < COLUMNAS; c++) {
            SDL_Rect cuadro = {c * TAM, f * TAM, TAM, TAM};
            // Si hay un '#', pinto un bloque azul para las paredes
            if (mapa[f][c] == '#') {
                SDL_SetRenderDrawColor(renderer, 0, 60, 200, 255);
                SDL_RenderFillRect(renderer, &cuadro);
            // Si hay un '.', pinto un puntito amarillo centrado para la comida
            } else if (mapa[f][c] == '.') {
                SDL_Rect pellet = {c * TAM + 13, f * TAM + 13, 6, 6};
                SDL_SetRenderDrawColor(renderer, 255, 220, 120, 255);
                SDL_RenderFillRect(renderer, &pellet);
            }
        }
    }
    // 3. Calculo la posición real de Pikachu en píxeles dentro de la ventana
   SDL_Rect pikachuDestino = {
    jugadorCol * TAM-8, jugadorFila * TAM-8, 48, 48
    };
    // Recorto el dibujo exacto de Pikachu desde su hoja de sprites
    SDL_Rect pikachuFrame = {
    framePikachu * 64, dirJugador * 64, 64, 64
    };
    // Dibujo a Pikachu en la pantalla
    SDL_RenderCopy(renderer,
               pikachuTexture,
               &pikachuFrame,
               &pikachuDestino);
    // 4. Dibujo a todos los Gengars que estén activos usando un ciclo
    for (i = 0; i < totalFantasmas; i++) {
    // Calculo la posición en píxeles de este Gengar en la pantalla   
    SDL_Rect gengarDestino = {
        fantasmaCol[i] * TAM-8, fantasmaFila[i] * TAM-8, 48, 48
    };
    // Recorto el dibujo exacto de este Gengar según hacia dónde camina
    SDL_Rect gengarFrame = {
    frameGengar * 48, dirFantasma[i] * 48, 48, 48
};

    // Dibujo al Gengar actual en la pantalla
    SDL_RenderCopy(renderer,gengarTexture, &gengarFrame, &gengarDestino);
}

    SDL_RenderPresent(renderer);
}

void actualizarTitulo(SDL_Window *window)
{
    char titulo[120];
    sprintf(titulo, "Pac-Man | Nivel: %d/3 | Vidas: %d | Puntos: %d", nivel, vidas, puntos);
    SDL_SetWindowTitle(window, titulo);
}
Uint32 ultimoMovimientoJugador = 0;

int main(int argc, char **argv)
{
    SDL_Window *window;
    SDL_Renderer *renderer;
    SDL_Event event;
    unsigned int ultimoFantasma = 0;
    // Sincronizo los números aleatorios con el reloj de la computadora
    srand((unsigned int)time(NULL));
    // Intento encender el sistema de video de SDL
    if (SDL_Init(SDL_INIT_VIDEO) != 0) {
        printf("Error SDL: %s\n", SDL_GetError());
        return 1;
    }
    // Intento activar el soporte para imágenes PNG
    if (!(IMG_Init(IMG_INIT_PNG) & IMG_INIT_PNG)) {
    printf("Error SDL_image: %s\n", IMG_GetError());
    return 1;
}
    // Creo la ventana principal y el motor de dibujo (renderer)
    window = SDL_CreateWindow("Pac-Man", SDL_WINDOWPOS_CENTERED, SDL_WINDOWPOS_CENTERED, WIDTH, HEIGHT, 0);
    renderer = SDL_CreateRenderer(window, -1, SDL_RENDERER_ACCELERATED);

    SDL_Surface *temp;
    // Intento cargar la hoja de dibujos de Pikachu
    temp = IMG_Load("pikachu.png");

    if (temp == NULL) {
    printf("Error cargando pikachu.png: %s\n", IMG_GetError());
    } else {
    pikachuTexture = SDL_CreateTextureFromSurface(renderer, temp);
    SDL_FreeSurface(temp);
    }
    // Intento cargar la hoja de dibujos de Gengar
    temp = IMG_Load("gengar.png");

    if (temp == NULL) {
    printf("Error cargando gengar.png: %s\n", IMG_GetError());
    } else {
    gengarTexture = SDL_CreateTextureFromSurface(renderer, temp);
    SDL_FreeSurface(temp);
    }
    // Compruebo que la ventana o el renderizador no hayan fallado
    if (window == NULL || renderer == NULL) {
        printf("Error al crear ventana o renderer: %s\n", SDL_GetError());
        SDL_Quit();
        return 1;
    }
    // Cargo el primer laberinto y detecto dónde están los personajes
    cargarMapa();
    buscarElementos();
    // ==========================================
    // BUCLE PRINCIPAL (Mantiene el juego vivo)
    // ==========================================
    while (jugando) {
        // Reviso si el jugador presionó una tecla o cerró la ventan
        while (SDL_PollEvent(&event)) {
            if (event.type == SDL_QUIT) jugando = false; // Clic en la X de la ventana

            if (event.type == SDL_KEYDOWN) {
                // Salir del juego si presiona ESC o la tecla X
                if (event.key.keysym.sym == SDLK_ESCAPE || event.key.keysym.sym == SDLK_x) jugando = false;
                // Configuro la velocidad según la flecha presionada
                if (event.key.keysym.sym == SDLK_UP) {
                        velFila = -1;
                        velCol = 0;
                }

                if (event.key.keysym.sym == SDLK_DOWN) {
                        velFila = 1;
                        velCol = 0;
                }

                if (event.key.keysym.sym == SDLK_LEFT) {
                            velFila = 0;
                        velCol = -1;
                }

            if (event.key.keysym.sym == SDLK_RIGHT) {
                        velFila = 0;
                        velCol = 1;
            }   
            }
        }
        // CONTROL DE TIEMPOS
        // Cada 120 ms avanzo un fotograma de la animación de las patitas
if(SDL_GetTicks() - ultimoFrame > 120)
{
    framePikachu = (framePikachu + 1) % 4;
    frameGengar = (frameGengar + 1) % 3;

    ultimoFrame = SDL_GetTicks();
}       // Cada 300 ms obligo a los Gengars a dar un paso
        if (SDL_GetTicks() > ultimoFantasma + 300) {
            moverFantasmas();
            ultimoFantasma = SDL_GetTicks();
        }
        // Cada 180 ms permito que Pikachu avance una casilla
        if(SDL_GetTicks() - ultimoMovimientoJugador > 180)
{
    moverJugador(velFila, velCol);
    ultimoMovimientoJugador = SDL_GetTicks();
}
        // Verifico constantemente si un enemigo tocó a Pikachu
        revisarColisiones();
        // CONTROL DE VICTORIA O DERROTA
        if (vidas <= 0)
{   // Si te quedas sin vidas, salta un mensaje flotante de error
    SDL_ShowSimpleMessageBox(
        SDL_MESSAGEBOX_ERROR,
        "CHEPAMON",
        "Has perdido todas tus vidas.",
        window
    );

    jugando = false;

        } else if (pellets <= 0) {
            // Si te comes todos los pellets y hay más niveles, pasas al siguiente
            if (nivel < TOTAL_NIVELES) {
                nivel++;
                cargarMapa();
                buscarElementos();
                SDL_Delay(500); // Pausa de medio segundo para prepararse
            } else {
                // Si ya era el último nivel, ganas el juego
                victoria = true;

SDL_ShowSimpleMessageBox(
    SDL_MESSAGEBOX_INFORMATION,
    "CHEPAMON",
    "Felicidades, completaste todos los niveles.",
    window
);

jugando = false;
            }
        }
        // Actualizo el marcador de la barra superior y pinto todo en pantalla
        actualizarTitulo(window);
        renderizar(renderer);
        SDL_Delay(16);
    }
    // Al perder o ganar, guardo los puntos en el historial txt
    guardarPuntaje();
    // Muestro el resultado final resumido
    if (victoria) {
        printf("Ganaste\n");
    } else {
        printf("Perdiste\n");
    }
    printf("Puntaje final: %d\n", puntos);

    // LIBERACIÓN DE MEMORIA
    SDL_DestroyTexture(pikachuTexture);
    SDL_DestroyTexture(gengarTexture);
    IMG_Quit();
    SDL_DestroyRenderer(renderer);
    SDL_DestroyWindow(window);
    SDL_Quit();
    return 0;
}
