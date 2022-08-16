#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <math.h>


#define VOID ' '
#define WALL '#'
#define FOOD '*'
#define MONEY '$'
#define BODY 'o'
#define HEAD_UP '^'
#define HEAD_DOWN 'v'
#define HEAD_RIGHT '>'
#define HEAD_LEFT '<'
#define MOV_CONT 'c'
#define MOV_CW 'h'
#define MOV_CCW 'a'
#define DEATH_CHARACTER 'X'


typedef struct{
    int win;
    int lose;
    int heatmap[100][100];
    int movements;
    int woPoints;
    int upMove;
    int downMove;
    int leftMove;
    int rightMove;
    int foodCounter;
    int moneyCounter;
    int score;
}tStats;

typedef struct{
    int lines;
    int columns;
    char arr[100][100];
    int foodQuantity;
    int moneyQuantity;
}tMap;

typedef struct{
    int bodyCount;
    int bodyLine[100];
    int bodyColumn[100];
    int lastHeadLine;
    int lastHeadColumn;
    int headLine;
    int headColumn;
    int headPos[100][100];
    char currHead;
}tSnake;

typedef struct{
    tSnake snake;
    tStats stats;
    tMap map;
}tGame;

tGame StartGame (tGame game, FILE *fresume);
tGame MoveHead (tGame game, char mov);

tMap ReadMap (FILE *gameMap, tMap map);
tMap ReadInitialState (char *ini);
tMap RefreshMap (tMap map, tSnake snake, char mov);
tMap UpdateVoid(tSnake snake, tMap map);
tMap CountMoneyFood(tMap map); 

char GetCurrHead(tSnake snake);

tSnake FindHead (tMap map);
tSnake UpdateHead(tSnake snake);
tSnake MoveHeadUp (tSnake snake, tMap map);
tSnake MoveHeadDown (tSnake snake, tMap map);
tSnake MoveHeadRight (tSnake snake, tMap map);
tSnake MoveHeadLeft (tSnake snake, tMap map);
tSnake RefreshBody (tSnake snake);
tSnake CountBody(tMap map);
tSnake AddBody (tSnake snake);
tSnake RefreshLastPos(tSnake snake); 

int IsWall (tMap map, int i, int j);
int IsBody (tMap map, int i, int j);
int IsHead (tMap map, int i, int j);
int CountMoney(tMap map);
int CountFood(tMap map);
int CheckFoodMoney(tMap map, tSnake snake);

tStats AddMov(tStats stats);
tStats AddMovUp(tStats stats);
tStats AddMovDown(tStats stats);
tStats AddMovLeft(tStats stats);
tStats AddMovRight(tStats stats);
tStats AddMovWoPoint(tStats stats);
tStats CheckWhichScore(tStats stats, tSnake snake, tMap map);
tStats InitializeHeatmap(tStats stats, tMap map);
tStats RefreshHeatMap(tStats stats, tSnake snake);
tStats SetWin(tStats stats);
tStats SetLose(tStats stats);
tStats InitializeStatsVariables(tStats stats);

void PrintInitialState(FILE *init, tMap map, tSnake snake);
void PrintStats(FILE *fstats, tStats stats);
void PrintResume(FILE *fresume, tStats stats, char mov, int type);
void PrintHeatmap(FILE *fheatmap, tStats stats, tMap map);
void PrintRanking(FILE *franking, tStats stats, tMap map);
void PrintMap(tStats stats, tMap map, char mov);

void PrintDeath (tStats stats, tMap map, tSnake snake, char mov);
void PrintWin(tStats stats, tMap map, tSnake snake, char mov);

int LoseCondition (tMap map, tSnake snake);
int WinCondition (tStats stats, tMap map);

int main (int argc, char *argv[])
{
    if(argc <= 1)
    {
        printf("ERRO: O diretorio de arquivos de configuracao nao foi informado");

        return 0;
    }
    tGame game;

    char directory[1001];

    FILE *gameMap;
    sprintf(directory, "%s/mapa.txt", argv[1]);
    gameMap = fopen(directory, "r");

    if (!gameMap)
    {
        printf("ERRO AO ABRIR mapa.txt\n");
        return 0;
    }
    
    game.map = ReadMap(gameMap, game.map);
    
    char directory_init[1001];
    FILE *init;
    sprintf(directory_init, "%s/saida/inicializacao.txt", argv[1]);
    init = fopen (directory_init, "w");
    PrintInitialState (init, game.map, game.snake);
    fclose(init);

    char directory_resume[1001];
    FILE *fresume;
    sprintf(directory_resume, "%s/saida/resumo.txt", argv[1]);
    fresume = fopen (directory_resume, "w");

    game = StartGame(game, fresume);

    fclose(fresume);
    char directory_stats[1001];
    FILE *fstats;
    sprintf(directory_stats, "%s/saida/estatisticas.txt", argv[1]);
    fstats = fopen (directory_stats, "w");
    PrintStats(fstats, game.stats);
    fclose(fstats);

    char directory_heatmap[1001];
    FILE *fheatmap;
    sprintf(directory_heatmap, "%s/saida/heatmap.txt", argv[1]);
    fheatmap = fopen (directory_heatmap, "w");
    PrintHeatmap(fheatmap, game.stats, game.map);
    fclose(fheatmap);

    char directory_ranking[1001];
    FILE *franking;
    sprintf(directory_ranking, "%s/saida/ranking.txt", argv[1]);
    franking = fopen (directory_ranking, "w");
    PrintRanking(franking, game.stats, game.map);
    fclose(franking);

    return 0;
}

tGame StartGame(tGame game, FILE *fresume)
{
    char mov;

    //initializing variables
    game.stats = InitializeHeatmap(game.stats, game.map);
    game.snake = FindHead(game.map);
    game.snake.currHead = HEAD_RIGHT;
    game.map = CountMoneyFood(game.map);
    game.stats = InitializeStatsVariables(game.stats);
    game.stats = RefreshHeatMap(game.stats, game.snake);
    game.snake.bodyCount = 0;

    while(scanf("%c%*c", &mov) == 1) // here we start to receive commands from terminal
    {
        game.snake = RefreshLastPos(game.snake); //refresh the last snake pos
        game.map = UpdateVoid(game.snake, game.map); // update the void behind the snake
        game = MoveHead(game, mov); // actually move the head
        game.stats = RefreshHeatMap(game.stats, game.snake); // add the move to the heatmap
        game.stats = CheckWhichScore(game.stats, game.snake, game.map); // check what score will the player receive

        if (LoseCondition(game.map, game.snake)) // check if the player hit wall or body, then proceed to lose
        {
            PrintResume(fresume, game.stats, mov, 4);
            PrintDeath(game.stats, game.map, game.snake, mov);
            game.stats = SetLose(game.stats);
            break;
        }
        
        if (WinCondition(game.stats, game.map)) // check if the number of food eaten is equal to the food map quantity, if so, wins
        {
            PrintResume(fresume, game.stats, mov, 3);
            game.map = RefreshMap(game.map, game.snake, mov);
            PrintWin(game.stats, game.map, game.snake, mov);
            game.stats = SetWin(game.stats);
            break;
        } 

        if (CheckFoodMoney(game.map, game.snake) == 1) // refresh the resume
        {
            PrintResume(fresume, game.stats, mov, 1);
        }
        else if (CheckFoodMoney(game.map, game.snake) == 2)
        {
            PrintResume(fresume, game.stats, mov, 2);
        }

        game.map = RefreshMap(game.map, game.snake, mov); // refresh the map with current game state

        PrintMap(game.stats, game.map, mov);
    }

    return game;
}

tMap ReadMap (FILE *gameMap, tMap map)
{
    fscanf(gameMap, "%d %d%*c", &map.lines, &map.columns);
    int i, j;

    for(i = 0; i < map.lines; i++)
    {
        for(j = 0; j < map.columns; j++)
        {
            fscanf(gameMap, "%c", &map.arr[i][j]);
        }
        fscanf(gameMap, "%*c");
    }

    return map;
}

void PrintInitialState(FILE *init, tMap map, tSnake snake)
{
    int i, j;

    for(i = 0; i < map.lines; i++)
    {
        for(j = 0; j < map.columns; j++)
        {
            fprintf(init, "%c", map.arr[i][j]);
        }
        fprintf(init, "\n");
    }

    snake = FindHead(map);

    fprintf(init, "A cobra comecara o jogo na linha %d e coluna %d\n", snake.headLine + 1, snake.headColumn + 1);
}

void PrintStats(FILE *fstats, tStats stats)
{
    fprintf(fstats, "Numero de movimentos: %d\n", stats.movements);
    fprintf(fstats, "Numero de movimentos sem pontuar: %d\n", stats.woPoints);
    fprintf(fstats, "Numero de movimentos para baixo: %d\n", stats.downMove);
    fprintf(fstats, "Numero de movimentos para cima: %d\n", stats.upMove);
    fprintf(fstats, "Numero de movimentos para esquerda: %d\n", stats.leftMove);
    fprintf(fstats, "Numero de movimentos para direita: %d\n", stats.rightMove);

    return;
}

void PrintDeath (tStats stats, tMap map, tSnake snake, char mov)
{
    int i, j;
    

    map.arr[snake.headLine][snake.headColumn] = DEATH_CHARACTER; // if the snake dies, change all the snake characters to X

    for(i = 0; i < snake.bodyCount; i++)
    {
        map.arr[snake.bodyLine[i]][snake.bodyColumn[i]] = DEATH_CHARACTER;
    }
    
    printf("\nEstado do jogo apos o movimento '%c':\n", mov);

    for(i = 0; i < map.lines; i++)
    {
        for(j = 0; j < map.columns; j++)
        {
            printf("%c", map.arr[i][j]);
        }
        printf("\n");
    }

    printf("Pontuacao: %d\nGame over!\nPontuacao final: %d\n", stats.score, stats.score);
}

void PrintMap(tStats stats, tMap map, char mov)
{
    int i, j;

    printf("\nEstado do jogo apos o movimento '%c':\n", mov);

    for(i = 0; i < map.lines; i++)
    {
        for(j = 0; j < map.columns; j++)
        {
            printf("%c", map.arr[i][j]);
        }
        printf("\n");
    }

    printf("Pontuacao: %d\n", stats.score);
}

void PrintWin(tStats stats, tMap map, tSnake snake, char mov)
{
    int i, j;

    printf("\nEstado do jogo apos o movimento '%c':\n", mov);

    for(i = 0; i < map.lines; i++)
    {
        for(j = 0; j < map.columns; j++)
        {
            printf("%c", map.arr[i][j]);
        }
        printf("\n");
    }

    printf("Pontuacao: %d\n", stats.score);
    printf("Voce venceu!\n");
    printf("Pontuacao final: %d\n", stats.score);
}

tMap RefreshMap (tMap map, tSnake snake, char mov)
{
    int i, j;
    map.arr[snake.headLine][snake.headColumn] = snake.currHead; // refresh the head

    for(i = 0; i < snake.bodyCount; i++)
    {
        map.arr[snake.bodyLine[i]][snake.bodyColumn[i]] = BODY; // refresh the body w current positions
    }

    return map;
}

tMap CountMoneyFood(tMap map)
{
    int i, j;

    for(i = 0; i < map.lines; i++)
    {
        for(j = 0; j < map.columns; j++)
        {
            if(map.arr[i][j] == MONEY)
            {
                map.moneyQuantity++;
            }
            else if (map.arr[i][j] == FOOD)
            {
                map.foodQuantity++;
            }
            
        }
    }

    return map;
}


tSnake FindHead(tMap map)
{
    tSnake snake;
    int i, j;

    for(i = 0; i < map.lines; i++)
    {
        for(j = 0; j < map.columns; j++)
        {
            if(IsHead(map, i, j))
            {
                snake.headLine = i;
                snake.headColumn = j;
                snake.headPos[i][j] = HEAD_RIGHT;
            }
        }
    }

    return snake;
}


int IsHead(tMap map, int i, int j)
{
    if(    map.arr[i][j] == HEAD_RIGHT
        || map.arr[i][j] == HEAD_LEFT
        || map.arr[i][j] == HEAD_DOWN
        || map.arr[i][j] == HEAD_UP
      )
    {
        return 1;
    }
    return 0;
}

int IsUpperHead(char currHead)
{
    return currHead == HEAD_UP; 
}

int IsRightHead(char currHead)
{
    return currHead == HEAD_RIGHT;
}

int IsLeftHead(char currHead)
{
    return currHead == HEAD_LEFT;
}

int IsBottomHead(char currHead)
{
    return currHead == HEAD_DOWN;
}


int CheckFoodMoney(tMap map, tSnake snake)
{
    if(map.arr[snake.headLine][snake.headColumn] == FOOD)
    {
        return 1;
    }
    else if (map.arr[snake.headLine][snake.headColumn] == MONEY)
    {
        return 2;
    }
    else
        return 0;
}


tGame MoveHead (tGame game, char mov)
{
    char head;
    game.stats = AddMov(game.stats);
    head = GetCurrHead(game.snake); // refresh the current head

    // here, we check in which conditions the head will move in which direction
    // example: the player press c, and the head is currently at the up direction, then it will MoveHeadUp
    // ex2: the player press h, and the head is currently at the up direction, the head will move to the right, > MoveHeadRight
    if((mov == MOV_CONT && IsUpperHead(head)) || (mov == MOV_CW && IsLeftHead(head)) || (mov == MOV_CCW && IsRightHead(head)))
    {
        game.stats = AddMovUp(game.stats); // refresh stats
        game.snake = MoveHeadUp(game.snake, game.map);
    }
    else if((mov == MOV_CONT && IsBottomHead(head)) || (mov == MOV_CCW && IsLeftHead(head)) || (mov == MOV_CW && IsRightHead(head)))
    {
        game.stats = AddMovDown(game.stats); //..
        game.snake = MoveHeadDown(game.snake, game.map);
    }
    else if((mov == MOV_CONT && IsRightHead(head)) || (mov == MOV_CW && IsUpperHead(head)) || (mov == MOV_CCW && IsBottomHead(head)))
    {
        game.stats = AddMovRight(game.stats);
        game.snake = MoveHeadRight(game.snake, game.map);
    }
    else if((mov == MOV_CONT && IsLeftHead(head)) || (mov == MOV_CW && IsBottomHead(head)) || (mov == MOV_CCW && IsUpperHead(head)))
    {
        game.stats = AddMovLeft(game.stats);
        game.snake = MoveHeadLeft(game.snake, game.map);
    }

    return game;
}

tSnake MoveHeadUp (tSnake snake, tMap map)
{
    snake.headLine--;
    if(snake.headLine == - 1) // if the head surpasses the top of the map, it needs to go to the bottom line
    {
        snake.headLine = map.lines - 1;
    }

    if(CheckFoodMoney(map, snake) == 1) // if the snake eats food, the body grows
    {
        snake = AddBody(snake);
    }
    else // refresh the body 
        snake = RefreshBody(snake);

    snake.currHead = HEAD_UP;
    snake = UpdateHead(snake);

    return snake;
}

tSnake MoveHeadDown (tSnake snake, tMap map)
{
    snake.headLine++;
    if(snake.headLine == map.lines) // if the head surpasses the bottom of the map, it needs to go to the top line
    {
        snake.headLine = 0;
    }

    if(CheckFoodMoney(map, snake) == 1)
    {
        snake = AddBody(snake);
    }
    else
        snake = RefreshBody(snake);

    snake.currHead = HEAD_DOWN;
    snake = UpdateHead(snake);

    return snake;
}

tSnake MoveHeadRight (tSnake snake, tMap map)
{
    snake.headColumn++;
    if(snake.headColumn == map.columns) // if the head surpasses the extreme right of the map, it needs to go back to the extreme left
    {
        snake.headColumn = 0;
    }

    if(CheckFoodMoney(map, snake) == 1)
    {
        snake = AddBody(snake);
    }
    else
        snake = RefreshBody(snake); 

    snake.currHead = HEAD_RIGHT;
    snake = UpdateHead(snake);

    return snake;
}

tSnake MoveHeadLeft (tSnake snake, tMap map)
{
    snake.headColumn--;
    if(snake.headColumn == -1) // if the head surpasses the extreme left of the map, it needs to go back to the right extreme
    {
        snake.headColumn = map.columns - 1;
    }

    if(CheckFoodMoney(map, snake) == 1)
    {
        snake = AddBody(snake);
    }
    else
        snake = RefreshBody(snake);

    snake.currHead = HEAD_LEFT;
    snake = UpdateHead(snake);

    return snake;
}


tSnake RefreshBody (tSnake snake)
{
    int i;
    // when i = 0, is the furthest, when i = bodycount - 1 is the closest to the head
    for(i = 0; i < snake.bodyCount; i++) // iterate through the body
    {
        if(i == snake.bodyCount - 1) // if i is the position closest to the head, it takes the last head position
        {
            snake.bodyLine[i] = snake.lastHeadLine;
            snake.bodyColumn[i] = snake.lastHeadColumn;
        }
        else // else, it takes the next body position, closer to the head
        {
            snake.bodyLine[i] = snake.bodyLine[i + 1];
            snake.bodyColumn[i] = snake.bodyColumn[i + 1];
        }
    }

    return snake;
}

tStats CheckWhichScore(tStats stats, tSnake snake, tMap map)
{
    if(!CheckFoodMoney(map, snake))
    {
        stats.woPoints++;
    }
    else if (CheckFoodMoney(map, snake) == 1)
    {
        stats.score++;
        stats.foodCounter++;
    }
    else if (CheckFoodMoney(map, snake) == 2)
    {
        stats.score += 10;
        stats.moneyCounter++;
    }

    return stats;
}


tSnake UpdateHead(tSnake snake)
{
    snake.headPos[snake.headLine][snake.headColumn] = snake.currHead; 

    return snake;
}

tMap UpdateVoid(tSnake snake, tMap map)
{
    int i = 0;
    if(snake.bodyCount == 0) // when there's no body, takes the last head pos to print void
    {
        map.arr[snake.lastHeadLine][snake.lastHeadColumn] = VOID;
    }
    else // else, take the furthest body position and print the void
    {
        map.arr[snake.bodyLine[i]][snake.bodyColumn[i]] = VOID;
    }

    return map;
}


tSnake RefreshLastPos (tSnake snake)
{
    snake.lastHeadLine = snake.headLine;
    snake.lastHeadColumn = snake.headColumn;

    return snake;
}

tSnake AddBody (tSnake snake)
{
    // to add the body, we take the last head position after moving and create the body there
    snake.bodyLine[snake.bodyCount] = snake.lastHeadLine;
    snake.bodyColumn[snake.bodyCount] = snake.lastHeadColumn;

    snake.bodyCount++;

    return snake;
}

char GetCurrHead(tSnake snake)
{
    return snake.currHead;
}


int LoseCondition (tMap map, tSnake snake)
{
    if( IsWall(map, snake.headLine, snake.headColumn) || IsBody(map, snake.headLine, snake.headColumn)) // check if the head hit wall or body
    {
        return 1;
    }

    return 0;
}

int WinCondition(tStats stats, tMap map)
{
    if(stats.foodCounter == map.foodQuantity)
    {
        return 1;
    }
    
    return 0;

} 


tStats InitializeHeatmap(tStats stats, tMap map)
{
    int i, j;

    for(i = 0; i < map.lines; i++)
    {
        for(j = 0; map.columns; j++)
        {
            stats.heatmap[i][j] = 0;
        }
    }

    return stats;
}

tStats RefreshHeatMap(tStats stats, tSnake snake)
{
    stats.heatmap[snake.headLine][snake.headColumn]++;

    return stats;
}

void PrintHeatmap(FILE *fheatmap, tStats stats, tMap map)
{
    int i, j;
    for(i = 0; i < map.lines; i++)
    {
        for(j = 0; j < map.columns; j++)
        {
            if( j < map.columns - 1)
                fprintf(fheatmap, "%d ", stats.heatmap[i][j]);
            else if( j == map.columns - 1)
                fprintf(fheatmap, "%d", stats.heatmap[i][j]);
        }
        fprintf(fheatmap, "\n");
    }
    
}

void PrintResume(FILE *fresume, tStats stats, char mov, int type)
{
    if(type == 1)
    {
        fprintf(fresume, "Movimento %d (%c) fez a cobra crescer para o tamanho %d\n", stats.movements, mov, stats.foodCounter + 1);
    }
    else if (type == 2)
    {
        fprintf(fresume, "Movimento %d (%c) gerou dinheiro\n", stats.movements, mov);
    }
    else if (type == 3)
    {
        fprintf(fresume, "Movimento %d (%c) fez a cobra crescer para o tamanho %d, terminando o jogo\n", stats.movements, mov, stats.foodCounter + 1);
    }
    else if (type == 4)
    {
        fprintf(fresume, "Movimento %d (%c) resultou no fim de jogo por conta de colisao\n", stats.movements, mov);
    }

}

void PrintRanking(FILE *franking, tStats stats, tMap map)
{
    int i, j;
    int max = 0;
    // firstly, iterate through the heatmap to get the max score
    for(i = 0; i < map.lines; i++)
    {
        for(j = 0; j < map.columns; j++)
        {
            if(stats.heatmap[i][j] > max)
                max = stats.heatmap[i][j];
        }
    }
    
    // then, we iterate through the map again, but decreasing the max score and checking which positions is the same score, then printing
    while(max > 0)
    {
        for(i = 0; i < map.lines; i++)
        {
            for(j = 0; j < map.columns; j++)
            {
                if(stats.heatmap[i][j] == max)
                {
                    fprintf(franking, "(%d, %d) - %d\n", i, j, max);
                }
            }
        }
        max--; // goes to the next score, until max = 0
    }
}


tStats AddMov(tStats stats)
{
    stats.movements++;
    return stats;
}

tStats AddMovUp(tStats stats)
{
    stats.upMove++;
    return stats;
}

tStats AddMovDown(tStats stats)
{
    stats.downMove++;
    return stats;
}

tStats AddMovLeft(tStats stats)
{
    stats.leftMove++;
    return stats;
}

tStats AddMovRight(tStats stats)
{
    stats.rightMove++;
    return stats;
}

int IsWall(tMap map, int i, int j)
{
    return map.arr[i][j] == WALL;
}

int IsBody(tMap map, int i, int j)
{
    return map.arr[i][j] == BODY;
}

tStats SetWin(tStats stats)
{
    stats.win = 1;

    return stats;
}

tStats SetLose(tStats stats)
{
    stats.lose = 1;

    return stats;
}

tStats InitializeStatsVariables(tStats stats)
{
    stats.movements = 0;
    stats.downMove = 0;
    stats.leftMove = 0;
    stats.rightMove = 0;
    stats.upMove = 0;
    stats.woPoints = 0;
    stats.foodCounter = 0;
    stats.moneyCounter = 0;

    return stats;   
}
