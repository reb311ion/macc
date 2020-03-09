# Data Exfiltration Via Custom Shorten Links

----

### Introduction 

Data exfiltration is the unauthorized copying, transfer or retrieval of data from a computer or server. Data exfiltration is a malicious activity performed through various techniques, typically by cybercriminals over the Internet or another network. Exfiltration consists of techniques that adversaries may use to steal data from your network. Once they’ve collected data, adversaries often package it to avoid detection. This can include compression and encryption. Data Exfiltration may happen in many ways, one of the creative ways out there is to exfiltrate data via DNS where The IP traffic is simply encoded using something like Base64, and broken into chunks that fit in DNS queries. The queries are then sent to the specially modified DNS server, where they are unpacked and sent out onto the internet. But this considered off-topic for now so we won't get deeper into data exfiltration techniques. 

### How can short links be abused for C2?

##### Background 

I was searching for an easy way to share text files quickly, a simple search led me to [transfer.sh](https://transfer.sh/), an easy file sharing service that enables users to store files right from the command line. Verhoef (the Author) once said:

> It is being used by a lot of people,” he said. “Some are using it for uploading log files, others are exporting complete video surveillance to us. **Sometimes it is being abused, by distributing malware, botnets and other malicious tools**, but we try to stop it as soon as possible. One time a porn website was serving porn photos through us, and when we found out we had all photos replaced by dogs and kittens.

![image-20200309003309937](E:\reb311ion\Misc\B1\image-20200309003309937.png)

Which isn't that surprising, Malware authors use almost everything they can reach to achieve their goals.  At this point the question was "can adversaries use the same way of sending malware for receiving back data?", and for the first moment it seemed very hard as for that to happen the attacker should know the randomly generated URI which is generated in the following format:

```
https://transfer.sh/[a-zA-Z0-9]{5,6}/MyFileName
```

Bruteforcing the URI won't be effective as the server delays its response and other methods like `HEAD` which could be used to check for the `200 OK` status code are not allowed:

![image-20200308190948868](E:\reb311ion\Misc\B1\image-20200308190948868.png)



##### Start writing code

I first took 5Min to write a simple `C#` client to use later:

<script src="https://gist.github.com/reb311ion/e9893958b4e4d7a07b1d59c2024b3cf9.js"></script>

![image-20200308193714361](E:\reb311ion\Misc\B1\image-20200308193714361.png)


After trying many things that failed I came across the idea of using custom shorten links, what I needed is the ability to create a custom shorten link in an automated manner and anonymously so the rules are:

- No API keys to access the service
- No captcha to stop us from accessing it without an API key

Surprisingly, there are services out there that let us create custom shorten links without captcha such as: 

- https://tiny.cc/
- https://bit.do/
- https://tinyurl.com/

![image-20200308194756484](E:\reb311ion\Misc\B1\image-20200308194756484.png)

The next step was to build a client for one of the above services, I chose [tiny.cc](https://tiny.cc/), this one was somehow challenging as the `POST` parameters names are getting randomized so you should parse the HTML source first and then submit the `POST` request:

<script src="https://gist.github.com/reb311ion/c29748f295bb1166c11968ca76a6fff2.js"></script>

![image-20200308200245831](E:\reb311ion\Misc\B1\image-20200308200245831.png)



##### Putting things together 

Now we can:

- Upload Files anonymously and get back the URL which we (the attacker) don't know its URI
- shorten the URL with custom URI that we know

for demo purpose I used the Keylogger [LimeLogger](https://github.com/NYAN-x-CAT/LimeLogger) to generate some data that an attacker maybe interested to harvest and built a basic C# Malware. The config section and variables including log file path was defined as the following:

<script src="https://gist.github.com/reb311ion/b1cde47fadde3d812103081a59b16ae5.js"></script>

The malware will do the following:

1 - Check if Mutex name is created. If true, terminate the process. If false, create the mutex with the given name:

<script src="https://gist.github.com/reb311ion/c0e156e9c74bb4adcf51711020ca5238.js"></script>

2 - Monitor and Log keystrokes in the `%TEMP%` directory under random name generated in the format `[a-zA-Z0-9]{10}\.log`:

<script src="https://gist.github.com/reb311ion/206d84b2c4fb9d5787081beaf3d62f50.js"></script>

3 - When log file size reaches a prespecified size, encrypt it with `AES/CBC` mode using stored `KEY` and `IV`:

<script src="https://gist.github.com/reb311ion/139414db2d424bd7ef4551ef1e47acd7.js"></script>

4 - Upload the encrypted log file to [transfer.sh](https://transfer.sh/) and create shorten `URL` using a prespecified `URI` with a number that gets incremented each time a shorten `URL` is created:

<script src="https://gist.github.com/reb311ion/7acc19f17473366327ea7a22926abe7b.js"></script>

5 - Go Back to step Two

Demo:

![image-20200308235918994](E:\reb311ion\Misc\B1\image-20200308235918994.png)



The full functioning POC can be found here: [c02](https://gist.github.com/reb311ion/3b9df9b23a73d7a327a2b6e79520fab6)

### Disclaimer

I'm not responsible for any actions, and or damages, caused by this content or the code snippets used here.
You bear the full responsibility of your actions and acknowledge that these code snippets were created for demonstration
purposes only and its main purpose is NOT to be used maliciously, or on any system that you do not own, or have the right to use.

---