---
title: svn_propset_mime_type
date: 2018-04-14 00:15
categories: svn
tags: 
- svn
- propset
- windows
- bat
---

``` bat
rem svn_cleanup_revert_update_wc.bat
set WC_DIR=%1

cd /d %WC_DIR%

if NOT EXIST "%WC_DIR%\.svn" (

echo "%WC_DIR%\.svn" is not exsits...............exit it.

exit /b 0

) ELSE (

echo begin excute

)
svn cleanup . %SVN_LOGIN_USER_PW%
svn revert -R . %SVN_LOGIN_USER_PW%
For /f "tokens=1,2" %%A in ('svn status --no-ignore') Do (
     If [%%A]==[?] ( Call :UniDelete %%B
     ) Else If [%%A]==[I] Call :UniDelete %%B
   )
svn update . %SVN_LOGIN_USER_PW%
goto :eof

:UniDelete delete file/dir
IF EXIST "%1\*" ( 
    RD /S /Q "%1"
) Else (
    If EXIST "%1" DEL /S /F /Q "%1"
)
goto :eof

```



``` bat
@ECHO OFF
rem set SVN_LOGIN_USER_PW=--username USER --password PASSWORD

set SVN_LOGIN_USER_PW=

set workcopy_dir=%1


call %~dp0svn_cleanup_revert_update_wc.bat %workcopy_dir%

cd /d "%workcopy_dir%"
FOR /F "eol=;tokens=*" %%i IN ('dir /s/b *.htm') DO call :SET_MIME text/html "%%i"
FOR /F "eol=;tokens=*" %%i IN ('dir /s/b *.js')  DO call :SET_MIME application/x-javascript "%%i"
FOR /F "eol=;tokens=*" %%i IN ('dir /s/b *.css') DO call :SET_MIME text/css "%%i"
FOR /F "eol=;tokens=*" %%i IN ('dir /s/b *.png') DO call :SET_MIME image/png "%%i"
FOR /F "eol=;tokens=*" %%i IN ('dir /s/b *.jpg') DO call :SET_MIME image/jpeg "%%i"
FOR /F "eol=;tokens=*" %%i IN ('dir /s/b *.gif') DO call :SET_MIME image/gif "%%i"

svn commit -m "USER_set properties in order to Visist HTML file in web broswer" %workcopy_dir% %SVN_LOGIN_USER_PW%

exit /b %ERRORLEVEL%

@ECHO ON
rem =========================================================================
rem call :SET_MIME %MIME_TYPE_TOSET% %FILE_PATH%
rem =========================================================================

:SET_MIME
set mime_type_ToSet=%1
set file_path=%2

set mime_type_current=1
for /f %%j in ('svn propget svn:mime-type %file_path%') do set mime_type_current=%%j
echo mime_type_current is %mime_type_current% 		mime_type_ToSet is %mime_type_ToSet%
if %mime_type_current% neq %mime_type_ToSet% (
echo svn propset svn:mime-type %mime_type_ToSet% %file_path%
svn propset svn:mime-type %mime_type_ToSet% %file_path%
)
goto :eof
```
