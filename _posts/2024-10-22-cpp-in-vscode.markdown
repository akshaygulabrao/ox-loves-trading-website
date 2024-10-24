---
layout: post
title:  "a bottom-up approach to c++ in vscode"
date:   2024-10-22 14:43:00 -0400
categories: cpp
---
I wanted to do this post for a long time. VsCode is largely made for 2 languages: Python, Javascript. The [official guide](https://code.visualstudio.com/docs/languages/cpp) does not say things in the way I like to describe things. It's not bad, but just not intuitive to me because it uses a top-down approach instead of a bottom-up approach[^1]. So here is a bottom-up approach to getting C++ working in VsCode.

I think of VsCode as a mini-operating system with a shell, editor, and daemon processes all-in-one. The editor is obviously that you can view files in real-time. The shell is able to interface with the operating system. The daemon processes render the graphics on the file that you are currently editing. The `.vscode` is basically a shell configuration file that allows some scripting automation with the press of a button, similar to `~/.bashrc` or `~/.zshrc`. Installing those extensions adds the scripts and daemons which make coding in C++ significantly easier.

Only Python and Javascript come with build configurations baked into the program. For C++, install the "C/C++", "C/C++ Extenstion Pack","CMake Tools" extensions by the author `ms-vscode`. After doing so, you will need to reload the window using the VsCode Shell that is accessed thru "Cmd(Ctrl) + Shift + P" in MacOS(Windows). Then run the command "Python Debugger: Clear Cache and Reload Window". You can also just exit VSCode and open it again. Verify that this step was completed correctly by confirming the bottom toolbar has the (Gear Symbol) Build, (Bug Symbol), (Play Symbol) options.
<br/>
<br/>
<img src="/images/vscode.png" alt="options" style="border: 5px solid #000000;">

Now, one of the limitations to this guide is that it only supports CMAKE. I do believe that CMake is the best way to build packages with c++ because it is more readable than make. When you use the CMake Extension with vscode it automatically uses ninja to build, which uses a DAG[^2] to decide the build order more efficiently. Now when you click the gear icon, everything will be properly invoked which allows for easy debugging and running scripts.

DO NOT click on the items on the top right toolbar when editing a C++ file. Those are build systems linked to native building wihtout CMake.

[^1]:[explanation](https://www.reddit.com/r/explainlikeimfive/comments/106e5v1/comment/j3i1zyk/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button)
[^2]:[Directed Acyclic Graph](https://en.wikipedia.org/wiki/Dependency_graph)