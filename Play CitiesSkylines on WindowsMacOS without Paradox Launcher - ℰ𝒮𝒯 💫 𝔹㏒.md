Haven't played Cities: Skylines for ages, so I re-installed it on Steam, after some updates there's this new Paradox Launcher which serves no purpose but wasting time. So I decided to remove it, googled some [guide](https://steamcommunity.com/sharedfiles/filedetails/?id=1976349559) and unfortunately it contains too many steps and some error occured while following it. Here's the simpler & correct step of what I did instead.

After the game is downloaded, open the Steam library, find the game, right click, choose `Properties...`

![](https://blog.est.im/images/2023/stdout-03-01.webp)

Next for the , type`Launch Options`

### on Windows

```
cmd /c start cities.exe %command%
```

### on MacOS

```
/Users/$USER/Library/Application\ Support/Steam/steamapps/common/Cities_Skylines/Cities.app/Contents/MacOS/Cities %command%
```

Make sure you replace with your username`$USER`

Then the game works like good o' times, straight to the menu.