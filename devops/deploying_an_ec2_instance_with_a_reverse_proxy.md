## Steps to Deploy

Steps to Deploy:

1 Spun up a Linux machine with .Net preinstalled 

2 Created a console app that listens to all IPs on port 8888 and run to verify

3 Installed Apache server `sudo yum install httpd`

4 Configure the appache config to forward port 80 traffic to our app 

`sudo vim /etc/httpd/conf/httpd.conf`

```html
<VirtualHost *:80>
    ProxyPreserveHost On
    ProxyPass "/" "http://0.0.0.0:8888/"
    ProxyPassReverse "/" "http://0.0.0.0:8888/"
</VirtualHost>
```

5 start Apache `sudo service httpd start`

6 start the .net app 

7 use browser access the website on port 80

code:

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Net;
using System.Text;

namespace DumpHttpRequests
{
    internal class Program
    {
        private static void Main(string[] args)
        {
            if (!HttpListener.IsSupported)
            {
                Console.WriteLine("Windows XP SP2 or Server 2003 is required to use the HttpListener class.");
                return;
            }
            // URI prefixes are required,
            var prefixes = new List<string>() { "http://*:8888/" };

            // Create a listener.
            HttpListener listener = new HttpListener();
            // Add the prefixes.
            foreach (string s in prefixes)            {
                listener.Prefixes.Add(s);
            }
            listener.Start();
            Console.WriteLine("Listening...");
            while (true)
            {
                // Note: The GetContext method blocks while waiting for a request.
                HttpListenerContext context = listener.GetContext();

                HttpListenerRequest request = context.Request;

                string documentContents;
                using (Stream receiveStream = request.InputStream)
                {
                    using (StreamReader readStream = new StreamReader(receiveStream, Encoding.UTF8))
                    {
                        documentContents = readStream.ReadToEnd();
                    }
                }
                Console.WriteLine($"Recived request for {request.Url}");
                Console.WriteLine(documentContents);

                // Obtain a response object.
                HttpListenerResponse response = context.Response;
                // Construct a response.
                string responseString = "<HTML><BODY> Hello world!</BODY></HTML>";
                byte[] buffer = System.Text.Encoding.UTF8.GetBytes(responseString);
                // Get a response stream and write the response to it.
                response.ContentLength64 = buffer.Length;
                System.IO.Stream output = response.OutputStream;
                output.Write(buffer, 0, buffer.Length);
                // You must close the output stream.
                output.Close();
            }
            listener.Stop();
        }
    }
}

```

  ssh -i ~/Downloads/ben-hello-world.pem ec2-user@ec2-54-206-11-250.ap-southeast-2.compute.amazonaws.com
  
  look into how to snapshot a machine too 
