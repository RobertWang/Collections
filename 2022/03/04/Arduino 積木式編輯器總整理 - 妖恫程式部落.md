> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.everlearn.tw](https://blog.everlearn.tw/arduino/arduino-%E7%A9%8D%E6%9C%A8%E5%BC%8F%E7%B7%A8%E8%BC%AF%E5%99%A8%E7%B8%BD%E6%95%B4%E7%90%86)

> 在這篇文章中，我將討論選擇 Arduino 積木式編輯器的考量事項，並對市面上眾多的積木式編輯器做一個摘要與整理，以作為選擇時的參考。

前言
--

雖然 Arduino 問世以久，但是身為這波自創浪潮的元老之一，Arduino 仍有相當的重要性。Arduino 官方雖然一直都有提供功能完整的 IDE，然而因為使用 C++ 做為開發語言，因此造成不少人上手時的門檻。尤其對非英語系國家的青少年來說，在學習的路上更是難上加難。也因此，這幾年陸陸續續發展出了許多不同的積木式編輯器，可做為學習時的敲門磚，甚至用來開發一般的專案也不成問題。

在這篇文章中，我將討論選擇編輯器的考量事項，並對市面上眾多的積木式編輯器做一個摘要與整理，以作為選擇時的參考。

考量事項
----

### Scratch 是否並不重要，一切回到需求

在這些眾多的編輯器當中，有不少跟 Scratch 有一定程度的關係，不管是透過外掛的方式、改寫、甚至是受到啟發。不過身為一個使用者，這倒不是我們最需在意的一點。雖然對於學習過 Scratch 的人來說，直接在熟悉的環境或操作方式下操控 Arduino 是很符合直覺的選擇，但是其實這些編輯器大多與 Scratch 有類似的設計方式，不至於需要太多熟悉的轉換時間。反倒是編輯器本身能否滿足我們的需求才是最值得考量的重點。

### 教學 vs 專案製作

既然滿足需求才是選擇編輯器的最主要考量，那我們就來看看需求是甚麼。

如果是以教學或學習為出發點，當然就是盡量以積木功能以及支援周邊元件的數量為主要考量。此外，中文化與否、安裝複雜度、以及穩定性也是很重要的考量項目。對於某些現場教學來說，是否支援離線版可能也是必須考量的重點。

如果是專案製作呢？當然就是先把專案所需的功能與元件完整列出來，然後找尋最為匹配的編輯器。基本上，一般類比、數位腳位的輸出入功能都是必備的積木，所以重點應放在需要特殊處理的周邊元件的支援度。舉例來說，如果專案需要用到 RGB Led 燈泡，那麼直接提供 RGB Led 燈泡控制積木的編輯器就會是比較方便的選擇。當然，RGB Led 燈泡的控制其實不難，直接控制不同腳位的輸出就可以達到同樣的目的，所以實務上不一定要如此考量，往往直接使用最為熟悉的編輯器即可。不過有時候某些硬體需要搭配特定的編輯器，此時我們就沒有太多的選擇。好在這些編輯器的設計方式都大同小異，只要有相關操作經驗，在程式編輯上不會有太多的困難。比較麻煩是安裝步驟差距甚大，甚至很容易卡關。

### 獨立與否很重要

除了周邊元件是否直接支援外，還有一個常常被忽略卻更為重要的考量，那就是是否支援程式燒錄的選項。簡單來說，Arduino 的程式設計可分為兩大分類，一類是操控 Arduino 時必須由兩個程式搭配而成，一個是執行在 Arduino 的特殊韌體程式，另外一個則是執行在電腦上的控制程式。在這種模式下，Arduino 就像一個魁儡一樣，沒有自己的自我意識，完全受控制程式的擺佈。電腦上的控制程式必須持續對 Arduino 下達控制指令才能產生作用，一旦電腦上的控制程式停止後 Arduino 就不會再產生任何反應。我將這種模式稱為魁儡模式，運作方式可參考下圖：

[![](https://blog.everlearn.tw/wp-content/uploads/2018/08/arduino-programming-mode-1.png)](https://blog.everlearn.tw/wp-content/uploads/2018/08/arduino-programming-mode-1.png)

魁儡模式下的 Arduino

問題來了，Arduino 怎麼會自願成為魁儡呢？電影裡的壞人，如果想要控制好人使其言聽計從，常常使用注射聽話藥劑這種方式。在這裡我們可以利用一樣的概念，先將聽話藥劑 (特殊韌體程式) 注射 (燒錄) 到 Arduino 裡。這種特殊韌體程式通常是 Firmata/FirmataPlus，但是也可能是自行開發的特殊韌體。我們修改上圖，加上燒錄的步驟：

![](https://blog.everlearn.tw/wp-content/uploads/2018/08/arduino-programming-mode-1a.png)

魁儡模式下的 Arduino (燒錄特殊韌體)

基本上，以 Scratch 為基礎的編輯器大多是這樣的運作方式。在這種模式下主要的運算由電腦上的控制程式加以執行，因此比較容易完成複雜的功能，但是另一方面卻也使得 Arduino 無法擺脫 USB 線的束縛，而且還必須完全依賴控制程式的指令。使用 WiFi 或藍芽等無線功能雖然可以讓 Arduino 擺脫 USB 線的束縛，但是卻依舊必須受限於控制程式，因此不管在架構上或是應用時都會受到不少限制。

如果要完全擺脫控制程式，則必須將我們寫好的程式直接燒錄至 Arduino。這一類編輯器將積木程式轉成 Arduino IDE 所支援的 C++ 程式語言，然後進行編譯並燒錄至 Arduino。透過這種方式，Arduino 上的韌體程式可以獨立運作，而不需要依賴額外的控制程式。我將這種模式稱為獨立模式，運作方式可參考下圖：

[![](https://blog.everlearn.tw/wp-content/uploads/2018/08/arduino-programming-mode-2.png)](https://blog.everlearn.tw/wp-content/uploads/2018/08/arduino-programming-mode-2.png)

獨立模式下的 Arduino

嚴格來說，魁儡模式與獨立模式都需要進行程式的燒錄，只不過傀儡模式下燒錄的是用來接受控制指令的特殊韌體，而獨立模式下則是燒錄我們所撰寫的程式。

這兩種模式之間並沒有哪種比較優秀的問題，只有合適不合適。也就是說我們必須從專案的整體架構來考量，如果需要或適合搭配控制程式，那就選擇魁儡模式的編輯器。如果需要獨立運作，那就選用獨立模式的編輯器。

最後，有些獨立運作的 Arduino 程式仍可以接受外部來的設定與控制。舉例來說，我們可以利用無線控制的方式來設定前述範例中 LED 燈泡的亮度。儘管如此，LED 燈泡的亮不亮以及所需亮度，仍需由 Arduino 上的程式做出最後決定，因此依舊屬於獨立模式開發方式。

結論
--

甚麼！明明都還沒有講到任何的積木式編輯器，怎麼就直接做結論了？因為可供選擇的編輯器實在太多，為了避免看到文章後面精神不濟，所以我們把結論搬到前面。

綜合來說，[WFduino](https://wf8266.wixsite.com/wfduino) (尤其是新版的 WFduino 2) 與 [motoBlockly](https://www.motoblockly.com/) 是目前兩個最適合使用的積木式編輯器。WFduino 支援 WF8266R，可以無線控制 Arduino。不過 WFduino 僅支援魁儡模式，而使用獨立模式的 motoBlockly 正好可以相互搭配。再加上可直接編輯 C++ 程式，對學習 Arduino 的 C++ 程式來說相當方便。即使遇到現成積木沒有支援的周邊元件，也可以透過 Arduino IDE 進行程式的功能擴充。

此外，Webduino Blockly 則是用來製作物聯網的優先選擇。至於 Transformer 也是可以考慮的選項，不過目前還不知道 Transformer 是否會支援新版的 Scratch 3，而且僅限於社群的應用也是必須考量的要點。想較於許多已經不再更新的編輯器，這幾個編輯器的功能與開發狀況都值得我們優先考慮使用。而且這幾個編輯器剛好都是台灣團隊所開發，中文的支援當然也都不成問題。

盡管積木式編輯器選擇眾多，而且可以支援不同需求的架構，但是積木式編輯器對周邊元件的支援度仍遠低於 Arduino 官方 IDE，所以對於複雜的專案亦可考慮使用 Arduino IDE 搭配其他程式語言一起完成專案的架構。

積木式編輯器摘要
--------

### S4A

可說是最老牌的 Arduino 積木式編輯器，以 Scratch 1.x 版本進行修改，穩定度高但是直接支援的周邊元件數量不算多，除了操作基本腳位外，僅支援馬達的控制。此外，Scratch 1.x 的功能比起 Scratch 2、甚至是 Scratch 3 來說都還是較為缺少些。積木本身沒有中文化，也是美中不足的地方。

### S2A

以 Scratch 2 外掛的方式與 Arduino 互動，從 s2a 演變為 s2a_fm，現在最新改版為 s2aio。除了操作基本腳位外，還支援伺服馬達、音調撥放等功能。介面支援中文，但是安裝過程頗為複雜，需要安裝 Python 以及相關套件，而且必須自行燒錄 Arduino 所需的韌體，對大多數新手來說是一個不小的門檻。

### Transformer

為[宇宙機器人團隊](https://www.kodorobot.com/)所研發的軟體，嚴格來說 Transformer 並不是一個積木式編輯器。但是透過 Transformer，可以直接開啟 S4A 與 S2A 的編輯器，並可自動燒錄 Arduino 所需的韌體，大幅減少安裝以及使用 S4A/S2A 時的複雜度，對教學或學習的人可說是一大福音。不過要注意 Transformer 社群版不可以用在營利目的，使用時必須多加注意。

### ScratchX 外掛

ScratchX 提供各式各樣的 Scratch 2 外掛，當然也包含 Arduino 的操控。除了一般性的積木外，ScratchX Arduino 外掛還支援事件型的積木，可以寫出更簡潔的互動程式。Arduino 端採用 Firmata 韌體，積木名稱則未支援中文。不過因為 ScratchX 使用者眾，因此仍是不少人使用的設計環境。

### mBlock

玩過 mBot 自走車的朋友對於 mBlock 一定不陌生，兩者都是[深圳市創客工場科技有限公司](https://www.makeblock.com/)所推出的產品。mBlock 3 以 Scratch 2 為基礎，除了可以用來操控 mBot，還可以用來連結 Arduino。因為是中國公司的產品，中文 (簡體) 的支援自然不成問題。而 mBlock 最特別之處就是可以同時支援兩種模式，也就是可以直接控制 Arduino，或是將程式燒錄至 Arduino。不過這兩種模式可以使用的積木是不一樣的，不少積木僅能在傀儡模式下使用。此外在獨立模式下，雖然可以看到積木所對應出的 C++ 程式碼，但是卻無法直接進行修改。必須複製到 Arduino IDE 中才能進行修改，如此一來才可以使用積木所不支援的功能。而最新版的 mBlock 5 以 Scratch 3 為基礎，但是卻以支援自的硬體產品為主，而不再支援 Arduino，著實可惜。

### BlockyDuino

BlockyDuino 編輯器使用獨立模式，產生的 C++ 程式必須自行複製到 Arduino IDE 進行編譯與燒錄，使用方便性稍嫌不足。雖然透過額外的 arduino_web_server.py 可以自動進行燒錄，但是安裝步驟卻有些繁瑣。BlockyDunio 與 arduino_web_server.py 沒有中文介面，而且已經許久未更新，再加上支援的周邊多以 Grove 元件為主，因此通常可考慮其他更為合適的選擇。

### ArduBlock

以外掛的方式替 Arduino IDE 加上積木式編輯功能，但是因為久未更新，因此在新的 Arduino IDE 1.8.x 系列無法正常運作，僅能使用舊版的 Arduino IDE 1.6，因此通常可考慮其他更為合適的選擇。

### miniBloq

miniBloq 同樣多年未更新，而且使用方式與其他積木式編輯器有不小的差異，因此有相當的上手難度。再加上無法正常運行在 Windows 10 的環境下，因此通常可考慮其他更為合適的選擇。

### Modkid Micro

Modkid Micro 同樣是一個已經不再維護的編輯器，開發商已經轉為開發 Modkit for VEX，成為支援自家硬體的付費軟體。

### motoBlockly

為[慧手科技 motoduino](http://www.motoduino.com/) 所研發的線上積木式編輯器，屬於獨立模式。編輯時除了可以看到對應的 C++ 檔案，甚至可以直接進行修改。而且提供額外的代理程式，可以自動將編輯好的程式燒錄至 Arduino。同時提供繁體中文與英文介面，上手門檻可謂相當的低。

### Webduino Blockly

Webduino Blockly 算是一個蠻獨特的產品，透過專屬的 Arduino 無線網路擴充版 Webduino Fly 達到遠端遙控的功能。而 Webdunio Blockly 開發出來的程式屬於網頁的形式，也就是說我們可以從世界各地來控制 Arduino，以達到物聯網的概念。

### WFduino

WFduino 有點類似 Transformer，透過轉介的方式支援多種編輯環境或硬體元件。新版的 WFduino 2 可使用的編輯環境包含 Scratch 2 與 Scratch 3，而硬體部分除了支援 Arduino、DiFi 外，還支援 WF8266R 進行遠端遙控。WFduino 2 提供許多 Scratch 3 的外掛，大幅提升 Scratch 3 與 Arduino 的互動能力。唯一可惜的是，WFduino 僅支援魁儡模式，而無法進行程式的燒錄。

[![](https://cdn.printfriendly.com/buttons/printfriendly-button.png)](# "Printer Friendly, PDF & Email")

Summary

![](https://blog.everlearn.tw/wp-content/uploads/2018/08/arduino-programming-mode-1a.png)

Article Name

Arduino 積木式編輯器總整理

Description

在這篇文章中，我將討論選擇 Arduino 積木式編輯器的考量事項，並對市面上眾多的積木式編輯器做一個摘要與整理，以作為選擇時的參考。

Author

Cyril Wang

Publisher Name

Everlearn Studio

Publisher Logo

![](https://blog.everlearn.tw/wp-content/uploads/2017/07/logo-icon.png)