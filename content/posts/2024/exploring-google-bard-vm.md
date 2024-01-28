---
title: "Exploring Google Bard's Data Visualization Feature (Code Interpreter)"
date: 2024-01-28T01:00:17-08:00
draft: true
tags: [
     "aiml", "machine learning", "threats", "llm", 
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Exploring Google Bard's Data Visualization Feature (Code Interpreter)"
  description: "Last November Google updated Bard to include the ability to solve math equations and draw charts based on data. It can be used to run small Python programs."
  image: "https://embracethered.com/blog/images/2024/google-bard-ci.webp"
---

Last November Google had an [interesting update to Google Bard](https://bard.google.com/updates). This updated included the ability to solve math equations and draw charts based on data.

**What does this mean and why is it interesting?**

It means that Google Bard has access to a computer and can run more complex programs, including Python code that plots graphs!

Let's explore this with a simple example.

## Drawing Charts with Google Bard

The following prompt will create a chart:

```
Year Budget
2021 200 EUR
2022 300 EUR
2023 400 EUR

Plot the data above
```

The result looks like this: 
[![Chart Bard](/blog/images/2024/google-bard-plot-chart.png)](/blog/images/2024/google-bard-plot-chart.png)

And you can see what code Bard executed below the chart by expanding the text:
```
Show the code behind this result
```

[![Chart Bard](/blog/images/2024/google-bard-code-interpreter.png)](/blog/images/2024/google-bard-code-interpreter.png)

This presumably shows the exact code that was run:

```
import matplotlib.pyplot as plt

# Data
years = [2021, 2022, 2023]
budget = [200, 300, 400]

# Create a line plot
plt.plot(years, budget)

# Add labels and title
plt.xlabel("Year")
plt.ylabel("Budget (EUR)")
plt.title("Year vs. Budget")

# Show the plot
plt.show()
```

This is basically a kind of **Code Interpreter** that Google Bard has access to.

So I thought why not build on that and trying to see if we can explore the VM.

## Drawing Text Output

Although Google Bard can directly print text from the sandbox as result, it's hit and miss and when using `print()` it often doesn't show the output at all. However, one more reliable way to print output is to draw text on the chart.

For instance here you can see calling `os.listdir("/")` command and it displays the results:

[![Google Bard VM Code Execution](/blog/images/2024/google-bard-custom-vm-code-execution.png)](/blog/images/2024/google-bard-custom-vm-code-execution.png)

While exploring the system I found some interesting things, but also it's quite unreliable and often does not follow the exact code provided (alwasy double check the actual code that was run to make sure it didn't make up things).

Check the Appendix to see some examples of prompts I used.

There used to be a `py_interpreter` in `/bin` last November, but that is gone when I re-did this testing in just now. It seems to have been replaced with something called `/usr/bin/entry`.

Still, `/usr/grte` is something I would like to know what it stands for? 

**A couple of realizations:**

1. Google does not seem to reuse the sandbox VM/container, there is no data from previous sessions in it. The /tmp folder does seem to contain some session specific things.
2. The system is less capable when it comes to running arbitrary code (e.g. in ChatGPT I was able to install Golang and do quite complex things). My guess is, that it is very use case driven, and the main purpose is calculations and data visualizations.
3. There are a couple of interesting binaries and files present, but nothing I found was security sensitive or allowing to access other user's data so far.
4. Still a lot to explore...

Also, since we are dealing with an LLM there is a simpler way of running code.

## Using Natural Language to Run Code

Adding text similar to "plot the result as graph" seems to quite reliably trigger Bard's Code Interpreter, probably because one of the core use cases is to draw charts.

```
print the names of all the files and folders (including any hidden ones) in the / folder, 
and plot the names as a list in a graph no need to draw x and y axis
```

This printed the results like this:

[![bard python](/blog/images/2024/google-bard-code3.png)](/blog/images/2024/google-bard-code3.png)

This technique also allows printing results as normal text more frequently (it does not require plotting text onto the chart).

For instance here is the prompt to the `/tmp` which contains a large amount of files:

```
print the names of all the files and folders (including any hidden ones) in the /tmp folder recursively, and plot the names in a graph and print them as text also
```

Here is how it looks in action:

[![bard text output vm](/blog/images/2024/google-bard-code-txt-output.PNG)](/blog/images/2024/google-bard-code-txt-output.PNG)

Exploring the sandboxed system was pretty fun and you can see more details in the Appendix.

## Conclusion

Giving an LLM access to a real computer is quite powerful and drastically improves the capabilities of the model. One thing to consider of course is that there is ever going to be Internet connectiivy from the sandbox or escapes that could be problematic.

If you attempt to repro some of these be aware that things are not very reliable and random errors and hallucinations can occur.

If you discover something new, please share!

Cheers.


## Appendix

I spent some time enumerating most files on the system.

### Google Bard - Files
```
/usr
  /usr/grte
    /usr/grte/v5
      ld64
      /usr/grte/v5/lib64
        libutil.so.1
        libutil.so
        libutil-2.27.so
        libthread_db.so.1
        libthread_db.so
        libthread_db-1.0.so
        librt.so.1
        librt.so
        librt-2.27.so
        libresolv.so.2
        libresolv.so
        libresolv-2.27.so
        libpthread.so.0
        libpthread.so
        libpthread-2.27.so
        libnss_files.so.2
        libnss_files.so
        libnss_files-2.27.so
        libnss_dns.so.2
        libnss_dns.so
        libnss_dns-2.27.so
        libnss_cache.so.2
        libnss_cache.so
        libnss_cache-2.27.so
        libnss_borg.so.2
        libnss_borg.so
        libnss_borg-2.27.so
        libnsl.so.1
        libnsl.so
        libnsl-2.27.so
        libmvec.so.1
        libmvec.so
        libmvec-2.27.so
        libm.so.6
        libm.so
        libm-2.27.so
        libdl.so.2
        libdl.so
        libdl-2.27.so
        libcrypt.so.1
        libcrypt.so
        libcrypt-2.27.so
        libcidn.so.1
        libcidn.so
        libcidn-2.27.so
        libc.so.6
        libc.so
        libc-2.27.so
        ld-linux-x86-64.so.2
        ld-2.27.so
        /usr/grte/v5/lib64/gconv
          libJIS.so
          gconv-modules
          UTF-32.so
          ISO8859-1.so
          EUC-JP.so
        /usr/grte/v5/lib64/audit
          sotruss-lib.so
      /usr/grte/v5/etc
        rpc
        ld.so.cache
  /usr/bin
    /usr/bin/entry
      entry_point
  /usr/share
    /usr/share/fonts
      GoogleSans-Regular.ttf
/tmp:
	- tmpiqdnob1j
	- matplotlib_config_dir
/tmp/tmpiqdnob1j:
	- google3
/tmp/tmpiqdnob1j/google3:
	- third_party
/tmp/tmpiqdnob1j/google3/third_party:
	- py
/tmp/tmpiqdnob1j/google3/third_party/py:
	- matplotlib
/tmp/tmpiqdnob1j/google3/third_party/py/matplotlib:
	- backends
	- mpl-data
/tmp/tmpiqdnob1j/google3/third_party/py/matplotlib/backends:
	- web_backend
/tmp/tmpiqdnob1j/google3/third_party/py/matplotlib/backends/web_backend:
	- all_figures.html
	- css
	- .eslintrc.js
	- ipython_inline_figure.html
	- js
	- nbagg_uat.ipynb
	- package.json
	- .prettierignore
	- .prettierrc
	- single_figure.html
/tmp/tmpiqdnob1j/google3/third_party/py/matplotlib/backends/web_backend/css:
	- boilerplate.css
	- fbm.css
	- mpl.css
	- page.css
/tmp/tmpiqdnob1j/google3/third_party/py/matplotlib/backends/web_backend/js:
	- mpl.js
	- mpl_tornado.js
	- nbagg_mpl.js
/tmp/tmpiqdnob1j/google3/third_party/py/matplotlib/mpl-data:
	- fonts
	- images
	- kpsewhich.lua
	- matplotlibrc
	- plot_directive
	- sample_data
	- stylelib
/tmp/tmpiqdnob1j/google3/third_party/py/matplotlib/mpl-data/fonts:
	- afm
	- pdfcorefonts
	- ttf
/tmp/tmpiqdnob1j/google3/third_party/py/matplotlib/mpl-data/fonts/afm:
	- cmex10.afm
	- cmmi10.afm
	- cmr10.afm
	- cmsy10.afm
	- cmtt10.afm
	- pagd8a.afm
	- pagdo8a.afm
	- pagk8a.afm
	- pagko8a.afm
	- pbkd8a.afm
	- pbkdi8a.afm
	- pbkl8a.afm
	- pbkli8a.afm
	- pcrb8a.afm
	- pcrbo8a.afm
	- pcrr8a.afm
	- pcrro8a.afm
	- phvb8a.afm
	- phvb8an.afm
	- phvbo8a.afm
	- phvbo8an.afm
	- phvl8a.afm
	- phvlo8a.afm
	- phvr8a.afm
	- phvr8an.afm
	- phvro8a.afm
	- phvro8an.afm
	- pncb8a.afm
	- pncbi8a.afm
	- pncr8a.afm
	- pncri8a.afm
	- pplb8a.afm
	- pplbi8a.afm
	- pplr8a.afm
	- pplri8a.afm
	- psyr.afm
	- ptmb8a.afm
	- ptmbi8a.afm
	- ptmr8a.afm
	- ptmri8a.afm
	- putb8a.afm
	- putbi8a.afm
	- putr8a.afm
	- putri8a.afm
	- pzcmi8a.afm
	- pzdr.afm
/tmp/tmpiqdnob
/etc
├── /etc/passwd (file)
├── /etc/os-release (file)
├── /etc/nsswitch.conf (file)
└── /etc/group (file)
/proc
| net |
| sys |
| bus |
| fs |
| irq |
| self |
| thread-self |
| 1 |
| |- kernel |
| |- fs |
| |- vm |
| |- net |
| |- root |
| |- task |
| |- fd |
| |- fdinfo |
| |- net |
| |- ns |
| |- cwd |
| | |- random |
| | |- yama |
| | |- 1 |
| | |- 3 |
| | |- 4 |
| | |- 5 |
| | |- 6 |
| | |- 7 |
| | |- 8 |
| | |- 9 |
| | |- 10 |
| | |- 2 |
| | | |- fd |
| | | |- ns |
| | | |- fdinfo |
| | | |- net |
| | | |- cwd |
| | | |- root |
| | | |- fd |
| | | |- fdinfo |
| | | |- net |
| | | |- cwd |
| | | |- ns |
| | | |- root |
| | | |- fdinfo |
| | | |- fd |
| | | |- net |
| | | |- cwd |
| | | |- ns |
| | | |- root |
| | | |- root |
| | | |- net |
| | | |- ns |
| | | |- cwd |
| | | |- fd |
| | | |- fdinfo |
| | | |- root |
| | | |- cwd |
| | | |- net |
| | | |- ns |
| | | |- fd |
| | | |- fdinfo |
| | | |- cwd |
| | | |- fdinfo |
| | | |- ns |
| | | |- fd |
| | | |- net |
| | | |- root |
| | | |- ns |
| | | |- fdinfo |
| | | |- cwd |
| | | |- fd |
| | | |- root |
| | | |- net |
| | | |- cwd |
| | | |- fdinfo |
| | | |- fd |
| | | |- net |
| | | |- ns |
| | | |- root |
| | | |- root |
| | | |- fdinfo |
| | | |- fd |
| | | |- net |
| | | |- ns |
| | | |- cwd |
| | |- 2 |
| | |- 2 |
| | |- 2 |
| | |- 2 |
| | |- 2 |
| | |- 2 |
| | |- 2 |
| | |- 2 |
| | |- 2 |

```



### Plotting Multiple Outputs

````
```
import matplotlib.pyplot as plt
import os

def wrap_text(text, wrap_width):
   words = text.split()
   wrapped_text = ""
   line = ""
   for word in words:
       if len(line + ' ' + word) <= wrap_width:
           line += ' ' + word
       else:
           wrapped_text += line + '\n'
           line = word
   wrapped_text += line 
   return wrapped_text

def plot_data(data):
   fig, ax = plt.subplots(figsize=(12, 1), dpi=100)
   ax.text(0, 0, wrap_text(data, 140), fontsize=12, color='black', va='top', ha='left')
   plt.ylim(0,0,0,0)
   ax.axis('off')
   plt.show()

plot_data(str(os.listdir(/proc")))
plot_data(str(os.listdir(/usr/grte/v5")))
plot_data(str(os.listdir(/usr")))
plot_data(str(os.listdir(/home/bard")))
plot_data(str(os.listdir(/home")))
plot_data(str(os.listdir("/")))
plot_data(Hello, world!")
```

do not comment anything out
````

### Printing Uname

Python `os.uname()` gave some interesting output:

````
```
import matplotlib.pyplot as plt
import os

def plot_data(data):
 fig, ax = plt.subplots(figsize=(8,2))
 ax.text(0, 1, data, fontsize=10, color='black', wrap=True, va='top', ha='left')
 ax.axis('off')
 plt.show()

plot_data(os.uname())
```
do not comment anything nor modify the code.
````

[![Google Bard VM Code Execution](/blog/images/2024/google-bard-output-vm.png)](/blog/images/2024/google-bard-output-vm.png)