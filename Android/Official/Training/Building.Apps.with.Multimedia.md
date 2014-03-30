# （未）多媒体

内容：

* 管理音频播放
* 管理音频焦点
* 处理音频输出硬件

## 管理音频播放

确保不要多个App同时播放音乐。

### 控制App的音量和Playback

* 让用户能控制音量。
* 让播放控制按键（硬件），如播放、暂停、下一首等，能控制播放

#### 识别使用哪个Audio Stream

Android为以下内容提供各自的音频流：播放音乐、alarms、通知、来电铃声、系统声音、in-call volume, and DTMF tones. 这样做的目的是允许用户独立的控制每个流的音量。

Most of these streams are restricted to system events, so unless your app is a replacement alarm clock, 你几乎总是使用[STREAM_MUSIC](http://developer.android.com/reference/android/media/AudioManager.html#STREAM_MUSIC)流播放音乐。

#### 利用硬件声音键控制你的App的音量

默认，按下音量控制修改当前活动的（active）音频流的音量。如果你的App目前未播放任何东西，按下音量键调整的是铃声音量。

在如果在游戏或音乐App中，按下音量键最好控制的是游戏或音乐音量，即使当前未播放。

为此你必须海沧是监听音量控制就，修改你的音频流的音量。Android提供`setVolumeControlStream()`，将音量键定向到你指定的音频流。

尽量早的调用该方法，一般在活动的`onCreate()`方法中。

	setVolumeControlStream(AudioManager.STREAM_MUSIC);

从此刻起，按下设备的音量键，影响的只是你指定的音频流，只要你的活动或Fragment可见。

#### 使用播放控制键（硬件）

一些设备或耳机上有playback按钮，如播放、暂停、停止、跳过、上一个等。当用户按下这样的键后，系统广播一个Intent，action是`ACTION_MEDIA_BUTTON`。

	<receiver android:name=".RemoteControlReceiver">
	    <intent-filter>
	        <action android:name="android.intent.action.MEDIA_BUTTON" />
	    </intent-filter>
	</receiver>

接收者要判断按下的是哪个键，用Intent的`EXTRA_KEY_EVENT`键可以获取该信息。`KeyEvent`类定义了一组`KEYCODE_MEDIA_*`常量，表示可能的媒体按钮，如`KEYCODE_MEDIA_PLAY_PAUSE`和`KEYCODE_MEDIA_NEXT`。

	public class RemoteControlReceiver extends BroadcastReceiver {
	    @Override
	    public void onReceive(Context context, Intent intent) {
	        if (Intent.ACTION_MEDIA_BUTTON.equals(intent.getAction())) {
	            KeyEvent event = (KeyEvent)intent.getParcelableExtra(Intent.EXTRA_KEY_EVENT);
	            if (KeyEvent.KEYCODE_MEDIA_PLAY == event.getKeyCode()) {
	                // Handle key press.
	            }
	        }
	    }
	}

Because multiple applications might want to listen for media button presses, you must also programmatically control when your app should receive media button press events.

The following code can be used within your app to register and de-register your media button event receiver using the AudioManager. When registered, your broadcast receiver is the exclusive receiver of all media button broadcasts.

	AudioManager am = mContext.getSystemService(Context.AUDIO_SERVICE);
	...
	
	// Start listening for button presses
	am.registerMediaButtonEventReceiver(RemoteControlReceiver);
	...
	
	// Stop listening for button presses
	am.unregisterMediaButtonEventReceiver(RemoteControlReceiver);

Typically, apps should unregister most of their receivers whenever they become inactive or invisible (such as during the onStop() callback). However, it’s not that simple for media playback apps—in fact, responding to media playback buttons is most important when your application isn’t visible and therefore can’t be controlled by the on-screen UI.

更好的注册/解除注册的时机是，当你的应用获取和失去*audio focus*。见下节。

### 管理音频焦点

潜在的，可能有多个App同时播放。为避免多个音乐App同时播放，Android使用音频焦点控制，只有获取音频焦点的App才能播放。

Before your app starts playing audio it should request—and receive—the audio focus. 它还要能监听音频焦点的丢失。

#### 请求音频焦点

Before your app starts playing any audio, it should hold the audio focus for the stream it will be using. This is done with a call to `requestAudioFocus()` which returns `AUDIOFOCUS_REQUEST_GRANTED` if your request is successful.

You must specify which stream you're using and whether you expect to require *transient* or *permanent* audio focus. Request transient focus when you expect to play audio for only a short time (for example when playing navigation instructions). Request permanent audio focus when you plan to play audio for the foreseeable future (for example, when playing music).

The following snippet requests permanent audio focus on the music audio stream. You should request the audio focus immediately before you begin playback, such as when the user presses play or the background music for the next game level begins.

	AudioManager am = mContext.getSystemService(Context.AUDIO_SERVICE);
	...
	
	// Request audio focus for playback
	int result = am.requestAudioFocus(afChangeListener,
	                                 // Use the music stream.
	                                 AudioManager.STREAM_MUSIC,
	                                 // Request permanent focus.
	                                 AudioManager.AUDIOFOCUS_GAIN);
	   
	if (result == AudioManager.AUDIOFOCUS_REQUEST_GRANTED) {
	    am.registerMediaButtonEventReceiver(RemoteControlReceiver);
	    // Start playback.
	}

Once you've finished playback be sure to call `abandonAudioFocus()`. This notifies the system that you no longer require focus and unregisters the associated `AudioManager.OnAudioFocusChangeListener`. In the case of abandoning transient focus, 使得被打断的App能继续播放。

	// Abandon audio focus when playback complete    
	am.abandonAudioFocus(afChangeListener);

When requesting transient audio focus you have an additional option: whether or not you want to enable "ducking." Normally, when a well-behaved audio app loses audio focus it immediately silences its playback. By requesting a transient audio focus that allows ducking you tell other audio apps that it’s acceptable for them to keep playing, provided they lower their volume until the focus returns to them.

	// Request audio focus for playback
	int result = am.requestAudioFocus(afChangeListener,
	                             // Use the music stream.
	                             AudioManager.STREAM_MUSIC,
	                             // Request permanent focus.
	                             	AudioManager.AUDIOFOCUS_GAIN_TRANSIENT_MAY_DUCK);
	   
	if (result == AudioManager.AUDIOFOCUS_REQUEST_GRANTED) {
	    // Start playback.
	}

Ducking is particularly suitable for apps that use the audio stream intermittently, such as for audible driving directions.

Whenever another app requests audio focus as described above, its choice between permanent and transient (with or without support for ducking) audio focus is received by the listener you registered when requesting focus.

#### 处理焦点丢失

If your app can request audio focus, it follows that it will in turn lose that focus when another app requests it. How your app responds to a loss of audio focus depends on the manner of that loss.

The `onAudioFocusChange()` callback method of the audio focus change listener you registered when request	ing audio focus receives a parameter that describes the focus change event. Specifically, the possible focus loss events mirror the focus request types from the previous section—permanent loss, transient loss, and transient with ducking permitted.

Generally speaking, a transient (temporary) loss of audio focus should result in your app silencing it’s audio stream, but otherwise maintaining the same state. You should continue to monitor changes in audio focus and be prepared to resume playback where it was paused once you’ve regained the focus.

If the audio focus loss is permanent, it’s assumed that another application is now being used to listen to audio and your app should effectively end itself. In practical terms, that means stopping playback, removing media button listeners—allowing the new audio player to exclusively handle those events—and abandoning your audio focus. At that point, you would expect a user action (pressing play in your app) to be required before you resume playing audio.

In the following code snippet, we pause the playback or our media player object if the audio loss is transient and resume it when we have regained the focus. If the loss is permanent, it unregisters our media button event receiver and stops monitoring audio focus changes.

	OnAudioFocusChangeListener afChangeListener = new OnAudioFocusChangeListener() {
	    public void onAudioFocusChange(int focusChange) {
	        if (focusChange == AUDIOFOCUS_LOSS_TRANSIENT
	            // Pause playback
	        } else if (focusChange == AudioManager.AUDIOFOCUS_GAIN) {
	            // Resume playback 
	        } else if (focusChange == AudioManager.AUDIOFOCUS_LOSS) {
	            am.unregisterMediaButtonEventReceiver(RemoteControlReceiver);
	            am.abandonAudioFocus(afChangeListener);
	            // Stop playback
	        }
	    }
	};

In the case of a transient loss of audio focus where ducking is permitted, rather than pausing playback, you can "duck" instead.

#### Duck!

Ducking is the process of lowering your audio stream output volume to make transient audio from another app easier to hear without totally disrupting the audio from your own application.

In the following code snippet lowers the volume on our media player object when we temporarily lose focus, then returns it to its previous level when we regain focus.

	OnAudioFocusChangeListener afChangeListener = new OnAudioFocusChangeListener() {
	    public void onAudioFocusChange(int focusChange) {
	        if (focusChange == AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK) {
	            // Lower the volume
	        } else if (focusChange == AudioManager.AUDIOFOCUS_GAIN) {
	            // Raise it back to normal
	        }
	    }
	};

A loss of audio focus is the most important broadcast to react to, but not the only one. The system broadcasts a number of intents to alert you to changes in user’s audio experience. The next lesson demonstrates how to monitor them to improve the user’s overall experience.

### （未）处理音频输出硬件
