---
title: jenkins 自动构建任务工具
date: 2018-01-17T15:41:38+08:00
tags: [Jenkins]
comments: true
---
有时候，Jenkins需要自动创建Job来构建运行某些任务，找了一些工具如下
  - Job Generator
  - Job DSL
  - Jenkins job builder

## Job Generator
这是一个Jenkins的插件，安装之后可以在Web页面上可以创建一个“Job Generator”的模板任务，在添加参数和设置构建条件之后就可以手动构建任务。这个插件很长时间没有更新了，有一个问题是不能使用环境变量，包括Jenkins定义的。用来手动调试是可行的。

具体的配置可以参考：https://wiki.jenkins.io/display/JENKINS/Job+Generator+Plugin

## Job DSL
这也是一个Jenkins的插件，使用Groovy脚本来创建任务，安装之后在某个任务上插入脚本片段就可以生成需要的Job。这个插件比较灵活，可以在脚本里控制所有的配置。

具体配置可参考：https://github.com/jenkinsci/job-dsl-plugin/wiki/Tutorial---Using-the-Jenkins-Job-DSL

## Jenkins job builder
这是一个python的模块，安装之后就可以在脚本中控制生成需要的Job，跟DSL的区别是不需要在任务中去插入脚本，也可以控制所有的配置，使用yaml文件要定义配置文件，任务模板等等。

### 安装
源码安装
```
git clone https://github.com/openstack-infra/jenkins-job-builder
sudo python setup.py install
```
也可以通过pip安装
```python
pip install --user jenkins-job-builder
```

如果有提示pbr版本过低，可以用pip重新安装最新版本即可
```
pip install pbr
```

### sample
创建sample.yaml文件并输入如下代码片段
```
- job:
    name: Very basic freestyle job
    project-type: freestyle
    description: 'Just contais a very basic jenkins job that echos "Hello world"'
    builders:
      - shell: echo 'Hello world!'
```
然后添加配置文件，可以参考官网的片段：
```
[job_builder]
ignore_cache=True
keep_descriptions=False
include_path=.:scripts:~/git/
recursive=False
exclude=.*:manual:./development
allow_duplicates=False

[jenkins]
user=jenkins
password=1234567890abcdef1234567890abcdef
url=https://jenkins.example.com
query_plugins_info=False
##### This is deprecated, use job_builder section instead
#ignore_cache=True

[plugin "hipchat"]
authtoken=dummy

[plugin "stash"]
username=user
password=pass
```
可以测试一下sample文件有没有问题
```
jenkins-jobs test ./sample1.yaml
```
如果没有问题就可以更新到Jenkins上了,在更新之前请确保用户名密码以及URL的正确
```
jenkins-jobs update ./sample1.yaml
```
更多的sample细节可参考：http://www.jeeatwork.com/?p=182
Jenkins job builder的官网：https://docs.openstack.org/infra/jenkins-job-builder/
