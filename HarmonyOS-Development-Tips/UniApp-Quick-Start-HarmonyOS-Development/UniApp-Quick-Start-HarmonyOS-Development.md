# UniApp Quick Start Guide for HarmonyOS Development

UniApp team has supported HarmonyOS application development since version `4.28.2024092502`, and now with version `4.36.2024112817`, it supports both HarmonyOS applications and atomic services development.

Let's get hands-on experience now.

## Environment Setup

- HBuilderX 4.24+ [Download Link](https://www.dcloud.io/hbuilderx.html)
- [DevEco Studio](https://developer.huawei.com/consumer/cn/download/)
  - HBuilderX 4.24+ requires DevEco-Studio 5.0.3.400+, HBuilderX 4.31+ requires DevEco-Studio 5.0.3.800+
  - HarmonyOS system version API 12 and above (DevEco-Studio has built-in HarmonyOS simulator)
- **HBuilderX 4.31+ built HarmonyOS runtime packages do not support x86_64 platform, which affects HarmonyOS simulators on Windows systems and some Mac systems. Real device debugging is required**

**The demonstration below is mainly for real devices**

## Project Requirements

When creating a UniApp project, you need to select Vue3, as Vue2 is not supported.

## Setup Process

1. Create a new project on AGC platform to obtain bundleName and debugging/release certificates
2. Download and install DevEco Studio
3. Use **DevEco Studio** to create a project, then configure bundleName and debugging/release certificates
4. Copy certificate-related configurations
5. Download and install HBuilder
6. HBuilder downloads related plugins
7. HBuilder configures DevEco Studio tool path
8. HBuilder creates UniApp Vue3 project
9. HBuilder configures HarmonyOS application certificates
10. HBuilder runs the project

## Create New Project on AGC Platform

You can choose to create a new project or [application](https://developer.huawei.com/consumer/cn/doc/app/agc-help-createproject-0000001100334664) based on your needs. Here we choose a project.

## Download and Install DevEco Studio

Download and install [DevEco Studio](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/ide-software-install-V5)

## bundleName and Debugging/Release Certificates

Since real devices require debugging certificates when debugging.

Applications need release certificates when publishing, so we configure both at once.

[Configuration Link](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/application-dev-overview-V5#section42841246144813)

![image-20241219092436235](uniapp%E6%9E%81%E9%80%9F%E4%B8%8A%E6%89%8B%E9%B8%BF%E8%92%99%E5%BC%80%E5%8F%91.assets/image-20241219092436235.png)

## DevEco Studio Create Project to Get Certificate Configuration Information

This step is mainly to get certificate configuration code, which will be needed when UniApp runs the project.

After creating a project with DevEco Studio, refer to the link for [certificate configuration](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/ide-signing-V5#section297715173233)

Get the configuration file `build-profile.json5` and copy the entire code to the UniApp created project later.

![image-20241219093119586](uniapp%E6%9E%81%E9%80%9F%E4%B8%8A%E6%89%8B%E9%B8%BF%E8%92%99%E5%BC%80%E5%8F%91.assets/image-20241219093119586.png)

## Download and Install HBuilder

[Download and install here](https://www.dcloud.io/hbuilderx.html)

## HBuilder Download Related Plugins

`Tools - Plugin Installation` Key plugins are `HarmonyOS, Vue3`

![image-20241219093722816](uniapp%E6%9E%81%E9%80%9F%E4%B8%8A%E6%89%8B%E9%B8%BF%E8%92%99%E5%BC%80%E5%8F%91.assets/image-20241219093722816.png)

## HBuilder Configure DevEco Studio Tool Path

Configure DevEco Studio tool path here `Tools - Settings`

![image-20241219093840013](uniapp%E6%9E%81%E9%80%9F%E4%B8%8A%E6%89%8B%E9%B8%BF%E8%92%99%E5%BC%80%E5%8F%91.assets/image-20241219093840013.png)

## HBuilder Create UniApp Vue3 Project

Create Vue3 project

![image-20241219093925639](uniapp%E6%9E%81%E9%80%9F%E4%B8%8A%E6%89%8B%E9%B8%BF%E8%92%99%E5%BC%80%E5%8F%91.assets/image-20241219093925639.png)

## HBuilder Configure HarmonyOS Application Certificates

Configure `\harmony-configs\build-profile.json5` in the project root directory. If it doesn't exist, create it manually.

Then copy and paste the certificate code into it.

![image-20241219094156393](uniapp%E6%9E%81%E9%80%9F%E4%B8%8A%E6%89%8B%E9%B8%BF%E8%92%99%E5%BC%80%E5%8F%91.assets/image-20241219094156393.png)

## HBuilder Run Project

Finally, run the project

![image-20241219094225699](uniapp%E6%9E%81%E9%80%9F%E4%B8%8A%E6%89%8B%E9%B8%BF%E8%92%99%E5%BC%80%E5%8F%91.assets/image-20241219094225699.png)

## Result

![image-20241219094414865](uniapp%E6%9E%81%E9%80%9F%E4%B8%8A%E6%89%8B%E9%B8%BF%E8%92%99%E5%BC%80%E5%8F%91.assets/image-20241219094414865.png)
