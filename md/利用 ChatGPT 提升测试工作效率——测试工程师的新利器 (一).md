> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/6fdGV4wgeEYBm-sq-vb5uQ)

导读
--

本文探讨了在测试工作中如何应用 ChatGPT 来提升效率。通过与 ChatGPT 进行人机对话，软件测试工程师可以进行提效，包括需求分析、测试用例编写、缺陷报告和自动化测试脚本生成等方面。通过充分利用 ChatGPT 的能力，软件测试工程师们可以更高效、准确地完成任务，提升测试质量。
















========================================================================================================================================================================

**01** 

**前言**

在今年的敏捷团队建设中，我通过 Suite 执行器实现了一键自动化单元测试。Juint 除了 Suite 执行器还有哪些执行器呢？由此我的 Runner 探索之旅开始了！

测试工程师在整个测试流程中，需求分析及编写测试用例在整个流程中占很大一部分比例。如果能通过 chatGPT 进行需求分析后，输出质量比较好的测试用例，将大大提升测试工程师的工作效率。然而经过测试有时候将需求输入给 chatGPT 后并不能生成令人满意的结果。但是对于系统或表单级别的需求，chatGPT 给出的结果基本可以让人满意。甚至可以通过给定前端功能代码直接生成测试测试用例。

例：对于完全不了解多租户系统，完全可以让 chatGPT 给出一些测试点和方向。说明：结果截图太长只展示了一部分。

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1U5s2OeabFXHwrAYwbevuf37fxibPVvvEM2XFlvnFGibqnCGvC1dKZcPCicSbCBtNNcLUwraqIEUEezQ/640?wx_fmt=png&from=appmsg)

例：给出指定表单各个字段输入参数，参数取值范围及约束条件，需要 chatGPT 生成各种用例组合。说明：结果截图太长只展示了一部分。

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1U5s2OeabFXHwrAYwbevuf3uganaXGicjMCHFx64KH64ibtEH6ReW1R0ahI33v7XI6WZothrEv4hVKA/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1U5s2OeabFXHwrAYwbevuf32Znh0IHMlfru5NszyVoh9gsb30HYe9B1Q1nAjpUa2K5icgRG5UOicOicQ/640?wx_fmt=png&from=appmsg)

```
示例代码：<div>
        <!-- +.withqrc 切换 -->
        <div>
            <div>
                    <div>
                        <div>登录账号：<span>wywangyanjie</span> </div>
                        <br>
                        <div>请在您的<span>京Me</span>上确认，登录验证码<br><span></span>，请在<span>分钟内</span>完成操作！</div>
                        <input type="hidden" name="checkCode?if_exists" value="">
                        <br>
                    </div>
                    <div>
                        <div>您的账号<span>wywangyanjie</span>，本次登录已超时，请重新登录！ </div>
                        <br>
                        <div><input type="button" onclick="refreshPage()" value="重新登录"></div>
                    </div>
                    <div>
                        <div>您的账号<span>wywangyanjie</span>，本次登录存在异常情况，请重新登录！ </div>
                        <br>
                        <div><input type="button" onclick="refreshPage()" value="重新登录"></div>
                    </div>
                    <div>
                        <div>您的账号<span>wywangyanjie</span>，已被拒绝登录，请重新登录！ </div>
                        <br>
                        <div><input type="button" onclick="refreshPage()" value="重新登录"></div>
                    </div>
                </form>
                <a href="javascript:;" tologintype="2" title="切换到扫码登录"></a>
            </div>
                <div>
                    <div></div>
                    <div>扫描成功！</div>
                    <div>请在手机上确认是否登录</div>
                </div>
                <a href="javascript:;" tologintype="1" title="切换到密码登录"></a>
            </div>
            <div>
                <div>
                    <div title="密码登录">
                        <div>密码登录</div>
                    </div>
                    <div title="扫码登录">
                        <div>扫码登录</div>
                    </div>
                </div>
                <div><input type="button" value="登  录"></div>
                <div><a href="/sso/findpwd/index">忘记密码？</a></div>
                <div><i></i><span>用户名或密码错误，请重试！</span>
                </div>
                <div>
                    <span>语言选择</span>
                    <div>
                        <select onchange="selectLan()">
                            <option value="zh_CN">中文</option>
                            <option value="en_US">English</option>
                        </select>
                    </div>
                </div>
            </div>
        </div>
    </div>

```

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1U5s2OeabFXHwrAYwbevuf38tQBVahqP4bkabibbAfgVyVhgga01HQsTjz272X908xJgthOS9R2ICw/640?wx_fmt=png&from=appmsg)

**03** 

 

**代码辅助生成**
==========

 

理解，首先 MCube 会依据模板缓存状态判断是否需要网络获取最新模板，当获取到模板后进行模板加载，加载阶段会将产物转换为视图树的结构，转换完成后将通过表达式引擎解析表达式并取得正确的值，通过事件解析引擎解析用户自定义事件并完成事件的绑定，完成解析赋值以及事件绑定后进行视图的渲染，最终将目标页面展示到屏幕。

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1U5s2OeabFXHwrAYwbevuf3AIp0b0TmbsqnHT9ibadwjyka54GicwaiavXlicvSgoicVGQ6MABklsiaxqxw/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1U5s2OeabFXHwrAYwbevuf3UibrmrPxHSZFGcQicjKWamuCzH6RzAgVIphxZHibIn9SplpjsTRKGnTiaw/640?wx_fmt=png&from=appmsg)图 8、9.

**04** 

  **接口测试用例生成**  

理解，首先 MCube 会依据模板缓存状态判断是否需要网络获取最新模板，当获取到模板后进行模板加载，加载阶段会将产物转换为视图树的结构，转换完成后将通过表达式引擎解析表达式并取得正确的值，通过事件解析引擎解析用户自定义事件并完成事件的绑定，完成解析赋值以及事件绑定后进行视图的渲染，最终将目

给 chatGPT 输入接口，及对应的入参和入参的限制约束条件，让其生成测试用例。chatGPT 给出的用例场景基本全部覆盖了，给定了四个参数生成了 20 条用例（这个功能可以打 10 分了）。实际工作中可以根据实际场景进行精简进行测试，但是如果系统要求比较高，例如银行系统可能就需要尽可能的全量覆盖。

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1U5s2OeabFXHwrAYwbevuf3JNOWjRKnaIdX5F4bfVLCcNMCYGK6UltlnzJ0vLtSfpv2kibg3af8eRA/640?wx_fmt=png&from=appmsg)图 10.

**05** 

  **接口自动化脚本生成**  

理解，首先 MCube 会依据模板缓存状态判断是否需要网络获取最新模板，当获取到模板后进行模板加载，加载阶段会将产物转换为视图树的结构，转换完成后将通过表达式引擎解析表达式并取得正确的值，通过事件解析引擎解析用户自定义事件并完成事件的绑定，完成解析赋值以及事件绑定后进行视图的渲染，最终将目

细节请参考 “京东技术” 公众号文章：《以效率为导向：用 ChatGPT 和 HttpRunner 实现敏捷自动化测试（二）》

实现过程思路：先将写好的接口自动化脚本例子提供给 chatGPT，然后将生成的接口测试用例提供给 chatGPT。其就可以按照给定的样例，将接口自动化测试用例自动转换为脚本。当然具体的断言还需要根据实际情况进行修改。说明：结果截图太长只展示了一部分。

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1U5s2OeabFXHwrAYwbevuf3kXLqBuiajO1uQiaYcFHo1LId85AcGKpiahx5iaZjxTw5hiatcSg1T8icPMIg/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1U5s2OeabFXHwrAYwbevuf3IUnw3f93R8Ze2PXU91Mwn5oHeoy0dFdkXfY0Jm9N6QykFlHrlZyaPg/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1U5s2OeabFXHwrAYwbevuf331KpV0Amx8BCod9TW55icFJcr1xcldubAfHsmRTfcs98d5oQauJpEgQ/640?wx_fmt=png&from=appmsg)图 11、12、13.

说明：以上自动化脚本是通过 Httprunner（https://httprunner.com/）框架实现。

**06** 

  **其他**  

理解，首先 MCube 会依据模板缓存状态判断是否需要网络获取最新模板，当获取到模板后进行模板加载，加载阶段会将产物转换为视图树的结构，转换完成后将通过表达式引擎解析表达式并取得正确的值，通过事件解析引擎解析用户自定义事件并完成事件的绑定，完成解析赋值以及事件绑定后进行视图的渲染，最终将目

其他方向，例如 SQL、DockerFile、Nginx 配置、Shell 脚本编写等等方方面面都可以让 chatGPT 帮忙，再此就不再一一截图举例了。这个对于未接触过相关知识的新手来说简直太友好了，虽然搜索引擎也可以找到对应内容，但是可能需要人工进行筛选整合。有了 chatGPT 辅助后，初级选手如果使用得当完全可以变成中级选手。

总体来说，使用 ChatGPT 生成代码，生成用例，生成 SQL、DockerFile、Nginx 配置和 Shell 脚本等，虽然能提供一定的指导和参考，但仍需要人工进行验证和优化，以确保生成的内容符合预期。尽管一些简单的东西可以自己从零到一完成，但是如果使用通过命令让 chatGPT 去自动生成完成，然后人工再修改校验肯定会比从零到一实现快。

**07** 

  **总结**  

理解，首先 MCube 会依据模板缓存状态判断是否需要网络获取最新模板，当获取到模板后进行模板加载，加载阶段会将产物转换为视图树的结构，转换完成后将通过表达式引擎解析表达式并取得正确的值，通过事件解析引擎解析用户自定义事件并完成事件的绑定，完成解析赋值以及事件绑定后进行视图的渲染，最终将目

在测试工作中可以辅助功能测试包括需求分析或解读代码（注意代码安全）后生成测试用例，还可以辅助生成代码，接口测试用例，自动化脚本等各个方向起作用。当然实际使用中可能会因为提示词的不同生成的结果需要人工多次对话训练才可以。但是使用 chatGPT 肯定比不用能提高工作效率。当然具体落地后如何进行量化提效抽象等等问题依然在探索中，迈开第一步后依然任重而道远。

写在最后的话：希望用您发财的小手 **点赞 转发 关注 在看**！感谢您的关注和支持！