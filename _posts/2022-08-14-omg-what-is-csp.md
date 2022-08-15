---
layout: post
title: OMG, what’s a CSP?
excerpt: But really, what is it? CSP, which stands for ‘Content Security Policy’, is a policy added to our request headers to the browser as an added layer of security to protect against Cross-Site Scripting (XSS) and data injection attacks. As a review, data injection attacks can be used for data theft, site defacement, and/or malware distribution.
author: david_suh
---

But really, what is it? CSP, which stands for ‘Content Security Policy’, is a policy added to our request headers to the browser as an added layer of security to protect against Cross-Site Scripting (XSS) and data injection attacks. As a review, data injection attacks can be used for data theft, site defacement, and/or malware distribution.

How does CSP work? When the browser makes a request to our server, our server includes the CSP in the request header that gets sent back to the browser. The CSP then tells the browser the sources that it should trust when loading resources such as scripts, styles, etc. Any attempts at data injection that goes against the CSP would be caught by the browser, break the site, and then report the attack back to us.

When creating the CSP, usually, the policy would have to be included in the head tag of the HTML that we serve to the browser. Something along the lines of this:

```html
<meta http-equiv=”Content-Security-Policy”
content=”default-src ‘self’; img-src https://*; child-src ‘none’ ;”>
```

This is fine, but looks pretty hard to read. Luckily, Rails comes with a ‘ready-made’ policy which we can add sources to. Here’s an mini-example of Sensei’s CSP:

```ruby
Rails.application.config.content_security_policy do |policy|
   policy.script_src  :self, :https
   policy.style_src   :self, :https, :unsafe_inline, 'http://fonts.googleapis.com/css'
   policy.default_src :self, :https, :ws
   # many more src’s can be added here
   # Specify URI for violation reports
   # policy.report_uri "/csp-violation-report-endpoint"
end

# If you are using UJS then enable automatic nonce generation
Rails.application.config.content_security_policy_nonce_generator = -> request { SecureRandom.base64(16) }
Rails.application.config.content_security_policy_nonce_directives = %w(script-src)
# Report CSP violations to a specified URI
# For further information see the following documentation:
# https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy-Report-Only
Rails.application.config.content_security_policy_report_only = true
```

Here’s a brief list of what we’re seeing here:
- `script_src`: allowed sources are from self (portal) and https. We handle inline scripts with nonces, more below
- `style_src`: you’ll see ‘unsafe_inline’ here because honestly, we have A LOT, and will need to change this in the future because this is not ideal
- `default_src`: checks against other resources that don’t have a specified source in the CSP
- `report_uri`: where we send reports to for CSP violations or warnings
- `csp_nonce_generator`: a nonce is a random secure string that is used to identify which resources are safe for the app to use
- `csp_nonce_directives`: where we tell the browser which resources we’re ‘noncing’. For example, any nonced inline scripts are allowed. Nonces are attributes that can be added to the javascript tags in our HTML, but having too many inline JS can make this difficult
- `csp_report_only`: where we determine if we want to enforce the policy or not, good for developers to use when building out the CSP, checking warnings in the chrome console during development for what sources the app currently needs to run

### Conclusion:
CSPs are extremely useful when written correctly, and can provide a lot of security while also exposing insecurities in our app. They are definitely meant to be built upon as the app grows and changes, and should be somewhere in the back of your mind the next time you think about adding inline scripts or styles.
