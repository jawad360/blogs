# The problem
As the web contents are becoming more dynamic everyday, it poses a threat of getting hacked and data being stolen. As the web browser uses script to execute dynamic content, it opens up a way wherein an attacker can inject a malicious code into trusted user website. The browser has no way to know that the script is malicious as it is present in the trusted website. This attacker script execution can access cookies, tokens and other details. One of the way where user can inject a malicious script is by exploiting a unsanitised user input. Attacker can inject a malicious script tag in input field like comments and when this comment is rendered on user, if it is not escaped correctly it'll execute the attacker script and exploit the user. These types of attacks are commonly known as XSS attack (https://owasp.org/www-community/attacks/xss/)

# What is CSP
Content-Security-Policy (CSP) provides a layer of protection from these types of attacks. Most of the XSS attacks happens as the browser is not aware of the authenticity of the script that it is executing and end up running malicious code being injected by the attacker. CSP provides a policy to inform the browser about the trusted sources of content, this includes scripts, styles, media, XHR request, etc, and only process these content. This is achieved using HTTP header defined by W3C and followed by all major browser. This helps in mitigating XSS attacks
`Content-Security-Policy: policy`
Now even if attacker is able to inject a malicious content, as it is not defined in the policy it will not be processed by the browser.

# How to use CSP
To implement a CSP a web document should contain an HTTP header. For eg
```
Content-Security-Policy: script-src 'self' https://accounts.upgrad.com
```
This informs the browser to load script content only from the current domain origin or from `https://accounts.upgrad.com'`. Now when a malicious script is present in the page document it’ll not be executed and throws an error
<Image>
Here it should be noted that the inline scripts and also `eval()` will also not be executed by default and throws an error. Policy can be configured as needed using various directives, for eg image can we used from any domain whereas script and XHR can be loaded from the current domain only, in this case the header will be
```
Content-Security-Policy: script-src 'self'; img-src *; connect-src ‘self’
```

# CSP directives
- `default-src`: This is a default directive, directive that are not specified will fallback to this default directive. If a script or style or connect etc are not specfied then it'll fallback to the default one
- `script-src` : This is the directive where you can specific the location from the script can be loaded. For eg `Content-Security-Policy: script-src 'self'` will only execute script from the current page's origin. Inline execution of sciprt will also throw and error. To support inline execution of script we should add `unsafe-inline` like `Content-Security-Policy: script-src 'self' 'unsafe-inline'`. Similarly to support execution of `eval()` add `unsafe-eval`
- `style-src` : This directive is similar to script-src directive, but this will decide from where the page styles will be loaded
- `connect-src` : This directives is used to restrict the XHR, WebSockets, etc calls which are used to connect.
- If we wish to load our resources only from a secured channel then we can add `https:` in our directive this will make sure that the specified resource will be loaded only from a secured channel `Content-Security-Policy: default-src https:`
- Different directive are sperated with ; like `script-src http.. ; style-src http..; connect-src http...`
- Assuming we are running a banking website and need to make sure only resources in the rules should be loaded, then in that case we can do `default-src 'none'` and then keep adding rules for other directives
- One can also use wildcard `*` to whitelist for eg `script-src *.upgrad.com` will execute script from all subdomain of `upgrad.com`.

# nonce and hashes
Inline script should not be used, but there are cases where it cannot be avoided when using plugins. In this case we can use nonce. Nonce are unique string for each pages and each request. This unique nonce string will be added in script tag and CSP header
```
<script nonce="EDNnf03nceIOfn39fn3e9h3sdfa">
  // Some inline code I can't remove yet, but need to asap.
</script>
```
```
Content-Security-Policy: script-src ‘nonce-EDNnf03nceIOfn39fn3e9h3sdfa’
```
In CSP header the nonce string must be appended with `nounce-` keyword.
Hashes can be used instead on nonce but in this case the script content need to be hashed using any popular hashing algorithm. This hash value will be provided in the CSP header similar to nonce
```
Content-Security-Policy: script-src ‘sha256-EDNnf03nceIOfn39fn3e9h3sdfa’
```
Here it should be noted that the `<script>` should not be hashed and in the CSP header the hash value should be appended with `sha*-`.

# Rules for Mitigating XSS
- Move inline scripts out-of-line
- Remove inline event script handler
- Avoid use of `eval()`
- Add script-src CSP policy

# Reporting
CSP also provides a mechanism to report all the CSP violations happening in real-time. This can be achieved using `report-to example.com` directive in the policy string. It send a `POST` request to `example.com`  with JSON payload containing fields like `blocked-uri`, `violated-directive`, etc.
```
Content-Security-Policy: policy report-to example.com
```
Before enforcing the actual CSP it is always good to see if the policy is working as expected and not blocking the content which is not expected. For this we can enable report only mode. This can be implemented using a similar header
```
Content-Security-Policy-Report-Only: policy report-to example.com
```
This will send a report to designated `report-to` URI. This will be helpful in configuring a good policy for a given usecase

# Useful resources
- https://www.cspisawesome.com/ here you can create CSP headers using a GUI and in a very intuitive manner.
- https://report-uri.com/home/generate using this website you can find CSP policies of different websites.
- https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html
