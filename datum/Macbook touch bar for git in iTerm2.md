![Macbook touch bar for git in iTerm2](https://refruity.xyz/content/images/size/w2000/2022/03/iterm-touch-bar-1.jpg)

I would like to share with you my settings for git in iTerm2.

You can use `git status`, `git add .`, `git commit -m ""`, `git pull`, `git push`, `git switch main`, `git switch master` and `git switch develop` just by clicking one of these cuties.

Configuration is pretty straightforward, but it can take some time to figure out the first time you try it. So here is a little guide.

Open `Profiles` -> `Open profiles` -> Choose a profile -> `Edit profiles` -> `Keys`.

![](https://refruity.xyz/content/images/2022/03/image.png)

Then choose `Add Touch Bar Item`, label it `git status`, choose action `Send text with vim special characters`. Then write `git status\n` in the bottom field.

![](https://refruity.xyz/content/images/2022/03/image-1.png)

Repeat for every other git command you want. For `git commit` write `git commit -m ""\u0002` in the bottom field. This will send the text `git commit -m ""` to the console and then send a left arrow so the cursor is inside the quotes. Then you can write the commit message and press `Enter`.

Then choose `View` -> `Customize Touch Bar...` and drag new commands to the touch bar.

That is all! Hope you find it useful.