---
title: "Leveraging built-in operating system indexing features for credential hunting"
date: 2020-04-26T11:56:51-07:00
draft: true
tags: [
        "red",
        "ttp",
        "book"
    ]
---

In the previous post we discussed the importance of performing active credential hunting for your organization to ensure clear text credentials in widely accessible locations and source code are identified before an adversary gets a hold of them. 

**Adversaries also leverage such techniques post-exploitation when looting workstations and servers for credentials and other interesting data.**

In this post we are going to explore using built-in operating system indexing features to search for information in files on the machine. We focus on Windows in this post, and you can find similar technqiues and tips in my book for macOS.

## Windows Built-in Indexing Features

**Querying the index is fast**, especially compared to manually grep'ing through files. And using indexing is especially useful when you want to search binary file types or generally files that have custom word-breakers or formats. 

In Windows the OS is indexing files constantly, and you can inspect the configuration and what kind of files are indexed by visiting the **"Indexing Options"** settings page:

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

I added this function to my [PowerShell Profile](https://devblogs.microsoft.com/scripting/understanding-the-six-powershell-profiles/), since its also useful during day to day work (not just red teaming).

There are more code examples on how to use Windows Search with a variety of technology stacks available on [Microsoft's Github](https://github.com/microsoft/Windows-classic-samples/tree/master/Samples/Win7Samples/winui/WindowsSearch). 

Let’s look at the search query language features in a bit more detail.

### Exploring more advanced full-text search features

There are a wide set of features that can be leveraged when querying, such as **CONTAINS**, **FREETEXT**, and **LIKE** usage in the **WHERE** clause. This following [link](https://docs.microsoft.com/en-us/windows/win32/search/-search-sql-ovwofsearchquery) gives an overview of Windows Search, as well as the query language’s capabilities, including *relevance* and *ranking*.

Feel free to read up on the documentation to explore the full power of full text searching.

### Searching the index of remote machines

If a host has SMB exposed its also possible (depending on configuration) to **search its index remotely**. This is extremly useful when searching for information on SMB file servers for instance. There are more details about this in my book, but basically you can leverage the **SCOPE** parameter in the query to do that.

That's it, hope this was useful.

### Red Team Strategies
If you liked this post and found it informative or inspirational, you might be interested in the book ["Cybersecurity Attacks - Red Team Strategies"](https://www.amazon.com/Cybersecurity-Attacks-Strategies-practical-penetration-ebook/dp/B0822G9PTM). The book is filled with creative ideas, research and fundamental techniques, as well as red teaming program and people mangement aspects.


## References
* [Windows Search - Examples](https://github.com/microsoft/Windows-classic-samples/tree/master/Samples/Win7Samples/winui/WindowsSearch)
* [Windows Search - SQL Overview](https://docs.microsoft.com/en-us/windows/win32/search/-search-sql-ovwofsearchquery)
* [PowerShell Profiles](https://devblogs.microsoft.com/scripting/understanding-the-six-powershell-profiles/)
