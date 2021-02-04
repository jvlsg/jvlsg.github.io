---
layout: post
title:  "Building a Broken Window: web security by counter-example pt.4"
categories: Projects Security
tags: projects security web HeadPage
---

[Part 1]({% link _posts/2020-06-30-Headpage-pt1.md %})
[Part 2]({% link _posts/2020-07-20-Headpage-pt2.md %})
[Part 3]({% link _posts/2020-08-14-Headpage-pt3.md %})

Welcome, friends, to the last part of this series. 

# General Defenses and Mitigations

First, let us quickly discuss two important techniques to make applications more resilient to the attacks we've seen thus far.

## Sanitize Input

![Input Whitelisting](/assets/headpage/input_whitelisting.jpg)

I have nagged through all posts about the dangers of user input, and so here we are. 

By considering any and all **user input as unsafe by default**, while also bearing in mind other sources of potentially dangerous input (partner networks/applications, intranet, and of course internet), we should make sure that said input is properly "decontaminated" before being used by other parts of the code. 

It goes by many names, **Input Whitelisting, Input Validation, Input Sanitization**, etc. Whatever you call it, do it.

Proper sanitization can defend against all the injection vulnerabilities we covered, path traversal and unrestricted file upload, i.e. almost all of the ones we discussed during this series. 

In general, and like in many other areas of security, it is best to **define a whitelist** of what the user is allowed to input and block everything else, rather than trying to figure out all possible "dangerous" characters an attacker might use. 

Sanitization can be implemented in many different ways, depending on the specifics of the technology / feature in use. Below are a few examples to illustrate our point:

- Limiting the type of data that can be input (e.g. only Alphanumeric Characters for certain fields, using regular expressions for the allowed types).
- Limiting input length
- Validating and restricting true filetypes for uploaded files
- Casting data to their appropriate data types
- Escaping / sanitizing previously stored that is displayed by the application (such as using Django's `autoescape` feature)
- etc.

For more on this subject, please refer to the following links

* https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html
* https://github.com/trendmicro/SecureCodingDojo/blob/master/trainingportal/static/codeBlocks/inputAllowListing.html
* https://dwheeler.com/secure-programs/Secure-Programs-HOWTO/validation-basics.html


## Parameterized Commands

![Shape sorter toy](/assets/headpage/parameterized_command.jpg)

_Placing user input in the parameters. Only the correct ones pass_

Another strategy to mitigate injection attacks, in particular SQL and Command injections, is to make use of parameterized commands. Instead of directly concatenating user input to a pre-built, literal command/query, we make use of functions that handle arguments to the query in a secure manner.

Below we demonstrate the use of parameterized commands in python, using the `subprocess` module to securely run the `ls` command, as well as a failed attempt to inject another command.

```
>>> x = "./"
>>> subprocess.run(["ls", "-l", "./"])
total 52
drwxr-xr-x  2 j j  4096 jul 29 22:34 Desktop
drwxr-xr-x 11 j j  4096 ago  1 12:05 Documents
drwxr-xr-x  4 j j 12288 ago 17 22:33 Downloads
...
CompletedProcess(args=['ls', '-l', './'], returncode=0)

>>> x="./; touch ./evil"
>>> subprocess.run(["ls","-l",x])
ls: cannot access './; touch ./evil': No such file or directory
CompletedProcess(args=['ls', '-l', './; touch ./evil'], returncode=2)

```
Notice how the error message states that we are trying to list a file/directory literally named `./; touch ./evil'`, that is, our attempt of adding an additional command was thwarted as the contents of `x` where not directly interpreted.

SQL queries can similarly be parameterized through the use of stored SQL procedures in the database, or using functions available on multiple programming languages, see [This link](https://cheatsheetseries.owasp.org/cheatsheets/Query_Parameterization_Cheat_Sheet.html) for some great examples.

In HeadPage's case however, it would make more sense to simply use the functions available in it's `model` classes to indirectly query and modify the database, like we discussed in [Part 2]({% link _posts/2020-07-20-Headpage-pt2.md %}).

# Conclusion

## Regarding secure development

It doesn't matter if you successfully develop a secure system, function, application, etc. if it's usage is obtuse, and threatens to slow business momentum, chances are it won't be used. 

The technologies used in HeadPage, mainly Django and Python, go in the other direction however, providing developers with **sturdy, yet easy to use tools** which remove the need to reimplement basic (or sometimes complex) countermeasures needed to make their applications more secure. 

Speaking anecdotally, this seems to be a trend among modern, production tested, languages, frameworks and technologies, and even if we can't ever rid ourselves from code vulnerabilities and threats, reducing their number or likelihood is most definitely a net positive. 

## This project as a whole

The HeadPage (side-)project, from it's inception in April to the time of this writing, lasted for around four months and while it lasted far more than I would have initially intended or preferred, it provided the perfect opportunity to study and put in practice a wide array of areas and topics related to web applications and computer security in general.

Python and Django, HTTP, in-transit cryptography with SSL/TLS, secure password hashing, SQL, shells and reverse shells. Even if these topics could not be covered in the depth expected by "higher level" security professionals, and the final and vague goal of "excellence" is still a long way ahead, finishing this project did help me to better see how each of the cogs fall in their respective places and the way they spin to make the machine run. 

Above all else, I hope these pages were also helpful in anyway to others who also yearn to improve themselves in this field. 

**Thank you very much for reading.**
