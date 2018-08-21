---
title: GitLab持续集成/部署-Runner核心介绍
comments: false
date: 2018-08-17 16:01:03
categories: DevOps
tags: GitLab-CI
img:
---

# GitLab持续集成/部署-Runner核心介绍
<!-- TOC -->

- [GitLab持续集成/部署-Runner核心介绍](#gitlab%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90%E9%83%A8%E7%BD%B2-runner%E6%A0%B8%E5%BF%83%E4%BB%8B%E7%BB%8D)
    - [GitLab-Runner简介](#gitlab-runner%E7%AE%80%E4%BB%8B)
    - [GitLab-Runner架构图](#gitlab-runner%E6%9E%B6%E6%9E%84%E5%9B%BE)
    - [构成组件](#%E6%9E%84%E6%88%90%E7%BB%84%E4%BB%B6)
        - [Pipeline](#pipeline)
        - [Stages](#stages)
        - [Jobs](#jobs)
    - [扩展](#%E6%89%A9%E5%B1%95)
        - [Shared Runners如何挑选Jobs](#shared-runners%E5%A6%82%E4%BD%95%E6%8C%91%E9%80%89jobs)
            - [Example 1](#example-1)
            - [Example 2](#example-2)
    - [参考](#%E5%8F%82%E8%80%83)

<!-- /TOC -->

## GitLab-Runner简介
GitLab提供了自动集成、测试，部署的服务，这个动作由GitLab Runner来实现。
> GitLab CI 脚本的执行者

## GitLab-Runner架构图  

![image](https://raw.githubusercontent.com/ChaosCoffee/ChaosCoffee.github.io/master/img/GitLab-Runner.png)  

## 构成组件
### Pipeline
 > 官方解释: `pipeline `是一组分阶段执行的作业（批处理）。一个阶段中的所有作业都是并行执行的（如果有足够的并发Runners），如果它们都成功，那么`pipeline`就会进入下一个阶段。如果其中一个作业失败，则不会（通常）执行下一个阶段
 
一般来说，`Pipeline` 表示构建任务，里面可以包含多个流程，如安装依赖、运行测试、编译、部署测试服务器、部署生产服务器等流程,
任何提交或者 `Merge Request` 的合并都可以触发 `Pipeline`

### Stages
`Stages` 表示构建阶段，一次 Pipeline 中定义多个 `Stages`。
- 所有 `Stages` 会按照顺序运行，即当一个 `Stage` 完成后，下一个 `Stage` 才会开始
- 只有当所有 `Stages` 完成后，该构建任务 (`Pipeline`) 才会成功
- 如果任何一个 `Stage` 失败，那么后面的 `Stages` 不会执行，该构建任务 (`Pipeline`) 失败

### Jobs
`Jobs` 表示构建工作，表示某个 `Stage` 里面执行的工作。一个 `Stages` 里面定义多个 `Jobs`， `Jobs` 会有以下特点：
相同 `Stage` 中的 `Jobs` 会并行执行
相同 `Stage` 中的 `Jobs` 都执行成功时，该 `Stage` 才会成功
如果任何一个 `Job` 失败，那么该 `Stage` 失败，即该构建任务 (`Pipeline`) 失败

## 扩展  
### Shared Runners如何挑选Jobs
> `Shared Runners`遵守我们称之为合理使用的进程队列。公平使用算法尝试从当前在共享Runners上运行的作业数量最少的项目中将作业分配给共享的Runners。  
>  
---
#### Example 1  
 
We have following jobs in queue:  

- Job 1 for Project 1
- Job 2 for Project 1
- Job 3 for Project 1
- Job 4 for Project 2
- Job 5 for Project 2
- Job 6 for Project 3

jobs会按照以下顺序分配:
1. 首先选择`Job 1`，因为它没有正在运行的作业（即所有项目）的作业编号最低
2. 接下来是`Job 4`，因为4现在是没有正在运行的作业的项目中的最低Job 编号（项目1正在运行作业）
3. 接下来是`Job 6`，因为6现在是没有正在运行的Job 的项目中的最低Job 编号（项目1和2有作业正在运行）
4. 接下来是`Job 2`，因为在运行的Job 数量最少的项目中（每个都有1个），它是最低的Job 编号
5. 接下来是`Job 5`，因为项目1现在有2个Job 正在运行，而在项目2和3之间，`Job 5`是剩余的最低Job 编号
6. 最后，我们选择了`Job 3` ...因为这是唯一剩下的工作

---
#### Example 2  

We have following jobs in queue:

- Job 1 for Project 1
- Job 2 for Project 1
- Job 3 for Project 1
- Job 4 for Project 2
- Job 5 for Project 2
- Job 6 for Project 3

jobs会按照以下顺序分配:
1. 首先选择`Job 1`，因为它没有正在运行的作业（即所有项目）的作业编号最低
2. 我们完成了工作1
3. 接下来是`Job 2`，因为在完成`Job 1`后，所有项目都有0个作业再次运行，2是最低可用作业号
4. 接下来是`Job 4`，因为项目1运行作业时，4是没有作业的项目中的最小数字（`Project 2`和3）
5. 我们完成了工作4
6. 接下来是`Job 5`，因为完成了`Job 4`，`Job 2`没有再次运行作业
7. 接下来是`Job 6`，因为`Project 3`是唯一没有正在运行的作业的项目
8. 最后，我们选择了Job 3 ...因为，这是唯一剩下的工作（`who says 1 is the loneliest number?`）

---
## 参考  
[Gitlab-CI/CD 持续集成](http://laiyuquan.com/2018/04/gitlab-ci-cd-%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90/)  
[Introduction to pipelines and jobs
](https://docs.gitlab.com/ee/ci/pipelines.html)  
[Configuring GitLab Runners](https://docs.gitlab.com/ee/ci/runners/README.html#assigning-a-runner-to-another-project)