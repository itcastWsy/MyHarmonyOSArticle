# 03-Quick Creation of Cangjie Projects

## Introduction

Cangjie projects have their own directory structure and file naming restrictions. If they are not created according to specifications, syntax highlighting and other plugins will be ineffective during development. Compilation and packaging may also fail.

## Creating Cangjie Projects

The Cangjie plugin previously installed on VSCode has built-in functionality for quickly creating Cangjie projects. There are two ways to create them:

### Visual Method

> In VSCode, press F1 or Ctrl + Shift + P (Command + Shift + P on Mac) to open the command palette, then follow these steps to create a Cangjie project:

1. Enter the keyword **Create Cangjie ..** to find the menu for creating new projects

   ![image-20241213060105633](03-快速创建仓颉工程.assets/image-20241213060105633.png)

2. Enter project information

   ![image-20241213060409063](03-快速创建仓颉工程.assets/image-20241213060409063.png)

3. Successfully created

   ![image-20241213060451980](03-快速创建仓颉工程.assets/image-20241213060451980.png)

### VSCode Terminal Command

> In VSCode, press F1 or Ctrl + Shift + P (Command + Shift + P on Mac) to open the command palette, then follow these steps to create a Cangjie project:

1. Select the create project command **cangjie:Create Cangjie Project** _without Project View_

   ![image-20241213060631546](03-快速创建仓颉工程.assets/image-20241213060631546.png)

2. Select compilation engine

   ![image-20241213061100309](03-快速创建仓颉工程.assets/image-20241213061100309.png)

3. Select output type

   ![image-20241213061106489](03-快速创建仓颉工程.assets/image-20241213061106489.png)

4. Select directory to store the project

   ![image-20241213061112564](03-快速创建仓颉工程.assets/image-20241213061112564.png)

5. Enter project name

   ![image-20241213061118990](03-快速创建仓颉工程.assets/image-20241213061118990.png)

6. Successfully created

   ![image-20241213061139871](03-快速创建仓颉工程.assets/image-20241213061139871.png)

## Project Directory Introduction

![image-20241213061504092](03-快速创建仓颉工程.assets/image-20241213061504092.png)

1. **.cache** - Automatically generated cache files
2. **.vscode** - Automatically generated editor settings files, only effective in the workspace
3. **src** - Business code area where we write code to import Cangjie's built-in libraries and get code suggestions
4. **cjpm.toml** - Project configuration file that records and sets project-related information

## Code Suggestions

Writing code in a legitimate project enables corresponding code suggestions:

![image-20241213061711657](03-快速创建仓颉工程.assets/image-20241213061711657.png)

## Compiling Projects

Quick project compilation:

![image-20241213062004350](03-快速创建仓颉工程.assets/image-20241213062004350.png)

## Summary

Cangjie development components have well-integrated functionality for creating and running projects. By following standard specifications, we can smoothly begin our first step in programming!
