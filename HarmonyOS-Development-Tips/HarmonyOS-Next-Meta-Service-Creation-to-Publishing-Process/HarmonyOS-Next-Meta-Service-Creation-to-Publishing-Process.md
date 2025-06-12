# HarmonyOS Next Atomic Service Complete Workflow: From Creation to Publishing

Following the previous article, this guide introduces the complete workflow of HarmonyOS Next atomic services from creation to publishing on the app store.

# HarmonyOS Next Atomic Service Complete Workflow: From Creation to Publishing

Following the previous article.

The main purpose of this article is to introduce the complete workflow of atomic services from creation to publishing.

# Create a New Project on AGC Platform

[Link](https://developer.huawei.com/consumer/cn/service/josp/agc/index.html#/harmonyOSDevPlatform/172249065903274453)

One project can contain multiple applications.

![image-20241124191241104](https://wsy996.obs.cn-east-3.myhuaweicloud.com/HarmonyOS%20Next%20%E7%AE%80%E5%8D%95%E4%B8%8A%E6%89%8B%E5%85%83%E6%9C%8D%E5%8A%A1%E5%BC%80%E5%8F%91/HarmonyOS%20Next%20%E7%AE%80%E5%8D%95%E4%B8%8A%E6%89%8B%E5%85%83%E6%9C%8D%E5%8A%A1%E5%BC%80%E5%8F%91.assets/image-20241124191241104.png?x-image-process=style/style-8860)

# Create a New Atomic Service Application in AGC

![image-20241124191300505](https://wsy996.obs.cn-east-3.myhuaweicloud.com/HarmonyOS%20Next%20%E7%AE%80%E5%8D%95%E4%B8%8A%E6%89%8B%E5%85%83%E6%9C%8D%E5%8A%A1%E5%BC%80%E5%8F%91/HarmonyOS%20Next%20%E7%AE%80%E5%8D%95%E4%B8%8A%E6%89%8B%E5%85%83%E6%9C%8D%E5%8A%A1%E5%BC%80%E5%8F%91.assets/image-20241124191300505.png?x-image-process=style/style-8860)

# Create a New Local Atomic Service Project

![image-20241124191643071](https://wsy996.obs.cn-east-3.myhuaweicloud.com/HarmonyOS%20Next%20%E7%AE%80%E5%8D%95%E4%B8%8A%E6%89%8B%E5%85%83%E6%9C%8D%E5%8A%A1%E5%BC%80%E5%8F%91/HarmonyOS%20Next%20%E7%AE%80%E5%8D%95%E4%B8%8A%E6%89%8B%E5%85%83%E6%9C%8D%E5%8A%A1%E5%BC%80%E5%8F%91.assets/image-20241124191643071.png?x-image-process=style/style-8860)

---

If you have successfully created an atomic service on the AGC platform, it will be displayed automatically here.

![image-20241124191744693](https://wsy996.obs.cn-east-3.myhuaweicloud.com/HarmonyOS%20Next%20%E7%AE%80%E5%8D%95%E4%B8%8A%E6%89%8B%E5%85%83%E6%9C%8D%E5%8A%A1%E5%BC%80%E5%8F%91/HarmonyOS%20Next%20%E7%AE%80%E5%8D%95%E4%B8%8A%E6%89%8B%E5%85%83%E6%9C%8D%E5%8A%A1%E5%BC%80%E5%8F%91.assets/image-20241124191744693.png?x-image-process=style/style-8860)

# Modify Atomic Service Name

![image-20241124191327227](https://wsy996.obs.cn-east-3.myhuaweicloud.com/HarmonyOS%20Next%20%E7%AE%80%E5%8D%95%E4%B8%8A%E6%89%8B%E5%85%83%E6%9C%8D%E5%8A%A1%E5%BC%80%E5%8F%91/HarmonyOS%20Next%20%E7%AE%80%E5%8D%95%E4%B8%8A%E6%89%8B%E5%85%83%E6%9C%8D%E5%8A%A1%E5%BC%80%E5%8F%91.assets/image-20241124191327227.png?x-image-process=style/style-8860)

# Modify Atomic Service Icon

Important: Publishing review is very strict.

![image-20241124191343134](https://wsy996.obs.cn-east-3.myhuaweicloud.com/HarmonyOS%20Next%20%E7%AE%80%E5%8D%95%E4%B8%8A%E6%89%8B%E5%85%83%E6%9C%8D%E5%8A%A1%E5%BC%80%E5%8F%91/HarmonyOS%20Next%20%E7%AE%80%E5%8D%95%E4%B8%8A%E6%89%8B%E5%85%83%E6%9C%8D%E5%8A%A1%E5%BC%80%E5%8F%91.assets/image-20241124191343134.png?x-image-process=style/style-8860)

1. First download any image yourself

2. Use Paint tool - Image Properties - Modify to 1024px

   ![PixPin_2024-11-24_19-15-16](https://wsy996.obs.cn-east-3.myhuaweicloud.com/HarmonyOS%20Next%20%E7%AE%80%E5%8D%95%E4%B8%8A%E6%89%8B%E5%85%83%E6%9C%8D%E5%8A%A1%E5%BC%80%E5%8F%91/HarmonyOS%20Next%20%E7%AE%80%E5%8D%95%E4%B8%8A%E6%89%8B%E5%85%83%E6%9C%8D%E5%8A%A1%E5%BC%80%E5%8F%91.assets/PixPin_2024-11-24_19-15-16.gif?x-image-process=style/style-8860)

3. Use Image Asset in the development tool to create the image

![image-20241124191814129](https://wsy996.obs.cn-east-3.myhuaweicloud.com/HarmonyOS%20Next%20%E7%AE%80%E5%8D%95%E4%B8%8A%E6%89%8B%E5%85%83%E6%9C%8D%E5%8A%A1%E5%BC%80%E5%8F%91/HarmonyOS%20Next%20%E7%AE%80%E5%8D%95%E4%B8%8A%E6%89%8B%E5%85%83%E6%9C%8D%E5%8A%A1%E5%BC%80%E5%8F%91.assets/image-20241124191814129.png?x-image-process=style/style-8860)

![image-20241124192132959](https://wsy996.obs.cn-east-3.myhuaweicloud.com/HarmonyOS%20Next%20%E7%AE%80%E5%8D%95%E4%B8%8A%E6%89%8B%E5%85%83%E6%9C%8D%E5%8A%A1%E5%BC%80%E5%8F%91/HarmonyOS%20Next%20%E7%AE%80%E5%8D%95%E4%B8%8A%E6%89%8B%E5%85%83%E6%9C%8D%E5%8A%A1%E5%BC%80%E5%8F%91.assets/image-20241124192132959.png?x-image-process=style/style-8860)

# Publishing Process

![image-20241127095335272](HarmonyOS%20Next%20%E5%85%83%E6%9C%8D%E5%8A%A1%E6%96%B0%E5%BB%BA%E5%88%B0%E4%B8%8A%E6%9E%B6%E5%85%A8%E6%B5%81%E7%A8%8B.assets/image-20241127095335272.png)

# Introduction to Signing Files

- **Key**: Contains public and private keys used in asymmetric encryption, stored in keystore files with format **.p12**. Public and private key pairs are used for digital signing and verification.
- **Certificate Signing Request**: Format **.csr**, full name Certificate Signing Request, contains the public key from the key pair and common name, organization name, organizational unit information, used to apply for digital certificates from AppGallery Connect.
- **Digital Certificate**: Format **.cer**, issued by Huawei AppGallery Connect.
- **Profile File**: Format **.p7b**, contains HarmonyOS application/atomic service package name, digital certificate information, describes the certificate permission list that applications/atomic services are allowed to request, and the device list that allows application/atomic service debugging (if the application/atomic service type is Release type, the device list is empty) and other content. Each application/atomic service package must contain a Profile file.

Among these, multiple atomic services can share **.p12**, **.csr**, **.cer** files. That means **.p7b** needs to be generated separately for each project.

# Generate Key and Certificate Signing Request Files

> This operation will produce two files

![image-20241127102037596](HarmonyOS%20Next%20%E5%85%83%E6%9C%8D%E5%8A%A1%E6%96%B0%E5%BB%BA%E5%88%B0%E4%B8%8A%E6%9E%B6%E5%85%A8%E6%B5%81%E7%A8%8B.assets/image-20241127102037596.png)

---

![image-20241127102233557](HarmonyOS%20Next%20%E5%85%83%E6%9C%8D%E5%8A%A1%E6%96%B0%E5%BB%BA%E5%88%B0%E4%B8%8A%E6%9E%B6%E5%85%A8%E6%B5%81%E7%A8%8B.assets/image-20241127102233557.png)

---

![image-20241127103001436](HarmonyOS%20Next%20%E5%85%83%E6%9C%8D%E5%8A%A1%E6%96%B0%E5%BB%BA%E5%88%B0%E4%B8%8A%E6%9E%B6%E5%85%A8%E6%B5%81%E7%A8%8B.assets/image-20241127103001436.png)

# Apply for Publishing Certificate and Profile Files

> This operation will also produce two files

## Steps to Apply for Publishing Certificate:

1. Log in to [AppGallery Connect](https://developer.huawei.com/consumer/cn/service/josp/agc/index.html#/), select **"Certificates, APP ID and Profile"**.

   ![image-20241127111351966](HarmonyOS%20Next%20%E5%85%83%E6%9C%8D%E5%8A%A1%E6%96%B0%E5%BB%BA%E5%88%B0%E4%B8%8A%E6%9E%B6%E5%85%A8%E6%B5%81%E7%A8%8B.assets/image-20241127111351966.png)

2. In the left navigation bar, select **"Certificates, APP ID and Profile > Certificates"**, enter the "Certificates" page, click **"Add Certificate"**.

   ![img](HarmonyOS%20Next%20%E5%85%83%E6%9C%8D%E5%8A%A1%E6%96%B0%E5%BB%BA%E5%88%B0%E4%B8%8A%E6%9E%B6%E5%85%A8%E6%B5%81%E7%A8%8B.assets/0000000000011111111.20241108162413.21428389402407934282631586754603.png)

3. In the pop-up "Add Certificate" window, fill in the certificate information to apply for, click "Submit".

   ![image-20241127103329588](HarmonyOS%20Next%20%E5%85%83%E6%9C%8D%E5%8A%A1%E6%96%B0%E5%BB%BA%E5%88%B0%E4%B8%8A%E6%9E%B6%E5%85%A8%E6%B5%81%E7%A8%8B.assets/image-20241127103329588.png)

4. Download the .cer file

   ![image-20241127103441091](HarmonyOS%20Next%20%E5%85%83%E6%9C%8D%E5%8A%A1%E6%96%B0%E5%BB%BA%E5%88%B0%E4%B8%8A%E6%9E%B6%E5%85%A8%E6%B5%81%E7%A8%8B.assets/image-20241127103441091.png)

5. **Obtain Publishing Certificate**

![image-20241127103512816](HarmonyOS%20Next%20%E5%85%83%E6%9C%8D%E5%8A%A1%E6%96%B0%E5%BB%BA%E5%88%B0%E4%B8%8A%E6%9E%B6%E5%85%A8%E6%B5%81%E7%A8%8B.assets/image-20241127103512816.png)

## Apply for Publishing Profile

Profile format is .p7b, contains HarmonyOS application/atomic service package name, digital certificate information, HarmonyOS application/atomic service allowed certificate permission list, and device list that allows application/atomic service debugging (if application/atomic service type is Release type, device list is empty) and other content. Each HarmonyOS application/atomic service package must contain a Profile file.

Steps to apply for publishing Profile:

1. Log in to AppGallery Connect, select **Certificates, APP ID and Profile**

   ![img](HarmonyOS%20Next%20%E5%85%83%E6%9C%8D%E5%8A%A1%E6%96%B0%E5%BB%BA%E5%88%B0%E4%B8%8A%E6%9E%B6%E5%85%A8%E6%B5%81%E7%A8%8B.assets/0000000000011111111.20241108162413.94137298091598438736514604862804.png)

2. In the left navigation bar, select **"Certificates, APP ID and Profile > Profile"**, enter the **"Profile"** page, click **"Add"** in the upper right corner.

   ![img](HarmonyOS%20Next%20%E5%85%83%E6%9C%8D%E5%8A%A1%E6%96%B0%E5%BB%BA%E5%88%B0%E4%B8%8A%E6%9E%B6%E5%85%A8%E6%B5%81%E7%A8%8B.assets/0000000000011111111.20241108162413.37859214279609577220399148607883.png)

3. On the **"Add Profile"** page, fill in the **Profile** information, click "Add" when complete.

   ![image-20241127103940121](HarmonyOS%20Next%20%E5%85%83%E6%9C%8D%E5%8A%A1%E6%96%B0%E5%BB%BA%E5%88%B0%E4%B8%8A%E6%9E%B6%E5%85%A8%E6%B5%81%E7%A8%8B.assets/image-20241127103940121.png)

4. Download Profile

   ![image-20241127104028586](HarmonyOS%20Next%20%E5%85%83%E6%9C%8D%E5%8A%A1%E6%96%B0%E5%BB%BA%E5%88%B0%E4%B8%8A%E6%9E%B6%E5%85%A8%E6%B5%81%E7%A8%8B.assets/image-20241127104028586.png)

5. Obtain Profile File

   ![image-20241127104100616](HarmonyOS%20Next%20%E5%85%83%E6%9C%8D%E5%8A%A1%E6%96%B0%E5%BB%BA%E5%88%B0%E4%B8%8A%E6%9E%B6%E5%85%A8%E6%B5%81%E7%A8%8B.assets/image-20241127104100616.png)

# Manual Signing

Configure your atomic service to use the certificates created above for manual signing.

![image-20241127104421049](HarmonyOS%20Next%20%E5%85%83%E6%9C%8D%E5%8A%A1%E6%96%B0%E5%BB%BA%E5%88%B0%E4%B8%8A%E6%9E%B6%E5%85%A8%E6%B5%81%E7%A8%8B.assets/image-20241127104421049.png)

# Package Build

![image-20241127104501011](HarmonyOS%20Next%20%E5%85%83%E6%9C%8D%E5%8A%A1%E6%96%B0%E5%BB%BA%E5%88%B0%E4%B8%8A%E6%9E%B6%E5%85%A8%E6%B5%81%E7%A8%8B.assets/image-20241127104501011.png)

Obtain APP file

![image-20241127104552646](HarmonyOS%20Next%20%E5%85%83%E6%9C%8D%E5%8A%A1%E6%96%B0%E5%BB%BA%E5%88%B0%E4%B8%8A%E6%9E%B6%E5%85%A8%E6%B5%81%E7%A8%8B.assets/image-20241127104552646.png)

# Create New Release

Return to AGC platform, create new release

![image-20241127104809629](HarmonyOS%20Next%20%E5%85%83%E6%9C%8D%E5%8A%A1%E6%96%B0%E5%BB%BA%E5%88%B0%E4%B8%8A%E6%9E%B6%E5%85%A8%E6%B5%81%E7%A8%8B.assets/image-20241127104809629.png)

# Edit Release Information

![image-20241127104905347](HarmonyOS%20Next%20%E5%85%83%E6%9C%8D%E5%8A%A1%E6%96%B0%E5%BB%BA%E5%88%B0%E4%B8%8A%E6%9E%B6%E5%85%A8%E6%B5%81%E7%A8%8B.assets/image-20241127104905347.png)

---

![image-20241127105255233](HarmonyOS%20Next%20%E5%85%83%E6%9C%8D%E5%8A%A1%E6%96%B0%E5%BB%BA%E5%88%B0%E4%B8%8A%E6%9E%B6%E5%85%A8%E6%B5%81%E7%A8%8B.assets/image-20241127105255233.png)

---

![image-20241127105420075](HarmonyOS%20Next%20%E5%85%83%E6%9C%8D%E5%8A%A1%E6%96%B0%E5%BB%BA%E5%88%B0%E4%B8%8A%E6%9E%B6%E5%85%A8%E6%B5%81%E7%A8%8B.assets/image-20241127105420075.png)

# Domain Name Filing

At this point, if your application is not filed, it will be rejected. The filing here means you need both a filed domain name and a filed atomic service.

One root domain can correspond to multiple atomic services. For example:

a.baidu.com - Atomic Service A

b.baidu.com - Atomic Service B

![image-20241127105504102](HarmonyOS%20Next%20%E5%85%83%E6%9C%8D%E5%8A%A1%E6%96%B0%E5%BB%BA%E5%88%B0%E4%B8%8A%E6%9E%B6%E5%85%A8%E6%B5%81%E7%A8%8B.assets/image-20241127105504102.png)

Filing is also a major step for beginners, so if you really want to publish an application, it's better to file first.

[Tencent Cloud Reference Link:](https://juejin.cn/post/6844903840165134344#heading-5)

![image-20241127105615225](HarmonyOS%20Next%20%E5%85%83%E6%9C%8D%E5%8A%A1%E6%96%B0%E5%BB%BA%E5%88%B0%E4%B8%8A%E6%9E%B6%E5%85%A8%E6%B5%81%E7%A8%8B.assets/image-20241127105615225.png)

---

After completing this step, you will get your own [filed domain name](https://console.cloud.tencent.com/domain/all-domain/all)

![image-20241127105819036](HarmonyOS%20Next%20%E5%85%83%E6%9C%8D%E5%8A%A1%E6%96%B0%E5%BB%BA%E5%88%B0%E4%B8%8A%E6%9E%B6%E5%85%A8%E6%B5%81%E7%A8%8B.assets/image-20241127105819036.png)

# File Atomic Service

Filing websites, applications, and atomic services follow the same process.

[File Atomic Service](https://console.cloud.tencent.com/beian/manage)

![image-20241127110100621](HarmonyOS%20Next%20%E5%85%83%E6%9C%8D%E5%8A%A1%E6%96%B0%E5%BB%BA%E5%88%B0%E4%B8%8A%E6%9E%B6%E5%85%A8%E6%B5%81%E7%A8%8B.assets/image-20241127110100621.png)

---

Fill in filing information

![image-20241127110412484](HarmonyOS%20Next%20%E5%85%83%E6%9C%8D%E5%8A%A1%E6%96%B0%E5%BB%BA%E5%88%B0%E4%B8%8A%E6%9E%B6%E5%85%A8%E6%B5%81%E7%A8%8B.assets/image-20241127110412484.png)

# Review Filing

![image-20241127110440114](HarmonyOS%20Next%20%E5%85%83%E6%9C%8D%E5%8A%A1%E6%96%B0%E5%BB%BA%E5%88%B0%E4%B8%8A%E6%9E%B6%E5%85%A8%E6%B5%81%E7%A8%8B.assets/image-20241127110440114.png)

1. After you complete the filling, Tencent Cloud will call you. If your information is incorrect, they will assist you in modifying it. Generally, they will call the same day.

2. After Tencent Cloud review passes, it will enter the **Management Bureau review stage**. They might call, or might not. If everything passes, you will receive an SMS on your phone. You must fill in the verification code from the SMS in the [Management Bureau filing system](https://beian.miit.gov.cn/#/Integrated/index) on the same day. At this point, the filing process is complete.

   ![image-20241127110908371](HarmonyOS%20Next%20%E5%85%83%E6%9C%8D%E5%8A%A1%E6%96%B0%E5%BB%BA%E5%88%B0%E4%B8%8A%E6%9E%B6%E5%85%A8%E6%B5%81%E7%A8%8B.assets/image-20241127110908371.png)

![image-20241127110949471](HarmonyOS%20Next%20%E5%85%83%E6%9C%8D%E5%8A%A1%E6%96%B0%E5%BB%BA%E5%88%B0%E4%B8%8A%E6%9E%B6%E5%85%A8%E6%B5%81%E7%A8%8B.assets/image-20241127110949471.png)

# AGC Resubmit for Review

![image-20241127111044105](HarmonyOS%20Next%20%E5%85%83%E6%9C%8D%E5%8A%A1%E6%96%B0%E5%BB%BA%E5%88%B0%E4%B8%8A%E6%9E%B6%E5%85%A8%E6%B5%81%E7%A8%8B.assets/image-20241127111044105.png)

# Follow-up

Continue to monitor AGC platform information for subsequent updates.
