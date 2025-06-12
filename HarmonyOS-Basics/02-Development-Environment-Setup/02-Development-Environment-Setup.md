# 02-Development Environment Setup

The preparation of HarmonyOS development environment mainly consists of the following steps:

1. Register as a developer
2. Real-name authentication
3. Create application
4. Download and install development tools
5. Create new project

# Register as a Developer

On the Huawei Developer Alliance website, [register as a developer](https://developer.huawei.com/consumer/cn/doc/start/registration-and-verification-0000001053628148) and complete [real-name authentication](https://developer.huawei.com/consumer/cn/doc/start/rna-0000001062530373).

1. Open [Huawei Developer Alliance official website](https://developer.huawei.com/consumer/cn/), click "Register" to enter the registration page.

2. You can register for a Huawei Developer Alliance account using an email address or mobile phone number.

If you register with an email address, please enter the correct email address and verification code, set a password, and click "**Register**".

![image-20241205232134712](02-开发环境搭建.assets/image-20241205232134712.png)

If you register using a mobile phone number, please enter the correct mobile phone number and verification code, set a password, and click "**Register**".

![image-20241205232154688](02-开发环境搭建.assets/image-20241205232154688.png)

3. If you agree to the [Huawei Account and Cloud Space Privacy Statement](https://id1.cloud.huawei.com/AMW/portal/agreements/accPrivacyStatement/zh-CN_accPrivacyStatement.html) and [Huawei Account and Cloud Space User Agreement](https://id1.cloud.huawei.com/AMW/portal/agreements/userAgreement/zh-CN_userAgreement.html?version=china), click "Agree". After successful registration, the real-name authentication page will be displayed.

# Real-Name Authentication

Real-name authentication is divided into individual authentication and enterprise authentication. The differences are as follows:

Enterprise developers enjoy more services than individual developers, as detailed in the following table:

| **Developer Type**   | **Services/Benefits Enjoyed**                                                                                                                                                                                                                                       |
| :------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Individual Developer | App Gallery, Themes, Product Management, Account, PUSH, New Game Reservation, Interactive Comments, Social, HUAWEI HiAI, Watch App Gallery, etc.                                                                                                                    |
| Enterprise Developer | App Gallery, Themes, First Release, Payment, Game Gift Packs, App Gallery Promotion, Product Management, Games, Account, PUSH, New Game Reservation, Interactive Comments, Social, HUAWEI HiAI, Watch App Gallery, Sports & Health, Cloud Testing, Smart Home, etc. |

## How Individual Developers Complete Real-Name Authentication

Individual developer real-name authentication is divided into four methods: personal bank card authentication, ID card manual review authentication, Huawei Cloud authorization authentication, and facial recognition authentication. The overall process for personal bank card authentication, ID card manual review authentication, and facial recognition authentication is as follows:

![image-20241205232355143](02-开发环境搭建.assets/image-20241205232355143.png)

The overall process for Huawei Cloud authorization authentication is as follows:

![image-20241205232404549](02-开发环境搭建.assets/image-20241205232404549.png)

- Choose facial recognition authentication method, see: [Facial Recognition Authentication](https://developer.huawei.com/consumer/cn/doc/start/facial-0000001271652446).
- Choose personal bank card authentication method, see: [Personal Bank Card Authentication](https://developer.huawei.com/consumer/cn/doc/start/ibca-0000001062388135).
- Choose manual review authentication method, see: [ID Card Manual Review Authentication](https://developer.huawei.com/consumer/cn/doc/start/micraa-0000001062276335).
- Choose Huawei Cloud authorization authentication method, see: [Huawei Cloud Authorization Authentication](https://developer.huawei.com/consumer/cn/doc/start/cloudrz-0000001200842383).

# Create Application

On [AppGallery Connect](https://developer.huawei.com/consumer/cn/service/josp/agc/index.html) (AGC), refer to [Create Project](https://developer.huawei.com/consumer/cn/doc/distribution/app/agc-help-createproject-0000001100334664) and [Create Application](https://developer.huawei.com/consumer/cn/doc/app/agc-help-createharmonyapp-0000001945392297) to complete the creation of **HarmonyOS** applications, enabling the use of various services.

One project can create multiple applications. This step is essential for later publishing and launching. If it's in the development stage, it can be omitted (development of AtomicServices also requires creating an application first).

## Create Project

1. Log in to [AppGallery Connect](https://developer.huawei.com/consumer/cn/service/josp/agc/index.html#/), click "My Projects".

2. On the project page, click "Add Project".

   ![img](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20241028162622.38612997838810625809584429697787:50001231000000:2800:4C717B1CDB0E0210A10FB964D7A70E5AA91387427E30785B2C2D51444D5692A2.png?needInitFileName=true?needInitFileName=true)

3. On the "Create Project" page, enter the project name and click "Create and Continue".

   ![img](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20241028162622.86813914295752710861848372737793:50001231000000:2800:AA437AD1625B917CC6647423852BFB05BB3E7C1266EE55272D261BC9DDB2307D.png?needInitFileName=true?needInitFileName=true)

4. After the project is created, you will enter the "Enable Analytics Service" page. The "Enable analytics service for this project" switch is turned on by default.

   - If your created project does not need to use Huawei Analytics service, turn off "Enable analytics service for this project" and click "Done" to complete the project creation.

     ![img](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20241028162622.19002787247306592402864234686403:50001231000000:2800:97BB8FAF824BD9F61E8755D2694A74C86E4EC8D38661A16273E0586B04FF41CD.png?needInitFileName=true?needInitFileName=true)

## Create Application

You can create applications and AtomicServices here.

To publish HarmonyOS applications/AtomicServices on AGC, you first need to create HarmonyOS applications/AtomicServices to generate a unique APP ID for HarmonyOS applications/AtomicServices.

1. Select **Certificates, APP ID and Profile**

   ![img](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20241127175411.15913325865515383710200338391669:50001231000000:2800:9E00DCE084E846EFFC1AB875614C220CFEF0DBAB116F8C7FF75C6917BD8A5399.png?needInitFileName=true?needInitFileName=true)

2. In the left navigation bar, select "Certificates, APP ID and Profile > APP ID", enter the "APP ID" page, and click "New" in the upper right corner.

   ![img](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20241127175411.07663264428552030954372289484555:50001231000000:2800:A74859F170F42D53CFBC108CF21FD76B252D367892F0574B6B14775A23542111.png?needInitFileName=true?needInitFileName=true)

3. Enter the "Set Application Development Basic Information" page, fill in the application basic information, and click "Next" after completion.

   ![img](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20241127175411.15593407155300228610291966651939:50001231000000:2800:5F9D852A74975722BFFD69DD5A917FC2AE9CB67FD784B2CC5FC4C45AEA0A4E35.png?needInitFileName=true?needInitFileName=true)

# Download and Install Development Tools

[Download development tools here](https://developer.huawei.com/consumer/cn/download/)

![image-20241205232941588](02-开发环境搭建.assets/image-20241205232941588.png)

HarmonyOS application development tool requirements:

To ensure DevEco Studio runs normally, it is recommended that the computer configuration meets the following requirements:

- Operating System: Windows 10 64-bit, Windows 11 64-bit
- Memory: 16GB and above
- Hard Disk: 100GB and above
- Resolution: 1280\*800 pixels and above

After downloading the installation package, open it directly to install.

![image-20241205233049765](02-开发环境搭建.assets/image-20241205233049765.png)

# Create and Run

After DevEco Studio installation is complete, you can verify whether the environment setup is correct by running a Hello World project. Next, we'll introduce how to create a project that supports Phone devices as an example.

## Create a New Project

1. Open DevEco Studio, click **Create Project** on the welcome page to create a new project.

2. According to the project creation wizard, choose to create **Application** or **Atomic Service**. Select the **Empty Ability** template, then click **Next**. For introduction to project templates and supported device types, please refer to [Project Template Introduction](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/ide-template-V5).

   ![img](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20241120180041.57682702319051059898404111840871:50001231000000:2800:750EB2B3BB504BD901138AEA485A30358A6B2F421ABCB298CA58BBF55380E630.png?needInitFileName=true?needInitFileName=true)

3. Fill in the project-related information and click **Finish**. For detailed introduction of each parameter, please refer to [Create a New Project](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/ide-create-new-project-V5).

   ![img](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20241120180041.63293130422053139410924648832209:50001231000000:2800:54D1D874D1544E839F193991E3909D3760CE301CC88DF677001D242D02B95F47.png?needInitFileName=true?needInitFileName=true)

   After the project is created, DevEco Studio will automatically synchronize the project.

## Run Hello World

1. Connect a real device with HarmonyOS system to the computer. For specific guidance and requirements, please refer to [Run Application/AtomicService](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/ide-running-app-V5).

2. Click **File** > **Project Structure...** > **Project** > **SigningConfigs** interface, check "**Support HarmonyOS**" and "**Automatically generate signature**", click "**Sign In**" as prompted by the interface, and log in with your Huawei account. After waiting for automatic signing to complete, click "**OK**". As shown below:

   ![img](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20241120180041.57904506421101917816417734235362:50001231000000:2800:136E017FC82F5B079F1A21015A6ABE4333DF3C379829CAE2295E99790E5065EA.png?needInitFileName=true?needInitFileName=true)

3. In the toolbar in the upper right corner of the editing window, click the ![img](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20241120180041.73236988803962067536801751006326:50001231000000:2800:6C9A6403301A47C122092A9A0A7C1969174861D1F60CA565E7D7A1F0D6418F9F.png?needInitFileName=true?needInitFileName=true) button to run. The effect is shown below:

   ![img](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20241120180041.63295744196955017470339832561465:50001231000000:2800:52709C88489C3A0C6E41CDB72835CDDE130124781FDCF03D76528CB952A7E26D.png?needInitFileName=true?needInitFileName=true)

Congratulations! You have successfully run your first application.

# Editor Tool Related Settings

[For settings like Chinese language and keyboard shortcuts, please refer to this article](https://blog.csdn.net/u013176440/article/details/139513030)
