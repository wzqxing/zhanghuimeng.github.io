---
title: 记一次难忘的Leetcode Internel Contest
urlname: an-unforgettable-leetcode-internel-contest
toc: true
date: 2018-12-02 16:53:24
updated: 2018-12-06 00:31:00
tags: [Leetcode, Leetcode Contest, Essay]
---

（这名字起得很像小学生了）

11月28日的时候，我收到了LeetCode的一封邮件，邀请我参加11月30日的第一次LeetCode Internal Contest。这场比赛仅对一小部分内部用户开放，目的是在正式比赛之前确保所有的题目都没有问题。这场比赛不会计入global ranking，参与之后也不能再参加之后的正式比赛。而且参加的用户会发LeetCoin。

我想，这听起来很有趣。虽然我不能去参加正式比赛提高自己的rating了，但是我还是可以比赛，而且还可以给LeetCode提bug，而且还可以获得一些LeetCoin（虽然这年头这些coin也许不像之前的奖励那样吸引人了）。所以我决定去申请一下。

## 题目出的锅

以我的评价标准，这场比赛并没有出锅；但是赛后有不少参赛者提出了虽然我没看出来但是好像很有道理的意见。正式比赛中修正了一部分内容。正式比赛过后，internel contest的题也都改过了，所以我只能凭借记忆回想题目哪些部分改了。

### 949. Largest Time for Given Digits

[题目地址](https://leetcode.com/problems/largest-time-for-given-digits/description/)

这道题基本没出什么问题，虽然有人尝试了一下负数输入，发现标准答案还是输出了一些东西（而不是报错）。这一点目前还没改。我记得有些LeetCode标程是会assert输入的，不过反正测试数据里肯定没有负数输入，所以好像也无所谓。

我感觉很多人应该都在`[0,0,0,0]`这个输入上错了一次。考察能否想到corner case应该是这一类题的目的之一，我觉得挺好的。不过这要真是一道面试题的话，我可能会说，我在工作环境中会尽可能调用库来处理时间格式

### 950. Reveal Cards In Increasing Order

[题目地址](https://leetcode.com/problems/reveal-cards-in-increasing-order/description/)

很多人都表示很喜欢这道题的思考过程。不过我提出的意见是，我读完整道题和样例之后，并没有明白题目到底是要求返回card的一个order，还是index的一个order。当时样例输入还是`[1,2,3,4,5,6,7]`（所以返回index order和card order的结果是相同的）。还有人表示，因为样例输入是递增的，而且题目描述里也提到了“递增”（虽然说的是“……返回使card**递增**的一种排列”），他以为输入已经是递增的了。现在样例输入已经改成了`[17,13,11,2,3,5,7]`，输出是`[2,13,3,11,5,17,7]`，所以很显然是要求返回card的order（而且输入也不保证是递增的）。

这道题修改的过程使我发觉，样例也是题目的重要组成部分。有些题目里没有说或者没有说清楚的东西可以通过样例来说明，所以选择一组好的样例（既能帮助说明问题，又不泄露题目想要考察的corner case）是很重要的。当然，有时候样例也起到提示一些corner case，降低题目难度的作用（比如一些复杂的模拟题）。

### 951. Flip Equivalent Binary Trees

[题目地址](https://leetcode.com/problems/flip-equivalent-binary-trees/description/)

好像没有什么修改，至少我没看出来。大家都觉得这道题挺好的。不过似乎某一组测试数据中出现了node value重复的情况，现在应该已经改掉了。

### 952. Largest Component Size by Common Factor

[题目地址](https://leetcode.com/problems/largest-component-size-by-common-factor/description/)

这道题的叙述足够清楚（难点在于怎么做，而不在于怎么理解）；不过现在的版本相比之前还是为样例加上了示意图，这一点挺好的。

我觉得这道题难（从通过率来看），但是没有那么难（从做法上来说，只要把素数打表和并查集结合起来就可以做了）。我记得这道题的难度分值原来是9分，后来调整到了8分；以及我才注意到，原来每场比赛的难度分值之和是固定的24分。

还有人注意到了这样一种现象：有人的做法复杂度比较高，但是由于比赛时提供出错的测试数据，所以他们可能会通过在特定测试点中直接输出结果来规避大数据，并且通过这种方法来通过整道题。之前比赛中的[943. Find the Shortest Superstring](/post/leetcode-943-find-the-shortest-superstring)也有这种现象。这可能是提供测试数据用来debug的OJ的共有问题。目前我看到的和我想到的解决方案包括：

* 干脆就不显示数据了
* 造更多的大数据（>100个），增加特判测试点的难度
* 增加输出量并限制源程序长度，防止打表
* 禁止这种行为，鼓励选手举报（我觉得这是个非常馊的主意）
* 参考其他OJ的赛制，比如在比赛中不测大样例，赛后再测

第二和第三种方案都可以增加跳测试点的难度，但大概并不能完全解决这个问题。至于第一种么……我感觉，LeetCode之所以是LeetCode，而不是CodeForces或者UVa或者POJ，比较关键的一点，就是它是给测试数据的，这一点接近于真实的面试环境，而不是ACM环境。（至于其他的点，我觉得LeetCode的社区环境也不错。）反正我做LeetCode Contest而不是CodeForces Contest的最主要的原因就是比赛时也能调试。（其他原因包括时间固定且合适。）

## 其他感受

这场比赛有种很social的感觉，每人都要做自我介绍什么的……我也藉此了解了几位题解区常见选手，嗯，大家都很强（而且都挺nice的）。能和他们一起参加这场比赛，我感到很荣幸。

发了1000个LeetCoin。不知道啥时候才能攒到换一件衣服。

今天我发现我的rating掉了（因为在正式比赛里我的成绩算是0分）。我对此感觉不是很满意，不更新rating我觉得是合理的，rating掉了算啥情况。。。