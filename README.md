</h> Detecting Unmanaged PowerShell/C-Sharp Injection </h>
<h2>Description</h2>
C# is a managed language, meaning that the code you write isn't directly turned into the basic machine instructions that your computer's hardware understands. Instead, it gets converted into a special kind of intermediate code called "bytecode", which a special system called the Common Language Runtime (CLR) reads and turns into executable machine instructions. We can leverage this knowledge to detect unusual C# injections using a special utility called Process Hacker.<br />
<h2>Utilities Used</h2>
</p>- Powershell</p>
</p>- C#</p>
</p>- Process Hacker</p>
<h2>Step-by-Step Walkthrough</h2>
</p> 1. The first step is to launch the Process Hacker executable in powershell. Using Process Hacker, we can observe a range of processes within our environment, with color-coded distinctions based on their names. One thing to note is that powershell.exe is green, which signifies that it is a managed process. You can chover over each process to check its managed status </p>
<img width="1433" alt="Screenshot 2024-07-02 at 3 08 23 PM" src="https://github.com/bpark1223/Detecting-Unmanaged-PowerShell-C-Sharp-Injection/assets/77799235/88d6ddea-34a3-43e9-924e-e2ea0f07b696">
</p> 2. We can check the module loads for powershell.exe, by right-clicking on powershell.exe, clicking "Properties", and navigating to "Modules". The presence of "Microsoft .NET Runtime...", clr.dll, and clrjit.dll should attract our attention. clr.dll, and clrjit.dll are DLLs used to execute C# based bytecode. If they are loaded in processes that do not require them, it suggests a potential execute-assembly or unmanaged PowerShell injection attack.
<img width="1427" alt="Screenshot 2024-07-02 at 3 14 15 PM" src="https://github.com/bpark1223/Detecting-Unmanaged-PowerShell-C-Sharp-Injection/assets/77799235/98b7bd9a-b1a1-402a-a098-a368850183b5">
</p> 3. To simulate this attack, I inject an unmanaged PowerShell-like DLL into a random process, such as spoolsv.exe. In order to do this, i first identify the PID of spoolsv.exe, which is "2380."
<img width="1415" alt="Screenshot 2024-07-02 at 3 22 00 PM" src="https://github.com/bpark1223/Detecting-Unmanaged-PowerShell-C-Sharp-Injection/assets/77799235/3362e8cc-494e-4f81-a188-4440a334d528">
</p> 4. Using the PID of spoolsv.exe, I now perform the PowerShell attack by injecting the following code into my PowerShell terminal: </p>
</p> powershell -ep bypass </p>
</p> Import-Module .\Invoke-PSInject.ps1
</p>  Invoke-PSInject -ProcId [2380] -PoshCode "V3JpdGUtSG9zdCAiSGVsbG8sIEd1cnU5OSEi" </p>
</p> 5. I can see in Process Hacker that the spoolsv.exe is green, indicating that it is now a managed process. </p>
<img width="275" alt="Screenshot 2024-07-02 at 3 47 07 PM" src="https://github.com/bpark1223/Detecting-Unmanaged-PowerShell-C-Sharp-Injection/assets/77799235/3a8ddcf0-32d6-4a3e-8c33-bec58b4ec9c0">
</p> 6. To confirm that "clrjit.dll" has been injected into the spoolsv executable, I go to Windows Event Viewer, filter for Event ID 7, and then perform a Find for clrjit.dll 
<img width="1383" alt="Screenshot 2024-07-02 at 3 57 23 PM" src="https://github.com/bpark1223/Detecting-Unmanaged-PowerShell-C-Sharp-Injection/assets/77799235/a3d7d8f0-df2a-4115-a4e5-52fce71317e9">

