# 02-Cangjie Development Environment Setup

This chapter mainly introduces how to set up a development environment for running Cangjie.

We can divide the development environment for running Cangjie language into 3 types:

1. Online execution provided by the official Cangjie website, the lowest-cost way to experience and run Cangjie programs.
2. Download and install Cangjie SDK, use VSCode to install Cangjie syntax highlighting plugin, develop Cangjie programs
3. Install Cangjie plugin in **DevEco Studio**, develop Cangjie programs

## Cangjie Language Online Experience

Open [https://cangjie-lang.cn/](https://cangjie-lang.cn/), click **Online Experience**

![image-20241211180435243](02-仓颉开发环境搭建.assets/image-20241211180435243.png)

---

---

![image-20241211180518909](02-仓颉开发环境搭建.assets/image-20241211180518909.png)

## Install Cangjie Plugin in VSCode

### Download and Install VSCode

First, we need to download the VSCode editing tool. Visual Studio Code (abbreviated as "VS Code") was officially announced by Microsoft at the Build developer conference on April 30, 2015. It is a cross-platform source code editor running on Mac OS X, Windows, and Linux, designed for writing modern Web and cloud applications.

[Download Link](https://code.visualstudio.com/)

![image-20241211231632255](02-仓颉开发环境搭建.assets/image-20241211231632255.png)

After successful installation, this is what it looks like when opened.

![image-20241211232846027](02-仓颉开发环境搭建.assets/image-20241211232846027.png)

### Download and Install Cangjie Plugin

[Download Link](https://cangjie-lang.cn/download)

![image-20241211233119666](02-仓颉开发环境搭建.assets/image-20241211233119666.png)

It's recommended to download the stable version to avoid some pitfalls.

![image-20241211233139036](02-仓颉开发环境搭建.assets/image-20241211233139036.png)

There are two items to download:

1. SDK
2. VSCode plugin

![image-20241211233546478](02-仓颉开发环境搭建.assets/image-20241211233546478.png)

### Install Cangjie SDK

After downloading the Cangjie SDK, double-click to install it directly.

![image-20241211233632785](02-仓颉开发环境搭建.assets/image-20241211233632785.png)

Open your terminal and enter the command to check if the Cangjie SDK is installed successfully:

```
cjc -v
```

![image-20241211233750170](02-仓颉开发环境搭建.assets/image-20241211233750170.png)

### Install VSCode Cangjie Plugin

This plugin is used to make VSCode recognize Cangjie syntax and run Cangjie programs.

![image-20241211234341812](02-仓颉开发环境搭建.assets/image-20241211234341812.png)

Extract it to get the following program.

![image-20241211234620705](02-仓颉开发环境搭建.assets/image-20241211234620705.png)

---

Install offline plugin in VSCode

![image-20241211234427504](02-仓颉开发环境搭建.assets/image-20241211234427504.png)

Installation successful

![image-20241211234916982](02-仓颉开发环境搭建.assets/image-20241211234916982.png)

At this point, you need to configure Cangjie SDK in VSCode, otherwise you won't be able to run Cangjie programs directly later.

![image-20241212000638058](02-仓颉开发环境搭建.assets/image-20241212000638058.png)

---

![image-20241212000808737](02-仓颉开发环境搭建.assets/image-20241212000808737.png)

So far, our VSCode development environment is all set up.

## Install Cangjie Plugin in **DevEco Studio** to Develop Cangjie Programs

If you want to install the Cangjie plugin in **DevEco Studio** to develop Cangjie programs, you need to apply first. [Application Link](https://developer.huawei.com/consumer/cn/activityDetail/cangjie-beta/)

After approval, an email will be sent to your mailbox. Follow the email instructions. Since this process is confidential, the specific steps won't be demonstrated.

After successful installation, the Cangjie plugin option will be displayed in **DevEco Studio**.

![image-20241212000332204](02-仓颉开发环境搭建.assets/image-20241212000332204.png)

## Our First Cangjie Program

Next, let's use VSCode to develop our first Cangjie program!

1. Create a new Cangjie file with the extension **.cj**

   ![image-20241212000847946](02-仓颉开发环境搭建.assets/image-20241212000847946.png)

2. Enter basic code

   ```c
   main() {
       println("Hello World")
   }
   ```

3. Compile the Cangjie program

   Open the terminal and enter the following command

   > cjc represents using the built-in command of the installed SDK to compile Cangjie programs
   >
   > .\01-hello.cj is the file to be compiled
   >
   > -o hello.exe means the compiled content is generated to hello.exe

   ```js
    cjc .\01-hello.cj  -o hello.exe
   ```

   ![image-20241212001524176](02-仓颉开发环境搭建.assets/image-20241212001524176.png)

4. Run the Cangjie program. At this point, enter the command in the terminal to run our Cangjie program:

   ```
    .\hello.exe
   ```

   ![image-20241212001606695](02-仓颉开发环境搭建.assets/image-20241212001606695.png)

5. You can also directly use VSCode's built-in run function for one-click compilation and execution

   ![image-20241212001830626](02-仓颉开发环境搭建.assets/image-20241212001830626.png)

---

​ Build and debug single file

​ ![image-20241212002037085](02-仓颉开发环境搭建.assets/image-20241212002037085.png)

---

Run successfully

![image-20241212002125620](02-仓颉开发环境搭建.assets/image-20241212002125620.png)

## Summary

The above are our 3 ways to set up the Cangjie development environment. The next chapter will start introducing basic syntax. Stay tuned.
