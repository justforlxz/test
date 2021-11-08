---
title: 使用inquirer提供交互式git commit
date: 2020-06-15 10:36:19
tags: [ 'Linux' ]
categories:
---

公司计划规范所有commit提交，开发部门综合出来了一份模板。

```text
title（应当使用陈述句，简短的描述这个提交所做的事情）

Description（详细说明代码的改动，包含代码的实现思路，以及为什么这么做，可能会影响哪些功能。对于代码的审核者，需要从这段描述中能完全理解代码中所有改动的内容）

Log: 写一段面向于产品的总结性内容，用于自动生成crp上的changlog，需要注意的事，这段描述必须从产品的角度考虑。
Bug: https://xxxxxxxxxxx 对应pms bug的链接
Issue: fix #xx 所修复的bug对于的github issue，其中 "fix #xx"是github关闭issue的规则，此处内容只需要满足github的要求即可，详情请参考 https://help.github.com/en/enterprise/2.16/user/github/managing-your-work-on-github/closing-issues-using-keywords
Task: http://xxxxxxxxxxxx 对应pms任务的链接
```

<!-- more -->

## python inquirer

之前在掘金上看到有人在使用交互式的commit来规范commit信息，觉得用起来挺不错的，刚好符合本次公司的要求，不过原项目是nodejs的，项目里肯定不能让每个开发都安装一个node,所以就找一下代替品，然后就发现了[python-inquirer](https://github.com/magmax/python-inquirer)。

使用起来也非常的方便，通过inquirer.Text、inquirer.List、inquirer.Checkbox就可以创建相应的交互，并把组合好的列表交给inquirer.prompt处理，返回一个对象，内部包含了所有做出的选择。

### Text

```python
import inquirer
questions = [
  inquirer.Text('name', message="What's your name"),
  inquirer.Text('surname', message="What's your surname"),
  inquirer.Text('phone', message="What's your phone number",
                validate=lambda _, x: re.match('\+?\d[\d ]+\d', x),
                )
]
answers = inquirer.prompt(questions)
```

### List

```python
import inquirer
questions = [
  inquirer.List('size',
                message="What size do you need?",
                choices=['Jumbo', 'Large', 'Standard', 'Medium', 'Small', 'Micro'],
            ),
]
answers = inquirer.prompt(questions)
```

### CheckBox

```python
import inquirer
questions = [
  inquirer.Checkbox('interests',
                    message="What are you interested in?",
                    choices=['Computers', 'Books', 'Science', 'Nature', 'Fantasy', 'History'],
                    ),
]
answers = inquirer.prompt(questions)
```

## 公司的模板

```python
#!/bin/python

import inquirer
import sys
from string import Template
import subprocess

def number_validation(answers, current):
  return int(current)

def empty_validation(answers, current):
  return bool(current)

def no_validation(answers, current):
  return True

def addList(name, message, list):
  return inquirer.List(name, message, list)

def addText(name, message, valid):
  return inquirer.Text(name, message, validate=valid)

def addCheck(_name, _message, _choices):
  return inquirer.Checkbox(_name, message=_message, choices=_choices)

questions = [
  addCheck("action", "选择非选项", ['Bug', 'Issue', 'Task'])
]

optinalAnswers = inquirer.prompt(questions)

questions = [
  addList('action', "select you action", ['fix', 'feat', 'refactor', 'docs', 'chore', 'style', 'pref', 'test']),
  addText("module", "input module name", no_validation),
  addText('title', "input title", empty_validation),
  addText('description', "input description", empty_validation),
  addText("log", "input log", empty_validation),
]

optinal = {
  "Bug": addText("bug", "input bug id", number_validation),
  "Issue": addText("issue", "input issue id", empty_validation),
  "Task": addText("task", "input task id", number_validation),
}

optinalMap = {
  "Bug": "bug",
  "Issue": "issue",
  "Task": "task",
}

for action in optinalAnswers["action"]:
  questions.append(optinal[action])

answers = inquirer.prompt(questions)

template = '${action}'

if answers["module"]:
  template += '(${module})'

template += ': ${title}\n\nDescription: ${description}\n\nLog: ${log}\n'

for action in optinalAnswers["action"]:
  template += action + ": ${" + optinalMap[action] + "}\n"

subprocess.run(["git", "commit", "-m", Template(template).substitute(answers)])

```

### 修改git editor

将上面的内容保存到/usr/bin/git-inquirer。

当我们执行git inquirer的时候就能看到交互，当操作完成后可以看到git log中message已经是按模板填充了。
