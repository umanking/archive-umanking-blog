---
layout: post
title: "[Tip] 개발 환경 셋팅하기"
date: 2019-08-06 09:25:51
categories: [TIP]
tags: [tip]
redirect_from: 
- 2019/08/06/setting-mac/
---
예전에 포맷하고 맥 다시 설치하면서 올렸던 내용인데, 새롭게 정리해서 다시 공유 합니다. 제가 주로 개발할때 쓰는 프로그램 및 플러그인들을 모아봤습니다. 

## 설치용 맥용 프로그램
사용툴
- [JetBrain toolbox](https://www.jetbrains.com/toolbox-app/)
- [VsCode](https://code.visualstudio.com/download)
- [PostMan](https://www.getpostman.com/downloads/)
- [Insomnia](https://insomnia.rest/)
- [SequelPro](https://sequelpro.com/download)

소스관련
- [SourceTree](https://www.sourcetreeapp.com/)
- [Iterms2(Zsh, Oh-my-zsh, dracular theme)](https://www.iterm2.com/downloads.html)

유틸성
- [Alfred4](https://www.alfredapp.com/)
- [Magnet](https://apps.apple.com/us/app/magnet/id441258766?mt=12)
- [Spectacle](https://www.spectacleapp.com/)
- [Karabiner](https://pqrs.org/osx/karabiner/)
- [Jolt of Caffein](https://apps.apple.com/us/app/jolt-of-caffeine/id1437130425?mt=12)
- [Notion](https://www.notion.so/desktop)
- [TextMate](https://macromates.com/download)

## Chrome Plugin
- [JsonFormatter](https://chrome.google.com/webstore/detail/json-formatter/bcjindcccaagfpapjjmafapmmgkkhgoa)
- [Octotree](https://chrome.google.com/webstore/detail/octotree/bkhaagjahfmjljalopjnoealnfndnagc)
- [Momentum](https://chrome.google.com/webstore/detail/momentum/laookkfknpbbblfpciffpaejjkokdgca)
- [EditThisCookie](https://chrome.google.com/webstore/detail/editthiscookie/fngmhnnpilhplaeedifhccceomclgfbg)
- [Google translate](https://chrome.google.com/webstore/detail/google-translate/aapbdbdomjkkjkaonfhkkikfgjllcleb)

## 한/영키 Right Command 키 변환
1. [karabiner 설치](https://pqrs.org/osx/karabiner/) 
2. Simple modifications 탭에서 right_command - f18로 추가 
3. System Preference - Keyboard - 단축키 탭 - 입력소스 - 이전소스 입력을 (right command) 키로 변경 


## github ssh키 등록
- `ssh-keygen -t rsa` 를 통해서 ssh key를 만든다.
- cat ~/.ssh/id_psa.pub 을 통해서 퍼블릭키를 복사한다. 
- github 계정의 SSH and GPG keys 에서 `New SSH Key` 를 등록한다. 


## brew, nodejs 설치
```shell
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
$ brew update
$ brew install node
```

## Zsh 설치
```sh
$ brew install zsh
$ curl -L https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh | sh
$ chsh -s /bin/zsh
```

## vim 설치

```sh
$ brew install neovim
$ brew tap caskroom/fonts
$ brew cask install font-hack-nerd-font
$ curl -sLf https://spacevim.org/install.sh | bash
```

```
테마변경
 ~/.SpaceVim.d/init.vim` 파일에 다음 항목 추가
let g:spacevim_colorscheme = 'onedark'
```

## Intellij Plugin 추천
- [Lombok](https://plugins.jetbrains.com/plugin/6317-lombok)
- [Save Action](https://plugins.jetbrains.com/plugin/7642-save-actions)

## VsCode Plugin 추천
- [Atom one dark theme](https://marketplace.visualstudio.com/items?itemName=akamud.vscode-theme-onedark)
- [Intellij keybinding](https://marketplace.visualstudio.com/items?itemName=k--kato.intellij-idea-keybindings)
- [Auto close tag](https://marketplace.visualstudio.com/items?itemName=formulahendry.auto-close-tag)
- [Auto Import](https://marketplace.visualstudio.com/items?itemName=steoates.autoimport)
- [Beautify](https://marketplace.visualstudio.com/items?itemName=HookyQR.beautify)
- [Bracket Pair Colorizer](https://marketplace.visualstudio.com/items?itemName=CoenraadS.bracket-pair-colorizer)
- [Indenticator](https://marketplace.visualstudio.com/items?itemName=SirTori.indenticator)
- [Output colorizer](https://marketplace.visualstudio.com/items?itemName=IBM.output-colorizer)
- [Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)
- [Material icon theme](https://marketplace.visualstudio.com/items?itemName=PKief.material-icon-theme)
- [Vscode-styld-components](https://marketplace.visualstudio.com/items?itemName=jpoissonnier.vscode-styled-components)
- [todo highlight](https://marketplace.visualstudio.com/items?itemName=wayou.vscode-todo-highlight)
- [Node.js extension pack](https://marketplace.visualstudio.com/items?itemName=waderyan.nodejs-extension-pack)
- [React extension pack](https://marketplace.visualstudio.com/items?itemName=jawandarajbir.react-vscode-extension-pack)



## vscode `code .` 설치

- vs code 실행
- shell command: `install code`  설치
- iterms에서 `code .` 하면 vscode로 열림

