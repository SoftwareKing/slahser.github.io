最近vim-airline有所更新,[spf13-vim](http://vim.spf13.com)内的代码着色插件提示缺失,下面我们手动修复一下.

ps:使用spf13-vim需要with lua的vim依赖,请如下安装:

` brew install vim --with-lua `

因为此前的vim-color-solarized插件换了仓库,而github上作者还没更新,我们需要修改如下文件:

`  vim ~/.vimrc.bundles `

![](http://7xqjx7.com1.z0.glb.clouddn.com/image/Screen%20Shot%202016-02-09%20at%2023.03.18.png?imageView2/2/h/600)

如图修改121,122行.

而后更新spf13-vim

` curl http://j.mp/spf13-vim3 -L -o - | sh `

done.