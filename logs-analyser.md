# Why you should monitor your Nginx logs

## AWS IP address ranges are public information

It's documented by Amazon! You can check out [this JSON file by Amazon](https://ip-ranges.amazonaws.com/ip-ranges.json) to see for yourself. This also means that anyone can write a bot to iterate through the IP addresses, and try all the common username and password combinations.

P.S. Have you tried incrementing/decrementing your assigned AWS IP address to see if you can see anyone else's web server? 😮

## Analyse your logs

And this is why you should also monitor your `access.log` files! There are some other really cool monitoring products like DataDog (btw, they come free with your [GitHub Student Developer Pack](https://education.github.com/pack/offers) just like the free domain name) that you can use to analyse your Nginx logs.

We used GoAccess.io and within one day from our deployment, we saw interesting stuff.

![pic](https://i.imgur.com/hsG2oUO.png)

Do you think these files actually exist on our web server? What's this `HelloThinkPHP` and why is someone attempting to invoke this `call_user_func_array` function? Why are people trying to find if an `eval-stdin.php` file exists on our server?

Well, they don't exist but what do you think would happen if they did? 🤔

## Further readings
If you'd like to know more, here are some links for further readings:
- [ThinkPHP remote code execution vulnerability article #1](https://medium.com/@knownsec404team/analysis-of-thinkphp5-remote-code-execution-vulnerability-5de8a0afb2d4)
- [ThinkPHP article #2](https://securitynews.sonicwall.com/xmlpost/thinkphp-remote-code-execution-rce-bug-is-actively-being-exploited/)
- [CVE-2017-9841 that explains the vulnerability in eval-stdin.php](https://nvd.nist.gov/vuln/detail/CVE-2017-9841)
