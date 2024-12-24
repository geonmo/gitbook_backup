---
description: This page explains how to access KISTI CMS Tier-3.
---

# How to access T3\_KR\_KISTI

## Notice on restricting access information disclosure.

To avoid security risks, open documents like Internet space do not provide detailed access information. This information will be notified individually through e-mail when you request a new account. Please use the information by accessing the following methods.&#x20;

## Resource information provided by T3\_KR\_KISTI.

KISTI CMS Tier-3 has two user interface (UI) servers that can be accessed from outside. Each UI server was installed as a CentOS7 Linux distribution. Also, there is cms-gpu01, a UI server that can be accessed internally. The server is equipped with NVidia P100, so it is recommended when using a program using machine learning.

## How to access in Linux or macOS environment.

On Linux or macOS, you can access the UI server using the ssh client program built into the operating system.

```
ssh -p [Port Number] [Host Name]
```

{% hint style="info" %}
When accessing using the ssh command, add -X or -Y as an option to use the X-Windows graphics environment. After accessing, you can check the value of the $DISPLAY environmental variable to see if X11 forwarding has been performed normally.
{% endhint %}

## How to connect from Windows.

Windows does not provide an ssh program at the OS level. Please use a separate program to access unless you use Windows developer mode to launch a bash. The most famous programs include putty and MobaXterm. Here, we will explain based on MobaXterm.

### MobaXterm

#### Reason for using mobaXterm

From a long time ago, we mainly used the "putty" program to use the ssh terminal environment in the window environment. However, because the "putty" program does not include X-Windows server, we had to install a separate X11 support program such as X-ming to use X11 forwarding. This is currently not recommended for beginners because the user installs the program separately and requires additional setup.

#### License&#x20;

The MobaXterm program is free anywhere if you download and install files from the website yourself. This includes a built-in X-Windows Server. Therefore, you can download and install it safely at school and company.&#x20;

However, redistribution (if you share the received file) is considered a license violation, so please be sure to guide them to download the file from the website.

#### Connection Setup

After running the installed MobaXterm, click the \[Session] - \[SSH] button and fill out the necessary information to create a session where you need to access it. After entering the password, you can access it without entering it. For more information, please check the information in the email which we sent you before.

![Fig1. Click a \[Session\] button](.gitbook/assets/mobaxterm_01.PNG)

![Fig2. Click a \[SSH\] button](<.gitbook/assets/mobaxterm_02 (1).PNG>)

{% hint style="info" %}
In the case of Windows environments, you can activate or disable the X11 forwarding function in individual client programs. If that function is disabled, you cannot use the X-Windows graphics environment as if you were connected without the -X or -Y option.
{% endhint %}

#### 홈페이지 정보&#x20;

[https://mobaxterm.mobatek.net/](https://mobaxterm.mobatek.net/)
