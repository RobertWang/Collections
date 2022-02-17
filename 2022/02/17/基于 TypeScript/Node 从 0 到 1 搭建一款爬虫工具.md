> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651595928&idx=2&sn=eb1a27b7d649a45606896519b75c0cea&chksm=8022f359b7557a4f5895541b650f1b48a87445a64a4012ead472bd1f1e1bbcd791def2c99527&mpshare=1&scene=1&srcid=0127GOKLIsFTKmDpR6eXDdhu&sharer_sharetime=1643254444605&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

前言

今天，我们将使用 TS 这门语言搭建一款爬虫工具。目标网址是什么呢？我们去上网一搜，经过几番排查之后，我们选定了这一个网站。

```
https://www.hanju.run/
```

一个视频网站，我们的目的主要是爬取这个网站上视频的播放链接。下面，我们就开始进行第一步。

第一步
---

俗话说，万事开头难。不过对于这个项目而言，恰恰相反。你需要做以下几个事情：

1.  我们需要创建一个项目文件夹
    
2.  键入命令，初始化项目
    

```
 npm init -y
```

3.  局部安装 typescript
    

```
  npm install typescript -D
```

4.  接着键入命令，生成 ts 配置文件
    

```
 tsc --init
```

5.  局部安装 ts-node，用于命令行输出命令
    

```
npm install -D ts-node
```

6.  在项目文件夹中创建一个`src`文件夹
    

然后我们在`src`文件夹中创建一个`crawler.ts`文件。

7.  在 package.json 文件中修改快捷启动命令
    

```
"scripts": {
    "dev-t": "ts-node ./src/crawler.ts"
  }
```

第二步
---

接下来，我们将进行实战操作，也就是上文中`crawler.ts`文件是我们的主战场。

我们首先需要引用的这几个依赖，分别是

```
import superagent from "superagent";
import cheerio from "cheerio";
import fs from "fs";
import path from "path";
```

所以，我们会这样安装依赖：

`superagent`作用是获取远程网址 html 的内容。

```
npm install superagent
```

`cheerio`作用是可以通过 jQ 语法获取页面节点的内容。

```
npm install cheerio
```

剩余两个依赖`fs`，`path`。它们是 node 内置依赖，直接引入即可。

我们完成了安装依赖，但是会发现你安装的依赖上会有红色报错。原因是这样的，`superagent`和`cheerio`内部都是用 JS 写的，并不是 TS 写的，而我们现在的环境是 TS。所以我们需要翻译一下，我们将这种翻译文件又称类型定义文件（以. d.ts 为后缀）。我们可以使用以下命令安装类型定义文件。

```
npm install -D @types/superagent
```

```
npm install -D @types/cheerio
```

接下来，我们就认认真真看源码了。

1.  安装完两个依赖后，我们需要创建一个`Crawler`类，并且将其实例化。
    
    ```
    import superagent from "superagent";
    import cheerio from "cheerio";
    import fs from "fs";
    import path from "path";
    
    class Crawler {  
      constructor() {    
      
      }
    }
    
    const crawler = new Crawler();
    ```
    
2.  我们确定下要爬取的网址，然后赋给一个私有变量。最后我们会封装一个`getRawHtml`方法来获取对应网址的内容。
    
    `getRawHtml`方法中我们使用了`async`/`await`关键字，主要用于异步获取页面内容，然后返回值。
    
    ```
    import superagent from "superagent";
    import cheerio from "cheerio";
    import fs from "fs";
    import path from "path";
    
    class Crawler { 
      private url = "https://www.hanju.run/play/39221-4-0.html"; 
      
      async getRawHtml() {   
        const result = await superagent.get(this.url);   
        return result.text;
      } 
      
      async initSpiderProcess() {  
        const html = await this.getRawHtml(); 
      }
      
      constructor() {  
        this.initSpiderProcess(); 
     }
    }
    
    const crawler = new Crawler();
    ```
    
3.  使用`cheerio`依赖内置的方法获取对应的节点内容。
    
    我们通过`getRawHtml`方法异步获取网页的内容，然后我们传给`getJsonInfo`这个方法，注意是 **string 类型**。我们这里通过`cheerio.load(html)`这条语句处理，就可以通过 jQ 语法来获取对应的节点内容。我们获取到了网页中视频的标题以及链接，通过键值对的方式添加到一个对象中。注：我们在这里定义了一个接口，定义键值对的类型。
    
    ```
     import superagent from "superagent";
     import cheerio from "cheerio";
     import fs from "fs";
     import path from "path";
    
     interface Info {  
       name: string; 
       url: string;
     }
    
     class Crawler { 
       private url = "https://www.hanju.run/play/39221-4-0.html"; 
     
       getJsonInfo(html: string) {   
         const $ = cheerio.load(html);  
         const info: Info[] = [];  
         const scpt: string = String($(".play>script:nth-child(1)").html());   
         const url = unescape(     
           scpt.split(";")[3].split("(")[1].split(")")[0].replace(/\"/g, "") 
         );   
         const name: string = String($("title").html());  
         info.push({  
           name,   
           url, 
         }); 
         const result = {   
           time: new Date().getTime(),     
           data: info,   
         };   
         return result; 
      }  
    
      async getRawHtml() { 
        const result = await superagent.get(this.url);
        return result.text; 
      } 
    
      async initSpiderProcess() { 
          const html = await this.getRawHtml();   
        const info = this.getJsonInfo(html); 
      } 
        constructor() { 
          this.initSpiderProcess();  
       }
     }
    
     const crawler = new Crawler();
    ```
    
4.  我们首先要在项目根目录下创建一个`data`文件夹。然后我们将获取的内容我们存入文件夹内的`url.json`文件（文件自动生成）中。
    
    我们将其封装成`getJsonContent`方法，在这里我们使用了`path.resolve`来获取文件的路径。`fs.readFileSync`来读取文件内容，`fs.writeFileSync`来将内容写入文件。注：我们分别定义了两个接口`objJson`与`InfoResult`。
    

```
import superagent from "superagent";
import cheerio from "cheerio";
import fs from "fs";
import path from "path";

interface objJson { 
  [propName: number]: Info[];
}

interface Info {  
  name: string;  
  url: string;
}

interface InfoResult { 
  time: number;  
  data: Info[];
}
class Crawler { 
  private url = "https://www.hanju.run/play/39221-4-0.html";  
  
  getJsonInfo(html: string) { 
    const $ = cheerio.load(html);   
    const info: Info[] = [];   
    const scpt: string = String($(".play>script:nth-child(1)").html());  
    const url = unescape(     
      scpt.split(";")[3].split("(")[1].split(")")[0].replace(/\"/g, "")  
    );  
    const name: string = String($("title").html());   
    info.push({   
      name,    
      url,    
    });   
    const result = {   
      time: new Date().getTime(),   
      data: info,   
    };  
    return result;  
} 

async getRawHtml() {  
  const result = await superagent.get(this.url);  
  return result.text; 
}  

getJsonContent(info: InfoResult) {    
  const filePath = path.resolve(__dirname, "../data/url.json");  
  let fileContent: objJson = {};  
  if (fs.existsSync(filePath)) { 
    fileContent = JSON.parse(fs.readFileSync(filePath, "utf-8"));   
  }   
  fileContent[info.time] = info.data;  
  fs.writeFileSync(filePath, JSON.stringify(fileContent));  
} 
  
  async initSpiderProcess() { 
    const html = await this.getRawHtml();  
    const info = this.getJsonInfo(html); 
    this.getJsonContent(info); 
  } 
  
  constructor() {
    this.initSpiderProcess(); 
  }
}

const crawler = new Crawler();
```

5.  运行命令
    
    ```
    npm run dev-t
    ```
    
6.  查看生成文件的效果
    
    ```
    {
      "1610738046569": [   
        {   
          "name": "《复仇者联盟4：终局之战》HD1080P中字m3u8在线观看-韩剧网",    
          "url": "https://wuxian.xueyou-kuyun.com/20190728/16820_302c7858/index.m3u8"  
         }  
       ], 
       "1610738872042": [  
         {    
           "name": "《钢铁侠2》HD高清m3u8在线观看-韩剧网",   
           "url": "https://www.yxlmbbs.com:65/20190920/54uIR9hI/index.m3u8"   
         } 
       ], 
       "1610739069969": [ 
         {   
         "name": "《钢铁侠2》中英特效m3u8在线观看-韩剧网",
         "url": "https://tv.youkutv.cc/2019/11/12/mjkHyHycfh0LyS4r/playlist.m3u8"  
       } 
     ]
     }
    ```
    

准结语
---

到这里真的结束了吗？

不！

不！

不！

真的没有结束。

我们会看到上面一坨代码，真的很臭~

我们将分别使用组合模式与单例模式将其优化。

优化一：组合模式
--------

**组合模式（Composite Pattern）**，又叫部分整体模式，是用于把一组相似的对象当作一个单一的对象。组合模式依据树形结构来组合对象，用来表示部分以及整体层次。这种类型的设计模式属于结构型模式，它创建了对象组的树形结构。

这种模式创建了一个包含自己对象组的类。该类提供了修改相同对象组的方式。

简言之，就是可以像处理简单元素一样来处理复杂元素。

首先，我们在 src 文件夹下创建一个`combination`文件夹，然后在其文件夹下分别在创建两个文件`crawler.ts`和`urlAnalyzer.ts`。

**crawler.ts**

`crawler.ts`文件的作用主要是处理获取页面内容以及存入文件内。

```
import superagent from "superagent";
import fs from "fs";
import path from "path";
import UrlAnalyzer from "./urlAnalyzer.ts";

export interface Analyzer { 
  analyze: (html: string, filePath: string) => string;
}

class Crowller {  
  private filePath = path.resolve(__dirname, "../../data/url.json"); 
  
  async getRawHtml() {  
    const result = await superagent.get(this.url);    
    return result.text;
  } 
  
  writeFile(content: string) { 
    fs.writeFileSync(this.filePath, content); 
  } 
  
  async initSpiderProcess() {  
    const html = await this.getRawHtml(); 
    const fileContent = this.analyzer.analyze(html, this.filePath);   
    this.writeFile(fileContent);  
  } 
  
  constructor(private analyzer: Analyzer, private url: string) {   
    this.initSpiderProcess();
  }
}

const url = "https://www.hanju.run/play/39257-1-1.html";

const analyzer = new UrlAnalyzer();
new Crowller(analyzer, url);
```

**urlAnalyzer.ts**

`urlAnalyzer.ts`文件的作用主要是处理获取页面节点内容的具体逻辑。

```
import cheerio from "cheerio";
import fs from "fs";import { Analyzer } from "./crawler.ts";
interface objJson { 
  [propName: number]: Info[];
}
interface InfoResult { 
  time: number; 
  data: Info[];
}
interface Info { 
  name: string; 
  url: string;
}

export default class UrlAnalyzer implements Analyzer { 
  private getJsonInfo(html: string) {  
    const $ = cheerio.load(html);  
    const info: Info[] = [];  
    const scpt: string = String($(".play>script:nth-child(1)").html());  
    const url = unescape(    
      scpt.split(";")[3].split("(")[1].split(")")[0].replace(/\"/g, "")  
    );  
    const name: string = String($("title").html());   
    info.push({   
      name,   
      url, 
    });  
    const result = {    
      time: new Date().getTime(),   
      data: info,  
    };  
    return result; 
  } 
  
  private getJsonContent(info: InfoResult, filePath: string) {   
    let fileContent: objJson = {};   
    if (fs.existsSync(filePath)) {   
      fileContent = JSON.parse(fs.readFileSync(filePath, "utf-8"));   
    }  
    fileContent[info.time] = info.data;  
    return fileContent;  
  } 
  
  public analyze(html: string, filePath: string) {  
    const info = this.getJsonInfo(html);  
    console.log(info);  
    const fileContent = this.getJsonContent(info, filePath);    
    return JSON.stringify(fileContent);
  }
}
```

可以在`package.json`文件中定义快捷启动命令。

```
  "scripts": {  
    "dev-c": "ts-node ./src/combination/crawler.ts"
  },
```

然后使用`npm run dev-c`启动即可。

优化二：单例模式
--------

**单例模式（Singleton Pattern）** 是 Java 中最简单的设计模式之一。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。

这种模式涉及到一个单一的类，该类负责创建自己的对象，同时确保只有单个对象被创建。这个类提供了一种访问其唯一的对象的方式，可以直接访问，不需要实例化该类的对象。

**应用实例：**

*   1、一个班级只有一个班主任。
    
*   2、Windows 是多进程多线程的，在操作一个文件的时候，就不可避免地出现多个进程或线程同时操作一个文件的现象，所以所有文件的处理必须通过唯一的实例来进行。
    
*   3、一些设备管理器常常设计为单例模式，比如一个电脑有两台打印机，在输出的时候就要处理不能两台打印机打印同一个文件。
    

同样，我们在 src 文件夹下创建一个`singleton`文件夹，然后在其文件夹下分别在创建两个文件`crawler1.ts`和`urlAnalyzer.ts`。

这两个文件的作用与上文同样，只不过代码书写不一样。

**crawler1.ts**

```
import superagent from "superagent";
import fs from "fs";
import path from "path";
import UrlAnalyzer from "./urlAnalyzer.ts";

export interface Analyzer { 
  analyze: (html: string, filePath: string) => string;
}

class Crowller { 
  private filePath = path.resolve(__dirname, "../../data/url.json"); 
  
  async getRawHtml() {  
    const result = await superagent.get(this.url); 
    return result.text; 
  } 
  
  private writeFile(content: string) {  
    fs.writeFileSync(this.filePath, content); 
  } 
  
  private async initSpiderProcess() {  
    const html = await this.getRawHtml();  
    const fileContent = this.analyzer.analyze(html, this.filePath);  
    this.writeFile(JSON.stringify(fileContent)); 
  } 
  
  constructor(private analyzer: Analyzer, private url: string) {  
    this.initSpiderProcess(); 
  }
}
const url = "https://www.hanju.run/play/39257-1-1.html";

const analyzer = UrlAnalyzer.getInstance();
new Crowller(analyzer, url);
```

**urlAnalyzer.ts**

```
import cheerio from "cheerio";
import fs from "fs";
import { Analyzer } from "./crawler1.ts";

interface objJson {
  [propName: number]: Info[];
}
interface InfoResult {
  time: number;
  data: Info[];
}
interface Info {
  name: string;
  url: string;
}
export default class UrlAnalyzer implements Analyzer {
  static instance: UrlAnalyzer;

  static getInstance() {
    if (!UrlAnalyzer.instance) {
      UrlAnalyzer.instance = new UrlAnalyzer();
    }
    return UrlAnalyzer.instance;
  }

  private getJsonInfo(html: string) {
    const $ = cheerio.load(html);
    const info: Info[] = [];
    const scpt: string = String($(".play>script:nth-child(1)").html());
    const url = unescape(
      scpt.split(";")[3].split("(")[1].split(")")[0].replace(/\"/g, "")
    );
    const name: string = String($("title").html());
    info.push({
      name,
      url,
    });
    const result = {
      time: new Date().getTime(),
      data: info,
    };
    return result;
  }

  private getJsonContent(info: InfoResult, filePath: string) {
    let fileContent: objJson = {};
    if (fs.existsSync(filePath)) {
      fileContent = JSON.parse(fs.readFileSync(filePath, "utf-8"));
    }
    fileContent[info.time] = info.data;
    return fileContent;
  }

  public analyze(html: string, filePath: string) {
     const info = this.getJsonInfo(html);
     console.log(info);
    const fileContent = this.getJsonContent(info, filePath);
    return JSON.stringify(fileContent);
  }

  private constructor() {}
}
```

可以在`package.json`文件中定义快捷启动命令。

```
 "scripts": {
     "dev-s": "ts-node ./src/singleton/crawler1.ts",
  },
```

然后使用`npm run dev-s`启动即可。

结语
--

这下真的结束了，谢谢阅读。希望可以帮到你。

完整源码地址：

```
https://github.com/maomincoding/TsCrawler
```

- EOF -