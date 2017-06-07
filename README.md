## 简单的inno setup安装脚本 

#define MyAppName "程序名称"
#define MyAppVersion "版本号"
#define MyAppPublisher "版权"
#define MyAppURL "产品URL"
#define MyAppExeName "exe文件名称"

[Setup]
; NOTE: The value of AppId uniquely identifies this application.
; Do not use the same AppId value in installers for other applications.
; (To generate a new GUID, click Tools | Generate GUID inside the IDE.)
AppId={{655A2DE6-C9A3-432E-951B-D773791C2653}
AppName={#MyAppName}
AppVersion={#MyAppVersion}
AppVerName={#MyAppName} {#MyAppVersion}
AppPublisher={#MyAppPublisher}
AppPublisherURL={#MyAppURL}
AppSupportURL={#MyAppURL}
AppUpdatesURL={#MyAppURL}
;DefaultDirName={code:GetDefaultDir}
DefaultDirName={pf}\{#MyAppName}
DefaultGroupName={#MyAppName}
OutputDir=.
;xxx_{#MyAppVersion}_Setup
OutputBaseFilename=
;..\xxx.Client\logo.ico
SetupIconFile=
Compression=lzma
SolidCompression=yes
ArchitecturesInstallIn64BitMode=x64
LicenseFile=License.txt
WizardImageFile=wca.bmp
ShowLanguageDialog=auto
LanguageDetectionMethod=locale
VersionInfoVersion={#MyAppVersion}
;xxx有限公司
VersionInfoCompany=
VersionInfoDescription={#MyAppName}
VersionInfoTextVersion={#MyAppVersion}
;Copyright ? 2016 xxx, Inc
VersionInfoCopyright=
VersionInfoProductName={#MyAppName}
VersionInfoProductVersion={#MyAppVersion}
ShowTasksTreeLines=True
AlwaysShowGroupOnReadyPage=True
AlwaysShowDirOnReadyPage=True
;Copyright ? 2016 xxx, Inc
AppCopyright=

[Languages]
Name: "chs"; MessagesFile: "compiler:Languages\ChineseSimplified.isl"
Name: "english"; MessagesFile: "compiler:Default.isl"

[LangOptions] 
DialogFontName=Verdana 
DialogFontSize=8 
DialogFontStandardHeight=13 
TitleFontName=Verdana 
TitleFontSize=29 
WelcomeFontName=Verdana 
WelcomeFontSize=12 
CopyrightFontName=Verdana 
CopyrightFontSize=12

[Messages]
;安装界面中显示的文字
BeveledLabel=xxxxx 

;写注册表
[Registry]
Root: HKCU; Subkey: "Software\{#MyAppName}\Client"; ValueType: string; ValueName: "INSTDIR"; ValueData: "{app}"; Flags: uninsdeletevalue

;快捷方式
[Tasks]
Name: "desktopicon"; Description: "{cm:CreateDesktopIcon}"; GroupDescription: "{cm:AdditionalIcons}"; Flags: checkedonce
Name: "quicklaunchicon"; Description: "{cm:CreateQuickLaunchIcon}"; GroupDescription: "{cm:AdditionalIcons}"; Flags: unchecked; OnlyBelowVersion: 0,6.1

;安装前删除的文件
[InstallDelete]
Type: files; Name: "{app}\xxx.dll"

     
[Files]
; 安装是复制的.Net Framework
Source: {tmp}\dotNetFx45_Full_setup.exe; DestDir: "{tmp}"; Flags: ignoreversion 

; 安装是复制的文件
Source: "..\bin\Debug\YTGame.exe"; DestDir: "{app}"; Flags: ignoreversion 


; 创建文件夹
Source: "..\bin\Debug\Sounds\*"; DestDir: "{app}\Sounds\"; Flags: igNoreversion recursesubdirs createallsubdirs

[Icons]
Name: "{group}\{#MyAppName}"; Filename: "{app}\{#MyAppExeName}"
Name: "{group}\{cm:ProgramOnTheWeb,{#MyAppName}}"; Filename: "{#MyAppURL}"
Name: "{group}\{cm:UninstallProgram,{#MyAppName}}"; Filename: "{uninstallexe}"
Name: "{commondesktop}\{#MyAppName}"; Filename: "{app}\{#MyAppExeName}"; Tasks: desktopicon
Name: "{userappdata}\Microsoft\Internet Explorer\Quick Launch\{#MyAppName}"; Filename: "{app}\{#MyAppExeName}"; Tasks: quicklaunchicon

;安装后运行主程序
[Run]
Filename: "{app}\{#MyAppExeName}"; Description: "{cm:LaunchProgram,{#StringChange(MyAppName, '&', '&&')}}"; Flags: nowait postinstall skipifsilent

	[Code]
	function IsDotNetDetected(version: string; service: cardinal): boolean;
	// Indicates whether the specified version and service pack of the .NET Framework is installed.
	//
	// version -- Specify one of these strings for the required .NET Framework version:
	//    'v1.1'          .NET Framework 1.1
	//    'v2.0'          .NET Framework 2.0
	//    'v3.0'          .NET Framework 3.0
	//    'v3.5'          .NET Framework 3.5
	//    'v4\Client'     .NET Framework 4.0 Client Profile
	//    'v4\Full'       .NET Framework 4.0 Full Installation
	//    'v4.5'          .NET Framework 4.5
	//    'v4.5.1'        .NET Framework 4.5.1
	//    'v4.5.2'        .NET Framework 4.5.2
	//    'v4.6'          .NET Framework 4.6
	//    'v4.6.1'        .NET Framework 4.6.1
	//
	// service -- Specify any non-negative integer for the required service pack level:
	//    0               No service packs required
	//    1, 2, etc.      Service pack 1, 2, etc. required
	var
	    key, versionKey: string;
	    install, release, serviceCount, versionRelease: cardinal;
	    success: boolean;
	begin
	    versionKey := version;
	    versionRelease := 0;
	
	    // .NET 1.1 and 2.0 embed release number in version key
	    if version = 'v1.1' then begin
	        versionKey := 'v1.1.4322';
	    end else if version = 'v2.0' then begin
	        versionKey := 'v2.0.50727';
	    end
	
	    // .NET 4.5 and newer install as update to .NET 4.0 Full
	    else if Pos('v4.', version) = 1 then begin
	        versionKey := 'v4\Full';
	        case version of
	          'v4.5':   versionRelease := 378389;
	          'v4.5.1': versionRelease := 378675; // or 378758 on Windows 8 and older
	          'v4.5.2': versionRelease := 379893;
	          'v4.6':   versionRelease := 393295; // or 393297 on Windows 8.1 and older
	          'v4.6.1': versionRelease := 394254; // or 394271 on Windows 8.1 and older
	        end;
	    end;
	
	    // installation key group for all .NET versions
	    key := 'SOFTWARE\Microsoft\NET Framework Setup\NDP\' + versionKey;
	
	    // .NET 3.0 uses value InstallSuccess in subkey Setup
	    if Pos('v3.0', version) = 1 then begin
	        success := RegQueryDWordValue(HKLM, key + '\Setup', 'InstallSuccess', install);
	    end else begin
	        success := RegQueryDWordValue(HKLM, key, 'Install', install);
	    end;
	
	    // .NET 4.0 and newer use value Servicing instead of SP
	    if Pos('v4', version) = 1 then begin
	        success := success and RegQueryDWordValue(HKLM, key, 'Servicing', serviceCount);
	    end else begin
	        success := success and RegQueryDWordValue(HKLM, key, 'SP', serviceCount);
	    end;
	
	    // .NET 4.5 and newer use additional value Release
	    if versionRelease > 0 then begin
	        success := success and RegQueryDWordValue(HKLM, key, 'Release', release);
	        success := success and (release >= versionRelease);
	    end;
	
	    result := success and (install = 1) and (serviceCount >= service);
	end;
	
	//安装.NET Framework 4.5
	function InitializeSetup(): Boolean;
	var   
	    ResultCode: Integer; 
	begin
	    if not IsDotNetDetected('v4.5', 0) then begin
	      if MsgBox('.NET Framework版本过低，需要升级新版本的.NET Framework？', mbConfirmation, MB_YESNO or MB_DEFBUTTON1) = IDYES then begin
	        // yes
	        ExtractTemporaryFile('dotNetFx45_Full_setup.exe');  
	        Exec(ExpandConstant('{tmp}/dotNetFx45_Full_setup.exe'), ' /passive /norestart', '', SW_SHOWNORMAL, ewWaitUntilTerminated, ResultCode); 
	        if ResultCode <> 0 then begin
	          MsgBox('.NET Framework没有安装成功，{#MyAppName}将退出安装！', mbInformation, MB_OK);
	          result := false;
	          Exit;
	        end;
	       end else
	        // no
	        result := false;
	        Exit;
	    end else
	      result := true;
	end;