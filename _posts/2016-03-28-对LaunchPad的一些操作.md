本文部分是转帖,来自[这里](http://sspai.com/33299)


调整LaunchPad图表大小(行列数) 

```sh
defaults write com.apple.dock springboard-columns -int 列数
defaults write com.apple.dock springboard-rows -int 行数
defaults write com.apple.dock ResetLaunchPad -bool TRUE
killall Dock
``` 

![](http://7xqjx7.com1.z0.glb.clouddn.com/image/Screen%20Shot%202016-03-28%20at%2019.47.02.png?imageView2/2/h/600)

附赠调整透明度 

```sh
defaults write com.apple.dock springboard-blur-radius -int 模糊度;
killall Dock
模糊度为0~255
```

重申一下,如果参加OSX内测遇到图标残留的情况 

```sh
defaults write com.apple.dock ResetLaunchPad -bool true
killall Dock
```
