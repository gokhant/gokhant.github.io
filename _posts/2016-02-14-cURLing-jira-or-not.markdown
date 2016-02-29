---
layout: post
title:  "52-cURLing Jira or Not?"
date:   2016-02-14 19:02:05 +0200
categories: [programming, general, jira]

tags: [programming, jira, cURL, openSSL, rest]

---
![Curling](/images/52/curling.jpg)
\\
\\
To cURL or not to cURL, that is the question. Most of the developers are familiar with [Atlassian's Jira](https://www.atlassian.com/software/jira).
While using jira, you may require to do some bulk operations; such as bulk modifying some issues: 
changing their *Affects version/s, Component/s etc*... 
\\
\\
This is easy; search the issues you'd like to change and then choose Bulk Operation. 
\\
\\
But what if you don't have bulk operation permission or you require to change something other than issues?
Suppose, you have lots of components and lots of versions for these components in your project and you lose track of them. 
It's a disaster when you want to select a version for an issue, an unending combobox of released and unrelased versions awaits you. 
You can archive unnecessary versions from project administration *one by one* so that they will not exist in the version combobox anymore; 
but it's obvious that it will take too much time to click each of them.
\\
\\
Jira has a [REST API](https://docs.atlassian.com/jira/REST/latest/) (who doesn't?) to use. This api can be used to archive the unwanted versions.
I will use Atlassian's DEMO project as a showcase. First of all; let's take all the versions of our 
project by calling [versions rest api](https://jira.atlassian.com/rest/api/latest/project/DEMO/versions):
\\
&nbsp;
{% highlight json linenos=table %}
[
{
	self: "https://jira.atlassian.com/rest/api/latest/version/12254",
	id: "12254",
	name: "Production",
	archived: false,
	released: false,
	projectId: 10820
},
{
	self: "https://jira.atlassian.com/rest/api/latest/version/12252",
	id: "12252",
	name: "Test",
	archived: false,
	released: false,
	projectId: 10820
},
{
	self: "https://jira.atlassian.com/rest/api/latest/version/12251",
	id: "12251",
	name: "Build",
	archived: false,
	released: true,
	projectId: 10820
},
{
	self: "https://jira.atlassian.com/rest/api/latest/version/12250",
	id: "12250",
	name: "Design",
	archived: false,
	released: false,
	projectId: 10820
}
][
{
	self: "https://jira.atlassian.com/rest/api/latest/version/12254",
	id: "12254",
	name: "Production",
	archived: false,
	released: false,
	projectId: 10820
},
{
	self: "https://jira.atlassian.com/rest/api/latest/version/12253",
	id: "12253",
	name: "Pilot",
	archived: true,
	released: false,
	projectId: 10820
}
]
{% endhighlight %}

The unarchived *(archived:false)* and unreleased *(released:false)* versions can easily be extracted from 
the JSON output above. For the appropriate versions, changing archived property is as simple as sending 
a PUT request to the URL in self attribute above. Here is the [api method guide](https://docs.atlassian.com/jira/REST/latest/#api/2/version-updateVersion).
\\
\\
[cURL](https://curl.haxx.se/) can be used to achieve this, it's just simple as that (Windows users, use cURL with SSL support):
\\
&nbsp;
{% highlight bash %}
curl -D- -u UsEr:PaZzWoRd -X PUT --data '{"archived": true}' -H "Content-Type: application/json" https://jira.atlassian.com/rest/api/latest/version/12251
{% endhighlight%}
\\
But wait! If you just do it like above, you will not succeed. Since you're trying to access 
an HTTPS resource, you require a CA certificate.  You can extract one  by using Internet Explorer 
or by using openssl, please check [this guide](https://curl.haxx.se/docs/sslcerts.html). After succesful extraction of the certificate, 
you can use cURL and if everything works well, you will get *HTTP/1.1 200 OK*.
\\
&nbsp;
{% highlight bash %}
curl -D- -u [User]:[Password] -X PUT --cacert [CertificateFile] --data '{"archived": true}' -H "Content-Type: application/json" https://jira.atlassian.com/rest/api/latest/version/12251
{% endhighlight%}
\\
According to your environment, it's easy to write a script to use curl for each 
version you'd like to update. Here is a sample to assign an issue to a specific user:
\\
&nbsp;
{% highlight bash %}
curl -D- -u [User]:[Password] -X PUT --cacert [CertificateFile] --data '{"fields": {"assignee":{"name":"gokhan.tuna"}}} https://jira.atlassian.com/rest/api/latest/issue/DEMO-1001 
{% endhighlight%}
\\
Here is another sample to make a [transition](https://docs.atlassian.com/jira/REST/latest/#api/2/issue-doTransition) (resolve, skip code review etc) on a specific issue. 
Please note that, instead of *PUT, POST* request has been used:
\\
&nbsp;
{% highlight bash %}
curl -D- -u [User]:[Password] -X POST --cacert [CertificateFile]  --data '{ "transition": { "id": "811" } }' -H "Content-Type:application/json" https://jira.atlassian.com/rest/api/latest/issue/DEMO-1001/transitions
{% endhighlight%}
\\
For the available transitions on an issue, you can visit [issue method](https://jira.atlassian.com/rest/api/latest/issue/DEMO-1001/transitions) of the api. 
Do not forget to check the [REST API](https://docs.atlassian.com/jira/REST/latest/) of Jira, there are tons of methods to explore.
