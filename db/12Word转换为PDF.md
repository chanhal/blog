> * 标题:　Word转换为PDF.md
> * 作者:　Hal Chan
> * 联系:　halchan@126.com  
> * 时间:　2017-09-11   


## 预编译头包含文件

// for_debug
#import "C:\\Program Files (x86)\\Common Files\\Microsoft Shared\\OFFICE14\\MSO.DLL" rename("RGB", "MSORGB") rename("DocumentProperties", "MSODocumentProperties")
using namespace Office;

#import "C:\\Program Files (x86)\\Common Files\\Microsoft Shared\\VBA\\VBA6\\VBE6EXT.OLB"  
using namespace VBIDE;

#import "C:\\Program Files (x86)\\Microsoft Office\\Office14\\MSWORD.OLB" rename_namespace("MSWord") auto_search auto_rename no_auto_exclude rename("ExitWindows", "WordExitWindows")
using namespace MSWord;


## 转换函数

```
BOOL CQualityReportHtmlDlg::ConvertFormat(CString strSourcePath, CString strTargetPath, MSWord::WdExportFormat wdExportFormat)
{
	BOOL                        result = FALSE;
	MSWord::_ApplicationPtr     pWdApplicationPtr;
	MSWord::_DocumentPtr        pWdDocumentPtr;

	COleVariant                 sourcePath = strSourcePath;
	COleVariant                 targetPath = strTargetPath;
	COleVariant                 vTrue((short)TRUE);
	COleVariant                 vFalse((short)FALSE);
	COleVariant                 vZero((short)0);
	COleVariant                 vOptional((long)DISP_E_PARAMNOTFOUND, VT_ERROR);

	CoInitialize(NULL);
	try
	{
		HRESULT hResult = pWdApplicationPtr.CreateInstance("Word.Application");
		if (hResult != S_OK)
		{
			AfxMessageBox(_T("Application创建失败，请确保安装了word 2000或以上版本!"), MB_OK | MB_ICONWARNING);
			CoUninitialize();
			return result;
		}

		pWdApplicationPtr->put_Visible(VARIANT_TRUE);  //vc10+word07+win7

		pWdDocumentPtr = pWdApplicationPtr->Documents->Open(sourcePath,
															vTrue,              // Confirm Conversion.  
															vTrue,              // ReadOnly.  
															vFalse,             // AddToRecentFiles.  
															vOptional,          // PasswordDocument.  
															vOptional,          // PasswordTemplate.  
															vOptional,          // Revert.  
															vOptional,          // WritePasswordDocument.  
															vOptional,          // WritePasswordTemplate.  
															vOptional,          // Format. // Last argument for Word 97  
															vOptional,          // Encoding // New for Word 2000/2002  
															vFalse,             // visible  
															vOptional,          // openAndRepair  
															vZero,              // docDirection  
															vOptional,          // NoEncodingDialog  
															vOptional);

		
		
		if (pWdDocumentPtr == NULL)
		{
			CoUninitialize();
			return result;
		}	

		//CComVariant FileName(_T("outputpdf.pdf"));
		//CComVariant FileFormat(17);
		COleVariant FileName = _T("d:\\outputpdf.pdf");
		COleVariant FileFormat((long)wdExportFormat);
		int ntemp = 17;

		hResult = pWdDocumentPtr->SaveAs(FileName, FileFormat);

//		CComVariant SaveChanges(false), OriginalFormat, RouteDocument;

		if (hResult == S_OK)
		{
			result = TRUE;
		}
	}
	catch (CException* e)
	{
		TCHAR   szError[1024];
		e->GetErrorMessage(szError, 1024);
		AfxMessageBox(_T("WORD文档转PDF异常：") + (CString)szError);
	}
	if (pWdDocumentPtr != NULL)
	{
		pWdDocumentPtr->Close();
		pWdDocumentPtr = NULL;
	}
	if (pWdApplicationPtr != NULL)
	{
		pWdApplicationPtr->Quit();
		pWdApplicationPtr = NULL;
	}
	CoUninitialize();
	return result;
}

```

## 调用  

// convert rtf to pdf
ConvertFormat(lpTempFile, csFilePathName, wdExportFormatPDF); // wdExportFormatPDF为定义好的宏
