> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.swyx.io](https://www.swyx.io/new-mac-setup-2021/)

> I set up a new Mac for work today. Here's everything I use on a Mac for fullstack web development.

I set up a new Mac for work today. Here's everything I use on a Mac for fullstack web development.

Unlike [Mina Markham](https://github.com/minamarkham/formation), I choose not to automate my setup because I only do it about once a year and I want the ability to make conscious changes each time.

> I previously tracked my new mac setup in [an old version of this page from 2018-2020](https://dev.to/swyx/my-new-mac-setup-4ibi).

[1hr Video Walkthrough](#1hr-video-walkthrough)
-----------------------------------------------

* * *

[OS/Browser Settings](#osbrowser-settings)
------------------------------------------

*   **Browser**: Download Chrome, set to default.
*   **Log in to**: (this helps with logins for the other services below)
    *   Twitter
    *   Github (more setup instructions below)
    *   Gmail
*   **System Settings**:
    *   Disable Spotlight search for all miscellaneous crap except apps and system preferences
        *   including [stupid Developer option](https://www.howtogeek.com/231829/how-to-disable-developer-search-results-in-spotlight-on-a-mac/) (make sure to add [Xcode.app](http://xcode.app/) to /Applications not /user/swyx/Applications)
    *   Disable Ask Siri
    *   Set to [Big cursor](https://www.lifewire.com/make-mac-mouse-pointer-bigger-2260808) for accessibility during presentation
    *   Fix trackpad direction: Trackpad -> Scroll & Zoom - Natural off
    *   Disable dictionary lookup: Trackpad -> Point & Click -> Look up & data detectors off
    *   (if using windows keyboard) [remap alt and cmd keys](https://superuser.com/questions/158561/how-can-i-remap-windows-and-alt-keys-in-os-x) for ergonomics
*   **Finder settings**:
    *   Preferences → show filename extensions
    *   Enable showing dotfiles (just hold Cmd + Shift + . (dot) in a Finder window)
    *   [Show path bar](https://www.tekrevue.com/tip/show-path-finder-title-bar/) in footer for easier navigation
    *   Prune the excessive sidebar bookmarks
        *   create "Work" folder and pin it
*   **Keyboard**:
    *   [remap command+Q to literally anything else](https://apple.stackexchange.com/questions/78948/how-to-disable-command-q-for-quit) - to prevent accidental close-all
    *   Shortcuts: copy picture of selected area to clipboard -> Cmd+E
*   **MacOS Dock:**
    *   Remove everything from the Dock except: Finder, System Preferences and Trash
    *   Turn Dock Auto Hiding on
        *   turn this on for MacOS Menu bar as well
*   **Chrome extensions**: (tied to Chrome account)
    *   Paywall blocker [https://github.com/iamadamdev/bypass-paywalls-chrome/](https://github.com/iamadamdev/bypass-paywalls-chrome/)
    *   See Tweets about any page [https://github.com/sw-yx/Twitter-Links-beta](https://github.com/sw-yx/Twitter-Links-beta?organization=sw-yx&organization=sw-yx) ([my blogpost here](https://www.swyx.io/twitter-metacommentary/))
    *   [Morpheon Dark theme](https://chrome.google.com/webstore/detail/morpheon-dark/mafbdhjdkjnoafhfelkjpchpaepjknad?hl=en)
    *   [Lastpass](https://chrome.google.com/webstore/detail/lastpass-free-password-ma/hdokiejnpimakedhajhdlcegeplioahd?hl=en-US)
    *   [Display Anchors](https://chrome.google.com/webstore/detail/display-anchors/poahndpaaanbpbeafbkploiobpiiieko/related?hl=en)
    *   [React Devtools](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi?hl=en)
    *   [Refined Github](https://github.com/sindresorhus/refined-github)
    *   [Code Copy](https://chrome.google.com/webstore/detail/codecopy/fkbfebkcoelajmhanocgppanfoojcdmg?hl=en)
    *   [Video Speed Controller](https://chrome.google.com/webstore/detail/video-speed-controller/nffaoalbilbmmfgbnbgppjihopabppdk?hl=en) ← VERY HIGHLY RECOMMENDED
    *   [Palettab](https://palettab.com/)
    *   [Privacy Badger](https://chrome.google.com/webstore/detail/privacy-badger/pkehgijcmpdhfbdbbnkijodmdjhbjlgp?hl=en-US)
    *   [RescueTime](https://chrome.google.com/webstore/detail/rescuetime-for-chrome-and/bdakmnplckeopfghnlpocafcepegjeap?hl=en-US)
    *   [uBlock Origin](https://chrome.google.com/webstore/detail/ublock-origin/cjpalhdlnbpafiamejdnhcphjbkeiagm?hl=en)
    *   [Octolinker](https://chrome.google.com/webstore/detail/octolinker/jlmafbaeoofdegohdhinkhilhclaklkp?hl=en)
    *   [async render toolbox](https://github.com/sw-yx/async-render-toolbox) (i made this)

[Setup Terminal](#setup-terminal)
---------------------------------

*   Copy my dotfiles (vimrc, zshrc, .gitignore_global): [https://gist.github.com/sw-yx/7fa1009e460ecb818d5e6d9ca4616a05](https://www.notion.so/7fa1009e460ecb818d5e6d9ca4616a05)
    
*   Install [ZSH](https://ohmyz.sh/) (first usage of `git` will prompt you to install git - takes 15 minutes)
    
    *   `git config --global user.name "swyx"`
        
    *   `git config --global user.email shawnthe1@gmail.com`
        
        *   Font - [Inconsolata for Powerline](https://github.com/powerline/fonts/blob/master/Inconsolata/Inconsolata%20for%20Powerline.otf)
            
        *   [autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)
            
        *   [syntax highlighting](https://github.com/zsh-users/zsh-syntax-highlighting)
            
            *   may need to [chmod stuff](https://stackoverflow.com/questions/13762280/zsh-compinit-insecure-directories) or warnings show at start of every session
                
                ```
                $ sudo chmod -R 755 /usr/local/share/zsh
                $ sudo chown -R root:staff /usr/local/share/zsh
                ```
                
*   [Hyper Terminal](https://hyper.is/)
    
    *   settings: [shell: '/bin/zsh'](https://gist.github.com/robertcoopercode/276d7cf66e9b0eea48c117fff1762a17#file-hyper-js-L60)
    *   settings: `fontFamily: '"Inconsolata for Powerline", Menlo, "DejaVu Sans Mono", Consolas, "Lucida Console", monospace',`
    *   [Fig](https://fig.io/) - context-aware autocomplete for terminal. Waitlisted now, but you can skip the waitlist [here](https://fig.io/invite?code=swyx) (i get nothing from this)
        *   More CLI tools recommended by Brendan Faik (founder of Fig) - `bat`, `exa`, `ripgrep`, and other [Rust CLI alternatives](https://zaiste.net/posts/shell-commands-rust/). Also [zsh abbreviations](https://github.com/momo-lab/zsh-abbrev-alias)

[Set up apps/environments](#set-up-appsenvironments)
----------------------------------------------------

*   Install [Homebrew](https://brew.sh/) - `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`
    
    i have a bunch more stuff in `brew list` but i'm not sure what i use actively. You can mass install these: `brew install $(cat packages.txt)`
    
    ```
    bat             gdbm            libuv           python@3.9
    brotli          gh              libyaml         readline
    c-ares          go              mpdecimal       ruby
    deno            gradle          nghttp2         sqlite
    diff-so-fancy   icu4c           node            xz
    fnm             jemalloc        openjdk         yarn
    fzf             libev           openssl@1.1     z
    ```
    
*   `brew install bat`
    
*   [Github CLI](https://github.com/cli/cli): `brew install github/gh/gh`
    
    *   you need to login to git - if you have 2fa enabled, you cant use your normal github password. try pushing to a repo and enter in a [Personal Access Token](https://stackoverflow.com/a/34919582) for password.
    *   then run `gh auth login`
    *   [add GitHub SSH key](https://docs.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent) (not optional)
*   `brew install fzf` - fuzzy finder - usage example is [here](https://twitter.com/swyx/status/1048009961723318272)
    
*   `brew install node`
    
    *   [Node.js/NPM](https://nodejs.org/en/download/)
    *   `npm login`
    *   `sudo npm install netlify-cli -g`
    *   `npm i -g sign-bunny fortune-node parrotsay`
    *   `npm install -g undollar` for removing $
    *   `sudo npm install -g @aws-amplify/cli`
    *   `amplify configure`
    *   `sudo npm install -g trash-cli`
*   `brew install yarn --ignore-dependencies` - [yarn](https://yarnpkg.com/en/docs/install#mac-stable) note
    
*   you may need to [work around Mac OS Sierra](https://stackoverflow.com/questions/43780207/installing-node-with-brew-fails-on-mac-os-sierra)
    
*   `brew install z` - [REALLY GOOD TRY IT](https://brewinstall.org/install-z-on-mac-with-brew/)
    
*   Misc
    
    *   `pip3 install --user powerline-status`
    *   go to a neutral folder and `git clone <https://github.com/powerline/fonts> && cd fonts && ./install.sh`
    *   [fnm](https://github.com/Schniz/fnm) faster alternative to [nvm](https://github.com/creationix/nvm): `curl -fsSL <https://fnm.vercel.app/install> | bash` or `brew install fnm`
    *   **[Anaconda](https://www.anaconda.com/download/#macos) Python distro** - be careful they tend to [modify your bash prompt without asking]([https://askubuntu.com/questions/1026383/why-does-base-appear-in-front-of-my-terminal-prompt\](https://askubuntu.com/questions/1026383/why-does-base-appear-in-front-of-my-terminal-prompt%5C))
    *   [Docker Desktop](https://hub.docker.com/editions/community/docker-ce-desktop-mac/)
    *   `brew install` [ffmpeg](https://ffmpeg.org/download.html) and then
    *   [https://github.com/tombonez/noTunes](https://github.com/tombonez/noTunes)
    *   download [Audacity](https://www.audacityteam.org/download/mac/) - and [install ffmpeg](https://manual.audacityteam.org/man/installing_ffmpeg_for_mac.html)
    *   `brew install java` - you could download [Java Development Kit](https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html) from Oracle but fuck them for putting it behind signup wall
    *   `brew install go` you may need to `export PATH=$PATH:/usr/local/go/bin`
    *   `brew install diff-so-fancy` - then set `git config --global core.pager "diff-so-fancy | less --tabs=4 -RFX"` - makes for much nicer git diff
        *   You can also diff with this bash function `dif() { git diff --color --no-index "$1" "$2" | diff-so-fancy; }` or with VSCode `code --diff file1.js file2.js`.
        *   You can also try [https://github.com/dandavison/delta](https://github.com/dandavison/delta)

[Setup Apps](#setup-apps)
-------------------------

*   Emojis: [https://matthewpalmer.net/rocket/](https://matthewpalmer.net/rocket/)
    
*   Password Manager: I use 1password for company and lastpass for personal
    
*   Window Manager: [https://www.spectacleapp.com/](https://www.spectacleapp.com/) launch at login
    
*   Clipboard Manager: [https://clipy-app.com/](https://clipy-app.com/)
    
*   Loom: [https://www.loom.com/desktop](https://www.loom.com/desktop)
    
*   Zoom: [https://zoom.us/download](https://zoom.us/download)
    
*   Screenshots: [https://cleanshot.com/](https://cleanshot.com/) (previously used [https://zapier.com/zappy](https://zapier.com/zappy))
    
*   Caffeine (Keep Mac awake for talks): [https://intelliscapesolutions.com/apps/caffeine](https://intelliscapesolutions.com/apps/caffeine)
    
    *   used to be [http://lightheadsw.com/caffeine/](http://lightheadsw.com/caffeine/)
    *   maintained version: [Amphetamine](https://roaringapps.com/app/amphetamine) (thanks Matt Mischuk!)
*   [NoTunes](https://github.com/tombonez/noTunes) - disable itunes/apple music
    
*   Video capture: [https://getkap.co/](https://getkap.co/)
    
*   Dual Screen: [https://www.duetdisplay.com/](https://www.duetdisplay.com/)
    
*   Gifs: [Licecap](https://www.cockos.com/licecap/)
    
*   [Slack](https://slack.com/downloads/osx) or [Discord](https://discord.com/)
    
*   OBS: [https://obsproject.com/](https://obsproject.com/)
    
*   Transcribing: [https://www.descript.com/download/mac](https://www.descript.com/download/mac)
    
*   SkyFonts: [https://www.fonts.com/web-fonts/google](https://www.fonts.com/web-fonts/google)
    
*   Microsoft Todo: [https://apps.apple.com/app/apple-store/id1274495053?mt=8](https://apps.apple.com/app/apple-store/id1274495053?mt=8)
    
*   Stretchly: [https://hovancik.net/stretchly/](https://hovancik.net/stretchly/)
    
*   Disk Space: [Disk Inventory X](http://www.derlien.com/) - you can clean node modules with [this bash command](https://twitter.com/swyx/status/1064672618450579457?s=20) or as a [cronjob](https://gist.github.com/zephraph/9169b9de4568b858f4b0e45fc41218b7)
    
*   SimpleNote: [https://apps.apple.com/us/app/simplenote/id692867256?ls=1&mt=12](https://apps.apple.com/us/app/simplenote/id692867256?ls=1&mt=12)
    
*   [Superhuman for Mac](http://superhuman.com/mac) and [https://mail.superhuman.com](https://mail.superhuman.com/)
    
*   Notion: [https://www.notion.so/desktop](https://www.notion.so/desktop)
    
    *   [https://chrome.google.com/webstore/detail/notion-web-clipper/knheggckgoiihginacbkhaalnibhilkk?hl=en](https://chrome.google.com/webstore/detail/notion-web-clipper/knheggckgoiihginacbkhaalnibhilkk?hl=en)
*   App Search/Utils: [https://www.alfredapp.com/](https://www.alfredapp.com/)
    
    *   set to Alfred Dark
    *   [airdrop to iphone/ipad](https://github.com/swmoon203/CrossShare/blob/master/Alfred%20Workflow/Cross%20Share%20Airdrop.alfredworkflow)
    *   [Cupcake Ipsum](http://www.packal.org/workflow/cupcake-ipsum)
*   Editor: Download [VS Code](https://code.visualstudio.com/download) (I used to use Insiders but the popups are super annoying). use Settings Sync to sync across machines
    
    *   have to set up powerline fonts "Meslo LG M for Powerline" ([download](https://github.com/powerline/fonts/blob/master/Meslo%20Slashed/Meslo%20LG%20M%20Regular%20for%20Powerline.ttf))
        
    *   auto-close-tag v0.5.6
        
    *   auto-rename-tag v0.0.15
        
    *   Bookmarks v9.1.0
        
    *   code-settings-sync v3.1.2
        
    *   debugger-for-chrome v4.10.2
        
    *   es7-react-js-snippets v1.8.7
        
    *   graphql-for-vscode v1.12.1
        
    *   mdx v0.1.0
        
    *   prettier-vscode v1.6.1
        
    *   python v2018.9.2
        
    *   python v0.2.3
        
    *   rainbow-brackets v0.0.6
        
    *   shades-of-purple v3.17.0
        
    *   vscode-graphql v0.1.5
        
    *   vscode-import-cost v2.9.0
        
    *   vscode-styled-components v0.0.23
        
    *   vscode-wakatime v1.2.3
        
    *   [TabNine](https://www.tabnine.com/) AI completions
        
    *   [File Utils](https://marketplace.visualstudio.com/items?item>GitHub Copilot</a></p>
                </li>
                <li>
                  <p>to try: <a href=) - recommended by [Stolinski](https://twitter.com/stolinski/status/1417247358430109697)
        
    *   Here's the full list you can run from command line
        
        ```
        code --install-extension 2gua.rainbow-brackets
        code --install-extension ahmadawais.shades-of-purple
        code --install-extension austenc.tailwind-docs
        code --install-extension bradlc.vscode-tailwindcss
        code --install-extension cpylua.language-postcss
        code --install-extension dbaeumer.vscode-eslint
        code --install-extension dsznajder.es7-react-js-snippets
        code --install-extension esbenp.prettier-vscode
        code --install-extension formulahendry.auto-close-tag
        code --install-extension formulahendry.auto-rename-tag
        code --install-extension GabrielNordeborn.vscode-graphiql-explorer
        code --install-extension GitHub.copilot
        code --install-extension golang.go
        code --install-extension heybourn.headwind
        code --install-extension jpoissonnier.vscode-styled-components
        code --install-extension kgscott.retreon
        code --install-extension kumar-harsh.graphql-for-vscode
        code --install-extension luyizhi.vscode-graphql
        code --install-extension ms-python.python
        code --install-extension ms-python.vscode-pylance
        code --install-extension ms-toolsai.jupyter
        code --install-extension msjsdiag.debugger-for-chrome
        code --install-extension NickScialli.svelte-dark
        code --install-extension octref.vetur
        code --install-extension oderwat.indent-rainbow
        code --install-extension sdras.night-owl
        code --install-extension silvenon.mdx
        code --install-extension svelte.svelte-vscode
        code --install-extension TabNine.tabnine-vscode
        code --install-extension ThreadHeap.serverless-ide-vscode
        code --install-extension tht13.python
        code --install-extension WakaTime.vscode-wakatime
        code --install-extension Wattenberger.footsteps
        code --install-extension wix.vscode-import-cost
        ```
        

* * *

[Other good "new laptop setup" lists:](#other-good-new-laptop-setup-lists)
--------------------------------------------------------------------------

*   [Tania Rascia's setup](https://www.taniarascia.com/setting-up-a-brand-new-mac-for-development/?ck_subscriber_id=591519942)
*   [Nick Nisi's dotfiles](https://github.com/nicknisi/dotfiles)
*   [Mathias Bynens macos defaults](https://github.com/mathiasbynens/dotfiles/blob/master/.macos)
*   [Jamon's MacOS maintenance tips](https://twitter.com/jamonholmgren/status/1175077165848653824)
*   You can [automate dotfiles/homebrew setup with Sheldon Hull's tool](https://www.sheldonhull.com/docs/shell/#installing-brew-packages)
*   Physical equipment setups from prominent people: [https://setups.co/](https://setups.co/)
*   please send me yours!