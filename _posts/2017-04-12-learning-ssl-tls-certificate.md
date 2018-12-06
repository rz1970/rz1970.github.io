---
title: SSL/TLS Certificate
tags: [ssl, tls]
---

## Certificate Types

Ensure that you choose a CA that offers the certificate type that you require. Many CAs offer variations of these certificate types under a variety of, often confusing, names and pricing structures. Here is a short description of each type:
* Single Domain: Used for a single domain, e.g. example.com. Note that additional subdomains, such as www.example.com, are not included
* Wildcard: Used for a domain and any of its subdomains. For example, a wildcard certificate for *.example.com can also be used for www.example.com and store.example.com
* Multiple Domain: Known as a SAN or UC certificate, these can be used with multiple domains and subdomains that are added to the Subject Alternative Name field. For example, a single multi-domain certificate could be used with example.com, www.example.com, and example.net

In addition to the aforementioned certificate types, there are different levels of validations that CAs offer. We will cover them here:
* Domain Validation (DV): DV certificates are issued after the CA validates that the requestor owns or controls the domain in question
* Organization Validation (OV): OV certificates can be issued only after the issuing CA validates the legal identity of the requestor
* Extended Validation (EV): EV certificates can be issued only after the issuing CA validates the legal identity, among other things, of the requestor, according to a strict set of guidelines. The purpose of this type of certificate is to provide additional assurance of the legitimacy of your organization's identity to your site's visitors. EV certificates can be single or multiple domain, but not wildcard
