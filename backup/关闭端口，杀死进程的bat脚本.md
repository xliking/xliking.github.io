```
@echo off
setlocal

:: 弹出输入框获取端口号
set "port="
set /p port=Please enter the port number to close: 

:: 检查是否输入了端口号
if "%port%"=="" (
    echo No port number entered.
    pause
    exit /b 1
)

set PORT=%port%

:: 查找使用指定端口的进程ID
set "PID="
for /f "tokens=5" %%a in ('netstat -aon ^| findstr ":%PORT%"') do (
    set PID=%%a
    goto :found
)

echo No process found running on port %PORT%
pause
exit /b 1

:found
if "%PID%"=="" (
    echo No process found running on port %PORT%
    pause
    exit /b 1
)

echo Terminating process %PID% running on port %PORT%...

:: 终止进程
taskkill /PID %PID% /F

if %errorlevel%==0 (
    echo Successfully terminated process %PID% running on port %PORT%
) else (
    echo Failed to terminate process %PID%
)

:: 保持窗口打开以查看结果
pause

endlocal
```