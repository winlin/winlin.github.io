---
layout: post
title: "Git Learning Notes"
description: "contain the note of git"
category: GIT
tags: 
---
{% include JB/setup %}

本文章的内容摘自[Pro Git](http://git-scm.com/book/zh/%E8%B5%B7%E6%AD%A5-Git-%E5%9F%BA%E7%A1%80)

###直接记录快照，而非差异比较
Git 和其他版本控制系统的主要差别在于，Git 只关心文件数据的整体是否发生变化，而大多数其他系统则只关心文件内容的具体差异。这类系统（CVS，Subversion，Perforce，Bazaar 等等）每次记录有哪些文件作了更新，以及都更新了哪些行的什么内容，请看图 1-4。 

![](images/cvs_theory_1.png)
图 1-4. 其他系统在每个版本中记录着各个文件的具体差异 

Git 并不保存这些前后变化的差异数据。实际上，Git 更像是把变化的文件作快照后，记录在一个微型的文件系统中。每次提交更新时，它会纵览一遍所有文件的指纹信息并对文件作一快照，然后保存一个指向这次快照的索引。为提高性能，若文件没有变化，Git 不会再次保存，而只对上次保存的快照作一链接。Git 的工作方式就像图 1-5 所示。 

![](images/git_theory_1.png)
图 1-5. Git 保存每次更新时的文件快照 

这是 Git 同其他系统的重要区别。它完全颠覆了传统版本控制的套路，并对各个环节的实现方式作了新的设计。Git 更像是个小型的文件系统，但它同时还提供了许多以此为基础的超强工具，而不只是一个简单的 VCS。稍后在第三章讨论 Git 分支管理的时候，我们会再看看这样的设计究竟会带来哪些好处。 

