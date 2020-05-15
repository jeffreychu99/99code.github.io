---
layout: post
title: Windows锁屏背景替换
categories: C++
description: Windows锁屏背景替换
keywords: Winows, 锁屏背景
---

# 概述

是否厌倦Windows一贯的锁屏界面，本章节内容介绍如何可以随心定制自己喜欢的界面。

主要介绍三个方面的技术点：

1、锁屏时，更换系统的锁屏界面

2、随机切换images文件夹下的图片

3、程序退出时恢复系统本来的面貌

图片预览

![](/images/posts/windows-lockscreen-bg/1.jpg)

![](/images/posts/windows-lockscreen-bg/2.jpg)

# Win7 

## 背景界面替换原理

第一步：监听锁屏事件；LRESULT CLockAPPSampleDlg::WindowProc(UINT message, WPARAM wParam, LPARAM lParam)

```cpp
LRESULT CLockAPPSampleDlg::WindowProc(UINT message, WPARAM wParam, LPARAM lParam) 
{
    // TODO: Add your specialized code here and/or call the base class
    switch(message)
    {
    case WM_WTSSESSION_CHANGE:
        {
            //MessageBox("WM_WTSSESSION_CHANGE", "Esmile", MB_OK);

            switch(wParam)
            {
            case WTS_SESSION_LOCK:
                {
                //锁屏事件
                }
                
                break;
            case WTS_SESSION_UNLOCK:
                {
                                //解锁时间
                                }
                break;
            default:
                break;
            }

        }
        break;
    case WM_DESTROY:
        WTSUnRegisterSessionNotification(m_hWnd);
        break;


    default:
        break;
    }

    return CDialog::WindowProc(message, wParam, lParam);
}
```
第二步：将图片拷贝C:\Windows\System32\oobe\info\Backgrounds目录下，名字可以任意；

```cpp
void CWin7DesktopUtil::copyFile(LPTSTR lpPicFile)
{
    TCHAR szPath[MAX_PATH] = { 0 };

    _tcscpy(szPath, _T("C:\\Windows\\System32\\oobe"));

    if (FALSE == PathFileExists(szPath)) {
        if(FALSE == CreateDirectory(szPath, NULL)){
            return;
        }
    }

    _tcscat(szPath, _T("\\info"));
    if (FALSE == PathFileExists(szPath)) {
        if(FALSE == CreateDirectory(szPath, NULL)){
            return;
        }
    }
    
    _tcscat(szPath, _T("\\Backgrounds"));
    if (FALSE == PathFileExists(szPath)) {
        if(FALSE == CreateDirectory(szPath, NULL)){
            return;
        }
    }

    _tcscat(szPath, _T("\\backgroundDefault.jpg"));
    ::CopyFile(lpPicFile, szPath, FALSE);
}
```

第三步：设置注册表

将SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Authentication\\LogonUI\\Background下的

OEMBackground键值设为1

```cpp
BOOL CWin7DesktopUtil::setOEMBackground(DWORD dwValue)
{
    HKEY hKey; 

    LPCTSTR lpRun = _T("SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Authentication\\LogonUI\\Background");
    long lRet = RegOpenKeyEx(HKEY_LOCAL_MACHINE, lpRun, 0, KEY_WRITE|KEY_WOW64_64KEY, &hKey); 
    if(lRet== ERROR_SUCCESS)
    {
        lRet = RegSetValueEx(hKey, _T("OEMBackground"), 0, REG_DWORD, (LPBYTE)&dwValue, sizeof(dwValue));
        if (ERROR_SUCCESS == lRet)
        {
            RegCloseKey(hKey); 
            return TRUE;
        }
        else
        {
            DWORD dwError = GetLastError();
        }

        RegCloseKey(hKey); 
    }

    return FALSE;
}
```

PS：

1、此处需要注意图片大小不能大于256K，否则设置无效。

2、考虑64位系统情况，拷贝之前调用

Wow64DisableWow64FsRedirection

## 定时器

这个很简单。

第一步：检索图片文件夹，保存到m_strPathArray数组中

```cpp
void CRandomImage::search(LPTSTR path)
{
    HANDLE hFind;  
    WIN32_FIND_DATA wfd;  
    TCHAR tempPath[MAX_PATH];  

    ZeroMemory(&wfd, sizeof(WIN32_FIND_DATA));  
    
    memset(tempPath, 0, sizeof(tempPath));  
    sprintf(tempPath, "%s\\*.*", path);  

    hFind = FindFirstFile(tempPath, &wfd);  
    if(INVALID_HANDLE_VALUE == hFind)  
    {  
        return;  
    }  
    do  
    {  
        if('.' == wfd.cFileName[0])  
        {  
            continue;  
        }  

        if(wfd.dwFileAttributes & FILE_ATTRIBUTE_DIRECTORY)  
        {  
            sprintf(tempPath, "%s\\%s", path, wfd.cFileName);  

            search(tempPath);  
        }  
        else  
        {  
            sprintf(tempPath, "%s\\%s", path, wfd.cFileName);  
            m_strPathArray.Add(tempPath);
        }  
    }while(FindNextFile(hFind, &wfd));  
    
    FindClose(hFind);  
}
```

第二步：从图片文件夹中随机选取一张图片

```cpp
const CString CRandomImage::getRandomImage()
{
    INT32 length = m_strPathArray.GetCount();
    if (length == 0) {
        return _T("");
    }

    {
        srand((unsigned)time(NULL));
        m_iRandomIndex = rand() % length;
    }while(m_iRandomIndex == length);
    

    return m_strPathArray.GetAt(m_iRandomIndex);
}
```

## 恢复
将
SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Authentication\\LogonUI\\Background下的

OEMBackground键值设为0

# xp
## 背景界面替换原理

xp不同win7，设置比较复杂，没有现成的接口可以调用。大家可以尝试一下，XP是桌面是什么图片，那么锁屏的时候就是什么背景。因此，设置xp锁屏背景，只需要在锁屏时，动态替换桌面背景即可（这里背景的替换会有点延时的），然后解锁时，恢复之前的桌面背景即可，这里你必须小心处理各种事件，否则，之前的桌面可能不能恢复成功。

```cpp
BOOL CXPDesktopUtil::SetWallpaper(LPTSTR lpPicFile, DWORD dwStyle)
{
    HRESULT hr; 
    IActiveDesktop* pIAD;   //创建接口的实例  
    CoInitialize(NULL);  
    hr = CoCreateInstance(CLSID_ActiveDesktop,NULL,CLSCTX_INPROC_SERVER,IID_IActiveDesktop,(void**)&pIAD); 
    if(!SUCCEEDED(hr))
    {
        return FALSE; 
    }

    //将文件名改为宽字符串,这是IActiveDesktop::SetWallpaper的要求 
    WCHAR wszWallpaper[MAX_PATH]; 
    MultiByteToWideChar(CP_ACP,0,lpPicFile,-1,wszWallpaper,MAX_PATH); 
    //设置墙纸 

    hr = pIAD-> SetWallpaper(wszWallpaper, 0); 
    if(!SUCCEEDED(hr))   
    {
        return TRUE; 
    }

    //设置墙纸的样式 
    WALLPAPEROPT wpo; 
    wpo.dwSize = sizeof(wpo); 
    wpo.dwStyle = dwStyle; 
    hr = pIAD->SetWallpaperOptions(&wpo,0); 
    if(!SUCCEEDED(hr))   
    {
        return FALSE; 
    }

    //应用墙纸的设置 
    hr = pIAD-> ApplyChanges(AD_APPLY_ALL); 
    if(!SUCCEEDED(hr)) 
    {
        return FALSE;
    }

    //释放接口的实例 
    pIAD-> Release(); 
    CoUninitialize(); 

    return   TRUE;
}
```

备份之前桌面的背景图片

```cpp
CString CXPDesktopUtil::backupWallPaper(LPTSTR lpPicFile)
{
    TCHAR szPath[MAX_PATH] = { 0 };

    CString strBackup = CAppUtil::getApplicationDirectory() + "backup";

    if (FALSE == PathFileExists(strBackup.GetBuffer())) {
        if(FALSE == CreateDirectory(strBackup.GetBuffer(), NULL)){
            return _T("");
        }
    }

    strBackup += _T("\\backgroundDefault.jpg");

    ::CopyFile(lpPicFile, strBackup.GetBuffer(), FALSE);

    return strBackup;
}
```

这里你可能需要关闭用户快速切换

```cpp
//关闭快速用户切换
    HKEY hKey; 

    LPCTSTR lpRun = _T("SOFTWARE\\Microsoft\\\Windows NT\\CurrentVersion\\Winlogon");
    long lRet = RegOpenKeyEx(HKEY_LOCAL_MACHINE, lpRun, 0, KEY_WRITE, &hKey); 
    if(lRet== ERROR_SUCCESS)
    {
        DWORD dwValue = 0;
        lRet = RegSetValueEx(hKey, _T("AllowMultipleTSSessions"), 0, REG_DWORD, (LPBYTE)&dwValue, sizeof(dwValue));
        if (ERROR_SUCCESS == lRet)
        {
        }
        else
        {
            DWORD dwError = GetLastError();
        }

        RegCloseKey(hKey); 
    }
```

## 定时器

同win7的。

## 恢复

替换之前备份的桌面背景即可。

**附完整的源代码：**

http://git.oschina.net/zhujf21st/LockApp
