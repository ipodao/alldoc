[toc]

## 创建Sprite实例

`Sprite`的子类`GameSprite`。使用图片路径创建精灵的静态方法：

```cpp
	GameSprite* GameSprite::gameSpriteWithFile(const char *pszFileName) {
        GameSprite *sprite = new GameSprite();
        if (sprite && sprite->initWithFile(pszFileName)) {
        	sprite->autorelease();
        	return sprite;
        }
        CC_SAFE_DELETE(sprite);
        return NULL;
    }
```

注意到以下几点：

- 判断父类初始化成功后，再返回。返回前调用`sprite->autorelease();`。
- 如果初始化失败，删除`CC_SAFE_DELETE(sprite);`，返回NULL。

## 游戏循环、定时

`GameLayer`是CCLayer的子类。`GameLayer::update`是它的一个成员。

```cpp
	// GameLayer头中
    void update (float dt); // 普通成员方法，不是父类中的

	// GameLayer的init()方法
	this->schedule(schedule_selector(GameLayer::update));
```

利用dt的累加，手工制造定时器
```cpp
    void GameLayer::update (float dt) {
        if (!_running) return;

        // update timers

        _meteorTimer += dt;
        if (_meteorTimer > _meteorInterval) {
            _meteorTimer = 0;
            this->resetMeteor();
        }
```

## 触摸检测

`player`是一个Sprite。

```cpp
    void GameLayer::ccTouchesBegan(CCSet* pTouches, CCEvent* event) {
        CCTouch* touch;
        CCPoint tap;
        // ...
                tap = touch->getLocation();
                    if (player->boundingBox().containsPoint(tap)) {
        // ...
```

## 辅助工具

### CCString

```cpp
    CCString *score = CCString::createWithFormat("%i", _player1Score);
    _player1ScoreLabel->setString(score->getCString());
```


