# Tennis-game
테니스 게임 과제
### 20210815 백대현 과제

https://github.com/100DH/Tennis-game/assets/93199016/b69874c0-2f3c-4943-a0d9-4fcc34bba03c

## 추가한 기능
1. 배경 음악 추가 - 게임에 배경음악을 추가하여 플레이가 지루하지 않도록 만들어 보았습니다.
2. 중앙 상단에 점수 표시 - AI와 플레이어의 점수를 표기하여 누가 이기고 있는지 한눈에 알아볼 수 있도록 만들었습니다.
3. 가짜 공 생성 - 공이 패들에 부딪힐 경우 80% 확률로 랜덤한 방향으로 0.5초간 날아가다가 사라지는 공 2개를 생성하여 플레이 난이도를 상승시켰습니다.
4. 공 속도 증가 - 공이 패들에 부딪힐 때 80% 확률로 공이 붉게 변하며 속도가 2배 빨라지도록 만들어 게임 플레이에 변수를 생성했습니다.
5. 속도 발판 - 게임 화면 가운데에 초록색 부분을 공이 지나가면 0.5초간 속도가 증가하도록 만들었습니다.

## 사용방법
1. visual studio 2022 버전을 실행한 뒤, 콘솔 앱 프로젝트를 생성합니다.
2. 콘솔 앱을 생성한 후, 소스파일에 github에 적혀있는 코드를 복사하여 붙여넣습니다.
3. 실행하기 전, 프로젝트 -> NuGet 패키지 관리 -> sfml 검색 후 SFML_VS2019를 설치합니다.
4. github에 제공된 resources 파일을 압축 해제하여 새로 만든 프로젝트의 cpp 파일이 있는 곳에 붙여넣은 뒤, 프로그램을 실행합니다.

## 과제 코드 
        ////////////////////////////////////////////////////////////
        // Headers
        ////////////////////////////////////////////////////////////
        #include <SFML/Graphics.hpp>
        #include <SFML/Audio.hpp>
        #include <cmath>
        #include <ctime>
        #include <cstdlib>
        #include <vector>
        
        #ifdef SFML_SYSTEM_IOS
        #include <SFML/Main.hpp>
        #endif
        
        std::string resourcesDir()
        {
        #ifdef SFML_SYSTEM_IOS
            return "";
        #else
            return "resources/";
        #endif
        }
        
        struct FakeBall
        {
            sf::CircleShape shape;
            float angle;
            sf::Clock timer;
        };
        
        ////////////////////////////////////////////////////////////
        /// Entry point of application
        ///
        /// \return Application exit code
        ///
        ////////////////////////////////////////////////////////////
        int main()
        {
            std::srand(static_cast<unsigned int>(std::time(NULL)));
        
            // Define some constants
            const float pi = 3.14159f;
            const float gameWidth = 800;
            const float gameHeight = 600;
            sf::Vector2f paddleSize(25, 100);
            float ballRadius = 10.f;
            const float initialBallSpeed = 400.f;
            float ballSpeed = initialBallSpeed;
            const float speedBoost = 1.5f;
        
            // Create the window of the application
            sf::RenderWindow window(sf::VideoMode(static_cast<unsigned int>(gameWidth), static_cast<unsigned int>(gameHeight), 32), "SFML Tennis",
                sf::Style::Titlebar | sf::Style::Close);
            window.setVerticalSyncEnabled(true);
        
            // Load the sounds used in the game
            sf::SoundBuffer ballSoundBuffer;
            if (!ballSoundBuffer.loadFromFile(resourcesDir() + "ball.wav"))
                return EXIT_FAILURE;
            sf::Sound ballSound(ballSoundBuffer);
        
            // Load the background music
            sf::Music backgroundMusic;
            if (!backgroundMusic.openFromFile(resourcesDir() + "clap.wav"))
                return EXIT_FAILURE;
            backgroundMusic.setLoop(true);
            backgroundMusic.play();
        
            // Create the SFML logo texture:
            sf::Texture sfmlLogoTexture;
            if (!sfmlLogoTexture.loadFromFile(resourcesDir() + "sfml_logo.png"))
                return EXIT_FAILURE;
            sf::Sprite sfmlLogo;
            sfmlLogo.setTexture(sfmlLogoTexture);
            sfmlLogo.setPosition(170, 50);
        
            // Create the left paddle
            sf::RectangleShape leftPaddle;
            leftPaddle.setSize(paddleSize - sf::Vector2f(3, 3));
            leftPaddle.setOutlineThickness(3);
            leftPaddle.setOutlineColor(sf::Color::Black);
            leftPaddle.setFillColor(sf::Color(100, 100, 200));
            leftPaddle.setOrigin(paddleSize / 2.f);
        
            // Create the right paddle
            sf::RectangleShape rightPaddle;
            rightPaddle.setSize(paddleSize - sf::Vector2f(3, 3));
            rightPaddle.setOutlineThickness(3);
            rightPaddle.setOutlineColor(sf::Color::Black);
            rightPaddle.setFillColor(sf::Color(200, 100, 100));
            rightPaddle.setOrigin(paddleSize / 2.f);
        
            // Create the ball
            sf::CircleShape ball;
            ball.setRadius(ballRadius - 3);
            ball.setOutlineThickness(2);
            ball.setOutlineColor(sf::Color::Black);
            ball.setFillColor(sf::Color::White);
            ball.setOrigin(ballRadius / 2, ballRadius / 2);
        
            // Load the text font
            sf::Font font;
            if (!font.loadFromFile(resourcesDir() + "tuffy.ttf"))
                return EXIT_FAILURE;
        
            // Initialize the pause message
            sf::Text pauseMessage;
            pauseMessage.setFont(font);
            pauseMessage.setCharacterSize(40);
            pauseMessage.setPosition(170.f, 200.f);
            pauseMessage.setFillColor(sf::Color::White);
        
        #ifdef SFML_SYSTEM_IOS
            pauseMessage.setString("Welcome to SFML Tennis!\nTouch the screen to start the game.");
        #else
            pauseMessage.setString("Welcome to SFML Tennis!\n\nPress space to start the game.");
        #endif
        
            // Initialize the score text
            sf::Text scoreText;
            scoreText.setFont(font);
            scoreText.setCharacterSize(30);
            scoreText.setPosition(gameWidth / 2.f, 10.f);
            scoreText.setFillColor(sf::Color::White);
        
            int playerScore = 0;
            int aiScore = 0;
        
            // Define the paddles properties
            sf::Clock AITimer;
            const sf::Time AITime = sf::seconds(0.1f);
            const float paddleSpeed = 400.f;
            float rightPaddleSpeed = 0.f;
            float ballAngle = 0.f; // to be changed later
        
            sf::Clock clock;
            sf::Clock effectClock; // Timer for the temporary effect
            bool effectActive = false; // Flag for the temporary effect
            bool isPlaying = false;
        
            std::vector<FakeBall> fakeBalls; // Vector to hold fake balls
        
            // Define the central speed boost zone
            sf::RectangleShape speedBoostZone(sf::Vector2f(200.f, gameHeight));
            speedBoostZone.setFillColor(sf::Color(0, 255, 0, 50)); // Semi-transparent green
            speedBoostZone.setPosition((gameWidth - 200.f) / 2.f, 0.f);
        
            bool speedBoostActive = false;
            sf::Clock speedBoostClock;
        
            while (window.isOpen())
            {
                // Handle events
                sf::Event event;
                while (window.pollEvent(event))
                {
                    // Window closed or escape key pressed: exit
                    if ((event.type == sf::Event::Closed) ||
                        ((event.type == sf::Event::KeyPressed) && (event.key.code == sf::Keyboard::Escape)))
                    {
                        window.close();
                        break;
                    }
        
                    // Space key pressed: play
                    if (((event.type == sf::Event::KeyPressed) && (event.key.code == sf::Keyboard::Space)) ||
                        (event.type == sf::Event::TouchBegan))
                    {
                        if (!isPlaying)
                        {
                            // (re)start the game
                            isPlaying = true;
                            clock.restart();
        
                            // Reset the position of the paddles and ball
                            leftPaddle.setPosition(10.f + paddleSize.x / 2.f, gameHeight / 2.f);
                            rightPaddle.setPosition(gameWidth - 10.f - paddleSize.x / 2.f, gameHeight / 2.f);
                            ball.setPosition(gameWidth / 2.f, gameHeight / 2.f);
        
                            // Reset the ball angle
                            do
                            {
                                // Make sure the ball initial angle is not too much vertical
                                ballAngle = static_cast<float>(std::rand() % 360) * 2.f * pi / 360.f;
                            } while (std::abs(std::cos(ballAngle)) < 0.7f);
        
                            // Reset the ball speed and color
                            ballSpeed = initialBallSpeed;
                            ball.setFillColor(sf::Color::White);
                            effectActive = false;
                            fakeBalls.clear(); // Clear any existing fake balls
                            speedBoostActive = false;
                        }
                    }
        
                    // Window size changed, adjust view appropriately
                    if (event.type == sf::Event::Resized)
                    {
                        sf::View view;
                        view.setSize(gameWidth, gameHeight);
                        view.setCenter(gameWidth / 2.f, gameHeight / 2.f);
                        window.setView(view);
                    }
                }
        
                if (isPlaying)
                {
                    float deltaTime = clock.restart().asSeconds();
        
                    // Move the player's paddle
                    if (sf::Keyboard::isKeyPressed(sf::Keyboard::Up) &&
                        (leftPaddle.getPosition().y - paddleSize.y / 2 > 5.f))
                    {
                        leftPaddle.move(0.f, -paddleSpeed * deltaTime);
                    }
                    if (sf::Keyboard::isKeyPressed(sf::Keyboard::Down) &&
                        (leftPaddle.getPosition().y + paddleSize.y / 2 < gameHeight - 5.f))
                    {
                        leftPaddle.move(0.f, paddleSpeed * deltaTime);
                    }
        
                    if (sf::Touch::isDown(0))
                    {
                        sf::Vector2i pos = sf::Touch::getPosition(0);
                        sf::Vector2f mappedPos = window.mapPixelToCoords(pos);
                        leftPaddle.setPosition(leftPaddle.getPosition().x, mappedPos.y);
                    }
        
                    // Move the computer's paddle
                    if (((rightPaddleSpeed < 0.f) && (rightPaddle.getPosition().y - paddleSize.y / 2 > 5.f)) ||
                        ((rightPaddleSpeed > 0.f) && (rightPaddle.getPosition().y + paddleSize.y / 2 < gameHeight - 5.f)))
                    {
                        rightPaddle.move(0.f, rightPaddleSpeed * deltaTime);
                    }
        
                    // Update the computer's paddle direction according to the ball position
                    if (AITimer.getElapsedTime() > AITime)
                    {
                        AITimer.restart();
                        if (ball.getPosition().y + ballRadius > rightPaddle.getPosition().y + paddleSize.y / 2)
                            rightPaddleSpeed = paddleSpeed;
                        else if (ball.getPosition().y - ballRadius < rightPaddle.getPosition().y - paddleSize.y / 2)
                            rightPaddleSpeed = -paddleSpeed;
                        else
                            rightPaddleSpeed = 0.f;
                    }
        
                    // Move the ball
                    float factor = ballSpeed * deltaTime;
                    ball.move(std::cos(ballAngle) * factor, std::sin(ballAngle) * factor);
        
        #ifdef SFML_SYSTEM_IOS
                    const std::string inputString = "Touch the screen to restart.";
        #else
                    const std::string inputString = "Press space to restart or\nescape to exit.";
        #endif
        
                    // Check collisions between the ball and the screen
                    if (ball.getPosition().x - ballRadius < 0.f)
                    {
                        isPlaying = false;
                        pauseMessage.setString("You Lost!\n\n" + inputString);
                        aiScore++;
                    }
                    if (ball.getPosition().x + ballRadius > gameWidth)
                    {
                        isPlaying = false;
                        pauseMessage.setString("You Won!\n\n" + inputString);
                        playerScore++;
                    }
                    if (ball.getPosition().y - ballRadius < 0.f)
                    {
                        ballSound.play();
                        ballAngle = -ballAngle;
                        ball.setPosition(ball.getPosition().x, ballRadius + 0.1f);
                    }
                    if (ball.getPosition().y + ballRadius > gameHeight)
                    {
                        ballSound.play();
                        ballAngle = -ballAngle;
                        ball.setPosition(ball.getPosition().x, gameHeight - ballRadius - 0.1f);
                    }
        
                    // Check the collisions between the ball and the paddles
                    // Left Paddle
                    if (ball.getPosition().x - ballRadius < leftPaddle.getPosition().x + paddleSize.x / 2 &&
                        ball.getPosition().x - ballRadius > leftPaddle.getPosition().x &&
                        ball.getPosition().y + ballRadius >= leftPaddle.getPosition().y - paddleSize.y / 2 &&
                        ball.getPosition().y - ballRadius <= leftPaddle.getPosition().y + paddleSize.y / 2)
                    {
                        if (ball.getPosition().y > leftPaddle.getPosition().y)
                            ballAngle = pi - ballAngle + static_cast<float>(std::rand() % 20) * pi / 180;
                        else
                            ballAngle = pi - ballAngle - static_cast<float>(std::rand() % 20) * pi / 180;
        
                        ballSound.play();
                        ball.setPosition(leftPaddle.getPosition().x + ballRadius + paddleSize.x / 2 + 0.1f, ball.getPosition().y);
        
                        // 80% chance to change ball color to red and double speed
                        if (std::rand() % 5 < 4)
                        {
                            ball.setFillColor(sf::Color::Red);
                            ballSpeed *= 2;
                            effectClock.restart();
                            effectActive = true;
        
                            // Create two fake balls
                            for (int i = 0; i < 2; ++i)
                            {
                                FakeBall fakeBall;
                                fakeBall.shape.setRadius(ballRadius - 3);
                                fakeBall.shape.setOutlineThickness(2);
                                fakeBall.shape.setOutlineColor(sf::Color::Black);
                                fakeBall.shape.setFillColor(sf::Color::White);
                                fakeBall.shape.setOrigin(ballRadius / 2, ballRadius / 2);
                                fakeBall.shape.setPosition(ball.getPosition());
                                fakeBall.angle = ballAngle + static_cast<float>(std::rand() % 360) * pi / 180;
                                fakeBall.timer.restart();
                                fakeBalls.push_back(fakeBall);
                            }
                        }
                    }
        
                    // Right Paddle
                    if (ball.getPosition().x + ballRadius > rightPaddle.getPosition().x - paddleSize.x / 2 &&
                        ball.getPosition().x + ballRadius < rightPaddle.getPosition().x &&
                        ball.getPosition().y + ballRadius >= rightPaddle.getPosition().y - paddleSize.y / 2 &&
                        ball.getPosition().y - ballRadius <= rightPaddle.getPosition().y + paddleSize.y / 2)
                    {
                        if (ball.getPosition().y > rightPaddle.getPosition().y)
                            ballAngle = pi - ballAngle + static_cast<float>(std::rand() % 20) * pi / 180;
                        else
                            ballAngle = pi - ballAngle - static_cast<float>(std::rand() % 20) * pi / 180;
        
                        ballSound.play();
                        ball.setPosition(rightPaddle.getPosition().x - ballRadius - paddleSize.x / 2 - 0.1f, ball.getPosition().y);
        
                        // 80% chance to change ball color to red and double speed
                        if (std::rand() % 5 < 4)
                        {
                            ball.setFillColor(sf::Color::Red);
                            ballSpeed *= 2;
                            effectClock.restart();
                            effectActive = true;
        
                            // Create two fake balls
                            for (int i = 0; i < 2; ++i)
                            {
                                FakeBall fakeBall;
                                fakeBall.shape.setRadius(ballRadius - 3);
                                fakeBall.shape.setOutlineThickness(2);
                                fakeBall.shape.setOutlineColor(sf::Color::Black);
                                fakeBall.shape.setFillColor(sf::Color::White);
                                fakeBall.shape.setOrigin(ballRadius / 2, ballRadius / 2);
                                fakeBall.shape.setPosition(ball.getPosition());
                                fakeBall.angle = ballAngle + static_cast<float>(std::rand() % 360) * pi / 180;
                                fakeBall.timer.restart();
                                fakeBalls.push_back(fakeBall);
                            }
                        }
                    }
        
                    // Revert the ball color and speed after 0.5 seconds
                    if (effectActive && effectClock.getElapsedTime() > sf::seconds(0.5f))
                    {
                        ball.setFillColor(sf::Color::White);
                        ballSpeed = initialBallSpeed;
                        effectActive = false;
                    }
        
                    // Update and remove fake balls after 0.5 seconds
                    for (auto it = fakeBalls.begin(); it != fakeBalls.end(); )
                    {
                        if (it->timer.getElapsedTime() > sf::seconds(0.5f))
                        {
                            it = fakeBalls.erase(it);
                        }
                        else
                        {
                            float fakeFactor = ballSpeed * deltaTime;
                            it->shape.move(std::cos(it->angle) * fakeFactor, std::sin(it->angle) * fakeFactor);
                            ++it;
                        }
                    }
        
                    // Check if ball is in the speed boost zone
                    if (ball.getPosition().x > speedBoostZone.getPosition().x &&
                        ball.getPosition().x < speedBoostZone.getPosition().x + speedBoostZone.getSize().x)
                    {
                        if (!speedBoostActive)
                        {
                            ballSpeed *= speedBoost;
                            speedBoostActive = true;
                            speedBoostClock.restart();
                        }
                    }
        
                    // Revert the ball speed after 0.5 seconds if it was boosted
                    if (speedBoostActive && speedBoostClock.getElapsedTime() > sf::seconds(0.5f))
                    {
                        ballSpeed = initialBallSpeed;
                        speedBoostActive = false;
                    }
        
                    // Update the score text
                    scoreText.setString("Player: " + std::to_string(playerScore) + "  AI: " + std::to_string(aiScore));
                }
        
                // Clear the window
                window.clear(sf::Color(50, 50, 50));
        
                if (isPlaying)
                {
                    // Draw the paddles, the ball, and the speed boost zone
                    window.draw(leftPaddle);
                    window.draw(rightPaddle);
                    window.draw(ball);
                    for (const auto& fakeBall : fakeBalls)
                    {
                        window.draw(fakeBall.shape);
                    }
                    window.draw(speedBoostZone);
                    window.draw(scoreText); // Draw the score
                }
                else
                {
                    // Draw the pause message
                    window.draw(pauseMessage);
                    window.draw(sfmlLogo);
                }
        
                // Display things on screen
                window.display();
            }
        
            return EXIT_SUCCESS;
        }
