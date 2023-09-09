---
layout: post
title: windows服务编程模板
category: 平台
keywords:
---

如果希望Windows程序在后台运行，可考虑写成Windows服务。在运行中输入services.msc查看当前系统中已经安装的服务列表。

Windows服务操作流程一般是：安装服务 => 启动服务 => 停止服务 => 卸载服务。

下面是一个简单的Windows服务编程例子，该服务的功能是每隔1秒钟输出一句\"hello\"。

```cpp
#include <windows.h>
#include <stdio.h>
#include <process.h>
#include <conio.h>

char ServiceName[] = "TestService";
char ServicePath[MAX_PATH];

SERVICE_STATUS ServiceStatus;
SERVICE_STATUS_HANDLE hStatus;

void WINAPI ServiceMain(DWORD argc, LPTSTR *argv);
void WINAPI ServiceHandle(DWORD control);

SERVICE_TABLE_ENTRY lpServiceStartTable[] = {
    {ServiceName, ServiceMain},
    {NULL, NULL}
};

void Log(char *fmt, ...) {
    char buf[1024] = "<demo> ";
    va_list ap;
    va_start(ap, fmt);
    vsprintf(buf + 7, fmt, ap);
    va_end(ap);
    OutputDebugString(buf);
}

void HandleOpenSCManagerError(int ret) {
    const char *p;
    switch (ret) {
        case ERROR_ACCESS_DENIED: p = "access denied"; break;
        case ERROR_DATABASE_DOES_NOT_EXIST: p = "database not exist"; break;
        case ERROR_INVALID_PARAMETER: p = "invalid parameter"; break;
        default: p = "error in registry functions called by scm"; break;
    }
    Log("OpenSCManager failed, code=%d, msg=[%s]", ret, p);
}

void HandleCreateServiceError(int ret) {
    const char *p;
    switch (ret) {
        case ERROR_ACCESS_DENIED: p = "access denied"; break;
        case ERROR_CIRCULAR_DEPENDENCY: p = "circular dependency"; break;
        case ERROR_DUP_NAME: p = "displayed name already exists"; break;
        case ERROR_INVALID_HANDLE: p = "invalid handle"; break;
        case ERROR_INVALID_NAME: p = "invalid service name"; break;
        case ERROR_INVALID_PARAMETER: p = "invalid parameter"; break;
        case ERROR_INVALID_SERVICE_ACCOUNT: p = "invalid account"; break;
        case ERROR_SERVICE_EXISTS: p = "service already exists"; break;
        default: p = "unknwon error"; break;
    }
    Log("CreateService failed, code=%d, msg=[%s]", ret, p);
}

void HandleOpenServiceError(int ret) {
    const char *p;
    switch (ret) {
        case ERROR_ACCESS_DENIED: p = "access denied"; break;
        case ERROR_INVALID_HANDLE: p = "invalid handle"; break;
        case ERROR_INVALID_NAME: p = "invalid service name"; break;
        case ERROR_SERVICE_DOES_NOT_EXIST: p = "service does not exist"; break;
        default: p = "unknown error"; break;
    }
    Log("OpenService failed, ret=%d, msg=[%s]", ret, p);
}

void HandleDeleteServiceError(int ret) {
    const char *p;
    switch (ret) {
        case ERROR_ACCESS_DENIED: p = "access denied"; break;
        case ERROR_INVALID_HANDLE: p = "invalid handle"; break;
        case ERROR_SERVICE_MARKED_FOR_DELETE: p = "already marked for deletion"; break;
        default: p = "error in registry functions called by scm"; break;
    }
    Log("DeleteService failed, ret=%d, msg=[%s]", ret, p);
}

void HandleStartServiceError(int ret) {
    const char *p;
    switch (ret) {
        case ERROR_ACCESS_DENIED: p = "access denied"; break;
        case ERROR_INVALID_HANDLE: p = "invalid handle"; break;
        case ERROR_PATH_NOT_FOUND: p = "path not found"; break;
        case ERROR_SERVICE_ALREADY_RUNNING: p = "already running"; break;
        case ERROR_SERVICE_DATABASE_LOCKED: p = "database locked"; break;
        case ERROR_SERVICE_DEPENDENCY_DELETED: p = "dependency deleted"; break;
        case ERROR_SERVICE_DEPENDENCY_FAIL: p = "dependency fail"; break;
        case ERROR_SERVICE_DISABLED: p = "service disabled"; break;
        case ERROR_SERVICE_LOGON_FAILED: p = "logon fail"; break;
        case ERROR_SERVICE_MARKED_FOR_DELETE: p = "serviced marked for deletion"; break;
        case ERROR_SERVICE_NO_THREAD: p = "create thread for service failed"; break;
        case ERROR_SERVICE_REQUEST_TIMEOUT: p = "request timeout"; break;
        default: p = "error in registry functions called by scm"; break;
    }
    Log("StartService failed, ret=%d, msg=[%s]", ret, p);
}

void HandleControlServiceError(int ret) {
    const char *p;
    switch (ret) {
        case ERROR_ACCESS_DENIED: p = "access denied"; break;
        case ERROR_DEPENDENT_SERVICES_RUNNING: p = "dependent service running"; break;
        case ERROR_INVALID_HANDLE: p = "invalid handle"; break;
        case ERROR_INVALID_PARAMETER: p = "invalid parameter"; break;
        case ERROR_INVALID_SERVICE_CONTROL: p = "invalid service control"; break;
        case ERROR_SERVICE_CANNOT_ACCEPT_CTRL: p = "service cannot accept control"; break;
        case ERROR_SERVICE_NOT_ACTIVE: p = "service not active"; break;
        case ERROR_SERVICE_REQUEST_TIMEOUT: p = "request timeout"; break;
        case ERROR_SHUTDOWN_IN_PROGRESS: p = "system is shutting down"; break;
        default: p = "error in registry functions called by scm"; break;
    }
    Log("ControlService failed, ret=%d, msg=[%s]", ret, p);
}

void Install() {
    SC_HANDLE hm = OpenSCManager(NULL, NULL, SC_MANAGER_ALL_ACCESS);
    if (!hm) return HandleOpenSCManagerError(GetLastError());
    SC_HANDLE hs = CreateService(hm, ServiceName, ServiceName, SERVICE_ALL_ACCESS,
        SERVICE_WIN32_OWN_PROCESS, SERVICE_AUTO_START, SERVICE_ERROR_NORMAL,
        ServicePath, NULL, NULL, NULL, NULL, NULL);
    if (!hs) {
        int ret = GetLastError();
        CloseServiceHandle(hm);
        return HandleCreateServiceError(ret);
    }
    Log("install service success");
    CloseServiceHandle(hs);
    CloseServiceHandle(hm);
}

void Uninstall() {
    SC_HANDLE hm = OpenSCManager(NULL, NULL, SC_MANAGER_ALL_ACCESS);
    if (!hm) return HandleOpenSCManagerError(GetLastError());
    SC_HANDLE hs = OpenService(hm, ServiceName, SERVICE_ALL_ACCESS);
    if (!hs) {
        int ret = GetLastError();
        CloseServiceHandle(hm);
        return HandleOpenServiceError(ret);
    }
    if (!DeleteService(hs)) {
        HandleDeleteServiceError(GetLastError());
    } else {
        Log("uninstall service success");
    }
    CloseServiceHandle(hs);
    CloseServiceHandle(hm);
}

void Start() {
    SC_HANDLE hm = OpenSCManager(NULL, NULL, SC_MANAGER_ALL_ACCESS);
    if (!hm) return HandleOpenSCManagerError(GetLastError());
    SC_HANDLE hs = OpenService(hm, ServiceName, SERVICE_ALL_ACCESS);
    if (!hs) {
        int ret = GetLastError();
        CloseServiceHandle(hm);
        return HandleOpenServiceError(ret);
    }
    if (!StartService(hs, 0, NULL)) {
        HandleStartServiceError(GetLastError());
    } else {
        Log("start service success");
    }
    CloseServiceHandle(hs);
    CloseServiceHandle(hm);
}

void Stop() {
    SC_HANDLE hm = OpenSCManager(NULL, NULL, SC_MANAGER_ALL_ACCESS);
    if (!hm) return HandleOpenSCManagerError(GetLastError());
    SC_HANDLE hs = OpenService(hm, ServiceName, SERVICE_ALL_ACCESS);
    if (!hs) {
        int ret = GetLastError();
        CloseServiceHandle(hm);
        return HandleOpenServiceError(ret);
    }
    SERVICE_STATUS status;
    if (!ControlService(hs, SERVICE_CONTROL_STOP, &status)) {
        HandleControlServiceError(GetLastError());
    } else {
        Log("stop service success");
    }
    CloseServiceHandle(hs);
    CloseServiceHandle(hm);
}

void WINAPI ServiceHandle(DWORD control) {
    switch (control) {
    case SERVICE_CONTROL_STOP:
    case SERVICE_CONTROL_SHUTDOWN:
        ServiceStatus.dwWin32ExitCode = 0;
        ServiceStatus.dwCurrentState = SERVICE_STOPPED;
        ServiceStatus.dwCheckPoint = 0;
        ServiceStatus.dwWaitHint = 0;
        break;
    case SERVICE_CONTROL_PAUSE:
        ServiceStatus.dwCurrentState = SERVICE_PAUSED;
        break;
    case SERVICE_CONTROL_CONTINUE:
        ServiceStatus.dwCurrentState = SERVICE_RUNNING;
        break;
    default:
        Log("unknown control type %d", control);
        break;
    }
    SetServiceStatus(hStatus, &ServiceStatus);
}

void WINAPI ServiceMain(DWORD argc, LPTSTR * argv) {
    DWORD status = 0, specificError = -1;
    ServiceStatus.dwServiceType = SERVICE_WIN32;
    ServiceStatus.dwCurrentState = SERVICE_START_PENDING;
    ServiceStatus.dwControlsAccepted = SERVICE_ACCEPT_STOP | SERVICE_ACCEPT_SHUTDOWN | SERVICE_ACCEPT_PAUSE_CONTINUE;
    ServiceStatus.dwWin32ExitCode = 0;
    ServiceStatus.dwServiceSpecificExitCode = 0;
    ServiceStatus.dwCheckPoint = 0;
    ServiceStatus.dwWaitHint = 0;
    
    hStatus = RegisterServiceCtrlHandler(ServiceName, ServiceHandle);
    if (!hStatus) {
        Log("RegisterServiceCtrlHandle failed, ret=%d", GetLastError());
    } else {
        ServiceStatus.dwCurrentState = SERVICE_RUNNING;
        ServiceStatus.dwCheckPoint = 0;
        ServiceStatus.dwWaitHint = 0;
        if (!SetServiceStatus(hStatus, &ServiceStatus)) {
            Log("SetServiceStatus failed, ret=%d", GetLastError());
        }
    }
}

void ProcListenThread(void *arg) {
    while (true) {
        Log("hello");
        Sleep(1000);
    }
}

void Run() {
    if (-1 == _beginthread(ProcListenThread, 0, NULL)) {
        Log("create listen thread failed, ret=%d", GetLastError());
    }
}

int main(int argc, char * argv[]) {
    GetModuleFileName(NULL, ServicePath, sizeof(ServicePath));
    if (argc == 1) {
        Run();
        if (!StartServiceCtrlDispatcher(lpServiceStartTable)) {
            Log("StartServiceCtrlDispatcher failed, ret=%d", GetLastError());
        }
    } else if (!stricmp(argv[1], "install")) {
        Install();
    } else if (!stricmp(argv[1], "start")) {
        Start();
    } else if (!stricmp(argv[1], "stop")) {
        Stop();
    } else if (!stricmp(argv[1], "uninstall")) {
        Uninstall();
    }
    return 0;
}
```

说明：

- 在VisualStudio中新建空的控制台应用程序即可，无需自动生成代码；
- 服务名称ServiceName可根据实际情况自行定义；
- 服务主体工作函数为ProcListenThread()；
- CreateService函数最后两参数为安装服务使用的用户名和密码，NULL代表系统用户；如果要指定为某个用户，则要求该用户存在，且密码正确，另外允许该用户作为服务登录，一般在组策略中做相关设置；
- 一般通过批处理脚本来启动和停止服务；

假设编译出来的可执行程序文件名为TestService.exe，启动服务脚本可以这样写：

```
@echo off
cd %~dp0
TestService install
TestService start
```

同样的，停止服务脚本如下：

```
@echo off
cd %~dp0
TestService stop
TestService uninstall
```

需要注意，脚本中给出的参数须与程序中比较判断的参数一致。

