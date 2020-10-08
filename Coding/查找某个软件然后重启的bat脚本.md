```bash
:FindGrab
    tasklist /nh|find /i "ProcessName"               
    if "%errorlevel%"=="0" (goto QuitFindGrab) else (goto FindGrab)
:QuitFindGrab
    taskkill /f /im ProcessName
```

