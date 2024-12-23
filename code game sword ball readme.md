#include <SDL.h> 

#include <SDL_image.h> 

#include <SDL_ttf.h> 

#include <iostream> 

#include <sstream> 

 

const int SCREEN_WIDTH = 1200; 

const int SCREEN_HEIGHT = 900; 

 

// Các trạng thái của nhân vật 

enum CharacterState { IDLE, IDLE_LEFT, ATTACK, ATTACK_LEFT, MOVE_LEFT, MOVE_RIGHT }; 

enum GameState { START, IN_GAME, GAMEOVER_PLAYER1, GAMEOVER_PLAYER2 }; 

GameState currentState = START; 

 

class Game { 

public: 

    Game(); 

    ~Game(); 

    void run(); 

    void loadMedia(); 

    void processInput(); 

    void update(); 

    void render(); 

    void reset(); 

 

 

private: 

    SDL_Window* window; 

    SDL_Renderer* renderer; 

    SDL_Texture* backgroundTexture; 

    TTF_Font* scoreFont = nullptr;  // Font để hiển thị điểm 

    SDL_Texture* scoreTexture1 = nullptr;  // Texture hiển thị điểm Player 1 

    SDL_Texture* scoreTexture2 = nullptr;  // Texture hiển thị điểm Player 2 

    SDL_Color textColor = { 255, 255, 255 }; // Màu chữ trắng 

    // Player 1 textures 

    SDL_Texture* idleSpriteSheetTexture; 

    SDL_Texture* idleLeftSpriteSheetTexture; 

    SDL_Texture* attackSpriteSheetTexture; 

    SDL_Texture* attackLeftSpriteSheetTexture; 

    SDL_Texture* moveLeftSpriteSheetTexture; 

    SDL_Texture* moveRightSpriteSheetTexture; 

 

    // Player 2 textures 

    SDL_Texture* idleSpriteSheetTextureP2; 

    SDL_Texture* idleLeftSpriteSheetTextureP2; 

    SDL_Texture* attackSpriteSheetTextureP2; 

    SDL_Texture* attackLeftSpriteSheetTextureP2; 

    SDL_Texture* moveLeftSpriteSheetTextureP2; 

    SDL_Texture* moveRightSpriteSheetTextureP2; 

 

    SDL_Texture* startScreenTexture; 

 

    // Player 1 

    CharacterState characterState1; 

    int currentFrame1; 

    bool isAttacking1; 

    bool jKeyReleased; 

    int posX1, posY1; 

    int speed1; 

 

    // Player 2 

    CharacterState characterState2; 

    int currentFrame2; 

    bool isAttacking2; 

    bool fKeyReleased; 

    int posX2, posY2; 

    int speed2; 

 

    int animationFrameDelay; 

    int animationTime1; 

    int animationTime2; 

 

    SDL_Texture* ballTexture; 

    int ballPosX = SCREEN_WIDTH / 2, ballPosY = 730; 

    int ballSpeedX = 5, ballSpeedY = 0; 

    float ballSpeedIncrement = 1.2; // Tốc độ tăng lên mỗi lần chém trúng 

    bool ballMovingToPlayer1 = true; 

 

    // Biến quản lý Player 1 

    int gameOverCurrentFrame1 = 0; 

    Uint32 gameOverAnimationTime1 = 0; 

    Uint32 gameOverFrameDelay1 = 125; 

    SDL_Texture* gameOverSpriteSheet1; 

 

    // Biến quản lý Player 2 

    int gameOverCurrentFrame2 = 0; 

    Uint32 gameOverAnimationTime2 = 0; 

    Uint32 gameOverFrameDelay2 = 125; 

    SDL_Texture* gameOverSpriteSheet2; 

 

}; 

 

// Khai báo kích thước khung hình 

const int IDLE_FRAME_WIDTH = 160; 

const int IDLE_FRAME_HEIGHT = 160; 

const int ATTACK_FRAME_WIDTH = 320; 

const int ATTACK_FRAME_HEIGHT = 160; 

const int MOVE_FRAME_WIDTH = 160; 

const int MOVE_FRAME_HEIGHT = 160; 

const int IDLE_FRAME_COUNT = 10; 

const int ATTACK_FRAME_COUNT = 7; 

const int MOVE_FRAME_COUNT = 6; 

// Kích thước khung hình 

const int GAMEOVER_FRAME_WIDTH = 1200; // 100 pixel 

const int GAMEOVER_FRAME_HEIGHT = 900;     // 900 pixel 

const int GAMEOVER_FRAME_COUNT = 12;       // 12 khung hình 

int scorePlayer1 = 0; // Điểm của Player 1 

int scorePlayer2 = 0; // Điểm của Player 2 

 

 

 

 

 

Game::Game() : window(nullptr), renderer(nullptr), backgroundTexture(nullptr), 

startScreenTexture(nullptr), gameOverSpriteSheet1(nullptr), gameOverSpriteSheet2(nullptr), 

idleSpriteSheetTexture(nullptr), idleLeftSpriteSheetTexture(nullptr), 

attackSpriteSheetTexture(nullptr), attackLeftSpriteSheetTexture(nullptr), 

moveLeftSpriteSheetTexture(nullptr), moveRightSpriteSheetTexture(nullptr), 

idleSpriteSheetTextureP2(nullptr), idleLeftSpriteSheetTextureP2(nullptr), 

attackSpriteSheetTextureP2(nullptr), attackLeftSpriteSheetTextureP2(nullptr), 

moveLeftSpriteSheetTextureP2(nullptr), moveRightSpriteSheetTextureP2(nullptr), 

characterState1(IDLE), characterState2(IDLE), 

currentFrame1(0), currentFrame2(0), 

animationFrameDelay(100), animationTime1(0), animationTime2(0), 

isAttacking1(false), isAttacking2(false), 

jKeyReleased(true), fKeyReleased(true), 

posX1(100), posY1(650), posX2(900), posY2(650), 

speed1(10), speed2(10) { 

    if (SDL_Init(SDL_INIT_VIDEO) < 0) { 

        std::cerr << "SDL could not initialize! SDL_Error: " << SDL_GetError() << std::endl; 

        exit(1); 

    } 

    // Khởi tạo thư viện TTF 

    if (TTF_Init() == -1) { 

        std::cerr << "SDL_ttf could not initialize! SDL_ttf Error: " << TTF_GetError() << std::endl; 

        exit(1); 

    } 

 

 

    window = SDL_CreateWindow("SwordBall", SDL_WINDOWPOS_CENTERED, SDL_WINDOWPOS_CENTERED, SCREEN_WIDTH, SCREEN_HEIGHT, SDL_WINDOW_SHOWN); 

    renderer = SDL_CreateRenderer(window, -1, SDL_RENDERER_ACCELERATED); 

} 

 

Game::~Game() { 

    SDL_DestroyTexture(backgroundTexture); 

    SDL_DestroyTexture(startScreenTexture); 

    SDL_DestroyTexture(gameOverSpriteSheet1); 

    SDL_DestroyTexture(gameOverSpriteSheet2); 

      

    SDL_DestroyTexture(idleSpriteSheetTexture); 

    SDL_DestroyTexture(idleLeftSpriteSheetTexture); 

    SDL_DestroyTexture(attackSpriteSheetTexture); 

    SDL_DestroyTexture(attackLeftSpriteSheetTexture); 

    SDL_DestroyTexture(moveLeftSpriteSheetTexture); 

    SDL_DestroyTexture(moveRightSpriteSheetTexture); 

 

    SDL_DestroyTexture(idleSpriteSheetTextureP2); 

    SDL_DestroyTexture(idleLeftSpriteSheetTextureP2); 

    SDL_DestroyTexture(attackSpriteSheetTextureP2); 

    SDL_DestroyTexture(attackLeftSpriteSheetTextureP2); 

    SDL_DestroyTexture(moveLeftSpriteSheetTextureP2); 

    SDL_DestroyTexture(moveRightSpriteSheetTextureP2); 

 

    SDL_DestroyTexture(ballTexture); 

 

    SDL_DestroyTexture(scoreTexture1); 

    SDL_DestroyTexture(scoreTexture2); 

    TTF_CloseFont(scoreFont);  // Đóng font 

    // Dọn dẹp thư viện TTF 

    TTF_Quit(); 

    SDL_DestroyRenderer(renderer); 

    SDL_DestroyWindow(window); 

    SDL_Quit(); 

} 

 

void Game::loadMedia() { 

    SDL_Surface* tempSurface = IMG_Load("assets/background.png"); 

    backgroundTexture = SDL_CreateTextureFromSurface(renderer, tempSurface); 

    SDL_FreeSurface(tempSurface); 

    // Tải font 

    scoreFont = TTF_OpenFont("assets/phongchu.ttf", 24);  

    if (!scoreFont) { 

        std::cerr << "Không thể tải font! Lỗi TTF: " << TTF_GetError() << std::endl; 

        exit(1); 

    } 

    // Player 1 Textures 

    idleSpriteSheetTexture = IMG_LoadTexture(renderer, "assets/Player1_dungphai.png"); 

    idleLeftSpriteSheetTexture = IMG_LoadTexture(renderer, "assets/Player1_dungtrai.png"); 

    attackSpriteSheetTexture = IMG_LoadTexture(renderer, "assets/Player1_chemphai_320x160x7.png"); 

    attackLeftSpriteSheetTexture = IMG_LoadTexture(renderer, "assets/Player1_chemtrai_320x160x7.png"); 

    moveLeftSpriteSheetTexture = IMG_LoadTexture(renderer, "assets/Player1_chaytrai.png"); 

    moveRightSpriteSheetTexture = IMG_LoadTexture(renderer, "assets/Player1_chayphai.png"); 

 

    // Player 2 Textures 

    idleSpriteSheetTextureP2 = IMG_LoadTexture(renderer, "assets/Player2_dungphai.png"); 

    idleLeftSpriteSheetTextureP2 = IMG_LoadTexture(renderer, "assets/Player2_dungtrai.png"); 

    attackSpriteSheetTextureP2 = IMG_LoadTexture(renderer, "assets/Player2_chemphai_320x160x7.png"); 

    attackLeftSpriteSheetTextureP2 = IMG_LoadTexture(renderer, "assets/Player2_chemtrai_320x160x7.png"); 

    moveLeftSpriteSheetTextureP2 = IMG_LoadTexture(renderer, "assets/Player2_chaytrai.png"); 

    moveRightSpriteSheetTextureP2 = IMG_LoadTexture(renderer, "assets/Player2_chayphai.png"); 

 

    ballTexture = IMG_LoadTexture(renderer, "assets/ball.png"); 

    if (!ballTexture) { 

        std::cerr << "Không thể tải texture của bóng! Lỗi SDL: " << SDL_GetError() << std::endl; 

    } 

    startScreenTexture = IMG_LoadTexture(renderer, "assets/start_screen.png"); 

    gameOverSpriteSheet1 = IMG_LoadTexture(renderer, "assets/path_to_gameover_player1_spritesheet.png"); 

    gameOverSpriteSheet2 = IMG_LoadTexture(renderer, "assets/path_to_gameover_player2_spritesheet.png"); 

 

} 

 

void Game::run() { 

    loadMedia(); 

    bool isRunning = true; 

 

    while (isRunning) { 

        processInput(); 

        update(); 

        render(); 

        SDL_Delay(16); 

    } 

} 

 

void Game::processInput() { 

    SDL_Event event; 

    while (SDL_PollEvent(&event)) { 

        if (event.type == SDL_QUIT) { 

            exit(0); 

        } 

 

        // Xử lý trong màn hình bắt đầu 

        if (currentState == START) { 

            if (event.type == SDL_KEYDOWN && event.key.keysym.sym == SDLK_SPACE) { 

                currentState = IN_GAME; // Nhấn SPACE để vào game 

            } 

        } 

        // Xử lý trong màn hình game over 

        else if (currentState == GAMEOVER_PLAYER1 || currentState == GAMEOVER_PLAYER2) { 

            if (event.type == SDL_KEYDOWN && event.key.keysym.sym == SDLK_r) { 

                reset(); // Nhấn R để reset game 

                currentState = START; 

            } 

        } 

        // Xử lý trong game 

        else if (currentState == IN_GAME) { 

            if (event.type == SDL_KEYDOWN) { 

                // Player 1 Controls 

                if (event.key.keysym.sym == SDLK_s && jKeyReleased && !isAttacking1) { 

                    // Kiểm tra xem nhân vật có đang tấn công hay không 

                    isAttacking1 = true; 

                    jKeyReleased = false; 

                    // Đổi trạng thái tấn công, kiểm tra hướng tấn công 

                    characterState1 = (characterState1 == MOVE_LEFT || characterState1 == IDLE_LEFT) ? ATTACK_LEFT : ATTACK; 

                    currentFrame1 = 0; 

                    animationTime1 = 0; 

                } 

                else if (event.key.keysym.sym == SDLK_a) { 

                    characterState1 = MOVE_LEFT; 

                } 

                else if (event.key.keysym.sym == SDLK_d) { 

                    characterState1 = MOVE_RIGHT; 

                } 

 

                // Player 2 Controls 

                if (event.key.keysym.sym == SDLK_DOWN && fKeyReleased && !isAttacking2) { 

                    // Kiểm tra xem nhân vật có đang tấn công hay không 

                    isAttacking2 = true; 

                    fKeyReleased = false; 

                    // Đổi trạng thái tấn công, kiểm tra hướng tấn công 

                    characterState2 = (characterState2 == MOVE_LEFT || characterState2 == IDLE_LEFT) ? ATTACK_LEFT : ATTACK; 

                    currentFrame2 = 0; 

                    animationTime2 = 0; 

                } 

                else if (event.key.keysym.sym == SDLK_LEFT) { 

                    characterState2 = MOVE_LEFT; 

                } 

                else if (event.key.keysym.sym == SDLK_RIGHT) { 

                    characterState2 = MOVE_RIGHT; 

                } 

            } 

            else if (event.type == SDL_KEYUP) { 

                // Player 1 

                if (event.key.keysym.sym == SDLK_s) jKeyReleased = true; 

                if (event.key.keysym.sym == SDLK_a) characterState1 = IDLE_LEFT; 

                if (event.key.keysym.sym == SDLK_d) characterState1 = IDLE; 

 

                // Player 2 

                if (event.key.keysym.sym == SDLK_DOWN) fKeyReleased = true; 

                if (event.key.keysym.sym == SDLK_LEFT) characterState2 = IDLE_LEFT; 

                if (event.key.keysym.sym == SDLK_RIGHT) characterState2 = IDLE; 

            } 

        } 

    } 

} 

 

 

 

void Game::update() { 

    if (currentState == GAMEOVER_PLAYER1) { 

        gameOverAnimationTime1 += 35; // Tăng thời gian 

 

        if (gameOverAnimationTime1 >= gameOverFrameDelay1) { 

            gameOverCurrentFrame1 = (gameOverCurrentFrame1 + 1) % GAMEOVER_FRAME_COUNT; 

            gameOverAnimationTime1 = 0; 

        } 

    } 

    else if (currentState == GAMEOVER_PLAYER2) { 

        gameOverAnimationTime2 += 35; // Tăng thời gian 

 

        if (gameOverAnimationTime2 >= gameOverFrameDelay2) { 

            gameOverCurrentFrame2 = (gameOverCurrentFrame2 + 1) % GAMEOVER_FRAME_COUNT; 

            gameOverAnimationTime2 = 0; 

        } 

    } 

 

    // Đặt lại trạng thái chém sau mỗi vòng lặp trong trò chơi 

   

 

    if (currentState == IN_GAME) { // Player 1 Animation 

        animationTime1 += 35; 

 

        if (isAttacking1) { 

            if (animationTime1 >= animationFrameDelay) { 

                currentFrame1++; 

                animationTime1 = 0; 

                if (currentFrame1 >= ATTACK_FRAME_COUNT) { 

                    currentFrame1 = 0; 

                    isAttacking1 = false; 

                    characterState1 = (characterState1 == ATTACK_LEFT) ? IDLE_LEFT : IDLE; 

                } 

            } 

        } 

        else if (characterState1 == MOVE_LEFT) { 

            posX1 -= speed1; 

            if (posX1 < 0) posX1 = 0; 

 

            if (animationTime1 >= animationFrameDelay) { 

                currentFrame1++; 

                animationTime1 = 0; 

                if (currentFrame1 >= MOVE_FRAME_COUNT) { 

                    currentFrame1 = 0; 

                } 

            } 

        } 

        else if (characterState1 == MOVE_RIGHT) { 

            posX1 += speed1; 

            if (posX1 > SCREEN_WIDTH - MOVE_FRAME_WIDTH) posX1 = SCREEN_WIDTH - MOVE_FRAME_WIDTH; 

 

            if (animationTime1 >= animationFrameDelay) { 

                currentFrame1++; 

                animationTime1 = 0; 

                if (currentFrame1 >= MOVE_FRAME_COUNT) { 

                    currentFrame1 = 0; 

                } 

            } 

        } 

        else { 

            if (animationTime1 >= animationFrameDelay) { 

                currentFrame1++; 

                animationTime1 = 0; 

                int frameCount = (characterState1 == IDLE_LEFT) ? IDLE_FRAME_COUNT : IDLE_FRAME_COUNT; 

                if (currentFrame1 >= frameCount) { 

                    currentFrame1 = 0; 

                } 

            } 

        } 

 

        // Player 2 Animation 

        animationTime2 += 35; 

 

        if (isAttacking2) { 

            if (animationTime2 >= animationFrameDelay) { 

                currentFrame2++; 

                animationTime2 = 0; 

                if (currentFrame2 >= ATTACK_FRAME_COUNT) { 

                    currentFrame2 = 0; 

                    isAttacking2 = false; 

                    characterState2 = (characterState2 == ATTACK_LEFT) ? IDLE_LEFT : IDLE; 

                } 

            } 

        } 

        else if (characterState2 == MOVE_LEFT) { 

            posX2 -= speed2; 

            if (posX2 < 0) posX2 = 0; 

 

            if (animationTime2 >= animationFrameDelay) { 

                currentFrame2++; 

                animationTime2 = 0; 

                if (currentFrame2 >= MOVE_FRAME_COUNT) { 

                    currentFrame2 = 0; 

                } 

            } 

        } 

        else if (characterState2 == MOVE_RIGHT) { 

            posX2 += speed2; 

            if (posX2 > SCREEN_WIDTH - MOVE_FRAME_WIDTH) posX2 = SCREEN_WIDTH - MOVE_FRAME_WIDTH; 

 

            if (animationTime2 >= animationFrameDelay) { 

                currentFrame2++; 

                animationTime2 = 0; 

                if (currentFrame2 >= MOVE_FRAME_COUNT) { 

                    currentFrame2 = 0; 

                } 

            } 

        } 

        else { 

            if (animationTime2 >= animationFrameDelay) { 

                currentFrame2++; 

                animationTime2 = 0; 

                int frameCount = (characterState2 == IDLE_LEFT) ? IDLE_FRAME_COUNT : IDLE_FRAME_COUNT; 

                if (currentFrame2 >= frameCount) { 

                    currentFrame2 = 0; 

                } 

            } 

        } 

 

 

 

        // Kiểm tra va chạm với cạnh trên và dưới (trục Y) 

        if (ballPosY <= 0) { 

            ballPosY = 0;  // Đặt bóng tại cạnh trên 

            ballSpeedY = -ballSpeedY;  // Đảo chiều vận tốc theo trục Y 

        } 

        else if (ballPosY >= SCREEN_HEIGHT - 64) { // 64 là kích thước bóng 

            ballPosY = SCREEN_HEIGHT - 64;  // Đặt bóng tại cạnh dưới 

            ballSpeedY = -ballSpeedY;  // Đảo chiều vận tốc theo trục Y 

        } 

 

        // Cho bóng đi xuyên qua các cạnh bên trái và phải 

        if (ballPosX <= 0) { 

            ballPosX = SCREEN_WIDTH - 64;  // Đặt bóng ở cạnh phải khi bóng ra ngoài cạnh trái 

        } 

        else if (ballPosX >= SCREEN_WIDTH - 64) { 

            ballPosX = 0;  // Đặt bóng ở cạnh trái khi bóng ra ngoài cạnh phải 

        } 

        // Giới hạn tốc độ bóng 

        if (abs(ballSpeedX) > 30) { 

            ballSpeedX = (ballSpeedX > 0) ? 30 : -30; // Giới hạn tốc độ tối đa là 10 

        } 

 

        if (abs(ballSpeedY) > 30) { 

            ballSpeedY = (ballSpeedY > 0) ? 30 : -30; // Giới hạn tốc độ tối đa là 10 

        } 

 

        // Cập nhật vị trí bóng 

        ballPosX += ballSpeedX; 

        ballPosY += ballSpeedY; 

 

 

 

 

 

 

        // Khởi tạo SDL_Rect cho quả bóng 

        SDL_Rect ballRect = { ballPosX, ballPosY, 64, 64 }; 

 

        // Khởi tạo SDL_Rect cho người chơi 1 

        SDL_Rect player1Rect = { posX1, posY1, MOVE_FRAME_WIDTH, MOVE_FRAME_HEIGHT }; 

 

        // Khởi tạo SDL_Rect cho người chơi 2 

        SDL_Rect player2Rect = { posX2, posY2, MOVE_FRAME_WIDTH, MOVE_FRAME_HEIGHT }; 

 

         

        // Vùng chém mở rộng cho Player 1 

        SDL_Rect attackRange1 = { posX1 - 50, posY1, 160 + 100, MOVE_FRAME_HEIGHT }; // Tăng chiều rộng vùng chém 

        if (SDL_HasIntersection(&ballRect, &attackRange1)) { 

            if (isAttacking1) { 

                ballMovingToPlayer1 = false; // Đổi hướng bóng về phía Player 2 

                ballSpeedX = abs(ballSpeedX) * ballSpeedIncrement; // Đảm bảo bóng bay về phía Player 2 và tăng tốc độ 

                ballSpeedY = -abs(ballSpeedX); // Đặt bóng bay lên với góc 45 độ (y âm để bay lên) 

               

            } 

        } 

 

        // Vùng chém mở rộng cho Player 2 

        SDL_Rect attackRange2 = { posX2 + 150, posY2, 100, MOVE_FRAME_HEIGHT }; // Tăng chiều rộng vùng chém 

        if (SDL_HasIntersection(&ballRect, &attackRange2)) { 

            if (isAttacking2) { 

                ballMovingToPlayer1 = true; // Đổi hướng bóng về phía Player 1 

                ballSpeedX = -abs(ballSpeedX) * ballSpeedIncrement; // Đảm bảo bóng bay về phía Player 1 và tăng tốc độ 

                ballSpeedY = -abs(ballSpeedX); // Đặt bóng bay lên với góc 45 độ (y âm để bay lên) 

              

            } 

        } 

 

 

 

        // Tăng phạm vi chạm của Player 1 

        SDL_Rect player1CollisionRange = { posX1 , posY1, MOVE_FRAME_WIDTH - 50, MOVE_FRAME_HEIGHT }; 

        if (SDL_HasIntersection(&ballRect, &player1CollisionRange)) { 

            if (isAttacking1) { 

                ballMovingToPlayer1 = false; // Đổi hướng bóng 

                ballSpeedX *= ballSpeedIncrement; // Tăng tốc độ bóng 

                 

            } 

            else { 

                // Kết thúc trò chơi khi bóng chạm vào Player 1 

                scorePlayer2++; 

                currentState = GAMEOVER_PLAYER1; 

            } 

        } 

        // Tăng phạm vi chạm của Player 2 

        SDL_Rect player2CollisionRange = { posX2  , posY2 , MOVE_FRAME_WIDTH + 100 , MOVE_FRAME_HEIGHT - 20 }; 

        if (SDL_HasIntersection(&ballRect, &player2CollisionRange)) { 

            if (isAttacking2) { 

                ballMovingToPlayer1 = true; // Đổi hướng bóng về Player 1 

                ballSpeedX *= ballSpeedIncrement; // Tăng tốc độ bóng 

                

            } 

            else { 

                // Kết thúc trò chơi khi bóng chạm vào Player2 

                scorePlayer1++; 

                currentState = GAMEOVER_PLAYER2; 

            } 

        } 

        // Điều chỉnh hướng bóng theo người chơi 

        ballSpeedX = ballMovingToPlayer1 ? -abs(ballSpeedX) : abs(ballSpeedX); 

 

    } 

} 

 

void Game::render() { 

    SDL_RenderClear(renderer); 

    if (currentState == START) { 

        // Vẽ màn hình START 

        SDL_RenderCopy(renderer, startScreenTexture, nullptr, nullptr); // startTexture là hình nền cho màn hình start 

    } 

    else if (currentState == GAMEOVER_PLAYER1) { 

        SDL_Rect srcRect1 = { 

            gameOverCurrentFrame1 * GAMEOVER_FRAME_WIDTH, // Tọa độ x 

            0, // Tọa độ y 

            GAMEOVER_FRAME_WIDTH, 

            GAMEOVER_FRAME_HEIGHT 

        }; 

        SDL_Rect destRect1 = { 

            (SCREEN_WIDTH - GAMEOVER_FRAME_WIDTH) / 2, // Canh giữa màn hình 

            (SCREEN_HEIGHT - GAMEOVER_FRAME_HEIGHT) / 2, 

            GAMEOVER_FRAME_WIDTH, 

            GAMEOVER_FRAME_HEIGHT 

        }; 

 

        SDL_RenderCopy(renderer, gameOverSpriteSheet1, &srcRect1, &destRect1); 

    } 

    else if (currentState == GAMEOVER_PLAYER2) { 

        SDL_Rect srcRect2 = { 

            gameOverCurrentFrame2 * GAMEOVER_FRAME_WIDTH, // Tọa độ x 

            0, // Tọa độ y 

            GAMEOVER_FRAME_WIDTH, 

            GAMEOVER_FRAME_HEIGHT 

        }; 

        SDL_Rect destRect2 = { 

            (SCREEN_WIDTH - GAMEOVER_FRAME_WIDTH) / 2, // Canh giữa màn hình 

            (SCREEN_HEIGHT - GAMEOVER_FRAME_HEIGHT) / 2, 

            GAMEOVER_FRAME_WIDTH, 

            GAMEOVER_FRAME_HEIGHT 

        }; 

 

        SDL_RenderCopy(renderer, gameOverSpriteSheet2, &srcRect2, &destRect2); 

    } 

    else if (currentState == IN_GAME) { 

 

        SDL_RenderCopy(renderer, backgroundTexture, nullptr, nullptr); 

 

        // Render Player 1 

        SDL_Rect srcRect1; 

        SDL_Rect destRect1 = { posX1, posY1, MOVE_FRAME_WIDTH, MOVE_FRAME_HEIGHT }; 

 

        if (characterState1 == IDLE) { 

            srcRect1 = { currentFrame1 * IDLE_FRAME_WIDTH, 0, IDLE_FRAME_WIDTH, IDLE_FRAME_HEIGHT }; 

            SDL_RenderCopy(renderer, idleSpriteSheetTexture, &srcRect1, &destRect1); 

        } 

        else if (characterState1 == IDLE_LEFT) { 

            srcRect1 = { currentFrame1 * IDLE_FRAME_WIDTH, 0, IDLE_FRAME_WIDTH, IDLE_FRAME_HEIGHT }; 

            SDL_RenderCopy(renderer, idleLeftSpriteSheetTexture, &srcRect1, &destRect1); 

        } 

        else if (characterState1 == ATTACK) { 

            srcRect1 = { currentFrame1 * ATTACK_FRAME_WIDTH, 0, ATTACK_FRAME_WIDTH, ATTACK_FRAME_HEIGHT }; 

            destRect1.w = ATTACK_FRAME_WIDTH; 

            SDL_RenderCopy(renderer, attackSpriteSheetTexture, &srcRect1, &destRect1); 

        } 

        else if (characterState1 == ATTACK_LEFT) { 

            srcRect1 = { currentFrame1 * ATTACK_FRAME_WIDTH, 0, ATTACK_FRAME_WIDTH, ATTACK_FRAME_HEIGHT }; 

            destRect1.w = ATTACK_FRAME_WIDTH; 

            SDL_RenderCopy(renderer, attackLeftSpriteSheetTexture, &srcRect1, &destRect1); 

        } 

        else if (characterState1 == MOVE_LEFT) { 

            srcRect1 = { currentFrame1 * MOVE_FRAME_WIDTH, 0, MOVE_FRAME_WIDTH, MOVE_FRAME_HEIGHT }; 

            SDL_RenderCopy(renderer, moveLeftSpriteSheetTexture, &srcRect1, &destRect1); 

        } 

        else if (characterState1 == MOVE_RIGHT) { 

            srcRect1 = { currentFrame1 * MOVE_FRAME_WIDTH, 0, MOVE_FRAME_WIDTH, MOVE_FRAME_HEIGHT }; 

            SDL_RenderCopy(renderer, moveRightSpriteSheetTexture, &srcRect1, &destRect1); 

        } 

 

        // Render Player 2 

        SDL_Rect srcRect2; 

        SDL_Rect destRect2 = { posX2, posY2, MOVE_FRAME_WIDTH, MOVE_FRAME_HEIGHT }; 

 

        if (characterState2 == IDLE) { 

            srcRect2 = { currentFrame2 * IDLE_FRAME_WIDTH, 0, IDLE_FRAME_WIDTH, IDLE_FRAME_HEIGHT }; 

            SDL_RenderCopy(renderer, idleSpriteSheetTextureP2, &srcRect2, &destRect2); 

        } 

        else if (characterState2 == IDLE_LEFT) { 

            srcRect2 = { currentFrame2 * IDLE_FRAME_WIDTH, 0, IDLE_FRAME_WIDTH, IDLE_FRAME_HEIGHT }; 

            SDL_RenderCopy(renderer, idleLeftSpriteSheetTextureP2, &srcRect2, &destRect2); 

        } 

        else if (characterState2 == ATTACK) { 

            srcRect2 = { currentFrame2 * ATTACK_FRAME_WIDTH, 0, ATTACK_FRAME_WIDTH, ATTACK_FRAME_HEIGHT }; 

            destRect2.w = ATTACK_FRAME_WIDTH; 

            SDL_RenderCopy(renderer, attackSpriteSheetTextureP2, &srcRect2, &destRect2); 

        } 

        else if (characterState2 == ATTACK_LEFT) { 

            srcRect2 = { currentFrame2 * ATTACK_FRAME_WIDTH, 0, ATTACK_FRAME_WIDTH, ATTACK_FRAME_HEIGHT }; 

            destRect2.w = ATTACK_FRAME_WIDTH; 

            SDL_RenderCopy(renderer, attackLeftSpriteSheetTextureP2, &srcRect2, &destRect2); 

        } 

        else if (characterState2 == MOVE_LEFT) { 

            srcRect2 = { currentFrame2 * MOVE_FRAME_WIDTH, 0, MOVE_FRAME_WIDTH, MOVE_FRAME_HEIGHT }; 

            SDL_RenderCopy(renderer, moveLeftSpriteSheetTextureP2, &srcRect2, &destRect2); 

        } 

        else if (characterState2 == MOVE_RIGHT) { 

            srcRect2 = { currentFrame2 * MOVE_FRAME_WIDTH, 0, MOVE_FRAME_WIDTH, MOVE_FRAME_HEIGHT }; 

            SDL_RenderCopy(renderer, moveRightSpriteSheetTextureP2, &srcRect2, &destRect2); 

        } 

        // Vẽ điểm cho Player 1 

        std::ostringstream scoreStream1; 

        scoreStream1 << scorePlayer1; 

        SDL_Surface* scoreSurface1 = TTF_RenderText_Solid(scoreFont, scoreStream1.str().c_str(), textColor); 

        scoreTexture1 = SDL_CreateTextureFromSurface(renderer, scoreSurface1); 

        SDL_FreeSurface(scoreSurface1); 

        SDL_Rect scoreRect1 = { 50, 20, 50, 50 }; // Vị trí hiển thị điểm Player 1 

        SDL_RenderCopy(renderer, scoreTexture1, nullptr, &scoreRect1); 

 

        // Vẽ điểm cho Player 2 

        std::ostringstream scoreStream2; 

        scoreStream2 << scorePlayer2; 

        SDL_Surface* scoreSurface2 = TTF_RenderText_Solid(scoreFont, scoreStream2.str().c_str(), textColor); 

        scoreTexture2 = SDL_CreateTextureFromSurface(renderer, scoreSurface2); 

        SDL_FreeSurface(scoreSurface2); 

        SDL_Rect scoreRect2 = { SCREEN_WIDTH - 100, 20, 50, 50 }; // Vị trí hiển thị điểm Player 2 

        SDL_RenderCopy(renderer, scoreTexture2, nullptr, &scoreRect2); 

        SDL_Rect ballRect = { ballPosX, ballPosY, 20, 20 }; // Kích thước bóng 

        SDL_RenderCopy(renderer, ballTexture, nullptr, &ballRect); 

    } 

    SDL_RenderPresent(renderer); 

} 

void Game::reset() { 

    // Đặt lại vị trí và trạng thái của các nhân vật 

    posX1 = 100; posY1 = 650; 

    posX2 = 900; posY2 = 650; 

    characterState1 = IDLE; 

    characterState2 = IDLE; 

 

    // Đặt lại bóng 

    ballPosX = SCREEN_WIDTH / 2; 

    ballPosY = 730; 

    ballSpeedX = 5; 

    ballSpeedY = 0; 

    ballMovingToPlayer1 = true; 

 

    // Đặt lại các biến khác (nếu cần) 

    currentFrame1 = 0; 

    currentFrame2 = 0; 

    isAttacking1 = false; 

    isAttacking2 = false; 

} 

 

 

int main(int argc, char* argv[]) { 

    Game game; 

    game.run(); 

    return 0; 

} 
