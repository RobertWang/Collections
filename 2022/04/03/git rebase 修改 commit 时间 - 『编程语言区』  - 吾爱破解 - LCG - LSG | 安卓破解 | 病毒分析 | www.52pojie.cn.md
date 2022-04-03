> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1615511&extra=page%3D1%26filter%3Dauthor%26orderby%3Ddateline)  ![](https://avatar.52pojie.cn/data/avatar/000/83/11/50_avatar_middle.jpg) 鴻渊 

### 废话

最近碰到一个需求，就是需要修改 git 项目最早 commit 时间，全部都往前推一段时间，身为渣渣的我，经历了几天几夜的奋斗，终于搞定了，特来分享一波，要是有发现哪里漏的，可以偷偷告诉我下。

### 步骤

git rebase 命令可以修改 commit 信息 包括时间、作者、描述等。特以此入手。下文 A 表示需要变基的分支

*   翻阅资料后，准备了一个不成熟的 shell 脚本，用于实现自动处理，脚本存在不完善部分，主要是缺少测试数据，有想法的可以自己补充一波
*   另 需注意 windows 环境下 `GIT_COMMITTER_DATE`命令无法执行，mac 的不确定，毕竟穷人买不起，window 环境下需在 `git bash` 中执行
*   通过 `_git log > /C/Users/hong/Desktop/log.txt_` 将 A 的 git 日志导出到 log.txt 文件
*   通过如下 java 代码 提取 log.txt 文件内容，同时自动生成待处理的 commit 脚本文件
    *   考虑到部分提交存在合并的情况，因而 rebase 时 需要处理的 commit 可能会跟 log.txt 顺序不一致，因而考虑生成多个 commit 脚本文件 循环调用执行
*   若想从首次提交开始变基 则使用 `git rebase -i --root`, 若只是想修改某个区间 (注：该区间指定的是一个前开后闭的区间) 的可使用 `git rebase -i [startpoint]  [endpoint]`,Emm 后面这种方式虽然可以变基，但是奈何能力太浅，变基后生成新分支之后我就不知道怎么覆盖到原分支了，有知道的大佬，有空可以介绍下
    *   rebase 会进入可编辑页面，具体使用方式参考 vim
    *   每个 commit 正常都是 pick 操作，可进行的修改如下
        *   pick：保留该 commit（缩写: p）
        *   reword：保留该 commit，但我需要修改该 commit 的注释（缩写: r）
        *   edit：保留该 commit, 但我要停下来修改该提交 (不仅仅修改注释)（缩写: e）
        *   squash：将该 commit 和前一个 commit 合并（缩写: s）
        *   fixup：将该 commit 和前一个 commit 合并，但我不要保留该提交的注释信息（缩写: f）
        *   exec：执行 shell 命令（缩写: x）
        *   drop：我要丢弃该 commit（缩写: d）
    *   edit 即可实现我的需求，因而统一通过`_%s#pick #e #g_`修改为 e，并保存, 其中 %s 表示全局替换， 若想按行替换 可改成 1,5s 表示 1-5 行，根据: set nu 看到的行号进行修改
*   直接在 git bash 上执行通用脚本即可 不需要传入参数
*   确定没问题 执行 git push --force-with-lease 强制更新， 建议强制更新之前备份原分支
    
    ### 用到的 java 代码
    

```
// 要往前多少天
int minusDays = 360;
// commit time 往前推操作截止时间
String cutOffTime = "2021-04-21 00:00:00";

@SneakyThrows
@Test
public void readLogFile() {
  // 日志文件路径   commit 脚本文件要生成到哪个文件夹中
  String filePath = "D:\\log.txt", dirPath = "D:\\gitsh\\";
  // 清空文件夹
  File file = new File(dirPath);
  if (file.exists()) {// 判断是否待删除目录是否存在
    String[] content = file.list();// 取得当前目录下所有文件和文件夹
    assert content != null;
    for(String name : content) {
      File temp = new File(dirPath, name);
      temp.delete();
    }
  } else {
    file.mkdir(); // 创建文件夹
  }
  BufferedReader reader = new BufferedReader(new FileReader(filePath));
  String s = null;
  List<String> list = new ArrayList<>();
  /**
   * log.txt 文件内容格式如下
   * commit c26785080088ff3f146ab438d02736fc90ebb903  当前commit hash 一般处理时取前7位
   * Merge: b39b38a 9747c96  合并的commit hash  需注意 rebase 是无法变更merge的
   * Author: name <email>  本次提交的author信息
   * Date:   Tue Dec 14 11:54:41 2021 +0800  提交时间
   *      本次提交描述
   *      
   * 提取日志信息
   */
  while ((s = reader.readLine()) != null) {
    if (s.startsWith("Author:")) {
      list.add(s.replaceFirst("Author:", "").trim());
    } else if (s.startsWith("Date:")) {
      list.add(dateConvert(s.replaceFirst("Date:", "").trim()));
    } else if (s.startsWith("Merge:")) {
      list.add(s);
    } else if (s.startsWith("commit ")) {
      list.add(s.substring(0, 14));
    }
  }
  reader.close();

  String jb = "GIT_COMMITTER_DATE='%s' GIT_AUTHOR_DATE='%s' git commit --amend --no-edit --date '%s' --author=\"%s\"";

  for (int i = 0, len = list.size(); i < len; i += 3) {
    // 因 rebase 无法变基到merge commit，但导出的log.txt中会包含 merge commit 所以需要剔除  剔除后生成commit 脚本文件即可
    if (list.get(i + 1).startsWith("Merge:")) {
      i += 1;
      continue;
    }
    if (i + 2 >= len) {
      continue;
    }
    String dateStr = list.get(i + 2), authorStr = list.get(i + 1);

    String[] authors = authorStr.split(" <");

    // 创建文件
    File temp = new File(dirPath, list.get(i).replaceFirst("commit ", "") + ".sh");
    BufferedWriter bw = new BufferedWriter(new FileWriter(temp));
    bw.write("#!/bin/bash");
    bw.newLine();
    // /C/Users/hong/Desktop/rebasejs.sh 上面提到的通用脚本
    bw.write("/C/Users/hong/Desktop/rebasejs.sh c '" + authors[0] + "' '" + authors[1].substring(0, authors[1].length() - 1) + "' '" + dateStr + "'");
    bw.newLine();  
    // 关闭流
    bw.close();
  }
}

@SneakyThrows
public String dateConvert(String dateStr) {
  // commit 时间格式处理
  SimpleDateFormat sdf = new SimpleDateFormat("EEE MMM dd HH:mm:ss yyyy z", Locale.US);
  Date d = sdf.parse(dateStr);
  Instant instant = d.toInstant();
  ZoneId zone = ZoneId.systemDefault();
  LocalDateTime localDateTime = LocalDateTime.ofInstant(instant, zone);
  // 在某一些时间之前 提交时间都往前推
  if (localDateTime.isBefore(DateUtil.parseDefault(cutOffTime))) {
    localDateTime = localDateTime.minusDays(minusDays);
  }
  return DateUtil.formatDefault(localDateTime);
}
```

### 通用脚本

```
#!/bin/bash
# git rebase 变基使用的脚本 首次调用可不传入参数
# 参数1 当值为 c 时 会自动判断当前变基执行到哪次commit
# 参数2 姓名
# 参数3 邮箱
# 参数4 时间

# 生成的每次commit 脚本文件夹
commit_sh_path=/C/Users/hong/Desktop/gitsh/
# 原本地环境的 author 信息
my_email=xxxx@xxx.com
my_name=xxxx

# 通用处理方法
rebaseCommit() {
    # 临时设置 author 信息为原本次commit提交的author信息
    git config user.name "$1"
    git config user.email "$2"
    # 修改日志信息以及作者信息
    GIT_COMMITTER_DATE="$3" GIT_AUTHOR_DATE="$3" git commit --amend --no-edit --date "$3" --author="$1 <$2>"
    git rebase --continue
    if [ $? -ne 0 ]
    then 
        echo '执行失败，开始自动合并'
        conflict
        exit 1
    fi
}
# 冲突自动处理
conflict() {
  # 通过 git status 命令可以获得当前 冲突情况
  status_res=`git status`
  status_res=`echo $status_res | grep -Eo "Unmerged path[s]?:.*[\\.][a-z0-9A-Z]+" | grep -Eo "(both (modified|deleted|added)|(deleted|added) by (us|them)):\s+[a-z0-9A-Z\\/\\.]+" | tail -n 1`
  if [ -z "$status_res" ];
  then
    echo "合并结束，继续下一个commit"
    checkOver
    exit 1
  fi
  # 获取冲突类型  冲突文件名  them表示正要合并的  us表示目前的
  type_res=`echo $status_res | grep -Eo "(modified|deleted|added)"`
  file_name=`echo $status_res | grep -Eo "[a-z0-9A-Z\\/\\.]+" | tail -n 1`
  obj_type=`echo $status_res | grep -Eo "(us|them)" | head -n 1`

  # 冲突类型分为 both (modified|deleted|added) 和 (modified|deleted|added) by (us|them) 
  # 目前这几种情况 (modified|deleted|added) by us没碰到过  
  # 要是各位大佬有碰到的话 可以把 git status 信息发给我参考完善下
  if [ "$type_res" == "modified" ];
  then
    if [ "$obj_type" == "us" ];
    then
      obj_type="ours"
      # 目前还没碰到当前情况 因而不确定合并方式 碰到会自动提前结束脚本
      echo "发现us: $status_res"
      exit 1
    else
      obj_type="theirs"
    fi
    # 冲突合并方式  一般合并 theirs
    git checkout --${obj_type} $file_name
    git add $file_name
  elif [ "$type_res" == "deleted" ];
  then
    # 冲突是删除文件时使用
    git rm $file_name
  elif [ "$type_res" == "added" ];
  then
    git add $file_name
  else
    echo "无匹配操作: $status_res"
    exit 1
  fi
    # 遍历 查看是否还有冲突
    conflict
}
# 检查是否还有需要处理的提交记录
checkOver() {
    # 查看是否存在需要执行的 commit 
    status_res=`git status`
    status_res=`echo $status_res | grep -Eo "Last command[s]? done.*(Next|No) command" | grep -Eo "edit [a-z0-9]{7}" | grep -Eo "[a-z0-9]{7}$" | tail -n 1`
    if [ ! $status_res ];
    then
        echo "执行完成"
        exit 1
    fi
    echo "执行$status_res"
    $commit_sh_path$status_res.sh
    # 删除已执行文件，方便后续执行完之后查看是否有漏的commit
    rm -rf $commit_sh_path$status_res.sh
}
if [ "$1" == "c" ];
then
  # 修改的commit是别人提交时，rebase 会导致出现两个author信息，因而需先取消全局设置的git author信息，等处理完了之后再重新设置
    git config --global --unset user.name
    git config --global --unset user.email
    rebaseCommit "$2" "$3" "$4"
    git config --global user.email "$my_email"
    git config --global user.name "$my_name"
fi

checkOver
```

### 生成的脚本文件

```
#!/bin/bash
# 通用脚本 参数1固定  参数2表示author名 3表示author邮箱 4表示要改为哪个时间
/D/rebasejs.sh 'c' 'xx' 'xxx@xx.com' '2019-06-30 15:32:04'
```

![](https://avatar.52pojie.cn/data/avatar/001/78/63/87_avatar_middle.jpg)coolinmind  感谢分享，这个实用![](https://static.52pojie.cn/static/image/smiley/default/4.gif)