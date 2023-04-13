---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://gitlab.com/assets/illustrations/saas-trial-illustration-7081b7c29d89b705060d655cd385a18528e641ad0daf99595a2df4a4d8666292.svg
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# persist drawings in exports and build
drawings:
  persist: false
# use UnoCSS (experimental)
css: unocss
---

# Gitlab CI/CD 入坑指北

<br>
Speaker：Weibiao Tu

<div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    让你快速上手GitLab CI/CD <carbon:arrow-right class="inline"/>
  </span>
</div>

---

# GitLab CI/CD 是什么?

GitLab CI/CD 是 GitLab 的一部分，用于所有持续方法（持续集成、交付和部署）。使用 GitLab CI/CD ，可以在不需要第三方应用程序或集成的情况下测试、构建和发布软件。

GitLab CI/CD 是一种使用持续方法进行软件开发的工具：

- **持续集成 CI** - 对于每次推送到存储库，您都可以创建一组脚本来自动构建和测试应用程序。
- **持续交付 CD** - 这是 CI 之外的一步，应用程序也是持续部署的，通过持续交付，可以手动触发部署。
- **持续部署 CD** - 这是 CI 之外的另一步，类似于持续交付。通过持续部署，可以自动触发部署，无须人工干预。

<br>
<br>

Read more about [Gitlab CI/CD](https://docs.gitlab.com/ee/ci/introduction/)

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

---

# Pipeline 流水线

*Pipeline*是持续集成、交付和部署的顶级组件。

*Pipeline*包括：

- **Job** - 作业，定义要做什么。例如，编译或测试代码的作业。
- **Stage** - 阶段，定义何时运行作业。例如，在编译代码的阶段之后运行测试的阶段。

*Job*由*Runner*执行。如果有足够的并发运行程序，那么同一阶段中的多个作业将并行执行：

- 如果一个阶段中的所有作业都成功，则*Pipeline*将进入下一个阶段。
- 如果一个阶段中的有任何作业失败，则下一个阶段（通常）不会执行，并且*Pipeline*提前结束。

一个典型的*Pipeline*可能由四个阶段组成，按以下顺序执行：

- **build** - 包含一个*compile*的作业。
- **test** - 包含多个*test*的作业。
- **staging** - 包含一个名为*deploy to stage*的作业。
- **production** - 包含一个名为*deploy to prod*的作业。

---

# Runner & Executor

*Runner*是在流水线中运行作业的应用，与 GitLab CI/CD 配合运作。*Runner*既可以在 Host 中运行，也可以在 Docker Container 内运行。

*Runner*是运行来自 GitLab 的 CI/CD 作业的代理，在 GitLab 实例和安装了 GitLab Runner 的机器之间建立通信。
根据您想要允许访问的用户，分为三种类型的 Runner：

- **Shared Runner** - 允许所有项目使用。
- **Group Runner** - 允许指定群组下的所有项目和子组使用。
- **Project Runner** - 允许指定的个人项目使用。

*Executor*是在流水线中运行作业的最终执行者，注册*Runner*时，必须选择一个执行器。
*Executor*执行器决定每个作业运行的环境，例如：

- 如果您想要 CI/CD 作业运行 Powershell 命令，则可以在 Windows 服务器上安装极狐 GitLab Runner，然后注册使用 Shell 执行器的*Runner*
- 如果您想要 CI/CD 作业在自定义 Docker 容器中运行命令，您可以在 Linux 服务器上安装极狐 GitLab Runner，然后注册使用 Docker 执行器的 Runner。

---

# Variable 变量

CI/CD 变量是一种环境变量。 可以将它们用于：

- 控制作业和流水线。
- 存储要重复使用的值。
- 避免在您的 .gitlab-ci.yml 文件中硬编码值。

变量类型：

- _Predefined Variable_ - 预定义变量，包含有关作业、流水线的触发和运行信息，所有作业可用
- _Global Variable_ - 全局变量，在顶层使用 variables 关键字定义，所有作业可用
- _Job Variable_ - 作业变量，在作业中使用 variables 关键字定义，仅作业可用
- _Project Variable_ - 项目变量，在项目 CI/CD-Variables 设置中添加，仅该项目可用
- _Group Variable_ - 群组变量，在群组 CI/CD-Variables 设置中添加，该群组下所有项目可用
- _Instance Variable_ - 实例变量，在 Admin CI/CD-Variables 设置中添加，该实例的群组和所有项目可用

---

# Variable 变量使用 1

变量使用：

- GitLab 端，在 .gitlab-ci.yml 中。
- GitLab Runner 端，在 config.toml 中。

扩展机制：

- GitLab 端
- GitLab Runner
- 执行 shell 环境

---

# Variable 变量使用 2

注意事项：

- 在作业中屏蔽全局变量时，请使用 variables: {}，而不是 unset 或者覆盖它
- 在作业中根据条件覆盖已声明的全局变量并且仍然需要作用域为全局时，需要在 workflow 中重新声明以覆盖
- 在 Script 中使用变量时，export/unset 变量并不能按照预期覆盖或删除全局变量，仅该 Script 有作用
- 在.gitlab-ci.yml 文件中使用变量，要多尝试，文档可能为直观表明，例如 artifacts:path 中使用
- 在不同的关键字中使用变量，变量的展开机制可能会不同，需要先查明变量的展开方式，避免出现副作用
- 展开可能发声在作业之前，如 Runner 尝试将变量展开传递给作业，也可能发生在作业运行时如 Script 动态
- 嵌套地定义变量时，Runner 按照定义顺序依次展开变量，需要定义好变量的顺序，避免发生空值等异常值

---

# Learn More

[GitLab](https://docs.gitlab.cn/jh/ci/)
