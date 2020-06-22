---
title: "Using built-in OS indexing features for credential hunting"
date: 2020-06-22T10:00:51-07:00
draft: true
tags: [
        "red",
        "ttp"
    ]
---

A few months ago we discussed the importance of [performing active credential hunting for your organization](/blog/posts/2020/hunting-for-credentials). 

This is to ensure clear text credentials in widely accessible locations and source code are identified before an adversary gets a hold of them. 

In this post we will explore **using built-in operating system indexing features** to search for information on the machine quickly.

Many of us use the indexing features (like Windows Search and Spotlight) daily via the UI. 

However it's also possible to query the index via command line. We will cover this for Windows and macOS:

* [Windows Indexing](#windows-built-in-indexing-features)
* [macOS Spotlight](#macos-and-spotlight)

Let's start with Windows.

# Windows Built-in Indexing Features

**Querying the index is fast**, especially compared to manually grep'ing through files. In this section we will create a simple script named `Invoke-WindowsSearch`, which will allow to search the index as seen in this screenshot:

![Invoke-WindowsSearch Example](/blog/images/2020/invoke-windowssearch.png)

Using indexing is also useful for **searching binary file types** or generally, **files that require custom word-breakers** or formats. 

First, in Windows the OS is indexing files constantly, and you can inspect the configuration and what kind of files are indexed by visiting the **"Indexing Options"** settings page:

![Windows Search - Index Options](/blog/images/2020/windows-indexing.png)

The configuration shows if the index includes meta data or also the file contents per each file type.

## Searching the Index

The index is stored in a SQL database and the following script shows how to connect and search using PowerShell, but you can use any other technology that you are comfortable with.

You can also find this script on my [Github:](https://github.com/wunderwuzzi23/searchutils/blob/master/Invoke-WindowsSearch.ps1)

```
function Invoke-WindowsSearch
{
    param
    (
     [Parameter()][string] $SearchString = "password"
    )
    $SearchString = $SearchString.Replace("'","''")
    $query   = "select system.itemname, system.itempathdisplay from systemindex where contains('$SearchString')"
    $provider = "Provider=Search.CollatorDSO.1;Extended?PROPERTIES='Application=Windows'"
    $adapter  = new-object System.Data.OleDb.OleDBDataAdapter -Argument $query, $provider
    $results = new-object System.Data.DataSet

    $adapter.Fill($results)
    $results.Tables[0]
}
```

*You'll notice escaping the single-quote, which is something you might not have to worry about as long as you aren't running it as part of a trusted subsystem, or take untrusted input.*

The following screenshot shows searching the entire machine (if running as an admin) for passwords within a few moments by querying the index:
 
![Invoke-WindowsSearch Example](/blog/images/2020/invoke-windowssearch.png)

From functional point of view, you can add this function to your [PowerShell Profile](https://devblogs.microsoft.com/scripting/understanding-the-six-powershell-profiles/), since its also useful during day to day work (not just red teaming).

There are more code examples on how to use Windows Search with a variety of technology stacks available on [Microsoft's Github](https://github.com/microsoft/Windows-classic-samples/tree/master/Samples/Win7Samples/winui/WindowsSearch). 

Let’s look at the search query language features in a bit more detail.

## Advanced search features

There are a wide set of features that can be leveraged when querying, such as **CONTAINS**, **FREETEXT**, and **LIKE** usage in the **WHERE** clause. This following [link](https://docs.microsoft.com/en-us/windows/win32/search/-search-sql-ovwofsearchquery) gives an overview of Windows Search, as well as the query language’s capabilities, including *relevance* and *ranking*.

Feel free to read up on the documentation to explore the full power of full text searching.

## Searching the index of remote Windows machines

**If a host has SMB exposed its also possible (depending on configuration) to search its index remotely**. This is extremly useful when searching for information on SMB file servers for instance. There are more details about this in my book, but basically you can leverage the **SCOPE** parameter in the query to do that.


# macOS and Spotlight

On macOS its super simple, just use the `mdfind` command.

The basic use case is as follows:

```
mdfind password
```

More information can be found here: https://www.unix.com/man-page/osx/1/mdfind/

That's it. :)

-------

## Red Team Strategies
If you liked this post and found it informative or inspirational follow me on Twitter  [@wunderwuzzi23](https://twitter.com/wunderwuzzi23). You might also be interested in my book ["Cybersecurity Attacks - Red Team Strategies"](https://www.amazon.com/Cybersecurity-Attacks-Strategies-practical-penetration-ebook/dp/B0822G9PTM). 

## References
* [Windows Search - Examples](https://github.com/microsoft/Windows-classic-samples/tree/master/Samples/Win7Samples/winui/WindowsSearch)
* [Windows Search - SQL Overview](https://docs.microsoft.com/en-us/windows/win32/search/-search-sql-ovwofsearchquery)
* [PowerShell Profiles](https://devblogs.microsoft.com/scripting/understanding-the-six-powershell-profiles/)
* [mdfind](https://www.unix.com/man-page/osx/1/mdfind/)
* [Embrace the Red](https://embracethered.com/index.html)

