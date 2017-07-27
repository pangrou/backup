---
title: ios之滑动条简单游戏
date: 2016-04-29 15:21:38
tags:
---
<!--more-->
一个简单小游戏，但是并不主流，不推荐学习

### 拖动滑动条，会弹出分数的提示框
```cpp
#import "ViewController.h"
#import "AboutViewController.h"
//播放音频视频
#import <AVFoundation/AVFoundation.h>

@interface ViewController ()

@property(nonatomic,strong)AVAudioPlayer *audioPlayer;

@end

@implementation ViewController

@synthesize Scores;
@synthesize gameTimes;
@synthesize Nums;
@synthesize slider;
@synthesize audioPlayer;

int score,currentValue,targetValue,currentTimes;
bool musicPlay = true;

- (IBAction)musicSlider:(UIButton *)sender{
    if(musicPlay) {
        musicPlay = false;
        [self playBackgroundMusic];
    }else {
        musicPlay = true;
        [audioPlayer stop];
    }
}

//播放背景音乐
- (void)playBackgroundMusic{
    NSString *musicPath = [[NSBundle mainBundle]pathForResource:@"no" ofType:@"mp3"];
    NSURL *url = [NSURL fileURLWithPath:musicPath];
    NSError *error;
    audioPlayer = [[AVAudioPlayer alloc]initWithContentsOfURL:url error:&error];
    audioPlayer.numberOfLoops = -1;
    if(audioPlayer == nil) {
        NSString *errorInfo = [NSString stringWithFormat:[error description]];
        NSLog(@"the error is :%d",errorInfo);
    }else {
        [audioPlayer play];
    }
}

- (IBAction)SliderMoved:(UISlider *)sender {
    currentValue = lrintf(sender.value);
}

- (IBAction)ShowScores:(id)sender {
    int currentScore = 100 - abs(currentValue - targetValue);
    NSString *msg = [NSString stringWithFormat:@"滑动条所在位置:%d,分数:%d",currentValue,currentScore];
    [[[UIAlertView alloc] initWithTitle:@"test" message:msg delegate:self cancelButtonTitle:@"好的，我知道了！" otherButtonTitles:nil, nil] show];
    score += currentScore;
}

-(void)alertView:(UIAlertView *)alertView didDismissWithButtonIndex:(NSInteger)buttonIndex{
    [self funcShow];
}

- (IBAction)startOver:(id)sender {
    //添加过渡效果
    CATransition *transition = [CATransition animation];
    transition.type = kCATransitionFade;
    transition.duration = 3;
    transition.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut];
    
    score = 0;
    currentTimes = 0;
    [self funcShow];
}

- (void)funcShow{
    [self sliderStart];
    [self randomNum];
    [self ScoreText];
    [self timesShow];
}

- (void)randomNum{
    targetValue = 1 + (arc4random()%100);
    Nums.text = [NSString stringWithFormat:@"%d",targetValue];
}

- (void)ScoreText{
    Scores.text = [NSString stringWithFormat:@"%d",score];
}

- (void)timesShow{
    currentTimes++;
    gameTimes.text = [NSString stringWithFormat:@"%d",currentTimes];
}

- (void)sliderStart{
    currentValue = 50;
    slider.value = currentValue;
}

- (void)viewDidLoad {
    [super viewDidLoad];
//    UIImage *thumbImageNormal = [UIImage imageNamed:@"SliderThumb-Normal"];
    [self funcShow];
}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}

- (IBAction)showInfo:(id)sender {
    AboutViewController *controller = [[AboutViewController alloc] initWithNibName:@"AboutViewController" bundle:nil];
    controller.modalTransitionStyle = UIModalTransitionStyleFlipHorizontal;
    [self presentViewController:controller animated:YES completion:nil];
    
    //主屏幕分辨率
    NSLog(@"x:%f,y:%f,height:%f,width:%f",[UIScreen mainScreen].bounds.origin.x,[UIScreen mainScreen].bounds.origin.y,[UIScreen mainScreen].bounds.size.height,[UIScreen mainScreen].bounds.size.width);
}
@end

```
关闭提示界面，新建一个类；
```cpp
- (IBAction)closeInfo:(id)sender {
    [self.presentingViewController dismissViewControllerAnimated:YES completion:nil];
}
```
