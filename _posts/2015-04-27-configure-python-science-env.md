---
layout: post
title: "Python科学计算环境配置"
description: ""
category: 编程
tags: [python]
---
{% include JB/setup %}


##1. 安装并配置pyenv

```bash
cd ~
git clone git://github.com/yyuu/pyenv.git .pyenv

echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init -)"' >> ~/.zshrc

exec $SHELL
```

##2. 使用pyenv安装不同python版本

```bash
#也可以安装其他版本
pyenv install miniconda-3.8.3

#查看版本
pyenv versions
pyenv global miniconda-3.8.3 #全局切换到该版本，局部切换使用local
```

##使用Miniconda安装科学计算环境

```bash
conda install numpy scipy matplotlib pandas scikit-learn ipython ipython-notebook PIL
```

参考[使用pyenv和miniconda配置科学计算环境](http://huangziwei.com/tech/setting-up-scientific-python-environment-in-os-x-10-10-using-miniconda/)

