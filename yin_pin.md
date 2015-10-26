# 音频

## 录音 `AVFoundation.h`

- 可以使用懒加载

![](/images/Snip20150815_51.png)

- 代码

```objc
- (IBAction)startRecord:(id)sender {
    // 录音文件保存地址
    NSURL *url = [NSURL URLWithString:@"file:///Users/song/Desktop/test.wav"];
    // 创建录音文件
    AVAudioRecorder *record = [[AVAudioRecorder alloc] initWithURL:url settings:nil error:nil];
    // 准备录音
    [record prepareToRecord];
    // 开始录音
    [record record];
    self.record = record;
}
- (IBAction)stopRecord:(id)sender {
    // 停止录音
    [self.record stop];
}
```
- 设置录制音频的质量

```
// 创建录音配置信息的字典
NSMutableDictionary *setting = [NSMutableDictionary dictionary];
// 音频格式
setting[AVFormatIDKey] = @(kAudioFormatAppleIMA4);
// 录音采样率(Hz) 如：AVSampleRateKey==8000/44100/96000（影响音频的质量）
setting[AVSampleRateKey] = @(8000.0);
// 音频通道数 1 或 2
setting[AVNumberOfChannelsKey] = @(1);
// 线性音频的位深度  8、16、24、32
setting[AVLinearPCMBitDepthKey] = @(8);
//录音的质量
setting[AVEncoderAudioQualityKey] = [NSNumber numberWithInt:AVAudioQualityHigh];
```

## 播放音效

- 可以使用懒加载

![](/images/Snip20150815_54.png)

- 通过字典实现多个文件的加载

![](/images/Snip20150815_55.png)

- 封装成工具类

```objc
//
//  SLQMusic.m
//  播放音乐
//
//  Created by Christian on 15/8/17.
//  Copyright (c) 2015年 slq. All rights reserved.
//

#import "SLQMusic.h"
#import <AVFoundation/AVFoundation.h>

@implementation SLQMusic

/**
 *	 静态全局变量
 */
static NSMutableDictionary *_players;

+ (void)initialize
{
    _players = [NSMutableDictionary dictionary];
}
/**
 *	通过名称播放歌曲
 */
+ (AVAudioPlayer *)playWithMusicName:(NSString *)musicName
{
    if (musicName.length <= 0) {
        return nil;
    }
    // 定义音频播放器
    AVAudioPlayer *player = nil;
    // 2.从字典中取player,如果取出出来是空,则对应创建对应的播放器
    player = _players[musicName];
    if (player == nil) {

        NSURL *url = [[NSBundle mainBundle] URLForResource:musicName withExtension:nil];
        if (url == nil) {
            return nil;
        }
        player = [[AVAudioPlayer alloc] initWithContentsOfURL:url error:nil];
        [player prepareToPlay];
        // 将player存入字典
        [_players setObject:player forKey:musicName];
    }

    // 3、播放音乐
    [player play];
    return player;
}

/**
 *	暂停播放
 */
+ (void)pauseWithMusicName:(NSString *)musicName
{
    if (musicName.length <= 0) {
        return;
    }
    // 定义音频播放器
    AVAudioPlayer *player = nil;
    // 2.从字典中取player,如果取出出来是空,就退出
    player = _players[musicName];
    if (player) {
        // 3、暂停播放音乐
        [player pause];
    }
}
/**
 *	停止播放
 */
+ (void)stopWithMusicName:(NSString *)musicName
{
    if (musicName.length <= 0) {
        return;
    }
    // 定义音频播放器
    AVAudioPlayer *player = nil;
    // 2.从字典中取player,如果取出出来是空,就退出
    player = _players[musicName];
    if (player) {
        // 3、暂停播放音乐
        [player stop];
        // 从字典中移除
        [_players removeObjectForKey:musicName];
        player = nil;
    }
}
@end
```

## 播放音乐

- `AVAudioPlayer`

![](/images/Snip20150815_57.png)

- 默认每一个url对应一个`AVAudioPlayer`对象，所以可以通过字典比较好（工具类封装）

## 毛玻璃效果实现

![](/images/Snip20150815_58.png)

- UIToolBar实现代码

```objc
/**
 *	设置背景图片毛玻璃效果
 */
- (void)setupBackgroundBlur
{
    // 使用UIToolBar实现
    UIToolbar *tool = [[UIToolbar alloc] init];
    [tool setBarStyle:UIBarStyleBlack];
    [self.largeImageView addSubview:tool];
    // 使用屏幕宽高计算frame
    tool.frame = CGRectMake(0, 0, [UIScreen mainScreen].bounds.size.width, [UIScreen mainScreen].bounds.size.height);
}
```
## QQ音乐
- 主界面搭建
- storyboard 添加手势

![](/images/storyboard添加手势.gif)

- `translatesAutoresizingMaskIntoConstraints`设置

	- 如果是从代码层面开始使用Autolayout,需要对使用的View的translatesAutoresizingMaskIntoConstraints的属性设置为NO.
	- 即可开始通过代码添加Constraint,否则View还是会按照以往的autoresizingMask进行计算.而在Interface Builder中勾选了Ues Autolayout,IB生成的控件的translatesAutoresizingMaskIntoConstraints属性都会被默认设置NO.


- 后台播放

![](/images/Snip20150819_9_1.png)

![](/images/屏幕快照 2015-08-17 15.37.26.png)

```objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // 后台播放音频设置
    AVAudioSession *session  = [AVAudioSession sharedInstance];

    // 设置播放类别
    [session setCategory:AVAudioSessionCategoryPlayback error:nil];

    // 激活会话
    [session setActive:YES error:nil];

    return YES;
}
```

- `MONowPlayingInfoCenter`

![](/images/Snip20150817_1.png)
![](/images/Snip20150817_2.png)

- 锁屏事件中心

```objc
- (void)setLockImage:(UIImage *)image
{
    SLQMusicItem *musicItem = [SLQMusicTool playingMusic];

    // 2、获取锁屏界面中心
    MPNowPlayingInfoCenter *lockCenter  = [MPNowPlayingInfoCenter defaultCenter];

    NSMutableDictionary *dict = [NSMutableDictionary dictionary];

    [dict setObject:musicItem.name forKey:MPMediaItemPropertyAlbumTitle];
    [dict setObject:musicItem.singer forKey:MPMediaItemPropertyAlbumArtist];
    MPMediaItemArtwork *artWork = [[MPMediaItemArtwork alloc] initWithImage:image];
    [dict setObject:artWork forKey:MPMediaItemPropertyArtwork];
    [dict setObject:@(self.duration) forKey:MPMediaItemPropertyPlaybackDuration];
    [dict setObject:@(self.currentTime) forKey:MPNowPlayingInfoPropertyElapsedPlaybackTime];

    lockCenter.nowPlayingInfo = dict;

    // 开始接受远程事件
    [[UIApplication sharedApplication] beginReceivingRemoteControlEvents];
}

/**
 *	接受远程事件
 */
- (void)remoteControlReceivedWithEvent:(UIEvent *)event
{
    switch (event.subtype ) {
        case UIEventSubtypeRemoteControlPlay:
        case UIEventSubtypeRemoteControlPause:
            [self playOrPause];
            break;
        case UIEventSubtypeRemoteControlNextTrack:
            [self nextMusic];
            break;
        case UIEventSubtypeRemoteControlPreviousTrack:
            [self previousMusic];
            break;
        default:
            break;
    }
}
```


- 监听事件

![](/images/Snip20150817_3.png)

- 锁屏歌词（图片）

## AVPlayer（远程音乐/视频）

- 可以播放网络音频或者视频，当然本地的也不在话下

- 播放远程音频

```objc
#pragma mark - 懒加载
- (AVPlayer *)player
{
    if (!_player) {
        // 初始化
        NSURL *url = [NSURL URLWithString:@"http://cc.stream.qqmusic.qq.com/C100003j8IiV1X8Oaw.m4a?fromtag=52"];
       _player = [AVPlayer playerWithURL:url];
    }
    return  _player;
}
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
    // 播放
    [self.player play];
}
```
- 播放远程视频
    - 播放视频必须添加layer，不然只有声音没有画面
```objc
#pragma mark - 懒加载
- (AVPlayer *)player
{
    if (!_player) {
        NSURL *url = [NSURL URLWithString:@"http://v1.mukewang.com/19954d8f-e2c2-4c0a-b8c1-a4c826b5ca8b/L.mp4"];
        _player = [AVPlayer playerWithURL:url ];
    }
    return  _player;
}
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
    [self.player play];
}
```
- 指定层才能正常显示，一般iOS中视频显示比例是9：16

```objc
#pragma mark - 懒加载
- (AVPlayer *)player
{
    if (!_player) {
        NSURL *url = [NSURL URLWithString:@"http://v1.mukewang.com/19954d8f-e2c2-4c0a-b8c1-a4c826b5ca8b/L.mp4"];
//        _player = [AVPlayer playerWithURL:url ];
        // 使用item方法的话，以后可以随时更改url
        AVPlayerItem *item = [AVPlayerItem playerItemWithURL:url];
        _player = [AVPlayer playerWithPlayerItem:item];
        // 添加播放层，这样才能显示出来
        AVPlayerLayer *layer = [AVPlayerLayer playerLayerWithPlayer:_player];
        // 一把iOS中视频比例是9：16
        layer.frame = CGRectMake(0, 0, self.view.bounds.size.width, self.view.bounds.size.width * 9 / 16);

        [self.view.layer addSublayer:layer];
    }
    return  _player;
}
```
- 自己封装AVPlayer

```objc
#import "VideoPlayView.h"

@interface VideoPlayView()

/* 播放器 */
@property (nonatomic, strong) AVPlayer *player;

// 播放器的Layer
@property (weak, nonatomic) AVPlayerLayer *playerLayer;

@property (weak, nonatomic) IBOutlet UIImageView *imageView;
@property (weak, nonatomic) IBOutlet UIView *toolView;
@property (weak, nonatomic) IBOutlet UIButton *playOrPauseBtn;
@property (weak, nonatomic) IBOutlet UISlider *progressSlider;
@property (weak, nonatomic) IBOutlet UILabel *timeLabel;

// 记录当前是否显示了工具栏
@property (assign, nonatomic) BOOL isShowToolView;

/* 定时器 */
@property (nonatomic, strong) NSTimer *progressTimer;

#pragma mark - 监听事件的处理
- (IBAction)playOrPause:(UIButton *)sender;
- (IBAction)switchOrientation:(UIButton *)sender;
- (IBAction)slider;
- (IBAction)startSlider;
- (IBAction)tapAction:(UITapGestureRecognizer *)sender;
- (IBAction)sliderValueChange;

@end

@implementation VideoPlayView

// 快速创建View的方法
+ (instancetype)videoPlayView
{
    return [[[NSBundle mainBundle] loadNibNamed:@"VideoPlayView" owner:nil options:nil] firstObject];
}

- (void)awakeFromNib
{
    self.player = [[AVPlayer alloc] init];
    self.playerLayer = [AVPlayerLayer playerLayerWithPlayer:self.player];
    [self.imageView.layer addSublayer:self.playerLayer];

    self.toolView.alpha = 0;
    self.isShowToolView = NO;

    [self.progressSlider setThumbImage:[UIImage imageNamed:@"thumbImage"] forState:UIControlStateNormal];
    [self.progressSlider setMaximumTrackImage:[UIImage imageNamed:@"MaximumTrackImage"] forState:UIControlStateNormal];
    [self.progressSlider setMinimumTrackImage:[UIImage imageNamed:@"MinimumTrackImage"] forState:UIControlStateNormal];

    [self removeProgressTimer];
    [self addProgressTimer];

    self.playOrPauseBtn.selected = YES;
}

- (void)layoutSubviews
{
    [super layoutSubviews];

    self.playerLayer.frame = self.bounds;
}

#pragma mark - 设置播放的视频
- (void)setPlayerItem:(AVPlayerItem *)playerItem
{
    _playerItem = playerItem;
    [self.player replaceCurrentItemWithPlayerItem:playerItem];
    [self.player play];
}

// 是否显示工具的View
- (IBAction)tapAction:(UITapGestureRecognizer *)sender {
    [UIView animateWithDuration:0.5 animations:^{
        if (self.isShowToolView) {
            self.toolView.alpha = 0;
            self.isShowToolView = NO;
        } else {
            self.toolView.alpha = 1;
            self.isShowToolView = YES;
        }
    }];
}
// 暂停按钮的监听
- (IBAction)playOrPause:(UIButton *)sender {
    sender.selected = !sender.selected;
    if (sender.selected) {
        [self.player play];

        [self addProgressTimer];
    } else {
        [self.player pause];

        [self removeProgressTimer];
    }
}

#pragma mark - 定时器操作
- (void)addProgressTimer
{
    self.progressTimer = [NSTimer scheduledTimerWithTimeInterval:1.0 target:self selector:@selector(updateProgressInfo) userInfo:nil repeats:YES];
    [[NSRunLoop mainRunLoop] addTimer:self.progressTimer forMode:NSRunLoopCommonModes];
}

- (void)removeProgressTimer
{
    [self.progressTimer invalidate];
    self.progressTimer = nil;
}

- (void)updateProgressInfo
{
    // 1.更新时间
    self.timeLabel.text = [self timeString];

    // 2.设置进度条的value
    self.progressSlider.value = CMTimeGetSeconds(self.player.currentTime) / CMTimeGetSeconds(self.player.currentItem.duration);
}

- (NSString *)timeString
{
    NSTimeInterval duration = CMTimeGetSeconds(self.player.currentItem.duration);
    NSTimeInterval currentTime = CMTimeGetSeconds(self.player.currentTime);

    return [self stringWithCurrentTime:currentTime duration:duration];
}

// 切换屏幕的方向
- (IBAction)switchOrientation:(UIButton *)sender {
    sender.selected = !sender.selected;
    if ([self.delegate respondsToSelector:@selector(videoplayViewSwitchOrientation:)]) {
        [self.delegate videoplayViewSwitchOrientation:sender.selected];
    }
}

- (IBAction)slider {
    [self addProgressTimer];
    NSTimeInterval currentTime = CMTimeGetSeconds(self.player.currentItem.duration) * self.progressSlider.value;

    // 设置当前播放时间
    [self.player seekToTime:CMTimeMakeWithSeconds(currentTime, NSEC_PER_SEC) toleranceBefore:kCMTimeZero toleranceAfter:kCMTimeZero];

    [self.player play];
}

- (IBAction)startSlider {
    [self removeProgressTimer];
}

- (IBAction)sliderValueChange {
    NSTimeInterval currentTime = CMTimeGetSeconds(self.player.currentItem.duration) * self.progressSlider.value;
    NSTimeInterval duration = CMTimeGetSeconds(self.player.currentItem.duration);
    self.timeLabel.text = [self stringWithCurrentTime:currentTime duration:duration];
}

- (NSString *)stringWithCurrentTime:(NSTimeInterval)currentTime duration:(NSTimeInterval)duration
{
    NSInteger dMin = duration / 60;
    NSInteger dSec = (NSInteger)duration % 60;

    NSInteger cMin = currentTime / 60;
    NSInteger cSec = (NSInteger)currentTime % 60;

    NSString *durationString = [NSString stringWithFormat:@"%02ld:%02ld", dMin, dSec];
    NSString *currentString = [NSString stringWithFormat:@"%02ld:%02ld", cMin, cSec];

    return [NSString stringWithFormat:@"%@/%@", currentString, durationString];
}

@end
```


- 多个文件一个`playWithPlayerItem:`
![](/images/Snip20150817_4.png)

- `MPMoviePlayerViewController`控制器

```objc
#import <MediaPlayer/MediaPlayer.h>

#pragma mark - 懒加载代码
- (MPMoviePlayerViewController *)playerVc
{
    if (_playerVc == nil) {
        NSURL *url = [NSURL URLWithString:@"http://v1.mukewang.com/19954d8f-e2c2-4c0a-b8c1-a4c826b5ca8b/L.mp4"];

        _playerVc = [[MPMoviePlayerViewController alloc] initWithContentURL:url];
    }
    return _playerVc;
}

- (IBAction)play {
    [self presentViewController:self.playerVc animated:YES completion:nil];
}
```
![](/images/MPMoviePlayerViewController使用.gif)

- `MPMoviePlayerController`控制器

```objc
- (MPMoviePlayerController *)player
{
    if (_player == nil) {
        NSURL *url = [NSURL URLWithString:@"http://v1.mukewang.com/19954d8f-e2c2-4c0a-b8c1-a4c826b5ca8b/L.mp4"];

        _player = [[MPMoviePlayerController alloc] initWithContentURL:url];

        // 设置控制器View所在的位置
        _player.view.frame = CGRectMake(0, 0, self.view.bounds.size.width, self.view.bounds.size.width * 9 / 16);

        // 设置播放器的控制模式
        _player.controlStyle = MPMovieControlStyleFullscreen;

        [self.view addSubview:self.player.view];
    }
    return _player;
}

- (IBAction)play {
    [self.player play];
}
```

![](/images/MPMoviePlayerController的使用.gif)
## 总结
### 一、音频播放
- 1.音效播放（短时间的音频文件）
	- 1> `AudioServicesCreateSystemSoundID`
	- 2> `AudioServicesPlaySystemSound`

- 2.音乐 播放（长时间的音频文件）
	- 1> `AVAudioPlayer`
		- 只能播放本地的音频文件
		- >`MPMusicPlayerController`

	- 2> `AVPlayer`
		- 能播放本地、远程的音频、视频文件
		- 基于Layer显示，得自己去编写控制面板

	- 3> `MPMoviePlayerController`
		- 能播放本地、远程的音频、视频文件
		- 自带播放控制面板（暂停、播放、播放进度、是否要全屏）

	- 4> `MPMoviePlayerViewController`
		- 能播放本地、远程的音频、视频文件
		- 内部是封装了`MPMoviePlayerController`
		- 播放界面默认就是全屏的
		- 如果播放功能比较简单，仅仅是简单地播放远程、本地的视频文件，建议用这个

	- 5> `DOUAudioStreamer`
		- 能播放远程、本地的音频文件
		- 监听缓冲进度、下载速度、下载进度

## 二、视频播放
- 1.音乐播放中2> 3> 4>

- 2.VLC

- 3.FFmpeg
	- kxmovie
	- Vitamio

## 三、录音
- 1.AVAudioRecorder






