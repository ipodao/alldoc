[toc]

## 3. Air Hockey

内容介绍：setting up the project's configuration, loading images, loading sounds, building a game for more than one screen resolution, and managing touch events.

这个游戏是双人游戏。

### 游戏配置

游戏特征：

- 支持多点触摸，因为是双人游戏
- 在大屏上玩
- 支持retina屏
- 屏幕水平

#### 行动：创建游戏工程

接下来将使用Xcode创建所有工程：

1. Open Xcode and create a new project with the Cocos2d-x basic template.
2. Call it Air Hockey, and set its **Device Family** to iPad.

#### 行动：应用规则

更新`AppController.mm`和`RootViewController.mm`：

Go to `AppController.mm` and inside the `(BOOL) application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions` method below the section where the `EAGLView *__glView` object is instantiated, write this line: `[__glView setMultipleTouchEnabled:YES];`（支持多点）

Now go to `RootViewController.mm` and look for the `shouldAutorotateToInterfaceOrientation` method. Change the line inside the method to read:
`return UIInterfaceOrientationIsPortrait( interfaceOrientation );`

Finally, a few lines below in the `supportedInterfaceOrientations` method change the line inside the conditional to: `return UIInterfaceOrientationMaskPortrait;`

### 支持retina屏

#### 行动：添加图片

打开`7341_03_RESOURCES.zip`。里面有两个文件夹分别是`hd`和`sd`，分别用于retina屏和非retina屏。将这两个文件夹拖动你工程的`Resources`文件夹。
Go back to Xcode. Select the Resources folder in your project navigation panel. Then go to `File| Add Files` to Air Hockey.
In the File window navigate to the Resources folder and select both the sd and hd folders. This is very important: Make sure Create folder references for any added folders is selected. Also make sure you selected Air Hockey as the target. It wouldn't hurt to make sure **Copy items to destination...** is also selected. Click Add.

It is very important that references are added to the actual folders, only in this way will Xcode be able to have two files with the same name inside the project and still keep them apart; one in each folder.

#### 行动：添加Retina支持

修改`AppDelegate.cpp`类。向`applicationDidFinishLaunching`方法，`pDirector->setOpenGLView( pEGLView)`之下添加：

```cpp
    CCSize screenSize = pEGLView->getFrameSize();
    CCEGLView::sharedOpenGLView()->setDesignResolutionSize(768, 1024, kResolutionExactFit);
    if (screenSize.width > 768) {
        CCFileUtils::sharedFileUtils()->setResourceDirectory("hd");
        pDirector->setContentScaleFactor(2);
    } else {
        CCFileUtils::sharedFileUtils()->setResourceDirectory("sd");
        pDirector->setContentScaleFactor(1);
    }
```

`setDesignResolutionSize`表达了，我设计游戏时使用的分辨率是`768 x 1024`。即所有的定位与字体都相对于这个分辨率。

`CCFileUtils`会先在**rces| sd (or hd)**中寻找资源，如果找不到，再在Resources中找。即两个分辨率都一样的文件可以放在Resources中，不用重复两份。例如声音文件。

### 添加声效

将`.wav`拖动到你的工程的`Resources`文件夹。Then go to Xcode, select the Resources folder in the file navigation panel and select **File | Add Files** to Air Hockey. Make sure the Air Hockey target is selected. Add the **wav** files.

在`AppDelegate.cpp`顶部添加：`#include "SimpleAudioEngine.h"`。在`USING_NS_CC`下添加：`using namespace CocosDenshion;`。

在`applicationDidFinishLaunching`方法，上一节添加的内容之下添加：

```cpp
    SimpleAudioEngine::sharedEngine()->preloadEffect(
    	CCFileUtils::sharedFileUtils()->fullPathFromRelativePath("hit.wav") );
    SimpleAudioEngine::sharedEngine()->preloadEffect(
    	CCFileUtils::sharedFileUtils()->fullPathFromRelativePath("score.wav") );
```

加载声音很费时，因此尽早做。

### 扩展`CCSprite`

我们需要一些额外的信息，因此扩展`CCSprite`。

#### 行动：定义`GameSprite`

GameSprite.h：

```cpp
    #ifndef __GAMESPRITE_H__
    #define __GAMESPRITE_H__
    #include "cocos2d.h"
    using namespace cocos2d;
    class GameSprite : public CCSprite {
    public:
        CC_SYNTHESIZE(CCPoint, _nextPosition, NextPosition);
        CC_SYNTHESIZE(CCPoint, _vector, Vector);
        CC_SYNTHESIZE(CCTouch *, _touch, Touch);
        GameSprite(void);
        ~GameSprite(void);
        static GameSprite* gameSpriteWithFile(const char *pszFileName);
        virtual void setPosition(const CCPoint& pos);
        float radius();
    };
    #endif // __GAMESPRITE_H__
```

利用宏创建三个合成（synthesized）属性：利用红创建getter和setter方法。You declare the type, the `protected` variable name, and the words that will be appended to the get and set methods.

`gameSpriteWithFile`用于实例化。

#### 行动：实现`GameSprite`

`GameSprite.cpp`：

```cpp
    #include "GameSprite.h"
    GameSprite::GameSprite(void) {
    	_vector = ccp(0,0);
    }
    GameSprite::~GameSprite(void) {
    }
    GameSprite* GameSprite::gameSpriteWithFile(const char *pszFileName) {
        GameSprite *sprite = new GameSprite();
        if (sprite && sprite->initWithFile(pszFileName)) {
        	sprite->autorelease();
        	return sprite;
        }
        CC_SAFE_DELETE(sprite);
        return NULL;
    }

    float GameSprite::radius() {
        return getTexture()->getContentSize().width * 0.5f;
    }
```

覆盖`CCNode`的`setPosition`方法。为的是，改变位置后，新值更新到`_nextPosition`：

```cpp
    void GameSprite::setPosition(const CCPoint& pos) {
        CCSprite::setPosition(pos);
        if (!_nextPosition.equals(pos)) {
        	_nextPosition = pos;
        }
    }
```

### game scene

Rename the `HelloWorldScene` files to `GameLayer`, and the class inside them from `HelloWorld` to `GameLayer`. References to the class must be changed in two lines of `AppDelegate.cpp`, and also the `include` statement at the top of `HelloWorldScene.cpp`. Of course, any scope resolution changes (the `HelloWorld::` funny bits) must be changed in `HelloWorldScene.cpp` as well.

#### 行动：`GameLayer`接口

修改`GameLayer.h`。替换为

```cpp
	#ifndef __GAMELAYER_H__
        #define __GAMELAYER_H__
        #define GOAL_WIDTH 400

        #include "cocos2d.h"
        #include "GameSprite.h"
        using namespace cocos2d;

        class GameLayer : public cocos2d::CCLayer
        {
            GameSprite * _player1;
            GameSprite * _player2;
            GameSprite * _ball;
            CCArray * _players;
            CCLabelTTF * _player1ScoreLabel;
            CCLabelTTF * _player2ScoreLabel;

            CCSize _screenSize; // 因为经常使用，存下来

            // 得分与更新得分的方法
            int _player1Score;
            int _player2Score;
            void playerScore (int player);

            public:
            ~GameLayer();
            virtual bool init();
            static CCScene* scene();
            CREATE_FUNC(GameLayer);
            virtual void ccTouchesBegan(CCSet* pTouches, CCEvent* event);
            virtual void ccTouchesMoved(CCSet* pTouches, CCEvent* event);
            virtual void ccTouchesEnded(CCSet* pTouches, CCEvent* event);
            void update (float dt);
        };
    #endif // __GAMELAYER_H__
```

#### 行动：实现`init()`

在调用父类方法`CCLayer::init`之后，添加：

```cpp
    _player1Score = 0;
    _player2Score = 0;
    _screenSize = CCDirector::sharedDirector()->getWinSize();
```

屏幕大小将用于定位精灵。

加载第一个精灵：

```cpp
    CCSprite * court = CCSprite::create("court.png");
    court->setPosition(ccp(_screenSize.width * 0.5, _screenSize.height * 0.5));
    this->addChild(court);
```

宏`ccp`用于创建点。

创建`GameSprite`精灵：

```cpp
    _player1 = GameSprite::gameSpriteWithFile("mallet.png");
    _player1->setPosition(ccp(_screenSize.width * 0.5,
    	_player1->radius() * 2));
    this->addChild(_player1);

    _player2 = GameSprite::gameSpriteWithFile("mallet.png");
    _player2->setPosition(ccp(_screenSize.width * 0.5,
    	_screenSize.height - _player1->radius() * 2));
    this->addChild(_player2);

    _ball = GameSprite::gameSpriteWithFile("puck.png");
    _ball->setPosition(ccp(_screenSize.width * 0.5,
    	_screenSize.height * 0.5 - 2 * _ball->radius()));
    this->addChild(_ball);
```

We create a `CCArray` method to store the player objects and we **retain** this array to keep a reference to it throughout the game:

```cpp
    _players = CCArray::create(_player1, _player2, NULL);
    _players->retain();
```

利用`CCLabelTTF`创建标签。once again the font size will be automatically scaled in the high definition version)。

```cpp
    _player1ScoreLabel = CCLabelTTF::create("0", "Arial", 60);
    _player1ScoreLabel->setPosition(ccp(_screenSize.width - 60,
    	_screenSize.height * 0.5 - 80));
    _player1ScoreLabel->setRotation(90);
    this->addChild(_player1ScoreLabel);

    _player2ScoreLabel = CCLabelTTF::create("0", "Arial", 60);
    _player2ScoreLabel->setPosition(ccp(_screenSize.width - 60,
    	_screenSize.height * 0.5 + 80));
    _player2ScoreLabel->setRotation(90);
    this->addChild(_player2ScoreLabel);
```

Label objects (`CCLabelTTF`) can use any of the fonts supported by the target system; these change from system to system, however. But there is an option of loading your own TTF files.

最后，声明`CCLayer`允许监听触摸事件。并开始排期主循环：

```cpp
    //listen for touches
    this->setTouchEnabled(true);
    //create main loop
    this->schedule(schedule_selector(GameLayer::update));
    return true;
```

在析构器中释放之前retain的数组：

```cpp
    GameLayer::~GameLayer() {
    	CC_SAFE_RELEASE(_players);
    }
```

![](ch3-demo-1.png)

#### 行动：处理多点触摸

`ccTouchesBegan`方法：

```cpp
    void GameLayer::ccTouchesBegan(CCSet* pTouches, CCEvent* event) {
        CCSetIterator i;
        CCTouch* touch;
        CCPoint tap;
        GameSprite * player;
        for( i = pTouches->begin(); i != pTouches->end(); i++) {
            touch = (CCTouch*) (*i);
            if(touch) {
                tap = touch->getLocation();
                for (int p = 0; p < 2; p++) {
                    player = (GameSprite *) _players->objectAtIndex(p);
                    if (player->boundingBox().containsPoint(tap)) {
                        player->setTouch(touch);
                    }
                }
            }
        }
    }
```

每个`GameSprite`都有一个`_touch`属性。如果触摸了某个精灵，就将`touch`存到其`_touch`属性中去。

`ccTouchesMoved`方法（外层略）：

```cpp
    for (int p = 0; p < _players->count(); p++) {
        player = (GameSprite *) _players->objectAtIndex(p);
        if (player->getTouch() != NULL && player->getTouch() == touch) {
        CCPoint nextPosition = tap;
        // 不能移出屏幕
        if (nextPosition.x < player->radius())
        	nextPosition.x = player->radius();
        if (nextPosition.x > _screenSize.width - player->radius())
        	nextPosition.x = _screenSize.width - player->radius();
        if (nextPosition.y < player->radius())
        	nextPosition.y = player->radius();
        if (nextPosition.y > _screenSize.height - player->radius())
        	nextPosition.y = _screenSize.height - player->radius();
        // 不能移出自己的半场
        if (player->getPositionY() < _screenSize.height * 0.5f)
        {
	        if (nextPosition.y >  _screenSize.height * 0.5 - player->radius())
        	{
        		nextPosition.y = _screenSize.height * 0.5 - player->radius();
        	}
        } else {
        	if (nextPosition.y < _screenSize.height * 0.5 + player->radius())
	        {
    		    nextPosition.y = _screenSize.height * 0.5 + player->radius();
        	}
        }

        player->setNextPosition(nextPosition);
        player->setVector(ccp(tap.x - player->getPositionX(),
	        tap.y - player->getPositionY()));
        }
    }
```

精灵的`vector`会在碰撞检测时用到。

`ccTouchesEnded`方法（外层略）：

```cpp
    for (int p = 0; p < _players->count(); p++) {
    	player = (GameSprite *) _players->objectAtIndex(p);
    	if (player->getTouch() != NULL && player->getTouch() == touch) {
    		player->setTouch(NULL);
    		player->setVector(ccp(0,0)););
    	}
    }
```

实现多点触摸的另一种方式是实现`CCTargetedTouchDelegate`协议。But this may result in the implementation of up to eight methods. You may go to the test code in samples/TestCpp/Classes/TouchesTest and review the code used in the Paddle.h and Paddle.cpp files for an example of `CCTargetedTouchDelegate` in action.

#### 行动：主循环

`update`方法。

对速度施加一点摩擦力（0.98f）。We store what its next position will be at the end of the iteration, if no collision occurred:

```cpp
    void GameLayer::update (float dt) {
        CCPoint ballNextPosition = _ball->getNextPosition();
        CCPoint ballVector = _ball->getVector();
        ballVector = ccpMult(ballVector, 0.98f);
        ballNextPosition.x += ballVector.x;
        ballNextPosition.y += ballVector.y;
```

接下来碰撞检测：

```cpp
        GameSprite * player;
        CCPoint playerNextPosition;
        CCPoint playerVector;

        float squared_radii = pow(_player1->radius() + _ball->radius(), 2);
        for (int p = 0; p < _players->count(); p++) {
            player = (GameSprite *) _players->objectAtIndex(p);
            playerNextPosition = player->getNextPosition();
            playerVector = player->getVector();
            float diffx = ballNextPosition.x - player->getPositionX();
            float diffy = ballNextPosition.y - player->getPositionY();
            float distance1 = pow(diffx, 2) + pow(diffy, 2);
            float distance2 = pow(_ball->getPositionX() - playerNextPosition.x, 2)
                + pow(_ball->getPositionY() - playerNextPosition.y, 2);
```

These conditions are checked both with the player's current position and its next position, so there is less risk of the ball moving "through" the player sprite between iterations.

If there is a collision, we grab the magnitudes of both the ball's vector and the player's vector, and calculate the force with which the ball will be pushed away. We update the ball's next position in that case, and play a nice sound effect through the `SimpleAudioEngine` singleton:

```cpp
            float mag_ball = pow(ballVector.x, 2) + pow(ballVector.y, 2);
            float mag_player = pow(playerVector.x, 2) + pow(playerVector.y, 2);
            float force = sqrt(mag_ball + mag_player);
            float angle = atan2(diffy, diffx);
            ballVector.x = force * cos(angle);
            ballVector.y = (force * sin(angle));
            ballNextPosition.x = playerNextPosition.x +
                (player->radius() + _ball->radius() + force) * cos(angle);
            ballNextPosition.y = playerNextPosition.y +
                (player->radius() + _ball->radius() + force) * sin(angle);
            SimpleAudioEngine::sharedEngine()->playEffect("hit.wav");
        }
    }
```

Next, check for collision between ball and screen sides. If found, we move the ball back into the court and play our sound effect here as well:

```cpp
    if (ballNextPosition.x < _ball->radius()) {
        ballNextPosition.x = _ball->radius();
        ballVector.x *= -0.8f;
        SimpleAudioEngine::sharedEngine()->playEffect("hit.wav");
    }
    if (ballNextPosition.x > _screenSize.width - _ball->radius()) {
        ballNextPosition.x = _screenSize.width - _ball->radius();
        ballVector.x *= -0.8f;
        SimpleAudioEngine::sharedEngine()->playEffect("hit.wav");
    }
```

At the top and bottom sides of the court we check to see if the ball has not moved through one of the goals through our previously defined `GOAL_WIDTH` property, as follows:

```cpp
    if (ballNextPosition.y > _screenSize.height - _ball->radius()) {
        if (_ball->getPosition().x < _screenSize.width * 0.5f - GOAL_WIDTH * 0.5f
        	|| _ball->getPosition().x > _screenSize.width * 0.5f + GOAL_WIDTH * 0.5f) {
	        ballNextPosition.y = _screenSize.height - _ball->radius();
    	    ballVector.y *= -0.8f;
        	SimpleAudioEngine::sharedEngine()->playEffect("hit.wav");
        }
    }
    if (ballNextPosition.y < _ball->radius() ) {
        if (_ball->getPosition().x < _screenSize.width * 0.5f - GOAL_WIDTH * 0.5f
        	|| _ball->getPosition().x > _screenSize.width * 0.5f + GOAL_WIDTH * 0.5f) {
            ballNextPosition.y = _ball->radius();
            ballVector.y *= -0.8f;
            SimpleAudioEngine::sharedEngine()->playEffect("hit.wav");
        }
    }
```

We finally update the ball information, and if the ball has passed through the goal posts (drum roll):

```cpp
    _ball->setVector(ballVector);
    _ball->setNextPosition(ballNextPosition);
    //check for goals!
    if (ballNextPosition.y < -_ball->radius() * 2) {
    	this->playerScore(2);
    }
    if (ballNextPosition.y > _screenSize.height + _ball->radius() * 2) {
    	this->playerScore(1);
    }
```

We call our helper method to score a point, and we finish the update with the placement of all the elements, now that we know where the `nextPosition` is for each one of the elements in the game:

```cpp
    _player1->setPosition(_player1->getNextPosition());
    _player2->setPosition(_player2->getNextPosition());
    _ball->setPosition(_ball->getNextPosition());
```

Whenever your gameplay depends on precise collision detection you will undoubtedly apply a similar logic of: position now, position next, collision checks, and adjustments to position next, if any collision has occurred.

#### 行动：更新分数

We start by playing a nice effect for a goal and stopping our ball:

```cpp
    void GameLayer::playerScore (int player) {
        SimpleAudioEngine::sharedEngine()->playEffect("score.wav");
        _ball->setVector(ccp(0,0));
```

Then we update the score for the scoring player, updating the score label in the process. And the ball moves to the court of the player against whom a point was just scored:

```cpp
        char score_buffer[10];
        if (player == 1) {
            _player1Score++;
            sprintf(score_buffer,"%i", _player1Score);
            _player1ScoreLabel->setString(score_buffer);
            _ball->setNextPosition(ccp(_screenSize.width * 0.5,
            	_screenSize.height * 0.5 + 2 * _ball->radius()));
        } else {
            _player2Score++;
            sprintf(score_buffer,"%i", _player2Score);
            _player2ScoreLabel->setString(score_buffer);
            _ball->setNextPosition(ccp(_screenSize.width * 0.5,
            	_screenSize.height * 0.5 - 2 * _ball->radius()));
        }
```

The players are moved to their original position and their `_touch` properties are cleared:

```cpp
        _player1->setPosition(ccp(_screenSize.width * 0.5,
        	_player1->radius() * 2));
        _player2->setPosition(ccp(_screenSize.width * 0.5,
        	_screenSize.height - _player1->radius() * 2));
        _player1->setTouch(NULL);
        _player2->setTouch(NULL);
    }
```

也可以使用`CCString`：

```cpp
    CCString * score = CCString::createWithFormat("%i", _player1Score);
    _player1ScoreLabel->setString(score->getCString());
```


## 4. Sky Defense

This time you will become acquainted with the power of **actions** in Cocos2d-x. I'll show you how an entire game can be built just by running the various action commands contained in Cocos2d-x, to make your sprites move, rotate, scale, fade, blink, and so on. And you can also use actions to animate your sprites by using multiple images, as in a movie. So let's get started.

In this chapter you will learn:

* 利用sprite sheets优化游戏开发
* 在游戏中使用bitmap fonts
* How easy it is to implement and run CCActions
* How to scale, rotate, swing, move, and fade out a sprite
* 使用多个.png做精灵动画
* How to create a universal game with Cocos2d-x

### The game – Sky Defense

游戏剧情：Meet our stressed out city of... your name of choice here. It's a beautiful day, when suddenly the sky begins to fall. There are meteors rushing towards the city and it is your job to keep it safe.

The player in this game can tap the screen to start growing a bomb. When the bomb is big enough to be activated, the player taps the screen again to detonate it. Any nearby meteor will explode into a million pieces. The bigger the bomb, the bigger the detonation and the more meteors can be taken out by it. But the bigger the bomb, the longer it takes to grow it.

But it's not just bad news coming down. There are also health packs dropping from the sky and if you allow them to reach the ground, you'll recover some of your energy.

#### The game settings

This is a **universal** game. It is designed for the iPad retina screen and it will be scaled down to fit other screens. 游戏屏幕横屏。不支持多点触摸。

#### The start project

解压从`7341_04_START_PROJECT.zip`开始。Only this time, the **Device Family** is set to **Universal**. And in `RootViewController.mm`, the supported interface orientation is set to Landscape. 这次我们只需要一个类`GameLayer.cpp`, and you will find that the interface for this class already contains all of the information it needs.

#### Adding screen support for a universal app

上一个工程只支持iPad屏。现在要支持更小的屏。打开`AppDelegate.cpp`，在`applicationDidFinishLaunching`中：

```cpp
    CCSize screenSize = pEGLView->getFrameSize();
    CCSize designSize = CCSize(2048, 1536);
    CCEGLView::sharedOpenGLView()->setDesignResolutionSize(designSize.width,
    	designSize.height, kResolutionExactFit);
    if (screenSize.height > 768) {
    	CCFileUtils::sharedFileUtils()->setResourceDirectory("ipadhd");
    } else if (screenSize.height > 320) {
    	CCFileUtils::sharedFileUtils()->setResourceDirectory("ipad");
    } else {
    	CCFileUtils::sharedFileUtils()->setResourceDirectory("iphone");
    }
    pDirector->setContentScaleFactor(screenSize.height/designSize.height);
```

Once again we tell our CCEGLView object (our OpenGL view) that we have designed the game for a certain screen size (the iPad retina screen) and once again we want our game screen to resize to match the screen on the device (`kResolutionExactFit`). Then we determine where to load our images from, based on the device's screen size. We have art for iPad retina, then for a regular iPad which is shared by iPhone retina, and finally for the regular iPhone. 最后设置缩放因数。

#### 添加背景音乐

`AppDelegate.cpp`：

```cpp
	SimpleAudioEngine::sharedEngine()->preloadBackgroundMusic(file);
    //lower playback volume for effects
    SimpleAudioEngine::sharedEngine()->setEffectsVolume(0.4f);
```

背景音乐的音量通过`setBackgroundMusicVolume`设置。

#### 初始化游戏

回到`GameLayer.cpp`，查看`init`发现游戏初始化涉及三个方法：`createGameScreen`, `createPools`, `createActions`。

使用对象池，为了不必在主循环中再初始化精灵。

There is a `CCArray` called `_fallingObjects` also created here, and we start playing the background music, with the loop flag set to true:

```cpp
    SimpleAudioEngine::sharedEngine()->playBackgroundMusic("background.mp3", true);
```

### 使用sprite sheets{{纹理贴图}}

A sprite sheet is a way to group multiple images together into one image file. In order to texture a sprite with one of these images you must have the information of where in the sprite sheet that particular image is found (its rectangle). Sprite sheets通常组织成两个文件：图片文件和数据文件。

I used TexturePacker to create these files for the game. You can find them inside the ipad, ipadhd, and iphone folders inside **Resources**. There is a **sprite_sheet.png** for the image and a **sprite_sheet.plist** that describes the individual frames inside the image.

This is what the **sprite_sheet.png** file looks like:

![](ch4-sprite-sheet.png)

Sprite sheets与一个特殊的`CCNode`类连用：`CCSpriteBatchNode`。当同一个节点内的多个精灵使用同一个图片文件时可以使用此类。With `CCSpriteBatchNode`, you can substantially reduce the number of calls during the rendering stage of your game, which will help when targeting less powerful systems, though **not noticeably** in the Apple device family.

`CCSpriteBatchNode`像其他任何节点一样可以充当容器。利用z-order可以将`CCSprites`在batch node呢分层排布。

#### 行动：创建一个`CCSpriteBatchNode`

下面实现`GameLayer.cpp`的`createGameScreen`方法。Just below the lines that add the `bg` sprite, we instantiate our batch node:

```cpp
    void GameLayer::createGameScreen() {
        //add bg
        CCSprite * bg = CCSprite::create("bg.png");
        ...
        CCSpriteFrameCache::sharedSpriteFrameCache()->
        	addSpriteFramesWithFile("sprite_sheet.plist");
        _gameBatchNode = CCSpriteBatchNode::create("sprite_sheet.png");
        this->addChild(_gameBatchNode);
```

要从sprite sheet创建batch node，需要先加载帧信息：将`sprite_sheet.plist`加载到`CCSpriteFrameCache`。然后用`sprite_sheet.png`创建batch node。（背景图片不在贴图内。）

创建`CCSprites`使用的帧名，也会出现在加载到`sprite_sheet.plist`中。

接下来向`CCSpriteBatchNode`添加精灵。首先是city:

```cpp
    CCSprite * sprite;
    for (int i = 0; i < 2; i++) {
        sprite = CCSprite::createWithSpriteFrameName("city_dark.png");
        sprite->setPosition(ccp(_screenSize.width * (0.25f + i * 0.5f),
            sprite->boundingBox().size.height * 0.5f));
        _gameBatchNode->addChild(sprite, kForeground);
        sprite = CCSprite::createWithSpriteFrameName("city_light.png");
        sprite->setPosition(ccp(_screenSize.width * (0.25f + i * 0.5f),
            sprite->boundingBox().size.height * 0.9f));
        _gameBatchNode->addChild(sprite, kBackground);
    }
```

然后是树：

```cpp
    //add trees
    for (int i = 0; i < 3; i++) {
        sprite = CCSprite::createWithSpriteFrameName("trees.png");
        sprite->setPosition(ccp(
        	_screenSize.width * (0.2f + i * 0.3f),
        	sprite->boundingBox().size.height * 0.5f));
        _gameBatchNode->addChild(sprite, kForeground);
    }
```

The screen so far is made up of two instances of city_dark.pngtiling at the bottom of the screen, and two instances of city_light.pngthat are also tiling. One needs to appear on top of the other, and for that we use the enumerated values declared in `GameLayer.h`:

```cpp
    enum {
        kBackground,
        kMiddleground,
        kForeground
    };
```

### Bitmap字体

`CCLabelBMFont`用bitmap图像显式字母，而`CCLabelTTF`用的是true type font文件。

The bitmap image we are using here was created with the program **GlyphDesigner**, and in essence it works just as a sprite sheet does. 其实`CCLabelBMFont`是`CCSpriteBatchNode`的子类, so it behaves just like a batch node. You have images for all of the individual characters that you'll need packed inside a PNG file (font.png), and then a data file (font.fnt) describing where each character is.

![](ch4-bitmap-font.png)

The difference between `CCLabelBMFont` and a regular `CCSpriteBatchNode` is that the data file also feeds the `CCLabelBMFont` object information on how to "write" with this font. In other words, how to space out the characters and lines correctly. 创建`CCLabelBMFont`是需要的参数依次是：初始字符串值、数据文件名，标签对象的宽度。

```cpp
    _scoreDisplay = CCLabelBMFont::create("0", "font.fnt", _screenSize.width * 0.3f);
```

可以通过`setString`改变标签内容：

```cpp
	_scoreDisplay->setString("My new Label");
```

#### 行动：创建`CCLabelBMFont`

仍然在`createGameScreen`方法中：

```cpp
    _scoreDisplay = CCLabelBMFont::create("0", "font.fnt",
    	_screenSize.width * 0.3f);
    _scoreDisplay->setAnchorPoint(ccp(1, 0.5));
    _scoreDisplay->setPosition(ccp(_screenSize.width * 0.8f,
		_screenSize.height * 0.94f));
    this->addChild(_scoreDisplay);

    // And then add a label to display the energy level:
    _energyDisplay = CCLabelBMFont::create("100%", "font.fnt",
    	_screenSize.width * 0.1f, kCCTextAlignmentRight);
    _energyDisplay->setPosition(ccp(_screenSize.width * 0.3f,
    	_screenSize.height * 0.94f));
    this->addChild(_energyDisplay);

    // _energyDisplaylabel后面的一个图标
    CCSprite * icon = CCSprite::createWithSpriteFrameName("health_icon.png");
    icon->setPosition(ccp(_screenSize.width * 0.15f,
    	_screenSize.height * 0.94f));
    _gameBatchNode->addChild(icon, kBackground);
```

#### 行动：添加最终的屏幕精灵

最后需要创建的精灵是云、炸弹和冲击波，及游戏的状态消息。

仍然在`createGameScreen`，添加云：

```cpp
    CCSprite * cloud;
    _clouds = CCArray::createWithCapacity(4); // 用数组，为了将来移动云
    _clouds->retain();
    float cloud_y;
    for (int i = 0; i < 4; i++) {
        cloud_y = i % 2 == 0 ? _screenSize.height * 0.4f : _screenSize.height * 0.5f;
        cloud = CCSprite::createWithSpriteFrameName("cloud.png");
        cloud->setPosition(ccp (_screenSize.width * 0.1f + i * _screenSize.width * 0.3f, cloud_y));
        _gameBatchNode->addChild(cloud, kBackground);
        _clouds->addObject(cloud);
    }
```

创建`_bomb`精灵，用户按住屏幕时会变大：

```cpp
    _bomb = CCSprite::createWithSpriteFrameName("bomb.png");
    _bomb->getTexture()->generateMipmap();
    _bomb->setVisible(false);
    CCSize size = _bomb->boundingBox().size;
    //add sparkle inside bomb sprite
    CCSprite * sparkle = CCSprite::createWithSpriteFrameName("sparkle.png");
    sparkle->setPosition(ccp(size.width * 0.72f, size.height * 0.72f));
    _bomb->addChild(sparkle, kMiddleground, kSpriteSparkle);
    //add halo inside bomb sprite
    CCSprite * halo = CCSprite::createWithSpriteFrameName("halo.png");
    halo->setPosition(ccp(size.width * 0.4f, size.height * 0.4f));
    _bomb->addChild(halo, kMiddleground, kSpriteHalo);
    _gameBatchNode->addChild(_bomb, kForeground);
```

`_shockwave`精灵，在`_bomb`消失后出现：

```cpp
    _shockWave = CCSprite::createWithSpriteFrameName("shockwave.png");
    _shockWave->getTexture()->generateMipmap();
    _shockWave->setVisible(false);
    _gameBatchNode->addChild(_shockWave);
```

最后，添加两条消息，分别用于游戏开始和结束状态：

```cpp
    _introMessage = CCSprite::createWithSpriteFrameName("logo.png");
    _introMessage->setPosition(ccp(_screenSize.width * 0.5f,
        _screenSize.height * 0.6f));
    _introMessage->setVisible(true);
    this->addChild(_introMessage, kForeground);

    _gameOverMessage = CCSprite::createWithSpriteFrameName("gameover.png");
    _gameOverMessage->setPosition(ccp(_screenSize.width * 0.5f,
        _screenSize.height * 0.65f));
    _gameOverMessage->setVisible(false);
    this->addChild(_gameOverMessage, kForeground);
```


`_bomb->getTexture()->generateMipmap();` With this, we are telling the framework to create antialiased copies of this texture in diminishing sizes (mipmaps), since we are going to scale it down later. This is optional of course, as sprites can be resized without first generating mipmaps, but if you notice a loss of quality in the scaled sprites, you can fix it by creating mipmaps for their texture.

> OpenGL的纹理大小必须是POT (power of two: 2, 4, 8, 16, and so on)。若不是Cocos2d-x将做两件事情：在内存中调整纹理大小，添加透明像素直到达到POT。或者，可能在某个`Assert`处停止执行。With textures used for mipmaps the framework will stop execution for non-POT textures.

I add the sparkle and the halo sprites as children to the `_bomb` sprite. This will use the container characteristic of `CCNodes` to our advantage. 当炸弹变大时，它的孩子也会跟着变大。

`addChild`的第三个参数是一个整数标签：
```cpp
	bomb->addChild(halo, kMiddleground, kSpriteHalo);
```

该标签来自`GameLayer.h`中的另一个枚举。利用此标签，可以从精灵中取到它的孩子：

```cpp
	CCSprite * halo = (CCSprite *) bomb->getChildByTag(kSpriteHalo);
```

![](ch4-demo-1.png)

#### 行动：创建对象池

池只是一个对象数组。在`createPools`方法：

```cpp
    void GameLayer::createPools() {
        CCSprite * sprite;
        int i;
        // 流星池
        _meteorPool = CCArray::createWithCapacity(50);
        _meteorPool->retain();
        _meteorPoolIndex = 0;
        for (i = 0; i < 50; i++) {
            sprite = CCSprite::createWithSpriteFrameName("meteor.png");
            sprite->setVisible(false);
            _gameBatchNode->addChild(sprite, kMiddleground, kSpriteMeteor);
            _meteorPool->addObject(sprite);
        }

        // 医疗包
        _healthPool = CCArray::createWithCapacity(20);
        _healthPool->retain();
        _healthPoolIndex = 0;
        for (i = 0; i < 20; i++) {
            sprite = CCSprite::createWithSpriteFrameName("health.png");
            sprite->setVisible(false);
            sprite->setAnchorPoint(ccp(0.5f, 0.8f));
            _gameBatchNode->addChild(sprite, kMiddleground, kSpriteHealth);
            _healthPool->addObject(sprite);
        }
    }
```

We'll use the corresponding pool index to retrieve objects from the arrays as the game progresses.

### CCActions

`CCNode`存储着节点的位置、缩放、旋转、可见性、透明度信息。可以通过`CCAction`类改变这些值，即产生动画。Actions一般通过静态方法create创建。第一个参数一般是action的时长。例如：
```cpp
	CCFadeOut *fadeout = CCFadeOut::create(1.0f);
```

`1.0f`表示1秒。令节点运行此Action：
```cpp
	mySprite->runAction(fadeout);
```

还可以创建action序列（`CCSequence`）；or you can apply easing effects (CCEaseIn, CCEaseOut, and so on) to your actions. 可以重复action数次（`CCRepeat`）甚至永远（`CCRepeatForever`）；可以指定一个回调函数，在Action完成后执行。

#### 行动：创建Actions

在`createActions`方法中，实例化在游戏中反复使用的actions。

```cpp
    void GameLayer::createActions() {
        //swing action for health drops
        CCFiniteTimeAction* easeSwing = CCSequence::create(
            CCEaseInOut::create(CCRotateTo::create(1.2f, -10), 2),
            CCEaseInOut::create(CCRotateTo::create(1.2f, 10), 2),
            NULL);
        _swingHealth = CCRepeatForever::create((CCActionInterval*) easeSwing);
        _swingHealth->retain();

        //action sequence for shockwave: fade out, callback when done
        _shockwaveSequence = CCSequence::create(
            CCFadeOut::create(1.0f),
            CCCallFunc::create(this, callfunc_selector(GameLayer::shockwaveDone)),
            NULL);
        _shockwaveSequence->retain();

		// action to grow bomb
        _growBomb = CCScaleTo::create(6.0f, 1.0);
        _growBomb->retain();

        //action to rotate sprites
        CCActionInterval* rotate = CCRotateBy::create(0.5f , -90);
        _rotateSprite = CCRepeatForever::create( rotate );
        _rotateSprite->retain();
```

### 精灵动画

动画仅是另一种形式的`CCAction`——改变的是`CCSprite`使用的纹理。动画action(`CCAnimate`)使用`CCAnimation`对象。`CCAnimation`包含动画所需的所有纹理。纹理（帧）是`CCSpriteFrame`对象，从`CCSpriteFrameCache`获取，后者包含`sprite_sheet.plist`中的信息。


#### 创建动画

仍在`createActions`方法。首先是流行到达城市时的爆炸。首先将帧加载到`CCAnimation`对象：

```cpp
        CCAnimation* animation;
        CCSpriteFrame * frame;
        //create CCAnimation object
        animation = CCAnimation::create();
        CCString * name;
        for(int i = 1; i <= 10; i++) {
            name = CCString::createWithFormat("boom%i.png", i);
            frame = CCSpriteFrameCache::sharedSpriteFrameCache()
            	->spriteFrameByName(name->getCString());
            animation->addSpriteFrame(frame);
        }
```

利用`CCAnimation`创建`CCAnimate`：

```cpp
        animation->setDelayPerUnit(1 / 10.0f);
        animation->setRestoreOriginalFrame(true);
        _groundHit = CCSequence::create(
        	CCMoveBy::create(0, ccp(0, _screenSize.height * 0.12f)),
        	CCAnimate::create(animation),
        	CCCallFuncN::create(this, callfuncN_selector(GameLayer::animationDone)),
        	NULL);
        _groundHit->retain();
```

The same steps are repeated to create the other explosion animation, which is used when the player hits a meteor or a health pack.

```cpp
        animation = CCAnimation::create();
        for(int i = 1; i <= 7; i++) {
            name = CCString::createWithFormat("explosion_small%i.png", i);
            frame = CCSpriteFrameCache::sharedSpriteFrameCache()
            	->spriteFrameByName(name->getCString());
            animation->addSpriteFrame(frame);
        }
        animation->setDelayPerUnit(0.5 / 7.0f);
        animation->setRestoreOriginalFrame(true);
        _explosion = CCSequence::create(
        	CCAnimate::create(animation),
        	CCCallFuncN::create(this, callfuncN_selector
        	(GameLayer::animationDone)),
        	NULL);
        _explosion->retain();
```

如果`setRestoreOriginalFrame`设为`true`，则在动画完成后，精灵将回到初始状态。

In both instances I make a call to an `animationDone` callback already implemented in the class. This makes the calling sprite invisible:

```cpp
	void GameLayer::animationDone (CCNode* pSender) {
		pSender->setVisible(false);
	}
```

### 令游戏运转

We will use a system of countdowns to add new meteors and new health packs, as well as a countdown that will incrementally make the game harder to play.

On touch, the player will start the game, if the game is not running, and also add bombs and explode them during gameplay. An explosion creates a shockwave.

在`update`中，检查`_shockwave`和下落对象的碰撞。Cocos2d-x will take care of all of the rest through our created actions and callbacks!

#### 行动：处理触摸

实现`ccTouchesBegan`方法，显来处理两个状态：进入和游戏结束。

```
    void GameLayer::ccTouchesBegan(CCSet* pTouches, CCEvent* event){
        //if game not running, we are seeing either intro or gameover
        if (!_running) {
        	//if intro, hide intro message
        	if (_introMessage->isVisible()) {
        		_introMessage->setVisible(false);
            //if game over, hide game over message
        	} else if (_gameOverMessage->isVisible()) {
        		SimpleAudioEngine::sharedEngine()->stopAllEffects();
        		_gameOverMessage->setVisible(false);
        	}
        this->resetGame();
        return;
    }
```

接下来处理触摸。这里只需要处理单点，因此调用`->anyObject()`：

```cpp
    CCTouch *touch = (CCTouch *)pTouches->anyObject();
    if (touch) {
        //if bomb already growing...
        if (_bomb->isVisible()) {
            //stop all actions on bomb, halo and sparkle
            _bomb->stopAllActions();
            CCSprite * child;
            child = (CCSprite *) _bomb->getChildByTag(kSpriteHalo);
            child->stopAllActions();
            child = (CCSprite *) _bomb->getChildByTag(kSpriteSparkle);
            child->stopAllActions();
            //if bomb is the right size, then create shockwave
            if (_bomb->getScale() > 0.3f) {
                _shockWave->setScale(0.1f);
                _shockWave->setPosition(_bomb->getPosition());
                _shockWave->setVisible(true);
                _shockWave->runAction(CCScaleTo::create(0.5f,
                    _bomb->getScale() * 2.0f));
                _shockWave->runAction((CCFiniteTimeAction*)_shockwaveSequence->copy()->autorelease());
                SimpleAudioEngine::sharedEngine()->playEffect("bombRelease.wav");
            } else {
            	// 炸弹不够大
                SimpleAudioEngine::sharedEngine()->playEffect("bombFail.wav");
            }
            _bomb->setVisible(false);
            //reset hits with shockwave, so we can count combo hits
            _shockwaveHits = 0;
            //if no bomb currently on screen, create one
        } else {
            CCPoint tap = touch->getLocation();
            _bomb->stopAllActions();
            _bomb->setScale(0.1f);
            _bomb->setPosition(tap);
            _bomb->setVisible(true);
            _bomb->setOpacity(50);
            _bomb->runAction((CCAction *) _growBomb->copy()->autorelease());
            CCSprite * child;
            child = (CCSprite *) _bomb->getChildByTag(kSpriteHalo);
            child->runAction((CCAction *) _rotateSprite->copy()->autorelease());
            child = (CCSprite *) _bomb->getChildByTag(kSpriteSparkle);
            child->runAction((CCAction *) _rotateSprite->copy()->autorelease());
         }
    }
```

#### 行动：开始和重启游戏

```cpp
    void GameLayer::resetGame(void) {
        _score = 0;
        _energy = 100;
        //reset timers and "speeds"
        _meteorInterval = 2.5;
        _meteorTimer = _meteorInterval * 0.99f;
        _meteorSpeed = 10;//in seconds to reach ground
        _healthInterval = 20;
        _healthTimer = 0;
        _healthSpeed = 15;//in seconds to reach ground
        _difficultyInterval = 60;
        _difficultyTimer = 0;
        _running = true;
        //reset labels
        CCString * value = CCString::createWithFormat("%i%s", _energy, "%");
    	_energyDisplay->setString(value->getCString());
    	value = CCString::createWithFormat("%i", _score);
    	_scoreDisplay->setString(value->getCString());
    }

    void GameLayer::stopGame() {
        _running = false;
        //stop all actions currently running
        int count = _fallingObjects->count();
        CCSprite * sprite;
        for (int i = count-1; i >= 0; i--) {
            sprite = (CCSprite *) _fallingObjects->objectAtIndex(i);
            sprite->stopAllActions();
            sprite->setVisible(false);
            _fallingObjects->removeObjectAtIndex(i);
        }
        if (_bomb->isVisible()) {
            _bomb->stopAllActions();
            _bomb->setVisible(false);
            CCSprite * child;
            child = (CCSprite *) _bomb->getChildByTag(kSpriteHalo);
            child->stopAllActions();
            child = (CCSprite *) _bomb
            ->getChildByTag(kSpriteSparkle);
            child->stopAllActions();
        }
        if (_shockWave->isVisible()) {
            _shockWave->stopAllActions();
            _shockWave->setVisible(false);
        }
    }
```

Already implemented in the class is the method that makes the game more difficult as time progresses. If you take a look at the method (`increaseDifficulty`) you will see that it reduces the interval between meteors, and reduces the time it takes for meteors to reach the ground.

#### （及以下未）行动：更新游戏














