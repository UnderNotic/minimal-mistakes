---
title: "<connectionManagement> can ruin your day"
excerpt: "Parallelizing http calls."
header:
  teaser: "assets/images/connectionManagement-can-ruin-your-day.png"
tags: 
  - .net
  - .netcore
  - c#
  - networking
---

Syntax highlighting is a feature that displays source code, in different colors and fonts according to the category of terms. This feature facilitates writing in a structured language such as a programming language or a markup language as both structures and syntax errors are visually distinct. Highlighting does not affect the meaning of the text itself; it is intended only for human readers.[^1]

[^1]: <http://en.wikipedia.org/wiki/Syntax_highlighting>

### GFM Code Blocks

GitHub Flavored Markdown [fenced code blocks](https://help.github.com/articles/creating-and-highlighting-code-blocks/) are supported. To modify styling and highlight colors edit `/_sass/syntax.scss`.

```css
#container {
  float: left;
  margin: 0 -240px 0 0;
  width: 100%;
}
```

{% highlight scss %}
.highlight {
  margin: 0;
  padding: 1em;
  font-family: $monospace;
  font-size: $type-size-7;
  line-height: 1.8;
}
{% endhighlight %}

```html
{% raw %}<nav class="pagination" role="navigation">
  {% if page.previous %}
    <a href="{{ site.url }}{{ page.previous.url }}" class="btn" title="{{ page.previous.title }}">Previous article</a>
  {% endif %}
  {% if page.next %}
    <a href="{{ site.url }}{{ page.next.url }}" class="btn" title="{{ page.next.title }}">Next article</a>
  {% endif %}
</nav><!-- /.pagination -->{% endraw %}
```

{% highlight html linenos %}
{% raw %}<nav class="pagination" role="navigation">
  {% if page.previous %}
    <a href="{{ site.url }}{{ page.previous.url }}" class="btn" title="{{ page.previous.title }}">Previous article</a>
  {% endif %}
  {% if page.next %}
    <a href="{{ site.url }}{{ page.next.url }}" class="btn" title="{{ page.next.title }}">Next article</a>
  {% endif %}
</nav><!-- /.pagination -->{% endraw %}
{% endhighlight %}

```csharp
using RestSharp;
using System;
using System.IO;
using System.Linq;
using System.Net;
using System.Net.Http;
using System.Net.NetworkInformation;
using System.Threading;
using System.Threading.Tasks;

namespace ConsoleApp2
{
    class TextHolder
    {
        public string Text { get; set; }
    }

    class Program
    {
        private static readonly string _address = "www.google.com";
        private static readonly string _fullAddress = $"http://{_address}";
        private static readonly string _addressIp = Dns.GetHostAddresses(_address)[0].ToString();
        private static readonly HttpClient _httpClient = new HttpClient();
        private static readonly RestClient _restClient = new RestClient(_fullAddress);

        static void Main(string[] args)
        {
            var timer = new Timer(
                PrintOpenTcpConnections,
                new TextHolder(),
                TimeSpan.Zero,
                TimeSpan.FromSeconds(1)
            );
            Run();
            Console.ReadLine();
            timer.Dispose(); //without this timer is garbage collected
        }

        static async void Run()
        {
            Console.WriteLine("\n HttpWebRequest");
            for (var i = 0; i < 6; i++)
            {
                CallWithWebRequest();
            }
            await Task.Delay(2000);
            Console.WriteLine("\n HttpClient");
            for (var i = 0; i < 6; i++)
            {
                CallWithHttpClient();

            }
            await Task.Delay(2000);
            Console.WriteLine("\n RestSharp");
            for (var i = 0; i < 6; i++)
            {
                CallWithRestSharp();
            }
        }

        static async void CallWithWebRequest()
        {
            var req = WebRequest.CreateHttp(_fullAddress);
            req.Method = "GET";
            req.KeepAlive = true;
            req.Timeout = 5000;
            req.Pipelined = false;
            req.UnsafeAuthenticatedConnectionSharing = false;
            var resp = await req.GetResponseAsync();
            resp.Close(); // Used to fulfill the request
            PrintReceived();
        }

        static async void CallWithHttpClient()
        {
            var resp = await _httpClient.GetAsync(_fullAddress);
            PrintReceived();
        }

        static async void CallWithRestSharp()
        {
            var resp = await _restClient.ExecuteTaskAsync(new RestRequest(Method.GET));
            PrintReceived();
        }

        static void PrintOpenTcpConnections(object state)
        {
            var textHolder = (TextHolder)state;

            IPGlobalProperties ipGlobalProperties = IPGlobalProperties.GetIPGlobalProperties();
            TcpConnectionInformation[] tcpConnInfoArray = ipGlobalProperties.GetActiveTcpConnections();

            var text = tcpConnInfoArray
                .Where(tcpi => tcpi.RemoteEndPoint.Address.ToString() == _addressIp)
                .Aggregate("", (acc, item) => $"{acc} \n Local: ${item.LocalEndPoint} Remote: ${item.RemoteEndPoint}");
            if (text != string.Empty && textHolder.Text != text)
            {
                textHolder.Text = text;
                Console.Write("\nTCP Connections:");
                Console.WriteLine(text);
            }
        }

        static void PrintReceived()
        {
            Console.WriteLine($"Response received at {DateTime.Now.ToString("ss:ffff")}");
        }
    }
}
```

### Code Blocks in Lists

Indentation matters. Be sure the indent of the code block aligns with the first non-space character after the list item marker (e.g., `1.`). Usually this will mean indenting 3 spaces instead of 4.

1. Do step 1.
2. Now do this:
   
   ```ruby
   def print_hi(name)
     puts "Hi, #{name}"
   end
   print_hi('Tom')
   #=> prints 'Hi, Tom' to STDOUT.
   ```
        
3. Now you can do this.

### GitHub Gist Embed

An example of a Gist embed below.

{% gist mmistakes/6589546 %}