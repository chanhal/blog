> * 标题:　NSIS注册COM组件.md
> * 作者:　Hal Chan
> * 联系:　halchan@126.com  
> * 时间:　2017-08-04   


# NSIS注册COM组件

## 进程内COM组件注册(dll, ocx)

```  
;注册
  Exec 'regsvr32.exe /s "$INSTDIR\vpwBoard.ocx"'
  Exec 'regsvr32.exe /s "$INSTDIR\vpvX264Encoder.ax"'
  Exec 'regsvr32.exe /s "$INSTDIR\h264dec.ax"'
  Exec 'regsvr32.exe /s "$INSTDIR\imageOle.dll"'
  
;删除
  ExecWait "regsvr32.exe /s /u $INSTDIR\vpwBoard.ocx"
  ExecWait "regsvr32.exe /s /u $INSTDIR\vpvX264Encoder.ax"
  ExecWait "regsvr32.exe /s /u $INSTDIR\h264dec.ax"
  ExecWait "regsvr32.exe /s /u $INSTDIR\imageOle.dll"
  
```


## 进程外COM组件注册(exe)

可行方式： 
```
ExecWait "$INSTDIR\sys\Support\DistCadEngine.exe /regserver"
```

类似下面两种方式不成功：
```
;  ExecWait '$INSTDIR\sys18(x64)\tmp.bat'
;  nsExec::exec "$INSTDIR\sys18(x64)\tmp.bat" 
```

## VC注册exe组件可行方式


```   
// csCmdFilePath为bat文件路径
BOOL RunCmd(CString csCmdFilePath)
{
	BOOL bSuc = FALSE;

	if ( ::PathFileExists(csCmdFilePath) )
	{
		CString csName = ::PathFindFileName(csCmdFilePath);
		csName = _T("/c ") + csName;

		int iPos       = csCmdFilePath.ReverseFind(_T('\\'));
		CString csDir  = csCmdFilePath.Left(iPos);

		CStringA csOperationA = "runas";
		CStringA csFileA      = "cmd";
		CStringA csParametersA = csName;
		CStringA csDirA        = csDir; //(x64)

		SHELLEXECUTEINFOA ShExecInfo = {0};
		ShExecInfo.cbSize = sizeof(SHELLEXECUTEINFO);
		ShExecInfo.fMask = SEE_MASK_NOCLOSEPROCESS;
		ShExecInfo.hwnd = NULL;
		ShExecInfo.lpVerb = "runas";
		ShExecInfo.lpFile = "cmd";
		ShExecInfo.lpParameters = csParametersA ;
		ShExecInfo.lpDirectory = "\"" + csDirA + "\""; // 注意路径加入双引号，防止中加有空格和特殊符号
		ShExecInfo.nShow = SW_HIDE; // SW_SHOW SW_HIDE
		ShExecInfo.hInstApp = NULL;

		if( !ShellExecuteExA(&ShExecInfo) )  
		{  
			DWORD dwStatus=GetLastError();  
			if(dwStatus==ERROR_CANCELLED)  
			{  
				bSuc = FALSE;
			}  
			else if(dwStatus==ERROR_FILE_NOT_FOUND)  
			{  
				bSuc = FALSE;
			}  
		}
		else
			bSuc = TRUE;

		if ( ShExecInfo.hProcess )
		{
			WaitForSingleObject(ShExecInfo.hProcess,INFINITE);
			CloseHandle(ShExecInfo.hProcess);
		}
	}
	else
	{
		OutputDebugString(_T("RunCmd: csCmdFilePath not exist."));
		bSuc = FALSE;	
	}

	return bSuc;
}
```   


### 注意  

* 执行bat时输入的路径需要用双引号来防止空格和特殊符号
* 写入bat文件中的路径也要用双引号来防止空格和特殊符号
* 尽量使用muti-byte的方式编码和执行
