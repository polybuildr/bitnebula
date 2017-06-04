+++
date = "2017-06-04T11:03:08+03:00"
draft = true
tags = ["dev", "golang", "webdev"]
title = "Content-aware Autoescaping in Go"

+++
I've been working with server-side generated HTML for several years now, and the problem of code injection into HTML pages has been pervasive. A couple of days back, I discovered something fantastic that Go has built right into the standard library to help with this: context-aware autoescaping in HTML templates.
<!--more-->

I've never really worked with Go - I'm running these examples on the [Go playground](https://play.golang.org) - but to begin with, let's look at the normal templating that Go offers.

```go
package main

import (
	"os"
	"text/template"
)

func main() {
	data := make(map[string]string)
	data["name"] = "Harry"
	data["school"] = "Hogwarts"
	tmplContent := "{{ .name }} goes to {{ .school }}"
	tmpl, _ := template.New("test").Parse(tmplContent)
	tmpl.Execute(os.Stdout, data)
}
```

and as output, we get `"Harry goes to Hogwarts"`. Apart from the unfamiliar syntax, this seems simple enough. Pretty much the same thing as string formatting in, say, Python. This code:

```python
data = {
    'name': 'Harry',
    'school': 'Hogwarts'
}
tmplContent = "{name} goes to {school}"
print(tmplContent.format(**data))

# or tmplContent.format(name="Harry", school="Hogwarts")
```

gives us the same output.

However, this gets interesting when you try to generate HTML the same way. If the data was instead

```go
data["name"] = "Harry<script>alert('you have been pwned')</script>"
data["school"] = "Hogwarts"
```

we'd get `"Harry<script>alert('you have been pwned')</script> goes to Hogwarts"` which is definitely not nice. But here's where the cool stuff starts. Instead of the `text/template`, let's use the drop-in replacement `html/template`.

```go
package main

import (
	"os"
	"html/template" // note the change here
)

func main() {
	data := make(map[string]string)
	data["name"] = "Harry<script>alert('you have been pwned')</script>"
	data["school"] = "Hogwarts"
	tmplContent := "{{ .name }} goes to {{ .school }}"
	tmpl, _ := template.New("test").Parse(tmplContent)
	tmpl.Execute(os.Stdout, data)
}
```

[Try it out on the Go playground](https://play.golang.org/p/wKSWLphKpC). The output is `"Harry&lt;script&gt;alert(&#39;you have been pwned&#39;)&lt;/script&gt; goes to Hogwarts"`. The troublesome characters were escaped automagically, and now we don't have a code injection anymore. Awesome!

So we've seen the autoescaping, but what's this about "context-awareness"? That's that part that really impressed me. Go's `html/template` package _understands_ HTML, along with CSS, Javascript and URIs. It knows what kind of escaping needs to be done where.

Let's see a few examples to make better sense of this. For the data string `"<b>You're</b> weird"`, we get the following outputs from the following templates. (`.` can be used to represent the entire single input given to the template.)

```
<p>{{ . }}</p>
=> <p>&lt;b&gt;You&#39;re&lt;/b&gt; weird</p>

<p class="{{ . }}"></p>
=> <p class="&lt;b&gt;You&#39;re&lt;/b&gt; weird"></p>
(this time the single quote got encoded too)

<a href="{{ . }}"></a>
=> <a href="%3cb%3eYou%27re%3c/b%3e%20weird"></a>
(and there's URL encoding)

<script>var s = '{{ . }}';</script>
=> <script>var s = '\x3cb\x3eYou\x27re\x3c\/b\x3e weird';</script>
```

That's just HTML content, attributes, URLs, and Javascript. There's a lot more! Go's docs for the [html/template package](https://golang.org/pkg/html/template/) have detailed information.

I thought this was a fantastic way to prevent accidental code injection in HTML. Developers are prone to make mistakes, and there have been an enormous number of cases ([Facebook](https://blog.detectify.com/2012/12/30/how-i-hacked-facebook-and-received-a-3500-usd-facebook-bug-bounty/), [Stack overflow](http://odedcoster.com/blog/2013/12/15/anatomy-of-a-xss-vulnerability-on-stack-overflow/), [server-side React](https://medium.com/node-security/the-most-common-xss-vulnerability-in-react-js-applications-2bdffbcc1fa0), etc.) involving XSS vulnerabilities in websites. A templating system that intelligently handles escaping for you? Kudos!
