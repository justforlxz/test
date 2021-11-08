---
title: 使用Google日历安排工作任务
p: ./use-google-calendar-to-finishe-work
date: 2018-11-09 21:15:45
tags:
categories:
---

目前我们正在尝试把工作的分配和讨论放在github上进行，这样可以使我们的用户和开发者更容易接触到我们，可以提bug和对需求进行讨。

但是使用起来还是有些不便，比如使用tower进行任务分配的时候，可以方便的移动一个任务到某个分类，或者指派一个时间。但是github上是基于issue的，并不是为了做这种事来设计的，所以需求上有一些出入。但是[@hualet](https://github.com/hualet)大佬根据github的api写了一个bot来做一点微小的事，当一个issue的assignees只剩QA的同事时，issue会被bot移动到测试栏中，只剩一个开发同事时(基本上是负责该任务的开发者)，会被移动到开发栏中。

但是因为不能做到比如今天、明天、下周等时间的显示，所以任务只能通过每天开会来口头告知时间，但是这并不妨碍我进行自己的任务时间安排。请出世界第一的神器(日历)。

<!-- more -->

我选择使用谷歌日历，~~才不是因为它有网页还有安卓客户端【哼~~

谷歌日历上支持新建三种类型，分别是活动、提醒和任务。活动是开始时间明确，但是结束时间未知的类型，适合用作对时间不严格的情况。提醒则是在活动的基础上添加了提供功能，在活动即将开始时发送通知提醒。任务则是熟悉的ToDoList，适合用来分配今天一定要做，但是时间未知的事。

我添加了每天的开会提醒，再开完会以后，我会把身上的新任务创建成task，然后再添加大概的活动来确定一下要完成的task。把今天没有时间做的task移动到明天，留在当天的task尽量要当天完成，可以得到今天的任务列表和延期列表，让我对要做的事有完整的控制。

谷歌日历的日视图和周视图会显示一条线，告诉你现在的时间，应该进行什么活动了。

![activity](use-google-calendar-to-finishe-work/activity.png)

![task](use-google-calendar-to-finishe-work/task.png)

![week](use-google-calendar-to-finishe-work/week.png)

在手机上需要使用两个app，Google calendar和Google task，活动和提醒需要calendar，task则需要单独使用一个app，只有网页上才是整合的。

![android-calendar](use-google-calendar-to-finishe-work/android-calendar.png)

![android-task](use-google-calendar-to-finishe-work/android-task.png)

![day](use-google-calendar-to-finishe-work/day.png)

因为我也是才开始用日历来分配任务的时间，所以记录的内容并不多，我也在摸索如何使用这些功能，但是我觉得使用日历来记录和管理时间是非常不错的一件事，我可以通过看某天的活动来回忆当天所做的事，也可以根据记录的内容来分析自己在某些任务上使用了多少的时间。
